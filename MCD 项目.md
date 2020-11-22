Japanese Macdonald's App Project

---



# Memebr

- tech lead: qinjunbin
- pm: james
- and: lin，john
- ios：neo，qinjunbin



# McD JMA Glossary 

**NGLP(next-gen loyalty program)**: mainly for issuing coupons that are used at the counter. The user management module also sits in NGLP. Originally was a standalone app. The backend is developed by Plexure(PLX).

**MOP(mobile order & pay)**: users can order on the phone and pick up at the store. Originally was a standalone app. The backend is developed by Plexure(PLX).

**JMA**: MOP was merged into NGLP and is called JMA now. The project of the merge is called convergence. The Plant inherited the code bases from other companies and finished the merge. PLX still does the backend.

**CMS** ([www.mcdonalds.co.jp](http://www.mcdonalds.co.jp)): the CMS is developed by The Plant to manage content for the website mainly. Part of the JMA content (the generic menu) is sourced from the CMS. The generic menu is also consumed by WebMOP.

**WebMOP** ([www.mcdonalds.co.jp/order](http://www.mcdonalds.co.jp/order)): the Web version of MOP. Both frontend and backend are done by The Plant.

**Overflow/transition**: The idea is to have JMA talking to both PLX backend and The Plant backend. Dividing by purchasers. New purchasers talk to The Plant. Existing purchasers talk to PLX. Eventually, all talk to The Plant backend.

**VMob SDK**: the sdk developed by PLX used in the NGLP app

**MOP SDK**. the sdk developed by PLX, currently only used in the JMA Android. iOS is using APIs directly.

**Generic Menu**: the menu that sourced from the CMS（Menu tab）

**Store Menu**: after picking a store, the store specific menu will be displayed.

**Dev release**: https://app.mcd.theplant-dev.com/ (theplant; theplant works)

**Firebase release**: UAT is released through Firebase.

**Analytics**: GA, Firebase, Bigquery.

**iA**: the design agency

**Ayudante**: the agency who provides analytics spec. (Jose is the contact)





232 22+175 =197

# 发布

https://app.mcd.douwantech.com/

theplant/theplant works

- staging（NGLP:STG MOP:STG)
- dev（NGLP:PROD MOP:DEV)
- uat（NGLP:PROD MOP:UAT）
- spinning off（NGLP:PROD MOP:PROD）





# Technology Stack

- dagger
- butterknife -> kt extension
- 
- volley
- retrofit + okhttp
- 
- db: realm
- 
- rxjava
- 

- [Branch](https://branch.io/glossary/sdk)
- USABILLA 客户调查



# gradle 从 5. -》6 

findbug 插件已经移除，使用 [SpotBugs plugin](https://plugins.gradle.org/plugin/com.github.spotbugs) 



# 类说明

- VMobManager 对 Vmob sdk 的封装
- 



# 代码追踪路径

## 闪屏到启动主页面

```
splash 
	main: McdonaldsJapanHomeActivity 
			onStart()
				InitJob.init(true)
					initMain
						apiResultCallback.onSuccess(new InitEvent(InitEvent.EventId.leaveMaintenance, new Bundle()));
							registrationSkipUser()
								syncUserConfig()
									finish()
										InitJob.onSuccess(new InitEvent(InitEvent.EventId.goStart, new Bundle()))
		onInitEvent (EventId.goStart) 
			super(BaseActivity).onInitEvent
				case goStart:
					MainActivity
```



## 主页登录 -》登录页面-》登录完成

LoginAct -> onStart, setUpActivity() -> onAuthEvent( AuthEvent[none]) 

-> case goReLoginMail: AuthJob.goReLoginMail(event, isCleanEmail)

-> AuthJob.onSuccess(event) - > LoginAct#onAuthEvent(AuthEvent[onSuccess])

-> replaceFragment(new ReLoginFragment(), bundle) -> ConcreteTestHandler#processMessage -> pushFragment

-> ReLoginFragment#@OnClick(R.id.reLoginButton) -> LoginAct#onAuthEvent(AuthEvent.EventId.doReLoginMail，none)

-> AuthJob.doReLoginMail(event) -> ContentsManager.login(userConfig, new LoginApiResultCallback(event)) 

-> logout() success -> getVMobManagerInstance().login -> syncUserConfig( onSuccess -> updateConsumer4NGLP2WithFelicaInit

-> Preference.setUserConfig(workUserConfig)

.. updateConsumer(...) -> getVMobManagerInstance().putConsumer .. getVMobManagerInstance().putConsumerTags 

-> Preference.setUserConfig(userConfig) .. putSubscriberKey(userConfig, apiResultCallback) -> post LoggedInEvent(isLoggerdIn = true) 但是 loggedInEvent 是没有方法处理的！！！！

关闭 loginActivity 页面的地方在 LoginApiResultCallback#onSuccess -> AuthJob.onSuccess -> post AuthEvent(onSuccess)

-> ContentsManager.logLogin("mail", true) 奔溃分析 .. LoadingMessageDialogFragment onDissmiss()  

-> post AuthEvent(goRegisterUserInfo，none) -> LoginAct#onAuthEvent id:goRegisterUserInfo

 -> post AuthEvent(goRegisterUserInfo, doInstantWinFbfAddPoints, none) -> if (fbfLinkId.isEmpty() || fbfLcpf.isEmpty()) true 

-> post AuthEvent(goRegisterUserInfo, callback) -> onAuthEvent(): goRegisterUserInfo -> callback 

-> AuthJob.goRegisterUserInfo(event) -> ContentsManager.syncUserConfig(isWriteBack false) -> onSuccess

-> AuthJob.onSuccess(event) -> post onAuthEvent(goRegisterUserInfo, onSuccess)

-> LoginAct: onAuthEvent(): goRegisterUserInfo -> loadingOnSuccess -> if (bundle.getBoolean(BundleKeys.isRegistered, false)) = true

-> post BackStackEvent(close) -> onBackStackEvent(): finish



## 底部 tab 点击到 coupon 加载

MainActivity#navigateToCoupon(null) -> CouponMainFragment#checkDeepLinkData(bundle)

-> args == null -> CouponJob.getCoupon(null, null, null) -> 





# 页面记录

```
AppTutorialActivity 闪屏

```



# Pending Job

https://theplanttokyo.atlassian.net/browse/MDX-1790

https://theplanttokyo.atlassian.net/browse/MDX-1787
https://theplanttokyo.atlassian.net/browse/MDX-1495



# Untest job



# Done job

https://theplanttokyo.atlassian.net/browse/MDX-1783

https://theplanttokyo.atlassian.net/browse/MDX-1742

https://theplanttokyo.atlassian.net/browse/MDX-1822 





# Test account

可下单信用卡 411111111111111

crossReference：1@a.cn/123123123



# github

https://github.com/theplant/mcd-android/tree/f/coupon_qrcode



# 查看栈顶Activity

adb -d shell dumpsys activity activities | grep mResumedActivity