# okhttp 连接

负责的拦截器：ConnectInterceptor



# 分析 ConnectInterceptor.intercept

```java
@Override public Response intercept(Chain chain) throws IOException {
  RealInterceptorChain realChain = (RealInterceptorChain) chain;
  Request request = realChain.request();
  Transmitter transmitter = realChain.transmitter();

  // 如果是 GET 请求，doExtensiveHealthChecks 就为 false，不做大量的健康检查
  // We need the network to satisfy this request. Possibly for validating a conditional GET.
  boolean doExtensiveHealthChecks = !request.method().equals("GET");
  Exchange exchange = transmitter.newExchange(chain, doExtensiveHealthChecks);

  return realChain.proceed(request, transmitter, exchange);
}
```

拦截方法里面代码很少，主要是为了生成一个 Exchange 对象，那 Exchange 是做什么的呢？



## Transmitter.newExchange

中文为“交换”，也许是负责数据交换用的。要构造 Exchange，得先构造一个 ExchangeCodec，那么 ExchangeCodec 又是什么？交换用的编码解码器？

```java
Exchange newExchange(Interceptor.Chain chain, boolean doExtensiveHealthChecks) {
  synchronized (connectionPool) {
    if (noMoreExchanges) {
      throw new IllegalStateException("released");
    }
    if (exchange != null) {
      throw new IllegalStateException("cannot make a new request because the previous response "
                                      + "is still open: please call response.close()");
    }
  }

  // 先要构造一个 ExchangeCodec
  ExchangeCodec codec = exchangeFinder.find(client, chain, doExtensiveHealthChecks);
  Exchange result = new Exchange(this, call, eventListener, exchangeFinder, codec);

  synchronized (connectionPool) {
    this.exchange = result;
    this.exchangeRequestDone = false;
    this.exchangeResponseDone = false;
    return result;
  }
}
```



### ExchangeCodec & ExchangeFinder

```java
ExchangeCodec codec = exchangeFinder.find(client, chain, doExtensiveHealthChecks);
Exchange result = new Exchange(this, call, eventListener, exchangeFinder, codec);
```

ExchangeCodec 是通过 exchangeFinder 找到的，exchangeFinder 在 RetryAndFollowUpInterceptor.intercept() 的时候通过 **transmitter.prepareToConnect(request)** 创建的：

### ExchangeFinder

```java
ExchangeFinder(Transmitter transmitter, RealConnectionPool connectionPool,
               Address address, Call call, EventListener eventListener) {
  this.transmitter = transmitter;
  this.connectionPool = connectionPool;
  this.address = address;
  this.call = call;
  this.eventListener = eventListener;
  this.routeSelector = new RouteSelector(
    address, connectionPool.routeDatabase, call, eventListener);
}
```

ExchangeFinder 的创建发生在 `Transmitter.prepareToConnect`

```java
// Transmitter.java
public void prepareToConnect(Request request) {
  if (this.request != null) {
    if (sameConnection(this.request.url(), request.url()) 
        && exchangeFinder.hasRouteToTry()) {
      return; // Already ready.
    }
    if (exchange != null) throw new IllegalStateException();

    if (exchangeFinder != null) {
      maybeReleaseConnection(null, true);
      exchangeFinder = null;
    }
  }

  this.request = request;
  this.exchangeFinder = 
    new ExchangeFinder(this, connectionPool, 
                       createAddress(request.url()), call, eventListener);
}
```



### ExchangeFinder.find 

通过这个方法找到 ExchangeCodec，核心在于找到个一个 RealConnection 而且是 HealthyConnection，然后用这个 RealConnection.newCodec 构造一个 ExchangeCodec

```java
public ExchangeCodec find(
  OkHttpClient client, Interceptor.Chain chain, boolean doExtensiveHealthChecks) {
  int connectTimeout = chain.connectTimeoutMillis();
  int readTimeout = chain.readTimeoutMillis();
  int writeTimeout = chain.writeTimeoutMillis();
  int pingIntervalMillis = client.pingIntervalMillis();
  boolean connectionRetryEnabled = client.retryOnConnectionFailure();

  try {
    RealConnection resultConnection = findHealthyConnection(connectTimeout, readTimeout,
                                                            writeTimeout, pingIntervalMillis, connectionRetryEnabled, doExtensiveHealthChecks);
    return resultConnection.newCodec(client, chain);
  } catch (RouteException e) {
    trackFailure();
    throw e;
  } catch (IOException e) {
    trackFailure();
    throw new RouteException(e);
  }
}
```



#### ExchangeFinder.findHealthyConnection

findHealthyConnection 方法做的事情，找到一个连接，且是 healthy 的，不然重复这个过程，直到找到一个 healthy 的连接为止。因此

```java
/**
   * Finds a connection and returns it if it is healthy. If it is unhealthy the process is repeated
   * until a healthy connection is found.
   */
private RealConnection findHealthyConnection(int connectTimeout, int readTimeout,
    int writeTimeout, int pingIntervalMillis, boolean connectionRetryEnabled,
    boolean doExtensiveHealthChecks) throws IOException {
  while (true) {
    RealConnection candidate = findConnection(connectTimeout, readTimeout, writeTimeout,
        pingIntervalMillis, connectionRetryEnabled);

    // 找到一个成功次数为 0 的，并且不是多工的连接（非多工即不是 http2），这就是个健康的连接
    // If this is a brand new connection, we can skip the extensive health checks.
    synchronized (connectionPool) {
      if (candidate.successCount == 0 && !candidate.isMultiplexed()) {
        return candidate;
      }
    }

    // 判断这个连接是否是 Healthy，主要是 socket 的判断和 http2 的判断，详细的看 isHealthy 实现
    // Do a (potentially slow) check to confirm that the pooled connection is still good. If it
    // isn't, take it out of the pool and start again.
    if (!candidate.isHealthy(doExtensiveHealthChecks)) {
      candidate.noNewExchanges();
      continue;
    }

    return candidate;
  }
}
```

##### ExchangeFinder.findConnection [详见下文]

查找连接涉及到初次查找，查找不到进而创建连接，以及非非初次查找进而重用连接的内容。

详细分析见 <分析 ExchangeFinder.findConnection>



##### RealConnection.isHealthy

```java
public boolean isHealthy(boolean doExtensiveChecks) {
  // socket 关闭，或者输入或者输出通道关闭了都是 no healthy
  if (socket.isClosed() || socket.isInputShutdown() || socket.isOutputShutdown()) {
    return false;
  }

  // http2 的 Healthy 判断后面深入再讨论
  if (http2Connection != null) {
    return http2Connection.isHealthy(System.nanoTime());
  }

  // 要进行大量检查的情况，请求是 GET 方法，doExtensiveChecks 为 false
  // 得到原本的 socket 读超时时间，将这个连接的 socket 读超时改为 1 ms（1 ms 还没读完，就超时了）
  // source = Okio.buffer(Okio.source(socket));
	// source.exhausted() 数据如果已经全部读完了会返回 true
  // 那么读超时设置这么短，如果还有数据，那肯定会读超时，超时了说明这个 socket 还 healthy
  // 最后再把这个连接的 socket 的超时时间改会原来的
  // 期间发生 io 异常了，说明 socket 关闭了，也是 no healthy
  if (doExtensiveChecks) {
    try {
      int readTimeout = socket.getSoTimeout();
      try {
        socket.setSoTimeout(1);
        if (source.exhausted()) {
          return false; // Stream is exhausted; socket is closed.
        }
        return true;
      } finally {
        socket.setSoTimeout(readTimeout);
      }
    } catch (SocketTimeoutException ignored) {
      // Read timed out; socket is good.
    } catch (IOException e) {
      return false; // Couldn't read; socket is closed.
    }
  } // if (doExtensiveChecks)

  return true;
}
```



#### RealConnection.newCodec

找到了一个 Healthy 的连接就可以通过 newCodec 创建 ExchangeCodec

```java
// RealConnection.java
ExchangeCodec newCodec(OkHttpClient client, Interceptor.Chain chain) 
  throws SocketException {
  if (http2Connection != null) {
    return new Http2ExchangeCodec(client, this, chain, http2Connection);
  } else {
    socket.setSoTimeout(chain.readTimeoutMillis());
    source.timeout().timeout(chain.readTimeoutMillis(), MILLISECONDS);
    sink.timeout().timeout(chain.writeTimeoutMillis(), MILLISECONDS);
    return new Http1ExchangeCodec(client, this, source, sink);
  }
}
```

- Http2 使用 **Http2ExchangeCodec**
- Http1.x 使用 **Http1ExchangeCodec**
- 两个类都是 ExchangeCodec 的实现

#### 

### ExchangeCodec -> Exchange -> Interceptor

ExchangeFinder 通过找到一个Healthy 连接，然后通过这个连接构造一个具体的 ExchangeCodec，再通过 Exchange 的构造器创建一个新的 Exchange，最后返回给拦截器 **ConnectInterceptor**，再传递给下一个拦截器

```java
// Transmitter.newExchange
Exchange newExchange(Interceptor.Chain chain, boolean doExtensiveHealthChecks) {
	...
  ExchangeCodec codec = exchangeFinder.find(client, chain, doExtensiveHealthChecks);
  Exchange result = new Exchange(this, call, eventListener, exchangeFinder, codec);

  synchronized (connectionPool) {
    this.exchange = result;
    this.exchangeRequestDone = false;
    this.exchangeResponseDone = false;
    return result;
  }
}
```



# 分析 ExchangeFinder.findConnection

涉及到查询可复用的连接，创建新连接，新连接连接 socket 等内容。

初次查找连接的逻辑，直接从代码注释里的“初次查找从这开始”看就可以，前面逻辑都是复用相关的

```java
// 入参分别是，连接超时，读超时，写超时，http2 ping 间隔时间，连接失败重试标志
private RealConnection findConnection(int connectTimeout, int readTimeout, int writeTimeout,
                                      int pingIntervalMillis, boolean connectionRetryEnabled) throws IOException {
  boolean foundPooledConnection = false;
  RealConnection result = null;
  Route selectedRoute = null;
  RealConnection releasedConnection;
  Socket toClose;
  synchronized (connectionPool) {
    if (transmitter.isCanceled()) throw new IOException("Canceled");
    hasStreamFailure = false; // This is a fresh attempt.
    
    // 这里会使用已经分配给 transmitter 的连接，已经分配的连接在创建新的 Exchange 上是被限制的
    // 如果是第一次进这来，那么 transmitter.connection 的值是 null
    // 那么 toClose 也是 null
    
    // Attempt to use an already-allocated connection. We need to be careful here because our
    // already-allocated connection may have been restricted from creating new exchanges.
    releasedConnection = transmitter.connection;
    toClose = transmitter.connection != null && transmitter.connection.noNewExchanges
      ? transmitter.releaseConnectionNoEvents()
      : null;

    if (transmitter.connection != null) {
      // We had an already-allocated connection and it's good.
      result = transmitter.connection;
      releasedConnection = null;
    }

		// 没有已分配的连接可以重用，从连接池里面获取
    // transmitterAcquirePooledConnection 传入的参数：
    // List<Route> routes 为 null，requireMultiplexed 为 false
    // 方法内容见: RealConnectionPool.transmitterAcquirePooledConnection
    // 方法最后返回如果是 true，transmitter.connection 就会有值
    if (result == null) {
      // Attempt to get a connection from the pool.
      // 最开始的时候，由于 connectionPool.connections 是空的，所以会返回 false
      if (connectionPool.transmitterAcquirePooledConnection(address, transmitter, null, false)) { 
        foundPooledConnection = true;
        result = transmitter.connection;
      } else if (nextRouteToTry != null) { // 最开始 nextRouteToTry 也是 null
        selectedRoute = nextRouteToTry;
        nextRouteToTry = null;
      } else if (retryCurrentRoute()) { // 这个条件需要 transmitter.connection 不为 null
        selectedRoute = transmitter.connection.route();
      }
    }
  }
  closeQuietly(toClose);

  if (releasedConnection != null) {
    eventListener.connectionReleased(call, releasedConnection);
  }
  if (foundPooledConnection) {
    eventListener.connectionAcquired(call, result);
  }
  if (result != null) {
    // If we found an already-allocated or pooled connection, we're done.
    return result;
  }

  // 初次查找从这开始
  // 前面逻辑是关于连接复用的，但是初次查找是不存在任何连接的，只能创建
  // 首先，使用了路由选择器来选择路由，路由选择器 routeSelector 是在 ExchangeFinder 构造器里创建的
  // If we need a route selection, make one. This is a blocking operation.
  boolean newRouteSelection = false;
  if (selectedRoute == null && (routeSelection == null || !routeSelection.hasNext())) {
    newRouteSelection = true;
    // routeSelection 类型是 RouteSelector.Selection：
    // 包含了 List<Route> routes 和 nextRouteIndex 下标索引，见 RouteSelector.Select
    routeSelection = routeSelector.next();
  }

  List<Route> routes = null;
  synchronized (connectionPool) {
    if (transmitter.isCanceled()) throw new IOException("Canceled");

    if (newRouteSelection) {
      // Now that we have a set of IP addresses, make another attempt at getting a connection from
      // the pool. This could match due to connection coalescing.
      routes = routeSelection.getAll();
      // 同样的，初次的情况 connectionPool.connections 是空的，所以不会成功
      if (connectionPool.transmitterAcquirePooledConnection(
        address, transmitter, routes, false)) {
        foundPooledConnection = true;
        result = transmitter.connection;
      }
    }

    if (!foundPooledConnection) {
      if (selectedRoute == null) {
        // 初次的情况，selectedRoute 肯定为 null
        // 取 routeSelection 下一个 Route 用于创建一个 RealConnection
        selectedRoute = routeSelection.next();
      }

      // Create a connection and assign it to this allocation immediately. This makes it possible
      // for an asynchronous cancel() to interrupt the handshake we're about to do.
      result = new RealConnection(connectionPool, selectedRoute);
      connectingConnection = result; //赋值给 “连接中的连接” 变量
    }
  }

  // 初次不会进这
  // If we found a pooled connection on the 2nd time around, we're done.
  if (foundPooledConnection) {
    eventListener.connectionAcquired(call, result);
    return result;
  }

  // 初次，创建好了 RealConnection，接着要做 TCP 连接和 TLS 握手了
  // Do TCP + TLS handshakes. This is a blocking operation.
  result.connect(connectTimeout, readTimeout, writeTimeout, pingIntervalMillis,
                 connectionRetryEnabled, call, eventListener);
  connectionPool.routeDatabase.connected(result.route()); // 黑名单中移除这个 route

  Socket socket = null;
  synchronized (connectionPool) {
    connectingConnection = null;
    // Last attempt at connection coalescing, which only occurs if we attempted multiple
    // concurrent connections to the same host.
    if (connectionPool.transmitterAcquirePooledConnection(address, transmitter, routes, true)) {
      // We lost the race! Close the connection we created and return the pooled connection.
      result.noNewExchanges = true;
      socket = result.socket();
      result = transmitter.connection;

      // It's possible for us to obtain a coalesced connection that is immediately unhealthy. In
      // that case we will retry the route we just successfully connected with.
      nextRouteToTry = selectedRoute;
    } else {
      // 初次的情况，把新建的连接放入连接池
      connectionPool.put(result);
      transmitter.acquireConnectionNoEvents(result);
    }
  }
  closeQuietly(socket);

  eventListener.connectionAcquired(call, result);
  return result;
}
```



