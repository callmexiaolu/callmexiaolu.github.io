## 谈谈Android中的View

在手机上，我们看到的一切都是view的呈现，安卓提供了一套控件库，但是有时候系统提供的控件并不能满足项目中的需求，因此需要进行自定义控件，这需要我们掌握view的工作原理才能更好的进行自定义。

### View的层级关系

安卓中所见的控件的来源几乎都是基于View和ViewGroup创建出来的。View是基本的控件元素，实现了触摸以及动画回调接口，所以能够处理交互事件，而ViewGroup继承于View，实现了ViewParent, ViewManager接口，所以能够进行对View的管理，作为View的容器。这两个类构成了安卓中的View树🌲结构。



![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1564426878011&di=f4d5a90c07506902e12f52ba3a0df79e&imgtype=0&src=http%3A%2F%2F5b0988e595225.cdn.sohucs.com%2Fimages%2F20190629%2F53e1723c2b9a496dafe993c5215364fa.jpeg)

​																								图1

就像大树一样，养分从根部传递到各处，安卓的view树也一样，从根部到各处的叶子🍃，一切的源头开始都是从根开始。

![](https://img-blog.csdn.net/20160408105916977)

​																						图2

### 树根（ViewRootImpl）

以下用树根替代ViewRootImpl类进行描述。树根中有几个方法很重要，如下

```java
//负责测量
 private void performMeasure(int childWidthMeasureSpec, int childHeightMeasureSpec) {
        if (mView == null) {
            return;
        }
        Trace.traceBegin(Trace.TRACE_TAG_VIEW, "measure");
        try {
            mView.measure(childWidthMeasureSpec, childHeightMeasureSpec);
        } finally {
            Trace.traceEnd(Trace.TRACE_TAG_VIEW);
        }
    }
//确定放置位置
private void performLayout(WindowManager.LayoutParams lp, int desiredWindowWidth,
            int desiredWindowHeight) {
      .............
        final View host = mView;
        if (host == null) {
            return;
        }
        Trace.traceBegin(Trace.TRACE_TAG_VIEW, "layout");
        try {
            host.layout(0, 0, host.getMeasuredWidth(), host.getMeasuredHeight());
          ...............
        }finally {
           ........
        }
}
//绘制
performDraw(){
  .......
    
}
```

这三个方法中调用了被依次调用，方法中调用顶层View的方法，完成树根（顶层View）的measure，layout，draw。

![](http://image.webreader.duokan.com/mfsv2/download/fdsc3/p01s50KI6XvJ/wPxRNtLOeLwo2E.jpg?thumb=1024x&scale=auto)

​																						图3



##### measure，layout，draw是啥？

👇下面用一张图来解释：

![](https://img-blog.csdn.net/20170119233820894?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

​																							图4

Android视图工作机制按顺序分为以下三步：

1. measure：确定View的宽高——测量

2. layout：确定View的位置——放置

   放置是通过view的上左右，下左右四个点的坐标进行放置。

3. draw：绘制出View的形状——绘制

4. MeasureSpec：测量规格——存储着大小的模式和值

   MeasureSpec是一个int型的整数，一个int类型有32位，最高两位表示模式，后面表示值。

所以，进行完这几步之后，View就出来了。

这里我们暂时就不过多去研究其背后绘制的原理等等。

### 枝干🌿（ViewGroup）

顾名思义，枝干容纳着许多树枝和树叶，ViewGroup作为一个容器，能够包含View和ViewGroup组成千变万化的视图。枝干需要知道树叶的大小才能供给相应的养分，同样，ViewGroup也需要知道View的大小才能更好的选择自身的大小。（即测量自身大小需要知道子view的大小，因为自身是一个容器）

```java
//ViewGroup中的测量子View们的大小方法
protected void measureChildren(int widthMeasureSpec, int heightMeasureSpec) {
       final int size = mChildrenCount;
       final View[] children = mChildren;
  //可以看出，这里是进行了遍历
       for (int i = 0; i < size; ++i) {
           final View child = children[i];
           if ((child.mViewFlags & VISIBILITY_MASK) != GONE) {
               measureChild(child, widthMeasureSpec, heightMeasureSpec);
           }
       }
   }

//测量子View的大小
protected void measureChild(View child, int parentWidthMeasureSpec,
        int parentHeightMeasureSpec) {
    
    final LayoutParams lp = child.getLayoutParams();
    final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
            mPaddingLeft + mPaddingRight, lp.width);
    final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
            mPaddingTop + mPaddingBottom, lp.height);
    child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
}

//ViewGroup类,继承View，实现ViewParent, ViewManager接口
@Override
    public final void layout(int l, int t, int r, int b) {
        if (!mSuppressLayout && (mTransition == null || !mTransition.isChangingLayout())) {
            if (mTransition != null) {
                mTransition.layoutChange(this);
            }
            super.layout(l, t, r, b);
        } else {
            // record the fact that we noop'd it; request layout when transition finishes
            mLayoutCalledWhileSuppressed = true;
        }
    }
 @Override
    protected abstract void onLayout(boolean changed,
            int l, int t, int r, int b);
```

树干千变万化(Android中的ViewGroup所衍生的各种容器也是一样，所以官方使用了抽象方法，让自己去实现onLayout()方法)，树干会传递养分给下一级，ViewGroup也一样，会传递一些事件（触摸事件，滑动事件等）给下一级的view，下一级接着再传递下一级。

### 树叶🍂（View）

树叶是整颗树的最外层，一眼望去最直观的表现。当然在Android中，View不仅仅是树叶，因为ViewGroup也继承于它，弹出的窗口也使用了它等等。

![](http://github.com/callmexiaolu/callmexiaolu.github.io/raw/master/img/post-shared-1.png)

​																							图5

可以看出有许多类直接或者间接继承于它。

### 从根开始的事件分发机制

用户在触摸屏幕的时候会产生触摸事件，这个事件从根开始传递，逐级传递，传递的方法如下：

```java
public boolean dispatchTouchEvent(MotionEvent ev)
```

传递的过程中也可以进行拦截：

```java
public boolean onInterceptTouchEvent(MotionEvent ev) 
```

最终进行消费：

```java
public boolean onTouchEvent(MotionEvent event)
```

从根部发出的事件中途没有被拦截，那么会一直传达到顶部，如果顶部没有消费，那么该事件会原路返回，直到根部，最终会返回给根部消费。

由于view没有拦截事件的方法，因此继承了View的子类调用super.dispatchTouchEvent方法的时候就把该事件分发到View了，接着该View进行决定是否消费该事件，如果不进行消费，那么事件就往回传递，最终回到根部。这一个过程就像一个U形一样。![img](http://github.com/callmexiaolu/callmexiaolu.github.io/raw/master/img/post-shared-2.png)

​																								图6

### 从根开始对整颗树的测量

如上面图三所示，通过不断循环调用这几个方法，从最终的子view一路返回测量到根部进行汇总，进而完成测量，整个过程就可以参考下图。

### 思想

这种不断往底层搜索，直到拿到合适的结果返回，没有合适的结果便回到上一个节点继续进行搜索的方法被称为[深度优先搜索（DFS）]([https://baike.baidu.com/item/%E6%B7%B1%E5%BA%A6%E4%BC%98%E5%85%88%E6%90%9C%E7%B4%A2/5224976](https://baike.baidu.com/item/深度优先搜索/5224976))。

![](https://gss0.bdstatic.com/-4o3dSag_xI4khGkpoWK1HF6hhy/baike/c0%3Dbaike80%2C5%2C5%2C80%2C26/sign=d0a66a1c97dda144ce0464e0d3debbc7/0ff41bd5ad6eddc48dab7af439dbb6fd52663375.jpg)

​																							图7

常用的实现方法为递归实现和栈实现。在本文中Android的View实现的方法为循环调用实现，即measure() ——>onMeasure()——>measure() ——>onMeasure………….

由[上述代码](#枝干🌿（ViewGroup）)中的measureChildren方法可以知道，size(当前ViewGroup下的子View数量)变量影响了速度，而这个size就是这棵树中每一层的view的数量。但是影响速度的仅仅是size吗？还有层数。如果树只有一层那么，时间复杂度就是O(n) ，因为只遍历一次子View数量。如果有几层，那么就代表子ViewGroup下还有子ViewGroup，时间复杂度就变成了O(n*m)，这里的m是子ViewGroup的数量。所以我们在进行布局设计的时候能尽量减少层级就减少，因为这会带来性能上的提升，这对于事件的传递和试图的绘制都有帮助

