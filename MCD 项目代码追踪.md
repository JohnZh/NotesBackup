# 	闪屏到启动主页面

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



# 主页 4 个 tab -》Fragment

```
HomeRootFragment
CouponMainFragment
MenuRootFragment
NewStoreChooseFragment

// 最开始显示 HOME
supportDelegate.loadMultipleRootFragment(R.id.main_fl_container, HOME,
                    mFragments[HOME],
                    mFragments[COUPON],
                    mFragments[MENU],
                    mFragments[STORE])
                    
HomeRootFragment 实际上是加载 HomeNewFragment
```





# 主页登录 -》登录页面-》登录完成

```
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

关闭 loginActivity 页面的地方在 LoginApiResultCallback#onSuccess -> AuthJob.onSuccess -> post AuthEvent(AuthEvent.EventId.doReLoginMail，onSuccess)

-> ContentsManager.logLogin("mail", true) 奔溃分析 .. LoadingMessageDialogFragment onDissmiss()  

-> post AuthEvent(goRegisterUserInfo，none) -> LoginAct#onAuthEvent id:goRegisterUserInfo

 -> post AuthEvent(goRegisterUserInfo, doInstantWinFbfAddPoints, none) -> if (fbfLinkId.isEmpty() || fbfLcpf.isEmpty()) true 

-> post AuthEvent(goRegisterUserInfo, callback) -> onAuthEvent(): goRegisterUserInfo -> callback 

-> AuthJob.goRegisterUserInfo(event) -> ContentsManager.syncUserConfig(isWriteBack false) -> onSuccess

-> AuthJob.onSuccess(event) -> post onAuthEvent(goRegisterUserInfo, onSuccess)

-> LoginAct: onAuthEvent(): goRegisterUserInfo -> loadingOnSuccess -> if (bundle.getBoolean(BundleKeys.isRegistered, false)) = true

-> post BackStackEvent(close) -> onBackStackEvent(): finish

```



# 主页 》用户信息》编辑用户信息

UserCenterActivity -》ucTvMemberEdit.onClick() -》LoginActivity.onStart():setUpActivity() -》根据传入的 eventId：goAccountEdit直接到 onAuthEvent() -》switch(): goAccountEdit -》case none: AuthJob.goAccountEdit(event) 同步一次用户信息之后 -》case default：替换 Fragment（AccountEditFragment）



issue：控件和业务逻辑耦合太严重



# 底部 tab 点击到 coupon 加载

MainActivity#navigateToCoupon(null) -> CouponMainFragment#checkDeepLinkData(bundle)

-> args == null -> CouponJob.getCoupon(null, null, null) -> 



# 首页最后的新闻图片点击打开 webViewShareActivity(WebViewActivity)

HomeNewFragment line333 -》 line354 ScreenTransitionUtil.onClickThroughUrl -》

```java
mcdonaldsjp://webapp?webAppUrl=https://www.mcdonalds.co.jp/campaign/choimc/?m=1
Scheme：mcdonaldsjp
host：webapp
```

构造出 OuterLinkScreenInfoWebShare 继承自 OuterLinkScreenInfo

```
                put(SCREEN_NAME_WEBAPP, new OuterLinkScreenInfoWebShare(WebViewShareActivity.class, false, WebViewShareActivity.Companion.newStartActivityBundle(false, true)));
```

执行 `screenInfo.postEvent(uri, isFinish, update); `ScreenTransitionUtil line 185

构造 bundle 

```
Bundle[{acceptWebStorage=true, acceptCookie=false}]
到
Bundle[{webAppUrl=https://www.mcdonalds.co.jp/campaign/choimc/?m=1, acceptWebStorage=true, acceptCookie=false}]
到
加上 FirebaseEvent(FirebaseEvent.Method.News.share, news, helper.adapterPosition - 1).build() 的内容：

Bundle[{webAppUrl=https://www.mcdonalds.co.jp/campaign/choimc/?m=1, acceptWebStorage=true, __FirebaseEvent.name=nglp2, __FirebaseEvent.params=Bundle[{item_name=mcdonaldsjp://webapp　test, item_brand=plexure, method=share, index=3, event_id=2, item_category=NBS, content_type=news, item_id=1876}], acceptCookie=false}]

最后 editBundle(bundle):
Bundle[{webAppUrl=https://www.mcdonalds.co.jp/campaign/choimc/?m=1, acceptWebStorage=true, __FirebaseEvent.name=nglp2, __FirebaseEvent.params=Bundle[{item_name=mcdonaldsjp://webapp　test, item_brand=plexure, method=share, index=3, event_id=2, item_category=NBS, content_type=news, item_id=1876}], acceptCookie=false}]
```

最后 postEvent(final Bundle bundle, final Boolean isFinish)，isFinish = falase

```java
EventBus.getDefault().post(getEvent(bundle, isFinish));

ScreenInfo.getEvent(final Bundle bundle, final Boolean isFinish) {
  ScreenEvent event = new ScreenEvent();
  event.setActivityClass(getActivityClass());
  event.setBundleData(bundle);
  event.setFinish(isFinish != null ? isFinish : isFinish());

  return event;
}
```

最后到 BaseActivity 的 onScreenEvent(ScreenEvent event)：

```java
ScreenEvent：

activityClass = {java.lang.Class@14208} "class jp.co.mcdonalds.android.view.webview.WebViewShareActivity"
bundleData = {android.os.Bundle@22169} "Bundle[{webAppUrl=https://www.mcdonalds.co.jp/campaign/choimc/?m=1, acceptWebStorage=true, __FirebaseEvent.name=nglp2, __FirebaseEvent.params=Bundle[{item_name=mcdonaldsjp://webapp　test, item_brand=plexure, method=share, index=3, event_id=2, item_category=NBS, content_type=news, item_id=1876}], acceptCookie=false}]"
isFinish = false

  // line  411 in BaseActivity
Intent intent = new Intent(this, event.getActivityClass());
intent.putExtras(event.getBundleData());
startActivity(intent);
if (event.isFinish()) this.finish();
```



# 记录首页的布局

- 首页的布局由一个带 header 的 recyclerview 组成（Fragment_home_new）

  - header：layout_news_header

- 但是这个 recyclerview 的 row view 比较复杂（content_news）

  - 新闻大图 news_image
  - 产品推荐列表 recommendedProductsLayout，包含一个 title（Featured）  和 recyclerview（用于显示产品图片和价格）（id：rvRecommendedProducts）
    - 产品列表的中单个产品的布局：product_list_item_new
    - 其中产品标签又是一个 recyclerview
  - 服务，当前为两个麦丹劳宅急送和 kodo，布局包含标题，两个服务的布局
  - newsPoint：pointcard 图

  注意：row 里面的元素不会都是显示的。有条件



# 店面优惠券下单页面逻辑

NewStoreChooseFragment 第四个 tab

点击 store，列表的 adapter 会把点击事件回调给 viewModel.isSaveStore

```kotlin
viewModel.isSaveStore.observe(this, Observer {
  if (it) {
    toMopCategory()
  }
})

toMop(storeViewModel.getStore(), menuType)

activity?.startActivity<MopCategoryActivity> {
            putExtra(MopCategoryActivity.PARAM_NAME_SELECTED_PRODUCT_MENU_ID, menuId)
            putExtra(MopCategoryActivity.PARAM_NAME_SELECTED_CATEGORY_ID, selectedCategoryId)
            putExtra(MopCategoryActivity.PARAM_NAME_SELECTED_IS_STORE_CLOSE, menuType == null)
            putExtra(MopCategoryActivity.PARAM_NAME_SELECTED_IS_SHOW_DT_ALERT, true)
            putExtra(MopCategoryActivity.PARAM_NAME_PRODUCT_IS_OFFER, selectedProduct?.isOffer)
            putExtra(MopCategoryActivity.PARAM_NAME_PRODUCT, selectedProduct)
        }

MopCategoryActivity.initViews:
loadRootFragment(R.id.fl_container, CategoryFragment.newInstance(selectedProductMenuId, isOffer, isStoreClose, isShowDtAlert, selectedProduct, selectedCategoryId))


TabPageIndicatorAdapter.getItem(): 
return OfferListFragment.newInstance(categoryList[position].name, selectedOfferId, selectedOfferInstanceUniqueId, isShowDtAlert)

ViewHolder:
itemBinding.couponUseButton.setOnClickListener {
  toDetail(offer)
}

ProductDetailsActivity.startOffer(context, offer, getString(R.string.product_category_coupon))
```

