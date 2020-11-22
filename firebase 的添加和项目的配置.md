firebase 的添加和项目的配置

---

# 环境要求

- targetsdk ~ 16
- com.android.tools.build:gradle 3.2.1 版或更高版本
- gradle ~4.1
- compilesdk ~28



# 项目、app、控制台

控制台：https://console.firebase.google.com/u/1/

可以创建多个项目，每个项目多个 app（ios，Android，web）



# Firebase 配置文件

- 下载 **google-services.json** 放到 app（模块）级别的目录下

- 项目级别 build.gradle 

  ```groovy
  buildscript {
  	repositories {
      // Check that you have the following line (if not, add it):
      google()  // Google's Maven repository
    }
    
    dependencies {
      // ...
      // Add the following line: 用于解析 json 文件
      classpath 'com.google.gms:google-services:4.3.4'  // Google Services plugin
    }
  }
  
  allprojects {
    // ...
    repositories {
      // Check that you have the following line (if not, add it):
      google()  // Google's Maven repository
      // ...
    }
  }
  ```

- app 级别的 build.gradle

  ```groovy
  apply plugin: 'com.android.application'
  // Add the following line:
  apply plugin: 'com.google.gms.google-services'  // Google Services plugin
  
  android {
    // ...
  }
  ```

- app 级别的依赖

  ```groovy
  dependencies {
    // ...
  
    // Import the Firebase BoM 使用了 bom 其他依赖可以不加版本号，但是 AS 不会提示 bom 的最新版本建议升级
    implementation platform('com.google.firebase:firebase-bom:25.12.0')
  
    // When using the BoM, you don't specify versions in Firebase library dependencies
  
    // Declare the dependency for the Firebase SDK for Google Analytics
    implementation 'com.google.firebase:firebase-analytics'
  
    // Declare the dependencies for any other desired Firebase products
    // For example, declare the dependencies for Firebase Authentication and Cloud Firestore
    implementation 'com.google.firebase:firebase-auth'
    implementation 'com.google.firebase:firebase-firestore'
  }
  ```



# Firebase 可用的 Lib（模块）

## AdMob

```
com.google.android.gms:play-services-ads
```

https://firebase.google.com/docs/admob/android/quick-start?authuser=1



## Cloud Message 推送

```
com.google.firebase:firebase-messaging
```

https://firebase.google.com/docs/cloud-messaging/android/client?authuser=1



## google 分析 

```
com.google.firebase:firebase-analytics	
```

https://firebase.google.com/docs/analytics/get-started?platform=android&authuser=1



初始 FirebaseAnalytics 实例对象用于记录事件：

```java
private FirebaseAnalytics mFirebaseAnalytics;
mFirebaseAnalytics = FirebaseAnalytics.getInstance(this);
```

记录 SELECT_CONTENT 事件例子

```java
Bundle bundle = new Bundle();
bundle.putString(FirebaseAnalytics.Param.ITEM_ID, id);
bundle.putString(FirebaseAnalytics.Param.ITEM_NAME, name);
bundle.putString(FirebaseAnalytics.Param.CONTENT_TYPE, "image");
mFirebaseAnalytics.logEvent(FirebaseAnalytics.Event.SELECT_CONTENT, bundle);

// kt
firebaseAnalytics.logEvent(FirebaseAnalytics.Event.SELECT_ITEM) {
    param(FirebaseAnalytics.Param.ITEM_ID, id)
    param(FirebaseAnalytics.Param.ITEM_NAME, name)
    param(FirebaseAnalytics.Param.CONTENT_TYPE, "image")
}
```

验证事件被记录发送。下面的命令会让 AS 的 logcat 显示事件

```
adb shell setprop log.tag.FA VERBOSE
adb shell setprop log.tag.FA-SVC VERBOSE
adb logcat -v time -s FA FA-SVC
```

### 调试事件（debugging event）实时

可以几乎实时的查看你记录的事件。

一般情况下事件记录的上传都是近乎一个小时左右再批量上传的（为了省电和减少网络数据的使用）。但是为了验证分析的实现上的正确性，debug 模式下数据是几乎实时的

启动 debug 模式：

```
adb shell setprop debug.firebase.analytics.app package_name
```

这个行为会一直存在直到你显示关闭它

```
adb shell setprop debug.firebase.analytics.app .none.
```

注意：debug 模式下记录的事件会排除在全部的分析数据之外，也不会在 bigquery 报告内

具体的报告看 **debugView**

### 记录事件 

注意，事件名称是大小写敏感的

事件的记录使用 logEvent api。firebase 提供了建议事件以及其可用的参数：参考

