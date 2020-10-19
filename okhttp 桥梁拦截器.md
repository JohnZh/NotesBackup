# okhttp 桥梁拦截器

Bridges from application code to network code. First it builds a network request from a user

request. Then it proceeds to call the network. Finally it builds a user response from the network

response.



翻译：

连接应用程序代码和网络代码。首先，从用户请求构建网络请求。然后，继续调用网络。最后，从网络响应构建一个用户响应



解释：

从代码的实现上解释，就是根据 http 的语义里面定义的头信息来填充请求对象，然后发送这个请求后，根据返回的响应和其头信息，做一些转换后再封装成合适的响应返回给上一层拦截器



# 代码分析

BridgeInterceptor.intercept 代码：

```java
Request userRequest = chain.request();
Request.Builder requestBuilder = userRequest.newBuilder();

// 请求体不为 null 的情况，需要定义请求头中 Content-Type 的值
RequestBody body = userRequest.body();
if (body != null) {
  MediaType contentType = body.contentType();
  if (contentType != null) {
    requestBuilder.header("Content-Type", contentType.toString());
  }

  // 请求体不为 null 的情况下，定义 Content-Length 代表内容长度
  // Content-Length 和 Transfer-Encoding 二选一
  long contentLength = body.contentLength();
  if (contentLength != -1) {
    requestBuilder.header("Content-Length", Long.toString(contentLength));
    requestBuilder.removeHeader("Transfer-Encoding");
  } else {
    requestBuilder.header("Transfer-Encoding", "chunked");
    requestBuilder.removeHeader("Content-Length");
  }
}

// HOST 开发者未设置就用 url 里面解析出来
if (userRequest.header("Host") == null) {
  requestBuilder.header("Host", hostHeader(userRequest.url(), false));
}

// 默认是 Connection: Keep-Alive
if (userRequest.header("Connection") == null) {
  requestBuilder.header("Connection", "Keep-Alive");
}

// 默认在为设置 Rang 的情况下，Accept-Encoding: gzip
// If we add an "Accept-Encoding: gzip" header field we're responsible for also decompressing
// the transfer stream.
boolean transparentGzip = false;
if (userRequest.header("Accept-Encoding") == null && userRequest.header("Range") == null) {
  transparentGzip = true;
  requestBuilder.header("Accept-Encoding", "gzip");
}

// cookieJar 默认是 NO_COOKIES 实现，loadForRequest 会返回空 List
List<Cookie> cookies = cookieJar.loadForRequest(userRequest.url());
if (!cookies.isEmpty()) {
  requestBuilder.header("Cookie", cookieHeader(cookies));
}

// User-Agent: okhttp/3.14.7
if (userRequest.header("User-Agent") == null) {
  requestBuilder.header("User-Agent", Version.userAgent());
}

// 调用下个拦截器进行处理
Response networkResponse = chain.proceed(requestBuilder.build());

// 对返回的响应头部进行解析，默认 NO_COOKIES 是不做处理的
// 否则，就会解析头部里面的 Set-Cookie 头的内容成一个 List，然后存入 cookieJar
HttpHeaders.receiveHeaders(cookieJar, userRequest.url(), networkResponse.headers());

// 重新构造一个 Response，newBuilder()方法会把当前获得的响应成员变量都赋值到新的 Response 里面
// 然后把发送出去的请求设置到新的这个 Response 里面
Response.Builder responseBuilder = networkResponse.newBuilder()
    .request(userRequest);

// 在设置了 Accept-Encoding: gzip 之后，如果收到的响应头部里面有 Content-Encoding: gzip
// 并且这通过这个响应头部判断这是有一个有响应实体的响应：
//   请求方法不是 HEAD
//   或者响应码小于 100 大于等于 200，并且不是 204（无内容）和 304（未修改）
//   或者 Content-Length 存在且有值，或者响应头有 Transfer-Encoding: chunked
// 用 gzip 算法解码内容，去掉 Content-Encoding，Content-Length 因为肯定和解码后的不符
if (transparentGzip
    && "gzip".equalsIgnoreCase(networkResponse.header("Content-Encoding"))
    && HttpHeaders.hasBody(networkResponse)) {
  GzipSource responseBody = new GzipSource(networkResponse.body().source());
  Headers strippedHeaders = networkResponse.headers().newBuilder()
      .removeAll("Content-Encoding")
      .removeAll("Content-Length")
      .build();
  responseBuilder.headers(strippedHeaders);
  String contentType = networkResponse.header("Content-Type");
  responseBuilder.body(new RealResponseBody(contentType, -1L, Okio.buffer(responseBody)));
}

return responseBuilder.build();
```





# 总结



## 请求头定义

- Content-Type：请求时告诉服务端数据类型，eg.

```java
Content-Type: text/html; charset=utf-8
Content-Type: application/json; charset=utf-8
Content-Type: application/x-www-form-urlencoded
Content-Type: multipart/form-data; boundary=something
(Content-Type: multipart/form-data; boundary=---------------------------974767299852498929531610575)
```

- Content-Length：和 Transfer-Encoding 二选一，数字字符串，表示数据内容长度

- Transfer-Encoding：chunked（按块传），指明传递数据时候用的编码方式.eg

```
Transfer-Encoding: chunked
Transfer-Encoding: compress
Transfer-Encoding: deflate
Transfer-Encoding: gzip
Transfer-Encoding: identity
```

以上 3 个只在请求体存在才需要填充

- Host：发送到的服务器主机名和端口号（端口号可选）
  - 开发者未设置就会从 url 里面解析出来（一般是域名），不加端口
- Connection: Keep-Alive，HTTP1.1 默认
- Accept-Encoding: gzip，如果开发者没有设置这个头，并且也没有设置 Rang 的情况下
- Cookie：存根，用于记录稳定的状态信息，客户端发服务端用 Cookie，服务端发客户端用 Set-Cookie

```
GET /sample_page.html HTTP/1.1
Host: www.example.org
Cookie: yummy_cookie=choco; tasty_cookie=strawberry

HTTP/1.0 200 OK
Content-type: text/html
Set-Cookie: yummy_cookie=choco
Set-Cookie: tasty_cookie=strawberry
[页面内容]
```

- User-Agent：让对端来识别发起请求的用户代理软件的应用类型、操作系统、软件开发商以及版本号

## 响应头处理

只对下面的几个头进行了特殊处理

- Set-Cookie：Cookie 相关，需要自定义，否则默认不处理

- Content-Encoding: gzip，说明数据又进行压缩编码，前提是请求发送时候也设置了 Accept-Encoding: gzip