## 第一次对某个连接发起请求

初次查找的流程：

1. 先通过路由选择器获取一个 RouteSelector.Selection 对象：**RouteSelector.next**
2. 通过 RouteSelector.Selection 对象的 next 方法获取一个 Route：**Selection.next**
3. 用步骤 2 的 Route 构造新 RealConnection 对象，然后进行连接操作：**RealConnection.connect**
4. 连接完毕，RealConnection 加入连接池：**RealConnectionPool.put**
5. 最后还要把 RealConnection 交给发射器 Transmitter：**Transmitter.acquireConnectionNoEvents**

### 1) RouteSelector.next: Selection

这个方法的作用通过代理的类型（直连，HTTP，Socks），构造一个 RouteSelector.Selection，包含一组用于后期连接使用的 Routes。（可以先了解一下 RouterSelector）

#### RouteSelector.Selection

RouteSelector 内部类，用来保存路由列表，以及封装了对路由列表的相关操作

```java
public static final class Selection {
  private final List<Route> routes;
  private int nextRouteIndex = 0;
  ...
}
```

分析 next 方法：

```java
public Selection next() throws IOException {
  if (!hasNext()) {
    throw new NoSuchElementException();
  }

  // Compute the next set of routes to attempt.
  List<Route> routes = new ArrayList<>();
  
  // 判断条件：nextProxyIndex < proxies.size()
  while (hasNextProxy()) {
    // Postponed routes are always tried last. For example, if we have 2 proxies and all the
    // routes for proxy1 should be postponed, we'll move to proxy2. Only after we've exhausted
    // all the good routes will we attempt the postponed routes.
    Proxy proxy = nextProxy(); // 拿到下一个代理对象
    // nextProxy() 逻辑如下: 
    //     Proxy result = proxies.get(nextProxyIndex++);
    //     resetNextInetSocketAddress(result);
    // 		 return result;
    
    // inetSocketAddresses 是通过域名解析得到的 ip + port 或 socks ip + port 构建的网络地址列表
    // 构造 rotues 列表（Route 和 RouteDatabase 详情查看下面的代码）
    // 简单说，RouteDatabase 是 route 的黑名单，记录着连接失败过的 route（底层就一个 Set）
    // Route 就是一个 Address + Proxy + 网络地址 inetSocketAddress 的一个包装类
    for (int i = 0, size = inetSocketAddresses.size(); i < size; i++) {
      Route route = new Route(address, proxy, inetSocketAddresses.get(i));
      // 判断这个 route 失败过没有，失败过的放入延迟列表
      if (routeDatabase.shouldPostpone(route)) {
        postponedRoutes.add(route);
      } else {
        routes.add(route);
      }
    } // for
		// 上面这个 for 循环执行完，如果 routes 已经不为空了，就直接 break
    // 这意味着就算设有多个代理，当通过第一个代理就构造出了可用的 Route 其他代理就会被先忽视
    if (!routes.isEmpty()) {
      break; // break while(hasNextProxy())
    }
  }

  // 如果到了这里，routes 还是空的， 说明所有代理所有的 Route 都是连接失败的（都在黑名单）
  // 那没办法了，只能把失败的那些 Route 再放回去再去试一遍
  if (routes.isEmpty()) {
    // We've exhausted all Proxies so fallback to the postponed routes.
    routes.addAll(postponedRoutes);
    postponedRoutes.clear();
  }

  return new Selection(routes);
}
```



#### RouteSelector.nextProxy

```java
/** Returns the next proxy to try. May be PROXY.NO_PROXY but never null. */
private Proxy nextProxy() throws IOException {
  if (!hasNextProxy()) {
    throw new SocketException("No route to " + address.url().host()
                              + "; exhausted proxy configurations: " + proxies);
  }
  Proxy result = proxies.get(nextProxyIndex++);
  resetNextInetSocketAddress(result);
  return result;
}
```

##### RouteSelector.resetNextInetSocketAddress

这个方法主要是在获取下一个 Proxy 之后，为了构造 Selection.routes  以及 Selection 做准备

```java
/** Prepares the socket addresses to attempt for the current proxy or host. */
private void resetNextInetSocketAddress(Proxy proxy) throws IOException {
  // Clear the addresses. Necessary if getAllByName() below throws!
  inetSocketAddresses = new ArrayList<>();

  String socketHost;
  int socketPort;
  // 代理方式是 DIRECT（无代理）或者 SOCKS
  if (proxy.type() == Proxy.Type.DIRECT || proxy.type() == Proxy.Type.SOCKS) {
    socketHost = address.url().host();
    socketPort = address.url().port();
  } else { // HTTP 代理模式
    SocketAddress proxyAddress = proxy.address();
    if (!(proxyAddress instanceof InetSocketAddress)) {
      throw new IllegalArgumentException(
        "Proxy.address() is not an " + "InetSocketAddress: " + proxyAddress.getClass());
    }
    InetSocketAddress proxySocketAddress = (InetSocketAddress) proxyAddress;
    socketHost = getHostString(proxySocketAddress);
    socketPort = proxySocketAddress.getPort();
  }

  if (socketPort < 1 || socketPort > 65535) {
    throw new SocketException("No route to " + socketHost + ":" + socketPort
                              + "; port is out of range");
  }

  if (proxy.type() == Proxy.Type.SOCKS) { 
    // 检查 socketHost 和 socketPort 合法性后重新创建一个 InetSocketAddress（ip+port）
    inetSocketAddresses.add(InetSocketAddress.createUnresolved(socketHost, socketPort));
  } else { 
    eventListener.dnsStart(call, socketHost);
		
    // 直连 和 HTTP 的代理，需要 dns 解析，把域名解析成 ip，一个域名对应多个 ip 因此是个数组
    // Try each address for best behavior in mixed IPv4/IPv6 environments.
    List<InetAddress> addresses = address.dns().lookup(socketHost);
    if (addresses.isEmpty()) {
      throw new UnknownHostException(address.dns() + " returned no addresses for " + socketHost);
    }

    eventListener.dnsEnd(call, socketHost, addresses);

    // 把 dns 查询出来的 InetAddress 加上端口包装成 InetSocketAddress，放入列表
    for (int i = 0, size = addresses.size(); i < size; i++) {
      InetAddress inetAddress = addresses.get(i);
      inetSocketAddresses.add(new InetSocketAddress(inetAddress, socketPort));
    }
  }
}
```

> SocketAddress 代表协议无关 Socket 地址，抽象类，因此只有基于某个协议的具体子类才有意义
>
> InetAddress 只是 IP 地址的封装
>
> InetSocketAddress 是 SocketAddress 实现类，实现 IP+端口的套接字地址

###### DNS.lookup

Address.dns 也是来自 okHttpClient.dns，默认值 Dns.SYSTEM

```java
Dns SYSTEM = hostname -> {
  if (hostname == null) throw new UnknownHostException("hostname == null");
  try {
    // 直接使用系统 api 获取这个域名所有的 ip 地址
    return Arrays.asList(InetAddress.getAllByName(hostname));
  } catch (NullPointerException e) {
    UnknownHostException unknownHostException =
      new UnknownHostException("Broken system behaviour for dns lookup of " + hostname);
    unknownHostException.initCause(e);
    throw unknownHostException;
  }
};
```



#### RouteSelector

创建于 ExchangeFinder 的构造器：

```java
ExchangeFinder(Transmitter transmitter, RealConnectionPool connectionPool,
               Address address, Call call, EventListener eventListener) {
  ...
	this.routeSelector = new RouteSelector(
    	address, connectionPool.routeDatabase, call, eventListener);  
}
```

```java
RouteSelector(Address address, RouteDatabase routeDatabase, Call call,
              EventListener eventListener) {
  this.address = address;
  this.routeDatabase = routeDatabase;
  this.call = call;
  this.eventListener = eventListener;

  resetNextProxy(address.url(), address.proxy());
}
// 其他成员变量

/* State for negotiating the next proxy to use. */
private List<Proxy> proxies = Collections.emptyList(); // 这个路由选择器所有的代理
private int nextProxyIndex; // 即将访问的代理的下标

/* State for negotiating the next socket address to use. */ 
private List<InetSocketAddress> inetSocketAddresses = Collections.emptyList();

/* State for negotiating failed routes */
private final List<Route> postponedRoutes = new ArrayList<>();
```

##### RouteSelector.resetNextProxy

重置下一个代理，这步其实只会发生在 RouteSelector 的初始化里，因此它也是初始化的一部分。要理解这个方法首先得先去下方了解一下 Address 和 Proxy。

如果 proxy 没有配置（即 okHttpClient 没有配置 proxy 字段），那么就会使用 ProxySelector 来为 RouteSelector 创建 proxies。

Address 的 proxySelector 也是来自 okHttpClient 的配置，这个值默认不为 null（参考 okHttpClient.Builder 代码），`proxySelector = ProxySelector.getDefault()`，默认是用反射生成 `sun.net.spi.DefaultProxySelector`

如果都没配置都按照默认的，得到的 proxiesOrNull 就会被赋值为一个 List，里面就一个元素：type 为 DIRECT 的 Proxy。因此 proxies 默认的话是一个类型为直连的 Proxy。

如果设置一个 Proxy，那么 proxies 也是单元素数组。

ProxySelector 返回 List 有多个 Proxy 的情况先不分析。（需要看 DefaultProxySelector 源码）

```java
/** Prepares the proxy servers to try. */
private void resetNextProxy(HttpUrl url, Proxy proxy) {
  if (proxy != null) { 
    // If the user specifies a proxy, try that and only that.
    proxies = Collections.singletonList(proxy);
  } else {
    // Try each of the ProxySelector choices until one connection succeeds.
    List<Proxy> proxiesOrNull = address.proxySelector().select(url.uri());
    // 什么都不设置 proxiesOrNull 不为 null，持有一个 type 为 DIRECT 的 Proxy
    proxies = proxiesOrNull != null && !proxiesOrNull.isEmpty() // true
        ? Util.immutableList(proxiesOrNull)
        : Util.immutableList(Proxy.NO_PROXY);
  }
  nextProxyIndex = 0;
}
```



#### Address

final 类，主要作为一个包装类。除了 host 和 port 字段来自 url，其他字段来都自 okHttpClient

```java
public final class Address {
  final HttpUrl url;
  final Dns dns;
  final SocketFactory socketFactory;
  final Authenticator proxyAuthenticator;
  final List<Protocol> protocols;
  final List<ConnectionSpec> connectionSpecs;
  final ProxySelector proxySelector;
  final @Nullable Proxy proxy;
  final @Nullable SSLSocketFactory sslSocketFactory;
  final @Nullable HostnameVerifier hostnameVerifier;
  final @Nullable CertificatePinner certificatePinner;
  ...
}
```

