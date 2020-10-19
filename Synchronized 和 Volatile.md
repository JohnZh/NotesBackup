# 学习视频

https://www.bilibili.com/video/BV1aQ4y1P7Me?p=1



# 自旋

循环+CAS



# CAS 原理

```java
	Unsafe native 方法
    虚拟机：C++实现：Atom::cmpxchg
      系统指令：lock comxchg
```



# Synchronized 原理

重量锁的概念：需要操作系统的线程管理，因此 lock comxchg 没经过线程管理，非重量级



## 对象内存布局

- markword：8 bytes
- klass pointer：8bytes compressed 4bytes（default）
- instance data
- padding



## markword 里重要的信息

- hashcode 只会存储原始的 hashcode，开发者自己重写的不会记录；还有系统方法生成；升级轻量锁后 hashcode 会和原来状态存到自己的线程栈里面，数据结构 Lock Record（LR）；升级重量锁会存在 objectMonitor 里面
- 分代年龄，4 bytes，最大值 15
- 锁信息（标记位和锁偏向）



## 锁升级中的 LR

LockRecord 用于轻量级锁优化，当解释器执行 monitorenter 字节码轻度锁住一个对象时，就会在获取锁的线程的栈上显式或者隐式分配一个 LockRecord。这个 LockRecord 存储锁对象 markword 的拷贝 (Displaced Mark Word)。

在拷贝完成后，首先会挂起持有偏向锁的线程，因为要进行尝试修改锁记录指针，MarkWord 会有变化，所有线程会利用 CAS 尝试将 MarkWord 的锁记录指针改为指向自己 (线程) 的锁记录，然后 lockrecord 的 owner 指向对象的 markword，修改成功的线程将获得轻量级锁。

失败则线程升级为重量级锁。释放时会检查 markword 中的 lockrecord 指针是否指向自己 (获得锁的线程 lockrecord)，使用 CAS 将 Displaced Mark Word 替换回对象头。如果成功，则表示没有竞争发生，如果替换失败则升级为重量级锁。

整个过程中，LockRecord 是一个线程内独享的存储，每一个线程都有一个可用 `Monitor Record` 列表。

**偏向锁的加锁也会有 LR，释放锁的时候弹出**。

**锁重入也没有用到计数器，而是 LR 的个数**



## 锁升级

1. 锁标记为2bit，锁偏向1bit
   1. 00：轻量级锁
   2. 10：重量级锁，
   3. 01：再看锁偏向 bit：1 偏向锁，0 无锁（101 偏向锁，001 无锁）
   4. 11：标记要被 GC
2. 偏向锁：线程A来获取锁，对象头里获锁线程 id 赋值为线程 A 的 id，锁偏向 1，锁标记 01
   1. 释放锁后，偏向锁 bit 为 0，
3. 轻量锁：轻量竞争。对象头获锁线程 id 会擦除。自旋。
4. 重量锁：竞争更加激烈。调用操作系统的线程管理，objectMonitor 来实现
   1. jdk1.6之前，有线程自旋超过 10 次 or 总的线程等待数超过 cpu 核心的 1/2，升级重量锁
   2. jdk1.6 之后，自适应自旋，即 jvm 自己决定什么时候升级重量锁
5. 偏向锁直接升级重量锁。过度竞争。wait 方法的调用



### 偏向锁流程

1. 偏向锁未启动，new 对象，轻量级锁
2. 偏向锁启动，new 对象，匿名偏向，即 new 出来的对象对象头里默认锁相关 bit 101，线程 id 0
3. 对 new 的对象加锁，对象头上赋值线程 id，即偏向锁



#### 偏向锁为什么会延迟 4 秒

默认情况下偏向锁是启动的，但是会有延时。但是打开偏向锁效率一定会高吗？为什么

在明明知道会发生竞争的情况下打开偏向锁效率并不会高，而偏向锁为什么会延迟 4 秒，因为 jvm 知道在启动的过程会有很多线程竞争（比如内存的锁定加载系统的类）



## 锁降级

