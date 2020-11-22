# Scrum Log

## 2020-11-12 Thursday

- 阅读了启动初始化的代码，请求的接口分别：
  - ConsumerGetApi
  - ConsumerTagsGetApi
  - ConsumerTagsPutApi
- 发现使用 TinyTask（Thread）的异步任务处理方式应该也造成了不少性能问题，建议改用线程池，或者去除



## 2020-11-18 Wednesday

- 修复了 viewpager2 Fragment 单页刷新 bug
- 修复了 coupon 描述信息无的问题（持久数据模型里面添加字段）
- 测试了覆盖安装，发现数据库现在的操作策略是**遇到要迁徙的情况（抛出异常）就删除重建**



## 2020-11-19 Thursday

- 开会了解 POC 及 overflow
- 阅读了 firebase GA 文档
- 删除了 fabric 残留代码，完全使用 Firebase GA



## 2020-11-20 Thursday

- 修改了 firebase 的 paymentType args，并测试
- 添加了 firebase map_select_store event，并测试
- 添加信用卡管理页面文字
- 

