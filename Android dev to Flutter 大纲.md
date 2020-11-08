代码具体参考：https://flutter.dev/docs/get-started/flutter-for/android-devs

# 声明式用户界面

相较于以往的用户界面如同对象操作一般，声明式用户界面会 rebuild 一遍它自己以及它的子 view 树



# View 和 Widget

Android 界面最小单元 View，Flutter 使用 Widget，Widget 两个状态：

- Stateless：运行时不可变
- Stateful：运行时可变。eg. http 请求得到数据更新 Widget



## 如何更新 Widget

关键在于创建一个 StatefulWidget，以及属于这个 Widget 的 State<StatefulWidget>，定义方法，方法内使用 setState() 改变状态，然后会导致这个 State.build 重新执行一次。



## 添加删除 Widget

不提供这样的支持，取代它的是在父 Widget 里（准确的说是父 Widget 的 state 里）定义方法，然后返回不同的 Widget



## 动画

使用 AnimationController 和 Ticker，前者用于控制暂停，停止，进度，反转。后者在垂直同步的时候发射信号，并产生一个线性的 0-1 的插值用于每一帧。开发者要做的就是创建一个或多个 animation，然后添加到 AnimationControler 上



## 画布绘制

CustomPaint 和 CustomPainter，后者实现绘制到画布的算法，且作为CustomPaint 的成员



## 自定义控件

不是采用继承，而是采用组合



# 意图 Intents

Intent 两个作用，Activity 之间导航，组件之间通信。

- Flutter 没有 intent
- Flutter 没有 Activity 和 Fragment 的等价物

使用 Navigator 和 Routes，前者管理 Routes，后者是 page 或者 screen 的抽象。

Navigator 的 push 和 pop 表示页面栈的进与退操作

取代 AndroidManifest 声明所有 Activity 的方式，Flutter 导航使用一对可选操作

- MaterialApp 使用 routes map
- WidgetApp 使用直接导航

**其他调用组件的方式使用插件进行原生混合开发**



## Futter 接收外部的 intent

Android 层 Activity 先接收，然后等待 Flutter 使用 MethodChannel 请求数据。



## startActivityForResult 效果

使用 Navigator push，前面加 await 操作，返回 Future 用于接收结果。在打开页面用 pop 退出栈同时返回结果：`pop({...})`



# 异步与 UI 

## runOnUiThread() 等价效果

Flutter 和 Android 一样是单线程模型，事件驱动，Loop 就如同 Android 的 Loop，添加到主线程，支持 Isolate。

但是 Flutter 并不像 Android 一样需要保证主线程不被阻塞。Dart 提供了异步工具，例如 async/await 来执行异步任务。

```dart
Future<void> loadData() async {
  String dataURL = "https://jsonplaceholder.typicode.com/posts";
  http.Response response = await http.get(dataURL); 
  setState(() {
    widgets = jsonDecode(response.body);
  });
}
```

请求完成，响应返回，会调用 setState，重建 view 树



## 在后台线程执行任务

没有类似 AsyncTask 的等价替换，用 async/await，或者 **Isolates 用于处理大量数据。**

Isolates 是多个独立线程，但是不共享主线程的 heap 内存，这意味着你不能访问主线程的变量也不能通过 setState() 更新 UI。

这表示 Isolates 是执行后需要回写到主线程，然后由主线程进行 setState()，更新 UI。（具体见 demo 里的 ReceivePort）



## okhttp 的等价替换

flutter 的 http package 虽然简单，但是没有 okhttp 的所有功能（有些需要自己实现），使用的时候要在 pubspec.yaml 文件里面添加 dependencies。



## 为运行比较久的任务添加进度效果

Android 里面使用 progressBar 来表示后台长任务的执行等待，**Flutter 使用 ProgressIndicator**。

根据状态在任务前显示，任务后消失或替换成其他 widget。通过 setState() 的调用，改变一个 boolean 值或条件，然后 rebuild view tree



# 项目结构 & 资源

## 不同分辨率图片资源文件的整理

区别于 Android 不同的密度放不同的问题夹，res/drawable-*，Flutter 放在 assets 文件夹，采用类似于 ios 的像素倍数方案，1x\2x\3x。虽然没有和 Android 一样的 dp 单位，但是也有自己的逻辑像素单位。

叫做 devicePixelRatio，代表实际物理像素在逻辑像素里面的比例

对应 Android 的不同密度文件夹如下：

| 密度文件夹 | 和像素比例 | 代表屏幕  | fluter 比例 | densityDpi |
| ---------- | ---------- | --------- | ----------- | ---------- |
| ldpi       | 1:0.75     | 240p      | 0.75x       | 120        |
| mdpi       | 1:1        | 320p      | 1x          | 160        |
| hdpi:      | 1:1.5      | 480p      | 1.5x        | 240        |
| xhdpi      | 1:2        | 720p      | 2x          | 320        |
| xxhdpi     | 1:3        | 1080p     | 3x          | 480        |
| xxxhdpi    | 1:4        | 1440p(2k) | 4x          | 640        |

