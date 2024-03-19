# NavController

获取 NavController 的三个方法：

- Fragment.findNavController()
- View.findNavController()
- Activity.findNavController(viewId: Int)

需要注意的是 NavController 是关联 NavHostFragment 的，无论用哪一个方法，viewId 都必须是 NavHostFragment 或者 NavHostFragment 是作为一个 parent 的。不然会抛出异常：IllegalStateException



# Navigation.createNavigateOnClickListener

还有一个 Navigation.createNavigateOnClickListener(@IdRes destId: int, bundle: Bundle) 这个会绑定一个 onClickListener 并且导航到destination，还可以追加一个 bundle 得参数

```kotlin
val button = view.findViewById<Button>(R.id.navigate_destination_button)
button?.setOnClickListener {
    findNavController().navigate(R.id.flow_step_one_dest, null)
}

// Navigation.createNavigateOnClickListener 例子
val button = view.findViewById<Button>(R.id.navigate_destination_button)
button?.setOnClickListener(
        Navigation.createNavigateOnClickListener(R.id.flow_step_one_dest, null)
)
```



# navOptions 和 动画

```kotlin
val options = navOptions {
    anim {
        enter = R.anim.slide_in_right // destination 进入动画，从 right 进入
        exit = R.anim.slide_out_left // 当前页面退出动画，从 left 退出
        popEnter = R.anim.slide_in_left // 点击 back 后，之前页面进入的动画，从 left 进入
        popExit = R.anim.slide_out_right // 当前页面退出的动画，从 right 出去
    }
}
view.findViewById<Button>(R.id.navigate_destination_button)?.setOnClickListener {
    findNavController().navigate(R.id.flow_step_one_dest, null, options)
}
```



# popUpTo

```kotlin
<fragment
    android:id="@+id/flow_step_two_dest"
    android:name="com.example.android.codelabs.navigation.FlowStepFragment">

    <argument
        .../>

    <action
        android:id="@+id/next_action"
        app:popUpTo="@id/home_dest">
    </action>
</fragment>
```

popUpTo 的效果：一直popup back-stack，直到推到 home_dest



# 传值的两个方法 



## 常规

使用 Bundle 配合 Navigation API

```kotlin
val bundle = bundleOf("amount" to amount)
view.findNavController().navigate(R.id.confirmationAction, bundle)

// 接收
val tv = view.findViewById<TextView>(R.id.textViewAmount)
tv.text = arguments?.getString("amount")
```



## Safe Args (recommend)

使用 gradle 插件，可以做到类型安全的传值，这也是官方推荐的

```groovy
// 项目的 build.gradle
buildscript {
    repositories {
        google()
    }
    dependencies {
        def nav_version = "2.7.7"
        classpath "androidx.navigation:navigation-safe-args-gradle-plugin:$nav_version"
    }
}

// app's build.gradle
plugins {
  	...
		id 'androidx.navigation.safeargs.kotlin'
  	...
}
```

传值的例子：从 MainMenuFragment -> 传值 -> SubMenuFragment

```xml
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/home_navigation"
    app:startDestination="@id/navigation_main_menu">

    <fragment
        android:id="@+id/navigation_main_menu"
        android:name="jp.co.kakuyasu.app.ui.home.MainMenuFragment"
        tools:layout="@layout/fragment_main_menu">

        <action
            android:id="@+id/action_main_menu_to_submenu"
            app:destination="@id/navigation_submenu"
            app:enterAnim="@anim/slide_in_from_right"
            app:exitAnim="@anim/fade_out"
            app:popEnterAnim="@anim/fade_in"
            app:popExitAnim="@anim/slide_out_to_right" />
    </fragment>

    <fragment
        android:id="@+id/navigation_submenu"
        android:name="jp.co.kakuyasu.app.ui.home.SubMenuFragment"
        tools:layout="@layout/fragment_submenu">

        <argument
            android:name="category"
            android:defaultValue='""'
            app:argType="string" />
    </fragment>

</navigation>
```

会自动生成类型安全的 **Direction** 类：

```kotlin
val action = MainMenuFragmentDirections.actionMainMenuToSubmenu(category.name)
findNavController().navigate(action)

// 接收
private val args: SubMenuFragmentArgs by navArgs()
val category = args.category
binding.titleBar.title = category
```



## 传值给 startDestination

两种方式，对应两种初始化方法

```kotlin
// 代码方式初始化 NavHost 
NavHostFragment.create(R.navigation.graph, args)

// 其他，比如 xml
navController.setGraph(R.navigation.graph, args)
navController.setGraph(navGraph, args)
```





# 和BottomNavigationView/Navigation Drawer/ActionBar 连用

```kotlin
private fun setupBottomNavMenu(navController: NavController) {
    val bottomNav = findViewById<BottomNavigationView>(R.id.bottom_nav_view)
    bottomNav?.setupWithNavController(navController)
}

private fun setupNavigationMenu(navController: NavController) {
    val sideNavView = findViewById<NavigationView>(R.id.nav_view)
    sideNavView?.setupWithNavController(navController)
}

// ActionBar 
val drawerLayout : DrawerLayout? = findViewById(R.id.drawer_layout)
appBarConfiguration = AppBarConfiguration(
        setOf(R.id.home_dest, R.id.deeplink_dest),
        drawerLayout)

// Up button
override fun onSupportNavigateUp(): Boolean {
    return findNavController(R.id.my_nav_host_fragment).navigateUp(appBarConfiguration)
}
```



# Deeplink

```kotlin
val args = Bundle()
args.putString("myarg", "From Widget");
val pendingIntent = NavDeepLinkBuilder(context)
        .setGraph(R.navigation.mobile_navigation)
        .setDestination(R.id.deeplink_dest)
        .setArguments(args)
        .createPendingIntent()

remoteViews.setOnClickPendingIntent(R.id.deep_link_button, pendingIntent)
```





# ProGuard considerations

如果需要 shrinking  code，要避免 Parcelable，Serializable，Enum 在缩小处理被混淆需要用：

- Use @Keep annotations.
- Use keepnames rules.

```kotlin
@Keep class ParcelableArg : Parcelable { ... }

@Keep class SerializableArg : Serializable { ... }

@Keep enum class EnumArg { ... }
```



## proguard-rules.pro

```json
...

-keepnames class com.path.to.your.ParcelableArg
-keepnames class com.path.to.your.SerializableArg
-keepnames class com.path.to.your.EnumArg

...
```



