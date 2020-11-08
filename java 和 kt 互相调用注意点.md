java 与 kotlin 互相调用

---

# java 调用 kt object 对象的静态函数

函数上需要添加 @JvmStatic 的注解

```kotlin
@JvmStatic
val users: List<User>
	get() = _users

@JvmStatic
fun addUser(user: User) {
  // Ensure the user isn't already in the collection.
  val existingUser = users.find { user.id == it.id }
  existingUser?.let { _users.remove(it) }
  // Add the user.
  _users.add(user)
}
```



# java 调用 kt 扩展函数

kt 的类名是根据文件名来的， StringUtils.kt 类名会被编译成  StringUtilsKt，java 可以直接以 StringUtilsKt 为前缀调用其扩展函数，也可以使用注解进行重命名

```kotlin
@file:JvmName("StringUtils")

package com.google.example.javafriendlykotlin
```



# kt 支持 Java 的函数重载

需要 @JvmOverloads 和 constructor 关键字，@JvmOverloads 会创建 java 可以调用的所有构造重载

```kotlin
data class User @JvmOverloads constructor(
    val id: Int,
    val username: String,
    val displayName: String = username.toTitleCase(),
    val groups: List<String> = listOf("guest")
)
```



# kt 对象属性的 get 函数支持 java 改名调用

使用注解 @JvmName 在属性之后，或者使用 @get:JvmName 在属性之前修改其 get 函数的名字

```kotlin
val hasSystemAccess
   @JvmName("hasSystemAccess")
   get() = "sys" in groups

@get:JvmName("hasSystemAccess")
val hasSystemAccess
   get() = "sys" in groups
```



# java 直接访问 kt 对象的属性

使用 @JvmField 修饰 kt 的对象就可以跳过生成 get 和 set，允许 java 直接访问属性字段

```kotlin
@JvmField val id: Int

// java
user.id
```



# java 访问 kt 对象的 object 对象的静态字段

使用 `const` 关键字的性能好与 @JvmField，但是 const 只能修饰原始类型，例如 int，float，String

在能使用 const 的代替 @JvmField 的时候一定要用

```kotlin
const val BACKUP_PATH = "/backup/user.repo"
```



# kt 没有 checked 异常但支持抛出异常给 java

使用 @Throws 注解告诉调用的 java 这个函数会抛出什么类型的异常

```kotlin
@JvmStatic
@Throws(IOException::class)
fun saveAs(path: String?) {}
```

不过这不会改变 kt 原来的行为，即在 kt 里面调用这个函数，依然不是强制要求进行 try-catch 的



# kt 在调用 java 的时候其他一些注意点

1. java/kt 尽量不要使用 kt 的硬关键字作为函数，从 kt 调用的时候硬关键字需要使用 反引号 "`" 包起来。软关键字，修饰符关键字，特殊标识符都允许

```kotlin
val callable = Mockito.mock(Callable::class.java)
Mockito.`when`(callable.call()).thenReturn(/* … */)
```



2. 避免使用 Any 的扩展函数或属性名称
3. 公共 API 中的每个非基元参数类型、返回类型和字段类型都应具有可为 null 性注释。不带注释类型会被理解为‘平台’类型
4. Lambda 参数位于最后
5. java 类的属性函数前缀必须准确 get/set/is，为了在 kt 中被识别为属性
6. kt 里如果要使用运算符重载，那么 java 的函数必须符合命名规则



# java 在调用 kt 的时候的其他注意点

1. 当 kt 类包含顶级函数或者属性应始终使用@file:JvmName("...") 提供一个合适的名字（不然就变成了...Kt，显示了技术细节，又没意义）
2. 可以考虑使用 @file:JvmMultifleClass 将多个文件的顶级成员组合到一个类里
3. 

