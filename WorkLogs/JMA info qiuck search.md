# 项目信息

STAG 测试页面 url：

- https://mcd:mcd%20works@cms-cdn.mcd.theplant-dev.com/jma_test/



# 代码追踪

### ://order 的 Deeplink 处理

```shell
Deeplink -> McdonaldsJapanHomeActivity#onResume, onResumed() -> InitJob.init(false) -> InitJob.onSuccess(event) -> EventBus.getDefault().post(event) -> McdonaldsJapanHomeActivity#onInitEvent(InitEvent event) -> case goStart: -> if (mainIntent != null) { ... } -> 
if (clickThroughUrl != null || gcmMessage != null) { 
	bundle.putBoolean(BundleKeys.isGcm, true);
	bundle.putString(BundleKeys.clickThroughUrl, clickThroughUrl)
} -> intent.putExtras(bundle), startActivity(intent) -> 

MainActivity#onResume -> else if (args != null && args.getBoolean(BundleKeys.isGcm, false)) { -> ScreenTransitionUtil.getScreenInfo(clickThroughUrl, new ScreenTransitionUtil.ScreenInfoListener() { -> 

mcdonaldsjp://order -> OuterLinkScreenInfo -> if (screenInfo != null && screenInfo.getActivityClass() != BaseActivity.this.getClass() && screenInfo.isOuterLink()) { -> screenInfo.postEvent(Uri.parse(clickThroughUrl), false, extras) -> 

MainActivity(BaseActivity)#onScreenEvent(ScreenEvent event) -> checkToStore(event) -> if (StoreChooseActivity.class.equals(event.getActivityClass())) { -> SelectedProduct selectedProduct = new SelectedProduct -> ProductManager.INSTANCE.toStore -> checkOrder -> context?.startActivity(intent), StoreFinderActivity
```



### 菜单加载的流程，点击菜单项

```
MenuRootFragment -> MenuMainFragment -> initAdapter(menuHeader: MenuHeader) -> MenuListFragment，每个 category 一个 -> bindData() -> checkData() -> viewModel.getMenuProductGroups(productCollectionsId) -> viewModel.productGroupList.observe -> initAdapter(it) -> initBannerView, initMenuGroupView -> 产品对应ProductAdapter -> helper.getView<View>(R.id.cardView).setOnClickListener { -> MenuDetailActivity.start
```

