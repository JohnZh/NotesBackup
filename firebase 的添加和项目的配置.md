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

需要安装 Google Play services



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

### 调试事件（debugView）实时 

可以几乎实时的查看你记录的事件。

一般情况下事件记录的上传都是近乎一个小时左右再批量上传的（为了省电和减少网络数据的使用）。但是为了验证分析的实现上的正确性，debug 模式下数据是几乎实时的

启动 debug 模式：

```shell
adb shell setprop debug.firebase.analytics.app package_name
```

eg.

```
adb shell setprop debug.firebase.analytics.app jp.co.mcdonalds.android.staging
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

### 控制台：Remote Config

https://console.firebase.google.com/u/1/project/mcd-dev-f605f/config

### 例子

```java
mFirebaseRemoteConfig = FirebaseRemoteConfig.getInstance();
FirebaseRemoteConfigSettings configSettings = new FirebaseRemoteConfigSettings.Builder()
  .setMinimumFetchIntervalInSeconds(3600)
  .build();
mFirebaseRemoteConfig.setConfigSettingsAsync(configSettings);
```

### 默认参数和获取

```
mFirebaseRemoteConfig.setDefaultsAsync(R.xml.remote_config_defaults);
// 获取的 api
getBoolean()
getByteArray()
getDouble()
getLong()
getString()
```

### 更新和激活

```java
 fetch() // 仅获取并存储在 RemoteConfigObject
 activate() // 使得参数可用

  //获取并激活可用
  mFirebaseRemoteConfig.fetchAndActivate()
  .addOnCompleteListener(this, new OnCompleteListener<Boolean>() {
    @Override
    public void onComplete(@NonNull Task<Boolean> task) {
      if (task.isSuccessful()) {
        boolean updated = task.getResult();
        Log.d(TAG, "Config params updated: " + updated);
        Toast.makeText(MainActivity.this, "Fetch and activate succeeded",
                       Toast.LENGTH_SHORT).show();

      } else {
        Toast.makeText(MainActivity.this, "Fetch failed",
                       Toast.LENGTH_SHORT).show();
      }
      displayWelcomeMessage();
    }
  });
```

### 节流

短时间内获取多次会抛出 FirebaseRemoteConfigFetchThrottledException

< 17.0.0 限制是 5 fetchs / 60min，新版限制更宽容

默认的 fetch 间隔是12小时，fetch 间隔的决定遵循以下规则：

- fetch(long)  方法参数
- FirebaseRemoteConfigSettings.setMinimumFetchIntervalInSeconds(long) 方法 参数
- 默认值 12 小时

### Remote Config 模板和版本

#### 模板文件

```json
  {
    "conditions": [
      {
        "name": "ios",
        "expression": "device.os == 'ios'"
      }
    ],
    "parameters": {
      "welcome_message": {
        "defaultValue": {
          "value": "Welcome to this sample app"
        },
        "conditionalValues": {
          "ios": {
            "value": "Welcome to this sample iOS app"
          }
        }
      },
      "welcome_message_caps": {
        "defaultValue": {
          "value": "false"
        }
      },
      "header_text": {
        "defaultValue": {
          "useInAppDefault": true
        }
      }
    },
    "version": {
      "versionNumber": "28",
      "updateTime": "2020-05-14T18:39:38.994Z",
      "updateUser": {
        "email": "user@google.com"
      },
      "updateOrigin": "CONSOLE",
      "updateType": "INCREMENTAL_UPDATE"
    }
  }
```

每次更新 parameters，都会创建一个新版本的 RC 模板并存储之前的模板为一个版本，这个版本你以后可以用于回退的需要。

版本号是有序增加（从初始值开始），所有模板都会有版本字段

**记住：RC 模板的生存周期是 90 天，总限制是300个版本。当前 App 使用的RC 模板不会过期。如果这个模板已经被激活超过 90 天，并且被更新的版本替换，它将无法在 retrieved（由于过期）**



#### 列出所有的RC 模板版本

控制台：Parameters tab，clock icon，Change history

也可以用 java 代码打出来

```java
ListVersionsPage page = FirebaseRemoteConfig.getInstance().listVersionsAsync().get();
while (page != null) {
  for (Version version : page.getValues()) {
    System.out.println("Version: " + version.getVersionNumber());
  }
  page = page.getNextPage();
}

// Iterate through all versions. This will still retrieve versions in batches.
page = FirebaseRemoteConfig.getInstance().listVersionsAsync().get();
for (Version version : page.iterateAll()) {
  System.out.println("Version: " + version.getVersionNumber());
}
```

数据结构：包含了模板元数据，更新时间，创建者，通过什么方式（console or REST API）

```json
{
  "versions": [{
    "version_number": "6",
    "update_time": "2018-05-12T02:38:54Z",
    "update_user": {
      "email": "jane@developer.org",
    },
    "description": "One small change on the console",
    "origin": "CONSOLE",
    "update_type": "INCREMENTAL_UPDATE"
  }]
```

#### 获取特定版本的模板

```java
Template template = FirebaseRemoteConfig.getInstance().getTemplateAtVersionAsync(versionNumber).get();
// See the ETag of the fetched template.
System.out.println("Successfully fetched the template with ETag: " + template.getETag());
```

控制台也可以看

#### 回退模板版本 (consolo 也可以)

```java
try {
  Template template = FirebaseRemoteConfig.getInstance().rollbackAsync(versionNumber).get();
  System.out.println("Successfully rolled back to template version: " + versionNumber);
  System.out.println("New ETag: " + template.getETag());
} catch (ExecutionException e) {
  if (e.getCause() instanceof FirebaseRemoteConfigException) {
    FirebaseRemoteConfigException rcError = (FirebaseRemoteConfigException) e.getCause();
    System.out.println("Error trying to rollback template.");
    System.out.println(rcError.getMessage());
  }
}
```



### Remote Config 加载策略

1. app 启动的时候 fetchAndActivate()，避免任何 ui 上引入注意的改变
2. 在 loading ui 的背后激活，使用这个策略建议加个超时机制，RC 默认是 1 分钟超时（对高质量 app 的启动来说这个时间太长了）
3. 加载新值下次使用：（这个策略等待时间是明显最少的，但是要看到变化必须启动两次 app）
   1. 启动，激活之前fetch 的值（瞬间的）
   2. 用户和 app 交换的同时获取新值（根据默认的 fetch 间隔）
   3. 直到下次app 启动，激活新值



### Remote Config 参数和条件（待看）

https://firebase.google.com/docs/remote-config/parameters?authuser=1



### RC 实时更新（待看）

https://firebase.google.com/docs/remote-config/propagate-updates-realtime?authuser=1#android_2



## 用户搜索（索引 indexing）

```
com.google.firebase:firebase-appindexing
```

https://firebase.google.com/docs/app-indexing/android/app?authuser=1



# Cloud Firestore

- Document Database，只存在 Documents 和 Collections，Collections 包含 Documents
- Documents 就想是一个类似于 map 的集合，可以存储 key-value 信息
  - value 可以是 String，Number，Json-Object/Array
- Documents 不能包含 Documents，但是可以指向 Collections，即 Subcollection，而其又可以包含其他 Documents
-  当获取一个 Document 的数据的时候，只是这个 Document 的数据，不包含其指向的 subcollection
- FirebaseRoot 只能包含 Collections
- 

