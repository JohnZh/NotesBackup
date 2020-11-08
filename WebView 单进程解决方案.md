# App WebView 单进程解决方案

# webview js 与 native 通信

## native 调用 webview js

| 方式                      | 优点     | 缺点                       | 使用场景                           |
| ------------------------- | -------- | -------------------------- | ---------------------------------- |
| 使用 loadUrl()            | 方便简洁 | 效率低，获取返回值麻烦     | 不需要获取返回值对性能要求低的情况 |
| 使用 evaluateJavascripe() | 效率高   | 向下兼容性差(仅 4.4及以上) | Android 4.4及以上                  |

> js 代码一定要在 onPageFinished 完成后才能调用

## webview js 调用 native

| 方式                                                         | 优点     | 缺点                                                      | 使用场景                                 |
| ------------------------------------------------------------ | -------- | --------------------------------------------------------- | ---------------------------------------- |
| 使用 addJavascriptInterface() 进行添加对象映射               | 方便简洁 | Android4.2 以下有漏洞问题                                 | Android 4.2 以上相对简单的互调场景       |
| 使用 WebViewClient # shouldOverUrlLoading 回调拦截 url       | 无漏洞   | 使用复杂，需要进行协议约束从 native 传值到 web 层比较繁琐 | 不需要返回值的场景（ios 主要使用该方式） |
| 使用 WebChromeClient # onJsAlert，onJsConfirm，onJsPrompt 拦截 js 对话框消息 | 无漏洞   | 使用复杂，需要协议约束                                    | 能满足大多数情况下的回调场景             |

### WebView Android 4.2 以下的漏洞
addJavascriptInterface() 会添加一个用于 js 调用的对象，由于 Android 4.2 以下没有 @JavascriptInterface 方法注解机制（Android 4.2 以上 js 只能调用  @JavascriptInterface 注解过的方法），那么添加的这个 js 映射对象就可以基于反射调用任何方法

漏洞测试代码：

```
<script type="text/javascript">
	function check() {
		for (var obj in window) {
			try {
				if ("getClass" in window[obj]) {
					try {
						window[obj].getClass();
						document.write('<span style="color:red">' + obj + '</span><br />')
					}
				}
			}
		}
	}
</script>
```

这段代码如果在 4.2 以下，添加了 addJavascriptInterface() 之后，那么打印出来应该是：通过 addJavascriptInterface() 添加的接口对象，以及 `searchBoxJavaBridge_`
如果打开了辅助功能，那么还有：`accessibilityTraversal`，`accessibility`

那么当 hacker 加载了一 html，并且获取到了其中一个对象 obj，然后使用下面方式就可以进行 hack 了

```
Class runtimeClass = obj.getClass().forName("java.lang.Runtime");
Object runtime = runtimeClass.getMethod("getRuntime", null).invoke(null, null); // 静态方法 getRuntime() 返回 Runtime 对象

// 最后 hacker 就可以通过 Runtime 的 exec() 执行任何命令，包括获取本地数据或者写入（如果有读写外存权限）
(Runtime (runtime)).exec(String command)

// Android 4.2 以上 js 只能调用  @JavascriptInterface 注解过的方法，因此其他方法都无法再调用
```

### 修复（绕过）这个安全问题
首先，4.2 版本以下肯定不能使用 addJavascriptInterface() 来添加接口对象类给 js 调用了，其次，还需要通过 removeJavascriptInterface() 删除 `searchBoxJavaBridge_ `，`accessibilityTraversal `，`accessibility `

其次，为了统一 html 端 js 调用，那么可以去给页面注入 js 代码，让前端调用：

```
// 4.2 以上调用的代码，webview 是接口对象，post 是接口方法
// android native
addJavascriptInterface(this, "webview");

@JavascriptInterface
public void post(String cmd, String params) {}

// 对应 webview js 的调用
window.webview.post(cmd, params); 

// 为了满足上面 webview js 的调用模式，我们要声明一个 webview 的 js 对象，对象里面有个一样的 post 方法
// post 方法的实现需要可以被 webview 拦截处理，可以是 jsPrompt，提供的内容要可以用于反射来调用到具体类的方法
window.webview = {
	post:function(cmd, params) {
		return prompt('app:' + JSON.stringify({object: 'webview', method: 'post', args:[cmd, params]}))
	}
}

// 注入了上面的 js 代码，前端就可以使用 window.webview.post(cmd, params) 了，而该方法实际是调用了 
// prompt('app:' + JSON.stringify({object: 'webview', method: 'post', args:[cmd, params]}))
// 这段 prompt 会被 Android native 的 onJsPrompt() 获取到
// 在根据 'app:' 这个命名空间得知这个是一个方法调用，解析后面内容之后再去调用相关类的相关方法
// 为了让注入的 js 代码自动执行，上面声明的对象还要放入自执行函数里，并且进行是否已经定义的判断

javascript:(function autofun() {
	if (typeof (window.webview) == 'undefined') {
		window.webview = {...}	
	} else {
		console.log("window.webview is exist.");
	}
})()
```
这是一个接口对象一个方法的写法，实现上还要满足多个的接口对象多个方法。

