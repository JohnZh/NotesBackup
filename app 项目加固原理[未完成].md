app 项目加固原理[未完成]

---



# 知识大纲

- apk 打包流程，以及流程中用到的工具

- 加固的原理

- dex 文件的结构和加密的细节

  



# apk 打包流程及工具

Android studio 创建 apk 本质上是使用 Android sdk 和 jdk 提供的各种工具进行一连串的任务，然后做出了可以安装的 apk

> 工具的路径：~/~/Library/Android/sdk/build-tools/具体版本/

1. aapt（-R 参数）生成 R.java（开发阶段）、tools/aidl 把 AIDL 文件变成 java 文件

2. jdk 的 javac 编译 java 文件为 class 文件

3. class 文件（包含了第三方 Lib）转变为 dex 文件（转变涉及到 desugar、proguard、dexCompile）

   工具根据 AS 的版本不同可分为：

   1. dx AS3.2 之前
   2. d8 AS3.2 开始，build tools 28，类似 desugar + dx
   3. r8 AS3.4 开始，类似 proguard  + d8

4. aapt 把资源（assset 除外，res 下的图片、xml，以及 Manifest）编译成二进制文件，并打成一个 apk，包含 AndroidManifest，二进制xml 文件和 **resources.arsc**，没有 dex 文件

   > resources.arsc 就是一张索引（id）和资源文件的对应关系表
   >
   > 最新的 AS 已经开始采用 aapt2，相比于aapt package 命令，aapt2 把打包分成了两步，compile+link
   >
   > 原因是为了可以单独打新包的时候可以单独编译修改过的资源，而不是编译全部

5. 再把 dex 和 aapt 的资源包打包成一个可运行的 apk（apkbuilder）

6. tools/apksigner 进行 apk 签名

7. tools/zipalign 进行对齐优化

   > 它会使 APK 中的所有未压缩数据（例如图片或原始文件）在 4 字节边界上对齐。这样一来，即可使用 `mmap()` 直接访问所有部分，即使其中包含具有对齐限制的二进制数据也没关系。这样做的好处是可以**减少运行应用时消耗的 RAM 容量**。



# 加固的原理

1. 解压原 apk 包到临时目录下（unzip）
2. 读取目录里面所有的 dex 文件，进行加密
3. 添加壳 dex 到临时目录。壳 dex 的作用：解密并加载原 dex
4. 重新打包（zip），签名，对齐



## 壳 dex 解密和加载原 dex