Address 的创建发生在 ExchangeFinder 的创建，即 `Transmitter.prepareToConnect`，再调用了 `Transmitter.createAddress(HttpUrl url): Address`，这个方法的逻辑就是根据 url 和 okHttpClient 的配置来创建一个 Address

```java
// Transmitter.java
private Address createAddress(HttpUrl url) {
  SSLSocketFactory sslSocketFactory = null;
  HostnameVerifier hostnameVerifier = null;
  CertificatePinner certificatePinner = null;
  if (url.isHttps()) {
    sslSocketFactory = client.sslSocketFactory();
    hostnameVerifier = client.hostnameVerifier();
    certificatePinner = client.certificatePinner();
  }

  return new Address(url.host(), url.port(), 
                     client.dns(), client.socketFactory(), 
                     sslSocketFactory, hostnameVerifier, certificatePinner, 
                     client.proxyAuthenticator(), client.proxy(), 
                     client.protocols(), client.connectionSpecs(), 
                     client.proxySelector());
}
```



#### Proxy

Proxy 的最初来源是 okHttpClient 的配置，如果不配置 proxy 成员变量（字段），那么 okHttpClient.proxy 默认是 null，那么在 Adress 里面的 proxy 也是 null。

在 RouteSelector 的构造器初始化调用 RouteSelector.resetNextProxy 的时候如果 Addrees.proxy 是 null，就会调用 Addrees.proxySelector 来生成。如果没配置过，那么默认返回的是 `Proxy.NO_PROXY`

而 NO_PROXY 就是私有构造器初始化的。

```java
public static final Proxy NO_PROXY = new Proxy();

private Proxy() {
  type = Type.DIRECT;
  sa = null;
}

// Proxy 就两成员变量：
private Proxy.Type type; // DIRECT, HTTP, SOCKS
private SocketAddress sa;    
```



### 2) RouteSelector.Selection.next：Route

发生在初次查找连接的第二步。就是简单的获取下一个 Route

```java
public Route next() {
  if (!hasNext()) {
    throw new NoSuchElementException();
  }
  return routes.get(nextRouteIndex++);
}
```

#### Route

这个类其实就是 Address，Proxy，InetSocketAddress 的一个封装类

```java
public final class Route {
	final Address address;
  final Proxy proxy;
  final InetSocketAddress inetSocketAddress;
	...
}
```



### RouteDatabase

核心就是失败过的 Route 的集合，注释里面也说的挺清楚了，这是一个Route 的黑名单，装着那些失败过的 route

```java
/**
 * A blacklist of failed routes to avoid when creating a new connection to a target address. This is
 * used so that OkHttp can learn from its mistakes: if there was a failure attempting to connect to
 * a specific IP address or proxy server, that failure is remembered and alternate routes are
 * preferred.
 */
final class RouteDatabase {
  private final Set<Route> failedRoutes = new LinkedHashSet<>();

  // 这个 route 连接失败了，加入黑名单
  /** Records a failure connecting to {@code failedRoute}. */
  public synchronized void failed(Route failedRoute) {
    failedRoutes.add(failedRoute);
  }

  // 这个 route 连上了，移除黑名单，
  /** Records success connecting to {@code route}. */
  public synchronized void connected(Route route) {
    failedRoutes.remove(route);
  }

  // 如果包含在这个失败集合中，说明这个 route 已经失败过了
  /** Returns true if {@code route} has failed recently and should be avoided. */
  public synchronized boolean shouldPostpone(Route route) {
    return failedRoutes.contains(route);
  }
}
```



### 3) RealConnection.connect

```java
public void connect(int connectTimeout, int readTimeout, int writeTimeout,
                    int pingIntervalMillis, boolean connectionRetryEnabled, Call call,
                    EventListener eventListener) {
  if (protocol != null) throw new IllegalStateException("already connected");

  RouteException routeException = null;
  // 获取 Address 的连接规格，这也是从 okHttpClient 里面获取的
  // 默认是 Util.immutableList(ConnectionSpec.MODERN_TLS, ConnectionSpec.CLEARTEXT)
  // 创建新的连接规格选择器，主要用于后面 TLS 会话，详见 <ConnectionSpec 连接规格>
  List<ConnectionSpec> connectionSpecs = route.address().connectionSpecs();
  ConnectionSpecSelector connectionSpecSelector = new  					ConnectionSpecSelector(connectionSpecs);

  // okHttpClient 如果不配置，默认是使用
  // X509TrustManager trustManager = Util.platformTrustManager();
  // this.sslSocketFactory = newSslSocketFactory(trustManager);
  // 这会创建一个信任所有（CA）根证书的 SSLSocketFactory
  // 但是 Address 里面的 sslSocketFactory 也只会在 url 是 https 的时候才会有值 
  // 详见 <X509TrustManager & SslSocketFactory>
  if (route.address().sslSocketFactory() == null) {
    if (!connectionSpecs.contains(ConnectionSpec.CLEARTEXT)) {
      throw new RouteException(new UnknownServiceException(
        "CLEARTEXT communication not enabled for client"));
    }
    String host = route.address().url().host();
    if (!Platform.get().isCleartextTrafficPermitted(host)) {
      throw new RouteException(new UnknownServiceException(
        "CLEARTEXT communication to " + host + " not permitted by network security policy"));
    }
  } else { 
    // 如果 url 是 https，即 sslSocketFactory 有值，说明要求使用加密协议
    // H2_PRIOR_KNOWLEDGE 为明文 http2 协议，因此抛出异常
    // address.protocols 默认是 Util.immutableList(Protocol.HTTP_2, Protocol.HTTP_1_1)
    if (route.address().protocols().contains(Protocol.H2_PRIOR_KNOWLEDGE)) {
      throw new RouteException(new UnknownServiceException(
        "H2_PRIOR_KNOWLEDGE cannot be used with HTTPS"));
    }
  }

  while (true) {
    try {
      // 隧道，代理类型为 HTTP，并且配置了 sslSocketFactory
      if (route.requiresTunnel()) {
        connectTunnel(connectTimeout, readTimeout, writeTimeout, call, eventListener);
        if (rawSocket == null) {
          // We were unable to connect the tunnel but properly closed down our resources.
          break;
        }
      } else { // 代理类型是直连或 socks
        connectSocket(connectTimeout, readTimeout, call, eventListener);
      }
      
      // 首先根据 sslSocketFactory 是否为 null，来判断是否是明文连接
      // url 是 https 的时候 sslSocketFactory 才会被赋值 Address
      // 其次，根据 address.protocols 是否包含 Protocol.H2_PRIOR_KNOWLEDGE 
      // 来判断是否使用明文 http2
      // address.protocols 不包含 Protocol.H2_PRIOR_KNOWLEDGE 的，最后都用明文 http1.1
      // 明文的情况，socket 被赋值为 connectSocket 创建的 rawSocket
      // 否则就会调用 connectTls 进行 Tls 会话，创建 SSLSocket，并最终赋值到 socket
      establishProtocol(connectionSpecSelector, pingIntervalMillis, call, eventListener);
      eventListener.connectEnd(call, route.socketAddress(), route.proxy(), protocol);
      break;
    } catch (IOException e) {
      closeQuietly(socket);
      closeQuietly(rawSocket);
      socket = null;
      rawSocket = null;
      source = null;
      sink = null;
      handshake = null;
      protocol = null;
      http2Connection = null;

      eventListener.connectFailed(call, route.socketAddress(), route.proxy(), null, e);

      if (routeException == null) {
        routeException = new RouteException(e);
      } else {
        routeException.addConnectException(e);
      }

      // connectionRetryEnabled 默认 true，对应 okHttpClient.retryOnConnectionFailure
      // 
      if (!connectionRetryEnabled || !connectionSpecSelector.connectionFailed(e)) {
        throw routeException;
      }
    }
  }

  if (route.requiresTunnel() && rawSocket == null) {
    ProtocolException exception = new ProtocolException("Too many tunnel connections attempted: "
                                                        + MAX_TUNNEL_ATTEMPTS);
    throw new RouteException(exception);
  }

  if (http2Connection != null) {
    synchronized (connectionPool) {
      allocationLimit = http2Connection.maxConcurrentStreams();
    }
  }
}
```

#### RealConnection

```java
public RealConnection(RealConnectionPool connectionPool, Route route) {
  this.connectionPool = connectionPool;
  this.route = route;
}

// 所有成员变量：
public final RealConnectionPool connectionPool; // 连接池
private final Route route; // 使用的路由（包含 Address，Proxy，网络地址 InetSocketAddress）

// 这个字段在 connect 方法初始化并且不会再重新分配
private Socket rawSocket; // 底层 tcp socket

// 应用层 socket，https 的情况就是基于 rawSocket 的 SSLSocket
// 如果是明文的 http，socket 会直接赋值 rawSocket
private Socket socket; 
private Handshake handshake; // 握手记录
private Protocol protocol; // 最终协议
private Http2Connection http2Connection; // 只有在协议使用 http2 的时候才存在
private BufferedSource source; // 输入流
private BufferedSink sink; // 输出流

/**
   * If true, no new exchanges can be created on this connection. Once true this is always true.
   * Guarded by {@link #connectionPool}.
   */
boolean noNewExchanges; 
// false 的时候可以用于创建 Exchange，标记成 true 了，这个连接就不能再用于创建 Exchange 了，而且值也不能再变成 false

// 
int routeFailureCount;

int successCount; // 成功次数
private int refusedStreamCount; 

// 当前这个连接能携带流的最大数
private int allocationLimit = 1;

// Transmitter 和 Call 是一对一的，这代表当前连接承载了多少个 Call（请求）
final List<Reference<Transmitter>> transmitters = new ArrayList<>();

/** Nanotime timestamp when {@code allocations.size()} reached zero. */
long idleAtNanos = Long.MAX_VALUE; 
// 其实是 transmitters.size() 达到 0 时候的时间戳
// transmitters.size()，说明当前连接是闲置的
```

#### RealConnection.connectTunnel [详见下文]

进行隧道连接的条件：Route.address.sslSocketFactory 非 null 并且代理类型为 HTTP

#### RealConnection.connectSocket

```java
/** Does all the work necessary to build a full HTTP or HTTPS connection on a raw socket. */
private void connectSocket(int connectTimeout, int readTimeout, Call call,
                           EventListener eventListener) throws IOException {
  Proxy proxy = route.proxy();
  Address address = route.address();

  // address.socketFactory 即 okHttpClient.socketFactory，默认 SocketFactory.getDefault()
  // 即 DefaultSocketFactory，DefaultSocketFactory.createSocket 就是 new Socket()
  rawSocket = proxy.type() == Proxy.Type.DIRECT || proxy.type() == Proxy.Type.HTTP
    ? address.socketFactory().createSocket()
    : new Socket(proxy);

  eventListener.connectStart(call, route.socketAddress(), proxy);
  rawSocket.setSoTimeout(readTimeout); // 设置 socket 读超时
  try {
    // 核心逻辑：socket.connect(address, connectTimeout) 详情见 <Platform 平台兼容>
    Platform.get().connectSocket(rawSocket, route.socketAddress(), connectTimeout);
  } catch (ConnectException e) {
    ConnectException ce = new ConnectException("Failed to connect to " + route.socketAddress());
    ce.initCause(e);
    throw ce;
  }

  // The following try/catch block is a pseudo hacky way to get around a crash on Android 7.0
  // More details:
  // https://github.com/square/okhttp/issues/3245
  // https://android-review.googlesource.com/#/c/271775/
  try {
    source = Okio.buffer(Okio.source(rawSocket)); // 基于这个 socket 创建输入流
    sink = Okio.buffer(Okio.sink(rawSocket)); // 基于这个 socket 创建输出流
  } catch (NullPointerException npe) {
    if (NPE_THROW_WITH_NULL.equals(npe.getMessage())) {
      throw new IOException(npe);
    }
  }
}
```



#### RealConnection.establishProtocol

```java
private void establishProtocol(ConnectionSpecSelector connectionSpecSelector,
                               int pingIntervalMillis, Call call, EventListener eventListener) throws IOException {
  if (route.address().sslSocketFactory() == null) {
    if (route.address().protocols().contains(Protocol.H2_PRIOR_KNOWLEDGE)) {
      // 这是明文的 http2 的情况，socket 直接赋值底层 rawSocket
      // 必须要求 okhttpClient.protocols 里面包含 Protocol.H2_PRIOR_KNOWLEDGE
      socket = rawSocket;
      protocol = Protocol.H2_PRIOR_KNOWLEDGE;
      startHttp2(pingIntervalMillis); // 创建 http2Connection 成员变量，并启动它
      return;
    }

    // 其他明文情况，都使用 http 1.1，
    // 即使 address.protocols 默认是 Util.immutableList(Protocol.HTTP_2, Protocol.HTTP_1_1)
    socket = rawSocket;
    protocol = Protocol.HTTP_1_1;
    return;
  }

  // 上面没有 return，运行到这就是执行加密的 http1.1 或者 http2 了
  eventListener.secureConnectStart(call);
  connectTls(connectionSpecSelector);
  eventListener.secureConnectEnd(call, handshake);

  // tls1.2 握手过程中会 ALPN 协商是使用 http1.1 或者 h2 	
  if (protocol == Protocol.HTTP_2) { 
    startHttp2(pingIntervalMillis);
  }
}
```

