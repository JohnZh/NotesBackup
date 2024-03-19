# Thinking in Compose

1. Describe what, not how.	
2. UI elements are functions.
3. State controls UI.
4. Events controls State



# Compose elements

## Surface 

用于绘制背景和阴影的基本组件，类似于 CardView



## Column\Row\Box

三个基本标准布局元素

- Column 列式布局，类似线性布局的纵向
- Row 行布局，类似线性布局的横向
- Box



## LazyColumn/LazyRow

类似于  `RecyclerView` ，只渲染屏幕上可见内容，`LazyColumn` 不会像 `RecyclerView` 一样回收其子级

它会在您滚动它时发出新的可组合项，并保持高效运行，发出可组合项的成本相对较低

```Kotlin
LazyColumn(modifier = Modifier.fillMaxWidth()) {
    items(items = names) {name ->
        Greeting(name)
    }
}
```





## 修饰符 Modifier

```kotlin
Modifier.fillMaxSize() // 用内容填到最大尺寸，类似 Wrap_content
Modifier.fillMaxWidth() // 类似于layout_width = match_parent
Modifier.weight(1.0f) // 类似于 layout_weight = 1.0
```



# 状态改变 UI 更新

- 不能使用常规变量来控制 UI 的更新，Compose 通过调用可组合函数将数据转换为界面，使用新数据重新创建 UI 的过程叫重组

- 常规变量的更改在重组过程中是无法被观测到的

- 可组合函数可以按任意顺序频繁执行，因此，**不能以代码的执行顺序或该函数的重组次数为判断依据**
- ` State` 和 `MutableState` 是两个接口，它们具有特定的值，每当该值发生变化时，它们就会触发界面更新（重组）

错误案例：

```kotlin
fun Greeting() {
	var expanded = false // Don't do this! 这个无法触发重组
	val expanded = mutableStateOf(false) // Don't do this! 这类型的变量会被 Greating 订阅，发生变化会通知调用改变 UI
  ....
}
```

因为可组合函数可以按任意顺序频繁执行，重组可随时发生，状态会重新设置成 false



## 记住状态值 remember

```kotlin
fun Greeting() {
  val expanded = remember { mutableStateOf(false) }
}

ElevatedButton(onClick = { expand.value = !expand.value }) {
    Text(text = if (expand.value) "Show less" else "Show more")
}
```



### 属性委托 by

对于这种状态变量，每次都要写 `.value` 的情况，可以使用 by 来进行属性委托，可避免写 `.value`

```kotlin
var shouldShowOnboarding by remember { mutableStateOf(true) }
// 相当于导入了：
// import androidx.compose.runtime.getValue
// import androidx.compose.runtime.setValue

onClick = { shouldShowOnboarding = false }
```



## 保留状态 rememberSaveable

remember 函数**仅在可组合项包含在组合中时**起作用

旋转屏幕后，整个 activity 都会重启，所有状态都将丢失，发生任何配置更改或者进程终止时，remember 都会失效

使用 `rememberSaveable`，而不使用 `remember`。这会保存每个在配置更改（如旋转）和进程终止后保留下来的状态。

在使用 lazyColumn 的时候不可见的 item 改变状态从可见到不可见再到可见也会有丢失状态问题，使用 `rememberSaveable`

```kotlin
var shouldShowOnboarding by rememberSaveable { mutableStateOf(true) }
```





# 状态升级

在可组合函数中，被多个函数读取或修改的状态应位于共同祖先实体中，此过程称为**状态提升**



下例，通过状态提升和 callback 的方式实现了状态的跨页面（onBoardingScreen & Greetings）更新

```kotlin
@Composable
fun MyApp(modifier: Modifier = Modifier) {
    var shouldShowOnboarding by remember { mutableStateOf(true) }

    if (shouldShowOnboarding) {
        onBoardingScreen(onClick = { shouldShowOnboarding = false })
    } else {
        Greetings()
    }
}

@Composable
fun onBoardingScreen(modifier: Modifier = Modifier, onClick: () -> Unit) {
    Column(
        modifier = modifier.fillMaxSize(),
        verticalArrangement = Arrangement.Center,
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Text("Welcome to the Basics Codelab!")
        Button(
            modifier = Modifier.padding(vertical = 24.dp),
            onClick = onClick
        ) {
            Text("Continue")
        }
    }
}
```





# 动画

## animateDpAsState 

```kotlin
val extraPadding by animateDpAsState(
    if (expanded) 48.dp else 0.dp,
    animationSpec = spring(
        dampingRatio = Spring.DampingRatioMediumBouncy,
        stiffness = Spring.StiffnessLow
    )
)

Text(
    text = "Hello $name!",
    modifier = Modifier
        .weight(1.0f)
        .padding(bottom = extraPadding.coerceAtLeast(0.dp)), // 注意这步
)
```

- animateDpAsState 会让 extraPadding 的值不断改变，触发函数调用，重组 UI

- animationSpec 给动画添加弹簧效果，两个参数对应阻尼和钢性

- extraPadding.coerceAtLeast 会保证 padding 的值最小是 0 dp，不加这个会导致crash（也许值出现了 negative）



# 样式和主题

