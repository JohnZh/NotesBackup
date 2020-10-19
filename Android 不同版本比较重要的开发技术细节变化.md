# Android 不同版本比较重要的开发技术细节变化

- 4.4：全屏沉浸式，提出 ART，但不是默认，还是 DVM 的 JIT 即时编译

  > JIT：just in time，把 dex/odex 翻译成原生指令码 smali 指令集，每次执行都翻译，无缓存

- 5.0：Material 工具包、ART Runtime：特性 AOT 预编译（安装时间增加、运行速度加快）

  > ART 对 JNI 开发影响比较大
  >
  > AOT：ahead of time，安装时预编译 dex 为 ELF，安装时间变长，但是运行时快，CPU 执行频率低（无 JIT）耗电降低
  >
  > GC 优化

- 6.0：运行时权限、移除对 Apache Http Client 支持

  > 使用 Apache Http Client  需要添加 android.useLibrary 'org.apache.http.legacy'

- 7.0：分屏，通知栏直接回复。混编：JIT + AOT

  > AOT：all of the time compilation 全时段编译
  >
  > 取消安装时预编译，JIT，并添加 jit code cache（内存缓存），并创建 profile 文件记录热点函数信息。在手机空闲和充电的时候，AOT 扫描 profile 文件对热点代码进行编译

- 8.0：添加通知渠道（通知类别），通知标志，通知超时，休眠，背景色，消息样式；自动填充框架；PIP（画中画）；自适应 icon；xml 对边属性

  > 对边属性：例如 layout_marginVertical 替代同时定义相同 layout_marginTop 和 layout_marginBottom

- 9.0：

  - WiFi RTT 室内定位
  - 屏缺口支持：DisplayCutout
  - 多摄像头支持：支持多个摄像头来访问多个视频流
  - 使用 ImageDecoder 取代 BitmapFactory & BitmapFactory.Options
  - Neural Networks API 1.1 机器学习
  - 系统代表您的应用提供生物识别身份验证对话框：目的是为了为生物识别提供可靠性

- 10：

  - 折叠屏支持
  - 5G 增强 API
  - 隐私：
    - 设备 ID 不可随便获取（防止设备跟踪）包括  IMEI，序列号和类似不可重置设备标识符
    - 分区存储
    - 禁止后台启动 Activity
    - 地理位置权限（用户可以允许应用仅在应用实际使用时（在前台运行）访问位置）
    - API 限制
  - 界面：
    - 手势导航
    - 全面屏优化
    - 深色主体
    - 通知栏优化：优先级、智能回复、建议操作
    - 分享 UI及 API
    - 新的长宽比
    - multi-resume：在 multi-window 状态下，所有位于顶层的用户可见并且可聚焦的 Activity 处于 Resume 状态。（被一个透明的 activity 覆盖、不可聚焦的不算）
    - system_alart_window 权限被废止
    - bubbles 交互
  - 系统：
    - TLS1.3 默认开启
    - BiometricPrompt 人脸、指纹等 API，生物识别登录
    - 更好的文字支持，更好的编解码器
    - ANGLE:OpenGL ES
    - 神经网络（Neural Networks） API 1.2
    - Google Play 推荐使用 Android App Bundle



# JetPack

