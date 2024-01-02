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