##### RealConnection.startHttp2 [详见下文]

```java
private void startHttp2(int pingIntervalMillis) throws IOException {
  // 读无超时时间
  socket.setSoTimeout(0); // HTTP/2 connection timeouts are set per-stream.
  http2Connection = new Http2Connection.Builder(true)
    .socket(socket, route.address().url().host(), source, sink)
    .listener(this)
    .pingIntervalMillis(pingIntervalMillis)
    .build();
  http2Connection.start();
}
```

http2 内容后面分析



##### RealConnection.connectTls

TLS 握手

```java
private void connectTls(ConnectionSpecSelector connectionSpecSelector) throws IOException {
  Address address = route.address();
  SSLSocketFactory sslSocketFactory = address.sslSocketFactory();
  boolean success = false;
  SSLSocket sslSocket = null;
  try {
    // 只有使用 SSLSocketFactory 创建的 SSLSocket 才能进行 TLS 的握手操作
    // Create the wrapper over the connected socket.
    sslSocket = (SSLSocket) sslSocketFactory.createSocket(
        rawSocket, address.url().host(), address.url().port(), true /* autoClose */);

    // 目的就是使用 sslSocket.setEnabledProtocols 和 sslSocket.setEnabledCipherSuites 
    // 把可用的连接规格的 tls 版本和加密套件设置到 SSLSocket 里面
    // 设置进去的 tls 版本和加密套件必须是连接规格里面有的，并且是 sslSocket 启用的
    // 具体的逻辑参考 <ConnectionSpec 连接规格> 
    // Configure the socket's ciphers, TLS versions, and extensions.
    ConnectionSpec connectionSpec = connectionSpecSelector.configureSecureSocket(sslSocket);
    if (connectionSpec.supportsTlsExtensions()) { // 支持 tls 扩展
      // address.protocols 默认是 Util.immutableList(Protocol.HTTP_2, Protocol.HTTP_1_1)
      // 来源依然是 okHttpClient 的默认配置
      // 根据平台（Android5+，10+）进行 tls 配置优化
      // 内部逻辑是配置 SessionTickets 和 的 ALPN（TLS 扩展），详见 <Platform 平台兼容>
      Platform.get().configureTlsExtensions(
          sslSocket, address.url().host(), address.protocols());
    }

    // tls 握手，阻塞直到握手完成
    // Force handshake. This can throw!
    sslSocket.startHandshake();
    // block for session establishment
    SSLSession sslSocketSession = sslSocket.getSession();
    // 通过握手会话返回的会话对象，构建握手记录对象
    Handshake unverifiedHandshake = Handshake.get(sslSocketSession);

    // okhttpClient 默认的域名校验器是 OkHostnameVerifier.INSTANCE，是个饿汉加载的单例
    // 内部逻辑是使用 OkHostnameVerifier 校验远端证书的合法性
    // Verify that the socket's certificates are acceptable for the target host.
    if (!address.hostnameVerifier().verify(address.url().host(), sslSocketSession)) {
      List<Certificate> peerCertificates = unverifiedHandshake.peerCertificates();
      if (!peerCertificates.isEmpty()) {
        X509Certificate cert = (X509Certificate) peerCertificates.get(0);
        throw new SSLPeerUnverifiedException(
            "Hostname " + address.url().host() + " not verified:"
                + "\n    certificate: " + CertificatePinner.pin(cert)
                + "\n    DN: " + cert.getSubjectDN().getName()
                + "\n    subjectAltNames: " + OkHostnameVerifier.allSubjectAltNames(cert));
      } else {
        throw new SSLPeerUnverifiedException(
            "Hostname " + address.url().host() + " not verified (no certificates)");
      }
    }

    // okhttpClient 默认 CertificatePinner 是 CertificatePinner.DEFAULT
    // CertificatePinner 是负责证书锁定的安全类，这的逻辑就是做证书锁定检查
    // Check that the certificate pinner is satisfied by the certificates presented.
    address.certificatePinner().check(address.url().host(),
        unverifiedHandshake.peerCertificates());

    // Success! Save the handshake and the ALPN protocol.
    // 支持 tls 扩展的情况下，获得 tls 协商后确定下来协议（http2，http1.1）
    String maybeProtocol = connectionSpec.supportsTlsExtensions()
        ? Platform.get().getSelectedProtocol(sslSocket)
        : null;
    socket = sslSocket; // 使用安全套接字作为通信套接字，区别于明文传输
    source = Okio.buffer(Okio.source(socket)); //输入流
    sink = Okio.buffer(Okio.sink(socket)); // 输出流
    handshake = unverifiedHandshake; // 握手记录
    protocol = maybeProtocol != null // 获取协议枚举值
        ? Protocol.get(maybeProtocol)
        : Protocol.HTTP_1_1;
    success = true;
  } catch (AssertionError e) {
    if (Util.isAndroidGetsocknameError(e)) throw new IOException(e);
    throw e;
  } finally {
    if (sslSocket != null) {
      Platform.get().afterHandshake(sslSocket); // 空实现
    }
    if (!success) { // tls 握手没成功，关闭 socket
      closeQuietly(sslSocket);
    }
  }
}
```

> Session ID：https 会话恢复的方法之一。TLS 握手的时候，ClientHello 带 Session Id，Server端会保存一个 Session Cache，通过 Session Id 可以查询到会话信息进行恢复（Server 发送 ChangeCipherSpec和Finish，Client 收到后会回复 ChangeCipherSpec 和 Finish）。没有会话信息，重新握手。
>
> ​	缺点：Server 存储会话信息，限制了 Server 的扩展能力；分布式系统中，同步问题
>
> Session Ticket：



###### Handshake

```java
public final class Handshake {
  private final TlsVersion tlsVersion; // tls 会话握手版本
  private final CipherSuite cipherSuite; // tls 会话最终选择的加密套装
  private final List<Certificate> peerCertificates; // 远端（serv）的证书列表
  private final List<Certificate> localCertificates; // 本地（android）的证书列表
  ...
}
```

###### Handshake.get

tls 会话期间的参数，构成握手记录

```java
public static Handshake get(SSLSession session) throws IOException {
  String cipherSuiteString = session.getCipherSuite();
  if (cipherSuiteString == null) throw new IllegalStateException("cipherSuite == null");
  if ("SSL_NULL_WITH_NULL_NULL".equals(cipherSuiteString)) {
    throw new IOException("cipherSuite == SSL_NULL_WITH_NULL_NULL");
  }
  
  // 从 CipherSuite 的静态 map 里面取出或者新建并返回
  CipherSuite cipherSuite = CipherSuite.forJavaName(cipherSuiteString);

  String tlsVersionString = session.getProtocol();
  if (tlsVersionString == null) throw new IllegalStateException("tlsVersion == null");
  if ("NONE".equals(tlsVersionString)) throw new IOException("tlsVersion == NONE");
  
  // 返回版本字符串对应的枚举值
  TlsVersion tlsVersion = TlsVersion.forJavaName(tlsVersionString);

  Certificate[] peerCertificates;
  try {
    // 返回远端在握手中用于表明自己身份的证书列表
    peerCertificates = session.getPeerCertificates();
  } catch (SSLPeerUnverifiedException ignored) {
    peerCertificates = null;
  }
  List<Certificate> peerCertificatesList = peerCertificates != null
      ? Util.immutableList(peerCertificates)
      : Collections.emptyList();

  // 返回在握手过程中表明本地身份的证书列表
  Certificate[] localCertificates = session.getLocalCertificates();
  List<Certificate> localCertificatesList = localCertificates != null
      ? Util.immutableList(localCertificates)
      : Collections.emptyList();

  return new Handshake(tlsVersion, cipherSuite, peerCertificatesList, localCertificatesList);
}
```



###### CipherSuite

```java
public final class CipherSuite {
  final String javaName;

  private static final Map<String, CipherSuite> INSTANCES = new LinkedHashMap<>();  
  
  private CipherSuite(String javaName) {
    if (javaName == null) {
      throw new NullPointerException();
    }
    this.javaName = javaName;
  }
  
    /**
   * @param javaName the name used by Java APIs for this cipher suite. Different than the IANA name
   *     for older cipher suites because the prefix is {@code SSL_} instead of {@code TLS_}.
   */
  public static synchronized CipherSuite forJavaName(String javaName) {
    CipherSuite result = INSTANCES.get(javaName);
    if (result == null) {
      // map 里面没存这个对象，就把 javaName 的名字转换下，然后再 get 一次
      // javaName TLS_ 开头就转成 SSL_，SSL_ 开头就转为 TLS_
      // 因为老的加密套件是使用 SSL_ 的
      result = INSTANCES.get(secondaryName(javaName));

      if (result == null) {
        // 通过上面的方式还找不到，那说明真的没有，创建添加
        result = new CipherSuite(javaName);
      }

      // Add the new cipher suite, or a confirmed alias.
      INSTANCES.put(javaName, result);
    }
    return result;
  }
}
```



###### TlsVersion

```java
public enum TlsVersion {
  TLS_1_3("TLSv1.3"), // 2016.
  TLS_1_2("TLSv1.2"), // 2008.
  TLS_1_1("TLSv1.1"), // 2006.
  TLS_1_0("TLSv1"),   // 1999.
  SSL_3_0("SSLv3"),   // 1996.
  ;
  ...
   
    
  public static TlsVersion forJavaName(String javaName) {
    switch (javaName) {
      case "TLSv1.3":
        return TLS_1_3;
      case "TLSv1.2":
        return TLS_1_2;
      case "TLSv1.1":
        return TLS_1_1;
      case "TLSv1":
        return TLS_1_0;
      case "SSLv3":
        return SSL_3_0;
    }
    throw new IllegalArgumentException("Unexpected TLS version: " + javaName);
  }
}
```



###### OkHostnameVerifier.verify

```java
@Override
public boolean verify(String host, SSLSession session) {
  try {
    Certificate[] certificates = session.getPeerCertificates();
    return verify(host, (X509Certificate) certificates[0]);
  } catch (SSLException e) {
    return false;
  }
}

public boolean verify(String host, X509Certificate certificate) {
  // 根据 Url 里是 ip 或者是 host 来和证书里面的域名或者证书做合法性验证（先不展开）
  // verifyAsIpAddress 只是粗略的判断一下 host 是不是 ipv4/ipv6 的 ip 格式
  return verifyAsIpAddress(host)
    ? verifyIpAddress(host, certificate)
    : verifyHostname(host, certificate);
}
```



###### verifyIpAddress & verifyHostname [待补充]

###### CertificatePinner 证书锁定 [待补充]

这个类负责 https 里面的证书锁定，抵御中间人攻击。

更多内容要去参阅 <CertificatePinner 证书锁定>

```java
public static final CertificatePinner DEFAULT = new Builder().build();

CertificatePinner(Set<Pin> pins, @Nullable CertificateChainCleaner certificateChainCleaner) {
  this.pins = pins;
  this.certificateChainCleaner = certificateChainCleaner;
}

// CertificatePinner.Builder
// private final List<Pin> pins = new ArrayList<>();
public CertificatePinner build() {
  return new CertificatePinner(new LinkedHashSet<>(pins), null);
}
```



### 4) RealConnectionPool.put

把新创建的连接放入连接池，同时开启一个清理线程

RealConnectionPool 的详细分析见 <ConnectionPoll 分析>

```java
void put(RealConnection connection) {
  assert (Thread.holdsLock(this));
  if (!cleanupRunning) {
    cleanupRunning = true;
    executor.execute(cleanupRunnable); // 启动清理闲置连接任务
  }
  connections.add(connection); // 放入连接池
}
```



### 5) Transmitter.acquireConnectionNoEvents

连接赋值给 transmitter.connection，同时还要把 transmitter 包装成弱应用加入到 RealConnection.transmitters 的 ArrayList

```java
void acquireConnectionNoEvents(RealConnection connection) {
  assert (Thread.holdsLock(connectionPool));

  if (this.connection != null) throw new IllegalStateException();
  this.connection = connection;
  connection.transmitters.add(new TransmitterReference(this, callStackTrace));
}
```



## 非初次对某个连接发起请求

1. 从连接池里面查找满足重用条件的连接：**RealConnectionPool.transmitterAcquirePooledConnection(address, transmitter, null, false)**

   1. route 列表 null
   2. requireMultiplexed 为 false

2. 如果找到了，直接放回这个连接作为结果

3. 如果找不到，通过路由选择器获取一个 RouteSelector.Selection 对象：**RouteSelector.next**

4. 获取 RouteSelector.Selection 的所有 Route，再次从连接池里面查找可重用连接：**RealConnectionPool.transmitterAcquirePooledConnection(address, transmitter, routes, false)**

   1. route 为当前 RouteSelector.Selection 的所有 Route
   2. requireMultiplexed 为 false

   加了 route 列表有什么区别呢？区别在 **RealConnection.isEligible(address, routes)**，如果这个方法要返回 true，而且是由于 route 列表导致的，那么首先符合条件的连接要是 http2，其次这个连接的 ip+port 要和 route 里面的某个 ip+port 相等。

   当然，这个 http2 的连接，由于 host 不同，还要验证证书是否覆盖这个新 host，证书锁定检查也要通过

