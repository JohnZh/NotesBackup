# HTTP（超文本传输协议）碎片记录

超文本传输协议：Hyper-Text Transfer Protocol 

# 历史版本
- HTTP 0.9：
	- 只有 GET 请求
	- 不支持消息报头
	- 只能返回 HTML 超文本
	- 无状态，每个事务建立单独的 TCP 连接，完成响应后关闭连接
- HTTP 1.0：对比 HTTP 0.9 
	- 增加了消息报头
	- 除了支持 GET 还支持 HEAD 和 POST
	- 添加了响应对象的状态行
	- 响应对象不限于 HTML 超文本
	- 支持长连接（默认短连接）（使用报头非标准字段 Connection:keep-alive）
	- 其他：缓存机制，认证机制，字符集支持，内容编码
	- 遗留问题：
		- 短连接机制，一个响应网页如果存在多个超连接资源，后续请求又需要进行重连接，增加了开销
		- [HOL Blocking](#head_of_line_blocking)
- HTTP 1.1：对比 HTTP 1.0
	- 加入除 GET/POST/HEAD 外的新方法。PUT/DELETE/OPTIONS/TRACE/CONNECT
	- 默认长连接（Connection:keep-alive，主动关闭值为 close）和 [Pipelining 管线化](#pipelining)
	- 范围请求。相关报头字段 Rang。只请求对象的部分。这也是断点续传的基础
	- 支持虚拟主机。相关报头字段 Host。
	- 添加了新的缓存的和身份认证的报头字段。eg. e-tags
	- 新的响应状态码。eg. 409（Conflict）资源状态冲突，410（Gone）资源永久删除
	- 遗留问题：
		- Pipelining 虽然可以多个请求在同一次 TCP 连接里面，但是响应的返回依然是有序的，因此 [HOL Blocking](#head_of_line_blocking) 依然存在
- HTTP 2：对比 HTTP 1.1
	- HTTP 1.1 的传输中 HEADER 是文本，BODY DATA可以是文本也可以二进制数据。而HTTP 2 完全采用了二进制的方式：
		- 帧，传输的最小单位，所有的控制，请求，响应，数据都会被包装成帧的，例如替代原先 HEADER 和 BODY 的 HEDAER Frame 和 DATA Frame。所有的帧都有一个固定数据结构 9 bytes 的头部，用于定义内容长度，类型，标示，Stream-id，保留位。然后再是二进制的具体传输内容 Payload
		- 流，只是个逻辑概念，代表客户端和服务端之间数据交换期间的双向帧序列。每个帧的 Stream-id 代表这个帧属于哪个流，待接收方得到全部归属某个于 Stream-id 的帧后重新组装成完整数据
	- 这种**二进制·分帧·流**的形式使得 HTTP 2 的传输中多了 Multiplex 的特性：多路复用（这里的多路其实是指多流）。一条 tcp 线路中，多个请求的发出无需等待最早的请求响应就可以收到后期请求的响应，这也解决了 [HOL Blocking](#head_of_line_blocking) 问题
	- 此外，HTTP 1.1 时代，很多客户端，浏览器为了提升 bandwidth 的利用率，对同一个域名会开启 6~8 个 TCP 连接。CPU 和内存的浪费显而易见，此外还有 DNS 查询，TCP三次握手，TCP 通道慢启动（slow start），通道间自竞争（self-competition）这些都是不能忽视的。而 HTTP 2 时代，由于其特性，对一个域名我们只会开启一个 TCP 连接，甚至在发现两个域名的 IP Address 一样后还会使用相同的连接
	- HEADER 压缩。HPACK。
	- 此外还加了流量窗口（流量控制），服务器推送，请求优先级，连接重置等特性

> 相关见[《HTTP 2.0 碎片记录：帧数据结构，流》](#)

<a id="pipelining"/>
## Pipelining 管线化
HTTP pipelining。将多个HTTP请求整批提交的技术，而在发送过程中不需先等待服务器的回应。但是服务器返回响应依然是 FIFO，因此 [HOL Blocking](#head_of_line_blocking) 依然存在。
		
<a id="head_of_line_blocking"/>	
## HOL Blocking
Head-of-line Blocking 队头阻塞。一列的数据包，第一个数据包（队头）受阻而导致整列数据包受阻。HTTP 1 中，请求-响应都是有序的，因此第一个请求的响应后才能进行第二个请求，HTTP 1.1 的管道化中，请求可以整批发出，但是服务器返回响应依然是 FIFO，这两个情况下，都无法避免第一个请求的响应延时导致后面响应都延时的情况。


# HTTP URL

URL是一种特殊类型的URI，包含了用于查找某个资源的足够的信息，其格式如下：

- http://host[":"port][abs_path]
	- http 表示要通过HTTP协议来定位网络资源；
	- host 表示合法的 Internet 主机域名或者 IP 地址；
	- port 指定一个端口号，为空则使用缺省端口 80；
	- abs_path 指定请求资源的 URI；（如果 URL 中没有给出 abs_path，那么当它作为请求 URI 时，必须以“/”的形式给出，通常这个工作浏览器自动帮我们完成）

```
1、输入：www.guet.edu.cn
浏览器自动转换成：http://www.guet.edu.cn/
2、http:192.168.0.116:8080/index.jsp 
```

# HTTP 1.1 Request 请求
http 请求由三部分组成，分别是：请求行、消息报头、请求正文

- in short: 行 + 头 + 正文


## 行（Line）
请求行以一个方法符号开头，以空格分开，后面跟着请求的 URI 和协议的版本：

**Method Request-URI HTTP-Version CRLF**

- Method 表示请求方法
- Request-URI 是一个统一资源标识符
- HTTP-Version 表示请求的 HTTP 协议版本
- CRLF 表示回车（\r）和换行（\n）（除了作为结尾的 CRLF 外，不允许出现单独的 CR 或 LF 字符）

```
GET /form.html HTTP/1.1 (CRLF)
```


### Method 请求方法

- GET     请求获取 Request-URI 所标识的资源
- POST    在 Request-URI 所标识的资源后附加新的数据
- HEAD    请求获取由 Request-URI 所标识的资源的响应消息报头
- PUT     请求服务器存储一个资源，并用 Request-URI 作为其标识
- DELETE  请求服务器删除 Request-URI 所标识的资源
- TRACE   请求服务器回送收到的请求信息，主要用于测试或诊断
- CONNECT 保留将来使用
- OPTIONS 请求查询服务器的性能，或者查询与资源相关的选项和需求

```
// POST 方法要求被请求服务器接受附在请求后面的数据，常用于提交表单

POST /reg.jsp HTTP/ (CRLF)
Accept:image/gif,image/x-xbit,... (CRLF)
...
HOST:www.guet.edu.cn (CRLF)
Content-Length:22 (CRLF)
Connection:Keep-Alive (CRLF)
Cache-Control:no-cache (CRLF)
(CRLF)         //该CRLF表示消息报头已经结束，在此之前为消息报头
user=jeffrey&pwd=1234  //此行以下为提交的数据

```

```
// HEAD

HEAD 方法与 GET 方法几乎是一样的，对于 HEAD 请求的回应部分来说，它的HTTP头部中包含的信息与通过 GET 请求所得到的信息是相同的。
利用这个方法，不必传输整个资源内容，就可以得到 Request-URI 所标识的资源的信息。该方法常用于测试超链接的有效性，是否可以访问，以及最近是否更新。

```

## 消息报头见 [HTTP Header](#header)

## 请求正文不展开

# HTTP 1.1 Response 响应
在接收和解释请求消息后，服务器返回一个HTTP 响应消息。

HTTP响应也是由三个部分组成，分别是：状态行 + 消息报头 + 响应正文

## 状态行

格式：**HTTP-Version Status-Code Reason-Phrase CRLF**

- HTTP-Version 表示服务器 HTTP 协议的版本
- Status-Code 表示服务器发回的响应状态代码
- Reason-Phrase 表示状态代码的文本描述

```
HTTP/1.1 200 OK （CRLF）
```

### 状态代码

有三位数字组成，第一个数字定义了响应的类别，且有五种可能取值：

- 1xx：指示信息--表示请求已接收，继续处理
- 2xx：成功--表示请求已被成功接收、理解、接受
- 3xx：重定向--要完成请求必须进行更进一步的操作
- 4xx：客户端错误--请求有语法错误或请求无法实现
- 5xx：服务器端错误--服务器未能实现合法的请求

常见状态代码、状态描述、说明：

```
200 OK      //客户端请求成功
400 Bad Request  //客户端请求有语法错误，不能被服务器所理解
401 Unauthorized //请求未经授权，这个状态代码必须和 WWW-Authenticate 报头域一起使用 
403 Forbidden  //服务器收到请求，但是拒绝提供服务
404 Not Found  //请求资源不存在，eg. 输入了错误的 URL
500 Internal Server Error //服务器发生不可预期的错误
503 Server Unavailable  //服务器当前不能处理客户端的请求，一段时间后可能恢复正常
```

## 消息报头见 [HTTP Header](#header)

## 响应正文即服务器返回的内容，不展开

<a id="header" />

# HTTP 1.1 Header 消息报头

HTTP 消息报头包括**普通报头、请求报头、响应报头、实体报头**

每一个报头域都是由 **[名字 + ":" + 空格 + 值]** 组成，消息报头域的名字是大小写无关的

## 普通报头
在普通报头中，有少数报头域用于所有的请求和响应消息，但并不用于被传输的实体，只用于传输的消息。

- Cache-Control 用于指定缓存指令，缓存指令是单向的（响应中出现的缓存指令在请求中未必会出现），且是独立的（一个消息的缓存指令不会影响另一个消息处理的缓存机制），HTTP 1.0 使用的类似的报头域为 Pragma。

>请求时的缓存指令包括：
>no-cache（用于指示请求或响应消息不能缓存）、no-store、max-age、max-stale、min-fresh、only-if-cached;

>响应时的缓存指令包括：
>public、private、no-cache、no-store、no-transform、must-revalidate、proxy-revalidate、max-age、s-maxage.

```
// 为了指示IE浏览器（客户端）不要缓存页面，服务器端的JSP程序可以编写如下：

response.sehHeader("Cache-Control","no-cache");
// response.setHeader("Pragma","no-cache"); 作用相当于上述代码，通常两者合用
```
这句代码将在发送的响应消息中设置普通报头域：Cache-Control:no-cache

- Date 普通报头域表示消息产生的日期和时间
- Connection 普通报头域允许发送指定连接的选项。例如指定连接是连续，或者指定“close”选项，通知服务器，在响应完成后，关闭连接


## 请求报头
请求报头允许客户端向服务器端传递请求的附加信息以及客户端自身的信息
。常用的请求报头:

- Accept 用于指定客户端接受哪些类型的信息。eg. Accept：image/gif，表明客户端希望接受GIF图象格式的资源；Accept：text/html，表明客户端希望接受html文本。

- Accept-Charset 用于指定客户端接受的字符集。eg. Accept-Charset:iso-8859-1,gb2312.如果在请求消息中没有设置这个域，缺省是任何字符集都可以接受。

- Accept-Encoding 类似于 Accept，但是它是用于指定可接受的内容编码。eg. Accept-Encoding:gzip.deflate.如果请求消息中没有设置这个域服务器假定客户端对各种内容编码都可以接受。

- Accept-Language
类似于 Accept，但是它是用于指定一种自然语言。eg. Accept-Language:zh-cn。如果请求消息中没有设置这个报头域，服务器假定客户端对各种语言都可以接受。

- Authorization 主要用于证明客户端有权查看某个资源。当浏览器访问一个页面时，如果收到服务器的响应代码为401（未授权），可以发送一个包含 Authorization 请求报头域的请求，要求服务器对其进行验证。

- Host（发送请求时，该报头域是必需的）
Host请求报头域主要用于指定被请求资源的Internet主机和端口号，它通常从HTTP URL中提取出来的，eg.
我们在浏览器中输入：http://www.guet.edu.cn/index.html
浏览器发送的请求消息中，就会包含Host请求报头域，如下：
Host：www.guet.edu.cn
此处使用缺省端口号80，若指定了端口号，则变成：Host：www.guet.edu.cn:指定端口号


- User-Agent
我们上网登陆论坛的时候，往往会看到一些欢迎信息，其中列出了你的操作系统的名称和版本，你所使用的浏览器的名称和版本，这往往让很多人感到很神奇，实际上，服务器应用程序就是从User-Agent这个请求报头域中获取到这些信息。User-Agent请求报头域允许客户端将它的操作系统、浏览器和其它属性告诉服务器。不过，这个报头域不是必需的，如果我们自己编写一个浏览器，不使用User-Agent请求报头域，那么服务器端就无法得知我们的信息了。

```
请求报头举例：

GET /form.html HTTP/1.1 (CRLF)
Accept:image/gif,image/x-xbitmap,image/jpeg,application/x-shockwave-flash,application/vnd.ms-excel,application/vnd.ms-powerpoint,application/msword,*/* (CRLF)
Accept-Language:zh-cn (CRLF)
Accept-Encoding:gzip,deflate (CRLF)
If-Modified-Since:Wed,05 Jan 2007 11:21:25 GMT (CRLF)
If-None-Match:W/"80b1a4c018f3c41:8317" (CRLF)
User-Agent:Mozilla/4.0(compatible;MSIE6.0;Windows NT 5.0) (CRLF)
Host:www.guet.edu.cn (CRLF)
Connection:Keep-Alive (CRLF)
(CRLF)

```

## 响应报头

- 响应报头允许服务器传递不能放在状态行中的附加响应信息

- 以及关于服务器的信息和对 Request-URI 所标识的资源进行下一步访问的信息。

常用的响应报头

- Location 响应报头域用于重定向接受者到一个新的位置。Location 响应报头域常用在更换域名的时候。

- Server 响应报头域包含了服务器用来处理请求的软件信息。与 User-Agent 请求报头域是相对应的。下面是 Server 响应报头域的一个例子：
Server：Apache-Coyote/1.1

- WWW-Authenticate 响应报头域必须被包含在 401（未授权的）响应消息中，客户端收到401 响应消息时候，并发送 Authorization 报头域请求服务器对其进行验证时，服务端响应报头就包含该报头域。eg. WWW-Authenticate:Basic realm="Basic Auth"


## 实体报头

请求和响应消息都可以传送一个实体。

一个实体由实体报头域和实体正文组成，但并不是说实体报头域和实体正文要在一起发送，可以只发送实体报头域。

实体报头定义了关于实体正文（eg. 有无实体正文）和请求所标识的资源的元信息。

常用的实体报头：

- Content-Encoding 实体报头域被用作媒体类型的修饰符，它的值指示了已经被应用到实体正文的附加内容的编码，因而要获得 Content-Type 报头域中所引用的媒体类型，必须采用相应的解码机制。Content-Encoding 这样用于记录文档的压缩方法，eg. Content-Encoding：gzip

- Content-Language 实体报头域描述了资源所用的自然语言。没有设置该域则认为实体内容将提供所有的语言阅读。eg. Content-Language:da

- Content-Length 实体报头域用于指明实体正文的长度，以字节方式存储的十进制数字来表示。

- Content-Type 实体报头域用语指明发送给接收者的实体正文的媒体类型。eg.Content-Type:text/html;charset=ISO-8859-1 或者 Content-Type:text/html;charset=GB2312

- Last-Modified 实体报头域用于指示资源的最后修改日期和时间。

- Expires 实体报头域给出响应过期的日期和时间。为了让代理服务器或浏览器在一段时间以后更新缓存中(再次访问曾访问过的页面时，直接从缓存中加载，缩短响应时间和降低服务器负载)的页面，我们可以使用Expires实体报头域指定页面过期的时间。eg. Expires：Thu，15 Sep 2006 16:23:12 GMT
HTTP1.1 的客户端和缓存必须将其他非法的日期格式（包括0）看作已经过期。

```
为了让浏览器不要缓存页面，我们也可以利用 Expires 实体报头域，设置为0
jsp 中程序如下：response.setDateHeader("Expires","0");
```