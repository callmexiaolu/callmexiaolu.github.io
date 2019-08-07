## ViewPager使用遇到的问题：

### ViewPager中的item显示在容器外

如果想要达到一下效果，那么可以在父容器中设置属性 android:clipChildren，该属性意思是：是否限制子View在其范围内。默认为true，因此我们设置为false，就可以让子view显示在外面了。

![](http://github.com/callmexiaolu/callmexiaolu.github.io/raw/master/img/post-8-7-1.png)

### ViewPager设置滑动动画

ViewPager提供了相对的接口来进行滑动动画，实现该ViewPager.PageTransformer就可以了。

```java
public interface PageTransformer {
    /**
     * Apply a property transformation to the given page.
     * 对给定的page施加一个属性切换效果
     *
     * @param page Apply the transformation to this page
                   将转换效果引用到此page
     * @param position Position of page relative to the current front-and-center                 
     *                 position of the pager. 0 is front and center. 1 is one full             
     *                 page position to the right, and -1 is one page position to the left.
     *                 相对于当前处于中心显示的page的postion。
     *                 0是当前页面。1是右侧页面。
     *                 -1是左侧页面。
     */
    void transformPage(View page, float position);
}
```



这里实现了一个滑动时，item进行放大缩小的效果：

```kotlin
/**
 * ViewPager的item滑动切换时展示的切换效果
 */
const val DEFAULT_MIN_SCALE = 0.85f

class ShadowTransformer : ViewPager.PageTransformer {

    private var mMinScale = DEFAULT_MIN_SCALE
    private lateinit var mPageTransformer : ViewPager.PageTransformer

    constructor() : super()

    constructor(minScale: Float) : this(minScale ,NoPageTransformer.getInstance())

    constructor(pageTransformer: ViewPager.PageTransformer): this(DEFAULT_MIN_SCALE ,pageTransformer)

    constructor(minScale: Float, pageTransformer: ViewPager.PageTransformer) {
        mMinScale = minScale
        mPageTransformer = pageTransformer
    }


    override fun transformPage(page: View, position: Float) {
        val pageWidth = page.width.toFloat()
        val pageHeight = page.height.toFloat()

        page.pivotY = pageHeight / 2
        page.pivotX = pageWidth / 2
        if (position < -1) { // [-Infinity,-1)
            // This page is way off-screen to the left.
            page.scaleX = mMinScale
            page.scaleY = mMinScale
            page.pivotX = pageWidth
        } else if (position <= 1) { // [-1,1]
            // Modify the default slide transition to shrink the page as well
            if (position < 0)
            //1-2:1[0,-1] ;2-1:1[-1,0]
            {

                val scaleFactor = (1 + position) * (1 - mMinScale) + mMinScale
                page.scaleX = scaleFactor
                page.scaleY = scaleFactor

                page.pivotX = pageWidth * (0.5f + 0.5f * -position)

            } else
            //1-2:2[1,0] ;2-1:2[0,1]
            {
                val scaleFactor = (1 - position) * (1 - mMinScale) + mMinScale
                page.scaleX = scaleFactor
                page.scaleY = scaleFactor
                page.pivotX = pageWidth * ((1 - position) * 0.5f)
            }


        } else { // (1,+Infinity]
            page.pivotX = 0f
            page.scaleX = mMinScale
            page.scaleY = mMinScale
        }
    }

}
```

该接口的方法参数相关变化验证以及动画实现可以看简书的文章：[ViewPager PageTransformer探索](https://www.jianshu.com/p/11a819bc5973)

### ViewPager使用

直接设置对应的adapter就好，api提供了PagerAdapter，直接继承实现就好。

```kotlin
class BannerAdapter : PagerAdapter() {

    override fun instantiateItem(container: ViewGroup, position: Int): Any {
        val view = LayoutInflater.from(container.context).inflate(R.layout.item_view_pager_banner, null)
        //val view = ImageView(this@BannerShowActivity)
        //view.scaleType = ScaleType.FIT_XY
        //view.setImageResource(img[position])
        container.addView(view)
        return view
    }

    override fun isViewFromObject(view: View, o: Any): Boolean  = view == o

    override fun getCount(): Int = 6

    override fun destroyItem(container: ViewGroup, position: Int, `object`: Any) {
        container.removeView(`object` as View)
    }
```

#### PagerAdapter方法解析

Ps:这里引用了博客[ViewPager与PagerAdapter深度解析](https://blog.csdn.net/zzxzhyt/article/details/50689308)一段话

* **instantiateItem(ViewGroup,int)**
  该方法在ViewPager.addNewItem(int, int)中被调用，主要作用是创建一个页面的Content View。这里再强调一遍，在重写这个方法时候，需要主动将该Content View添加到ViewPager里面去,因为ViewPager没有主动添加该Content View为其Child View

* **destroyItem(ViewGroup, int, Object) **
  相应的ViewPager不会主动的添加Content View，也不会主动的移除Content View,所以亦需要在重写该方法的时候主动从ViewPager中移除该Content View。在ViewPager以下方法会调用destroyItem(ViewGroup, int, Object)：

  ​			ViewPager.setAdapter().这里调用该方法去销毁旧Adapter的View

  ​			ViewPager.dataSetChanged(),这里调用该方法去销毁过期数据的View

  ​			ViewPager.populate().ViewPager有一个ViewPager会保持一定的页面数，所以这里要把不再持有的页	面以及相应的Content View销毁。

* **isViewFromObject(View,Object) **
  我们需要重写isViewFromObject(View,Object) 让ViewPager.infoForChild(View )能够正确判断某个Child View对应的ItemInfo信息。这个方法我们不重写也不会对ViewPager有影响。

* **getCount()**
  返回ViewPager中item的个数



### ViewPager实现只加载当前页面内容

如果 ViewPager中使用的item是fragment。在Fragment中有个方法：

```java
  public void setUserVisibleHint(isVisibleToUser: Boolean){
   ..........
 }
```

同时也有个方法：

```java
public boolean getUserVisibleHint() {
        return this.mUserVisibleHint;
    }
```

第一个方法是设置Fragment是否为用户所见，第二个方法是返回当前Fragment是否能被用户所见。

因此我们在进行数据加载的时候可以根据getUserVisibleHint()返回的值来进行判断是否加载。

**我为啥用这个方法？**

因为我使用一个Fragment，里面包含了ViewPager，然后ViewPager的item是两个Fragment，两个Fragment厨初始化的时候就会进行刷新数据。场景要求进入外层Fragment的时候请求一次数据，然后在ViewPager中进行下拉刷新的同时也请求外层Fragment刷新数据的方法。



我在刷新事件中进行判断fragment是否为用户所见，从而进行外层的数据的刷新，这样子减少了一次数据请求。因为原来是外层请求一次，内层各一次，就是三次。也算是一点小优化吧。



***当然最好的方法是进行懒加载***



为啥？我这里虽然没有进行请求数据，然后内部的数据以及对象已经初始化了，这是因为ViewPager自带的预加载功能，默认预加载下一个页面。

### ViewPager的懒加载

ViewPager原本的本质就是提前加载下一个页面，这样子就方便用户进行查看，但是如果加载的数据量过大就会造成卡顿，降低用户体验。因此就出现了懒加载——只加载当前页面。

使用的例子：

![](https://img-blog.csdn.net/20160220161317443)



当然你可以设置ViewPager的setOffscreenPageLimit()方法，该方法设置最大能加载缓存的item数量，即允许多少page存在屏幕之外。源码中默认为1，因此设置为0也不能实现只加载当前页面。



#### 懒加载如何实现？

##### 前置知识——熟知Fragment的生命周期



