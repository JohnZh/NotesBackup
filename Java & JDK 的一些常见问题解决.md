Java & JDK 的一些常见问题解决



# 命令

## 查看当前终端的 java 版本

```shell
java -version
```



## 查看系统内所有 JAVA 版本和路径 

```shell
/usr/libexec/java_home -V
```



# 遇到的问题以及解决

## 终端的 Jdk 版本和 Android studio gradle 的版本不一样

引发的问题是在命令行里 gradlew  命令无法使用，或者遇到因为 java 版本跟不上的问题

1. 打开 AS 查看 Gradle 使用 JAVA_HOME，比如 /Applications/Android Studio.app/Contents/jbr/Contents/Home

2. 终端执行命令：

   ```shell
   export JAVA_HOME="/Applications/Android Studio.app/Contents/jbr/Contents/Home"
   ```

3. 一般第二步结束后就可以解决问题，如果不行就还需要把 JavaHome 加入 path

