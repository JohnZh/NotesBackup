Kotlin 特性

---

# 表达式（控制流程）

## if 表达式

if 是个表达式，最后一个表达式作为返回值，kt 无三元运算符

```kotlin
val max = if (a > b) a else b
val max = if (a > b) {
    print("Choose a")
    a
} else {
    print("Choose b")
    b
}
```

作为表达式的时候 else 是强制的，作为语句（声明）不是强制的。when 也是如此



## when 替换 switch

```kotlin
when (x) {
    0, 1 -> print("x == 1 or x == 0")
    2 -> print("x == 2")
    else -> { // Note the block
        print("x is neither 1 nor 2")
    }
}
```

遇到满足的语句在结束。else 类似 java 的 default，作为表达式有范围值的时候是强制存在的，除非满足了所有情况肯定不会进入 else（eg. enum 和 seal）

### when 条件支持表达式和范围（in）

```kotlin
when (x) {
    parseInt(s) -> print("s encodes x")
    in 1..10 -> print("x is in the range")
    in validNumbers -> print("x is valid")
    !in 10..20 -> print("x is outside the range")
    else -> print("s does not encode x")
}
```

### when 作为表达式和函数和 is 一起使用

因此特性 when 经常替代 if-else 链使用

```kotlin
fun hasPrefix(x: Any) = when(x) {
    is String -> x.startsWith("prefix")
    else -> false
}
```

### when 代替 if-else

```kotlin
when {
  x.isOdd() -> print("x is odd")
  y.isEven() -> print("y is even")
  else -> print("x + y is even")
}
```

kt1.3 开始甚至可以用下面的语法

```kotlin
fun Request.getBody() =
        when (val response = executeRequest()) {
            is Success -> response.body
            is HttpError -> throw HttpException(response.status)
        }
```



## for 循环

迭代提供迭代器的任何东西，类似于 foreach

```kotlin
for (item in collection) print(item)
for (item: Int in ints) {
    // ...
}
```

> 提供迭代器的任何东西指
>
> 具备成员或者扩展函数 iterator()，其返回值假设是 it
>
> it 有成员或者扩展函数 next()
>
> it 有成员或者扩展函数  hasnext() 返回 Boolean
>
> 这三个函数需要被标记为 operator

### for 循环范围（in）

范围的循环会被编译成基于 index 的遍历，不会创建迭代器

```kotlin
for (i in 1..3) {
    println(i)
}
for (i in 6 downTo 0 step 2) {
    println(i)
}
```

使用 index 遍历 array 或者 list

```kotlin
for (i in array.indices) {
    println(array[i])
}

// 或者使用 withIndex() 
for ((index, value) in array.withIndex()) {
    println("the element at $index is $value")
}
```



## while 循环无变化

while 和 do-while 没什么变化

```kotlin
while (x > 0) {
    x--
}

do {
    val y = retrieveData()
} while (y != null) // y is visible here!
```

## 支持 break 和 continue 在循环中操作

