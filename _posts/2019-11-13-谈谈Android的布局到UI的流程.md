## Android应用界面View是如何绘制的



布局中的xml文件，经过activity或者dialog的setContentView或者inflate方法设置后产生View

### 1.Activity的setContentView方法与LayoutInflate方法

#### 1.1 setContentView

**Activity中三个重载的setContentView方法：**

```java
public void setContentView(int layoutResID) {
        getWindow().setContentView(layoutResID);
        initWindowDecorActionBar();
    }

    public void setContentView(View view) {
        getWindow().setContentView(view);
        initWindowDecorActionBar();
    }

    public void setContentView(View view, ViewGroup.LayoutParams params) {
        getWindow().setContentView(view, params);
        initWindowDecorActionBar();
    }
```

这里调用了getWindow返回的Window抽象类的setContentView方法，调用了唯一实现类的PhoneWindow的setContentView方法。

**AppCompatActivity中三个重载的setContentView方法**：

```java
    @Override
    public void setContentView(@LayoutRes int layoutResID) {
        getDelegate().setContentView(layoutResID);
    }

    @Override
    public void setContentView(View view) {
        getDelegate().setContentView(view);
    }

    @Override
    public void setContentView(View view, ViewGroup.LayoutParams params) {
        getDelegate().setContentView(view, params);
    }
```

这里的setContentView方法调用了AppCompatDelegate抽象类的方法，这个抽象类的实现类是AppCompatDelegateImpl，那就直接看调用实现的方法：

```java
@Override
    public void setContentView(int resId) {
        ensureSubDecor(); //进行mSubDecor初始化
        //通过mSubDecor进行初始化contentParent
        ViewGroup contentParent = (ViewGroup) mSubDecor.findViewById(android.R.id.content);
      	//移除contentParent中所有视图view
        contentParent.removeAllViews();
        //把我们的布局id通过inflate方法实例化添加到contentParent
        LayoutInflater.from(mContext).inflate(resId, contentParent);
        mOriginalWindowCallback.onContentChanged();
    }
```

可以看到通过**LayoutInflater**执行**inflate()方法**把resId添加到**contentParent**之下，这个contentParent是一个ViewGroup，是一个有view Id的组件，通过mSubDecor进行赋值。mSubDecor初始化就是在该方法第一行中，调用ensureSubDecor()方法。

```java
private void ensureSubDecor() {
        if (!mSubDecorInstalled) {
            //初始化mSubDecor
            mSubDecor = createSubDecor();
            ............
        }
    }

 private ViewGroup createSubDecor() {
       //获取AppCompatActivity的主题属性
        TypedArray a = this.mContext.obtainStyledAttributes(styleable.AppCompatTheme);
       //设置属性
        if (!a.hasValue(styleable.AppCompatTheme_windowActionBar)) {  
        ....
        }
        ......
        if (!mWindowNoTitle) { //根据是否有title进行不同实例化
            if (mIsFloating) {
              //如果是floating，就inflate弹窗带title样式的view
                subDecor = (ViewGroup) inflater.inflate(
                        R.layout.abc_dialog_title_material, null);
                mHasActionBar = mOverlayActionBar = false;
            } else if (mHasActionBar) {
                ....
               
                subDecor = (ViewGroup) LayoutInflater.from(themedContext)
                        .inflate(R.layout.abc_screen_toolbar, null);

                mDecorContentParent = (DecorContentParent) subDecor
                        .findViewById(R.id.decor_content_parent);
                mDecorContentParent.setWindowCallback(getWindowCallback());
                ....
            }
        } else {
            if (mOverlayActionMode) {
                subDecor = (ViewGroup) inflater.inflate(
                        R.layout.abc_screen_simple_overlay_action_mode, null);
            } else {
                subDecor = (ViewGroup) inflater.inflate(R.layout.abc_screen_simple, null);
            }
            ....
        }
       ....
         //开始进行contentView的一些设置
         final ViewGroup windowContentView = (ViewGroup) mWindow.findViewById(android.R.id.content);
        if (windowContentView != null) {
           //通过content获取控件，递归获取其所有view，添加到contentView中
            while (windowContentView.getChildCount() > 0) {
                final View child = windowContentView.getChildAt(0);
                windowContentView.removeViewAt(0);
                contentView.addView(child);
            }
            //设置id
            windowContentView.setId(View.NO_ID);
            contentView.setId(android.R.id.content);
            
            if (windowContentView instanceof FrameLayout) {
                ((FrameLayout) windowContentView).setForeground(null);
            }
        }
        // Now set the Window's content view with the decor
        mWindow.setContentView(subDecor);
        ....
        return subDecor;
}
```