[`FirebaseAnalytics.Event`](https://firebase.google.com/docs/reference/android/com/google/firebase/analytics/FirebaseAnalytics.Event?authuser=1) & [`FirebaseAnalytics.Param`](https://firebase.google.com/docs/reference/android/com/google/firebase/analytics/FirebaseAnalytics.Param?authuser=1)文档

关于参数，可以添加到任何事件上

自定义参数：可以注册用于你的分析报告，在 [audience](https://support.google.com/firebase/answer/6317509?hl=en&ref_topic=6317489&authuser=1) 里一般可以作为过滤器使用，并且会应用到每份报告上。如果连接到了 bigquery 自定义参数也会被包含在导出到 bigquery 的数据中

VALUE 参数：通常用于累加一个关键指标，这个指标同样是关于一个事件的。例如收入，举例，事件，积分

#### 自定义事件

如果“建议事件”无法满足需要，可以使用自定义事件：

```java
Bundle params = new Bundle();
params.putString("image_name", name);
params.putString("full_text", text);
mFirebaseAnalytics.logEvent("share_image", params);

//kt
firebaseAnalytics.logEvent("share_image") {
    param("image_name", name)
    param("full_text", text)
}
```

#### 设置默认参数

默认参数会关联所有的事件。记录所有的事件都会带上他们。logEvent 的时候如果设置了和默认参数相同的参数，那么默认值会被覆盖

```java
Bundle parameters = new Bundle();
params.putString("level_name", "Caverns01");
params.putInt("level_difficulty", 4);
mFirebaseAnalytics.setDefaultEventParameters(parameters);

//kt
val parameters = Bundle().apply {
    this.putString("level_name", "Caverns01")
    this.putInt("level_difficulty", 4)
}
firebaseAnalytics.setDefaultEventParameters(parameters)
  
  // 清除默认参数
firebaseAnalytics.setDefaultEventParameters(null)
```

在 logcat 里面验证事件被记录，使用之前提及过的 adb 的三个命令，帮助快速验证事件被发生

#### 查看事件在 Dashboard 的 Events 下

Dashboard 的更新始终是周期性的，为了快速的测试，使用之前提及的 logcat 输出

### 设置用户属性

Ga 会自动记录用户属性，不需要其他代码去使其生效。如果需要收集额外的数据，每个项目可以配置 25 个不同的用户属性。注意属性名是大小写敏感的。不能使用Google 保留的用户属性：

- Age
- Gender
- Interest

```
firebaseAnalytics.setUserProperty("favorite_food", food)
```

### 追踪屏幕

GA 会自动追踪，手动设置使用 logEvent(FirebaseAnalytics.Event.SCREEN_VIEW)

```kotlin
Bundle bundle = new Bundle();
bundle.putString(FirebaseAnalytics.Param.SCREEN_NAME, screenName);
bundle.putString(FirebaseAnalytics.Param.SCREEN_CLASS, "MainActivity");
mFirebaseAnalytics.logEvent(FirebaseAnalytics.Event.SCREEN_VIEW, bundle);

//kt
firebaseAnalytics.logEvent(FirebaseAnalytics.Event.SCREEN_VIEW) {
    param(FirebaseAnalytics.Param.SCREEN_NAME, screenName)
    param(FirebaseAnalytics.Param.SCREEN_CLASS, "MainActivity")
}
```

### 设置用户 id

允许为使用这个 app 的单独的用户存储一个 userId，这是可选的。使用场为同一个用户生成一个唯一 id，然后将分析的数据和多个分析提供者一起分析，又或者公司内部的多个 app 的情况里这个用户的分析

注意：不要用那种可以标识出用户信息的东西做 id，比如 email，这很不安全

一般算法，为每个用户使用一个内部的 id号（公司内部认知），更好的情况，这个 id 号的 hash 值。

如果你只对同一个用户，同一个 app，同一个设备的事件感兴趣，使用 user_pseudo_id。这个值是分析自动生成的，为每个事件存储在 bigquery 里面

设置用户 id 的代码：

```
mFirebaseAnalytics.setUserId("123456");
```



## 崩溃分析：Crashlytics

```
com.google.firebase:firebase-crashlytics	
```

https://firebase.google.com/docs/crashlytics/get-started?platform=android&authuser=1



## 性能监控

```
com.google.firebase:firebase-perf
```

https://firebase.google.com/docs/perf-mon/get-started-android?authuser=1



## 远端配置：Firebase Remote Config

```
com.google.firebase:firebase-config
```

定义云端参数，并更新值，从而在本地依赖值改变 ui 或行为

https://firebase.google.com/docs/remote-config/use-config-android?authuser=1



## 用户搜索（索引 indexing）

```
com.google.firebase:firebase-appindexing
```

https://firebase.google.com/docs/app-indexing/android/app?authuser=1



