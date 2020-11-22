# 反射

匿名子类获取父类的第一个泛型参数类型，适合写 Callback 的时候使用。

```java
public Type getGenericType() {
  Type genericSuperclass = getClass().getGenericSuperclass();
  if (genericSuperclass instanceof Class) {
    throw new RuntimeException("Missing type parameter.");
  }
  ParameterizedType parameterizedType = (ParameterizedType) genericSuperclass;
  return parameterizedType.getActualTypeArguments()[0];
}
```



# Viewpager2 显示部分左右 slide

```kotlin
private fun initViewPager2(couponList: List<CouponRedeemed>) {
  val pagerAdapter = QrcodeSlidePagerAdapter(this, couponList)
  val horizontalMarginPx = MyApplication.getContext().dpToPx(30f).toInt()
  viewPager2.addItemDecoration(HorizontalMarginItemDecoration(this, horizontalMarginPx))

  val nextItemVisiblePx = MyApplication.getContext().dpToPx(18f).toInt()
  val pageTranslationX = nextItemVisiblePx + horizontalMarginPx
  val pageTransformer = PageTransformer { page: View, position: Float ->
                                         page.translationX = -pageTranslationX * position
                                         
                                         // Next line scales the item's height. 
                                         // You can remove it if you don't want this effect
                                         // 高度缩放效果，不使用可以移除
    																			page.scaleY = 1 - (0.25f * abs(position))
                                        }
  viewPager2.setPageTransformer(pageTransformer)

  viewPager2.offscreenPageLimit = 1
  viewPager2.adapter = pagerAdapter

  viewPager2.registerOnPageChangeCallback(object : ViewPager2.OnPageChangeCallback() {
    override fun onPageSelected(position: Int) {
      pageIndicator.move(position)
    }
  })
}
```

```kotlin
class HorizontalMarginItemDecoration(context: Context, horizontalMarginPx: Int) 
: RecyclerView.ItemDecoration(){

  private val horizontalMarginPx: Int = horizontalMarginPx

  override fun getItemOffsets(outRect: Rect, view: View, parent: RecyclerView, state: RecyclerView.State) {
    outRect.right = horizontalMarginPx
    outRect.left = horizontalMarginPx
  }
}

private class QrcodeSlidePagerAdapter(fa: FragmentActivity, couponList: List<CouponRedeemed>) 
: FragmentStateAdapter(fa) {

  val couponList = couponList

  override fun getItemCount(): Int {
    return couponList.size
  }

  override fun createFragment(position: Int): Fragment {
    var couponRedeemed = couponList[position]
    return CouponQrcodeFragment.newInstance(couponRedeemed.mergedId)
  }
}
```