ProductDetailsActivity 也是一个空白的 Activity，需要根据 type（DetailsType）去加载对应的 Fragment

```kotlin
else if (type == DetailsType.coupon) {
  val offer = it.getSerializableExtra(PARAM_OFFER) as Offer
  val fragment = ProductDetailsOfferAddFragment.newInstance(offer)
  loadFragment(fragment)
} 
```

```kotlin
BaseApiFragment<VM : BaseApiViewModel, BD : ViewDataBinding> : NewBaseFragment<VM, BD>
	ProductDetailsFragment<VM : ProductDetailsViewModel, BD : FragmentProductDetailsBinding> ---loadImages() --- 在 onResume() 里面
		
		ProductDetailsMultipleFragment<VM : ProductDetailsMultipleViewModel, BD : FragmentProductDetailsBinding> 
---loadImages() -- onDataReady() --- from viewModel.multipleModel.startLoadingData(), ProductDetailsComboAddModel/ProductDetailsComboEditModel
			
			ProductDetailsComboEditFragment
			ProductDetailsComboAddFragment 
					: ProductDetailsMultipleFragment<ProductDetailsMultipleViewModel, FragmentProductDetailsBinding>
				ProductDetailsOfferAddFragment
			
		ProductDetailsMaxSetAddFragment ---loadImages() --- onDataReady() --- from view.loadData(productCode: Int, product: ProductCombo?)
		
		ProductDetailsAdditionAddFragment + loadImages() // this is update
		
		ProductDetailsSingleAddFragment ---loadImages() --- bindData() --- from viewModel.loadData(product)
		
		ProductDetailsSingleEditFragment（unused）---loadImages()
		ProductDetailsAdditionEditFragment (unused)
		
```

## 产品数据展示

NewBaseFragment 方法定义了模板方法调用，依次是：(除了第一个，其他都待实现)

```kotlin
binding = DataBindingUtil.inflate(inflater, layoutResId, container, false)

initViewModel() {viewModel = createViewModel(viewModelClass)}		
bindViewModel()
bindViews()
bindViews(savedInstanceState)
bindData()
```

NewBaseFragment 的子类：BaseApiFragment 抽象类， 定义了一个抽象方法待实现：

```kotlin
abstract fun onDataReady() // 在 apiCallFinishEvent 被监听到的时候调用

实现 bindData() 进行通用事件的监听 loadEvent(viewModel)
	loadingSpinnerEvent
	apiCallFinishEvent
	errorMessageEvent
```

BaseApiFragment 的子类：ProductDetailsFragment 抽象类，不过它没有抽象方法：

ProductDetailsMultipleFragment 抽象类

```kotlin
abstract fun bindModel() // 定义在 bindData() 方法体内
```

ProductDetailsComboAddFragment 最有意义的 bindModel() 方法已经被 ProductDetailsOfferAddFragment 重载，因此不需要看，核心：将传过来的 offer（优惠券和图片 url）构造 offerVM 传入 vm（傻逼操作，继承关系更蠢）

```kotlin
viewModel.bindData(ProductDetailsOfferModel(offer, imageUrl))

BaseViewModel:ViewModel
BaseApiViewModel
  ProductDetailsMultipleModel
    ProductDetailsComboAddModel
      ProductDetailsOfferModel
```

### 数据加载的入口

```kotlin
ProductDetailsMultipleFragment.bindData() ->  viewModel.multipleModel.startLoadingData()
// viewModel.multipleModel 已经被赋值为 ProductDetailsOfferModel(offer, imageUrl)
// 但是 startLoadingData() 属于 ProductDetailsComboAddModel
override fun startLoadingData() {
  if (comboItemList.isEmpty()) { 
    // comboItemEditList 是的 ProductDetailsMultipleModel mutableMapOf<Int, ComboItemData>()
    // 一开始肯定是 empty
    requestComboDataFromServer() 
    // 调用的是 ProductDetailsOfferModel.requestComboDataFromServer()....mlgb 重点
  }
}
```

### 数据流程追踪

- ProductDetailsOfferModel.requestComboDataFromServer() -> success -> 

  ```kotlin
  comboFromServer = data as ProductCombo
  updateItems()
  callUiWhenApiCallFinishedEvent()
  ```

  - comboFromServer = data as ProductCombo 

  - ProductDetailsComboAddModel.updateItems() -> updateBaseItems() -> 循环 comboFromServer.baseProducts 将 comboItemList（Map）按 0...baseProducts.size-1填充

    ```kotlin
    for (baseProduct in comboFromServer.baseProducts) {
      comboItemList[comboItemList.size] 
      	= ComboItemData(comboFromServer.code, baseProduct, productChoice = null, isBaseProduct = true)
    }
    ```

  - 继续 updateItems() -> 循环 comboFromServer.choices

    ```kotlin
    for (productChoice in comboFromServer.choices) {
      if (productChoice.preselectDefaultProduct == true) {
        comboItemList[comboItemList.size] = ComboItemData(comboFromServer.code, productChoice.defaultProduct, productChoice)
      } else {
        comboItemList[comboItemList.size] = ComboItemData(comboFromServer.code, null, productChoice)
      }
    }
    
    data class ComboItemData(
            val parentCode: Int,
            /**
             * The item product after choose the option, this can also be base product.
             */
            var itemProduct: Product? = null,
            /**
             * The product choice before choose the option.
             */
            var productChoice: ProductChoice? = null,
            /**
             * Check if the current product is a base product in combo.
             */
            var isBaseProduct: Boolean = false,
            /**
             * Check if the current product is a base product in combo.
             */
            var isAdditionalChoice: Boolean = false
    ) : Serializable
    ```

  - BaseApiViewModel.callUiWhenApiCallFinishedEvent -> apiCallFinishEvent.postValue(LiveEvent(true)) -> BaseApiFragment 监听 apiCallFinishEvent 执行 onDataReady+hideLoadingSpinner -> ProductDetailsMultipleFragment.onDataReady()

    - ```
      viewModel.onDataReady() //更新产品信息
      loadImages //更新产品图片
      adapter.notifyDataSetChanged()
      ```





ProductDetailsOfferAddFragment 父父父父类 ProductDetailsFragment.kt（**多继承的垃圾代码**）

ProductDetailsFragment.kt 205 加入购物车，addToCart 并且 popself

```kotlin
private fun addToCart() {
  viewModel.isError = false
  viewModel.addToOrUpdateCart()
  viewModel.isAddToCart.postValue(true)
  //        updateCartIcon(true)
  logEventAtFirebaseAnalytics(FirebaseTag.CustomEvent.ADD_TO_CART_SUCCESS)
  //        TrackUtil.menuDetailAddToCart(TrackUtil.MenuItem(viewModel.productName?.value,viewModel.productId?.value ?: 0,
  //                "product_item","",viewModel.productPrice.value?:""
  //        ))
}
```

回到 ProductDetailsOfferAddFragment.kt

```kotlin
viewModel?.isAddToCart?.observe(this, Observer {
  if (it) {
    Cart.sharedInstance().cacheOfferList(offer) // 用 sp gson 缓存 offer
  }
})
```

再回到 CategoryFragment 的 onSupportVisible，**checkOrder()**

```kotlin
override fun onSupportVisible() {
        super.onSupportVisible()
        isGotoEdit = false
        checkOrder()
        viewModel?.checkCouponCategory()
    }
```



# Combo 产品页面进入逻辑

```
ProductDetailsMultipleFragment -> init -> type: Int = it.getIntExtra(PARAM_TYPE, DetailsType.addition) -> type == DetailsType.combo -> checkComboStatus(comboCode: Int) -> loadFragment(ProductDetailsComboAddFragment.newInstance(comboCode, comboFromServer))

comboFromServer: ProductCombo


```



