# 反射获取泛型类型

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

  fun getFragment(pos: Int): Fragment? {
    return fa.supportFragmentManager.findFragmentByTag("f" + pos)
  }
}
```



# ViewSwitcher + 上出下入的动画

```kotlin
class EmergenciesNotifier(context: Context?, attrs: AttributeSet?) : TextSwitcher(context, attrs) {

    companion object {
        private const val SCHEDULE_PERIOD = 2_000L
        private const val WHAT_SCHEDULE = 1;
    }

    private val announcementList = mutableListOf<Announcement>()
    private var count = 0
    private val mainHandler = object : Handler(Looper.getMainLooper()) {

        override fun handleMessage(msg: Message) {
            if (msg.what == WHAT_SCHEDULE) {
                if (announcementList.isNotEmpty()) {
                    val index = count++.rem(announcementList.size)
                    if (count == announcementList.size) count = 0
                    setText(announcementList[index].title)
                    // logDebug( "run: ")
                }
                if (!hasMessages(WHAT_SCHEDULE)) {
                    sendEmptyMessageDelayed(WHAT_SCHEDULE, SCHEDULE_PERIOD)
                }
            }
        }
    }


    init {
        val padding = TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP, 12f,
                resources.displayMetrics).toInt()

        setFactory {
            var textView = TextView(context)

            TextViewCompat.setTextAppearance(textView, R.style.fontStyle_JP_Medium)
            textView.ellipsize = TextUtils.TruncateAt.END
            textView.maxLines = 1
            textView.setTextColor(Color.parseColor("#DA111E"))
            textView.setTextSize(TypedValue.COMPLEX_UNIT_SP, 13f)
            textView.setPadding(0, padding, 0, padding)

            textView
        }
        setPadding(padding, 0, padding, 0)

        val inAnim = TranslateAnimation(
                TranslateAnimation.RELATIVE_TO_PARENT, 0F,
                TranslateAnimation.RELATIVE_TO_PARENT, 0F,
                TranslateAnimation.RELATIVE_TO_PARENT, 1F,
                TranslateAnimation.RELATIVE_TO_PARENT, 0F).apply {
            duration = resources.getInteger(android.R.integer.config_longAnimTime).toLong()
        }
        val outAnim = TranslateAnimation(TranslateAnimation.RELATIVE_TO_PARENT, 0F,
                TranslateAnimation.RELATIVE_TO_PARENT, 0F,
                TranslateAnimation.RELATIVE_TO_PARENT, 0F,
                TranslateAnimation.RELATIVE_TO_PARENT, -1F).apply {
            duration = resources.getInteger(android.R.integer.config_longAnimTime).toLong()
        }
        inAnimation = inAnim
        outAnimation = outAnim


        val data = "[\n" +
                "{\n" +
                "title: \"全国 131 店舗で展開中の “McCafé by Barista” にて エスプレッソを使用した全 14 商品が 2 年ぶりに生まれ変わります！ エチオピア豆 50％使用したフルーティーで華やかな香りが特長 12 月 2 日 (水) よりご提供開始\",\n" +
                "link: \"https://www.mcdonalds.co.jp/company/news/2020/1124a/\"\n" +
                "},\n" +
                "{\n" +
                "title: \"新型コロナウイルス感染拡大防止の取り組みとお知らせ\",\n" +
                "link: \"https://www.mcdonalds.co.jp/company/news/200304a/\"\n" +
                "},\n" +
                "{\n" +
                "title: \"日本マクドナルドでは、飲食店としてお客様に温かいお食事をご提供する社会的役割を果たしつつ、お客様、従業員をはじめ全ての皆様の安全を最優先し、感染症対策として、全国の店舗で以下の取り組みを行っております。\",\n" +
                "link: \"\"\n" +
                "}\n" +
                "]"
        val type = object : TypeToken<List<Announcement>>() {}.type
        announcementList.addAll(Gson().fromJson(data, type))
    }

    override fun onVisibilityChanged(changedView: View, visibility: Int) {
        super.onVisibilityChanged(changedView, visibility)
        // logDebug("onVisibilityChanged: $visibility")
        when (visibility) {
            INVISIBLE, GONE -> {
                if (mainHandler.hasMessages(WHAT_SCHEDULE)) {
                    mainHandler.removeCallbacksAndMessages(null)
                    // logDebug("stop  gone")
                }
            }
            VISIBLE -> {
                if (!mainHandler.hasMessages(WHAT_SCHEDULE)) {
                    mainHandler.sendEmptyMessage(WHAT_SCHEDULE)
                    // logDebug("start  visible")
                }
            }
        }
    }

    override fun onAttachedToWindow() {
        super.onAttachedToWindow()
        // logDebug("onAttachedToWindow")
        mainHandler.sendEmptyMessage(WHAT_SCHEDULE)
    }

    override fun onDetachedFromWindow() {
        super.onDetachedFromWindow()
        // logDebug("onDetachedFromWindow")
        mainHandler.removeCallbacksAndMessages(null)
    }

    fun setAnnouncementList(list: List<Announcement>) {
        announcementList.clear()
        announcementList.addAll(list)
    }
}
```