[Returns and jumps](https://kotlinlang.org/docs/reference/returns.html)



# 返回和跳转

- return 默认从最近的封闭的函数和匿名函数返回
- break 接收最近的封闭的循环
- continue 下一次循环

可与更大的表达式连用，比如 elvis 表达式

```kotlin
val s = person.name ?: return
```



## break continue 和标签

在 kt 里面任何表达式都可以被标记为标签，格式是末尾跟着 @

```kotlin
loop@ for (i in 1..100) {
    for (j in 1..100) {
        if (...) break@loop
    }
}
```

break 跳到标记点不再进入 for 执行循环，如果是 continue 跳到标记继续进入 loop-i 进入下一次循环



## return 和 标签

正常方法中不管是在方法体还是方法内部的 lambda 表达式里面，return 都是直接返回最外层方法

如果要返回 lambda 表达式的执行起点就要配合标签使用

```kotlin
fun foo() {
    listOf(1, 2, 3, 4, 5).forEach lit@{
        if (it == 3) return@lit 
      // local return to the caller of the lambda, i.e. the forEach loop
        print(it)
    }
    print(" done with explicit label")
}
```

forEach 迭代，return 会到 lit 的位置继续迭代，所以打印了 1245

### return 方法的隐射标签

```kotlin
fun foo() {
    listOf(1, 2, 3, 4, 5).forEach {
        if (it == 3) return@forEach 
      // local return to the caller of the lambda, i.e. the forEach loop
        print(it)
    }
    print(" done with implicit label")
}
```

结果和之前的例子一样，返回 1245

### return 匿名函数

要达到一样的效果，可以使用

```kotlin
fun foo() {
    listOf(1, 2, 3, 4, 5).forEach(fun(value: Int) {
        if (value == 3) return  
        // local return to the caller of the anonymous fun, i.e. the forEach loop
        print(value)
    })
    print(" done with anonymous function")
}
```



### return 标签并带返回值

```kotlin
return@a 1
```



# 类和继承

类关键字 class，类的声明包含类头和类体（花括号包裹），头和体都是可选的

```kotlin
class Invoice { /*...*/ }
class Invoice // 无体类
```



## 构造器

类可以包含一个主构造器和多个次构造器，构造器是类头的一部分，关键字 constructor 可以被省略

```kotlin
class Person constructor(firstName: String) { /*...*/ }
class Person(firstName: String) {}
```

主构造器不包含任何代码，初始化使用 init 代码块。或者直接类体初始化。两者执行是交错的。主构造器的入参也可以使用在类体成员变量初始化里。

```kotlin
class InitOrderDemo(name: String) {
    val firstProperty = "First property: $name".also(::println)
    
    init {
        println("First initializer block that prints ${name}")
    }
    
    val secondProperty = "Second property: ${name.length}".also(::println)
    
    init {
        println("Second initializer block that prints ${name.length}")
    }
}
```

### 主构造器声明属性

主构造器可以直接声明类成员属性，并且可以是 val 或者 var，如果需要可见修饰符或注解，那么 constructor 关键字必须加，且修饰符和注解在 constructor 之前

```kotlin
class Person public @Inject constructor(val fisrtName: String, 
                                        val lastName: String,
                                        var age: Int
                                       ) {...}
```



## 次构造器

次构造器需要 constructor 为前缀

如果一个类有主构造器，那么次构造器需要代理主构造器，使用 this 关键字

init 模块以及字段初始化先于次构造器之前执行，就算是主构造器不存在的情况

```kotlin
class Person(val name: String) {
    var children: MutableList<Person> = mutableListOf<>()
  	init {
      println("before secondary constructor")
    }
    constructor(name: String, parent: Person) : this(name) {
        parent.children.add(this)
    }
}
```

非抽象类不声明主构造器，它也会有一个无参数的主构造器，可见性的 public。

如果不想要一个public 的主构造器，声明一个私有的主构造器：

```kotlin
class DontCreateMe private constructor () { /*...*/ }
```

注意，在 JVM 里面如果主构造器的所有参数都带默认值，编译器会生成一个额外的无参构造器并使用默认值。这使得 kotlin 在使用jackson 和 JPA 使用无参构造器创建类对象时候更加简单

```kotlin
class Customer(val customerName: String = "")
```



## 创建类的对象

kotlin 没 new 关键字，因此调用构造器创建对象就和调用常规函数一样

```kotlin
val invoice = Invoice()
val customer = Customer("Joe Smith")
```



### 嵌套的类和内部类及其实例化

内部类同样可以访问外部类，因为有外部类的引用

```kotlin
// 嵌套类
class Outer {
    private val bar: Int = 1
    class Nested {
        fun foo() = 2
    }
}
val demo = Outer.Nested().foo() // == 2

// 内部类
class Outer {
    private val bar: Int = 1
    inner class Inner {
        fun foo() = bar
    }
}
val demo = Outer().Inner().foo() // == 1
```

### 匿名内部类用 object 对象表达式

匿名内部类的创建用 object 表达式

```kotlin
window.addMouseListener(object : MouseAdapter() {

    override fun mouseClicked(e: MouseEvent) { ... }

    override fun mouseEntered(e: MouseEvent) { ... }
})
```

或者当接口就一个参数的时候，可以使用 lambada 表达式

```kotlin
var listener = ActionListenr {println("clicked")}
```



## 类成员

- 构造器和 init 块
- 函数
- 属性
- 嵌套类和内部类
- 对象声明



## 继承

kotlin 默认超类 `Any`，类似 Java Object。也有三个函数：equals，hashCode，toString。

默认 kotlin 所有类都是 final。不能被继承，如果想要被继承，用 open 关键字。

```kotlin
open class Base(p: Int)
class Derived(p: Int) : Base(p)
```

如果衍生类没有主构造器，那么次构造器就必须使用 super 关键字初始化基类型，或者代理另一个会做初始化基类的次构造器。不同的次构造器有能调用不同的基类构造器

```kotlin
class MyView : View {
    constructor(ctx: Context) : super(ctx)

    constructor(ctx: Context, attrs: AttributeSet) : super(ctx, attrs)
}
```



## 覆写函数

只有 open 的类并且标记 open 的函数可以被覆写。标记 override 的函数也是属于 open 的，如果要让它不可再被覆写，override 之前标记 final

```kotlin
open class Shape {
    open fun draw() { /*...*/ }
    fun fill() { /*...*/ }
}

class Circle() : Shape() {
    override fun draw() { /*...*/ }
}

open class Rectangle() : Shape() {
    final override fun draw() { /*...*/ }
}
```



## 覆写属性

衍生类也需要用 override 开头用于覆写属性，并且属性是要可兼容的，并且这个属性是可被初始化的可以有 get 函数 的

```kotlin
open class Shape {
    open val vertexCount: Int = 0
}

class Rectangle : Shape() {
    override val vertexCount = 4
}
```

val 可以被 var 覆写，但是反过来不行。因为 var 可以有 set，但是 val 没有。

可以在主构造器使用 override 关键字作为部分属性声明

```kotlin
interface Shape {
    val vertexCount: Int
}

class Rectangle(override val vertexCount: Int = 4) : Shape // Always has 4 vertices

class Polygon : Shape {
    override var vertexCount: Int = 0  // Can be set to any number later
}
```



## 派生类初始化顺序

基类在派生类之前初始化，应当避免 open 属性在构造器，init 块，和属性初始化行中

```kotlin
open class Base(val name: String) {

    init { println("Initializing Base") } 

    open val size: Int = 
        name.length.also { println("Initializing size in Base: $it") } 
}

class Derived(
    name: String,
    val lastName: String,
) : Base(name.capitalize().also { println("Argument for Base: $it") }) {

    init { println("Initializing Derived") }

    override val size: Int =
        (super.size + lastName.length).also { println("Initializing size in Derived: $it") }
}
```

output：

```kotlin
Constructing Derived("hello", "world")
Argument for Base: Hello
Initializing Base
Initializing size in Base: 5
Initializing Derived
Initializing size in Derived: 10
```



## 调用超类实现

派生类使用 super 关键字，派生类的内部类使用 super@Outer 访问外部类的超类

```kotlin
open class Rectangle {
    open fun draw() { println("Drawing a rectangle") }
    val borderColor: String get() = "black"
}

class FilledRectangle : Rectangle() {
    override fun draw() {
        super.draw()
        println("Filling the rectangle")
    }

    val fillColor: String get() = super.borderColor
}

class FilledRectangle: Rectangle() {
    fun draw() { /* ... */ }
    val borderColor: String get() = "black"
    
    inner class Filler {
        fun fill() { /* ... */ }
        fun drawAndFill() {
            super@FilledRectangle.draw() // Calls Rectangle's implementation of draw()
            fill()
            println("Drawn a filled rectangle with color ${super@FilledRectangle.borderColor}") // Uses Rectangle's implementation of borderColor's get()
        }
    }
}
```



## 覆写规则

类继承多个基类，基类有相同的 open 的成员函数，派生类必须提供自己对于这个 open 成员函数的覆写实现。

为了表示是从哪个超类继承实现的，使用 super<BaseTypeName> 的方式来表示

```kotlin
open class Rectangle {
    open fun draw() { /* ... */ }
}

interface Polygon {
    fun draw() { /* ... */ } // interface members are 'open' by default
}

class Square() : Rectangle(), Polygon {
    // The compiler requires draw() to be overridden:
    override fun draw() {
        super<Rectangle>.draw() // call to Rectangle.draw()
        super<Polygon>.draw() // call to Polygon.draw()
    }
}
```



## 抽象类

抽象成员可抽象，可以继承非抽象类并覆写其 open 成员为抽象的

```kotlin
open class Polygon {
    open fun draw() {}
}

abstract class Rectangle : Polygon() {
    abstract override fun draw()
}
```



## 伴生对象

如果你需要一个函数，调用它却不需要通过类对象，但是又需要访问这个类的内部（例如一个工厂函数），使用伴生对象，访问类成员却直接把类名作为修饰符



# 对象和伴生对象

## object 关键字

object 关键字，快速创建单例（类和对象）并且是线程安全的

```kotlin
object CarFactory {
    val cars = mutableListOf<Car>()
    
    fun makeCar(horsepowers: Int): Car {
        val car = Car(horsepowers)
        cars.add(car)
        return car
    }
}

// access
val car = CarFactory.makeCar(150)
println(CarFactory.cars.size)
```

## 伴生对象 Companion objects

静态函数或者属性关联到类的而不是对象的，在伴生对象里面声明

```kotlin
class Car(val horsepowers: Int) {
    companion object Factory {
        val cars = mutableListOf<Car>()

        fun makeCar(horsepowers: Int): Car {
            val car = Car(horsepowers)
            cars.add(car)
            return car
        }
    }
}

// accses
val car = Car.Factory.makeCar(105)
println(Car.Factory.cars.size)
```

### 伴生对象也是 object

伴生对象也是线程安全的，一个类只能有一个伴生对象，且多个伴生对象不可嵌套。

如果需要和 java 集成，类里的伴生对象也需要是真实静态，因此需要用 @JvmStatic 注解

伴生对象可以通过包含它的类的名称访问，但是包含它的类的对象不行

kotlin 也不支持类级别的函数被子类覆写

子类重新声明伴生对象只会吞噬父类的伴生对象

如果需要可覆写的类级别函数，就和常规静态开放函数一样（无法访问对象成员）



## object 对象表达式

```kotlin
interface Vehicle {
    fun drive(): String
}

fun start(vehicle: Vehicle) = println(vehicle.driv
```

使用对象表达式可以创建一个匿名类同时创建一个它的实例（匿名对象）：

```kotlin
start(object : Vehicle {
    override fun drive() = "Driving really fast"
})
```

如果父类型有构造器， 那么必须使用类型名加括号的形式调用构造。能定义多个父类型，但通常情况最多一个。

匿名对象没有名字，因此不能作为一个返回类型，如果一定要返回匿名对象，那么函数的返回值类型必须是 `Any`

尽管 object 关键字被使用，一个新的匿名类对象可以在任何时候被创造，只要对象表达式被评估



# 属性和字段

属性可以是 var 也可以是 val，直接通过名字访问。

get/set 语法，[] 代表可选

```kotlin
var <propertyName>[: <PropertyType>] [= <property_initializer>]
    [<getter>]
    [<setter>]
```

var 和 val 的区别，val 只能有 get，不能有 set

```
var allByDefault: Int? // 错误：需要显式初始化器，隐含默认 getter 和 setter
var initialized = 1 // 类型 Int、默认 getter 和 setter

val simple: Int? // 类型 Int、默认 getter、必须在构造函数中初始化
val inferredType = 1 // 类型 Int 、默认 getter
```



## 自定义 set 和 get

```kotlin
var stringRepresentation: String
    get() = this.toString()
    set(value) {
        setDataFromString(value) // 解析字符串并赋值给其他属性
    }
```

kt1.1 开始属性的类型可以从 get 函数推导出来的话就不用写

```kotlin
val isEmpty 
	get() = this.size == 0  // 具有类型 Boolean
```



## get set 改变可见性和添加注解

```kotlin
var setterVisibility: String = "abc"
    private set // 此 setter 是私有的并且有默认实现

var setterWithAnnotation: Any? = null
    @Inject set // 用 Inject 注解此 setter
```



## 幕后字段和幕后属性



## 编译器常量

只读字段在编译期就已经知道，使用 const 修饰。这种属性满足下面要求：

- 顶层或者Object 声明或者伴生对象的一个成员
- String 或原生类型初始化值
- 没有自定义 get

常量可以用于注解

```kotlin
const val SUBSYSTEM_DEPRECATED: String = "This subsystem is deprecated"

@Deprecated(SUBSYSTEM_DEPRECATED) fun foo() { …… }
```



## 延时初始化

通常非null 属性的的初始化应该在构造器里面完成，但是遇到属性初始化使用注解注入或者在单元测试的时候这就会显得不方便。当你不能在构造器里面做非 null 初始化却依然想要避免在 class 体内的非 null 检查使用 lateinit 修饰符

```kotlin
public class MyTest {
    lateinit var subject: TestSubject

    @SetUp fun setup() {
        subject = TestSubject()
    }

    @Test fun test() {
        subject.method()  // dereference directly
    }
}
```

lateinit 只能用于 var，且无自定义 get/set。kt1.2 后也用于顶层属性和本地变量。属性和变量必须非 null，且非基础类型。

未初始化前被访问会抛出异常。

### kt1.2 后检查是否被初始化

```kotlin
if (foo::bar.isInitialized) {
    println(foo.bar)
}
```



# 集合和集合操作

## 集合类型和关系

- Iterable（顶层）
- MutableIterable，Collection 继承 Iterable
- MutableCollection 继承 MutableIterable，Collection
- Collection 下两个子类：List，Set
- MutableIList，MutableISet 分别继承 MutableICollection、List 和 MutableICollection、Set
- Map 单独一个类型，子类为 MutableIMap



### 集合的多态

```kotlin
fun printAll(strings: Collection<String>) {
    for(s in strings) print("$s ")
    println()
}

val stringList = listOf("one", "two", "one")
val stringSet = setOf("one", "two", "three")
printAll(stringList)
printAll(stringSet)
```



### MutableCollection 的写操作（增删）

分别用了自定义扩展函数，filterTo 扩展函数， MutableICollection 减操作

```kotlin
fun List<String>.getShortWordsTo(shortWords: MutableList<String>, maxLength: Int) {
    this.filterTo(shortWords) { it.length <= maxLength }
    // throwing away the articles
    val articles = setOf("a", "A", "an", "An", "the", "The")
    shortWords -= articles
}

fun main() {
    val words = "A long time ago in a galaxy far far away".split(" ")
    val shortWords = mutableListOf<String>()
    words.getShortWordsTo(shortWords, 3)
    println(shortWords)
}
```



### List 两种索引方式

```kotlin
val numbers = listOf("one", "two", "three", "four")
println("Third element: ${numbers.get(2)}")
println("Fourth element: ${numbers[3]}")
```

### List 内容比较

List 只要 size 相同，元素数据结构和值相同就相同

```kotlin
val bob = Person("Bob", 31)
val people = listOf(Person("Adam", 20), bob, bob)
val people2 = listOf(Person("Adam", 20), Person("Bob", 31), bob)
println(people == people2) // true

bob.age = 32
println(people == people2) // false
```



### MutableList 提供 List 模式的操作

List 模式的操作，特定位置crud。kt 里面 List 默认实现就是 ArrayList，内部可变数组

```kotlin
val numbers = mutableListOf(1, 2, 3, 4)
numbers.add(5)
numbers.removeAt(1)
numbers[0] = 0
numbers.shuffle()
```



### Set 

元素唯一，通常无序，null 可以有，但也是唯一。size 相同每个元素相同的 set 相同

MutableSet 是 Set 基于 MutableCollection 的‘写操作’实现。

MutableSet 默认实现是 LinkedHashSet 因此顺序可预知：

```kotlin
val numbers = setOf(1, 2, 3, 4)  // LinkedHashSet is the default implementation
val numbersBackwards = setOf(4, 3, 2, 1)

println(numbers.first() == numbersBackwards.first()) // false
println(numbers.first() == numbersBackwards.last()) // true
```

另一个可选的实现是 HashSet，无序，内存使用更少



### Map

不继承 Collection，但是也是 kt 的集合类型

访问 keys 和 values

```kotlin
val numbersMap = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key4" to 1)
println("All keys: ${numbersMap.keys}")
println("All values: ${numbersMap.values}")
if ("key2" in numbersMap) println("Value by key \"key2\": ${numbersMap["key2"]}")    
if (1 in numbersMap.values) println("The value 1 is in the map")
if (numbersMap.containsValue(1)) println("The value 1 is in the map") // same as previous
```

**equals 的比较只管内容，不管顺序**

MutableMap  是 Map 的写操作实现，默认实现是 LinkedHashMap，意味着有序。可选实现是 HashMap

```kotlin
val numbersMap = mutableMapOf("one" to 1, "two" to 2)
numbersMap.put("three", 3)
numbersMap["one"] = 11

println(numbersMap)
```



## 集合操作

集合操作在标准库里分两种：成员函数（来自集合接口）、扩展函数

### 扩展函数（常见操作）

- 转换
- 过滤
- 加减
- 分组
- 检索集合的部分
- 检索单个元素
- 排序
- 聚合操作

下列操作**返回结果**，不改变原始集合：

```kotlin
val numbers = listOf("one", "two", "three", "four") // numbers 不变
val longerThan3 = numbers.filter { it.length > 3 } // result is stored in `longerThan3`
println("numbers longer than 3 chars are $longerThan3")
```

**定义目标对象**，集合操作会把结果追加到目标对象里面：

```kotlin
val numbers = listOf("one", "two", "three", "four")
val filterResults = mutableListOf<String>()  //destination object
numbers.filterTo(filterResults) { it.length > 3 }
numbers.filterIndexedTo(filterResults) { index, _ -> index == 0 }
println(filterResults) // contains results of both operations
// output：[three, four, one]
```

为了方便，可以定义目标对象同时还可以返回结果：

```kotlin
// filter numbers right into a new hash set, 
// thus eliminating duplicates in the result
val result = numbers.mapTo(HashSet()) { it.length }
println("distinct item lengths are $result")
// out: distinct item lengths are [3, 4, 5]
```



### 写操作

列表：[Write operations](https://kotlinlang.org/docs/reference/collection-write.html)

List 写操作：[List specific operations](https://kotlinlang.org/docs/reference/list-operations.html#list-write-operations)

Map 写操作：[Map specific operations](https://kotlinlang.org/docs/reference/map-operations.html#map-write-operations)

有一些函数改变集合原始数据，有些返回新结果：

```kotlin
val numbers = mutableListOf("one", "two", "three", "four")
val sortedNumbers = numbers.sorted() // four..one
println(numbers == sortedNumbers)  // false
numbers.sort()
println(numbers == sortedNumbers)  // true
```



## 范围和数列

`..` 操作符代表一个范围，代表 RangeTo()。in 和 !in 用于辅助 RangeTo() 函数

由于简单，看例子就可以明白：

```kotlin
if (i in 1..4) {  // 等同于 1 <= i && i <= 4
    print(i)
}

// 反向迭代
for (i in 4 downTo 1) print(i)

// 步长
for (i in 1..8 step 2) print(i)
for (i in 8 downTo 1 step 2) print(i)

// 开区间 i in [1, 10), 10被排除
for (i in 1 until 10) { 
    print(i)
}
```



### 定义'区间'并判断是否在区间内

```kotlin
val versionRange = Version(1, 11)..Version(1, 30)
println(Version(0, 9) in versionRange) // false
println(Version(1, 20) in versionRange) // true
```



## 数列

区间可视为数列，kt 的数列类型 IntProgression，LongProgression，CharProgression

`..`定义了first 和 last，但是迭代并不都会到 last

```kotlin
for (i in 1..9 step 3) print(i) // 最后一个元素是 7 
```

### 范围和集合函数连用

```kotlin
println((1..10).filter { it % 2 == 0 }) // [2,4,6,8,10]
println((1..10).map { it % 2 }) // [1,0,1,0,1,0,1,0,1,0]
```





# 扩展属性和扩展函数





# 作用域函数（清单）

作用域函数的唯一目的就是在对象的上下文环境中执行一代码块。作用域是指 lambda 表达式代码的区域

**let，run，with，apply，also**

作用域函数都非常相似（使用和语义），他它们两个主要区别：

- 引用上下文对象的方式
- 返回值



## 区别 this/it 根据引用上下文对象

在 lambda 表达式里面，上下文对象是作为一个接受者（this）还是一个lambda 表达式的参数（it）

通常 run，with，apply 会引用这个上下文对象作为一个接受者（this）

而 let，also 作为 lambda 表达式的参数

```kotlin
val adam = Person("Adam").apply { 
    this.age = 20                       // same as this.age = 20 or adam.age = 20
    this.city = "London"
}
println(adam)

val i = getRandomInt()
fun getRandomInt(): Int {
    return Random.nextInt(100).also {
        writeToLog("getRandomInt() generated value $it")
    }
}
// 当作为参数的时候，还可以自定义参数的名称
fun getRandomInt(): Int { value ->
    return Random.nextInt(100).also {
        writeToLog("getRandomInt() generated value $value")
    }
}
```



## 区别 this/it 根据返回值类型

also 和 apply 主要是作用于上下文本身，因此可以在同一个对象上进行链式调用：

```kotlin
val numberList = mutableListOf<Double>()
numberList.also { println("Populating the list") }
    .apply {
        add(2.71)
        add(3.14)
        add(1.0)
    }
    .also { println("Sorting the list") }
    .sort()
```

另外，还可以忽略返回值只用使用作用域函数再为变量创建一个临时区域

```kotlin
val numbers = mutableListOf("one", "two", "three")
with(numbers) {
    val firstItem = first()
    val lastItem = last()        
    println("First item: $firstItem, last item: $lastItem")
}
```



## 作用域函数一般模式

### let it

使用 it 作为上下文对象，返回值是 lambda 表达式的结果（最后一个语句）

```kotlin
val numbers = mutableListOf("one", "two", "three", "four", "five")
numbers.map { it.length }.filter { it > 3 }.let { 
    println(it)
    // and more function calls if needed
} 
// 甚至是
numbers.map { it.length }.filter { it > 3 }.let(::println) // :: 代表实现函数的对象
```

let 通常被用于非 null 的对象执行代码块，因此为了更安全的调用和 `?.` 一起使用。

```kotlin
val length = str?.let { 
    println("let() called on $it")        
    processNonNullString(it)      // OK: 'it' is not null inside '?.let { }'
    it.length
}
```

为了更好的可读性，使用新变量代替 it

```kotlin
val numbers = listOf("one", "two", "three", "four")
val modifiedFirstItem = numbers.first().let { firstItem ->
    println("The first item of the list is '$firstItem'")
    if (firstItem.length >= 5) firstItem else "!" + firstItem + "!"
}.toUpperCase()
println("First item after modifications: '$modifiedFirstItem'")
```



### with this

非扩展函数，上下文会作为一个参数传入进去，但是在 lambda 表达式里面，上下文作为一个Receiver（this），返回值是 lambda 最后的语句

```kotlin
val numbers = mutableListOf("one", "two", "three")
with(numbers) {
    println("'with' is called with argument $this")
    println("It contains $size elements")
}

// 或者是这样，省略了 this.
val firstAndLast = with(numbers) {
    "The first element is ${first()}," +
    " the last element is ${last()}"
}
println(firstAndLast)
```



### run this 和 just run

上下文对象作为一个 Receiver（this），返回值是 lambda 最后的语句。

run 做的事和 with 类似，但是调用像 let（类似上下文对象的扩展函数）

run 在 lambda 包含对象初始化和计算返回值的时候非常有用

```kotlin
val service = MultiportService("https://example.kotlinlang.org", 80)

val result = service.run {
    port = 8080
    query(prepareRequest() + " to port $port")
}

// the same code written with let() function:
val letResult = service.let {
    it.port = 8080
    it.query(it.prepareRequest() + " to port ${it.port}")
}
```

除了在 Receiver 上调用 run 这样的用法，还可以像一个非扩展函数一样使用它，目的是为了执行多个语句的代码块为了得到一个表达式

```kotlin
val hexNumberRegex = run {
    val digits = "0-9"
    val hexDigits = "A-Fa-f"
    val sign = "+-"

    Regex("[$sign]?[$digits$hexDigits]+")
}

for (match in hexNumberRegex.findAll("+1234 -FFFF not-a-number")) {
    println(match.value)
}
```



### apply this

上下文对象作为 Receiver，返回值是是上下文对象自己。

主要作用就是对当前的 Receiver 对象的成员进行操作，配置

```kotlin
val adam = Person("Adam").apply {
    age = 32
    city = "London"        
}
println(adam)
```



### also it

上下文对象作为参数（it），返回值是上下文对象自己

场景：需要上下文作为属性，或者不想屏幕来自外部作用域的 this 引用的时候

```kotlin
val numbers = mutableListOf("one", "two", "three")
numbers
    .also { println("The list elements before adding new one: $it") }
    .add("four")
```



### 总结和指南

| 函数    | 对象引用 | 返回值            | 是否是扩展函数             |
| :------ | :------- | :---------------- | :------------------------- |
| `let`   | `it`     | Lambda 表达式结果 | 是                         |
| `run`   | `this`   | Lambda 表达式结果 | 是                         |
| `run`   | -        | Lambda 表达式结果 | 不是：调用无需上下文对象   |
| `with`  | `this`   | Lambda 表达式结果 | 不是：把上下文对象当做参数 |
| `apply` | `this`   | 上下文对象        | 是                         |
| `also`  | `it`     | 上下文对象        | 是                         |

- 对一个非空（non-null）对象执行 lambda 表达式：`let`
- 将表达式作为变量引入为局部作用域中：`let`
- 对象配置：`apply`
- 对象配置并且计算结果：`run`
- 在需要表达式的地方运行语句：非扩展的 `run`
- 附加效果：`also`
- 一个对象的一组函数调用：`with`



### takeIf 和 takeUnless

标准库方法，条件成立和不成立的时候返回对象，因为会返回 null，因此在链式处理的时候应该进行 `?.`处理	

```kotlin
val number = Random.nextInt(100)

val evenOrNull = number.takeIf { it % 2 == 0 }
val oddOrNull = number.takeUnless { it % 2 == 0 }
println("even: $evenOrNull, odd: $oddOrNull")

// 和 ?. 连用
val caps = str.takeIf { it.isNotEmpty() }?.toUpperCase()
//val caps = str.takeIf { it.isNotEmpty() }.toUpperCase() //compilation error
println(caps)
```

和作用域函数连用的时候十分有用，从字符串中找一个子串并打印：

```kotlin
// 常规写法
fun displaySubstringPosition(input: String, sub: String) {
    val index = input.indexOf(sub)
    if (index >= 0) {
        println("The substring $sub is found in $input.")
        println("Its start position is $index.")
    }
}

// takeif + let
fun displaySubstringPosition(input: String, sub: String) {
  input.index(sub).takeif{ it >= 0 }?.let {
        println("The substring $sub is found in $input.")
        println("Its start position is $index.")    
  }
}
```



# 类型检查和转换

## is !is 运行时检查类型及智能转换

```kotlin
if (obj is String) {
    print(obj.length) // obj is automatically cast to String
}

if (obj !is String) { // same as !(obj is String)
    print("Not a String")
}
else {
    print(obj.length)
}
```

**智能转换**的核心是依然**编译器检测类型**，下面都是依赖编译器检测的例子

```kotlin
if (x !is String) return

print(x.length) // x is automatically cast to String

// x is automatically cast to string on the right-hand side of `||`
if (x !is String || x.length == 0) return

// x is automatically cast to string on the right-hand side of `&&`
if (x is String && x.length > 0) {
    print(x.length) // x is automatically cast to String
}
```



## 智能转换和 when 还有循环连用

```kotlin
when (x) {
    is Int -> print(x + 1)
    is String -> print(x.length + 1)
    is IntArray -> print(x.sum())
}
```



## 不安全的转型

使用 as 优先转型符，但是如果 y 是 null 那么会抛出异常，为了更安全的使用可以采用第二种写法

```kotlin
val x: String = y as String
var x: String? = y as String?
```

“非安全”转型符不等于 kt/js 的 unsafeCast<T>() 

对于可能为 null 的数据也可以使用**“安全”转型符** `as?` 失败的时候会返回 null

```kotlin
val x: String? = y as? String
```



## 类型擦除和泛型类型检查[待续]



# 异常

kt 所有异常都是 Throwable 的子类。包含信息，栈信息，可选原因。基本上和 java 一样



# Kotlin 调用 Java 代码





# kotline 样式

https://developer.android.com/kotlin/style-guide

