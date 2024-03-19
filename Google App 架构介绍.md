# Google App 架构分层建议

- 界面层
- 网域层（optional）
- 数据层



# 界面层

显示数据，反应用户交互（按钮，输入框）变化。一般分为 2 部分

- 呈现数据的界面元素，View or Compose
- 存储数据、向界面提供数据以及处理逻辑的**状态容器**，一般是 ViewModel



## UI layer pipeline

- App data -> UI data
- UI data -> UI elememts
- User events -> UI changes
- Repeat





# 数据层

数据层包含业务逻辑，其决定应用价值，包含决定应用数据 CRUD 的规则



数据层由多个 Repo 组成，每个仓库可以包含多个数据源。应该为不用类型的数据分别创建一个 Repo

eg. 电影相关数据的 MoviesRepository，付款数据相关的 PaymentsRepository



## 存储库类负责的任务

- 向应用的其余部分公开数据。
- 集中处理数据变化。
- 解决多个数据源之间的冲突。
- 对应用其余部分的数据源进行抽象化处理。
- 包含业务逻辑。



## 数据源类

每个数据源类应仅负责处理一个数据源，可以是：

- 文件
- 网络来源
- 本地数据库



# 网域层（可选）

网域层是位于界面与数据层之间的可选层

网域层负责封装复杂的业务逻辑，或者由多个 ViewModel 重复使用的简单业务逻辑。请仅在需要时使用该层，例如处理复杂逻辑或支持可重用性。



## 网域层举例

此层中的类通常称为“用例”或“交互方”。每个用例都应仅负责单个功能。

如果多个 ViewModel 依赖时区在屏幕上显示适当的消息，则您的应用可能具有 `GetTimeZoneUseCase` 类





# 相关技术

- [Kotlin Flows](https://developer.android.com/kotlin/flow)
- 