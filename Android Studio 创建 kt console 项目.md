Android Studio 创建 kt console 项目

---

修改原纯 java 项目的 build.gradle：

```groovy
apply plugin: 'java'

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])

    testImplementation 'junit:junit:4.12'
}
```

为：

```
apply plugin: 'java'
apply plugin: 'kotlin'

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation 'org.openjdk.jol:jol-core:0.12'

    testImplementation 'junit:junit:4.12'
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk8:$kotlin_version"
}
buildscript {
    ext.kotlin_version = '1.4.20-RC'
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
    }
}
repositories {
    mavenCentral()
}
compileKotlin {
    kotlinOptions {
        jvmTarget = "1.8"
    }
}
compileTestKotlin {
    kotlinOptions {
        jvmTarget = "1.8"
    }
}
```



创建第一个 kt 文件：任意名字（eg. MyKt.kt）

```kotlin
object Main {
    @JvmStatic
    fun main(args: Array<String>) {
        print("helllo kt")
    }
}
```

AS 会在 mian 方法边上出现一个 run 的图标，可以点击选择直接运行，或者添加一个 kt 的运行配置