# 商店选择15个鸡块小食的下单流程

15个鸡块商品详情页面

```
ProductDetailsActivity -> type:addition -> ProductDetailsAdditionAddFragment.newInstance(it.getSerializableExtra(PARAM_ADDITION_PRODUCT) as Product) -> loadFragment -> 

ProductDetailsAdditionAddFragment:                 viewModel.loadData(it.getSerializable(PARAM_NAME_ADDTIONAL_CHOICE_PRODUCT) as Product)

// Product.additionalChoices 对象数组里面个数不同加载的产品选项 ui 也不同，主要是单选和多选的区别，具体查看数据结构
if (viewModel.isAdditional() > 0 && > 1) -> ProductSauseChoiceListFragment.newInstanceForResult(viewModel.curProduct, resultCallBack)) -> replaceFragment(fragment) 

viewModel.isAdditional() == 1 -> ProductChoiceListFragment.newInstanceForResult

// Product.additionalChoices.size 用于确定每个配料可选项的最大数量，代表可选配料（酱料）最大值

ProductDetailsAdditionAddFragment 继承 ProductDetailsFragment，视图：fragment_product_details

点击 checkout -> onClickBottomButton() -> goToCartScreen() -> OrderReviewActivity: 选择外卖，堂食，支付方式，最后 Confirm

confirm：触发 submitOrder: orderReviewViewModel?.submitOrder(orderReviewBinding.btSubmit), plxSDk.submitOrder, success -> isOrderSubmitted.value = true -> OrderReviewActivity(isOrderSubmitted:true) -> 

OrderConfirmActivity(selectOrderPickupType) -> “Send My Order to Kitchen” button -> btArrivedStore.clickListener -> orderViewModel.onArrivedStoreButtonClick() -> processOrderActionCheckIn:true -> orderViewModel.processOrderAction(ActionType.CHECK_IN) -> processOrderAction(data as FullOrder, actionType) -> ActionType.CHECK_IN -> PENDING: isOrderReadyForCheckIn:true -> showCheckInDialog() ->

isShowInputLayout = false -> ("are you nearby?" dialog)CheckInConfirmDialog.show -> btArrived.setOnClickListener -> onButtonClickListener?.onArrived() -> OrderConfirmActivity.pay() -> 

Cart.sharedInstance().getSelectedOrderPaymentType() == OrderPaymentType.LINE_PAY -> plxsdk.authorizeExternalPayment(orderId, paypay, successUrl, errorUrl, cancelUrl, callback) -> isPaid = false -> orderViewModel.authorizeLinePay() -> success -> isOrderAuthorized.value = LiveEvent(data as AuthorizePaymentProviderResponse) -> 

OrderConfirmActivity.orderViewModel.isOrderAuthorized -> startActivity<InAppBrowserActivity>(TAG_LINE_PAY) -> InAppBrowserActivity:shouldOverrideUrlLoading -> url.contains(Constants.MAIN_URL) -> inAppBrowserBinding.webView.loadUrl(newUrl) -> load new Url and redir -> mcd-jp-mobileorder://success -> setResult(Activity.RESULT_OK, intent) finish() -> OrderConfirmActivity.onActivityResult -> 

orderViewModel.checkInOrder(PaymentProvider.LINEPAY) -> checkInOrder.build() -> plxsdk.checkin -> isOrderCheckedIn.postValue(true) -> OrderConfirmActivity: orderViewModel.isOrderCheckedIn.observe -> startActivity<OrderCollectActivity> { putExtra("isFromCheckIn", true)} -> 

OrderCollectActivity（取餐号页面）
```



submit oder json:

```json
{
    "appBundleId":"jp.co.mcdonalds.android",
    "appVersion":"5.1.61(206)(50161)",
    "customerDetails":{
        "emailAddress":"john@theplant.jp",
        "firstName":"",
        "lastName":""
    },
    "items":[
        {
            "offerIsRepeatable":false,
            "items":[
                {
                    "offerIsRepeatable":false,
                    "productCode":6049,
                    "productName":"Mustard Sauce",
                    "quantity":1,
                    "shouldBeBaseItem":true,
                    "unitPrice":0.0
                },
                {
                    "offerIsRepeatable":false,
                    "productCode":6049,
                    "productName":"Mustard Sauce",
                    "quantity":1,
                    "shouldBeBaseItem":true,
                    "unitPrice":0.0
                }
            ],
            "productCode":1670,
            "productName":"Chicken McNuggets 15Pcs",
            "quantity":1,
            "shouldBeBaseItem":false,
            "unitPrice":580.0
        }
    ],
    "dayPart":"DAY",
    "orderType":"EATIN",
    "platform":"android",
    "storeId":13157,
    "storeName":"池袋北口店"
}
```

full order: 

```
{
    "appVersion":"5.1.61(206)(50161)",
    "consumerId":"500e6640-8b14-4644-8310-3621f88453ee",
    "customerDetails":{
        "emailAddress":"john@theplant.jp",
        "firstName":"",
        "lastName":""
    },
    "dateCreated":"2021-02-01T04:37:50.4591552Z",
    "displayOrderNumber":"M33",
    "items":[
        {
            "offerIsRepeatable":false,
            "items":[
                {
                    "offerIsRepeatable":false,
                    "productCode":6049,
                    "productName":"Mustard Sauce",
                    "quantity":1,
                    "shouldBeBaseItem":true,
                    "unitPrice":0.0
                },
                {
                    "offerIsRepeatable":false,
                    "productCode":6049,
                    "productName":"Mustard Sauce",
                    "quantity":1,
                    "shouldBeBaseItem":true,
                    "unitPrice":0.0
                }
            ],
            "productCode":1670,
            "productName":"Chicken McNuggets 15Pcs",
            "quantity":1,
            "shouldBeBaseItem":false,
            "unitPrice":580.0
        }
    ],
    "dayPart":"Day",
    "orderFullId":"2021020104375001315771629",
    "orderId":"71629",
    "orderType":"EatIn",
    "pickupType":"FrontCounter",
    "platform":"android",
    "status":"Pending",
    "storeId":"13157",
    "storeName":"池袋北口店",
    "totalAmount":580.0
}
```

authorized:

```
{"appVersion":"5.1.61(206)(50161)","consumerId":"500e6640-8b14-4644-8310-3621f88453ee","customerDetails":{"emailAddress":"john@theplant.jp","firstName":"","lastName":""},"dateCreated":"2021-02-01T04:37:50.4591552Z","displayOrderNumber":"M33","items":[{"offerIsRepeatable":false,"items":[{"offerIsRepeatable":false,"items":[],"productCode":6049,"productName":"Mustard Sauce","quantity":1,"shouldBeBaseItem":true,"unitPrice":0.0},{"offerIsRepeatable":false,"items":[],"productCode":6049,"productName":"Mustard Sauce","quantity":1,"shouldBeBaseItem":true,"unitPrice":0.0}],"productCode":1670,"productName":"Chicken McNuggets 15Pcs","quantity":1,"shouldBeBaseItem":false,"unitPrice":580.0}],"dayPart":"Day","orderFullId":"2021020104375001315771629","orderId":"71629","orderType":"EatIn","pickupType":"FrontCounter","platform":"android","status":"Pending","storeId":"13157","storeName":"池袋北口店","totalAmount":580.0}
```

before checkin:

