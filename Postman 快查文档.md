[toc]

# 功能概要

## 基本

- 提供了包含 HTTP GET/POST/PUT/PATCH/DELETE 在内的十多种方法
- GET 请求支持了 '?&' 参数拼接模式和路径参数模式。eg. customer/:id，支持手动 urlEncode 参数
- 请求体（request body）支持 form，url-encoded, raw, binary, GraphQL，并且在选择后会自动添加 Content-Type Header
  - raw 支持 text、json、xml、html 等
  - 默认是 Content-Type 是 None，binary 没有 Content-Type Header
  - 如果自己设置 Content-Type，Postman 自动添加的 Content-Type Header 值会被覆盖
- 支持 Authorizing request，见 Authorization Tab，设置了授权信息，这些信息会被添加到相关的 HTTP 报文字段中，比如 Headers
- 支持配置和管理 Cookie
- 支持设置配置开关：SSL 证书，重定向，自动 encode URL 等
- 支持多种模式的响应 Response 数据（body）的查看预览
- 支持响应 Response 的状态，耗时，数据大小，cookie，网络和ssl 证书验证信息的查看
- 







