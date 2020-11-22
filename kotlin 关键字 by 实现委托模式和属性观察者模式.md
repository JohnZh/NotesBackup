kotlin 关键字 by 实现委托模式和属性观察者模式

---

# by 实现委托模式

```kotlin
interface Window {
    fun open()
}

class MinWindow(val x: Int) : Window {
    override fun open() {
        print("This window has $x glasses")
    }
}

class BigWindow(delegate: Window) : Window by delegate

object Main {
    @JvmStatic
    fun main(args: Array<String>) {
      BigWindow(MinWindow(4)).open()
    }
}
```

理解：BigWindow 和 MinWindow 都继承 Window 接口，BigWindow 的实现通过 by 关键字，使用委托模式，委托 MinWindow 对象 delegate 来实现其继承 Window 的所有公共方法。

因此，这里会打印：This window has 4 glasses



# by 实现属性观察者模式

```kotlin
object Main {
    @JvmStatic
    fun main(args: Array<String>) {
        println(name)

        name = "Wang"
        name = "Zheng"
    }
}

var name: String by Delegates.observable("zhang", { property, oldValue, newValue ->
    println("property.name: ${property.name}, oldValue: $oldValue, newValue: $newValue")
})
```

理解：name 通过 by 关键字和属性代理接口，即 Delegates.observable 返回值  ReadWriteProperty<Any?, T> 实现属性的监听。初始值  "zhang"，一旦属性发生改变，action 方法（lambda 表达式）就会收到通知