Flutter 的 assets 文件夹是任意的， 并没有预定义位置，只需要最后在 pubspec.yaml 里面声明一下，flutter 会自动打包

Flutter之前版本原生资源和 Flutter 资源无法互相访问。**flutter beta2.0 开始，flutter 资源文件被存储在原生层的 asset folder 里面，并且可以使用 assetManager 访问**

```java
val flutterAssetStream = assetManager.open("flutter_assets/assets/my_flutter_asset.png")
```

**flutter beta2.0 开始，flutter 依然无法访问原生的资源文件，但是可以访问 assets**

加入资源和使用的例子：icon.png，文件夹名 images

```dart
images/icon.png
images/2.0x/icon.png
images/3.0x/icon.png

// pubspec.yaml 声明文件夹
assets:
	- images/icon.jpeg
    
// 使用 AssetImage 或者 是哟 Image widget
1. return AssetImage("images/icon.png");
2. return Image.asset("images/icon.png"); // in build() function
```



## 字符串和本地化

flutter 当前没有系统资源文件存放字符串，因此现在使用类来存放和访问字符串资源。

```dart
class Strings {
  static String welcomeMessage = "Welcome To Flutter";
}
// access
Text(Strings.welcomeMessage)
```

原生层的直接访问功能还在开发

**推荐使用  [intl package](https://pub.dev/packages/intl) 实现国际化和本地化功能**



## Gradle 文件的等价物

flutter 用自己的 build system 以及 Pub package manager。

flutter 项目中的 android 文件夹中的 gradle 文件只在添加原生依赖时候使用。

使用 pubspec.yaml 添加额外的 flutter 依赖。flutter packages 资源： [pub.dev](https://pub.dev/flutter/packages/)



# Activities & Fragments

## Activity 和 Fragment 的等价替换

Android 里面，Activity 代表 user 聚焦的事物，Fragment 代表行为或者一部分用户界面，帮助模块化代码，为大屏幕精致化用户界面。

flutter 使用 widget 替换他们两个。参考 [Flutter for Android Developers: How to design Activity UI in Flutter](https://blog.usejournal.com/flutter-for-android-developers-how-to-design-activity-ui-in-flutter-4bf7b0de1e48)

### 在 flutter 里设计 Activity UI [例子]



## 监听 Activity 生命周期

Android 里面同过生命周期 hooks 或者在 Application 里面注册 ActivityLifecycleCallbacks 来监听 Activity 的生命周期。

flutter 没有这些概念。flutter 使用 WidgetsBindingObserver 观察 hook 所有的生命周期事件，监听 didChangeAppLifecycleState() ，该方法会接收 state 回调

可被观察的生命周期：

- inactive：app 非活动状态，未接收到用户输入。该事件只对 ios，Android 无
- paused：app 对用户不可见，不响应用户输入，运行于后台，等价 onPause()
- resumed: 等价 onPostResume()
- suspending: 挂起状态。等价 onStop()。无 ios 的等价事件

> 文档：[`AppLifecycleStatus` documentation](https://api.flutter.dev/flutter/dart-ui/AppLifecycleState-class.html)

虽然 FlutterActivity 捕获了所有的 Activity 生命周期事件并发给了 Flutter 引擎，但是很多事件已经屏蔽了。因为 flutter 认为只有很少的情况需要监听那些屏蔽掉的状态。

使用那些状态进行资源的获取和释放请在原生端完成



# 布局

## LinearLayout 

Flutter 中使用 Row 或者 Column 可以达到一样的效果，分别是垂直方向和水平方向

参考： [Flutter for Android Developers: How to design LinearLayout in Flutter](https://proandroiddev.com/flutter-for-android-developers-how-to-design-linearlayout-in-flutter-5d819c0ddf1a)



## RelativeLayout 

组合 Column，Row 和 Stack widgets

参考：[StackOverflow](https://stackoverflow.com/questions/44396075/equivalent-of-relativelayout-in-flutter)



## ScrollView 

flutter 使用 ListView widget，flutter 里面 listview 又是scrollview又是listview



## flutter 处理 landscape 转变

FlutterView 处理这个配置变化，前提是 AndroidManifest.xml 里面有

```xml
android:configChanges="orientation|screenSize"
```



# 手势监听和 touch 事件处理

## 点击

如果widget 有 onPressed 参数，直接使用，如果没有，使用 GestureDetector 包装这个 widget，然后传入 GestureDetector onTap 参数



## 其他的手势

使用 GestureDetector：

- Tap
  - onTapDown 接触屏幕。可能会发生单击
  - onTapUp 停止接触（离开）屏幕。可能会触发单击
  - onTap 单击事件触发
  - onTapCancel 单击事件取消
- onDoubleTap 双击
- onLongPress 长按
- onVerticalDragStart 
- onVerticalDragUpdate
- onVerticalDragEnd
- onHorizontalDragStart
- onHorizontalDragUpdate
- onHorizontalDragEnd



# ListView 和 Adapters

ListView 在 Flutter 里面就是 ListView



## ListView Item 单击

直接用 GestureDetector 包装widgets 列表里面的单个 widget，然后传入 onTap 或其他参数



## 动态更新 ListView

同样是使用 setState，只是改变状态的对象不是 ListView，而是 widgets 列表。直接使用 ListView 并更新，需要重创建 Widget 列表，因此可能会涉及到 `List.from(widgets)` 数据拷贝。但是大量数据的情况下这样是效率低下的。

因此，使用 ListView.builder 来替代单纯使用 ListView 加载大量数据，builder 本质上就类似于 Android 里面 RecyclerView。ListView.binder 的 itemBuilder 参数就类似于 Android adapter 的 getView。关键是不再需要 rebuild ListView，不涉及到 Widget 列表的拷贝



# 文本

## 自定义字体

Android 里面，从 Android O 开始，需要创建字体资源文件，并传入到 TextView 的 FontFamily 参数中才能应用字体

Flutter，字体资源文件放入任意文件夹，然后 pubspec.yaml 声明，widget 里面引用它

```yaml
fonts:
   - family: MyCustomFont
     fonts:
       - asset: fonts/MyCustomFont.ttf
       - style: italic
```

使用

```dart
body: Center(
      child: Text(
        'This is a custom font text',
        style: TextStyle(fontFamily: 'MyCustomFont'),
      ),
    ),
```



## Text widget 的样式修改

Text 使用 TextStyle 对象作为样式参数，TextStyle 有许多可以配置的参数：

- color
- decoration
- decorationColor
- decorationStyle
- fontFamily
- fontSize
- fontStyle
- fontWeight
- hashCode
- height
- inherit
- letterSpacing
- textBaseline
- wordSpacing



# 表单输入

使用 TextField 作为输入控件

参考： [Retrieve the value of a text field](https://flutter.dev/docs/cookbook/forms/retrieve-input) 来自  [Flutter cookbook](https://flutter.dev/docs/cookbook)



## hint 

TextField 的 decoration 参数



## 显示合法性验证错误提示

使用 setState() 改变显示条件，decoration 参数的实参 InputDecoration() 定义 hintText 参数之外再需要定义 errorText，这会在状态改变rebuild Scaffold 的时候把 hint 改变



# Flutter 插件

## GPS 传感器

[geolocator](https://pub.dev/packages/geolocator)

## 摄像头

[image_picker](https://pub.dev/packages/image_picker)

## Facebook 登录

[flutter_facebook_login](https://pub.dev/packages/flutter_facebook_login)

## 使用 Firebase 功能

大多数 firebase 功能都包含在[first party plugins](https://pub.dev/flutter/packages?q=firebase)

## 自定义原生集成

如果Flutter或者社区插件缺少了原生平台特定的功能，你可以自己根据[developing packages and plugins](https://flutter.dev/docs/development/packages-and-plugins/developing-packages) 自己构建

Flutter 插件的架构，类似于 Android 的 EventBus，发射一个消息，然后让接受者进程发生一个结果给你。接受者是运行在 Android 或 ios 的原生层的代码

## NDK 使用

如果要 Flutter 使用 NDK，很可能需要构建你自己的插件，插件先和原生app 通信，然后通过 jni 调用 native，得到响应结果后再发送回 flutter，渲染结果。直接从 flutter 调用 native 代码当前不支持



# 主题

flutter 与生俱来实现了 Material 主题，不需要和 Android 一样在 xml 里面声明，然后在 AndroidManifest 里面引用，只需要在顶层 widget 里面声明主题 widget，MaterialApp。

也能使用 WidgetsApp 也提供了很多一样的功能，但是没有 MaterialApp 丰富

对任意的子组件声明颜色和和样式，使用 MaterialApp 的 theme 参数



# 数据库和本地存储

## Shared Preferences

使用 [Shared_Preferences plugin](https://pub.dev/packages/shared_preferences)，这个插件包装了 Shared Preferences 和 NSUserDefaults（IOS）

## SQLite

插件  [SQFlite](https://pub.dev/packages/sqflite)



# 调试

使用 [DevTools](https://flutter.dev/docs/development/tools/devtools) 套装调试 Flutter 或者 Dart apps

DevTools 功能：

- 分析检查 Heap

- 检查 widget tree
- log分析
- 调试，观察被执行的代码行数
- 调试内存泄露和内存碎片

参考文档：[DevTools](https://flutter.dev/docs/development/tools/devtools)



# 推送通知

Android 里面可以使用 Firebase Cloud Messaging 配置推送通知到 app

flutter，[Firebase Messaging](https://github.com/FirebaseExtended/flutterfire/tree/master/packages/firebase_messaging) 插件，参考文档：[firebase_messaging](https://pub.dev/packages/firebase_messaging)