5. 如果找到了，直接放回这个连接作为结果



方法的解析以及 HTTP2 参考下文



## 失败重试对某个连接发起请求

首先得先清楚失败重试的触发条件，即异常发生

明确重试拦截器会处理哪些异常：

- RouteException，来源：
  - RealConnection.connect：下面情况会抛出 RouteException
    - 未配置 sslSocketFactory，但连接规格 ConnectionSpec不包含明文 CLEARTEXT
    - 未配置 sslSocketFactory，但项目网络配置不支持明文
    - 配置 sslSocketFactory 了，但是协议包含 H2_PRIOR_KNOWLEDGE（与服务端指定明文 HTTP2）
    - 连接过程中（连接隧道或者连接 socket，tls会话）出现的 IOException 导致无法再次连接的情况
      - 无备用链接规格
      - ProtocolException 
      - InterruptedIOException 
      - SSLHandshakeException 且是由于 CertificateException
      - SSLPeerUnverifiedException
    - 连接完毕后，发现隧道创建失败了（超过了隧道尝试创建数）
- 其余 IOException
  - ConnectionShutdownException 需要被用于判断请求是否已经发出。抛出点在 http2 创建新流的时候或者进行设置的时候。相关方法 `Http2Connection.newStream`，`Http2Connection.setSettings`

再捕获到异常后，会先进行恢复判断

### RetryAndFollowUpInterceptor.recover

判断是和连接拦截器抛出 RouteException 的判断存在交叉

```java
private boolean recover(IOException e, Transmitter transmitter,
    boolean requestSendStarted, Request userRequest) {
  
  // 使用者（开发者）标记了不需要重试，不重试
  // The application layer has forbidden retries.
  if (!client.retryOnConnectionFailure()) return false;

  // 请求已经发出的，请求实体是一次性的，不重试
  // We can't send the request body again.
  if (requestSendStarted && requestIsOneShot(e, userRequest)) return false;

  // This exception is fatal.
  // 不可恢复的情况：
  // 协议异常 ProtocolException
  // 中断异常 InterruptedIOException 而且是 SocketTimeoutException，但是请求已经发送
  // SSL 握手异常 SSLHandshakeException，原因是由于证书异常 CertificateException
  // SSLPeerUnverifiedException
  if (!isRecoverable(e, requestSendStarted)) return false;

  // No more routes to attempt.
  // exchangeFinder.hasStreamFailure() && exchangeFinder.hasRouteToTry()
  if (!transmitter.canRetry()) return false;

  // For failure recovery, use the same route selector with a new connection.
  return true;
}
```

由于上述的失败条件的触发，以及排除了无法重试（只能抛出异常）的情况，请求会进行重试





# RealConnection.connectTunnel 连接隧道

进行隧道连接的条件是 Proxy 的类型是 HTTP，`Proxy.Type.HTTP`，并且 address.sslSocketFactory 不为 null，okhttpClient.sslSocketFactory  默认不为空，但是在 createAddress 的时候只有要求 url 的为 https 才会把 sslSocketFactor 赋值给 address.sslSocketFactory。

因此，这意味着**必须是 Proxy 类型为 HTTP，以及 url 是 https 才会进行隧道连接**

```java
private void connectTunnel(int connectTimeout, int readTimeout, int writeTimeout, Call call,
    EventListener eventListener) throws IOException {
  Request tunnelRequest = createTunnelRequest(); // 创建一个隧道请求，CONNECT
  HttpUrl url = tunnelRequest.url();
  for (int i = 0; i < MAX_TUNNEL_ATTEMPTS; i++) {
    // 连接 socket 临时创建好读写流
    connectSocket(connectTimeout, readTimeout, call, eventListener); 
    // 创建隧道：创建临时 Http1ExchangeCodec 进行请求发送，接收响应
    // 200 响应，隧道建立成功，createTunnel 返回 null，跳出 for
    // 407 代理要求认证，会再次调用 address.proxyAuthenticator.authenticate 构造一个用于认证的请求
    // createTunnel 的循环会把这个用于认证的请求再次通过 Http1ExchangeCodec 请求
    // 如果 407 的响应总包含 Connection:close，说明连接会关闭，认证请求返回到这，继续下一次 for
    tunnelRequest = createTunnel(readTimeout, writeTimeout, tunnelRequest, url);

    if (tunnelRequest == null) break; // Tunnel successfully created.

    // The proxy decided to close the connection after an auth challenge. We need to create a new
    // connection, but this time with the auth credentials.
    closeQuietly(rawSocket);
    rawSocket = null;
    sink = null;
    source = null;
    eventListener.connectEnd(call, route.socketAddress(), route.proxy(), null);
  }
}
```

## RealConnection.createTunnelRequest 

```java
/**
 * Returns a request that creates a TLS tunnel via an HTTP proxy. Everything in the tunnel request
 * is sent unencrypted to the proxy server, so tunnels include only the minimum set of headers.
 * This avoids sending potentially sensitive data like HTTP cookies to the proxy unencrypted.
 *
 * <p>In order to support preemptive authentication we pass a fake “Auth Failed” response to the
 * authenticator. This gives the authenticator the option to customize the CONNECT request. It can
 * decline to do so by returning null, in which case OkHttp will use it as-is
 */
private Request createTunnelRequest() throws IOException {
  
  // 构造代码连接请求
  Request proxyConnectRequest = new Request.Builder()
      .url(route.address().url())
      .method("CONNECT", null)
      .header("Host", Util.hostHeader(route.address().url(), true))
      .header("Proxy-Connection", "Keep-Alive") // For HTTP/1.0 proxies like Squid.
      .header("User-Agent", Version.userAgent())
      .build();

  // 构造一个假的验证响应
  Response fakeAuthChallengeResponse = new Response.Builder()
      .request(proxyConnectRequest)
      .protocol(Protocol.HTTP_1_1)
      .code(HttpURLConnection.HTTP_PROXY_AUTH)
      .message("Preemptive Authenticate")
      .body(Util.EMPTY_RESPONSE)
      .sentRequestAtMillis(-1L)
      .receivedResponseAtMillis(-1L)
      .header("Proxy-Authenticate", "OkHttp-Preemptive")
      .build();

  // 调用 address 的代理认证器，通过 route 和假的验证响应构造一个认证请求
  // 默认的代理认证器是 Authenticator.NONE，authenticate 方法返回 null
  Request authenticatedRequest = route.address().proxyAuthenticator()
      .authenticate(route, fakeAuthChallengeResponse);

  return authenticatedRequest != null
      ? authenticatedRequest
      : proxyConnectRequest;
}
```

## RealConnection.createTunnel

```java
private Request createTunnel(int readTimeout, int writeTimeout, Request tunnelRequest,
                             HttpUrl url) throws IOException {
  // Make an SSL Tunnel on the first message pair of each SSL + proxy connection.
  String requestLine = "CONNECT " + Util.hostHeader(url, true) + " HTTP/1.1";
  while (true) {
    Http1ExchangeCodec tunnelCodec = new Http1ExchangeCodec(null, null, source, sink);
    source.timeout().timeout(readTimeout, MILLISECONDS);
    sink.timeout().timeout(writeTimeout, MILLISECONDS);
    tunnelCodec.writeRequest(tunnelRequest.headers(), requestLine);
    tunnelCodec.finishRequest();
    Response response = tunnelCodec.readResponseHeaders(false)
      .request(tunnelRequest)
      .build();
    tunnelCodec.skipConnectBody(response);

    switch (response.code()) {
      case HTTP_OK:
        // Assume the server won't send a TLS ServerHello until we send a TLS ClientHello. If
        // that happens, then we will have buffered bytes that are needed by the SSLSocket!
        // This check is imperfect: it doesn't tell us whether a handshake will succeed, just
        // that it will almost certainly fail because the proxy has sent unexpected data.
        if (!source.getBuffer().exhausted() || !sink.buffer().exhausted()) {
          throw new IOException("TLS tunnel buffered too many bytes!");
        }
        return null;

      case HTTP_PROXY_AUTH:
        tunnelRequest = route.address().proxyAuthenticator().authenticate(route, response);
        if (tunnelRequest == null) throw new IOException("Failed to authenticate with proxy");

        if ("close".equalsIgnoreCase(response.header("Connection"))) {
          return tunnelRequest;
        }
        break;

      default:
        throw new IOException(
          "Unexpected response code for CONNECT: " + response.code());
    }
  }
}
```



# ConnectionPoll 分析

连接的复用池，连接拦截器里非常重要的存在

## ConnectionPool 默认配置和初始化

同样配置在 OkHttpClient（Builder）里面，okHttpClient.connectionPool，`new ConnectionPool()`

默认配置的参数：

- 最大闲置连接数：5
- 连接最大保活时间：5 分钟

### ConnectionPool

```java
public final class ConnectionPool {
  final RealConnectionPool delegate;

  /**
   * Create a new connection pool with tuning parameters appropriate for a single-user application.
   * The tuning parameters in this pool are subject to change in future OkHttp releases. Currently
   * this pool holds up to 5 idle connections which will be evicted after 5 minutes of inactivity.
   */
  public ConnectionPool() {
    this(5, 5, TimeUnit.MINUTES);
  }

  public ConnectionPool(int maxIdleConnections, long keepAliveDuration, TimeUnit timeUnit) {
    this.delegate = new RealConnectionPool(maxIdleConnections, keepAliveDuration, timeUnit);
  }

  /** Returns the number of idle connections in the pool. */
  public int idleConnectionCount() {
    return delegate.idleConnectionCount();
  }

  /** Returns total number of connections in the pool. */
  public int connectionCount() {
    return delegate.connectionCount();
  }

  /** Close and remove all idle connections in the pool. */
  public void evictAll() {
    delegate.evictAll();
  }
}
```



### Transmitter 发射器

创建于 `RealCall.newCall`

```java
public Transmitter(OkHttpClient client, Call call) {
  this.client = client;
  this.connectionPool = Internal.instance.realConnectionPool(client.connectionPool());
  this.call = call;
  this.eventListener = client.eventListenerFactory().create(call);
  this.timeout.timeout(client.callTimeoutMillis(), MILLISECONDS);
}
```

#### Internal.instance.realConnectionPool

```java
public RealConnectionPool realConnectionPool(ConnectionPool connectionPool) {
    return connectionPool.delegate;
}
```

返回的就是 okHttpClient.connectionPool.delegate: RealConnectionPool



## RealConnectionPoll

```java
public RealConnectionPool(int maxIdleConnections, long keepAliveDuration, TimeUnit timeUnit) {
  this.maxIdleConnections = maxIdleConnections;
  this.keepAliveDurationNs = timeUnit.toNanos(keepAliveDuration);

  // Put a floor on the keep alive duration, otherwise cleanup will spin loop.
  if (keepAliveDuration <= 0) {
    throw new IllegalArgumentException("keepAliveDuration <= 0: " + keepAliveDuration);
  }
}

// 所有成员变量：

// 清理任务线程池
private static final Executor executor = new ThreadPoolExecutor(0 /* corePoolSize */,
      Integer.MAX_VALUE /* maximumPoolSize */, 60L /* keepAliveTime */, TimeUnit.SECONDS,
      new SynchronousQueue<>(), Util.threadFactory("OkHttp ConnectionPool", true));

// 连接池核心数据结构循队列
private final Deque<RealConnection> connections = new ArrayDeque<>(); 

final RouteDatabase routeDatabase = new RouteDatabase(); // Route 的黑名单

boolean cleanupRunning; // 是否在进行连接的清理的标记

private final int maxIdleConnections; // 最大闲置连接数，控制资源的过度使用
private final long keepAliveDurationNs; // 连接的保活时间，为了最大限度的复用资源

private final Runnable cleanupRunnable = () -> {...} // 清理任务 
```

### RealConnectionPool.put

如果未启动清理任务，那么启动清理任务。放入连接到连接池

```java
void put(RealConnection connection) {
  assert (Thread.holdsLock(this));
  if (!cleanupRunning) {
    cleanupRunning = true;
    executor.execute(cleanupRunnable);
  }
  connections.add(connection); 
}
```



### RealConnectionPool.cleanupRunnable

```java
private final Runnable cleanupRunnable = () -> {
  while (true) {
    long waitNanos = cleanup(System.nanoTime());
    if (waitNanos == -1) return;
    if (waitNanos > 0) {
      long waitMillis = waitNanos / 1000000L;
      waitNanos -= (waitMillis * 1000000L);
      synchronized (RealConnectionPool.this) {
        try {
          RealConnectionPool.this.wait(waitMillis, (int) waitNanos);
        } catch (InterruptedException ignored) {
        }
      }
    }
  }
};
```

#### RealConnectionPool.clear

