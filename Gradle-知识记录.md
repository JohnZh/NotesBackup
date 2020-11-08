---
layout: post_no_cmt
title:  "Gradle 知识记录"
date:   2020-06-30 00:06:00 +0800
categories: [android, java]
---

# Gralde 介绍
自动化构建工具，一个编程框架

使用 Groovy 语言，先会编译成字节码，然后通过 JVM 加载运行

# Gradle 基本概念
- [Lifecycle 生命周期](#lifecycle)
- [Wrapper：包装](#wrapper)
- [Closure：闭包](#closure)
- [Project：项目](#project)
- [Tasks：任务](#task)
- [Hooks：钩子](#hooks)
- [Plugin：插件](#plugin)

<a id="lifecycle" />

## Lifecycle
gradle 的生命周期，初始化，配置，执行
- 初始化：执行 settings.gradle，确定参与 build 的项目，为每个项目创建一个 Project 实例。当然对于单项目的工程来说，settings.gradle 不一定存在
- 配置：与 Project 一一对应的 build.gradle 会被执行，所有 Project 实例会被配置。这个阶段也会配置 Project 里面的 Task ，建立 Task 的有向无环图
- 执行：根据之前配置得到的有向无环图执行 Task

<a id="wrapper" />

## Wrapper
Gradle 当前处于一个快速迭代的阶段，Gradle 本身是为了提高开发效率，但是版本的配置无疑是麻烦的，因此引入 Gradle Wrapper 来屏蔽版本管理，Wrapper 会读取配置文件中 Gradle 的版本，下载项目需要的 Gradle 版本

AS 安装时自带 Wrapper 功能包，路径：

```
/Applications/Android\ Studio.app/Contents/plugins/android/lib/templates/gradle/wrapper/

drwxr-xr-x@ 3 john  staff    96 Dec  1  2015 gradle
-rw-r--r--  1 john  staff  5296 May  8  2018 gradlew
-rw-r--r--  1 john  staff  2260 May  8  2018 gradlew.bat

// 以上为 Wrapper 命令行工具，gradle/ 里面的内容（下面）在创建新项目时候都会拷贝进去一份

/Applications/Android\ Studio.app/Contents/plugins/android/lib/templates/gradle/wrapper/gradle/wrapper/

-rw-r--r--  1 john  staff  54329 Apr 19  2019 gradle-wrapper.jar
-rw-r--r--  1 john  staff    200 Apr 19  2019 gradle-wrapper.properties
```
gradlew 的命令也是为了屏蔽 gradle 本身命令的多变性，执行该命令的时候如果没有下载 gradle 会根据配置下载

具体看 Wrapper 下载下来的 gradle 版本，查看 gradle-wrapper.properties 文件

```
#Thu Mar 19 16:01:40 CST 2020
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-5.6.4-all.zip
```
下载下来全部版本的文件路径：~/.gradle/wrapper/dists/

> 当 AS 无法下载或下载缓慢的情况，推荐自己去国内镜像下载，并放到该文件夹下，后缀为-all：源码+二进制可执行文件+文档，-bin：二进制可执行文件，-src：源码

<a id="closure" />

## Closure
Groovy 对闭包的描述：
闭包，是一个代码块，或一个匿名函数，在外部方法调用时，可以将其作为方法的实参传递给方法的形参，并在方法内部回调此匿名函数，且在回调此匿名函数时可以传递实参到匿名函数的内部去接收，并执行此匿名函数

同时，此代码块或匿名函数也可以赋值给一个变量，使其具有自执行的能力，且最后一行的执行语句作为匿名函数的返回

> 参考 JS 的闭包：fun add(y) { return fun(x) { return x+y} } 第一个函数对第二个函数构成了闭包

### 举例
定义一个闭包，并且自我调用（执行）：

```
def t = {
    println "hello"
}

// 自我调用
t() // or t.call()

// output:
hello
```

查看返回值：

```
def t = {
    println "hello"
    "result"
}

// 自我调用
println "return: "  + t() 

// output:
hello
return: result
```

对闭包（匿名函数）的传参，未声明形参，默认用 it 接收

```
def t = {
    println "hello " + it
}

t("world")

// output:
hello world
```
声明形参：

```
def t = {
    x -> 
        println "hello " + x
}

t("world")
```
多个形参：

```
def t = {
    x, y -> 
        println "hello " + x + ", " + y
}

t("world", "too")

// output:
hello world, too
```

闭包作为实参传递给方法形参：

```
class People {
    String getName(Closure closure) {
        closure("john", "zhang")
    }
}

def name = {
    x, y ->
        println "name is " + x + " " + y
}

new People().getName(name)

// output
john zhang

// 不对闭包赋值 name，最后的语句可以写为
new People().getName({
    x, y ->
        println "name is " + x + " " + y
})

// 甚至闭包作为最后一个参数，可以写为
new People().getName {
    x, y ->
        println "name is " + x + " " + y
}
```
很面熟？项目代码：

```
allprojects {
    repositories {
        google()
        jcenter()
    }
}

// 对应的方法如下
Project.java:
    void allprojects(Closure configureClosure);
    void repositories(Closure configureClosure);
    MavenArtifactRepository google();
    MavenArtifactRepository jcenter();
```

再看个复杂点的例子：
```
dirs.each { dir ->
    println "src/$dir/java"
}
    
// dirs 类型是 List<String> 对应的调用 each 方法
public static <T> List<T> each(List<T> self, @ClosureParams(FirstGenericType.class) Closure closure) {
    return (List)each((Iterable)self, closure);
}

public static <T> Iterable<T> each(Iterable<T> self, @ClosureParams(FirstGenericType.class) Closure closure) {
    each(self.iterator(), closure);
    return self;
}

public static <T> Iterator<T> each(Iterator<T> self, @ClosureParams(FirstGenericType.class) Closure closure) {
    while(self.hasNext()) {
        Object arg = self.next();
        closure.call(arg);
    }

    return self;
}
```
实现了dirs 里面的元素作为 dir 参数循环作为参数传入闭包执行 println 操作

### 省略括号的总结

顶级表达式，括号可以省略，包括了闭包作为最后一个入参的时候：

```
println "hello" 等价 println("hello")
method a, b 等价 method(a, b)
dirs.each({...}) 等价 dirs.each {...}
```

但是也有不能省略的情况，内嵌方法或者右侧的赋值：

```
def f(n) {...}

// 以下情况是错误的
println f 1
def m = f 1
```

<a id="project" />

## Project
每个 module 对会对应一个 build.gradle, 每个 build.gradle 都会创建一个 Project 领域对象，编写脚本实际上是操作领域对象

因此看项目有几个 project 就只要看要几个 build.gradle。Android 项目一般几个 module 就有几个 build.gradle，查看 settings.gradle 就可以知道，此外不要忽略根目录下面还有一个 build.gradle，对应 rootProject

<a id="task" />

## Task 
一个 Project 本质上就是一个 Task 对象的集合。Task 也是最小执行单元，各自完成其片段工作，例如，编译类，运行单元测试，打包文件

定义 Task

```
task printTask {
    println 'hello'
}
```
也可以用 taskContainer 创建

```
this.tasks.create(name: 'printTask') {
    println 'hello too'
}
```

定义好的 task 会在 gradle 面板里面出现，且不是 doLast() 和 doFirst() 里面的内容会在 gradle 配置阶段就执行

### doLast & doFirst
doFirst 闭包外调用早于闭包内调用，doLast 相反：

```
task printTask {
    doFirst {println "2"}
    doLast {println "3"}
}

printTask.doFirst {println "1"}
printTask.doLast {println "4"}
```

### task 配置
task 提供了 name，group，type，description，dependOn，overwrite，action 等配置。举例

```
task printTask(group:'mygroup', description: 'this is my task') {
    // 定义分组和任务描述
}

task myTask(dependOn:[taskA, taskB]) {
    // 定义依赖，需要先执行 taskA 和 taskB（无序），才会执行 myTask
}

// 定义 task 依赖一组 task
task myTask {
    dependsOn(this.tasks.findAll {
        t -> t.group == 'myGroup'
    })
}
```

### task 执行顺序
task 执行默认是无序的，可以使用 mustRunAfter()，shouldRunAfter()，finalizedBy() 来设置顺序

must/shouldRunAfter() 只是宽松的定义了一个执行顺序，且是要求几个 task 都执行的情况下，它会规定其执行顺序。比如

```
task taskA {}
task taskB {}
task taskC {}

taskA.mustRunAfter(taskC)
taskC.mustRunAfter(taskB)
```
只有 `gradlew taskA taskB taskC`，三者都运行才会按照 B-C-A 的顺序执行

区别于 dependsOn 和 finalizedBy

taskA.finalizedBy(taskB)，会在 taskA 执行完后去执行 taskB
taskA.dependsOn(taskB)，会在 taskA 执行之前先去执行 taskB
命令都是 `gradlew taskA`

<a id="hooks" />

## Hooks
钩子，即挂在主干执行流程上其他分岔流程，想象一条宽大的路，钩子就像是垂直于这条路径的一些小路。在评估或者执行的时候，从大路的起点开始，经过小路的时候过去一下，执行完成了再回到大路继续执行

反应在 gradle 的代码中就是多个 hooks 的方法

hooks 点会遍布在生命周期的多个阶段

### 初始化

settings.grdle 文件

- gradle.buildStrt() 发生在初始化阶段前
- gradle.settingsEvaluated() 发生在 settings.gradle 文评估（执行）之后
- gradle.projectsLoaded() 初始化阶段结束时，所有 project 加载完毕

### 配置

根 build.gradle

- gradle.beforeProject() 配置阶段开始时，在评估 Project 之前
- project.beforeEvaluate() 同 gradle.beforeProject()，区别在于前者针对一个 Project，后者针对所有

```
gradle.beforeProject { project ->
    project...
}
// 等价于
allprojects {
    beforeEvaluate { project->
        project...
    }
}
```
同理还有
- gradle.afterProject()
- project.afterEvaluate()
- gradle.projectsEvaluated() 配置阶段结束时，所有的 build.gradle 评估之后，区别于 gradle.afterProject() 不论评估成功与否都会执行

配置的目的是为了构建“执行”的 task 有序无环图，这个图简单的可以理解为 task 的依赖执行顺序

- gradle.taskGraph.whenReady() 当这个图已经准备好了

### 执行

根 build.gradle

- gradle.taskGraph.beforeTask() tasks 执行之前

```
gradle.taskGraph.beforeTask { task ->
    task...
}
```
- gradle.taskGraph.afterTask()
- gradle.buildFinished() 发生在执行阶段后


<a id="plugin" />

## Plugin

插件 todo


## 关于 gradle 同步慢的问题及修复
同步慢无非是下列原因：
1. gradlew 下载 gradle 速度慢，这个问题之前 Wrapper 已经提及，fix 的方法是直接用别的下载器单独下载，或者替换-all 的下载直接下载-bin
2. 已经手工下载了 gradle，但是同步依旧很慢，问题或许出现在项目的依赖（项目插件或者 module 的依赖）下载上。我们需要添加修改或者添加仓库。fix 可以是添加国内的公共依赖的仓库镜像，比较多的是使用 ali 的仓库：

```
// buildscript 和 allprojects 两个闭包方法里面添加：
buildscript {
  repositories {
      maven { url 'https://maven.aliyun.com/repository/google' }
      maven { url 'https://maven.aliyun.com/repository/gradle-plugin' }
      maven { url 'https://maven.aliyun.com/repository/public/' }
  }
  dependencies {
      classpath 'com.android.tools.build:gradle:version'
  }
}
  
allprojects {
  repositories {
      maven { url 'https://maven.aliyun.com/repository/google' }
      maven { url 'https://maven.aliyun.com/repository/public/' }
  }
}
```
地址：https://maven.aliyun.com/mvn/view

3. 依赖版本号‘+’的滥用，版本号最后使用‘+’会不定期的去搜索新版本，这会导致之前已经编译好的状态重新被打破，需要重新下载编译
4. 比较根本的解决方案：搭建企业自己的本地仓库

## 参考
- [Gradle Doc](https://docs.gradle.org/current/dsl/index.html)
- [Groovy 语法参考](https://groovy-lang.org/syntax.html)
- [Android Gradle 插件文档](https://google.github.io/android-gradle-dsl)