在 GC 线程来的时候会有锁降级



## Synchronized 原理参考阅读

https://blog.csdn.net/asd804171023/article/details/105297372/



# Volatile 特性与原理



- 可见性
- 有序性



## 可见性

先了解一下 CPU 和内存的这个架构以及从内存读取数据的过程

一颗 CPU拥有多核，每个核心都有自己 L1、L2缓存，每个 CPU 还有一个 L3 缓存

CPU 要从内存读取数据进行计算，由于内存速度和 CPU 速度差了近 100 倍，需要先将数据从内存拷贝到 L3，L2，L1 里面再到寄存器，然后由 ALU（Arithmetic Logic Unit：运算单元）计算

> 4 核 8 线程 CPU：8 线程指的就是 4 个物理核，一核一个 ALU，每个核的 ALU 对应两组寄存器，每个寄存器存放一个线程数据

> 线程切换，CPU 执行线程就是把线程数据放入寄存器里面 ALU 去计算。切换就是把线程数据先存起来，取另外的线程数据都寄存器里面用 ALU 计算



### Cache line 缓存行

数据是一块一块的从内存读到缓冲再到 CPU，一块的大小是 **64 bytes**，称为缓存行，cache line

> 64 bytes 是一个折中值。缓存行越大，局部性空间效率越高，读取越慢；反之亦然



### 缓存一致性协议

MESI（Intel CPU） 等其他缓存一致性协议（MSI，MOSI等等），又称缓存锁。它如何保证多线程的缓存一致性呢？

> MESI 四个状态：Modified，Exclusive，Shared，Invalid

A 线程里面改变了 x 变量，标记为 cache line Modified，写回内存，然后通知其他线程 x 变量所在的 cache line Invalid，其他线程就会去内存重新加载 cache line



#### 利用缓存行对齐提高数据性能

核心思想就是把变量加上 long 数据填充到 64bytes，然后在线程读入这个变量的时候就是整个缓存行，所以就不存在改变变量需要失效其他线程缓存行的问题，因此不停改变变量速度反而变快了



## 有序性



### 指令重排序

情景：CPU 在读指令1：去内存读数据，CPU 会利用等待时间优先执行指令 2

当然重排序必须保证指令改变顺序后结果的一致性，这是 CPU 的优化策略



### as-if-serial 和 happen-before

- as-if-serial：看上去好像是序列化执行下来的，即就算是发生了指令重排，执行下来结果是一致的
- happens-before：JVM 规定的一系列关于重排序的规则，目的也是为了结果的一致性



### volatile 有序性实现细节

内存屏障：JVM 要求所有的 java 虚拟机必须实现 4 种屏障：LoadLoad，StoreStore，LoadStore，StoreLoad

volatile JVM 层实现细节：

volatile 写：StoreStoreBarrier|volatile 写|StoreLoadBarrier

volatile 读：volatile 读|LoadLoadBarrier|LoadStoreBarrier

操作系统层实现细节：

Lock 指令，作用：将当前CPU或核心对应的缓存刷新回内存，并通知其他 CPU或核心对应的缓存失效（CPU 总线嗅探机制），并且在执行到这里的时候所有的指令都不会越过它，实现了有序性



# Volatile 和 单例模式

单例双重检查模式静态对象是否要用 volatile 修饰？

对象在创建的时候执行了类加载检查，分配内存，赋零值，设置对象头（指令：new），调用<init>方法（调用构造器），赋值给引用。如果不加 volatile，那么赋值给引用和调用<init>方法这两个步骤运行重排序。

运行到下面代码 1 处的线程，发现 s 不为 null，但是可能之前的线程还在 synchronized 代码块，执行可能只做了赋值引用，还没做初始化，那么 1 处的线程返回的对象就是零值对象，如果改变内容再被初始化，那么就出现了脏数据

```java
class SingleInstance {
	private static SingleInstance s; 
  if (s == null) { // 1
    synchronized(this) {
      if (s == null) {
        s = new SingleInstance();
      }
    }
  }
  return s;
}
```