```
{"appVersion":"5.1.61(206)(50161)","consumerId":"500e6640-8b14-4644-8310-3621f88453ee","customerDetails":{"emailAddress":"john@theplant.jp","firstName":"","lastName":""},"dateCreated":"2021-02-01T04:37:50.4591552Z","displayOrderNumber":"M33","items":[{"offerIsRepeatable":false,"items":[{"offerIsRepeatable":false,"items":[],"productCode":6049,"productName":"Mustard Sauce","quantity":1,"shouldBeBaseItem":true,"unitPrice":0.0},{"offerIsRepeatable":false,"items":[],"productCode":6049,"productName":"Mustard Sauce","quantity":1,"shouldBeBaseItem":true,"unitPrice":0.0}],"productCode":1670,"productName":"Chicken McNuggets 15Pcs","quantity":1,"shouldBeBaseItem":false,"unitPrice":580.0}],"dayPart":"Day","orderFullId":"2021020104375001315771629","orderId":"71629","orderTransaction":{"externalProvider":"paypay","paymentType":"ExternalProvider"},"orderType":"EatIn","pickupType":"FrontCounter","platform":"android","status":"Authorized","storeId":"13157","storeName":"池袋北口店","totalAmount":580.0}
```

checkin data:

```json
{
    "fullOrderId":"2021020104092501315771831",
    "orderType":"EAT_IN",
    "paymentProvider":"PAYPAY",
    "pickupType":"TABLE_SERVICE",
    "tableNumber":444
}
```



# 商店选择 Potenage DAI

```
type == DetailsType.combo -> checkComboStatus(comboCode 9100) -> Plx sdk get data from comboCode -> isMaxSet(comboFromServer): choices 数组里面对象的的 code 都相同 -> ProductDetailsMaxSetAddFragment -> 选择 sauce -> onClickBottomButton() -> addToCart() -> popSelf() -> CategoryFragment.onSupportVisible() -> checkOrder() -> val orderList = Cart.sharedInstance().getOrderItems() -> initOrderList() -> 

```

Plx sdk get data from comboCode:

```json
{"baseProducts":[{"categoryId":3,"productCode":1610,"displayOrder":5140,"productImages":["https://mopuatfd.plexure.io/product-images/product_mobile/1610"],"inOutage":false,"recommended":false,"localisedProductName":{"cartName":"Chicken McNuggets 5Pcs","menuName":"Chicken McNuggets 5Pcs","variantName":"Chicken McNuggets 5Pcs"},"dayPart":"BREAKFAST_DAY_MENU","productOptions":[],"priceList":[{"price":150,"priceCode":"EATIN"},{"price":150,"priceCode":"TAKEOUT"},{"price":150,"priceCode":"OTHER"}],"productClass":"PRODUCT","status":"ACTIVE"},{"categoryId":3,"productCode":1610,"displayOrder":5140,"productImages":["https://mopuatfd.plexure.io/product-images/product_mobile/1610"],"inOutage":false,"recommended":false,"localisedProductName":{"cartName":"Chicken McNuggets 5Pcs","menuName":"Chicken McNuggets 5Pcs","variantName":"Chicken McNuggets 5Pcs"},"dayPart":"BREAKFAST_DAY_MENU","productOptions":[],"priceList":[{"price":150,"priceCode":"EATIN"},{"price":150,"priceCode":"TAKEOUT"},{"price":150,"priceCode":"OTHER"}],"productClass":"PRODUCT","status":"ACTIVE"},{"categoryId":3,"productCode":2050,"displayOrder":5130,"productImages":["https://mopuatfd.plexure.io/product-images/product_mobile/2050"],"inOutage":false,"recommended":false,"localisedProductName":{"cartName":"McFry (L)","menuName":"McFry (L)","variantName":"McFry (L)"},"dayPart":"DAY_MENU","productOptions":[],"priceList":[{"price":200,"priceCode":"EATIN"},{"price":200,"priceCode":"TAKEOUT"},{"price":290,"priceCode":"OTHER"}],"productClass":"PRODUCT","status":"ACTIVE"}],"calculatedComboPrice":[{"price":500,"priceCode":"EATIN"},{"price":590,"priceCode":"OTHER"},{"price":500,"priceCode":"TAKEOUT"}],"choices":[{"choiceProductCode":7251,"defaultProduct":{"categoryId":0,"productCode":6048,"displayOrder":0,"productImages":["https://mopuatfd.plexure.io/product-images/product_mobile/6048"],"inOutage":false,"recommended":false,"localisedProductName":{"cartName":"BBQ Sauce","menuName":"BBQ Sauce","variantName":"BBQ Sauce"},"dayPart":"BREAKFAST_DAY_MENU","productOptions":[{"localisedProductName":{"cartName":"BBQ Sauce","menuName":"BBQ Sauce","variantName":"BBQ Sauce"},"dayPart":"BREAKFAST_DAY_MENU","optionType":"Sauce","productCode":6048},{"localisedProductName":{"cartName":"Mustard Sauce","menuName":"Mustard Sauce","variantName":"Mustard Sauce"},"dayPart":"BREAKFAST_DAY_MENU","optionType":"Sauce","productCode":6049},{"localisedProductName":{"cartName":"No Sauce","menuName":"No Sauce","variantName":"No Sauce"},"dayPart":"BREAKFAST_DAY_MENU","optionType":"Sauce","productCode":708001}],"priceList":[{"price":0,"priceCode":"EATIN"},{"price":0,"priceCode":"TAKEOUT"},{"price":0,"priceCode":"OTHER"}],"productClass":"NON_FOOD_PRODUCT","status":"ACTIVE"},"isHidden":false,"preselectDefaultProduct":false,"notPartOfOriginalCombo":true},{"choiceProductCode":7251,"defaultProduct":{"categoryId":0,"productCode":6048,"displayOrder":0,"productImages":["https://mopuatfd.plexure.io/product-images/product_mobile/6048"],"inOutage":false,"recommended":false,"localisedProductName":{"cartName":"BBQ Sauce","menuName":"BBQ Sauce","variantName":"BBQ Sauce"},"dayPart":"BREAKFAST_DAY_MENU","productOptions":[{"localisedProductName":{"cartName":"BBQ Sauce","menuName":"BBQ Sauce","variantName":"BBQ Sauce"},"dayPart":"BREAKFAST_DAY_MENU","optionType":"Sauce","productCode":6048},{"localisedProductName":{"cartName":"Mustard Sauce","menuName":"Mustard Sauce","variantName":"Mustard Sauce"},"dayPart":"BREAKFAST_DAY_MENU","optionType":"Sauce","productCode":6049},{"localisedProductName":{"cartName":"No Sauce","menuName":"No Sauce","variantName":"No Sauce"},"dayPart":"BREAKFAST_DAY_MENU","optionType":"Sauce","productCode":708001}],"priceList":[{"price":0,"priceCode":"EATIN"},{"price":0,"priceCode":"TAKEOUT"},{"price":0,"priceCode":"OTHER"}],"productClass":"NON_FOOD_PRODUCT","status":"ACTIVE"},"isHidden":false,"preselectDefaultProduct":false,"notPartOfOriginalCombo":true}],"productCode":9100,"productImages":["https://mopuatfd.plexure.io/product-images/product_mobile/9100"],"inOutage":false,"localisedComboName":{"cartName":"PoteNage DAI","menuName":"PoteNage DAI","variantName":"A la carte"}}
```



# 添加到购物车的数据结构、代码

```kotlin
// ProductDetailsComboAddModel.kt
comboFromServer?.let {
    return OrderItem(
            quantity, 0.0, false, null,null, false, it.code,
            getProductName(), getSubCartItems(), true, it, null, null, null)
}

comboFromServer: ProductCombo
getSubCartItems() -> ComboItemData

// ProductDetailsMaxSetAddViewModel.kt
getOrderItem() -> OrderItem(
                    quantity.value ?: 1, 0.0, false, null, null, false, it.code,
                    productName.value, getSubCartItems(), true, it, null, null, null

getSubCartItems() -> ComboItemData


// ProductDetailsOfferModel.kt

```

## Potenage DAI

```
orderItem.product: ProductCombo
```



# 登录和用户信息相关控件继承关系

```kotlin
RelativeLayout
	LoginCommonView<CharSequence>
		LoginBaseView
			LoginEditView ----> 密码邮箱控件
				LoginEditViewNew
					LoginEditZipCodeView
```

> 控件和业务逻辑耦合太高了

发现了一处 LoginEditView 控件的问题：在填写入内容的时候，父类 LoginCommonView 已经添加了 @OnTextChanged({R.id.loginEditText})，由于底层的持有 OnTextChangedListener 的是一个 list，子类的 @OnTextChanged 并不会覆盖父类的，因此：

