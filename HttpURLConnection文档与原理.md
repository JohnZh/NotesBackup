HttpUrlConnection 文档与原理



# 使用

1. 通过 [URL#openConnection()](./URL.html#openConnection()) 获取一个新的 HttpUrlConnection，需要转换类型
2. 准备请求（Request），主要属性是 URL。请求头可以包含多种元数据，例如证书，内容类型（Content-Type），会话存根（Session Cookies）
3. 可选。请求体。需要调用 [setDoOutput(true)](./URLConnection.html#setDoOutput(boolean))。发送数据是通过 [URLConnection.getOutputStream()](./URLConnection.html#getOutputStream()) 返回的输出流来写出
4. 读响应（Response）。响应头通常包含的元数据例如响应体内容类型，长度，修改的时间，会话存根。响应实体的内容可以从 [URLConnection.getInputStream()](./URLConnection.html#getInputStream()) 的输入流读取出来。如果没有响应实体，返回空流
5. 断开。一旦响应实体内容读完，HttpUrlConnection 应该被关闭，通过 [disconnect()](./HttpURLConnection.html#disconnect()) 。断开操作会释放被连接持有的资源，这些资源会被关闭或者重用

## 例子

```java
URL url = new URL("http://www.android.com/");
HttpURLConnection urlConnection = (HttpURLConnection) url.openConnection();
try {
  InputStream in = new BufferedInputStream(urlConnection.getInputStream());
  readStream(in);
} finally {
  urlConnection.disconnect();
}
```

## https

使用 [URL#openConnection()](./URL.html#openConnection()) 访问 https 的 url会返回一个 **HttpsURLConnection**，它允许覆写默认的  [HostnameVerifier](../../javax/net/ssl/HostnameVerifier.html)和 [SSLSocketFactory](../../javax/net/ssl/SSLSocketFactory.html)。一个应用在程序级的 SSLSocketFactory 是从 [SSLContext](../../javax/net/ssl/SSLContext.html) 创建的，它能提供一个自定义的 [X509TrustManager](../../javax/net/ssl/X509TrustManager.html) 用于验证服务器证书链，同时也提供一个自定义的 [X509KeyManager](../../javax/net/ssl/X509KeyManager.html) 用于提供用户证书。更多内容看 [HttpsURLConnection](../../javax/net/ssl/HttpsURLConnection.html)



## 响应处理

HttpUrlConnection 会追踪 5 个 http 重定向，它会追踪重定向从一个源服务器到另一个。但是它实现上不会追踪 Https 到 Http 的重定向，也不会追踪 Http 到 Https 的重定向。

如果 Http 响应有错误产生，[URLConnection.getInputStream()](./URLConnection.html#getInputStream()) 会抛出一个 [IOException](../io/IOException.html)。用 [getErrorStream()](./HttpURLConnection.html#getErrorStream()) 读取错误的响应结果。响应头还是可以用正常的方式读出 [URLConnection.getHeaderFields()](./URLConnection.html#getHeaderFields())。



## POST 内容

上传数据到服务器，配置 [setDoOutput(true)](./URLConnection.html#setDoOutput(boolean))

为了最好的性能，如果数据体长度已知应该使用 [setFixedLengthStreamingMode(int)](./HttpURLConnection.html#setFixedLengthStreamingMode(int)) 或者未知数据体长度应该使用 [setChunkedStreamingMode(int)](./HttpURLConnection.html#setChunkedStreamingMode(int))。不然的话，HttpURLConnection 会被强制缓冲请求体（Request body）到内存（在请求内容被发送前），这会浪费堆（有可能耗尽）和增加延时



### 举例

```java
HttpURLConnection urlConnection = (HttpURLConnection) url.openConnection();
try {
  urlConnection.setDoOutput(true);
  urlConnection.setChunkedStreamingMode(0);

  OutputStream out = new BufferedOutputStream(urlConnection.getOutputStream());
  writeStream(out);

  InputStream in = new BufferedInputStream(urlConnection.getInputStream());
  readStream(in);
} finally {
  urlConnection.disconnect();
}
```



## 性能

HttpUrlConnection 返回的输入和输出流是无缓冲（**not buffered**）的。因此大部分的调用应该使用[BufferedInputStream](../io/BufferedInputStream.html) 和 [BufferedOutputStream](../io/BufferedOutputStream.html) 包裹返回的流。调用只有在批量读写时也许会忽略缓冲。



从服务器传送或接收大量数据的时候，使用流来限制一次接收或发送的数据量。除非你需要内存里有完整的实体，不然都以流的方式处理它（而不是存储完整的单个字节数组 bytes array 或字符串）



为了减少延时，HttpUrlConnection 会复用相同的底层 socket，用于多对 Request/Response。因此 Http 连接也许会保持打开状态长于必要的时间。调用 [disconnect()](./HttpURLConnection.html#disconnect()) 也许会将 socket 还回已连接的 socket（connected socket）的池里



默认情况，HttpURLConnection 的请求的实现上，服务会使用 gzip压缩，[URLConnection.getInputStream()](./URLConnection.html#getInputStream()) 的调用会自动的解压数据。 Content-Encoding 和 Content-Length 响应头在这个例子中会被清除。Gzip压缩可以通过在请求头里设置“可以接受编码”的方式来使其失效

```java
urlConnection.setRequestProperty("Accept-Encoding", "identity");
```

设置 Accept-Encoding 请求头会显式地失效自动解压功能，同时保留完整的响应头部。调用者必须根据需求和  Content-Encoding 响应头去进行解压操作



[URLConnection.getContentLength()](./URLConnection.html#getContentLength()) 返回传输的字节数，但不能被用于预估可以从 [URLConnection.getInputStream()](./URLConnection.html#getInputStream()) 读出的压缩流里有多少字节。正确的做法，读取流直到其耗尽，例如当 [InputStream#read](../io/InputStream.html#read()) 返回 -1



## 处理需要登录的网络

有一些 wifi 网络会阻止网络访问直到用户点击通过一个登录页面。这样的登录页面通常情况是通过 HTTP 重定向实现的。你可以使用 [URLConnection.getURL()](./URLConnection.html#getURL()) 来测试是否你的连接已经被非期望的重定向了。这个检测需要在响应头已经被接收到后才会生效，你可以通过调用 [URLConnection.getHeaderFields()](./URLConnection.html#getHeaderFields()) 或者 [URLConnection.getInputStream()](./URLConnection.html#getInputStream()) 来触发响应头的接收。下面例子展示了响应是否被重定向了非期望的 host：

```java
HttpURLConnection urlConnection = (HttpURLConnection) url.openConnection();
  try {
  InputStream in = new BufferedInputStream(urlConnection.getInputStream());
  if (!url.getHost().equals(urlConnection.getURL().getHost())) {
  // we were redirected! Kick the user out to the browser to sign on?
  }
  ...
  } finally {
  urlConnection.disconnect();
}
```



## Http 认证

HttpURLConnection 支持 HTTP 基础认证（ [HTTP basic authentication](http://www.ietf.org/rfc/rfc2617)）。使用 [Authenticator](./Authenticator.html) 来设置虚拟机级的认证处理器

```java
Authenticator.setDefault(new Authenticator() {
  protected PasswordAuthentication getPasswordAuthentication() {
    return new PasswordAuthentication(username, password.toCharArray());
  }
});
```

除非配合https，不然这也不是用户认证的安全机制



## Sessions with Cookies

为了在客户端和服务器之间建立和维护一个潜在的长活的会话，HttpURLConnection 包含了一个可扩展的 Cookie  manager。启动 VM 级别的 Cookie 管理请使用 [CookieHandler](./CookieHandler.html) 和 [CookieManager](./CookieManager.html)

```java
CookieManager cookieManager = new CookieManager();
CookieHandler.setDefault(cookieManager);
```

默认情况，CookieManager 只接受来自源服务器（[origin server](http://www.w3.org/Protocols/rfc2616/rfc2616-sec1.html) ）的 Cookies。另外两个策略，[CookiePolicy#ACCEPT_ALL](./CookiePolicy.html#ACCEPT_ALL) 和 [CookiePolicy#ACCEPT_NONE](./CookiePolicy.html#ACCEPT_NONE)，也可以实现 [CookiePolicy](./CookiePolicy.html) 实现自己的策略



默认的 CookieManager 保存所有接受的 cookies 在内存里。当 VM 退出的时候，这些 cookies 会被遗忘（消失）。用 [CookieStore](./CookieStore.html) 定义自定义 cookie store。



除了HTTP Response 设置的 cookies 外，你可以自己编码设置 cookies。为了把 cookies 包含在HTTP 请求头里，cookies 必须有域名和路径属性的设置



默认情况，新的 HttpCookie 对象只能和支持 [RFC 2965](http://www.ietf.org/rfc/rfc2965.txt) cookies 的服务一起工作。许多 web 服务只支持老的规范 [RFC 2109](http://www.ietf.org/rfc/rfc2109.txt)。为了和大部分 web 服务兼容，cookie verison 要设置为 0。例如为了接收法语的 www.twitter.com

```java
HttpCookie cookie = new HttpCookie("lang", "fr");
cookie.setDomain("twitter.com");
cookie.setPath("/");
cookie.setVersion(0);
cookieManager.getCookieStore().add(new URI("http://twitter.com/"), cookie);
```



## HTTP 方法

HttpUrlConnection 默认使用 GET。 如果已经设置了 [setDoOutput(true)](./URLConnection.html#setDoOutput(boolean)) 会使用 POST。通过[setRequestMethod(String)](./HttpURLConnection.html#setRequestMethod(java.lang.String)) 可以设置其他的 HTTP 方法（OPTIONS，HEAD，PUT，DELETE 以及 TRACE）



## 代理

默认情况，HttpURLConnection 会直接连接到源服务。它也可以通过 [Proxy.Type#HTTP](./Proxy.Type.html#HTTP) 或者 [Proxy.Type#SOCKS](./Proxy.Type.html#SOCKS) 代理进行连接。为了使用代理，创建连接的时候使用 [URL#openConnection(Proxy)](./URL.html#openConnection(java.net.Proxy))



## IPv6 支持

HttpURLConnection 包含对 IPv6 透明的支持。对于同时包含IPv4 和 IPv6 地址的主机，它会尝试连接每个主机地址，直到连接建立



## 响应缓存

Android 4.0 (Ice Cream Sandwich, API level 15) 包含响应缓存，关于在你的 app 里面建立 HTTP 缓存的说明参考 `android.net.http.HttpResponseCache` 





# 原理

## 获取 HttpURLConnection

构建 URL 通过 openConnection 获取 HttpURLConnection 对象。

### URL 

对传入的 url 进行二次处理（比如去除头尾不可见字符），解析出协议，并通过协议创建 URLStreamHandler

#### URLStreamHandler 的创建

首先，会从系统属性（java.protocol.handler.pkgs）里面拿一批包前缀，这些前缀是用 ‘|’ 分隔的，然后遍历这些前缀，构造 **前缀 + 协议字符串（protocol）+ ".Handler"** 的字符串，然后使用系统类加载器加载，如果加载到 Class，再用 newInstance 方法调用默认构造器来实例化 URLStreamHandler 对象

不过实际上，上面系统属性拿出来是空的，于是会调用 **createBuiltinHandler(protocol)**

```java
private static URLStreamHandler createBuiltinHandler(String protocol)
        throws ClassNotFoundException, InstantiationException, IllegalAccessException {
    URLStreamHandler handler = null;
    if (protocol.equals("file")) {
        handler = new sun.net.www.protocol.file.Handler();
    } else if (protocol.equals("ftp")) {
        handler = new sun.net.www.protocol.ftp.Handler();
    } else if (protocol.equals("jar")) {
        handler = new sun.net.www.protocol.jar.Handler();
    } else if (protocol.equals("http")) {
        handler = (URLStreamHandler)Class.
                forName("com.android.okhttp.HttpHandler").newInstance();
    } else if (protocol.equals("https")) {
        handler = (URLStreamHandler)Class.
                forName("com.android.okhttp.HttpsHandler").newInstance();
    }
    return handler;
}
```

http/https 使用的是最后两个类。Android 4.4 开始 HttpURLConnection 底层实现替换成了 okhttp，这里的包名使用了 jarjar 替换了

> jarjar 网站：https://code.google.com/archive/p/jarjar
>
> 原理：使用 Ant 任务使用通配符匹配方式替换类名和打包，使用 ASM 用字节码转换来改变这些被重命名的类的引用，为移动资源文件和转换字面量提供特殊处理
>
> 如何使用：
>
> https://github.com/pantsbuild/jarjar/blob/master/src/main/java/org/pantsbuild/jarjar/help.txt
>
> **替换包名的功能**：
>
>   java -jar jarjar.jar process <rulesFile> <inJar> <outJar>  
>
> ```java
> // external/okhttp/jarjar-rules.txt,
> rule com.squareup.** com.android.@1
> rule okio.** com.android.okio.@1
> ```

项目地址：https://android.googlesource.com/platform/external/okhttp/

路径：/android/src/main/java/com/squareup/okhttp

类：HttpHandler.java，HttpsHandler.java



### Url.openConnection

实际上是调用了 URLStreamHandler.openConnection(this)，即 

```java
HttpHandler.openConnection(URL url)
HttpsHandler.openConnection(URL url)
```

#### HttpHandler.openConnection

实际调用

```java
newOkUrlFactory(null /* proxy */).open(Url url)
```

OkUrlFactory  的创建调用栈：

```java
// HttpHandler.java
newOkUrlFactory(null /* proxy */)
	createHttpOkUrlFactory(proxy)	
		OkHttpClient client = new OkHttpClient();
		// 配置 OkHttpClient
		client.setConnectTimeout(0, TimeUnit.MILLISECONDS);
		client.setReadTimeout(0, TimeUnit.MILLISECONDS);
		client.setWriteTimeout(0, TimeUnit.MILLISECONDS);
		client.setFollowRedirects(HttpURLConnection.getFollowRedirects());
    client.setFollowSslRedirects(false);
    client.setConnectionSpecs(CLEARTEXT_ONLY);
    if (proxy != null) {
      client.setProxy(proxy);
		}
		
		OkUrlFactory okUrlFactory = new OkUrlFactory(client);
		return okUrlFactory;
```

OkUrlFactory.open(URl url) 调用栈：

```java
open(URL url)
	open(url, client.getProxy()) {
    String protocol = url.getProtocol();
    OkHttpClient copy = client.copyWithDefaults();
    copy.setProxy(proxy);

    if (protocol.equals("http")) return new HttpURLConnectionImpl(url, copy, urlFilter);
    if (protocol.equals("https")) return new HttpsURLConnectionImpl(url, copy, urlFilter);
    throw new IllegalArgumentException("Unexpected protocol: " + protocol);
  }	
```

HttpURLConnection 的实际实现就是

```java
HttpURLConnectionImpl.java
HttpsURLConnectionImpl.java
```

#### HttpsHandler.openConnection

HttpsHandler 继承 HttpHandler 但没有自己实现 openConnection，但是实现了 newOkUrlFactory

```java
newOkUrlFactory(null /* proxy */).open(url)

newOkUrlFactory(null /* proxy */)
	createHttpsOkUrlFactory(proxy)
  	OkUrlFactory okUrlFactory = HttpHandler.createHttpOkUrlFactory(proxy);

		OkHttpClient okHttpClient = okUrlFactory.client();
		// 配置 okhttp 支持 okhttps
		okHttpClient.setProtocols(HTTP_1_1_ONLY);
		okHttpClient.setConnectionSpecs(Collections.singletonList(TLS_CONNECTION_SPEC));
		okHttpClient.setCertificatePinner(CertificatePinner.DEFAULT);
		okHttpClient.setHostnameVerifier(HttpsURLConnection.getDefaultHostnameVerifier());
		okHttpClient.setSslSocketFactory(HttpsURLConnection.getDefaultSSLSocketFactory());

		return okUrlFactory
```

那么最后还是调用了 **OkUrlFactory.open(URl url)**，那么最后的http/https 的实现还是：

HttpURLConnectionImpl/HttpsURLConnectionImpl	

```java
HttpURLConnectionImpl extends HttpURLConnection extends URLConnectiona
```

```java
HttpsURLConnectionImpl extends DelegatingHttpsURLConnection extends HttpsURLConnection 					extends HttpURLConnection extends URLConnection {

  private final HttpURLConnectionImpl delegate;
  
  public HttpsURLConnectionImpl(URL url, OkHttpClient client, URLFilter filter) {
    this(new HttpURLConnectionImpl(url, client, filter));
  }
  
  public HttpsURLConnectionImpl(HttpURLConnectionImpl delegate) {
    super(delegate);
    this.delegate = delegate;
  }
}
```

```java
DelegatingHttpsURLConnection extends HttpsURLConnection {
	private final HttpURLConnection delegate;

  public DelegatingHttpsURLConnection(HttpURLConnection delegate) {
    super(delegate.getURL());
    this.delegate = delegate;
  }
}
```



存档：https://www.jianshu.com/p/35ecbc09c160

## HttpURLConnectionImpl





## HttpsURLConnectionImpl