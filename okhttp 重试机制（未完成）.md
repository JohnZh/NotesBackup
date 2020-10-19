# okhttp 重试机制

负责这个的拦截器：RetryAndFollowUpInterceptor



# 代码分析

RetryAndFollowUpInterceptor.intercept 代码：

```java
Request request = chain.request();
RealInterceptorChain realChain = (RealInterceptorChain) chain;
Transmitter transmitter = realChain.transmitter();

int followUpCount = 0;
Response priorResponse = null;
while (true) {
  transmitter.prepareToConnect(request);

  if (transmitter.isCanceled()) {
    throw new IOException("Canceled");
  }

  Response response;
  boolean success = false;
  try {
    // 未捕获到任何异常，说明请求成功，标记值修改
    response = realChain.proceed(request, transmitter, null);
    success = true;
  } catch (RouteException e) {
    // 路由异常，路由失败，5 种情况会抛出路由异常：
    // 1. 
    // 2. 
    // 3. 
    // 4. 
    // 5.
    // The attempt to connect via a route failed. The request will not have been sent.
    if (!recover(e.getLastConnectException(), transmitter, false, request)) {
      throw e.getFirstConnectException();
    }
    continue;
  } catch (IOException e) {
    // An attempt to communicate with a server failed. The request may have been sent.
    // 如果是是连接关闭异常（ConnectionShutdownException），说明请求发生还没开始
    boolean requestSendStarted = !(e instanceof ConnectionShutdownException);
    if (!recover(e, transmitter, requestSendStarted, request)) throw e;
    continue;
  } finally {
    
    // 发生了异常，且不能恢复，继续向上抛出异常后，还会到这来执行释放资源代码
    // The network call threw an exception. Release any resources.
    if (!success) {
      transmitter.exchangeDoneDueToException();
    }
  }

  // 第一次 priorResponse 是 null，最后会进行 priorResponse = response
  // 将前一次的响应体改 null 后再把前一次响应设到当前响应的 priorResponse 变量上
  // newBuilder() 会执行 new Builder(this)，把当前响应全部变量都赋值给新建的 builder
  // 然后 build() 会执行 new Response(this)，重新创建一个响应把 builder 里的变量再赋值个新的响应
  // 因此 response 对象的内容（变量，不包括 body）虽然没变，但是对象并不是同一个
  //
  // Attach the prior response if it exists. Such responses never have a body.
  if (priorResponse != null) {
    response = response.newBuilder()
        .priorResponse(priorResponse.newBuilder()
                .body(null)
                .build())
        .build();
  }

  // 返回的就是 response.exchange	
  Exchange exchange = Internal.instance.exchange(response);
  Route route = exchange != null ? exchange.connection().route() : null;
  Request followUp = followUpRequest(response, route);

  if (followUp == null) {
    if (exchange != null && exchange.isDuplex()) {
      transmitter.timeoutEarlyExit();
    }
    return response;
  }

  RequestBody followUpBody = followUp.body();
  if (followUpBody != null && followUpBody.isOneShot()) {
    return response;
  }

  closeQuietly(response.body());
  if (transmitter.hasExchange()) {
    exchange.detachWithViolence();
  }

  if (++followUpCount > MAX_FOLLOW_UPS) {
    throw new ProtocolException("Too many follow-up requests: " + followUpCount);
  }

  request = followUp;
  priorResponse = response;
}
```

发生路由异常（RouteException）和 IO 异常（IOException）的情况下，判断是否能恢复的 recover 方法。如果能恢复，返回 true，循环 continue，再发送一次请求；不能恢复，返回 false，throw 异常，这次请求就结束了

```java
private boolean recover(IOException e, Transmitter transmitter,
                        boolean requestSendStarted, Request userRequest) {
  // 下面返回 false 的情况是不需要恢复的情况，
  // 如果是路由异常（RouteException）requestSendStarted 为 false
  // 如果是连接关闭异常（ConnectionShutdownException）requestSendStarted 为 false
  
  // 应用层，即开发者配置了不要失败重连
  // The application layer has forbidden retries.
  if (!client.retryOnConnectionFailure()) return false;

  // 请求已经发出了，并且请求是 oneshot 请求（请求体是 oneshot，或者异常是 FileNotFoundException）
  // We can't send the request body again.
  if (requestSendStarted && requestIsOneShot(e, userRequest)) return false;

  // 不可恢复的情况：
  // 
  // 协议异常（ProtocolException）无法恢复
  // 
  // 如果是中断异常（InterruptedIOException），且是异常是 Socket 超时（SocketTimeoutException）
  // 并且请求发送还未开始（requestSendStarted 为 false）
  // 那么，这种中断异常还可以恢复。否则，不能恢复
  // 
  // SSL 握手异常（SSLHandshakeException）并且原因是证书问题（CertificateException）
  // 那么，这个情况也无法恢复
  // 
  // SSL 端未证实异常（SSLPeerUnverifiedException） 无法恢复
  //
  // This exception is fatal.
  if (!isRecoverable(e, requestSendStarted)) return false;

  // 发射器无法重试，也会导致无法恢复，原因是什么？
  // No more routes to attempt.
  if (!transmitter.canRetry()) return false;

  // For failure recovery, use the same route selector with a new connection.
  return true;
}
```

