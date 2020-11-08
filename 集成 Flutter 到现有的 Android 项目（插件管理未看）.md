集成 Flutter 到现有的 Android 项目

---



# 项目配置

两个方式：

1. 使用 AS 直接创建 Flutter Module。setting.gradle / build.gradle 会自动添加相关配置
2. 自己使用 flutter 命令创建，然后导入（略）



# 添加到单个（常规） Flutter 页面

## 注册 FlutterActivity 到 AndroidManifest.xml（必须）

Flutter 使用 FlutterActivity 为 app 提供 Flutter 体验，因此这是必须的

```xml
<activity
  android:name="io.flutter.embedding.android.FlutterActivity"
  android:theme="@style/LaunchTheme"
  android:configChanges="orientation|keyboardHidden
                         |keyboard|screenSize|locale|layoutDirection
                         |fontScale|screenLayout|density|uiMode"
  android:hardwareAccelerated="true"
  android:windowSoftInputMode="adjustResize"
  />
```

`@style/LaunchTheme` 可以替换任意的 Android 主题。作用是为 FlutterActivity 设置基础颜色。只在 Flutter 渲染它自己颜色前有效



## 启动 FlutterActivity

注册了 FlutterActivity 后，可以在任意点启动它。Flutter 使用 route 表示页面，下为从 '/' 和 '/my_route' 页面启动的例子：

```java
// onClick 1
startActivity(
      FlutterActivity.createDefaultIntent(currentActivity)
    );
// onClick 2
startActivity(
      FlutterActivity
        .withNewEngine()
        .initialRoute("/my_route")
        .build(currentActivity)
      );
```

withNewEngine() 会在 FlutterActivity 内部创建一个 FlutterEngine 实例，这会带来耗时的初始化工作。

优化方案：预准备（pre-warmed），缓存 FlutterEngine，这能最小化 Flutter 初始化时间



## 使用 cached FlutterEngine（可选/推荐）

每个 FlutterActivity 默认都会创建自己的 FlutterEngine，FlutterEngine 的初始化很耗时，表现就是 Flutter UI 可见的延时。因此最后对 FlutterEngine 进行 pre-warmed。下面代码在自定义 Application 的 onCreate()

```java
// in MyApplication.onCreate()

// Instantiate a FlutterEngine.
flutterEngine = new FlutterEngine(this);

// Start executing Dart code to pre-warm the FlutterEngine.
flutterEngine.getDartExecutor().executeDartEntrypoint(
  DartEntrypoint.createDefault()
);

// Cache the FlutterEngine to be used by FlutterActivity.
FlutterEngineCache
  .getInstance()
  .put("my_engine_id", flutterEngine);
```

当使用 FlutterActivity 和 FlutterFragment 你应该使用缓存的 FlutterEngine：

```java
// onClick
startActivity(
      FlutterActivity
        .withCachedEngine("my_engine_id")
        .build(currentActivity)
      );
  }
```

> 注意：执行 executeDartEntrypoint 的时候，Dart 入口方法就会开始执行。当执行 runApp() 的时候，Flutter app 就好像在运行在一个 0 尺寸的 Window，直到 FlutterEngine 被附着到 FlutterActivity，FlutterFragment 或者 FlutterView 上。
>
> **确保 Flutter app 加热和显式内容的时间内 App 行为的恰当性**

> 使用 cached FlutterEngine 注意事项：FlutterEngine 的生命周期比任何显式它的 FlutterActivity/ FlutterFragment 都长。也就是说 Dart 代码在 pre-warmed 的时候就开始执行，并且在 FlutterActivity/FlutterFragment 毁灭后依然继续执行。
>
> **为了停止执行和清理资源，从 FlutterEngineCache 获取 FlutterEngine，使用 FlutterEngine.destroy() 毁灭它**

> 性能不是预热和缓存 FlutterEngine 的唯一原因。一个预热的 FlutterEngine 独立于 FlutterActivity 执行 Dart 代码。这意味着 FlutterEngine 可以被用于任意时刻执行任意的 Dart 代码，例如非 UI 逻辑的，像网络，数据缓存，Service 内部的后台行为。但使用 FlutterEngine 执行后台行为的时候，请准守Android 后台执行限制

> **注意：Flutter deubg/release build 有很大的性能区别。评估性能要使用 release build**



### 在预热和缓存 FlutterEngine 之前初始化 routes

FlutterActivity 和 FlutterFragment 不提提供路由初始化，但是一旦 FlutterEngine 预热，Dart 代码就会执行，之后再初始化路由就有点晚，因此

```dart
// Instantiate a FlutterEngine.
flutterEngine = new FlutterEngine(this);
// Configure an initial route.
flutterEngine.getNavigationChannel().setInitialRoute("your/route/here");
// Start executing Dart code to pre-warm the FlutterEngine.
flutterEngine.getDartExecutor().executeDartEntrypoint(
  DartEntrypoint.createDefault()
);
// Cache the FlutterEngine to be used by FlutterActivity or FlutterFragment.
FlutterEngineCache
  .getInstance()
  .put("my_engine_id", flutterEngine);
```

一旦 Dart 初步执行 runApp，FlutterEngine 就会显示期望的 route。

在执行了 runApp 之后改变初始化的 route 是无效的。在不同的 Activity 和 Fragment 使用相同的 FlutterEngine 并转变 route 需要设置一个 Method channel，并显式地指示 Dart 改变 Navigator routes



# 添加半透明的 Flutter 页面

场景：弹窗模式

## 设置 FlutterActivity 半透明主题

