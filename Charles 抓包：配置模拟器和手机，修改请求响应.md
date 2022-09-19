# Charles 抓包：配置 Android 模拟器和手机

## Charles 配置 

1. Charles 菜单：Proxy，macOs Proxy
2. Proxy Setting...：Http Port：8888
3. Proxy，SSL Proxiyng Settings，Enable SSL Proxying，添加 `*.*`



## Android 模拟器 Charles 配置（WIFI 代理）

1. 修改 Wi-Fi 配置：AndroidWifi，打开菜单：Modify Network

2. 配置手动代理：

   ```shell
   Proxy: Manual
   Proxy hostname: 10.0.2.2 (this isn’t local IP, just put 10.0.2.2)
   Proxy port: 8888
   And press “Save”
   ```

3. 打开浏览器安装证书：https://chls.pro/ssl

4. Android N 开始，需要配置 App 相信 Charles SSL Cert：项目中添加 ‘网络安全配置文件：network_security_config.xml’

   ```xml
   res/xml/network_security_config.xml
    
    <network-security-config> 
     <debug-overrides> 
       <trust-anchors> 
         <!-- Trust user added CAs while debuggable only -->
         <certificates src="user" /> 
       </trust-anchors> 
     </debug-overrides> 
   </network-security-config>
   
   <applicationandroid:networkSecurityConfig="@xml/network_security_config">
   ```



## Android 手机 Charles 配置（WIFI 代理）

也需要配置手动代理，和模拟器非常类似，区别就是 host 需要填入电脑的 ip

端口也是 8888

安装证书：

- 方法 1：Help，SSL Proxying，Install Charles Root Certificate on a Mobile Device or Remote Browser
- **方法 2：手机设置好代理后访问：charlesproxy.com/getssl，会弹出下载证书，安装**（推荐）
- 方法 3：Save Charles Root Certificate...，然后使用 adb push 命令推送到手机 sdcard，然后双击安装



### 手机安装 pem 证书

1. 手机设置 -》安全和xx（锁屏，隐私）
2. 更多安全设置（高级）-》xx（加密、安全）和凭依 -》
3. 从存储设备安全 -》找到 .pem 文件



# Charles 抓包之对响应 Response 的修改

3 种方法：

- Map（Map Local、Map Remote）：长期地将某一请求重定向到另一个指定的网络地址或者本地 JSON 文件
- Rewrite：功能适合对网络请求和响应进行一些正则替换
- Breakpoints：对网络请求进行一些临时性的修改，但是对超时时间过短的请求不合适

参考：https://zhuanlan.zhihu.com/p/149948890



