# Android OS 进入 cmd 模式

## adb shell

如果输入命令发现权限问题（permission denied）切换为 root 用户：

```shell
su root
```



# 推送文件到手机

## adb push

```shell
adb push ~/Desktop/mono.txt /sdcard/clash/
```



# Android 11 无线调试

```shell
adb pair ipaddr:port
```

