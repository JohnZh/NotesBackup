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

关键点：高阶函数和 lambda 都是具备函数功能的对象

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