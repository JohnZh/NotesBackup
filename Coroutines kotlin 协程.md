in short, kotlin 多线程异步框架，底层实现类似 Java 的线程池

核心 lib：

- **kotlinx-coroutines-core** — Main interface for using coroutines in Kotlin
- **kotlinx-coroutines-android** — Support for the Android Main thread in coroutines



#  基础概念
- Scope: 创建和运行协程，提供生命周期事件
- Context: Scope 为其内部协程提供了一个上下文，能包含一些变量，并提供变量读写，比如 CorotinineName
- Suspending functions: 挂起方法，运行与协程内部的方法（可以被挂起）
- Jobs: 协程句柄
- Deferred: 协程结果，类似 Java Future
- Dispatcher: 管理当前哪个线程运行于协程中

# Dispathcer
- Main
	- UI 程序的主线程，例如 Android
	- 通常需要被定义在主线程
- Default
	- Cpu 密集工作
- IO
	- 网络通信文件读写工作
- Unconfined
	- 协程继承下来的 dispatcher，系统决定（再学习下）
- newSingleThreadContext("MyThread")
	- 强制创建一个新线程
- 