在createSubDecor方法中，先是获取该Activity的属性(有无标题栏，标题栏属性)进行设置，接着根据是否有标题栏等条件inflate不同的布局实例化赋值给subDecor，接着获取xml布局中的view添加到contentView中，然后通过window的setContentView方法设置subDecor。这个mWindow是抽象类Window的实现类PhoneWindow类。

###### PhoneWindow的setContentView方法：

```java
@Override
    public void setContentView(int layoutResID) {
        if (mContentParent == null) {
            installDecor();//初始化DecorView
        } else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            mContentParent.removeAllViews();
        }

        if (hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            final Scene newScene = Scene.getSceneForLayout(mContentParent, layoutResID,
                    getContext());
            transitionTo(newScene);
        } else {
            mLayoutInflater.inflate(layoutResID, mContentParent);//加载我们给的布局id，实例化后添加到mContentParent中
        }
        mContentParent.requestApplyInsets();
        final Callback cb = getCallback();
        if (cb != null && !isDestroyed()) {
            cb.onContentChanged();
        }
        mContentParentExplicitlySet = true;
    }


private void installDecor() {
        mForceDecorInstall = false;
        if (mDecor == null) {
            mDecor = generateDecor(-1);
           ......
        } else {
            mDecor.setWindow(this);
        }
        if (mContentParent == null) {
            mContentParent = generateLayout(mDecor);//这个方法很长很长，它返回一个ViewGroup
          //该方法获取window属性，设置判断是否沾满屏幕，判断ActionBar，标题，设置flag。
          //根据features选择加载哪种根布局,最终加载到contentParent
          .......
        }
}

protected DecorView generateDecor(int featureId) {
       .......
       //这里初始化DecorView
        return new DecorView(context, featureId, this, getAttributes());
    }
```

Activity初始化的时候布局文件没有加载，根布局mContentParent为null，因此会执行installDecor()方法，并且界面未展示，mDecor也为null（因为mDecor为DecorView)，看下图便知道之间的关系，因此执行了方法generateDecor(-1)，接着执行generateLayout(mDecor)方法，返回一个根布局ViewGroup。执行完installDecor()方法后，接着执行inflate，实例化布局。

