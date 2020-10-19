# Transmitter 代码分析



## 创建

RealCall.newRealCall 

```java
static RealCall newRealCall(OkHttpClient client, Request originalRequest, boolean forWebSocket) {
  // Safely publish the Call instance to the EventListener.
  RealCall call = new RealCall(client, originalRequest, forWebSocket);
  call.transmitter = new Transmitter(client, call);
  return call;
}
```



## 构造器

```java
public Transmitter(OkHttpClient client, Call call) {
  this.client = client;
  this.connectionPool = Internal.instance.realConnectionPool(client.connectionPool());
  this.call = call;
  // 默认的事件监听是空实现
  this.eventListener = client.eventListenerFactory().create(call);
  // 默认 callTimeoutMillis  是 0
  this.timeout.timeout(client.callTimeoutMillis(), MILLISECONDS);  
}
```



## 成员变量

```java
private final OkHttpClient client;
private final RealConnectionPool connectionPool;
private final Call call;
private final EventListener eventListener; 
private final AsyncTimeout timeout = () -> {cancel()};
private @Nullable Object callStackTrace;

private Request request;
private ExchangeFinder exchangeFinder;

// Guarded by connectionPool.
public RealConnection connection;
private @Nullable Exchange exchange;
private boolean exchangeRequestDone;
private boolean exchangeResponseDone;
private boolean canceled;
private boolean timeoutEarlyExit;
private boolean noMoreExchanges;
```



## 成员方法与调用

```java
// Transmitter.java
.timeout():Timeout; // 得到 AsyncTimeout timeout 对象
.timeoutEnter(); // 调用 timeout.enter()，RealCall.execute 的时候被调用
.timeoutExit(@Nullable IOException cause);
.callStart(); // RealCall.execute 的时候被调用，记录请求开始执行

// 准备创建一个流来运输请求，会创建 ExchangeFinder，第一个重试拦截器，创建连接或者重用连接前调用
.prepareToConnect(Request request); 
// prepareToConnect 内部用于创建 ExchangeFinder 的时候创建 Address 使用
.createAddress(HttpUrl url):Address; 

// 创建 Exchange 用于运输请求和响应，用于连接拦截器
.newExchange(Interceptor.Chain chain, boolean doExtensiveHealthChecks):Exchange;

// 连接对象关联到 transmitter 对象上，连接持有实际负责通信的 socket 对象
// 创建新的连接，或者在查找到可复用的连接后调用这个方法，把连接管理到 transmitter
// 并且把这个 transmitter 加入到连接的 transmitters 弱引用 ArrayList
.acquireConnectionNoEvents(RealConnection connection);
  
// 释放连接对象，同时也删除连接里面的 transmitter 弱引用，也删除连接池里的这个连接对象
// 查找连接 ExchangeFinder.findConnection 和 Transmitter.maybeReleaseConnection 被调用
.releaseConnectionNoEvents():Socket;

// 根据 noMoreExchanges 值判断是否抛出异常，或者是否 Exchange 对象
// 在请求处理线程池抛出拒绝异常的时候调用
// 在 RealCall.getResponseWithInterceptorChain 责任链返回响应 chain.proceed(originalRequest) 的
// 时候，捕获到了 IO 异常的时候会调用
.exchangeDoneDueToException():IOException;

// Transmitter 私有方法，被 noMoreExchanges，prepareToConnect，exchangeMessageDone 调用
// 是否不再被需要的连接，在数据交换完成之后，以及不再需要数据交换后
.maybeReleaseConnection(@Nullable IOException e, boolean force):IOException;

// 能否重试，根据 exchangeFinder.hasStreamFailure 和 exchangeFinder.hasRouteToTry 判断
// 在重试拦截器的 recvoer 判断能否恢复与服务器的会话里面调用
.canRetry():true;

// 加锁连接池后，判断 exchange 是否为 null
// 重试连接池里面使用
.hasExchange():true;

// 快速关闭当前持有的 socket 连接。中断任何执行请求中的的线程。
// 调用处：RealCall.cancel()
.cancel();

// 连接池加锁后，返回调用 cancel() 后修改的标记值
// ExchangeFinder.findConnection、RealCall.isCanceled\getResponseWithInterceptorChain
// RetryAndFollowUpInterceptor.intercept 中用于判断是否抛出 IOException("Canceled")
.isCanceled():true;
```

# Transmitter 流程追踪

## 开始执行

ReaCall.execute

```
transmitter.timeoutEnter();
transmitter.callStart();
```



## 绑定到责任链

RealCall.getResponseWithInterceptorChain

```java
Interceptor.Chain chain = new RealInterceptorChain(interceptors, transmitter, null, 0,
        originalRequest, this, client.connectTimeoutMillis(),
        client.readTimeoutMillis(), client.writeTimeoutMillis());
```



## 准备连接 (RetryAndFollowUpInterceptor)

RetryAndFollowUpInterceptor.intercept()

```java
Transmitter transmitter = realChain.transmitter();
transmitter.prepareToConnect(request);
```

```java
public void prepareToConnect(Request request) {
  if (this.request != null) { // 第一次请求，this.request 为 null
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
  this.exchangeFinder 
    = new ExchangeFinder(this, connectionPool, createAddress(request.url()), call, eventListener);
}
```



## 构造数据交换对象（ConnectInterceptor）

ConnectInterceptor.intercept()

```java
// We need the network to satisfy this request. Possibly for validating a conditional GET.
boolean doExtensiveHealthChecks = !request.method().equals("GET");
Exchange exchange = transmitter.newExchange(chain, doExtensiveHealthChecks);
```

### Exchange 的创建

```java
// transmitter.newExchange
ExchangeCodec codec = exchangeFinder.find(client, chain, doExtensiveHealthChecks);
Exchange result = new Exchange(this/*transmitter*/, call, eventListener, exchangeFinder, codec);

// Exchange 构造器
public Exchange(Transmitter transmitter, Call call, EventListener eventListener,
                ExchangeFinder finder, ExchangeCodec codec) {
  this.transmitter = transmitter;
  this.call = call;
  this.eventListener = eventListener;
  this.finder = finder;
  this.codec = codec;
}
```