- LoginCommonView.onTextChanged -> onValidate(false) -> doValidate(false): LoginBaseView 覆写了
- 接着：LoginEditView.onTextChanged -> LoginCommonView.onValidate(true: focused, s) -> LoginCommonView.onValidate(true) -> doValidate(true): LoginBaseView 覆写了

以上为重复调用问题的一个原因，还有第二个原因，ButterKnife 框架的误用：

ButterKnife 每 bind(view) 一次，底层就会使用 apt 生成一个 xxView_viewbinding$N 类来进行事件监听器的注册，findviewById 代码的编写，因此：

- 以上的继承关系，每个类里面都有 bind(view) 代码，意味着 apt 会生成多个内部类，也就意味着事件（onTextChanged）会添加多次，具体证据可以看 debug 下的调用 stack，可以看到多个 xxView_viewbinding$N 类，调用 onTextChangedListener



```kotlin
LoginBaseFragment // 大部通用逻辑写在了这里
	ReLoginFragment  // 布局总文件和少量控件逻辑在这里
```



## onFocusChanged 会触发 onValidate 的情况

view_login_edit.xml / no

fragment_offer_list.xml /no

view_account_edit.xml /no

view_login_switch.xml /yes /**unused**

view_login_edit_new.xml /no

view_login_spinner.xml /**yes / 账户编辑页面（fragment_account_edit_new.xml）** LoginSpinnerView（numberChildView），LoginGenderSpinnerView（genderSpinnerView）

view_login_switch_mango.xml /**yes / unused**

view_login_radio.xml /**yes** / LoginRadioView -> LoginBaseFragment, genderView -> fragment_login_user_info.xml, @+id/genderView, UserInfoFragment, 这些老的页面，在 TinyToolbarActivity 有类名使用，但是已经**不会再被调用**

view_login_date_picker.xml **/yes/ LoginDatePickerView -> fragment_account_edit_new.xml**、fragment_login_user_info.xml（**unused**） -> LoginBaseFragment, birthdayView, childBirthDayView



## LoginCommonView: TextWathcer(id=loginEditText) 会触发 onValidate(boolean focused, T0 s) 的地方

LoginCommonView 删除会产生的影响。下面的布局存在 id = loginEditText 的控件

```
view_login_edit.xml 布局不再被使用
view_account_edit -> LoginEditViewNew:LoginEditView:LoginBaseView -> fragment_account_edit_new.xml 账户信息编辑页
	memberIdView + nickNameView + mailAddressView + oldPasswordView
	
view_login_edit_new.xml -> LoginEditView:LoginBaseView
view_login_date_picker.xml -> LoginDatePickerView:LoginBaseView -> fragment_account_edit_new.xml 账户信息编辑页(生日控件和孩子生日控件)、fragment_login_user_info.xml（不再使用了）
```

**注意：private T0 validateArg; // note: when view is EditView, the member is a CharSequence reference**



# Usabilla 的相关功能

UsabillaManager

OrderCollectActivity



# 初始化工作和维护页面的显示

> https://cfg-japan-e-staging.vmobapps.com/v3/configurations 
>
> ```
> InitJob.init(true) -> initMain(initMode, ApiSuccessResultCallback<InitEvent>) -> ContentsManager.getSettingExtendedData(initMode, ApiResultCallback<List<Setting>>()) -> getVMobManagerInstance().getSettingExtendedData(initMode, apiResultCallback) -> getObjects -> CheckVMobSdkInit（Thread1，initMode：true） -> .... mainThread ->
> 
> McdonaldsJapanHomeActivity.onResume() -> BaseActivity.onResume() -> BaseActivity.onResumed() -> InitJob.init(false) -> initMain(initMode, ApiSuccessResultCallback<InitEvent>) -> ContentsManager.getSettingExtendedData(initMode, ApiResultCallback<List<Setting>>()) -> getVMobManagerInstance().getSettingExtendedData(initMode, apiResultCallback) -> getObjects -> CheckVMobSdkInit（Thread2，initMode：false）-> .... mianThread ->
> 
> ------------------ splash ----
> 
> MainActivity.onResume() -> BaseActivity.onResume() -> else if (!showCoachMarkActivity()) {onResumed();} -> MainActivity.onResumed() -> super.onResumed() -> InitJob.init(false) -> initMain(initMode, ApiSuccessResultCallback<InitEvent>) -> ContentsManager.getSettingExtendedData(initMode, ApiResultCallback<List<Setting>>()) -> getVMobManagerInstance().getSettingExtendedData(initMode, apiResultCallback)
> 
> getSettingExtendedData -> getObjects -> CheckVMobSdkInit -> SettingGetApi.getInstance().callApi()[mainTrhead] -> 启动子线程 -> 
> 
> initMode: true -> searchCriteria != null && isStaticInstance() {return _instance == this} -> isSkip: true -> [回到 mainThread]createInstance().callApi(callApiArg, searchCriteria, resultCallback) -> 启动子线程 -> （isStaticInstance()：false）导致 isSkip: false -> [回到 mainThread] isExpire(): true -> doCallApi(resultCallback) -> 启动子线程：TinyTask.doInBackground.doCallApiMain -> getObjects -> 
> SettingGetApi.getObjects -> VMob.getInstance().getConfigurationManager().getExtendedData
> 
> initMode: false -> searchCriteria == null -> isSkip: false -> [回到 mainThread] isExpire(): true -> ...
> ```

注意：连续多次调用 InitJob.init(false)，会在主线程执行多次 callApi，启动多个子线程，但是由于使用了 SettingGetApi.getInstance() 是单例，下面代码的 getSemaphore().tryAcquire() 会将单例对象的信号量变为 0，那么其余的子线程就会 tryAcquire 失败，进而 return null，在 onPostExecute 里面 isSkip == null  不会做请求操作，节流成功

```java
protected Boolean doInBackground(Void params) {
    Boolean isSkip;
    if (isStaticInstance() && searchCriteria != null) {
        isSkip = Boolean.TRUE;
    }
    else if (getSemaphore() == null) {
        isSkip = isStaticInstance();
    }
    else if (getSemaphore().tryAcquire()) {
        isSkip = Boolean.FALSE;
    }
    else {
        return null;
    }
    if (!isSkip) {
        if (getGroupSemaphore() != null) {
            getGroupSemaphore().acquireUninterruptibly();
        }
    }
    return isSkip;
}
```

因此，/v3/configurations 这个 api 的多次触发和多线程无关，只和每次页面打开就会触发一次请求执行有关（请求未必一定会发送，vmobsdk 里面也做了节流操作，短时间内的 rtt，仅第一次会发送请求，后面的重复请求似乎是直接使用缓存，Charles 只拦截到了一次请求，但是代码的 callback 里面数据回的）



InitJob.init(true) 和 InitJob.init(false) 区别：

initMain 的回调返回 InitEvent.EventId：goMaintenance， goVersionUp，leaveMaintenance

如果 leaveMaintenance && initMode(true) -> registrationSkipUser()：

```
第一次 !Preference.isLoggedIn() -> createSkipUser(true, apiResultCallback) -> syncUserConfig(false, new ApiSuccessResultCallback<JapanUserModel>()) -> getVMobManagerInstance().getConsumer() -> ConsumerGetApi ->                 new DoNext(userConfigs == null || userConfigs.size() == 0 ? baseUserModel : userConfigs.get(0)) ->                 copyUser(baseUserModel, userConfig) & getVMobManagerInstance().getConsumerTags() -> ConsumerTagsGetRequest -> onSuccess:doNext() -> apiResultCallback.onSuccess(userConfig) -> if(isClear){                 skipUserConfig = new JapanUserModel();} -> updateConsumer4NGLP2() 
```

- SettingGetApi：/v3/configurations
- copyUser(baseUserModel, userConfig) 网络获取下来的 userConfig 比本地创建的 baseUserModel 多了 UserName，并且也初始化过 memberId，详见：Consumer4Vmob 构造器





