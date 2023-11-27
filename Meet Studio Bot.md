# 什么是 Meet Studo Bot

简单说，Android 开发相关的问答式 AI，帮开发人员更高效的写代码



# 如何建立

- 最新版的 canary 
- 推荐同意 Studio Bot 发生数据到 Google
- 位置 View: Tool Windows: Studio Bot
- 登录 Google 账号



# 问 Bot 问题

```shell
How do I add camera support to my app?
I want to create a Room database.
Can you remind me of the format for javadocs?
What is dark theme?
What's the best way to get location on Android?
```

Bot 会记住上下文，所以可以继续发问：

```shell
Can you give me the code for this in Kotlin?
Can you show me how to do it in Compose?
```

甚至可以问 AS 的问题：

```
How do I analyze jank in my app?
Where do I find the CPU profiler?
```



# 使用 Bot 技巧

- 明确的。如果有明确的框架，API，方式是我们需要的，包含关键词在问题里
- 描述答案的结构。如果你想要插入的代码是有结构的，明确这些指令
- 把复杂的一个问题简化成若干个简单的小问题



# Bot 是如何帮忙的

- 生成需要插入的代码，甚至是添加需要或者流行的依赖
- 提供有帮助的资源。例如一些相关文档，网页



# FAQ

- 驱动 Bot 的模型是什么？
  - Codey，PaLM2 的派生
- Bot 会发送你的代码到 Google 吗？
  - 不会
- 我的代码会训练 Bot 吗？
  - 不会。只有用户使用的统计数据会发生过去
- Bot 会给出精确和安全的相应吗？
  - 早期，还是试验性阶段。需要你自己核对
- Bot 对编码有帮助吗？
  - 是的。但是由于是实验性阶段，结果还是要你自己审查

.......

