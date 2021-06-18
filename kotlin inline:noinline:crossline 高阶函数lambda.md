kotlin inline/noinline/crossline 高阶函数/lambda

---

# inline/noinline/crossline

- inline：声明在编译时，将函数的代码拷贝到调用的地方（内联）。本质上是为了执行效率
- noinline：声明 `inline` 函数的形参中，不希望内联的 `lambda`（默认是内联）。
  - inline 的 lambda 体内使用 return 会直接结束最外层调用 lambda 的 fun，不直接结束的方法是 return + 标签
  - 使用 noinline 标记  lambda 是非内联的，这会禁止 lambda 内部使用 return（lambda 默认最后一句话为返回值）
- crossinline : 表明 inline 函数的形参中的  lambda 不能有  return。

> ref: https://juejin.im/entry/6844903730450530311



# 高阶函数/lambda

记重点：**高阶函数和 lambda 都是具备函数功能的对象**



## 一般类型

```kotlin
fun <T, R> Collection<T>.fold(
    initial: R, 
    combine: (acc: R, nextElement: T) -> R
): R {
    var accumulator: R = initial
    for (element: T in this) {
        accumulator = combine(accumulator, element)
    }
    return accumulator
}
```

combine 就是一个函数类型的参数，其类型：`(acc: R, nextElement: T) -> R`

这个函数类型：形参类型为 R 和 T 的且返回值为 R 的类型

fold 的逻辑就是 for 当前集合，然后调用 combine 函数处理 accumulator 和集合的每一个元素，再把值赋值个 accumulator

```kotlin
val items = listOf(1, 2, 3, 4, 5)

// Lambdas are code blocks enclosed in curly braces.
items.fold(0, { 
    // When a lambda has parameters, they go first, followed by '->'
    acc: Int, i: Int -> 
    print("acc = $acc, i = $i, ") 
    val result = acc + i
    println("result = $result")
    // The last expression in a lambda is considered the return value:
    result
})

// Parameter types in a lambda are optional if they can be inferred:
val joinedToString = items.fold("Elements:", { acc, i -> acc + " " + i })

// Function references can also be used for higher-order function calls:
val product = items.fold(1, Int::times)

// 输出
acc = 0, i = 1, result = 1
acc = 1, i = 2, result = 3
acc = 3, i = 3, result = 6
acc = 6, i = 4, result = 10
acc = 10, i = 5, result = 15
joinedToString = Elements: 1 2 3 4 5
product = 120
```



### 函数类型举例

```kotlin
(Int) -> String
val onClick: () -> Unit = …… // 返回值为空：Unit 不能省略
(A, B) -> C // 接受参数分别为 A，B 的类型，返回值为 C 的类型
suspend () -> Unit 或者 suspend A.(B) -> C // 挂起函数
(x: Int, y: Int) -> Point // 可以选择性地包含函数的参数名，名称可用于表明参数的含义
((Int, Int) -> Int)? // 函数类型可 null
(Int) -> ((Int) -> Unit) 等价于 (Int) -> (Int) -> Unit // 函数类型解析为右优先，且返回值可以为函数类型
((Int) -> (Int)) -> Unit // 同上面不等价 

A.(B) -> C // 带接收器的函数类型，接收器类型为 A
```



### 函数类型的实例（对象）

- lambda：{ a, b -> a + b }

- 匿名函数：

  ```kotlin
  fun(s: String): Int {
    return s.toIntOrNull()?:0
  }
  ```

- 已有声明的可调用引用：

  - 顶层、局部、成员、扩展函数：::isOdd, String::toInt
  - 顶层、成员、扩展属性：List<Int>::size
  - 构造函数：::Regex

- 继承于函数类型的类的对象：

  ```kotlin
  class IntTransformer: (Int) -> Int {
    override operator fun invoke(x: Int): Int = TODO()
  }
  
  val intFun: (Int) -> Int = IntTransformer()
  val intFun2 = {i:Int -> i + 1}
  ```



### 难点：带与不带接收者的函数类型*非字面*值可以互换

```kotlin
(A, B) -> C 和 A.(B) -> C // 可以互相接收

fun main(args: Array<String>) {
  val repeatFun: String.(Int) -> String = { this.repeat(it) }

  fun runTransformation(f: (String, Int) -> String): String {
    return f("hello", 3)
  }
  val result = runTransformation(repeatFun) // OK

  println("result = $result")
}
```



### 带有接收者的函数字面值

```kotlin
val sum: Int.(Int) -> Int = { other -> plus(other) }

val sum = fun Int.(other: Int): Int = this + other

println(4.sum(4))
```



## 带接收者的类型

```kotlin
A.(B) -> C
```

这个类型表示，这个函数的接受者类型为 A，1 个参数且类型为 B，返回值为 C。

```kotlin
inline fun <reified T : Activity> Activity.startActivity(requestCode: Int = -1, 
                                                         crossinline action: Intent.() -> Unit = {}) {
    startActivityForResult(
            Intent(this, T::class.java)
                    .apply {
                        action.invoke(this)
                    }
            , requestCode)
}

// 使用
startActivity<OrderCollectActivity> {
                putExtra("isFromCheckIn", true)
            }
```

以上面的例子来解释，接受者的类型为 Intent，即 Lambda {} 代码块内 this（上下文对象） 的类型