# 登录后：同步用户信息和 TAG

- ConsumerLoginApi：/logins，并且在执行这个 api 前会调用 /v3/configurations
- ConsumerGetApi：/consumers，数据：co.vmob.sdk.consumer.model.Consumer
- ConsumerPutApi：/consumers，这个类会先 get 一次然后 put
- ConsumerTagsGetRequest：/consumers/me/tagvalues

```
ContentsManager.login -> logout -> getVMobManagerInstance().login -> success -> syncUserConfig(false, cb)
```

## 同步用户信息 syncUserConfig

```
getVMobManagerInstance().getConsumer[api: /consumers] -> VMob.getInstance().getConsumerManager().getConsumer -> data: Consumer -> List<Consumer4Vmob>, Consumer4Vmob(consumer) -> List<JapanUserModel>// Consumer4Vmob: JapanUserModel -> DoNext(List<JapanUserModel>.get(0)) -> copyUser[本地 to 网络获取的user数据] -> getVMobManagerInstance().getConsumerTags[api: /consumers/me/tagvalues] -> VMob.getInstance().getConsumerManager().getTags -> data: List<String> -> List<Tag4Vmob>[0] -> List<Tag>[0] -> 使用 tags[0] 更新 userModel(本地写死了几个 userConfig.setCouponNews(Boolean.FALSE);
                            					userConfig.setSpecialGiftNews(Boolean.FALSE);
                            					userConfig.setProductNews(Boolean.FALSE);) 

-> cb.onSuccess(userModel) -> 更新 userModel：

loginUserConfig.setLoggedIn(Boolean.TRUE);
loginUserConfig.setConvertedUser(Boolean.TRUE);
loginUserConfig.setSkip(Boolean.FALSE);
//loginUserConfig.setIsConnectedFacebook(Boolean.FALSE);
loginUserConfig.setConnectedTwitter(Boolean.FALSE); // 到处为止 userModel 还没写回 sp

updateConsumer4NGLP2WithFelicaInit(true, loginUserConfig, apiResultCallback);

static public void updateConsumer4NGLP2WithFelicaInit(...) {
        JapanUserModel workUserConfig = Preference.getUserConfig();
        workUserConfig.setSkip(userConfig.getSkip());
        workUserConfig.setLoggedIn(userConfig.getLoggedIn());
        Preference.setUserConfig(workUserConfig); // 写入 sp
        
        userConfig.setLastRCN(workUserConfig.getLastRCN()); // R Card Number
        userConfig.setPendingRCN(workUserConfig.getPendingRCN());
        userConfig.setLastDCN(workUserConfig.getLastDCN()); // D Card Number
        userConfig.setPendingDCN(workUserConfig.getPendingDCN());
        {
            {
                final Tag.TagBuilder tagBuilder = Tag.builder();
                tagBuilder.isBluetooth(Preference.getConsumerTags4Bluetooth());
                tagBuilder.isLocationService(Preference.getConsumerTags4locationService());
                tagBuilder.isNGLP2(Boolean.TRUE); // 本地设置
                updateConsumer(isWriteBack, userConfig, tagBuilder, apiResultCallback);
            }
        }
    }

updateConsumer：
使用传入的 userConfig 构造 Consumer，调用 put /consumer -> success -> 
tagBuilder.isFelica(false);
tagBuilder.prefectureTag(zipcode.getPrefecture().getTag());
tagBuilder.childNumber(userConfig.getNumberChild() == null ? JapanUserModel.ChildNumber.error :userConfig.getNumberChild());

tagBuilder.couponNews(Boolean.FALSE);
tagBuilder.specialGiftNews(Boolean.FALSE);
tagBuilder.productNews(Boolean.FALSE);

tagBuilder.deprecatedPushMessage(userConfig.getDeprecatedPushMessage());
tagBuilder.registrationSkipped(userConfig.getSkip());
tagBuilder.registeredDPoint(userConfig.getRegisteredDPoint());
tagBuilder.registeredRPoint(userConfig.getRegisteredRPoint());

Tag tag = tagBuilder.build()
getVMobManagerInstance().putConsumerTags(tag, cb) -> VMob.getInstance().getConsumerManager().getTags -> success -> data: List<String> -> tag4: Tag4Vmob(data) -> tag4.setUpdateTags(tag) -> tag4.addTags 和 removeTags 都为空，即 tags 和本地相同，直接 cb.onSuccess(res: List<Tag4Vmob>)，否则 VMob.getInstance().getConsumerManager().updateTags(tag.getAddTags(), tag.getRemoveTags(), cb) 更新远端 -> cb.onSuccess(res) 注意这里返回的 res 是未更新前的 -> Preference.setUserConfig(userConfig) -> apiResultCallback.onSuccess(userConfig)[syncUserConfig 方法出传入的 cb]
```

### copyUser

```
// Tag/Preferenceにのみ保存されているものをコピー
dstUserConfig.setSkip(srcUserConfig.getSkip());

// Preferenceにのみ保存されているものをコピー
dstUserConfig.setLoggedIn(srcUserConfig.getLoggedIn());
dstUserConfig.setConvertedUser(srcUserConfig.getConvertedUser());
dstUserConfig.setConfirmWhenClaim(srcUserConfig.getConfirmWhenClaim());
dstUserConfig.setLastRCN(srcUserConfig.getLastRCN());
dstUserConfig.setPendingRCN(srcUserConfig.getPendingRCN());
dstUserConfig.setLastDCN(srcUserConfig.getLastDCN());
dstUserConfig.setPendingDCN(srcUserConfig.getPendingDCN());

AdvExId = null
AdvRsv = null
DPointNo = null
accessTokenFb = null
accessTokenTw = null
birthDay = null
childBirthDay = null
confirmWhenClaim = null
couponNews = {java.lang.Boolean@22360} false
deprecatedPushMessage = {java.lang.Boolean@22407} true
email = ""
firstName = ""
gender = {java.lang.Integer@22410} -1
isConnectedFacebook = {java.lang.Boolean@22360} false
isConnectedTwitter = {java.lang.Boolean@22360} false
isConvertedUser = {java.lang.Boolean@22407} true
isLoggedIn = {java.lang.Boolean@22407} true
isRegistered = {java.lang.Boolean@22360} false
isSkip = {java.lang.Boolean@22407} true
jobType = null
lastDCN = null
lastName = ""
lastRCN = null
mascotId = null
memId = null
memberId = "Cksrw==_"
newNGLP2appUser = {java.lang.Boolean@22360} false
nickName = null
numberChild = {java.lang.Integer@22413} 0
password = null
pendingDCN = null
pendingEcrm = null
pendingRCN = null
productNews = {java.lang.Boolean@22360} false
region = null
registeredDPoint = null
registeredRPoint = null
specialGiftNews = {java.lang.Boolean@22360} false
state = null
thirdPartySecretTw = null
thirdPartyUserIdTw = null
userId = null
userName = null
zipCode = null
```

### List(String) -> Tag4Vmob: Tag

Tag4Vmob 虽然继承 Tag，但是在转的过程中，如果当前 vmob 未登陆，会 setRegistrationSkipped(Boolean.TRUE)

```java
if (!VMob.getInstance().getAuthenticationManager().isLoggedIn()) {
  src.setRegistrationSkipped(Boolean.TRUE);
}
return src;
```

获取到的 Tag 数据 json

```json
{
	"tagValueReferenceCodes": ["mop_user", "profile_num_children_0", "bluetooth_disabled", "LastOpenedApp_0_1_Days", "profile_news_coupon_no", "SampleGroup7", "LastRedeemed_0_6_Days", "device-os-android", "profile_new_products_no", "profile_special_gift_no", "android_miseru", "Early_Engagement_201604", "Region_X", "location_service_disabled", "push_messages_disabled"]
}
```

字段：标签字符串数组里没有值，字段对应为 null