```java
/**
   * Performs maintenance on this pool, evicting the connection that has been idle the longest if
   * either it has exceeded the keep alive limit or the idle connections limit.
   *
   * <p>Returns the duration in nanos to sleep until the next scheduled call to this method. Returns
   * -1 if no further cleanups are required.
   */
long cleanup(long now) {
  int inUseConnectionCount = 0;
  int idleConnectionCount = 0;
  RealConnection longestIdleConnection = null;
  long longestIdleDurationNs = Long.MIN_VALUE;

  // Find either a connection to evict, or the time that the next eviction is due.
  synchronized (this) {
    for (Iterator<RealConnection> i = connections.iterator(); i.hasNext(); ) {
      RealConnection connection = i.next();

      // If the connection is in use, keep searching.
      if (pruneAndGetAllocationCount(connection, now) > 0) {
        inUseConnectionCount++; // 使用中的连接数
        continue;
      }

      // 到这，说明 pruneAndGetAllocationCount == 0，空闲连接
      idleConnectionCount++;

      // connection.idleAtNanos = now - keepAliveDurationNs; 
      // If the connection is ready to be evicted, we're done.
      long idleDurationNs = now - connection.idleAtNanos;
      if (idleDurationNs > longestIdleDurationNs) {
        longestIdleDurationNs = idleDurationNs; // 所有连接里面最长的空闲时长
        longestIdleConnection = connection; // 空闲最久的连接
      }
    } // for

    // 最长空闲时长超过保活时间 或者 空闲连接数大于最大空闲连接数
    if (longestIdleDurationNs >= this.keepAliveDurationNs
        || idleConnectionCount > this.maxIdleConnections) {
      // We've found a connection to evict. Remove it from the list, then close it below (outside
      // of the synchronized block).
      connections.remove(longestIdleConnection);
    } else if (idleConnectionCount > 0) { // 空闲连接数大于 0
      // A connection will be ready to evict soon.
      return keepAliveDurationNs - longestIdleDurationNs;
    } else if (inUseConnectionCount > 0) { // 使用中连接数大于 0
      // All connections are in use. It'll be at least the keep alive duration 'til we run again.
      return keepAliveDurationNs;
    } else {
      // No connections, idle or in use.
      cleanupRunning = false;
      return -1;
    }
  }

  closeQuietly(longestIdleConnection.socket());

  // Cleanup again immediately.
  return 0;
}
```

##### RealConnectionPool.pruneAndGetAllocationCount

```java
/**
   * Prunes any leaked transmitters and then returns the number of remaining live transmitters on
   * {@code connection}. Transmitters are leaked if the connection is tracking them but the
   * application code has abandoned them. Leak detection is imprecise and relies on garbage
   * collection.
   */
private int pruneAndGetAllocationCount(RealConnection connection, long now) {
  List<Reference<Transmitter>> references = connection.transmitters;
  for (int i = 0; i < references.size(); ) {
    Reference<Transmitter> reference = references.get(i);

    if (reference.get() != null) {
      i++;
      continue;
    }

    // 跳过所有的还未释放的 Transmitter 弱引用
    
    // We've discovered a leaked transmitter. This is an application bug.
    TransmitterReference transmitterRef = (TransmitterReference) reference;
    String message = "A connection to " + connection.route().address().url()
      + " was leaked. Did you forget to close a response body?";
    Platform.get().logCloseableLeak(message, transmitterRef.callStackTrace);

    references.remove(i); 
    connection.noNewExchanges = true; // 这里标记 true 后，这个连接不再被重用

    // If this was the last allocation, the connection is eligible for immediate eviction.
    if (references.isEmpty()) {
      // 当前时间 - 保活时间，相当于说连接已经空闲了一段“保活时间”，可以被马上回收了
      connection.idleAtNanos = now - keepAliveDurationNs; 
      return 0;
    }
  }

  return references.size(); // 返回与这个连接关联的 Transmitter 数
}
```



### RealConnectionPool.transmitterAcquirePooledConnection

遍历 connections 循环数组（ArrayDeque），取出每个 connection 进行检查。符合条件的会被赋值到 transmitter

```java
boolean transmitterAcquirePooledConnection(Address address, Transmitter transmitter,
                                           @Nullable List<Route> routes, boolean requireMultiplexed) {
  assert (Thread.holdsLock(this));
  for (RealConnection connection : connections) {
    // 如果要求多工，但是 connection 不是多工，下一个
    if (requireMultiplexed && !connection.isMultiplexed()) continue;
    // connection 不合格，下一个（合格逻辑后面再分析）
    if (!connection.isEligible(address, routes)) continue;
    // 合格的 connection，赋值到 transmitter 里，返回 true
    transmitter.acquireConnectionNoEvents(connection);
    return true;
  }
  // 全部都遍历完了还没返回，那就返回 false
  return false;
}	
```



## RealConnection & RealConnectionPool

连接的过程中已经涉及到很多 RealConnection 的 api，这里是补充和连接池相关的 api

### RealConnection.isEligible

新建一个Call，需要新建一个 Transmitter，transmitter.prepareToConnect 的时候会新建一个 ExchangeFinder，ExchangeFinder 也会需要新建一个 Address。

RealConnection.isEligible 的目的就是根据新建的 Address 和已有的 RealConnection.route.address 进行判断来确定这个连接是否可以被重用

```java
boolean isEligible(Address address, @Nullable List<Route> routes) {
  // address 是新建 Call 的
  
  // 连接的已经关联的 transmitters 数量大于等于 allocationLimit（默认 5）
  // noNewExchanges 标记为 true，这个连接不能再用于创建 Exchange 了
  // If this connection is not accepting new exchanges, we're done.
  if (transmitters.size() >= allocationLimit || noNewExchanges) return false;

  // If the non-host fields of the address don't overlap, we're done.
  // Internal.instance.equalsNonHost 调用的是 route.address().equalsNonHost(address)
  // 这个方法比较了 Address 里面的所有成员变量是否相同，除了 host 和 socketFactory
  // 不过核心目的就是为了不考虑 host 的情况下，看 address 是否相同，不同的话，连接无法重用
  if (!Internal.instance.equalsNonHost(this.route.address(), address)) return false;

  // If the host exactly matches, we're done: this connection can carry the address.
  // 新的 Address 的 url.host 如果和连接route.address.url.host 相同，就可以重用连接
  if (address.url().host().equals(this.route().address().url().host())) {
    return true; // This connection is a perfect match.
  }

  // At this point we don't have a hostname match. But we still be able to carry the request if
  // our connection coalescing requirements are met. See also:
  // https://hpbn.co/optimizing-application-delivery/#eliminate-domain-sharding
  // https://daniel.haxx.se/blog/2016/08/18/http2-connection-coalescing/
  
  // 执行到了这，因为相同域名导致连接重用的情况已经都判断完了
  // 但是重用连接或者合并连接的需求在其他情况下依然是可行的

  // 1. This connection must be HTTP/2.
  if (http2Connection == null) return false; // http2 才有机会重用连接

  // 2. The routes must share an IP address. 
  // routeMatchesAny 方法：
  // routes 存在有代理类型和 InetSocketAddress 当前连接 route 的代理类型和 InetSocketAddress 
  // 相同的元素返回 true，即比较了代理类型和 IP+PORT
  if (routes == null || !routeMatchesAny(routes)) return false;

  // 3. This connection's server certificate's must cover the new host.
  // 证书主机名验证器必须得相同，不同也不能重用
  // url 端口必须一样，主机名不同但是证书是相同的，也可以
  if (address.hostnameVerifier() != OkHostnameVerifier.INSTANCE) return false;
  if (!supportsUrl(address.url())) return false;

  // 4. Certificate pinning must match the host.
  // 证书锁定必须是相同的主机名
  try {
    address.certificatePinner().check(address.url().host(), handshake().peerCertificates());
  } catch (SSLPeerUnverifiedException e) {
    return false;
  }

  return true; // The caller's address can be carried by this connection.
}
```

#### 对符合重用连接条件的请求的总结 [待补充]





# ConnectionSpec 连接规格

**ConnectionSpec** 其实就是连接的配置，是要进行明文传输还是 TLS 握手后进行加密传输，其中 TLS 又有几个配置选项（当然也可以自定义）。

**ConnectionSpecSelector** 的主要作用就是找到一个可用的连接规格，标记是否有备用的连接规格，把找到的这个可用的连接规格里面可用的 tls 版本和加密套件设置到 SSLSocket 里面

连接规格 ConnectionSpec 使用在 RealConnection 要进行连接操作的时候，具体到

```java
RealConnection.conncet // 连接
	RealConnection.establishProtocol // 协议建立
		RealConnection.connectTls // TLS 连接（TLS 握手）
```

对它进行配置，目的是在于明确可以建立的连接类型：明文，TLS 加密传输，其中 TLS 加密传输又可配置加密套件，TLS 版本，是否支持扩展

okhttp 内置了几个 TLS 加密传输规格：

- MODERN_TLS，流行的配置，可用于连接大部分服务器
- RESTRICTED_TLS，受限的配置，需要较新的客户端和服务端，最安全
- COMPATIBLE_TLS，兼容的配置，向下兼容老的客户端和服务端，不推荐使用，应该考虑升级

## ConnectionSpec 类结构

```java
public final class ConnectionSpec {
	final boolean tls; // 是否支持 tls
  final boolean supportsTlsExtensions; // 是否支持 tls 扩展
  final @Nullable String[] cipherSuites; // 加密套件
  final @Nullable String[] tlsVersions; // 支持 tls 版本
  ...
}
```



## okhttp 默认配置 MODERN_TLS & CLEARTEXT

默认配置，支持明文和最流行的 TLS 配置

```java
static final List<ConnectionSpec> DEFAULT_CONNECTION_SPECS = Util.immutableList(
    ConnectionSpec.MODERN_TLS, ConnectionSpec.CLEARTEXT);
```

```java
public static final ConnectionSpec MODERN_TLS = new Builder(true)
      .cipherSuites(APPROVED_CIPHER_SUITES)
      .tlsVersions(TlsVersion.TLS_1_3, TlsVersion.TLS_1_2)
      .supportsTlsExtensions(true)
      .build();

/** Unencrypted, unauthenticated connections for {@code http:} URLs. */
public static final ConnectionSpec CLEARTEXT = new Builder(false).build();
```



## ConnectionSpecSelector 连接规格选择器

提供了对连接规格列表的封装和操作

```java
ConnectionSpecSelector(List<ConnectionSpec> connectionSpecs) {
  this.nextModeIndex = 0;
  this.connectionSpecs = connectionSpecs;
}

// 全部成员变量
private final List<ConnectionSpec> connectionSpecs;
private int nextModeIndex;
private boolean isFallbackPossible;
private boolean isFallback;
```



## ConnectionSpec 方法

### ConnectionSpec.isCompatible

根据 SSLSocket 检查这个连接规格是否可用。

nonEmptyIntersection 方法，入参是 Comparator 接口（类似于函数指针），然后比较的字符串数组 first[]，second[]，两个 for 循环，比较两个字符串数组，`comparator.compare(first[i], second[j]) == 0` ，即相等，返回 true。

```java
public boolean isCompatible(SSLSocket socket) {
  if (!tls) { // 不支持 tls，不兼容
    return false;
  }

  // tlsVersions 存在和 SSLSocket 启用的协议中相同的项，这个连接规格可用
  // 否则，不可用
  if (tlsVersions != null && !nonEmptyIntersection(
    Util.NATURAL_ORDER, tlsVersions, socket.getEnabledProtocols())) {
    return false;
  }

  // cipherSuites 存在和 SSLSocket 启用的加密套件中相同的项，规格可用
  // 否则，不兼容
  if (cipherSuites != null && !nonEmptyIntersection(
    CipherSuite.ORDER_BY_NAME, cipherSuites, socket.getEnabledCipherSuites())) {
    return false;
  }

  return true;
}
```



### ConnectionSpec.apply

概括这个方法的作用：构造一个连接规格，这个规格里面的 tls 版本和加密套件等于 “当前连接规格” 的 tls 版本和加密套件与 SSLSocket 启用的 tls 版本和加密套件的最大子集。最后会把这个连接规格的 tls 版本和加密套件设置到 SSLSocket 的启用中

```java
void apply(SSLSocket sslSocket, boolean isFallback) {
  ConnectionSpec specToApply = supportedSpec(sslSocket, isFallback);

  // 将 “新建的连接规格” tls 版本数组和加密套件数组设置到 SSLSocket 里面，其中
  //
  // 新建的连接规格：包含当前连接规格加密套件名称和 SSLSocket 起效加密套件名称的最大子集
  // 新建的连接规格：包含当前连接规格 tls 版本和 SSLSocket 起效 tls 版本的最大子集
  //
  // 如果 isFallback 为 true，并且当前 SSLSocket 支持的加密套件有 TLS_FALLBACK_SCSV
  // 那么“新建的连接规格”的加密套件中还会包含 TLS_FALLBACK_SCSV
  
  if (specToApply.tlsVersions != null) {
    sslSocket.setEnabledProtocols(specToApply.tlsVersions);
  }
  if (specToApply.cipherSuites != null) {
    sslSocket.setEnabledCipherSuites(specToApply.cipherSuites);
  }
}
```

#### ConnectionSpec.supportedSpec

