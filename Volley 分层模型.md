# Volley 无缓存的分层模型（4 层）

1. 一个不要求缓存的请求发出，进入**请求队列**

2. NetworkDispather（线程 * 4）从**请求队列**取出请求，调用 Network（BasicNetwork）的 performRequest 获取 NetworkResponse

3. Network（BasicNetwork）的 performRequest 调用 HttpStack（HurlStack\HttpClientStack\自定义 Stack）performRequest 获取 HttpResponse

   > org.apache.http.HttpResponse 已经废弃，实现 HttpStack 接口目的是为了统一返回结果，因为向下兼容的 HttpClient（< Android2.3）（HttpClientStack）也是返回 HttpResponse

4. HttpStack（以 HurlStack 为例）的 performRequest 调用 HttpURLConnection 进行网络请求，结果封装成了 BasicHttpResponse 返回

   > BasicHttpResponse extends AbstractHttpMessage implements HttpResponse

总共是 **4 层模型**

- 4 -> 3 层：返回 HttpResponse 并封装成 NetworkResponse。除了返回 NetworkResponse，在这一层还要负责从第 4 层抛出的异常的捕获，并根据异常，判断是进行重试，继续异常向上抛，封装成 VolleyError 继续向上抛

  > VolleyError extends Exception
  >
  > 子类：NoConnectionError，ServerError，NetworkError，这几个都是因为 IOException
  >
  > 纯异常：
  >
  > - MalformedURLException，错误的 URL
  > - SocketTimeoutException，超时重试后抛出
  > - ConnectTimeoutException，超时重试后抛出
  > - ConnectTimeoutException 和 SocketTimeoutException 区别：C 代表连接超时，即连接都未建立，可以认为是请求超时；而 S 代表的是 socket 读写超时，即连接已经建立了，可以认为是服务器响应超时

- 3 -> 2 层：返回正常 NetworkResponse 到 NetworkDispather，之后会调用 Request 的 parseNetworkResponse 方法，将 NetworkResponse 解析成 Response；如果是捕获到上抛的 VolleyError 或者是纯异常，异常要重新封装成 VolleyError

- 2 -> 1 层：NetworkDispather 的成员变量 ResponseDelivery（ExecutorDelivery）会调用 postResponse 和 postError 把 Response 和 Error 通过 MainHandler post 给主线程。由于只能 post Runnable，会先构造一个 ResponseDeliveryRunnable（包含 Request、Response，postError 的时候 VolleyError 会被加入到 Reponse 里面）

  > new ExecutorDelivery(new Handler(Looper.getMainLooper())

- 1 层：ResponseDeliveryRunnable 的 run 方法被执行，进而 Requst 的 deliverResponse 和 deliverError 方法被执行，进而 Listener 或者 Callback 被最终调用，将 Response 和 VolleyError 传出



# Volley 可缓存的分层模型（5 层）

1. 对于一个需要缓存的请求，会先根据请求的 cacheKey 去 map 中查看是否有一个 List 记录当前中国 cacheKey 的所有请求，没有的首先会以 cacheKey 为 key，null 为 value 加入 map，然后加入**缓存队列**；如果 map 中存在了 key 为 cacheKey 的节点，取出或生成一个 Queue，并把请求加入 Queue。

   > 这个 Queue 的目的是什么？

2. 加入缓存队列后，CacheDispatcher（Thread * 1）从队列取出请求，查看缓存 Cache（DiskBasedCache） 内是否有这个缓存实体，没有实体就加入**请求队列**，于是后面进入了“无缓存的分层模型”内容；

   1. 对于存在缓存实体的请求，先要判断缓存是否过期，过期，请求加入**请求队列**
   2. 对于缓存实体没有过期但却需要刷新的请求，通过 ExecutorDelivery 返回缓存实体构造的 NetworkResponse 到主线程的同时把请求加入**请求队列**
   3. 对于缓存实体没有过期也不需要刷新的请求，通过 ExecutorDelivery 返回缓存实体构造的 NetworkResponse 到主线程

3. NetworkDispather（线程 * 4）从**请求队列**取出请求，调用 Network（BasicNetwork）的 performRequest 获取 NetworkResponse

4. ....

总的来说就是多加了一层，第 1 层和之前的 1 是在同一层的，所以总共 5 层

- 3 -> 2 层：返回正常 NetworkResponse 到 NetworkDispather，之后会调用 Request 的 parseNetworkResponse 方法，这个方法在将 NetworkResponse 解析成 Response 的时候，调用了 HttpHeaderParser.parseCacheHeaders(response) 构造了缓存实体，并通过 Response.success 存入到 Response 里面

- 2-> 1层：接着又是 NetworkDispather 里面ResponseDelivery（ExecutorDelivery）的工作。ResponseDeliveryRunnable 里面除了和之前相同的调用调用 Requst.deliverResponse 还会调用 Request.finish，然后会调用 RequestQueue.finish，然后对于需要进行缓存的请求，就会取出之前 map 里面 cacheKey 对应的 Queue，然后一次性加入到**缓存队列**中



**那么 Queue 的目的是什么？**首先，CacheDispatcher 线程数量是 1，这意味着加入**缓存队列**的请求你就算一次放入多个，它也只能一个一个的去处理，不像处理**请求队列**的 NetworkDispather，可以 4 个任务并发处理。

这也意味着，按顺序加入的实现方式，**缓存队列**里面可能会出现连续几个多个相同的请求，而且要一个一个线性执行（第一个执行完后，后续相同的可能就直接用缓存里面的数据了）。但按这样的实现方式，它其实是加快了不同的请求的执行。eg. AAAA 三个请求 + BBB 两个请求，按这样的就会是 ABAAABB 的执行顺序