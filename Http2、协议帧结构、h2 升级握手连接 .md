
# HTTP 2 VS HTTP 1
- HTTP 1.1 中 Header 是文本，Body data 可以是文本也可以二进制数据；HTTP 2 中使用了[**二进制·分帧·流**](#frames)，这也是 HTTP 2 Multiplex 特性的基础
- HTTP 1.1 时代，客户端，浏览器为了提升带宽的利用率，对同一个域名会开启 6~8 个 TCP 连接；HTTP 2 一个域名一个 TCP 连接，甚至在发现两个域名的 IP Address 一样后还会使用相同的连接
- 不仅是优化了 HTTP 1.1 资源使用，最重要的是 HTTP 2 解决了 HOL Blocking 队头阻塞
- 此外还加入了头部压缩 HPACK 算法，流量控制（流量窗口），服务器推送，请求优先级，连接重置等特性

> 更多详情，查阅官方英文文档：[《Hypertext Transfer Protocol Version 2》](https://httpwg.org/specs/rfc7540.html)



<a id="frames" />

## 二进制·分帧·流
- 帧：传输的最小单位，所有的控制，请求，响应，数据都会被包装成帧的，例如替代原先 HEADER 和 BODY 的 HEDAER Frame 和 DATA Frame。所有的帧都有一个 9 Bytes 的固定数据结构的头部，定义了内容长度，类型，标示，Stream-id，保留位，最后是二进制的具体传输内容 Payload
- 流：逻辑概念，代表客户端和服务端之间数据交换期间的双向帧序列。每个帧的 Stream-id 代表这个帧属于哪个流，待接收方得到全部归属某个于 Stream-id 的帧后重新组装成完整数据

# 帧的结构
了解一下帧的结构可以便于理解 HTTP 2 的传输方式

## Frame header 帧头部

帧都有一个固定的头部，9 字节（72 bit）和一个长度可变的负载 Payload：

```
+-----------------------------------------------+
 |                 Length (24)                   |
 +---------------+---------------+---------------+
 |   Type (8)    |   Flags (8)   |
 +-+-------------+---------------+-------------------------------+
 |R|                 Stream Identifier (31)                      |
 +=+=============================================================+
 |                   Frame Payload (0...)                      ...
 +---------------------------------------------------------------+
```

- Length：帧的长度，不包括头部。24 位无符号数。一般小于等于 2^14(16384)，除非 SETTINGS_MAX_FRAME_SIZE 设置更大的数（取值：2^14 ~ 2^24 - 1）
- [Type](#type)：帧类型。Type = unknown 的帧会被忽略。
- Flags：帧类型的标志符。不同帧类型不同语义。如果该帧类型未定义语义，必须赋值（0x0）。
- R：保留位。必须保持 unset(0x0)
- Stream Identifier：流标识。客户端创建的为奇数，服务端的为偶数。值（0x0）不标识流，只用于连接控制的相关帧
- Payload：主体内容，帧类型决定



<a id="type" />

### Type 帧的类型
Type 使用了 8-bit 的空间，0xf0 ~ 0xff 保留用于实验使用，当前注册使用如下：

Frame Type|Code|说明
---|---|---
[DATA](#data)|0x0|数据帧
[HEADERS](#headers)|0x1|报头帧：打开一个流或者携带一个首部片段
PRIORITY|0x2|优先级帧
RST_STREAM|0x3|流终止帧：取消一个流或者表示发生一个错误
[SETTINGS](#settings)|0x4|设置帧
PUSH_PROMISE|0x5| 推送帧
PING|0x6|判断一个空闲连接是否依然可用
GOWAY|0x7|发起关闭连接的请求或警示严重错误。关闭前会处理完之前建立的流
WINDOW_UPDATE|0x8|窗口更新帧：执行流量控制。可指定具体 Stream_Identifier 或者整个连接 Stream_Identifier(0x0)。只有数据帧受流量控制，初始化流量窗口后，发生多少 Payload，流量窗口减少多少，不足后就无法发送，使用该帧增加窗口大小
CONTINUATION|0x9|延续帧：用于连续传送首部片段序列



<a id="data"/>

## DATA 数据帧

```
+---------------+
 |Pad Length? (8)|
 +---------------+-----------------------------------------------+
 |                            Data (*)                         ...
 +---------------------------------------------------------------+
 |                           Padding (*)                       ...
 +---------------------------------------------------------------+
```

- Pad Length：Padding 长度，出现说明头部 PADDED Flag（比特位 3 号位：.... 1...）被设置
- Data：传递的数据
- Padding：填充字节，没有具体语义，发送时必须设为 0，作用是混淆报文长度。接收端没有义务核对 padding，但是也许会把非 0 padding 当做连接错误 PROTOCOL_ERROR

### DATA 帧头部的 Flags
- END_STREAM(0x1)：bit 0，即.... ...1，是否为最后一帧。设置这个 flag 会使流进入 half-closed 或者 closed 状态
- PADDED(0x8)：bit 3，1 表 padding 存在

![sample1](./sample1.jpg)



<a id="headers"/>

## HEADERS 报头帧

```
+---------------+
 |Pad Length? (8)|
 +-+-------------+-----------------------------------------------+
 |E|                 Stream Dependency? (31)                     |
 +-+-------------+-----------------------------------------------+
 |  Weight? (8)  |
 +-+-------------+-----------------------------------------------+
 |                   Header Block Fragment (*)                 ...
 +---------------------------------------------------------------+
 |                           Padding (*)                       ...
 +---------------------------------------------------------------+
```

- Pad Length：同 DATA 帧
- E：流依赖的专一性标记位，出现则表示 PRIORITY flag 被设置
- Stream Dependency：依赖流的 id，出现则表示 PRIORITY flag 被设置
- Weight：8-bit 无符号整数，值：1~256，出现则表示 PRIORITY flag 被设置
- Header Block Fragment：报头块片段
- Padding：同 DATA 帧

### HEADERS 帧头部 Flags
- END_STREAM(0x1)：bit 0，同 DATA 帧，设置的时候，还可以跟 CONTINUATION 帧，作为 HEADER 帧的一部分
- END_HEADERS(0x4)：bit 2，表示包含一个完整的 Header block，不能跟 CONTINUATION 帧
- PADDED(0x8)：bit 3，同 DATA 帧
- PRIORITY (0x20)：bit 5，1 表 E，Stream Dependency，Weight 存在

![sample2](./sample2.jpg)

# HTTP 2 的连接
```
客户端 发送：

GET / HTTP/1.1
Host: server.example.com
Connection: Upgrade, HTTP2-Settings
Upgrade: h2c
HTTP2-Settings: <base64url encoding of HTTP/2 SETTINGS payload>
```
>HTTP 2 Over TLS 的情况下，h2c -> h2

```
服务端不支持 HTTP 2 的响应：

HTTP/1.1 200 OK
Content-Length: 243
Content-Type: text/html
```
之后按照 HTTP 1.1 规范进行连接通信

```
服务端支持 HTTP 2 的响应：

HTTP/1.1 101 Switching Protocols
Connection: Upgrade
Upgrade: h2c

[ HTTP/2 connection ...
```

- 连接建立初步完成，服务端第一帧会发送 SETTINGS 帧，作为连接序言(preface)
- 客户端收到 101 响应之后也必须发送连续序言，其中含有一个 Magic 二进制符号和一个 SETTINGS 帧

#### [3.5. HTTP/2 Connection Preface](https://httpwg.org/specs/rfc7540.html#ConnectionHeader)

- 每个端需要发送一个连接序言作为协议的**最后确认**以及为连接**做初始设定**
- 客户端的序言以 0x505249202a20485454502f322e300d0a0d0a534d0d0a0d0a 这样的二进制符号开始（字符串形式：PRI * HTTP/2.0\r\n\r\nSM\r\n\r\n），这就是 Magic，然后跟着一个 SETTINGS 帧，这个 SETTINGS 帧可能为空。
- 客户端发送客户端连接序言是在收到 101 响应时马上发生的，或者作为第一个应用数据发送（在预先知道两端支持 HTTP 2 的情况下）（图 23 行）
- 服务端连接序言则是一个可能为空的 SETTINGS，并作为连接中第一个帧发送（图 26 行）

#### [6.5.3. Settings Synchronization](https://httpwg.org/specs/rfc7540.html#SettingsSync)
- 接受端当收到非 ACK SETTINGS 帧，参数更新需要是尽可能的快，并且是SETTINGS 里面的所有值处理必须是有序的，处理完后马上发送值为 ACK 的 SETTINGS 帧给发送端，以便发送端知道参数已经生效并以此为传输依据（图 29.31 行）

![sample3](./sample3.jpg)

图中也可以看出为了避免不必要的延时，在客户端发送了序言后，就允许马上发送其他帧而不用等待接收服务端的序言。但是，也许服务端序言中的 SETTINGS 帧也许包含了一些参数也许是告知客户端如何与服务端交流的。因此在有些配置中，为了避免错误，服务端发送 SETTINGS 在客户端发送其他帧之前是完全可能的

<a id="settings" />
## SETTINGS 设置帧
因为 SETTINGS 帧作用域为整个连接，因此头部的 Stream-id 必须是 0x0

```
+-------------------------------+
|       Identifier (16)         |
+-------------------------------+-------------------------------+
|                        Value (32)                             |
+---------------------------------------------------------------+
```

### SETTINGS 帧的 Identifier
- SETTINGS_HEADER_TABLE_SIZE (0x1)：用于解析 Header Block 的压缩表的大小，初始值 4096 Bytes
- SETTINGS_ENABLE_PUSH (0x2)：服务器推送，初始值 1，允许推送
- SETTINGS_MAX_CONCURRENT_STREAMS (0x3)：发送端允许接收端创建的最大流数
- SETTINGS_INITIAL_WINDOW_SIZE (0x4)：发送端所有流的流量控制窗口大小，初始值2^16 - 1(65535)。最大值 2^31 - 1。超出会返回 FLOW_CONTROL_ERROR
- SETTINGS_MAX_FRAME_SIZE (0x5)：发送端允许接收的最大帧负载的字节数，初始值 2^14(16384)，最大值 2^24 - 1。如果不在此范围内，返回 PROTOCOL_ERROR
- SETTINGS_MAX_HEADER_LIST_SIZE (0x6)：通知接收端，发送端准备接收的首部列表大小的最大字节数。基于未压缩的首部域大小，包括名称和值字节长度，外加每个首部域 32 Bytes 开销

### SETTINGS 帧头部 Flags
- ACK：bit-0，1 代表接受发生方的请求并同意。作为 ACK 响应时，帧的 Payload 必须为空

# 