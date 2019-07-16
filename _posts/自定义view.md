## 自定义view

### 属性配置

values目录下attrs.xml文件进行view的属性自定义。

格式：

```xml
<resources>
    <declare-styleable name="CircleView">
        <attr name="titleText" format="string" />
        <attr name="subtitleText" format="string" />
        <attr name="titleSize" format="dimension" />
        <attr name="subtitleSize" format="dimension" />
        <attr name="strokeWidthSize" format="float" />
        <attr name="fillRadius" format="float" />
        <attr name="titleColor" format="color" />
        <attr name="subtitleColor" format="color" />
        <attr name="fillColor" format="color" />
        <attr name="strokeColorValue" format="color" />
        <attr name="backgroundColorValue" format="color" />
         <attr name="pointer_size" format="dimension" />
    </declare-styleable>
  
  <declare-styleable name="CircleImageView">
        <attr name="border_width" format="dimension" />
        <attr name="border_color" format="color" />
    </declare-styleable>
</resources>

```

### 代码编写

**编写之前应该想好如何实现，最佳思路。避免过程中途频繁更改代码，影响效率。**

#### 自定义ImageView

```ko
class CircleImageView : AppCompatImageView {

    private var mBorderColor: Int = Color.WHITE
    private var mBorderWidth: Int = 0

    private var mWidth = 0
    private var mRadius = 0f

    private val mBitmapPaint = Paint()
    private val mBorderPaint = Paint()
    private val mMatrix = Matrix()

    private lateinit var mCurrentBitmapShader: BitmapShader
    private lateinit var mBitmap: Bitmap
    private var mBitmapSize = 0
    private var mScale = 0f

    constructor(context: Context) : this(context, null)

    constructor(context: Context, attributes: AttributeSet?) : this(context, attributes, 0)

    constructor(context: Context, attributes: AttributeSet?, defStyle: Int) : super(context, attributes, defStyle) {
        val typedArray = context.obtainStyledAttributes(
                attributes,
                R.styleable.CircleImageView,
                0,
                0)
        mBorderColor = typedArray.getColor(R.styleable.CircleImageView_border_color, Color.WHITE)
        mBorderWidth = typedArray.getDimensionPixelSize(R.styleable.CircleImageView_border_width, 0)
        typedArray.recycle()
        mBitmapPaint.isAntiAlias = true
        initBorderPaint()
    }

    override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec)
        mWidth = min(measuredWidth, measuredHeight)
        mRadius = mWidth / 2f
        setMeasuredDimension(mWidth, mWidth) //让view宽高一样
        initBitmapShader()
    }

    override fun onDraw(canvas: Canvas) {
        if (!::mBitmap.isInitialized || !::mCurrentBitmapShader.isInitialized) {
            initBitmapShader()
        }
        canvas.drawCircle(mRadius, mRadius, mRadius - mBorderWidth, mBitmapPaint)
        canvas.drawCircle(mRadius, mRadius, mRadius - mBorderWidth / 2, mBorderPaint)
    }

    /**
     * drawable转bitmap
     */
    private fun getBitmap(drawable: Drawable): Bitmap {
        if (drawable is BitmapDrawable) {
            return drawable.bitmap
        }
        val w = drawable.intrinsicWidth
        val h = drawable.intrinsicHeight
        val bitmap = Bitmap.createBitmap(w, h, Bitmap.Config.ARGB_8888)
        drawable.setBounds(0, 0, w, h)
        drawable.draw(Canvas(bitmap))//创建画布
        return bitmap
    }

    private fun initBitmapShader() {
        drawable?.run {
            //判断上一次画的图片是否和当前图片相同
            if (!::mBitmap.isInitialized) {
                mBitmap = getBitmap(this)
                mCurrentBitmapShader = BitmapShader(getBitmap(this), Shader.TileMode.CLAMP, Shader.TileMode.CLAMP)
            }
            if (mBitmap !== getBitmap(this)) {
                mBitmap = getBitmap(this)
                mCurrentBitmapShader = BitmapShader(mBitmap, Shader.TileMode.CLAMP, Shader.TileMode.CLAMP)
            }
            //将bitmap作为着色器，意思是在指定地方绘制bitmap
            mBitmapSize = min(mBitmap.width, mBitmap.height)
            mScale = mWidth * 1.0f / mBitmapSize
            //矩阵变换，用于放大缩小
            mMatrix.setScale(mScale, mScale)
            //设置变换矩阵
            mCurrentBitmapShader.setLocalMatrix(mMatrix)
            //设置shader
            mBitmapPaint.shader = mCurrentBitmapShader
        }

    }

    private fun initBorderPaint() {
        mBorderPaint.reset()
        mBorderPaint.isAntiAlias = true
        mBorderPaint.color = mBorderColor
        mBorderPaint.strokeWidth = mBorderWidth.toFloat()
        mBorderPaint.style = Paint.Style.STROKE
    }

    fun setBorderColor(@ColorRes color: Int) {
        mBorderColor = ContextCompat.getColor(context, color)
        initBorderPaint()
        invalidate()
    }

}

```

##### 具体思路

构造方法中读取属性、measure后知道控件的大小，然后设置相关参数、draw；

**大忌：onDraw中不要进行对象的创建。因为onDraw方法会频繁调用，因此在这里创建对象会导致创建大量的对象**

代码中实现一个带边框的圆形imageView。

* 先把得到的图片转换为bitmat拉伸适应整个控件
* 创建bitmap的着色器，并设置画笔的着色器为该bitmapShader。
* 使用该画笔进行画圆。这时候就实现了圆形imageView。
* 创建一个画边框的画笔并设置相应参数，接着进行画边框。

（个人理解：着色器是把bitmap存进画笔里面，让画笔进行画bitmap出来。这里bitmap拉伸填满了整个控件。如果进行画圆，那就是在bitmap上截取一个圆，然后进行绘制）