intersect 方法，从第一个数组和第二个数组得到一个最大子集

```java
private ConnectionSpec supportedSpec(SSLSocket sslSocket, boolean isFallback) {
	// 找到 SSLSocket 启用的加密套件和连接规格中的加密套件名字的最大子集
  String[] cipherSuitesIntersection = cipherSuites != null
    ? intersect(CipherSuite.ORDER_BY_NAME, sslSocket.getEnabledCipherSuites(), cipherSuites)
    : sslSocket.getEnabledCipherSuites();
  // 找到 SSLSocket 启用的协议协议和连接规则中的 tls 协议的最大子集
  String[] tlsVersionsIntersection = tlsVersions != null
    ? intersect(Util.NATURAL_ORDER, sslSocket.getEnabledProtocols(), tlsVersions)
    : sslSocket.getEnabledProtocols();

  // In accordance with https://tools.ietf.org/html/draft-ietf-tls-downgrade-scsv-00
  // the SCSV cipher is added to signal that a protocol fallback has taken place.
  String[] supportedCipherSuites = sslSocket.getSupportedCipherSuites();
  // 从 SSLSocket 支持的加密套件里面找到 TLS_FALLBACK_SCSV 的下标
  int indexOfFallbackScsv = indexOf(
    CipherSuite.ORDER_BY_NAME, supportedCipherSuites, "TLS_FALLBACK_SCSV");
  if (isFallback && indexOfFallbackScsv != -1) {
    // 把 TLS_FALLBACK_SCSV 加入到子集最后
    cipherSuitesIntersection = concat(
      cipherSuitesIntersection, supportedCipherSuites[indexOfFallbackScsv]);
  }

  // 构建一个新的连接规格
  return new Builder(this)
    .cipherSuites(cipherSuitesIntersection)
    .tlsVersions(tlsVersionsIntersection)
    .build();
}
```



## ConnectionSpecSeletor 方法

### ConnectionSpecSelector.configureSecureSocket

根据 SSLSocket 启用的 tls 版本和加密套件，找到一个可用的连接规格，同时标记是否有备用的连接规格，最后把这个连接规格应用到 SSLSocket 上

```java
ConnectionSpec configureSecureSocket(SSLSocket sslSocket) throws IOException {
  ConnectionSpec tlsConfiguration = null;
  for (int i = nextModeIndex, size = connectionSpecs.size(); i < size; i++) {
    ConnectionSpec connectionSpec = connectionSpecs.get(i);
    if (connectionSpec.isCompatible(sslSocket)) {
      tlsConfiguration = connectionSpec;
      nextModeIndex = i + 1;
      break;
    }
  }
  // 上面的 for 循环是为了找到一个连接规格，这个连接规格支持 tls
  // 且 tls 版本和加密套件与当前 SSLSocket 起效 tls 版本和加密套件存在子集

  // 如果没好到合适的连接规格，没办法，只能抛出异常了
  if (tlsConfiguration == null) {
    // This may be the first time a connection has been attempted and the socket does not support
    // any the required protocols, or it may be a retry (but this socket supports fewer
    // protocols than was suggested by a prior socket).
    throw new UnknownServiceException(
        "Unable to find acceptable protocols. isFallback=" + isFallback
            + ", modes=" + connectionSpecs
            + ", supported protocols=" + Arrays.toString(sslSocket.getEnabledProtocols()));
  }

  // 标记 “是否有可以备用的连接规格”
  // 逻辑是从后面的（nextModeIndex）的连接规格中查找合适的连接规格作为应急，有就会返回 true
  isFallbackPossible = isFallbackPossible(sslSocket);

  // 实际实现代码：tlsConfiguration.apply(sslSocket, isFallback);
  // 实际上是基于 tlsConfiguration 和 SSLSocket tls 版本和加密套件名的最大子集，创建新的连接规格
  // 然后用 sslSocket.setEnabledProtocols 和 sslSocket.setEnabledCipherSuites 
  // 把新的连接规格的 tls 版本和加密套件名设置进去
  //
  // 如果 isFallback 为 true，并且当前 SSLSocket 支持的加密套件有 TLS_FALLBACK_SCSV
  // 那么 TLS_FALLBACK_SCSV 也会被设置进去
  Internal.instance.apply(tlsConfiguration, sslSocket, isFallback);

  // 把这个合适的 tlsConfiguration 返回
  return tlsConfiguration;
}
```

#### ConnectionSpecSelector.isFallbackPossible

这个方法其实和之前找一个合适的连接规格的逻辑是一样的，目的是为了标记 “是否有备用的连接规格”

```java
private boolean isFallbackPossible(SSLSocket socket) {
  for (int i = nextModeIndex; i < connectionSpecs.size(); i++) {
    if (connectionSpecs.get(i).isCompatible(socket)) {
      return true;
    }
  }
  return false;
}
```



### ConnectionSpecSelector.connectionFailed

```java
boolean connectionFailed(IOException e) {
  // Any future attempt to connect using this strategy will be a fallback attempt.
  isFallback = true;  
  // 这个成员变量标记为 true 会影响到 
  // configureSecureSocket -> Internal.instance.apply -> ConnectionSpec.apply
  // -> ConnectionSpec.supportedSpec -> SSLSocket 支持的加密套件中有 TLS_FALLBACK_SCSV 的情况下
  // TLS_FALLBACK_SCSV 会被设入 SSLSocket 启用的加密套件配置中

  if (!isFallbackPossible) { // 无备用连接规格，取消重试
    return false;
  }

  // If there was a protocol problem, don't recover.
  if (e instanceof ProtocolException) { // 协议异常，取消重试
    return false;
  }

  // If there was an interruption or timeout (SocketTimeoutException), don't recover.
  // For the socket connect timeout case we do not try the same host with a different
  // ConnectionSpec: we assume it is unreachable.
  if (e instanceof InterruptedIOException) { //线程中断异常或者 Socket 读写超时，取消重试
    return false;
  }

  // Look for known client-side or negotiation errors that are unlikely to be fixed by trying
  // again with a different connection spec.
  if (e instanceof SSLHandshakeException) { 
    // If the problem was a CertificateException from the X509TrustManager, do not retry.
    if (e.getCause() instanceof CertificateException) {
      return false; // 握手异常且由于是证书导致，取消重试
    }
  }
  if (e instanceof SSLPeerUnverifiedException) {
    // e.g. a certificate pinning error.
    return false; // 证书域名验证未通过的异常，取消重试
  }

  // Retry for all other SSL failures.
  return e instanceof SSLException; // 其他属于 SSLException 的，重试
}
```





# Platform 平台兼容

这个类主要是负责平台兼容性的，里面涉及到了 Android 和 openJdk。由于 Android 平台在对 http(s) 的兼容上在不同版本上也存在兼容问题，因此存在了这个类的意义。

okhttp3.14.x 在 Android 平台主要用于处理 Android 5.0+ 和 Android 10 的 https 的兼容问题。Android5.0 之前的兼容问题需要查看 okhttp3.12.x 的实现

```java
// 静态单例懒加载
private static final Platform PLATFORM = findPlatform();  

public static Platform get() {
  return PLATFORM;
}
```



## Platform.findPlatform

```java
private static Platform findPlatform() {
  Platform android10 = Android10Platform.buildIfSupported();

  if (android10 != null) { // 大于等于 api 29 的会使用 Android10Platform
    return android10;
  }

  // 小于 android 10 大于等于 android 5 的 Platform
  // 小于 android 5 的用 okhttp3.12.x
  Platform android = AndroidPlatform.buildIfSupported();

  if (android != null) {
    return android;
  }

  // 下面这些是其他平台的
  if (isConscryptPreferred()) {
    Platform conscrypt = ConscryptPlatform.buildIfSupported();

    if (conscrypt != null) {
      return conscrypt;
    }
  }

  Platform jdk9 = Jdk9Platform.buildIfSupported();

  if (jdk9 != null) {
    return jdk9;
  }

  Platform jdkWithJettyBoot = Jdk8WithJettyBootPlatform.buildIfSupported();

  if (jdkWithJettyBoot != null) {
    return jdkWithJettyBoot;
  }

  // Probably an Oracle JDK like OpenJDK.
  return new Platform();
}
```



### Android10Platform

继承于 AndroidPlatform，大部分方法都是对 AndroidPlatform 的重写

```java
class Android10Platform extends AndroidPlatform {
  Android10Platform(Class<?> sslParametersClass) {
    super(sslParametersClass, null, null, null, null, null);
  }
  ...
}
```



### Android10Platform.buildIfSupported

根据 Build.VERSION.SDK_INT 知道当前版本，并且通过反射得到 SSL 参数的实现类，并进行初始化

```java
public static @Nullable Platform buildIfSupported() {
  try {
    // 返回 Build.VERSION.SDK_INT
    if (getSdkInt() >= 29) {
      Class<?> sslParametersClass =
        Class.forName("com.android.org.conscrypt.SSLParametersImpl");

      return new Android10Platform(sslParametersClass);
    }
  } catch (ReflectiveOperationException ignored) {
  }

  return null; // Not an Android 10+ runtime.
}
```



### AndroidPlatform

这个类的作用主要是通过反射获取和 SSL/TSL 协议相关的类然后进行例如 SessionTicket，ALPN （TLS 的扩展）的设置

```java
AndroidPlatform(Class<?> sslParametersClass, Class<?> sslSocketClass, Method setUseSessionTickets,
    Method setHostname, Method getAlpnSelectedProtocol, Method setAlpnProtocols) {
  this.sslParametersClass = sslParametersClass;
  this.sslSocketClass = sslSocketClass;
  this.setUseSessionTickets = setUseSessionTickets;
  this.setHostname = setHostname;
  this.getAlpnSelectedProtocol = getAlpnSelectedProtocol;
  this.setAlpnProtocols = setAlpnProtocols;
}

// 其他成员变量
private final CloseGuard closeGuard = CloseGuard.get();
```



### AndroidPlatform.buildIfSupported

这个方法会构建 Android 5 及其以上版本（小于 Android10）的 Platform 类。

**低于 Android5 的版本会抛出异常。低于 Android 5 以下的版本可以使用低版本的 okHttp（3.12.x）**。

当前版本 3.14.7。

```java
public static @Nullable Platform buildIfSupported() {
  if (getSdkInt() == 0) { // Build.VERSION.SDK_INT == 0
    return null;
  }

  // Attempt to find Android 5+ APIs.
  Class<?> sslParametersClass;
  Class<?> sslSocketClass;

  try {
    // api 10 android 2.3.3 添加
    sslParametersClass = Class.forName("com.android.org.conscrypt.SSLParametersImpl");
    // api 6 添加
    sslSocketClass = Class.forName("com.android.org.conscrypt.OpenSSLSocketImpl");
  } catch (ClassNotFoundException ignored) {
    return null; // Not an Android runtime.
  }
  if (Build.VERSION.SDK_INT >= 21) { // Android 5.0
    try {
      Method setUseSessionTickets = sslSocketClass.getDeclaredMethod(
        "setUseSessionTickets", boolean.class);
      Method setHostname = sslSocketClass.getMethod("setHostname", String.class);
      Method getAlpnSelectedProtocol = sslSocketClass.getMethod("getAlpnSelectedProtocol");
      Method setAlpnProtocols = sslSocketClass.getMethod("setAlpnProtocols", byte[].class);
      return new AndroidPlatform(sslParametersClass, sslSocketClass, setUseSessionTickets,
                                 setHostname, getAlpnSelectedProtocol, setAlpnProtocols);
    } catch (NoSuchMethodException ignored) {
    }
  }
  throw new IllegalStateException(
    "Expected Android API level 21+ but was " + Build.VERSION.SDK_INT);
}
```



## AndroidPlatform.getSSLContext

不存在 Android10Platform 重写。

根据不同的 Android 版本返回不一样 SSLContext 对象：Android[4.1 ~ 5]，即 api：[16，22)，会明确指定 TLSv1.2 的 SSLContext。

其他版本，SSLContext(TLS)，在实际的运行选择中，会优先选择最佳的协议版本， Android5 及以上也会优先选择 TLSv1.2，除非有些这个版本的手机特别

```java
@Override public SSLContext getSSLContext() {
  boolean tryTls12;
  try {
    tryTls12 = (Build.VERSION.SDK_INT >= 16 && Build.VERSION.SDK_INT < 22);
  } catch (NoClassDefFoundError e) {
    // Not a real Android runtime; probably RoboVM or MoE
    // Try to load TLS 1.2 explicitly.
    tryTls12 = true;
  }

  if (tryTls12) {
    try {
      return SSLContext.getInstance("TLSv1.2");
    } catch (NoSuchAlgorithmException e) {
      // fallback to TLS
    }
  }

  try {
    return SSLContext.getInstance("TLS");
  } catch (NoSuchAlgorithmException e) {
    throw new IllegalStateException("No TLS provider", e);
  }
}
```



## AndroidPlatform.connectSocket

不存在 Android10Platform 重写

