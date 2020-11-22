## 个人信息

- 张栩恺 / 男 / 1990
- 本科 / 软件工程 / 杭州电子科技大学 / 12年“校优秀毕业生”
- 手机：13567124531
- 邮箱：kk24@qq.com
- 博客：[https://johnzh.github.io](https://johnzh.github.io)
        & [https://blog.csdn.net/wyzxk888](https://blog.csdn.net/wyzxk888)
- 项目总列表：[https://www.linkedin.com/in/johnzh1](https://www.linkedin.com/in/johnzh1)

## 工作经历

- 2019.6 - 2020.10 / 独立开发
	- 独立进行 app 开发工作。并独立创作发布了「[Program Timer](https://johnzh.github.io/programTimer)」中英文版
- 2019.1 - 2019.6 / 杭州沙曼网络科技有限公司 / 技术合伙人
	- 「遇见女神」系列 SNS App，项目经理，Android 开发
- 2015.5 - 2018.12 / 浙江松柏信息技术有限公司 / Android 组长 / 客户端主管
	- 负责「财牛投资」「期货交易通」「乐米金融」「未来交易所」等项目的 App 端开发与管理
- 2011.7-2015.4 / 杭州美源信息技术咨询有限公司（美资）/ 实习 / 软件开发
	- 负责「Lompang」「Revel」「Tourism Victoria」等 Android App 的开发

## 个人能力

- 精通 Java，熟悉 Koltin，熟悉 MVC，MVP，MVVM 架构模式以及常用设计模式

- 熟悉 Android Framework 层常用技术及原理，如 View 布局测量绘制，Touch 事件分发，Handler 消息机制，AIDL 进程间通信，对 Binder 通信也有一定了解

- 熟悉自定义控件的开发，常见问题分析与解决，对性能优化也有一定经验

- 熟悉多线程编程，对锁的原理，JMM线程模型都有深入了解，阅读过 ReentrantLock、ConcurentHashmap、线程池等与并发编程的有关源码

- 熟悉网络编程，熟悉 TCP/HTTP/HTTPS 等协议，阅读过 Volley，OkHttp 源码熟悉其架构设计思想及许多底层实现细节

- 熟悉组件化开发项目架构，基于 APT 手写过路由（模块）组件

- 熟悉 Jetpack 最新组件 LiveData，ViewModel，Databinding 了解其原理

- 熟悉常用第三方框架或工具，了解其原理，如 Retrofit，EventBus，GreenDao，BufferKnife，LruCache，DiskLruCache等，具备通过源码解决问题的能力

- 了解热修复及插件化技术的原理

- 了解 Flutter 架构和底层渲染树原理

- 熟悉 Sketch 设计软件，可以自己分析需求，画设计稿，并设计制作图标和切图

  
  
  



## 项目举例

### MyKline / 开源项目：可扩展可定制的 K 线控件

为了满足 k 线控件的指标可添加，样式可重写，绘制布局可改动的需求而写的 k 线控件

[Github：https://github.com/JohnZh/MyKline](https://github.com/JohnZh/MyKline)


### 未来交易所：BitFuture

数字货币交易所，分为 5 个模块，分别为首页：包括最新上的数字货币标的，涨跌幅榜；行情：包括基于几个基准货币的不同币种的行情列表，以及点击进入详细行情；法币交易：基于 p2p 的 usdt 与法币的交易模块；币币交易：类似于股票交易的限价和挂价的快速交易功能，当前订单查看功能；我的，个人信息，包含了资产，交易记录，订单，系统消息等功能

**项目职责：**

- 项目进度任务管理，参与需求分析评估
- 基础框架模块，公共模块的迁移及改善。例如
	- 网络层添加相同 url 不同验证码图片的需求添加无缓存图片下载
	- 网络层添加占位符式参数的 url 支持
	- 修改统一启动器，添加单 Activity 加载不同 Fragment 的显示模式
	- 基于适配器模式封装弹框，分离弹窗的控制逻辑，UI以及业务逻辑
	- 通用的可定制的行控件，头部控件，可添加多个 HEAD 和 FOOT 的 RecyclerView 
	- 基于 Websocket 框架，对 IM，交易，推送功能涉及到的协议及功能的二次封装
- 独立研究 K 线的指标计算，重写更为专业的 K 线控件，并做了部分优化
- 业务功能开发：币币交易，挂价，限价买卖，行情展示，k 线，分时图，深度图，5 档行情等等



### 乐米金融：金融小白第一站

金融教育类软件。最后一个版本共 5 大模块，分别为首页：主要是展示性内容，金融知识训练，以及子功能入口；资讯：展示型功能，包括实时资讯，运营文章等，跳转网页，评论等；游戏：操盘对战和 k 线对决；姐说，语音类答疑功能和语音学习资料；我的，包括个人信息，个人账户，系统通知，软件反馈在内的一系列功能

**项目职责：**

- 项目进度任务管理，参与需求分析评估
- 参与金融游戏的数据及协议的制定与完善
- 基础框架模块，公共模块的搭建，例如
	- 网络模块修改 Volley 源码，满足业务需求支持 Cookies，二次封装支持相同 url 的 GET 请求的同步和异步，参考 HTTP MULTIPART 报文格式实现文件上传，配置支持 HTTPS
	- 基于单例和观察者模式二次封装 websocket 框架，用于行情数据的获取，以及游戏的通信
   - 基于单例和观察者模式二次封装 MediaPlayer，屏蔽内部状态变化及处理细节，只留出接口进行控制以及状态通知
   	- 开发基于 AIDL 的单进程 WebView 模块，利用“JS 注入+ JsPrompt + 自定义注解”修复漏洞（通过反射获取运行时并执行 hack）统一网页调用和回调
- 业务功能开发：
	 - 金融游戏的实现：k 线对决：给定一段行情，在限定的时间内进行买卖操作，计算盈亏，获利多为获胜；操盘对决：实时行情，双人游戏，模拟期货操盘买多，卖空和平仓，限定时间内，获利多为获胜
 	- 股票行情模块，绘制股票特色的简易分时图和 K 线图
  	- 部分“姐说”功能开发，多个页面的语音播放及 UI 控制
  	- MA 线训练：缓慢的出现两条 MA 线，长短周期，在相交处暂停判断买与卖的单机小游戏



### 期货交易通 

国内国际期货大众产品杠杆交易软件。功能如下：首页，通知轮播，热门产品列表；行情，配有分时图和带 MA 和 交易量的 K 线图行情页面闪电下单，且带有闪电下单功能（即看着行情就立马下单）；交易，展示手续费和总金额，可配置止盈止损；订单查询，可查询历史订单以及历史盈亏；直播，老师直播，实时交流；个人中心，包括个人信息，个人账户，系统通知，软件反馈在内的一系列功能

**项目职责：** 

- 新人业务培训，Git Flow 培训以及 Code Review & Refactor
- 主导开发，负责开发 70% 以上的功能
    - 二次封装基于 Volley 的网络模块，基于链式构建者模式的调用方式，以及基于策略模式实现网络请求过程中的 UI 展示
    - 基类 Activity，Fragment，关联通用 UI 的调用，以及网络请求的取消
    - 通用组件和工具开发：例如，二次封装对话框支持单例，链式调用，自定义 View，和基类 Activity 同生命周期；基于 ViewPager 的无限广告栏；封装 Intent 实现统一界面启动器
    - 简易分时图，K 线图的绘制
    - 封装 socket 获取实时行情
    - 先后集成 letv 直播和网易云播 sdk 完成直播功能
	- 使用 gradle 进行多渠道，变种包配置及打包


## 致谢

感谢您花时间阅读我的简历，期待能有机会与您共事。