ref:

https://blog.csdn.net/luoshengyang/article/details/42379729

https://blog.csdn.net/iblade/article/details/78959750

https://blog.csdn.net/a78270528/article/details/80318571



# Dalvik（DVM）与 JVM 区别

- DVM 没有遵循 JVM 规范
- 基于寄存器实现
  - 区别于基于操作数栈的 JVM：可移植性强，指令数量更多
  - 基于寄存器：效率更高，指令长度更长（基于栈 1 个字节，寄存器 2 个字节）
- 执行字节码不同，JVM 解释执行 .class 文件，DVM 解释执行 .dex
  - .dex：.class 通过 DX 工具整合成一个 .dex（也可以是多个），去除了很多.class 冗余信息，提高了查找效率
- 一个 app 对应一个 DVM 实例。系统创建 app 会通知 Zygote fork 自己，创建和初始化一个DVM 实例运行app



# DVM Heap
