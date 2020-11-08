加速 AS 构建

---

# 确保使用最新的 ide，sdk tools 和 gradle



# 创建开发的 flavor

```groovy
dev {
      // To avoid using legacy multidex when building from the command line,
      // set minSdkVersion to 21 or higher. When using Android Studio 2.3 or higher,
      // the build automatically avoids legacy multidex when deploying to a device running
      // API level 21 or higher—regardless of what you set as your minSdkVersion.
      minSdkVersion 21
      versionNameSuffix "-dev"
      applicationIdSuffix '.dev'
    }
```

​	

# 启动单变体同步（**Only sync the active variant** ）

AS:**Preferences > Experimental > Gradle** 默认启动



# 避免编译不必要的资源

```groovy
dev {
  minSdkVersion 21
  versionNameSuffix "-dev"
  applicationIdSuffix '.dev'

  // The following configuration limits the "dev" flavor to using
  // English stringresources and xxhdpi screen-density resources.
  resConfigs "en", "xxhdpi"

}
```



# 调试的 build 停用 Crashlytics

```groovy
buildTypes {
    debug {
      ext.enableCrashlytics = false
    }
```

还需要更改 Fabric，在运行时为 debug build 停用 Crashlytics

```kotlin
// Initializes Fabric for builds that don't use the debug build type.
Crashlytics.Builder()
        .core(CrashlyticsCore.Builder().disabled(BuildConfig.DEBUG).build())
        .build()
        .also { crashlyticsKit ->
            Fabric.with(this, crashlyticsKit)
        }
```

禁止 Crashlytics buildId

```groovy
buildTypes {
    debug {
      ext.alwaysUpdateBuildId = false
    }
}
```



# 调试使用静态的配置值

```groovy
int MILLIS_IN_MINUTE = 1000 * 60
int minutesSinceEpoch = System.currentTimeMillis() / MILLIS_IN_MINUTE

android {
    ...
    defaultConfig {
        // Making either of these two values dynamic in the defaultConfig will
        // require a full APK build and reinstallation because the AndroidManifest.xml
        // must be updated.
        versionCode 1
        versionName "1.0"
        ...
    }

    // The defaultConfig values above are fixed, so your incremental builds don't
    // need to rebuild the manifest (and therefore the whole APK, slowing build times).
    // But for release builds, it's okay. So the following script iterates through
    // all the known variants, finds those that are "release" build types, and
    // changes those properties to something dynamic.
    applicationVariants.all { variant ->
        if (variant.buildType.name == "release") {
            variant.mergedFlavor.versionCode = minutesSinceEpoch;
            variant.mergedFlavor.versionName = minutesSinceEpoch + "-" + variant.flavorName;
        }
    }
}
```



# 使用静态的依赖版本，包括 gralde 和 dependency

例如

```groovy
classpath 'com.android.tools.build:gradle:3.4.1'
```



# 启动离线模式

AS **Preferences > Build, Execution, Deployment > Gradle** 勾选 offline work



# 为自定义构建逻辑创建任务 



# 将图片转换为 WebP 格式 

与 PNG 相比，WebP 提供了更好的压缩。但是在解压的时候，cpu 使用率有小幅的上升。AS 提供了将图片转 WebP 的工具



# 停用 PNG 处理 

如果不想或无法把图片转为 WebP，每次 build 也可以停止自动图片压缩，从而提高 build 速度。AS3.0及其以上，针对 debug build 默认是停止图片压缩的，针对其他的 build 类型停止，使用下面代码：

```groovy
android {
    buildTypes {
        release {
            // Disables PNG crunching for the release build type.
            crunchPngs false // 发布的时候要改为 true
        }
    }

// If you're using an older version of the plugin, use the
// following:
//  aaptOptions {
//      cruncherEnabled false
//  }
}
```



# 启用构建缓存

android gradle 插件 2.3.0 以上默认打开。默认路径：~/.android/build-cache/

> 缓存：未打包的 aar 和 经过 dex 预处理的远程依赖项

清除 build 缓存，`./gradlew cleanBuildCache`

停用构建缓存，一般不停用，`gradle.properties` 文件`android.enableBuildCache=false`



# 使用增量注释处理器 



# 对构建进行性能剖析【未完成】

https://developer.android.com/studio/build/optimize-your-build#profile-command-line