1. allTags： 为一个 HashMap<String, Boolean>()，所有的 Str 为 key，默认值 V 为 null
2. childNumber 对应 List 里面 profile_num_children_0，最后数字几就是几
3. couponNews: profile_news_coupon_yes/profile_news_coupon_no -> Boolean.TRUE/FALSE
4. specialGiftNews: profile_special_gift_yes/profile_special_gift_no
5. productNews: profile_new_products_yes/profile_new_products_no
6. isFelica: android_kazasu/android_miseru/iphone_miseru -> Boolean.true/false/false
7. deprecatedPushMessage：push_messages_enabled/push_messages_disabled -> t/f
8. registrationSkipped: registration_skipped ，有 true，无 false
9. isBluetooth：bluetooth_enabled/bluetooth_disabled
10. isLocationService：location_service_enabled/location_service_disabled
11. isNGLP2：NGLP2，有 true，无 false，如果是 NGLP1，也是 false
12. registeredDPoint：Registered_D_Point
13. registeredRPoint：Registered_R_Point



### setUpdateTags(tag) 逻辑

```
addTags = new ArrayList<>(); // 新增标签
removeTags = new ArrayList<>(); // 删除标签
```

使用 tag：Tag 构造出一个 updateTags：List<String>，遍历 updateTags，和事先构造好的 HashMap<String, Boolean> 的 List 做比较：HashMap<string, boolean> key 是 tag，一个 hashmap 是一个类型分组。这个 List 包含了所有 tag 值且用 map 分了组，组中只能有一个唯一选择。

allTags 包含刚刚获取下来（/consumers/me/tagvalues） tag 的值，addTags 放 allTag 里面没有的，但 updateTags 有，即新增的；

removeTags 放 allTag 里面有的，但是 updateTags 里面只有其同组的其他值，即从 allTag 里面删除的 tag



# 从 adb shell action.VIEW 打开 url 进去 app 的处理路径

命令：adb shell am start -a android.intent.action.VIEW -d "mcdonaldsjp://coffee-stamp-card?cid=1&wi=1"

- ```
  McdonaldsJapanHomeActivity.onCreate -> BaseActivity.onCreate(): args = getIntent().getExtras()
  
  McdonaldsJapanHomeActivity.onCreate: DeepLinkManager.INSTANCE.handleCustomDomain(getIntent(),this) 不处理
  
  McdonaldsJapanHomeActivity.onResume -> BaseActivity.onResume: 没有处理
  
  initJob 初始化后 -> McdonaldsJapanHomeActivity.onInitEvent(final InitEvent event) -> super.onInitEvent(event)
  -> BaseActivity.onIntnet(event.goStart) -> Intent mainIntent = getIntent() -> if (clickThroughUrl != null || gcmMessage != null) -> 构造 bundle 放入启动 MainActivity 的 Intent 启动 MainActivity
  
  // 构造的 bundle：Bundle[{fromSplash=true, isGcm=true, clickThroughUrl=mcdonaldsjp://coffee-stamp-card?cid=1}]
  ```

- ```
  MainActivity.onCreate -> BaseActivity.onCreate -> args = getIntent().getExtras() -> MainActivity.onResume -> BaseActivity.onResume -> if (args != null && args.getBoolean(BundleKeys.isGcm, false)) -> ScreenTransitionUtil.getScreenInfo(clickThroughUrl, new ScreenTransitionUtil.ScreenInfoListener() { -> openScreen(ScreenTransitionUtil.ScreenInfo screenInfo) -> if (screenInfo != null && screenInfo.getActivityClass() != BaseActivity.this.getClass() && screenInfo.isOuterLink()) -> screenInfo.postEvent(Uri.parse(clickThroughUrl), false, extras) -> onScreenEvent(ScreenEvent event) -> 
  
  ScreenEvent:
  activityClass = {java.lang.Class@14435} "class jp.co.mcdonalds.android.view.instantWin.coffeestamp.CSMainActivity"
  bundleData = {android.os.Bundle@22537} "Bundle[{fromSplash=true, cid=1, isGcm=true, clickThroughUrl=mcdonaldsjp://coffee-stamp-card?cid=1}]"
  isFinish = false
  
  ScreenTransitionUtil.getScreenInfo:
  screenInfo = {jp.co.mcdonalds.android.util.ScreenTransitionUtil$OuterLinkScreenInfoWMandatory4DPoint@22054} 
     mandatoryKeys = {java.util.Collections$SingletonList@22061}  size = 1 // elem: cid
     activityClass = {java.lang.Class@14435} "class jp.co.mcdonalds.android.view.instantWin.coffeestamp.CSMainActivity"
  ```

  

# 首页获取广告的数据

## api:/advertisements

```
VMob.getInstance().getAdvertisementsManager().getAdvertisements(builder.create(), resultCallback)
```

https://adv-japan-e-staging.vmobapps.com/v3/advertisements?channel=MAN&offset=0&limit=100&ignoreDailyTimeFilter=false

gr5egsu6liqvuzljsqyzwma6qvym2w4ydsizctbxsbdpyvwyyolq



# 首页广告（splash）业务规则

- 每天显示一次，相关字段 cmsSplash.newImpressionDate，每显示一次就会加一天，cmsSplash.newImpressionDate如果大于当前时间，那么就不显示
- 此外，还有次数限制，原始数据是 display_limt，后为 limit，计数的字段是 impressions，impressions > limit 表示为超过了最大显示次数，那么就不显示



# Twitter 注册的代码逻辑

```
doRegisterTwitter -> AuthJob.doRegisterTwitter(event) ->	
		JapanUserModel userConfig = new JapanUserModel();
		userConfig.setAccessTokenTw(accessTokenTw);
    userConfig.setThirdPartyUserIdTw(thirdPartyUserId);
    userConfig.setThirdPartySecretTw(thirdPartySecret);
    ...
		userConfig.setEmail(bundle.getString(LoginActivity.BundleKeys.email, null))
		userConfig.setPassword(bundle.getString(LoginActivity.BundleKeys.password, null));
		...
		
ContentsManager.signUp(userConfig, callback) -> 
		userConfig.setConnectedFacebook(Boolean.FALSE);
    userConfig.setConnectedTwitter(Boolean.FALSE);
    userConfig.setNewNGLP2appUser(Boolean.TRUE);
    getVMobManagerInstance().signUp(userConfig, callback) -> syncUserConfig(isWriteBack = false, callback)
    
    syncUserConfig:callback -> 
    		getVMobManagerInstance().getConsumer --> userConfigs.get(0)
    		DoNext -> getVMobManagerInstance().getConsumerTags -> 
    		callback.onSuccess(userConfig) -> 
    				newUserConfig.setCouponNews(Boolean.FALSE);
            newUserConfig.setSpecialGiftNews(Boolean.FALSE);
            newUserConfig.setProductNews(Boolean.FALSE);
            ...
            updateConsumer4NGLP2WithFelicaInit(true, newUserConfig, apiResultCallback);
            		JapanUserModel workUserConfig = Preference.getUserConfig();
                workUserConfig.setSkip(userConfig.getSkip());
                workUserConfig.setLoggedIn(userConfig.getLoggedIn());
                Preference.setUserConfig(workUserConfig);
                ...
                updateConsumer(isWriteBack, userConfig, tagBuilder, apiResultCallback);
										getVMobManagerInstance().putConsumer(userConfig, callback) ->
												success -> 
														...
														getVMobManagerInstance().putConsumerTags(tag, callback) -> 
																success -> 
																		if (isWriteBack = true) Preference.setUserConfig(userConfig);
																		putSubscriberKey(userConfig, apiResultCallback)
																	
         		ProductManager.INSTANCE.checkMaintenance(new ProductManager.MopStateListener()) -> 
								if (OverflowConfig.usingOverflow()) -> isOverflowEnabled() 
								-> TrackUtil.setBackend(remote = ‘mop’) -> 				FirebaseUtil.setUserIdAtFirebaseUserProperty("backend", backend) -> MyApplication.getContext().trackInfo

// updateConsumer4NGLP2WithFelicaInit 内的 updateConsumer 会调用 vmobsdk 进行异步请求，于是下面的 ProductManager.INSTANCE.checkMaintenance 方法就会继续执行，进而调用到 MyApplication.getContext().trackInfo() 而获取 email，这个时候由于异步，email 很可能并未存入 sp
```



