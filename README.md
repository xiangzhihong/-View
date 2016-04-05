上次去一个公司面试，面试官问了一个题，怎么用android的自定义view实现一个公章的效果，据说这是华为之前的面试题，我想了下，要是公章的效果，最外层是一个圆，里面是一个五角星，但是这文字怎么画呢，比较难搞，后来回来看了下java的api，发现人家的Path里面本来就提供了这么一个方法：

public void addArc(RectF oval, float startAngle, float sweepAngle) {
    addArc(oval.left, oval.top, oval.right, oval.bottom, startAngle, sweepAngle);
}
然后人家解释说了，根据狐线的角度生成相应的路径，所以我们就可以给文字设置一个相应绘制区域，使其绘制的文字都在这个区域内，
path.addArc(oval,-(firstrad-textPadding*i/2), textPadding);
接下来我们只需要在这个区域内把文字绘制上去就行了。
好的，下面是全部代码：

首先继承自View，我们在构造里面初始化，同样为了方便程序的扩展性，我们用自定义属性，
<declare-styleable name="Seal">
    <attr name="scale_text_size" format="dimension" />
    <attr name="scale_text_color" format="color" />
    <attr name="scale_text" format="string" />
    <attr name="scale_text_padding" format="float" />
    <attr name="circle_stroke_width" format="dimension" />
    <attr name="circle_color" format="color" />
    <attr name="circle_radius" format="dimension" />
</declare-styleable>

然后我们初始化的时候主要初始化文字，文字大小，文字间距，文字颜色等等，
private void initViews(AttributeSet attrs, int defStyle) {
    TypedArray typedArray = getContext().obtainStyledAttributes(attrs, R.styleable.Seal, defStyle, 0);
    circleText = typedArray.getString(R.styleable.Seal_scale_text);
    textSize = typedArray.getDimension(R.styleable.Seal_scale_text_size, 20);
    scaleTextColor = typedArray.getColor(R.styleable.Seal_scale_text_color, getResources().getColor(R.color.c9));
    textPadding=typedArray.getFloat(R.styleable.Seal_scale_text_padding,50);
    circleStrokeWidth = typedArray.getDimensionPixelSize(R.styleable.Seal_circle_stroke_width, 3);
    circleColor = typedArray.getColor(R.styleable.Seal_circle_color, getResources().getColor(R.color.c9));
    circleRadius = typedArray.getDimensionPixelSize(R.styleable.Seal_circle_radius, 7);
    typedArray.recycle();
}
接下来我们在重写Ondraww（Canvas canvas）
@Override
protected void onDraw(Canvas rootCanvas) {
    super.onDraw(rootCanvas);
    Bitmap image = Bitmap.createBitmap(getWidth(), getHeight(), Bitmap.Config.ARGB_8888);
    Canvas canvas = new Canvas(image);
    Paint paint=new Paint();

    drawRing(canvas,paint);
    drawStar(canvas);
    drawText(canvas);
    rootCanvas.drawBitmap(image, 0, 0, null);
}

接下来是对应的三个方法：画圆环（ring）,五角星（star）,文字（text）
//圆环
private void drawRing(Canvas canvas, Paint paint) {
    centre = canvas.getWidth() / 2; // 获取圆心的x坐标
    radius = (int) (centre - circleStrokeWidth / 2); // 圆环的半径
    paint.setColor(Color.RED); // 设置圆环的颜色
    paint.setStyle(Paint.Style.STROKE); // 设置空心
    paint.setStrokeWidth(circleStrokeWidth); // 设置圆环的宽度
    paint.setAntiAlias(true); // 消除锯齿
    canvas.drawCircle(centre, centre, radius, paint); // 画出圆环
}
//绘制五角星
private void drawStar(Canvas canvas){
    float start_radius = (float) ((radius / 2)*1.1);
    int x = centre, y = centre;
    float x1,y1,x2,y2,x3,y3,x4,y4,x5,y5;
    float r72 = (float) Math.toRadians(72);
    float r36 = (float) Math.toRadians(36);
    //顶点
    x1 = x;
    y1 = y - start_radius;
    //左1
    x2 = (float) (x - start_radius*Math.sin(r72));
    y2 = (float) (y - start_radius*Math.cos(r72));
    //右1
    x3 = (float) (x + start_radius*Math.sin(r72));
    y3 = (float) (y - start_radius*Math.cos(r72));
    //左2
    x4 = (float) (x - start_radius*Math.sin(r36));
    y4 = (float) (y + start_radius*Math.cos(r36));
    //右2
    x5 = (float) (x + start_radius*Math.sin(r36));
    y5 = (float) (y + start_radius*Math.cos(r36));

    //连接各个节点，绘制五角星
    Path path = new Path();
    path.moveTo(x1, y1);
    path.lineTo(x5, y5);
    path.lineTo(x2, y2);
    path.lineTo(x3, y3);
    path.lineTo(x4, y4);
    path.close();

    Paint paint = new Paint();
    paint.setColor(Color.RED);

    canvas.drawPath(path, paint);
}
private void drawText(Canvas canvas){
    Paint paint = new Paint();
    paint.setColor(Color.RED);
    paint.setTypeface(Typeface.DEFAULT_BOLD);
    paint.setTextAlign(Paint.Align.CENTER);
    paint.setTextSize(radius/5+5);
    //圆弧文字所在矩形范围
    RectF oval=new RectF(0, 0, 2*radius, (float) (2*radius));
    //第一个文字偏移角度，其中padding/2为文字间距
    float firstrad = 90 + textPadding * (circleText.length()) / 4 - textPadding/8;
    for(int i = 0; i < circleText.length(); i++){
        Path path = new Path();
        //根据角度生成弧线路径
        path.addArc(oval,-(firstrad-textPadding*i/2), textPadding);
        canvas.drawTextOnPath(String.valueOf(circleText.charAt(i)), path, -(float) (radius/3),(float) (radius/3), paint);
    }
}
最后在我们需要的视图中引用下就好了
<com.xzh.sealmaster.view.SealView
    android:layout_width="200dp"
    android:layout_height="200dp"
    android:layout_gravity="center"
    app:scale_text_size="16sp"
    app:scale_text_padding="50"
    app:scale_text="华为上海有限公司"
    />
有需要源码的，请到下面地址下载：http://download.csdn.net/detail/xiangzhihong8/9479372
