- Kotlin Annotation vs Java Annotation
- Kotlin APT

# Kotlin Annotation

## 元注解

相比 Java 有类似的元注解

```kotlin
@Target
@Retention
@Repeatable // allows using the same annotation on a single element multiple times
@MustBeDocumented // specifies that the annotation is part of the public API and should be included in the class or method signature shown in the generated API documentation
```

Sample

```kotlin
@Target(AnnotationTarget.CLASS, 
		AnnotationTarget.FUNCTION, 
		AnnotationTarget.TYPE_PARAMETER, 
		AnnotationTarget.VALUE_PARAMETER, 
		AnnotationTarget.EXPRESSION) 
@Retention(AnnotationRetention.SOURCE) 
@MustBeDocumented 
annotation class Fancy

@Fancy // AnnotationTarget.ClASS
class Foo {

	@Fancy // AnnotationTarget.FUNCTION
	fun baz(@Fancy foo: Int): Int { // AnnotationTarget.VALUE_PARAMETER
		 return (@Fancy 1) } 
	 }
```

Source code，可见和 Java 大同小异

```Kotlin
public enum class AnnotationTarget {  
    /** Class, interface or object, annotation class is also included */  
    CLASS,  
    /** Annotation class only */  
    ANNOTATION_CLASS,  
    /** Generic type parameter */  
    TYPE_PARAMETER,  
    /** Property */  
    PROPERTY,  
    /** Field, including property's backing field */  
    FIELD,  
    /** Local variable */  
    LOCAL_VARIABLE,  
    /** Value parameter of a function or a constructor */  
    VALUE_PARAMETER,  
    /** Constructor only (primary or secondary) */  
    CONSTRUCTOR,  
    /** Function (constructors are not included) */  
    FUNCTION,  
    /** Property getter only */  
    PROPERTY_GETTER,  
    /** Property setter only */  
    PROPERTY_SETTER,  
    /** Type usage */  
    TYPE,  
    /** Any expression */  
    EXPRESSION,  
    /** File */  
    FILE,  
    /** Type alias */  
    @SinceKotlin("1.1")  
    TYPEALIAS  
}
```

如果需要在主构造器上使用，那么需要使用关键字 `constructor`，也可以用在属性访问上

```Kotlin

class Foo @Inject constructor(dependency: MyDependency) { }

class Foo { 
	var x: MyDependency? = null 
		@Inject set 
}
```


## 注解构造器

不同于 Java 在注解里面声明属性来添加注解声明时候的参数，Kotlin 的注解有注解构造器
```Kotlin

// java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Table {
    public String name() default "";
}

@Table(name = "Users")
class Users {
}

// kotlin
annotation class Special(val why: String)

@Special("example") 
class Foo {
}

```

> 注解参数不能有允许 null 的类型，类似 Java 默认值不能使用 null

如果注解的某个参数类型也是也是注解，那么参数是不需要 @ 符号的

```Kotlin
annotation class ReplaceWith(val expression: String) 

annotation class Deprecated( val message: String, val replaceWith: ReplaceWith = ReplaceWith("")) 

@Deprecated("This function is deprecated, use === instead", ReplaceWith("this === other"))
```

如果注解需要一个 Class 类型作为参数，使用 Kotlin Class，即 KClass

```Kotlin

import kotlin.reflect.KClass annotation 

class Ann(val arg1: KClass<*>, val arg2: KClass<out Any>) 

@Ann(String::class, Int::class) class MyClass

```

## Kotlin 的注解可以实例化

>https://github.com/Kotlin/KEEP/blob/master/proposals/annotation-instantiation.md

## 注解用于 Lambdas 表达式

```kotlin
annotation class Suspendable

val f = @Suspendable { Fiber.sleep(10) }
```



## 注解使用目标

在 kt 的构造方法里面去注解一些 JAVA 的属性

```Kotlin
class Example(
	@field:Ann val foo, // annotate Java field 
	@get:Ann val bar, // annotate Java getter 
	@param:Ann val quux) // annotate Java constructor parameter
```



注解整个文件

```Kotlin
@file:JvmName("Foo") 

package org.jetbrains.demo
```



多个注解，同个目标

```Kotlin
class Example { 
	@set:[Inject VisibleForTesting] // Inject and VisibleForTesting are annotations
	var collaborator: Collaborator 
}
```



### 支持目标的完整列表

- file

- Property 该注解的目标对 Java 是不可见的

- field

- get 属性的 getter

- set 属性的 setter

- receiver 一个扩展方法或属性的 receiver 参数

  ```kotlin
  fun @receiver:Fancy String.myExtension() { ... }
  ```

- param 构造器参数

- setparam 属性 settter 的参数

- delegate 存储代理属性的代理实例



如果不明确制定使用目标，会根据 @Target 来选择目标，有多个适用，则按照下面顺序优先适配

- param
- property
- field



## Java 注解

100 % 兼容 kotlin 注解

```kotlin
class Tests {
    // apply @Rule annotation to property getter
    @get:Rule val tempFolder = TemporaryFolder()

    @Test fun simple() {
        val f = tempFolder.newFile()
        assertEquals(42, getTheAnswer())
    }
}
```



### Java 注解的传参

```kotlin
// Java
public @interface Ann {
    int intValue();
    String stringValue();
}

// Kotlin
@Ann(intValue = 1, stringValue = "abc") 
class C

// Java
public @interface AnnWithValue {
    String value();
}

// Kotlin
@AnnWithValue("abc")
class C

// Java 数组参数
public @interface AnnWithArrayValue {
    String[] value();
}

// Kotlin
@AnnWithArrayValue("abc", "foo", "bar") 
class C

// Java 数组参数带参数名称
public @interface AnnWithArrayMethod {
    String[] names();
}

@AnnWithArrayMethod(names = ["abc", "foo", "bar"])
class C
```



### 访问注解实例的值

```kotlin
// Java
public @interface Ann {
    int value();
}

// Kotlin
fun foo(ann: Ann) {
    val i = ann.value
}
```



### Android API < 26 的问题

避免生成 `TYPE_USE` 和 `TYPE_PARAMETER` 注解目标，使用新的编译器参数：

```shell
-Xno-new-java-annotation-targets
```



### 可重复注解

https://kotlinlang.org/docs/annotations.html#repeatable-annotations