# generic menu 路径和相关页面类

```
MenuRootFragment.newInstance() -> MenuMainFragment.newInstance() -> initAdapter() -> MenuPagerAdapter -> MenuListFragment
```



# Logout 流程

```
UserCenterActivity, ucTvLogout.setOnClickListener -> EventBus.getDefault().post(AuthEvent(AuthEvent.EventId.goLogout -> KBaseActivity, onAuthEvent(), AuthEvent.EventId.goLogout, dialog -> EventBus.getDefault().post(AuthEvent(AuthEvent.EventId.goLogout, AuthEvent.EventType.onSuccess -> onAuthEvent(), AuthEvent.EventId.goLogout, onSuccess -> EventBus.getDefault().post(AuthEvent(AuthEvent.EventId.doLogout -> EventBus.getDefault().post(AuthEvent(AuthEvent.EventId.doDeleteCoupon, event.eventId (AuthEvent.EventId.doLogout) -> onAuthEvent(), AuthEvent.EventId.doDeleteCoupon -> doDeleteCoupon() onSuccess -> AuthJob.onSuccess(event) -> EventBus.getDefault().post(new AuthEvent(event.getCallerEventId(), event.getCallerEventId(), AuthEvent.EventType.callback -> onAuthEvent(), AuthEvent.EventId.doLogout, AuthEvent.EventType.callback -> PointCardJob.logout(), AuthJob.doLogout(event) -> ContentsManager.logout -> AuthJob.onSuccess(event) -> 
```



# Store PLP 到 PDP

```
MopCategoryActivity -> CategoryFragment -> TabPageIndicatorAdapter -> ProductListFragment -> 
```



# DeepLink 到 Store PLP

```
BaseActivity.onScreenEvent(ScreenEvent event) -> checkToStore(event) -> ProductManager.INSTANCE.toStore -> checkOrder(context, selectedProduct, isDeepLink, isHomeBannerClick, isCheckOrder) -> ovf:StoreFinderActivity/jma:NewStoreChooseActivity -> StoreFinderFragment/NewStoreChooseFragment -> topMop()：

checkMaintenance():onLine() -> handleAppLinkProduct()
```



# titlebar 点击 pointcard 打开 dpointcard

```
dpoint 未登录：
Mainactivity: toolBarTvPointName onclick -> showPoint() -> rakuten/docomo(dialog) -> docomo -> showPointEvent(type) -> onPointCardStartEvent(event)/event.type = 1 -> PointCardJob.showDPointCard() -> 

DPointCardManager.getInstance().showPointCard() -> sdkContextInterface.showDpointCard();

dpoint 登录了：
Mainactivity: toolBarTvPointName onclick -> showPointEvent(PointCardJob.getSelectType())/ selectType:1 -> onPointCardStartEvent(event) -> PointCardJob.showDPointCard() -> DPointCardManager.getInstance().showPointCard() -> sdkContextInterface.showDpointCard()
```

简单记录一下打开 app 初始化 pointcard 的流程：

1. 初始化 dpoint sdk
2. 获取到 dp sdk 的 card number
3. 会和本地 userconfig 里面的 card number 比较一下
4. 不同则进行 api call 的更新

因此改变了 dpont 的账号后只有重启了，plx 后台的数据才会更新；同理，用户登录后输入 dp 账号也需要重启才会更新 plx 后台的 comminq



## 详细记录 dpoint 初始化的流程

1. 入口：PointCardJob.initialize，如果RC 不设置，会走 RPointGivePriorityToInitializer

2. 然后：initializeRPointCard -》initializeDPointCard -》DPointCardManager.getInstance().initialize

3. 然后：sdkStatus = SdkStatus.NotInitialized -》switch -》构造 sdkContextInterface 

4. 然后：sdkContextInterface.initialize(initializeListener) 进行初始化，结果通知到 initializeListener

5. initializeListener 会把结果通知到 initialize 之前注册的 DPointCardInitializeEvent 事件

6. 标记 ContentsManager.Preference.setPointCardDcmSdkInitResult(true)，sdkStatus = SdkStatus.Initialized

7. 通过 listener.onSuccess() 回调到 PointCardJob，步骤 2，然后 onInitializeSuccess()

8. 然后：onInitializeSuccess 会依次做两个事：1/onPointCardInit(event)，2/post PointCardInitEvent

   1. 然后：onPointCardInit -》PointCardJob.changeSelectType 发送了 PointCardSettingUpdateEvent。

      再然后是 PointCardJob.doPointCardResult(TAG)：

      1. PointCardJob.doPointCardResult 核心是注册 registDPointCardNumber/registRPointCardNumber
      2. registDPointCardNumber -》 DPointCardManager.getInstance().waitOnInitialized -》onSuccess -》new PostCardNumber() 目的是 post CardNumber，**但是这里代码有问题，似乎并没有 post 上去，必须要kill-start app 才能完成 plx 后台的更新**

   2. post PointCardInitEvent：这个主要是用于更新 UI 的

   



# Main Tab Coupon HappyMeal 顶部 banner 点击

```
CouponAdapter.initBannerView -> DebounceHelper.click(bannerImage, 500, Runnable { ->                 EventBus.getDefault().post(CouponBannerClickEvent(coupon.url)) -> CouponMainFragment.onCouponBannerClickEvent -> ScreenTransitionUtil.onClickThroughUrl(couponBannerClickEvent.url, false, null) -> Uri uri = Uri.parse(urlList) mcdonaldsjp://coupons?cat=hm&i1=32452 -> ScreenTransitionUtil.ScreenTransition: OuterLinkScreenInfoCouponNew -> screenInfo.postEvent(uri, isFinish, null) -> BaseActivity.onScreenEvent(ScreenEvent event) -> ((MainActivity) this).checkDeepLinkIntent(event) -> KEY_INTENT_COUPONS -> {navigateToCoupon(event.bundleData, true)} -> (mFragments[COUPON] as CouponMainFragment).checkDeepLinkData(bundle) -> newTargetCoupon = targetCoupon
```

2::20210604050312-https://www.stg.mcd.qorcommerce.com/media_library/3309/file.jpg-hm

- bannerUrl：mcdonaldsjp://coupons?cat=hm&i1=32452



# generic Menu -> order -> store finder -> 显示store 记录

```
ProductManager.checkOrder -> NewStoreChooseActivity -> load NewStoreChooseFragment -> mapView.setOnCameraIdleListener -> viewModel.loadStores(visibleRegion) -> loadDataWithCurrentLocation(currentVisible.center, radius.toInt()) -> isDataReady.value = true -> viewModel.fillStoreList(featureFilters) -> val storeVM = StoreViewModel.newInstance(it[0]), storeVM.storeType = StoreViewModel.StoreType.LASTORDER -> storeVMs.add(0, storeVM) -> storesListAdapter.updateItems(it)
```



## 下单后 store 被保存在 lastOrder 里面，然后通知store finder tab 刷新

```
HomeNewFragment.onResume -> homeViewModel?.getLastOrder() -> lastOrder(ProductManager.getCacheLastOrder()), SpUtil.get(PRODUCT_MANAGER_CACHE_NAME)?.getString(KEY_LAST_ORDER, "") -> checkLastStore(it) -> 	StoreCache.cacheLastOrderStore(store), EventBus.getDefault().post(StoreLastOrderUpdateEvent()) -> NewStoreChooseFragment.onStoreLastOrderUpdateEvent -> viewModel.fillStoreList(featureFilters) -> val lastOrderStoreList = getLastOrderStore(): ProductManager.getCacheLastOrder() 提取 store ->  val storeVM = StoreViewModel.newInstance(it[0]) -> storeVMs.add(0, storeVM) -> storesListAdapter.updateItems(it)
```





# store finder -> 显示store 记录 -> 点击 store 打开产品购买

```
StoresListAdapter -> clickListener = OnClickListener {} -> isSaveStore.postValue(true) -> toMopCategory() -> 
```







