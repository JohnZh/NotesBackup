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
- 支持请求 Request 分组，用于组织，自动化测试
- 支持多个作用域的[变量](#variable)，方便 api 测试，自动化测试
- 



<a id="variable" />

## 变量

5 个作用域：

- 全局 Global：Collection，Requests，test script，environment 都可以访问（全域）
- 集合 Collection：集合内部所有请求
- 环境 Environment：所有集合都可以访问，不过只能选择一个 env 应用到集合上
- 数据 Data：数据变量来自于 CSV Json 文件的定义的数据
- 本地（临时） Local：只能用在请求脚本上，应用于单个请求或集合，运行完后不在可用

作用范围依次 Global > Collection > Environment > Data > Local，且作用域小的变量同名的情况下会覆盖作用域大的变量值

### 变量使用

- 访问：{{ variable}}

- 在 script 里面使用：

  ```shell
  // define
  pm.globals.set("variable_key", "variable_value");
  pm.collectionVariables.set("variable_key", "variable_value");
  pm.environment.set("variable_key", "variable_value");
  pm.variables.set("variable_key", "variable_value");
  // remove
  pm.environment.unset("variable_key");
  
  // usage
  //access a variable at any scope including local
  pm.variables.get("variable_key");
  //access a global variable
  pm.globals.get("variable_key");
  //access a collection variable
  pm.collectionVariables.get("variable_key");
  //access an environment variable
  pm.environment.get("variable_key");
  
  // log
  console.log(pm.variables.get("variable_key"));
  ```

- Using **Persist** will make your current value sync with Postman's servers and be reflected for anyone sharing your collection or environment. To reset your current local values to reflect the initial (shared) values, use **Reset**.

- 动态变量：

  ```shell
  {{$guid}} : A v4 style guid
  {{$timestamp}}: The current timestamp (Unix timestamp in seconds)
  {{$randomInt}}: A random integer between 0 and 1000
  ```



## 
