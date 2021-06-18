> Japanese Macdonald's App Project

---

# Memebr

- tech lead: qinjunbin
- pm: james
- and: lin，john
- ios：neo，qinjunbin
- product：nate



# 设计

JP：https://www.sketch.com/s/mWAmJ

EN：https://www.sketch.com/s/KrRVe



# McD JMA Glossary 

**NGLP(next-gen loyalty program)**: mainly for issuing coupons that are used at the counter. The user management module also sits in NGLP. Originally was a standalone app. The backend is developed by Plexure(PLX).

**MOP(mobile order & pay)**: users can order on the phone and pick up at the store. Originally was a standalone app. The backend is developed by Plexure(PLX).

**JMA**: MOP was merged into NGLP and is called JMA now. The project of the merge is called convergence. The Plant inherited the code bases from other companies and finished the merge. PLX still does the backend.

**CMS** ([www.mcdonalds.co.jp](http://www.mcdonalds.co.jp)): the CMS is developed by The Plant to manage content for the website mainly. Part of the JMA content (the generic menu) is sourced from the CMS. The generic menu is also consumed by WebMOP.

**WebMOP** ([www.mcdonalds.co.jp/order](http://www.mcdonalds.co.jp/order)): the Web version of MOP. Both frontend and backend are done by The Plant.

**Overflow/transition**: The idea is to have JMA talking to both PLX backend and The Plant backend. Dividing by purchasers. New purchasers talk to The Plant. Existing purchasers talk to PLX. Eventually, all talk to The Plant backend.

**VMob SDK**: the sdk developed by PLX used in the NGLP app

- doc: https://plexurereleasenotes.z19.web.core.windows.net/99-release_notes/android.html

**MOP SDK**. the sdk developed by PLX, currently only used in the JMA Android. iOS is using APIs directly.

**Generic Menu**: the menu that sourced from the CMS（Menu tab）

**Store Menu**: after picking a store, the store specific menu will be displayed.

**Dev release**: https://app.mcd.theplant-dev.com/ (theplant; theplant works)

**Firebase release**: UAT is released through Firebase.

**Analytics**: GA, Firebase, Bigquery.

**iA**: the design agency

**Ayudante**: the agency who provides analytics spec. (Jose is the contact)



# 发布

https://app.mcd.douwantech.com/

theplant / theplant works

- staging（NGLP:STG MOP:STG)
- dev（NGLP:PROD MOP:DEV)
- uat（NGLP:PROD MOP:UAT）
- spinning off（NGLP:PROD MOP:PROD）



# Firebase 控制台

https://console.firebase.google.com/u/1/project/mcd-dev-f605f/analytics/app/android:com.plexure.mopsdk.testapp/overview/~2F%3Ft%3D1606290875802&fpn%3D717673049648&swu%3D1&sgu%3D1&sus%3Dupgraded&cs%3Dapp.m.dashboard.overview&g%3D1



# 需求 PRD [Google doc]

## firebase

https://docs.google.com/presentation/d/1dm4E1NWa_VtILD6mGyAb_M-x5vSC_bQX3Ku9zsR3WZk/edit?ts=5fbf13d6#slide=id.p1



# 技术栈

- dagger
- butterknife -> kt extension
- 
- volley
- retrofit + okhttp
- 
- gson
- moshi
- 
- https://github.com/square/retrofit/tree/master/retrofit-converters/scalars 请求和响应支持纯文本

- 
- db: realm
- 
- rxjava
- 
- [Branch](https://branch.io/glossary/sdk)
- USABILLA 客户调查



# 待学技术栈

- 协程 coroutine
  - Android 协程：https://juejin.cn/post/6844903857072373773
  - retrofit+coroutine https://juejin.cn/post/6844903869990830093
  - koltin inline + refined 泛型原理
- GitHub Action：https://docs.github.com/cn/free-pro-team@latest/actions/reference/workflow-syntax-for-github-actions
- App Links
  - https://www.jianshu.com/p/b3ee359bb87b
  - https://developer.android.com/studio/write/app-link-indexing
- AppsFlyer
- protobuf 核心算法
- OkHttp3Downloader
- GCM
- rxjava 原理：https://www.bilibili.com/video/BV1wA411i7Ln?from=search&seid=6054173766483425278
- Rxjava2+Retrofit：https://juejin.cn/post/6844903839502434311
- 



# gradle 从 5. -》6 

findbug 插件已经移除，使用 [SpotBugs plugin](https://plugins.gradle.org/plugin/com.github.spotbugs) 



# 类说明

- VMobManager 对 Vmob sdk 的封装
- 



# Pending Work

- https://theplanttokyo.atlassian.net/browse/MDX-1832 设计和实现不够全面（设计作废 revert）

- ~~https://theplanttokyo.atlassian.net/browse/MDX-1580 zip code 无法更新~~

- https://theplanttokyo.atlassian.net/browse/MDX-1849 下单出现 “Open Choice error”

> 最早出现在 MDX-1563，是由于 NP# 接收的数据和 NP6 接收的数据结构不同导致的。
>
> https://theplanttokyo.atlassian.net/browse/MDX-1563
>
> NP# 接收的数据格式是产品数据（eg. 汉堡）包含非产品（成员变量 items[]）（eg. 芥末酱，无酱）
>
> NP6 接收的数据格式是产品数据和非产品数据同一个层级，属于同一个数组内
>
> 因此提交的数据格式采用的是 NP# 的数据格式，但是在 items[] 的每个非产品的数据结构里加了 ShouldBeBaseItem:true 成员变量，backend 在收到订单数据的时候，会把 items 的数据移除外层（与产品同一层）达到兼容的目的 NP6 的目的
>
> **注意：产品和非产品的数据结构是一样的**
>
> 
>
> 在下单的时候获取产品数据后，会有一个 additionalChoices，内部有个 choiceProductCode，通过请求它获取非产品（辅料）的数据（这个事情待验证）

- https://theplanttokyo.atlassian.net/browse/MDX-1543 linkto https://theplanttokyo.atlassian.net/browse/MDX-1860 可能要做

  > 打开 kodo webview 的时候 url 传入参数，参数已经在 number 里面定好
  >
  > 你需要先看一下老的 kodowebview 是怎么写的

- https://theplanttokyo.atlassian.net/browse/MMF-65 

- https://theplanttokyo.atlassian.net/browse/MMF-78

  - 同一个页面的 bugs

- https://theplanttokyo.atlassian.net/browse/MDX-1864 still doing

- https://theplanttokyo.atlassian.net/browse/MDX-1874

  - 和 neo 一起完成，不是这个 sprint 的任务



sptint 18

- ~~https://theplanttokyo.atlassian.net/browse/MDX-1845~~
- ~~https://theplanttokyo.atlassian.net/browse/MDX-1846~~
  - 字体问题
  - stagingDebug: 90,611,173 bytes (90.6 MB on disk)
- ~~https://theplanttokyo.atlassian.net/browse/MDX-1881~~ 20->21

- ~~https://theplanttokyo.atlassian.net/browse/MDX-1856 prompt when typing/pasting invalid symbols in password field~~
- ~~https://theplanttokyo.atlassian.net/browse/MDX-1857 Stuck on Sign page~~
- https://theplanttokyo.atlassian.net/browse/CNV-318 Download page does not change only for Android devices / Android端末のみダウンロードページが遷移しない
- ~~[https://theplanttokyo.atlassian.net/browse/MDX-1879](https://theplanttokyo.atlassian.net/browse/MDX-1879) 复制 Caffee Stamp 代码~~

**adb shell am start -a android.intent.action.VIEW -d “[mcdonaldsjp://coffee-stamp-card?cid=1&wi=1](mcdonaldsjp://coffee-stamp-card?cid=1&wi=1)”**



sprint 19

- ~~https://theplanttokyo.atlassian.net/browse/MDX-1889 remote config control qrcode scan entrance show/hide~~
  ~~https://theplanttokyo.atlassian.net/browse/MDX-1899 Send My Order to Kitchen 替换 Proceed to Pick Up~~
  https://theplanttokyo.atlassian.net/browse/MDX-1872 (这个用charlse看下, 什么情况下会多发, 这个现在都是vmob sdk发的)

  /consumers/me/tagvalues 见代码追踪流程

  > 先不改

- https://theplanttokyo.atlassian.net/browse/MDX-1929?focusedCommentId=83444&page=com.atlassian.jira.plugin.system.issuetabpanels%3Acomment-tabpanel#comment-83444 问题待分析

- Issues:
  \1. https://theplanttokyo.atlassian.net/browse/MDX-1932 done
  \2. https://theplanttokyo.atlassian.net/browse/MDX-1934 done
  \3. https://theplanttokyo.atlassian.net/browse/MDX-1935 done
  \4. https://theplanttokyo.atlassian.net/browse/MDX-1936 done

  > 需要的网站和相关的issue.CMS:
  > https://admin-tvk048.vmobapps.com/Account/Logon
  >
  > nate@theplant.jp
  >
  > cqi)qtyP9Fc)xn
  >
  > Coupons:
  > https://mcdonalds-japan-resetcoupons2.azurewebsites.net/
  > nate@theplant.jp
  > cqi)qtyP9Fc)xn
  
- 去掉蒙层
- https://theplanttokyo.atlassian.net/browse/MDX-1929 webview geo @lin done



sprint 20

![image (1)](/Users/john/Downloads/image (1).png)

![image (2) copy](/Users/john/Downloads/sprint21/image (2) copy.png)

- https://theplanttokyo.atlassian.net/browse/MDX-1886 done

  这两个做了, 先不merge.

- 这两个 issue 需要帮助确认一下：

- https://theplanttokyo.atlassian.net/browse/MDX-1849 按照 Aki 的回复，下单的测试环境设备是 NP6，那我们应该如何进行重现与测试工作？

  > **池袋北口店**，T444，这个他们知道是测试的单，就不会准备食物
  >
  > 测试视频已上传


paypay callback url:

-  https://www.paypay.ne.jp/app/cashier?code=https%3A%2F%2Fqr.paypay.ne.jp%2F28180104HcbYl78wClzMOhST



errorCode 400

This order failed to check-in, please try again or contact customer support

mop.consumerorders.service.orderCheckInFailed

ORDER_DEAUTHORIZE_FAILED

- ~~那后这个 task 如果看 comment 里的说明还是不明白的话，可以来问我。https://theplanttokyo.atlassian.net/browse/MDX-1797?atlOrigin=eyJpIjoiNWQxZjI5ZWEyZDYyNDk2ZjlhMmRhOWUzNTAxODk0MDciLCJwIjoiaiJ9~~

- https://theplanttokyo.atlassian.net/browse/MDX-1966 

  > - **run productDebug (use the newest released version branch)**
  > - **enable debug:**
  > - adb shell setprop log.tag.GoogleTagManager VERBOSE
  > - **open the preview URL in emulator, it will restart JMA:**
  >
  > https://tagmanager.google.com/mcpr/jp.co.mcdonalds.android?id=GTM-59QXWDT&gtm_auth=6Oe_6mrwFQOMsjiSGT9yhA&gtm_preview=6
  >
  > - **go to Coupon tab (not mop)**
  > - **test these coupon redemption and collect logs**
  > - **regular coupon**
  > - **happy meal (on happy meal tab)**
  > - **happy meal (on all coupons tab)**
  >
  > **Note:**
  >
  > 1. **redumption means tapping the “order at the store” button**
  > 2. **when collect logs, clear current log before scrolling to the target coupon and order button, so the collected log is minimal**
  >
  > Create files: happy meal on happy meal tab.txt, happy mean on all tab.txt, regular.txt.



sprint 21

![image (3)](/Users/john/Downloads/sprint21/image (3).png)

![image (2)](/Users/john/Downloads/sprint21/image (2).png)

- https://theplanttokyo.atlassian.net/browse/MDX-1887

- https://www.figma.com/file/indbO2NX5HAns5qql406wo/JMA---Splash-Screen-Ideas?node-id=209%3A238

  

- https://theplanttokyo.atlassian.net/browse/MDX-1740 我二次确认一下要加的逻辑：api 会新增 is_rainforest_alliance_certified_center 字段，如果存在该字段且为 true，RA 文字还是使用当前固定的文字，图片使用固定的网络图片 https://www.mcdonalds.co.jp/assets/images/rainforest_coffee.png

  > 正确

  





# 测试账号

可下单信用卡 411111111111111

crossReference：1@a.cn/123123123



paypay：

PayPay account:
070 3605 9258
Pwd: PayPayworks303!
这个手机现在在James那里，登录需要验证码你可以问下他



# 记录 bug 的 google doc

https://docs.google.com/spreadsheets/d/1k9XwgkHaMFRDb0hFVl-Y3AfHxvW7mud1Ud544KlyjDI/edit#gid=1556965999

- duplicate 重复
- bug fixed 开发修好了 等待QA review 
- still not fixed QA检查过发现bug没修复 
- Done QA检查发现bug已经修复
- hold 目前这个bug不修



# github

https://github.com/theplant/mcd-android/



# Plx sdk（Vmob sdk）url

https://plexurereleasenotes.z19.web.core.windows.net/99-release_notes/android.html#july-20-2018



# 查看栈顶 Activity

adb shell dumpsys activity activities | grep mResumedActivit



# 老设计搞和字体的 Map

- Hiragino Sans: W7 = Noto Sans CJK JP: Bold
- Hiragino Sans: W5 = Noto Sans CJK JP: Medium
- Hiragino Sans: W3 = Noto Sans CJK JP: DemiLight
- Exception: Medium is applied for top app bar.



# 打包流程

1. 修改 build.gradle 为最新版本

```
def VERSION_NUMBER = "285"
```

2. 提交并打标签，git action 自动打包

```shell
git commit -a -m 'build: 5.1.110(285)'
git push
git tag releases-285
git push origin releases-285
```

apk 路径：oss://appstore-data/builds

3. 修改 oss 上版本日志文件（oss: //appstore-data/mcd/android-note.js）

copy 一份之前的版本日志 jsonobject，然后修改 title，time，items

4. 打 uat 包

打好的 apk 路径在 oss://appstore-data/builds/202/，如果没有 uat 的 apk，那么需要你自己再打一次并手工上传 firebase

5. 上传 uat apk 到 firebase



## 打包日志

```json
{
        title: "Build: 281",
        time: "2021.06.01 18:00",
        items: [
          "mdx-2166, mdx-1616, mdx-2189"
            ]
    },
```



# 可删除的文件

```
LoginSwitchMangoView，LoginSwitchView
view_login_switch_mango.xml
view_login_switch.xml

/Users/john/Desktop/mcd-android/app/src/main/res/layout/view_login_edit.xml

NewsActivity
NewsFragment
NewsAdapter
content_news_card.xml
```



# OVF

\1. IOS的build好了(240)
\2. 现在开始IOS和Android只要测试JMA的那些build(TF下载). 这个下面的[https://overflow.mcd.douwantech.com](https://overflow.mcd.douwantech.com/) build就不测试了.
\3. 然后现在只有Staging的build可以支持切换到WMOP(点击logo 8下, 然后setting里面选).
\4. 这段时间你们测试下，下载各个版本是不是默认打开都是用MOP系统(搜索139, 如果出来的store带UAT, STG, 那就是MOP的系统). 如果有意外，麻烦群里说下.
\5. 最好下build的时候，有时候是删除再下载，有时候是升级下载, 这样新安装和升级的情况也能模拟出来.
\6. 麻烦你们尽可能用Prod(MOP)去测试下功能, 需要做一次回归测试, 看下是不是没问题. 测试下单再用STG/UAT去测试.
\7. UAT的WMOP要到3/11才可能会有
\8. (重要) 麻烦你们优先测试下WMOP(STG build)下订单的各种支付和中断订单, BE改了订单的流程，改了很多地方，所以需要在细查一次. Mark答应客户周一给他们Beta测试的, 所以可能就只有明天可以测试了. ![:man-bowing:](https://a.slack-edge.com/production-standard-emoji-assets/13.0/apple-medium/1f647-200d-2642-fe0f@2x.png)



# AppsFlyers 账号

[nate+mcd@theplant.jp](mailto:nate+mcd@theplant.jp)

t@xjHM9b8QdWkLu%



# https://theplant.udemy.com 账号

jason@theplant.jp

Y-yBy7P-ctmUT9



# https://cms.mcd.theplant-dev.com 账号

https://cms.mcd.theplant-dev.com/admin/products

mcd/mcd works