方案：override addJavascriptInterface() 和 removeJavascriptInterface() 然后使用一个 map 保存其他的接口对象和名称，最后统一生成 js 注入代码

注意点：

- 注入的 js 代码要在每个新页面都注入一次
- 为了生成注入的 js 代码的方便，可以自定义 @JsInterface 注解，反射获取方法信息
- 为了拦截 prompt 解析的方便，js 接口定义的方法参数全部使用 String

源码参考：

- [SecureWebView.java](https://github.com/JohnZh/SnippetProject/raw/master/webview/src/main/java/com/john/webview/SecureWebView.java)
- [SecureWebChromeClient.java](https://github.com/JohnZh/SnippetProject/raw/master/webview/src/main/java/com/john/webview/client/SecureWebChromeClient.java)

## 其他安全问题

### 密码明文

webview 的密码保存是明文的，且这个功能在 >= api 18 不做支持了，因此使用老的 api 将这个功能关闭：WebSettings.setSavePassword(false)

### file:// 协议漏洞

不需要被其他 app 导出的 webview 不要添加 `exported="true"`，需要导出的应该关闭 file:// 协议的访问。相关 api

```
WebSettings#setAllowFileAccess 
WebSettings#setAllowFileAccessFromFileURLs [OS> 4.1 api16 默认 false]
WebSettings#setAllowUniversalAccessFromFileURLs [OS> 4.1 api16 默认 false]
```
hacker 唤起 `exported="true"` 的 webview 通过 <data> 传入 url，直接或间接通过 url 打开 file:// 协议制定的 hack 文件就可以获取 app 的私有数据

**另一种思路**，由于 file:// 指向的 hack 文件一般都是执行 js 来进行 hack，那么当加载 url 为 file:// 的时候，可以禁用 js：`WebSettings#setJavaScriptEnabled(false)`


## 通信方案
为了满足调用的泛用性，定义单独一个方法，使用 方法(命令, 参数) 的模式


## 架构搭建
为了避免 webview 加载 OOM 和无法预知的错误导致 webview 错误而导致 App 崩溃，推荐使用单进程去开启一个 webview 客户端。基于 C/S 模型的 IBinder 进程间通信。因此，架构如下：

1. webview client -- 发送命令 --> app service（业务例如路由原生页面，打开其他 app，进行下载等等）
2. webview client <-- 返回结果 -- app service

再细化：

1. webview 进程：webview js -> native code -> remote service aidl proxy -> remote service aidl stub -> main app 进程 service
2. main app 进程 service -> aidl interface from proxy -> remote service aidl proxy -> native code -> webview js

## 完整项目
[SnippetProject](https://github.com/JohnZh/SnippetProject) 项目的 webview 模块


## 漏洞

- [CVE-2012-6636：addJavascriptInterface 接口会引起远程代码执行漏洞](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2012-6636)
- [CVE-2013-4710：某些特定机型会存在 addJavascriptInterface API 引起的远程代码执行漏洞](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2013-4710)
- [CVE-2014-1939：WebView 中内置导出的 “searchBoxJavaBridge_” Java Object 可能被利用，实现远程任意代码](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-1939)
- [CVE-2014-7224：accessibility 可被利用实现远程任意代码执行](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-7224)
- [CVE-2014-1939：accessibilityTraversal 可被利用实现远程任意代码执行](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-1939)

# Webview 的调试
[https://developers.google.com/web/tools/chrome-devtools/remote-debugging/webviews?hl=zh-cn](https://developers.google.com/web/tools/chrome-devtools/remote-debugging/webviews?hl=zh-cn)


# 更新
- webView.evaluateJavascript() 方法在高版本系统上无法调用到 js，程序没有任何反馈，调试 Webview 发现调用到 js 的时候，发出了 Uncatch error，是由于调用 java 代码产生的。于是对 webView.evaluateJavascript() 进行异常捕获，发现了 “Webview 方法在‘JavaBridge’线程上被调用”的异常，那么需要把这个方法调用放到和 Webview 一个线程

```
binding.webView.post(() -> {
    if (Build.VERSION.SDK_INT <= Build.VERSION_CODES.JELLY_BEAN_MR2) {
        binding.webView.loadUrl(jsCallbackMethod);
    } else {
        binding.webView.evaluateJavascript(jsCallbackMethod, null);
    }
});
```
- 子进程开启 Webview 之后，在主进程也开启 Webview，导致异常退出，或者“服务连接”泄露问题。原因是 Android P（28）开始，Webview 的数据路径从“共享”变成一个进程一个路径，如果一个 app 下有多个进程要用 Webview，那么意味着需要多个路径，因为默认路径只能被一个进程使用。

```
// 解决方案：就使用一个 Webview。这是最简单的
// 其次，设置数据路径的后缀：可以在 Application 里面设置

if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.P) {
        WebView.setDataDirectorySuffix(getProcessName());
}
// 这个方法要求在所有 Webview 初始化之前被调用，不然会出错，详情看文档
```