![img](http://github.com/callmexiaolu/callmexiaolu.github.io/raw/master/img/phoneWindow.png)



**总结：**两个Activity加载方法最后都是利用PhoneWindow来进行DecorView的初始化，接着再加载根布局，根布局contentParent中加载我们传进去的布局id，这样子就完成了加载。

如果关系抽象化可以这样子比较：Window是一块电子屏(抽象类)，PhoneWindow是一块手机电子屏(继承于Window)，DecorView就是电子屏要显示的内容，Activity就是手机电子屏安装位置。

#### 1.2 LayoutInflate

我们上面分析的Activity和AppcompatActivity的setContentView()方法中都有用到LayoutInflate的inflate()方法。

```java
public View inflate(@LayoutRes int resource, @Nullable ViewGroup root, boolean attachToRoot) {
        final Resources res = getContext().getResources();
        if (DEBUG) {
            Log.d(TAG, "INFLATING from resource: \"" + res.getResourceName(resource) + "\" ("
                    + Integer.toHexString(resource) + ")");
        }

        final XmlResourceParser parser = res.getLayout(resource);
        try {
            return inflate(parser, root, attachToRoot);
        } finally {
            parser.close();
        }
    }
```

源码中使用了XmlResourceParser来解析XML布局文件，有兴趣的可以去了解一下，接着继续调用inflate方法。

```java
public View inflate(XmlPullParser parser, @Nullable ViewGroup root, boolean attachToRoot) {
        synchronized (mConstructorArgs) {
            Trace.traceBegin(Trace.TRACE_TAG_VIEW, "inflate");

            final Context inflaterContext = mContext;
            //拿到xml各个标签属性
            final AttributeSet attrs = Xml.asAttributeSet(parser);
            Context lastContext = (Context) mConstructorArgs[0];
            mConstructorArgs[0] = inflaterContext;
            View result = root;

            try {
                //寻找开始标签
                int type;
                while ((type = parser.next()) != XmlPullParser.START_TAG &&
                        type != XmlPullParser.END_DOCUMENT) {
                   
                }

                if (type != XmlPullParser.START_TAG) {
                    throw new InflateException(parser.getPositionDescription()
                            + ": No start tag found!");
                }

                final String name = parser.getName();
                ........
                if (TAG_MERGE.equals(name)) {
                    if (root == null || !attachToRoot) {
                        throw new InflateException("<merge /> can be used only with a valid "
                                + "ViewGroup root and attachToRoot=true");
                    }

                    rInflate(parser, root, inflaterContext, attrs, false);
                } else {
                    //传进节点和标签名进行
                   //该方法内调用createView()方法，利用反射创建view返回
                    final View temp = createViewFromTag(root, name, inflaterContext, attrs);

                    ViewGroup.LayoutParams params = null;

                    if (root != null) {
                        	....
                        // Create layout params that match root, if supplied
                        params = root.generateLayoutParams(attrs);
                        if (!attachToRoot) {
                            // Set the layout params for temp if we are not
                            // attaching. (If we are, we use addView, below)
                            temp.setLayoutParams(params);
                        }
                    }

                    // Inflate all children under temp against its context.
                    // 实例化所有处于根节点下的子View
                    rInflateChildren(parser, temp, attrs, true);

                    //参数root不空，就把实例化的布局view添加到root上
                    if (root != null && attachToRoot) {
                        root.addView(temp, params);
                    }

                    //root为空就直接返回实例化的布局
                    if (root == null || !attachToRoot) {
                        result = temp;
                    }
                }

            } catch (XmlPullParserException e) {
                final InflateException ie = new InflateException(e.getMessage(), e);
                ie.setStackTrace(EMPTY_STACK_TRACE);
                throw ie;
            } catch (Exception e) {
                final InflateException ie = new InflateException(parser.getPositionDescription()
                        + ": " + e.getMessage(), e);
                ie.setStackTrace(EMPTY_STACK_TRACE);
                throw ie;
            } finally {
                // Don't retain static reference on context.
                mConstructorArgs[0] = lastContext;
                mConstructorArgs[1] = null;

                Trace.traceEnd(Trace.TRACE_TAG_VIEW);
            }

            return result;
        }
    }
```

可以看出inflate方法中先是从布局中解析拿到标签属性，寻找开始标签，然后循环利用反射解析子View，最终添加到根View。后续根据传进来的参数root决定是否指定父布局。

1. 如果root为null，attachToRoot将失去作用，设置任何值都没有意义。

2. 如果root不为null，attachToRoot设为true，则会给加载的布局文件的指定一个父布局，即root。

3. 如果root不为null，attachToRoot设为false，则会将布局文件最外层的所有layout属性进行设置，当该view被添加到父view当中时，这些layout属性会自动生效。

4. 在不设置attachToRoot参数的情况下，如果root不为null，attachToRoot参数默认为true。

### 2.界面中View的绘制

那么得到了View，该如何显示到界面上呢？

这时候需要了解到View的绘制了，View的绘制流程都会经历三个主要的流程，measure、layout、draw。

##### 首先要进行Measure：

```java
public final void measure(int widthMeasureSpec, int heightMeasureSpec) {
        ......
        //回调onMeasure()方法
        onMeasure(widthMeasureSpec, heightMeasureSpec);
        ......
    }

//measure的主要目的就是对View树中的每个View的mMeasuredWidth和mMeasuredHeight进行赋值，
//所以一旦这两个变量被赋值意味着该View的测量工作结束

protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
  
        setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
                getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
    }

//可以看见setMeasuredDimension传入的参数都是通过getDefaultSize返回的

//此外在ViewGroup中还有measureChildren（）方法测量子View的规格，实际上这是一个递归的过程
 protected void measureChildren(int widthMeasureSpec, int heightMeasureSpec) {
        final int size = mChildrenCount;
        final View[] children = mChildren;
        for (int i = 0; i < size; ++i) {
            final View child = children[i];
            if ((child.mViewFlags & VISIBILITY_MASK) != GONE) {
                //调用方法测量子View
                measureChild(child, widthMeasureSpec, heightMeasureSpec);
            }
        }
    }

//ViewGroup还提供了子View的另外测量方法
//measureChild()  measureChildWithMargins（）
```

为整个View树计算实际的大小，然后设置实际的高和宽，每个View控件的实际宽高都是由父视图和自身决定的。实际的测量是在onMeasure方法进行，所以在View的子类需要重写onMeasure方法，这是因为measure方法是final的，不允许重载，所以View子类只能通过重载onMeasure来实现自己的测量逻辑。

##### layout

##### draw（绘制）

```java
//方法位于View.java
public void draw(Canvas canvas) {
       ......
        /*
         * Draw traversal performs several drawing steps which must be executed
         * in the appropriate order:
         *
         *      1. Draw the background
         *      2. If necessary, save the canvas' layers to prepare for fading
         *      3. Draw view's content
         *      4. Draw children
         *      5. If necessary, draw the fading edges and restore layers
         *      6. Draw decorations (scrollbars for instance)
         */

        // Step 1, draw the background, if needed
        int saveCount;

        if (!dirtyOpaque) {
            drawBackground(canvas);
        }

        // skip step 2 & 5 if possible (common case)
        final int viewFlags = mViewFlags;
        boolean horizontalEdges = (viewFlags & FADING_EDGE_HORIZONTAL) != 0;
        boolean verticalEdges = (viewFlags & FADING_EDGE_VERTICAL) != 0;
        if (!verticalEdges && !horizontalEdges) {
            // Step 3, draw the content
            if (!dirtyOpaque) onDraw(canvas);

            // Step 4, draw the children
            dispatchDraw(canvas);

            drawAutofilledHighlight(canvas);

            // Overlay is part of the content and draws beneath Foreground
            if (mOverlay != null && !mOverlay.isEmpty()) {
                mOverlay.getOverlayView().dispatchDraw(canvas);
            }

            // Step 6, draw decorations (foreground, scrollbars)
            onDrawForeground(canvas);

            // Step 7, draw the default focus highlight
            drawDefaultFocusHighlight(canvas);

            if (debugDraw()) {
                debugDrawFocus(canvas);
            }

            // we're done...
            return;
        }
      	.......
        // Step 2, save the canvas' layers
        int paddingLeft = mPaddingLeft;

        final boolean offsetRequired = isPaddingOffsetRequired();
        if (offsetRequired) {
            paddingLeft += getLeftPaddingOffset();
        }

        int left = mScrollX + paddingLeft;
        int right = left + mRight - mLeft - mPaddingRight - paddingLeft;
        int top = mScrollY + getFadeTop(offsetRequired);
        int bottom = top + getFadeHeight(offsetRequired);

        if (offsetRequired) {
            right += getRightPaddingOffset();
            bottom += getBottomPaddingOffset();
        }

        final ScrollabilityCache scrollabilityCache = mScrollCache;
        final float fadeHeight = scrollabilityCache.fadingEdgeLength;
        int length = (int) fadeHeight;

        // clip the fade length if top and bottom fades overlap
        // overlapping fades produce odd-looking artifacts
        if (verticalEdges && (top + length > bottom - length)) {
            length = (bottom - top) / 2;
        }

        // also clip horizontal fades if necessary
        if (horizontalEdges && (left + length > right - length)) {
            length = (right - left) / 2;
        }
        .....

        saveCount = canvas.getSaveCount();

        int solidColor = getSolidColor();
        if (solidColor == 0) {
            if (drawTop) {
                canvas.saveUnclippedLayer(left, top, right, top + length);
            }

            if (drawBottom) {
                canvas.saveUnclippedLayer(left, bottom - length, right, bottom);
            }

            if (drawLeft) {
                canvas.saveUnclippedLayer(left, top, left + length, bottom);
            }

            if (drawRight) {
                canvas.saveUnclippedLayer(right - length, top, right, bottom);
            }
        } else {
            scrollabilityCache.setFadeColor(solidColor);
        }

        // Step 3, draw the content
        //第三步，绘制内容
        if (!dirtyOpaque) onDraw(canvas);

        // Step 4, draw the children
        //第四步，绘制孩子
        dispatchDraw(canvas);

        // Step 5, draw the fade effect and restore layers
        final Paint p = scrollabilityCache.paint;
        final Matrix matrix = scrollabilityCache.matrix;
        final Shader fade = scrollabilityCache.shader;

        if (drawTop) {
            matrix.setScale(1, fadeHeight * topFadeStrength);
            matrix.postTranslate(left, top);
            fade.setLocalMatrix(matrix);
            p.setShader(fade);
            canvas.drawRect(left, top, right, top + length, p);
        }

        if (drawBottom) {
            matrix.setScale(1, fadeHeight * bottomFadeStrength);
            matrix.postRotate(180);
            matrix.postTranslate(left, bottom);
            fade.setLocalMatrix(matrix);
            p.setShader(fade);
            canvas.drawRect(left, bottom - length, right, bottom, p);
        }

        if (drawLeft) {
            matrix.setScale(1, fadeHeight * leftFadeStrength);
            matrix.postRotate(-90);
            matrix.postTranslate(left, top);
            fade.setLocalMatrix(matrix);
            p.setShader(fade);
            canvas.drawRect(left, top, left + length, bottom, p);
        }

        if (drawRight) {
            matrix.setScale(1, fadeHeight * rightFadeStrength);
            matrix.postRotate(90);
            matrix.postTranslate(right, top);
            fade.setLocalMatrix(matrix);
            p.setShader(fade);
            canvas.drawRect(right - length, top, right, bottom, p);
        }

        canvas.restoreToCount(saveCount);

        drawAutofilledHighlight(canvas);

        // Overlay is part of the content and draws beneath Foreground
        if (mOverlay != null && !mOverlay.isEmpty()) {
            mOverlay.getOverlayView().dispatchDraw(canvas);
        }

        // Step 6, draw decorations (foreground, scrollbars)
        //第六步，绘制装饰物(前景，滑动条)
        onDrawForeground(canvas);

        if (debugDraw()) {
            debugDrawFocus(canvas);
        }
    }
```



* 1.对View的背景进行绘制

  ```java
   private void drawBackground(Canvas canvas) {
          //获取xml中设置了background属性的文件
          final Drawable background = mBackground;
          if (background == null) {
              return;
          }
          //设置背景在控件中的范围边界，方法里面的边界是根据layout过程中确定的位置
          // mBackground.setBounds(0, 0, mRight - mLeft, mBottom - mTop);
          setBackgroundBounds();
          .......
          final int scrollX = mScrollX;
          final int scrollY = mScrollY;
          if ((scrollX | scrollY) == 0) {
            //调用Drawable的draw()方法来完成背景的绘制工作
              background.draw(canvas);
          } else {
              canvas.translate(scrollX, scrollY);
            //调用Drawable的draw()方法来完成背景的绘制工作
              background.draw(canvas);
              canvas.translate(-scrollX, -scrollY);
          }
      }
  ```

  

* 3.对View的内容进行绘制

  ```java
  /**
       * Implement this to do your drawing.
       *
       * @param canvas the canvas on which the background will be drawn
       */
      protected void onDraw(Canvas canvas) {
      }
  ```

  方法没有在view中实现，而是在继承了view的各类子view中实现具体逻辑，因为每个view控件的展示各不相同。

* 4.对当前View的所有子View进行绘制

  ```java
  /**
       * Called by draw to draw the child views. This may be overridden
       * by derived classes to gain control just before its children are drawn
       * (but after its own view has been drawn).
       * @param canvas the canvas on which to draw the view
       */
      protected void dispatchDraw(Canvas canvas) {
  
      }
  ```

  方法没有实现，注释表明了如果View包含子类需要重写该方法，ViewGroup中就重写了该方法。

* 6.对View的前景，滚动条进行绘制

  TextView中可设置滚动条，如果没设置就默认不进行绘制了。

##### draw原理总结

可以看见，绘制过程就是把View对象绘制到屏幕上，整个draw过程需要注意如下细节：

- 如果该View是一个ViewGroup，则需要递归绘制其所包含的所有子View。
- View默认不会绘制任何内容，真正的绘制都需要自己在子类中实现。
- View的绘制是借助onDraw方法传入的Canvas类来进行的。
- View是先绘制背景最后才绘制前景，因此一些设置前景显示的view会导致背景失效(ImageView)
- 在获取画布剪切区（每个View的draw中传入的Canvas）时会自动处理掉padding，子View获取Canvas不用关注这些逻辑，只用关心如何绘制即可。
- 默认情况下子View的ViewGroup.drawChild绘制顺序和子View被添加的顺序一致，但是你也可以重载ViewGroup.getChildDrawingOrder()方法提供不同顺序。



那么如何调用并走到draw这个流程的呢。回到PhoneWindow中

```java
//PhoneWindow.java
@Override
    public void addContentView(View view, ViewGroup.LayoutParams params) {
        if (mContentParent == null) {
            installDecor();
        }
        if (hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            // TODO Augment the scenes/transitions API to support this.
            Log.v(TAG, "addContentView does not support content transitions");
        }
       //这里开始添加之前从xml文件中解析出来的View，添加进主布局内容中
        mContentParent.addView(view, params);
        mContentParent.requestApplyInsets();
        final Callback cb = getCallback();
        if (cb != null && !isDestroyed()) {
            cb.onContentChanged();
        }
    }

//最后调用到该方法，方法位于ViewGroup.java中
public void addView(View child, int index, LayoutParams params) {
        .....
        //这里调用requestLayout是开始进行Layout方法调用的开始
        requestLayout();
        //调用invalidate方法开始进行绘制
        invalidate(true);
        addViewInner(child, index, params, false);
    }


    public void requestLayout() {
        //如果有之前的测量缓存，那就清理掉
        if (mMeasureCache != null) mMeasureCache.clear();

        if (mAttachInfo != null && mAttachInfo.mViewRequestingLayout == null) {
            //得到View Tree的Root
            ViewRootImpl viewRoot = getViewRootImpl();
            if (viewRoot != null && viewRoot.isInLayout()) {
                if (!viewRoot.requestLayoutDuringLayout(this)) {
                    return;
                }
            }
            mAttachInfo.mViewRequestingLayout = this;
        }

        mPrivateFlags |= PFLAG_FORCE_LAYOUT;
        mPrivateFlags |= PFLAG_INVALIDATED;

        if (mParent != null && !mParent.isLayoutRequested()) {
            //调用mParent的方法，自上向下进行layout
            mParent.requestLayout();
        }
        if (mAttachInfo != null && mAttachInfo.mViewRequestingLayout == this) {
            mAttachInfo.mViewRequestingLayout = null;
        }
    }

//ViewRootImpl.java
 public void dispatchInvalidateDelayed(View view, long delayMilliseconds) {
        Message msg = mHandler.obtainMessage(MSG_INVALIDATE, view);
        //使用handler发送一条消息
        mHandler.sendMessageDelayed(msg, delayMilliseconds);
    }

  @Override
        public void handleMessage(Message msg) {
            switch (msg.what) {
                //同样在ViewRootImpl中接受消息，进行调用View的invalidate方法
                case MSG_INVALIDATE:
                    ((View) msg.obj).invalidate();
                    break;
                .......
            }
     }

```

经过requestLayout方法后接着进行invalidate方法，requestLayout方法先是从根开始进行层层向下进行layout后接着进行绘制。invalidate方法则是向上请求，到viewParent接口的实现类ViewRootImpl中再进行绘制调用。

![](https://img-blog.csdn.net/20150531111928069)



关于上段代码中的测量缓存数据清理，是因为我们在改变一个布局的时候会重新resLayout一次，这时候会重新走一遍流程。另外我们在写自定义View的时候会经常用到invalidate方法，因为该方法能够重新绘制。

- 直接调用invalidate方法.请求重新draw，但只会绘制调用者本身。

- 触发setVisibility方法。 当View可视状态在INVISIBLE转换VISIBLE时会间接调用invalidate方法，继而绘制该View。当View的可视状态在INVISIBLE\VISIBLE 转换为GONE状态时会间接调用requestLayout和invalidate方法，同时由于View树大小发生了变化，所以会请求measure过程以及draw过程，同样只绘制需要“重新绘制”的视图。

- 触发setEnabled方法。请求重新draw，但不会重新绘制任何View包括该调用者本身。

  

  

  

  

  

  

