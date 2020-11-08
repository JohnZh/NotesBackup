# dx、d8、r8 概述
Android 的 VM 从最开始 Dalvik，到后面的 ART，运行的字节码文件都是 dex 文件。这意味着：

1. java 文件或 kotlin 文件，以及类库，先需要 javac 或 kotlinc 变成 class 文件后（该过程是 Compile）
2. 接着 Transform（包含 Desugar、Proguard）
3. 最后通过 dex 工具处理，然后才会变成 Android VM 可执行的 dex 文件

>desugar：把 java7 和 java8 的语法特性转变为 java6 的语法



- dx：第一代 dex 工具，把 class 文件转变为 dex 文件，然而它只支持到 java6 语法，因此使用它之前必须Desugar

- d8：第二代 dex 工具，加入 Desugar 功能，直接可以把 java 8（lambda） 的 java 文件产出 dex 文件。此外还加入了很多新的高效的指令，同时提供了向下兼容（兼容 Dalvik 虚拟机没有的指令）

  > 向下兼容问题：举个例子，一些 4.4 的设备的上 VM（Dalvik） 的实现没有 d8 新加的指令（比如 not int 指令）。当 dx 的过程发生在那些设备上，d8 不会使用那些新增高效的指令，而是使用其他现有的字节码指令来实现功能，而新增的指令只会在 ART VM（即 5.0 及其以上版本）才会使用。
  >
  > 
  >
  > 当然，这件事情其实 R8 也还在做。因为不同的手机厂商对 VM 的实现不同导致不得不做。不是单就 Dalvik，ART 的实现上也是
  >
  > 
  >
  > PS：手动调用 d8 进行 class 文件的编译，加上 --min-api 21，然后使用 dexdump $dexfile 就能看到使用了新的高效的指令

- r8：第三代 dex 工具，d8 的衍生，在 d8 基础上加入 Proguard 功能。3 大功能：

  - 摇树优化（tree shaking）：从 app 删除没用到的类，字段和方法
  - 代码优化：在指令级别让代码更加的小更高效
  - 命名混淆：重命名剩余的类，字段和方法的名称为短的和无意义的名字，这个主要作用是减少代码的体积

# d8 desugar 的过程

1. 源代码
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201026040042250.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d5enhrODg4,size_16,color_FFFFFF,t_70#pic_center)

2. 第一次转变，添加了静态方法实现 lambda 表达式![在这里插入图片描述](https://img-blog.csdnimg.cn/20201026035442360.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d5enhrODg4,size_16,color_FFFFFF,t_70#pic_center)


3. 第二次转变，创建了新的类 Java8$1 实现 Consumer 接口
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201026035551277.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d5enhrODg4,size_16,color_FFFFFF,t_70#pic_center)


4. 第三次转变，Java8$1创建内部静态对象
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201026035631855.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d5enhrODg4,size_16,color_FFFFFF,t_70#pic_center)

5. 但是实际上如果你写了一个 Java8 的内部类，它会被命名成 Java8$1，因此实际生成了下面的代码，这个新生成的类名称实际上是所有信息的 hash 值
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201026035659287.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d5enhrODg4,size_16,color_FFFFFF,t_70#pic_center)
**注意：每个 lambda 表达式会创建一个新的类**，如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201026035725710.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d5enhrODg4,size_16,color_FFFFFF,t_70#pic_center)

> 静态方法的 lambda 表达式不会这样，比如 System.out::println(s)