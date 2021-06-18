# gradle groovy 几个特点

- 和 java 兼容

- 返回值是最后一句

  ```groovy
  int f(int i) {
   println(i)
   i * 3
   def y = i
  }
  
  println f(2)
  
  // output
  2
  2
  ```

- def a = "a"，def 类似于 java 的 Object

- 



# groovy Strings

- 支持单双引号和多行表达式

  ```groovy
  def s = "gradle" // 'gradle'
  def s = """
  		line 1
  		line 2
  """
  ```

- 支持插入值

  ```groovy
  def x = 4
  println "x is $x"
  
  def course = "gradle"
  println "I'm training in ${course.toUppperCase()}"
  ```



# groovy properties

```groovy
class Person {
	String name
	Integer age
	Persion(name, age) {
	
	}
}
def p = new Person("F", 20)
println p.age

// map sample
Map m = new HashMap()
m.put("foo", "F")
println m.get("foo")
m.goo = "U"
println m.foo

// output
F
U
```



# groovy closures

闭包

```groovy
def c = { param ->
	println param
}
def c = {
  println it
}
c("hello world") // output hello world

def c = {a, b, c
  println a
	println b
	println c
  10
}
c("A", "B", "C")
// output
A
B
C
10
```



## Closures type

- 作为传参

  ```groovy
  def mul(closure) {
  	closure() * 2
  }
  
  def mul2(closure, factor) {
  	closure() * factor
  }
  
  println mul {4}
  println mul2({4}, 3)
  // 圆括号可选：
  println mul2 {4}, 3
  
  // output: 
  8
  12
  ```

- 作为成员

  ```groovy
  class Person {
  	String name = "f"	
    
    Closure sayName = {
      println name
    }
  }
  def p = new Persion()
  p.sayName
  ```

- delegate

  ```groovy
  class Person {
  	String name = "f"
  	
    Person(name) {
      this.name = name
    }
    
    def executeInside(Closure c) {
      c.delegate = this
      c()
    }
  }
  
  def p = new Person("zxk")
  // println p.name
  // equals:
  p.execuateInside {
    println name
  }
  ```



# gradle object

6 个关键的 interface：

- Script <interface>，offical doc: https://docs.gradle.org/current/dsl/org.gradle.api.Script.html
- Project <interface>
- Gradle <interface>
- Settings <interface>
- Task
- Action



## gradle object doc





# gradle lifecycle

3 phases:

- Initialization 对应下面文件
  - init.gradle + others.gradle
  - settings.gradle -> multi-project
- configuration对应 
  - build.gradle
- execution 对应 
  - build.gradle 执行 tasks

在 initialization phase，会先执行 evaluate init.gradle+others.gradle，然后执行 settings.gradle（多 project 时候）。

> init.gradle + others.gradle 会位于 .gradle/init.d/。gradle run 的时候会遍历这个路径





# gradle properties

- 文件名 gradle.properties
- 在 project 根目录 或者 GRADLE_HOME
- 在 gradle.properties 和 .gradle 文件（eg. build.gradle/settings.gradle）也可以定义 properties
- 可以使用 hasProperties 方法来判断属性是否存在
- 可以使用 gradle.ext（ExtraPropertiesExtension）来定义额外的属性，也可以用  hasProperties 方法

```groovy
gradle.ext.sayHello = "Hello"
gradle.ext.timestamp = {
	Def df = new Simpledateformat("yyyy-mm-dd hh: mm: ssz")  
  df.setTimezone(Timezone.getTimezone("UTC"))
  return df.format(new Date())
}
```



# gradle tasks and actions

- project 包含多个 tasks

- task 包含多个 actions

- task 通过 addFirst/addList 方法添加 action

- task 内部实际上是一个 action list，addFirst 加到了 list 头部，addLast 加到了 list 尾部

  - 这意味着，最后的 addFirst 会执行，最后的 addLast 最后执行

- task 可以设置 description、group，同过 task api，也可以：

  ```groovy
  task hello {
  	description = "d"
    group = "group1"
  }
  ```



## task 的依赖关系

dependsOn，被依赖的先执行

```groovy
task task1 {
	doLast {logger.info "execute task1"}
}
task lastTask (dependsOn: 'task1') {
	doLast {logger.info "execute last task"}
}

// 依赖多个
task lastTask (dependsOn: ['task1', 'task2']) {
	doLast {logger.info "execute last task"}
}

// task1 同样也可以 dependsOn 其他的 task，eg task0，那么执行顺序就是 task0，task1，task2，lastTask
```

- gradle 会构建 task graph 来定义task 的执行顺序，因此不会出现 task 重复执行的情况
  - eg. task1 和 task2 同时dependsOn task0, task 1 又 依赖 task2，执行顺序 task0，task2，task1



### filter task

```groovy
task task1 dependsOn (tasks.findAll { t -> t.name.startsWith("do")}) {  
} 

task task1 dependsOn (['task0',tasks.findAll { t -> t.name.startsWith("do")}]) {  
} 
```

### conditional logic

```groovy
if (condition) {
	task1 dependsOn task.findAll {t -> t.name.startsWith("do")}
}
```



## task graph

主要是一些比较重要的 hook 方法：

> https://docs.gradle.org/current/javadoc/org/gradle/api/execution/TaskExecutionGraph.html

- whenReady. eg

  ```groovy
  task task1 {
    logger.info "$name $version"
  }
  
  task task2 {
  }
  
  if (Math.random() < 0.5) {
    task1.dependsOn task2
  }
  
  project.gradle.taskGraph.whenReady { taskGraph ->
    if (taskGraph.hasTask(task2)) {
      project.verison = 1.0
    } else {
      project.version = 0.5
    }
  }
  ```

- beforeTask，afterTask

  - hook 在每个 task 的之前和之后。eg

  ```groovy
  gradle.taskGraph.beforeTask { task ->
  	logger.info "before $task.name"
  }
  gradle.taskGraph.afterTask {
  	logger.info "after $task.name"
  }
  ```



# gradle pluign

## java

https://docs.gradle.org/current/userguide/java_plugin.html#java_plugin

eg. 使用 jar task 生成 jar 文件：

```groovy
jar {
  // 创建清单文件
    manifest { 
        attributes "Verson" : "1.0.0", "Main-Class" : "MyApp"
    }
  
  // 把第三方 jar 也添加到 jar 里面
  	from {
		    project.configurations.runtimeClasspath.collect {file -> project.zipTree(file)}
    }
}
```



## web application
