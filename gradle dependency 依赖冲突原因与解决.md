gradle 依赖冲突原因与解决

---



# Transitive Dependency

中文，传递依赖。

当你声明依赖，而这些依赖又有他们自己的依赖，而这些依赖的依赖就叫传递依赖（Transitive Dependency）



# 查看所有的依赖

```sh
./gradlew app:dependencies
```



# 依赖不加载传递依赖

所有的：

```groovy
configurations.all {
   transitive = false
}
dependencies {
  ...
}
```

单个依赖不加载传递依赖：

```groovy
dependencies {
  implementation('fr.avianey.com.viewpagerindicator:library:2.4.1.1@aar') {
    transitive = true
  }
}
```



# 版本冲突及 gradle 的策略

出现版本冲突的时候，gradle 的策略是一般处理结果会使用版本最高的，但是有时候会出现降级。这种处理策略一般发生在相同类型的依赖的时候，**不同类型的依赖或者当版本直接内容差距过大的时候就会发生冲突**，举个例子，



## 强制使用同一个版本

```groovy
// 组为 com.google.code.gson，而名不为 gson 的所有依赖版本使用 2.8.6
configurations.all {
  resolutionStrategy.eachDependency { details ->
    def requested = details.requested
    if (requested.group == 'com.google.code.gson') {
      if (!requested.name.startsWith("gson")) {
        details.useVersion '2.8.6'
      }
    }
  }
}
```



## exclude 移除特定的传递依赖

其他情况出现冲突（版本差距过大，而发生不兼容情况），使用 exclude 来移除特定的传递依赖



所有的

```groovy
configurations {
  all*.exclude group: 'org.hamcrest', module: 'hamcrest-core'
}
```

特定的某个

```kotlin
implementation('com.google.android.gms:play-services-maps:17.0.0') {
  exclude group: 'com.google.android.gms', module: 'play-services-base'
  exclude group: 'com.google.android.gms', module: 'play-services-basement'
}

implementation('io.branch.sdk.android:library:5.0.3@aar') {
  exclude module: 'answers-shim'
}
```

