# kotlin collection transformation 集合转换

- map
- zip
- associate
- flatten
- string representation



# map: 元素变换

对每个元素进行转换，转好的元素构成一个新的集合，并返回新集合

- map、mapIndex

- mapNotNull、mapIndexedNotNull

  - 带过滤 null 值的 map

  - ```kotlin
    numbers.mapNotNull { if ( it == 2) null else it * 3 }
    numbers.mapIndexedNotNull { idx, value -> if (idx == 0) null else value * idx }
    
    // [3, 9]
    // [2, 6]
    ```

- 特别对 map 集合类型的：

  - mapKeys

  - mapValues

    ```kotlin
    numbersMap.mapKeys { it.key.uppercase() }
    numbersMap.mapValues { it.value + it.key.length }
    ```



# zip: 元素 -> Pair

两相等的 list 进行 zip，返回一个 Pair List

两个不相等的 list，返回 Pair List 取 size 最小值

```kotlin
val colors = listOf("red", "brown", "grey")
val animals = listOf("fox", "bear", "wolf")
val twoAnimals = listOf("fox", "bear")

colors zip animals // [(red, fox), (brown, bear), (grey, wolf)]
colors.zip(twoAnimals) // [(red, fox), (brown, bear)]
```



## zip 加变换

```kotlin
val colors = listOf("red", "brown", "grey")
val animals = listOf("fox", "bear", "wolf")

colors.zip(animals) { color, animal -> "The ${animal.replaceFirstChar { it.uppercase() }} is $color"}
// [The Fox is red, The Bear is brown, The Wolf is grey]
```



## unzip 逆操作

返回值是一个 Pair，元素是两个 List

```kotlin
val numberPairs = listOf("one" to 1, "two" to 2, "three" to 3, "four" to 4)
numberPairs.unzip() // ([one, two, three, four], [1, 2, 3, 4])
```



# associate 联合 用于构造 Map

## associateWith 元素转换构造 map 的 value

```kotlin
val numbers = listOf("one", "two", "three", "four")
println(numbers.associateWith { it.length })

// {one=3, two=3, three=5, four=4}
```



## associateBy 元素转换构造 map 的 key 和 value

遇到转变后 key 相同的情况，取最后一个元素的转变结果

```kotlin
val numbers = listOf("one", "two", "three", "four")

println(numbers.associateBy { it.first().uppercaseChar() })
// {O=one, T=three, F=four}。T=two 被丢弃

println(numbers.associateBy(keySelector = { it.first().uppercaseChar() }, valueTransform = { it.length }))
// {O=3, T=5, F=4}
```



## associate  

由于创建了 short-living pair 对象，请不要在性能要求很高的地方使用

```kotlin
val names = listOf("Alice Adams", "Brian Brown", "Clara Campbell")
println(names.associate { name -> parseFullName(name).let { it.lastName to it.firstName } })  

// {Adams=Alice, Brown=Brian, Campbell=Clara}

private fun parseFullName(name: String): FullName {
  val split = name.split(" ")
  return FullName(split[0], split[1])
}

private data class FullName(val firstName: String, val lastName: String)
```



# Flatten﻿ 变平数据，用于嵌套型数据的操作

用于集合的集合，eg. Set 的 List

```kotlin
val numberSets = listOf(setOf(1, 2, 3), setOf(4, 5, 6), setOf(1, 2))
println(numberSets.flatten())

// [1, 2, 3, 4, 5, 6, 1, 2]
```

## flatMap: 转变一个集合的元素成为另外一种集合，然后再变平

```kotlin
val containers = listOf(
  StringContainer(listOf("one", "two", "three")),
  StringContainer(listOf("four", "five", "six")),
  StringContainer(listOf("seven", "eight"))
)

// 先 map 再 flat
// 由于 StringContainer 不是集合，因此先把这种元素转变为集合，然后在 flat
println(containers.flatMap { it.values }) 

// [one, two, three, four, five, six, seven, eight]

data class StringContainer(val values: List<String>)
```



# String representation﻿ 集合格式化为 String

## joinToString

```kotlin
val numbers = listOf("one", "two", "three", "four")

println(numbers)         
println(numbers.joinToString())
// [one, two, three, four]
// one, two, three, four
```



### 添加分隔符，前缀，后缀

```kotlin
val numbers = listOf("one", "two", "three", "four")    
println(numbers.joinToString(separator = " | ", prefix = "start: ", postfix = ": end"))

// start: one | two | three | four: end
```



### 限制元素个数，并添加省略元素的表达式

```kotlin
val numbers = (1..100).toList()
println(numbers.joinToString(limit = 10, truncated = "<...>"))

// 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, <...>
```



### 自定义表达式，直接用 lambda

```kotlin
val numbers = listOf("one", "two", "three", "four")
println(numbers.joinToString { "Element: ${it.uppercase()}"})

// Element: ONE, Element: TWO, Element: THREE, Element: FOUR
```



## joinTo - 集合变 string 后并追加到其他 string 上

```kotlin
val listString = StringBuffer("The list of numbers: ")
numbers.joinTo(listString)
// The list of numbers: one, two, three, four
```



