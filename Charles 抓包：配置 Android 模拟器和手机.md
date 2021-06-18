http://mms.msg.eng.t-mobile.com/mms/wapenc



https://medium.com/@daptronic/the-android-emulator-and-charles-proxy-a-love-story-595c23484e02



charlesproxy.com/getssl





# Charles 抓包：配置 Android 模拟器和手机

## Charles 配置

1. Charles 菜单：Proxy，macOs Proxy
2. Proxy Setting...：Http Port：8888
3. Proxy，SSL Proxiyng Settings，Enable SSL Proxying，添加 `*.*`



## Android 模拟器 Charles 配置

2. 打开模拟器菜单（三个点）：Setting，Proxy，Manual proxy configuration

   1. Hostname：本机 ip（打开 Network 查看）
   2. Port number：8888

3. 完成 2 后，可以在 Charles 上看到拦截到的 https 请求 ip。继续安装 Charles SSL Cert（证书）

4. 在模拟器的浏览器里面打开：charlesproxy.com/getssl，会弹出下载证书，安装

5. Android N 开始，需要配置 App 相信 Charles SSL Cert：通过添加 ‘网络安全配置文件’

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

6. 配置 APN（Access Point Name）

   1. Setting，Network and Internet，Mobile Network，Advanced，Access Point Names

   2. 直接搜索 Access Point Names 也可以找到，添加一个和当前使用一样的 APN，唯一区别是添加了 Proxy & Port

      1. Name: xxxx
      2. APN：epc.tmobile.com
      3. MMSC：http://mms.msg.eng.t-mobile.com/mms/wapenc
      4. Save and selected

      

## Android 手机 Charles 配置

其他的配置类似，关键在于安装证书：

- 方法 1：Help，SSL Proxying，Install Charles Root Certificate on a Mobile Device or Remote Browser
- 方法 2：手机设置好代理后访问：charlesproxy.com/getssl，会弹出下载证书，安装
- 方法 3：Save Charles Root Certificate...，然后使用 adb push 命令推送到手机 sdcard，然后双击安装



### 手机安装 pem 证书

1. 手机设置 -》安全和xx（锁屏，隐私）

2. 更多安全设置（高级）-》xx（加密、安全）和凭依 -》
3. 从存储设备安全 -》找到 .pem 文件