```xml
<style name="MyTheme" parent="@style/MyParentTheme">
  <item name="android:windowIsTranslucent">true</item>
</style>

<activity
  android:name="io.flutter.embedding.android.FlutterActivity"
  android:theme="@style/MyTheme"  	android:configChanges="orientation|keyboardHidden|keyboard|screenSize|locale|layoutDirection|fontScale|screenLayout|density|uiMode"
  android:hardwareAccelerated="true"
  android:windowSoftInputMode="adjustResize"
  />
```

## 透明模式启动 FlutterActivity

```java
startActivity(
  FlutterActivity
    .withNewEngine()
    .backgroundMode(FlutterActivityLaunchConfigs.BackgroundMode.transparent)
    .build(context)
);

// Using a cached FlutterEngine.
startActivity(
  FlutterActivity
    .withCachedEngine("my_engine_id")
    .backgroundMode(FlutterActivityLaunchConfigs.BackgroundMode.transparent)
    .build(context)
);
```

> 注意：Flutter 的内容也应该具有半透明的背景



# 添加 FlutterFragment 到 app

**如果 Activity 同样适用的，优先考虑 FlutterActivity，更快更简单**



## 添加 FlutterFragment 到宿主 Activity 伴随创建 FlutterEngine

如果只是和常规 Fragment 一样，在 Activity.onCreate 里面通过 getSupportFragmentManager 来添加，对于显示 UI（页面 route 是 '/'） 创建新的 FlutterEngine 是够了，但是要达到满足所有的Flutter 行为是不够的，Flutter 是基于宿主 Activity 到 FlutterFragment 的各种系统信号来工作的。因此，在 Activity 的生命周期里面，需要分别调用 FlutterFragment 响应的生命周期方法



同样的，FlutterFragment 会创建它自己的 FlutterEngine 对象，这也会耗时，也会产生短时间的空白。优化要使用预热和缓存 FlutterEngine



初始化 route 也要在pre-warmed 和 cached FlutterEngine 之前。同 FlutterActivity



## 展示闪屏

就算 pre-warmed FlutterEngine 被使用，Flutter 初步显示依然需要一些等待时间。为了提升 UE，Flutter 支持第一帧前的闪屏显示

参考：[splash screen guide](https://flutter.dev/docs/development/ui/advanced/splash-screen)



## 执行特定路由

使用 FlutterFragment.builder

```dart
// With a new FlutterEngine.
FlutterFragment flutterFragment = FlutterFragment.withNewEngine()
    .initialRoute("myInitialRoute/")
    .build();
```

## 从特定入口去执行

常规的 Flutter app 只有一个 Dart 入口：main()。但是可以定义其他入口

FlutterFragment 支持定义其他入口

```dart
FlutterFragment flutterFragment = FlutterFragment.withNewEngine()
    .dartEntrypoint("mySpecialEntrypoint")
    .build();
```

这个配置会导致执行 Dart 的入口变为：mySpecialEntrypoint()

> 注意：再已被预热缓存的 FlutterEngine 上做这些是无效的



## 控制 FlutterFragment 的渲染模式

FlutterFragment 支持 SurfaceView 和 TextureView 的渲染模式。

SurfaceView 是不能插入大 viewTree 之间的，要么最底下，要么最上面。另外，Android N 之前 SurfaceView 不能被执行动画，因为他们的布局和渲染和其余的 view 是不同步的。

基于上述的情况，可以使用 TextureView 替代  SurfaceView。（默认是 SurfaceView）

```dart
// With a new FlutterEngine.
FlutterFragment flutterFragment = FlutterFragment.withNewEngine()
    .renderMode(FlutterView.RenderMode.texture)
    .build();

// With a cached FlutterEngine.
FlutterFragment flutterFragment = FlutterFragment.withCachedEngine("my_engine_id")
    .renderMode(FlutterView.RenderMode.texture)
    .build();
```



## 半透明的 FlutterFragment

FlutterFragment 渲染默认是 SurfaceView 模式，并且非透明黑色背景。这是基于性能的考虑。**半透明的背景其实对性能来说有消极的影响**。不过还是有需要

> 注意，当使用 SurfaceView 用于渲染半透明的时候，它会设置它的 z-index 高于其他所有的 views，这意味着它会显示在所有的 views 之前。如果你需要介于 view 之间。用 TextureView 模式

```dart
// Using a new FlutterEngine.
FlutterFragment flutterFragment = FlutterFragment.withNewEngine()
    .transparencyMode(FlutterView.TransparencyMode.transparent)
    .build();

// Using a cached FlutterEngine.
FlutterFragment flutterFragment = FlutterFragment.withCachedEngine("my_engine_id")
    .transparencyMode(FlutterView.TransparencyMode.transparent)
    .build();
```



# FlutterFragment 和宿主 Activity 的关系

有些 app 使用 Fragment 作为全部的用户界面。使用 Fragment 控制系统的元素，例如状态栏，导航栏，方向，是合理的情况。

但是有些 app，Fragment 只是用于展示UI 的局部。比如，一个抽屉布局，一个视频播放器，一张卡片。这个时候，FlutterFragment 影响了同一个 Window下的系统元素是不合适的。

因此 shouldAttachEngineToActivity() 的作用就是控制 FlutterFragment 是否应该影响系统的 UI 元素

```dart
// Using a new FlutterEngine.
FlutterFragment flutterFragment = FlutterFragment.withNewEngine()
    .shouldAttachEngineToActivity(false)
    .build();

// Using a cached FlutterEngine.
FlutterFragment flutterFragment = FlutterFragment.withCachedEngine("my_engine_id")
    .shouldAttachEngineToActivity(false)
    .build();
```

默认是 true，会影响系统的元素

> 注意，有些插件也许期望或需要 Activity 的引用，禁用访问之前确保你使用的插件中没有需要 Activity 的



# 管理插件和依赖