```java
@Override public void connectSocket(Socket socket, InetSocketAddress address,
                                    int connectTimeout) throws IOException {
  try {
    socket.connect(address, connectTimeout);
  } catch (AssertionError e) {
    if (Util.isAndroidGetsocknameError(e)) throw new IOException(e);
    throw e;
  } catch (ClassCastException e) {
    // On android 8.0, socket.connect throws a ClassCastException due to a bug
    // see https://issuetracker.google.com/issues/63649622
    if (Build.VERSION.SDK_INT == 26) {
      throw new IOException("Exception in connect", e);
    } else {
      throw e;
    }
  }
}
```



## AndroidPlatform.configureTlsExtensions

存在 Android10Platform 的重写

```java
@Override public void configureTlsExtensions(
  SSLSocket sslSocket, String hostname, List<Protocol> protocols) {
 	// sslSocketClass 为 com.android.org.conscrypt.OpenSSLSocketImpl 的 class 类
  if (!sslSocketClass.isInstance(sslSocket)) {
    return; // No TLS extensions if the socket class is custom.
  }
  try {
    // Enable SNI and session tickets.
    if (hostname != null) {
      // 反射调用 sslSocket.setUseSessionTickets(true)
      setUseSessionTickets.invoke(sslSocket, true);
      // 反射调用 sslSocket.setHostname(hostname)
      // This is SSLParameters.setServerNames() in API 24+.
      setHostname.invoke(sslSocket, hostname);
    }

    // 反射调用 sslSocket.setAlpnProtocols(concatLengthPrefixed(protocols))
    // Enable ALPN.
    setAlpnProtocols.invoke(sslSocket, concatLengthPrefixed(protocols));
  } catch (IllegalAccessException | InvocationTargetException e) {
    throw new AssertionError(e);
  }
}
```

### Android10Platform.configureTlsExtensions

Android 10+ configureTlsExtensions 方法存在重写

```java
@SuppressLint("NewApi")
@IgnoreJRERequirement
@Override public void configureTlsExtensions(
    SSLSocket sslSocket, String hostname, List<Protocol> protocols) {
  // Android 10 使用了新的 api SSLSockets.setUseSessionTickets(sslSocket, true);
  enableSessionTickets(sslSocket);

  SSLParameters sslParameters = sslSocket.getSSLParameters();

  // 配置 ALPN，也使用了 Android 10 的 api，sslParameters.setApplicationProtocols
  // Enable ALPN. 
  String[] protocolsArray = Platform.alpnProtocolNames(protocols).toArray(new String[0]);
  sslParameters.setApplicationProtocols(protocolsArray);

  sslSocket.setSSLParameters(sslParameters);
}

private void enableSessionTickets(SSLSocket sslSocket) {
  if (SSLSockets.isSupportedSocket(sslSocket)) {
    SSLSockets.setUseSessionTickets(sslSocket, true);
  }
}
```



## AndroidPlatform.getSelectedProtocol

```java
@Override public @Nullable String getSelectedProtocol(SSLSocket socket) {
  if (!sslSocketClass.isInstance(socket)) {
    return null; // No TLS extensions if the socket class is custom.
  }
  try {
    // 反射调用 SSLSocket.getAlpnSelectedProtocol 获取最后协商出来后要使用的协议
    byte[] alpnResult = (byte[]) getAlpnSelectedProtocol.invoke(socket);
    return alpnResult != null ? new String(alpnResult, UTF_8) : null;
  } catch (IllegalAccessException | InvocationTargetException e) {
    throw new AssertionError(e);
  }
}
```

### Android10Platform.getSelectedProtocol

Androd 10 重写这个方法，直接使用新的 api 获取 tls 协商确定的协议

```java
@IgnoreJRERequirement
@Override public @Nullable String getSelectedProtocol(SSLSocket socket) {
  String alpnResult = socket.getApplicationProtocol();

  if (alpnResult == null || alpnResult.isEmpty()) {
    return null;
  }

  return alpnResult;
}
```



## AndroidPlatform.isCleartextTrafficPermitted

逻辑：

- 反射获取到 android.security.NetworkSecurityPolicy 的 Class，调用 getIntance 获取到单例对象
- 如果找不到这个类，说明当前版本 < api23，直接调用 (Platform)super.isCleartextTrafficPermitted，返回 true。
- 然后调用 api24 添加的 api：isCleartextTrafficPermitted(hostname) 判断是否允许明文网络传输
- 如果这个 api 没有找到（无方法异常），说明当前是系统版本 api23，调用 api23 的方法，isCleartextTrafficPermitted()，一般无法再出现无方法异常

其实这个逻辑完全可以使用版本判断的方法来实现：如下

```java
if (sdk < 23) return true;
else if (sdk == 23) NetworkSecurityPolicy.getInstance().isCleartextTrafficPermitted();
else if (sdk >= 24)
  NetworkSecurityPolicy.getInstance().isCleartextTrafficPermitted(hostname);
```

okhttp 的源代码：

```java
@Override public boolean isCleartextTrafficPermitted(String hostname) {
  try {
    Class<?> networkPolicyClass = Class.forName("android.security.NetworkSecurityPolicy");
    Method getInstanceMethod = networkPolicyClass.getMethod("getInstance");
    Object networkSecurityPolicy = getInstanceMethod.invoke(null);
    return api24IsCleartextTrafficPermitted(hostname, networkPolicyClass, networkSecurityPolicy);
  } catch (ClassNotFoundException | NoSuchMethodException e) {
    return super.isCleartextTrafficPermitted(hostname);
  } catch (IllegalAccessException | IllegalArgumentException | InvocationTargetException e) {
    throw new AssertionError("unable to determine cleartext support", e);
  }
}
```

### Android10Platform.api24IsCleartextTrafficPermitted

```java
private boolean api24IsCleartextTrafficPermitted(String hostname, Class<?> networkPolicyClass,
    Object networkSecurityPolicy) throws InvocationTargetException, IllegalAccessException {
  try {
    Method isCleartextTrafficPermittedMethod = networkPolicyClass
        .getMethod("isCleartextTrafficPermitted", String.class);
    return (boolean) isCleartextTrafficPermittedMethod.invoke(networkSecurityPolicy, hostname);
  } catch (NoSuchMethodException e) {
    return api23IsCleartextTrafficPermitted(hostname, networkPolicyClass, networkSecurityPolicy);
  }
}
```

#### Android10Platform.api23IsCleartextTrafficPermitted

```java
private boolean api23IsCleartextTrafficPermitted(String hostname, Class<?> networkPolicyClass,
    Object networkSecurityPolicy) throws InvocationTargetException, IllegalAccessException {
  try {
    Method isCleartextTrafficPermittedMethod = networkPolicyClass
        .getMethod("isCleartextTrafficPermitted");
    return (boolean) isCleartextTrafficPermittedMethod.invoke(networkSecurityPolicy);
  } catch (NoSuchMethodException e) {
    // Platform.isCleartextTrafficPermitted(hostname) return true
    return super.isCleartextTrafficPermitted(hostname); 
  }
}
```



# X509TrustManager & SslSocketFactory

也是属于 okHttpClient 可配置项，和 ConnectionSpec 连接规格有关系，在 okHttpClient 初始化的时候有以下逻辑

```java
// OkHttpClient(Builder builder) 方法
boolean isTLS = false;
for (ConnectionSpec spec : connectionSpecs) { // 连接规格里面定义了 TLS 的规格，isTLS 就会为 true
  isTLS = isTLS || spec.isTls();
}

// 如果配置过 sslSocketFactory 或者配置的规格里不存在 tls 的
if (builder.sslSocketFactory != null || !isTLS) { 
  this.sslSocketFactory = builder.sslSocketFactory;
  this.certificateChainCleaner = builder.certificateChainCleaner;
} else { // builder.sslSocketFactory == null && isTLS，Builder.sslSocketFactory 默认是 null
  X509TrustManager trustManager = Util.platformTrustManager();
  this.sslSocketFactory = newSslSocketFactory(trustManager);
  this.certificateChainCleaner = CertificateChainCleaner.get(trustManager);
}
```

## Utils.platformTrustManager 

获得系统已安装的根证书的管理器

```java
// Utils.platformTrustManager
public static X509TrustManager platformTrustManager() {
  try {
    TrustManagerFactory trustManagerFactory = TrustManagerFactory.getInstance(
      TrustManagerFactory.getDefaultAlgorithm());
    trustManagerFactory.init((KeyStore) null);
    TrustManager[] trustManagers = trustManagerFactory.getTrustManagers();
    if (trustManagers.length != 1 || !(trustManagers[0] instanceof X509TrustManager)) {
      throw new IllegalStateException("Unexpected default trust managers:"
                                      + Arrays.toString(trustManagers));
    }
    return (X509TrustManager) trustManagers[0];
  } catch (GeneralSecurityException e) {
    throw new AssertionError("No System TLS", e); // The system has no TLS. Just give up.
  }
}
```

## okHttpClient.newSslSocketFactory

然后用根证书的管理器 和 SSLContext 创建 SSLSocketFactory

```java
private static SSLSocketFactory newSslSocketFactory(X509TrustManager trustManager) {
  try {
    // 当前 Android 版本[4.1 ~ 5.0]
    // 即 api 大于等于 16 小于 22，返回 SSLContext.getInstance("TLSv1.2")
    // 其他版本，返回 SSLContext.getInstance("TLS") ，最终协议也会选择一个最好的版本，比如 TLSv1.2
    SSLContext sslContext = Platform.get().getSSLContext();
    sslContext.init(null, new TrustManager[] { trustManager }, null);
    return sslContext.getSocketFactory();
  } catch (GeneralSecurityException e) {
    throw new AssertionError("No System TLS", e); // The system has no TLS. Just give up.
  }
}
```



## TLS 自签证书的配置 [待补充]

okHttpClient 默认的实现是信任所有系统自带的根证书的CA机构颁发的证书。即发出请求给服务端，在做 TLS 握手证书验证的时候，只会信任系统自带根证书的 CA 机构颁发的证书。

这意味着，如果是我们自己服务器自签证书，这个配置是无法满足的。因此需要去实现**自签证书信任管理  + 系统证书的信任管理**。实现以后补充。



# CertificatePinner 证书锁定



# Android 与 TLS/ALPN/HTTPS/HTTP2 

HTTPS = HTTP + TLS

HTTP 当前主要版本，HTTP1.1，HTTP2

TLS 当前主要版本，SSL3，TLS1，TLS1.1，TLS1.2，TLS1.3

Android 版本对 TLS 的支持，16：Android4.1，20：Android4.4，29：Android10，相关类 SSLContext

| 协议   | 支持的 api | 默认启动的 api |
| ------ | ---------- | -------------- |
| SSL3   | 1+         | 1+             |
| TLS1   | 1+         | 1+             |
| TLS1.1 | 16+        | 20+            |
| TLS1.2 | 16+        | 20+            |
| TLS1.3 | 29+        | 29+            |

Android 对 HTTP 的支持，HttpUrlConnection 当前底层也是使用 okhttp 的代码，但是只支持 Http1.1。因此Android 要实现对 HTTP2 的支持，得使用 okhttp

Android 对于 ALPN 的支持是从 Android5.0 开始

最后再看 HTTPS，从上述信息可得，Android4.1 开始，要实现 HTTP1.1+ （SSL3 ~ TLS1.2） 都是可行的，**但是 Android4.1（16）~ 4.4（20），TLS1.2 默认没生效，因此不管在用 HttpUrlConnection 还是 okhttp 都可能需要配置**

再看 HTTP2 +（SSL3 ~ TLS1.2）由于需要 ALPN 的协商才能完成 TLS 的协议协商，因此这种组合只能限定在 Android5（21） 之后



## 为什么 okhttp 3.14.x 只支持 Android 5+

需要支持 Android 5 以下，兼容到 Android 2.3 需要使用 okhttp 3.12.x

原因之一：

> The OkHttp 3.12.x branch supports Android 2.3+ (API level 9+) and Java 7+. These platforms lack support for TLS 1.2 and should not be used. But because upgrading is difficult we will backport critical fixes to the 3.12.x branch through December 31, 2020
>
> The OkHttp 3.12.x 支持 Android2.3+（API 9+）以及 Java7。这些平台缺乏对 TLS1.2 的支持，不应再被使用了，但是升级又是比较困难的，因此我们会维护 3.12.x 到 2020.12.31

结合前面总结的内容：

- Android 5 之前对 tls1.2 的支持不友好，然后也不支持 ALPN，即 3.12.x 其实是无法实现 H2 的 HTTPS
- Android 5 之后，okhttp3.14.x 完全实现了完整的 http1.1 和 h2 的 https



## ALPN（TLS 扩展）

**应用层协议协商**（**Application-Layer Protocol Negotiation**，简称 **ALPN**）是一个传输层安全协议 (TLS) 的扩展，ALPN 使得应用层可以协商在安全连接层之上使用什么协议，避免了额外的往返通讯，并且独立于应用层协议。 ALPN 是用于 HTTP/2 连接的 TLS 扩展

主要作用是 TLS  会话发送 ClientHello 时候附带支持的协议列表（http2，http1.1），然后再 ServerHello 的时候会回复协议的版本（会发生 http1.1 的降级）

> 服务端是否支持 ALPN 要看 openSLL 的版本