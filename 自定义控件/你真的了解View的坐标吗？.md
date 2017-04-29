## 闲聊

View，对我们来说在熟悉不过了，从接触Android开始，我们就一直在接触View，界面当中到处都是 View，比如我们经常用到的TextView，Button，LinearLayout等等，但是我们真的了解View吗？尤其是View的坐标。mLeft,mRight,mY,mX,mTranslationY,mScoollY,相对于屏幕的坐标等等这些概念你真的清楚了吗？如果真的清楚了，那你没有必要度这篇博客，如果你还是有一些模糊，建议花上几分钟的时间读一下，这篇博客较短，花个几分钟的时间就可以阅读完。

为什么要写这一篇博客呢？

因为掌握View的坐标很重要，尤其是对于自定义View，学习动画有重大的意义。

这篇博客主要讲解一下问题

- View 的 getLeft()和get Right()和 getTop() 和getBottom()
- View 的 getＹ()， getTranslationY() 和 getTop() 之间的联系
- View 的 getScroolY 和 View 的 scrollTo() 和 scrollBy()
- event.getY 和 event.getRawY()
- 扩展，怎样获取状态栏（StatusBar）和标题栏（titleBar）的高度

## 基本概念

![img](http://upload-images.jianshu.io/upload_images/2050203-0f00e9fa06081e32.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

简单说明一下（上图Activity采用默认Style，状态栏和标题栏都会显示）：最大的草绿色区域是屏幕界面，红色次大区域我们称之为“应用界面区域”，最小紫色的区域我们称之为“View绘制区域”；屏幕顶端、应用界面区之外的那部分显示手机电池网络运营商信息的为“状态栏”，应用区域顶端、View绘制区外部显示Activity名称的部分我们称为“标题栏”。

从这张图片我们可以看到
在Android中，当ActionBar存在的情况下，

```
屏幕的 高度=状态栏+应用区域的高度=状态栏的 高度+（标题栏的 高度+View 绘制区域的高度）
```

当ActionBar不存在的情况下

```
屏幕的高度=状态栏+应用区域的高度=状态栏的 高度+（View 绘制区域的 高度）
```

## View 的 getLeft()和getRight()和 getTop() 和getBottom()

```java
View.getLeft() ;
View.getTop() ;
View.getBottom();
View.getRight() ;
```

top是左上角纵坐标，left是左上角横坐标，right是右下角横坐标，bottom是右下角纵坐标,都是相对于它的**直接父View**而言的，而不是相对于**屏幕**而言的。这一点要区分清楚。那那个坐标是相对于屏幕而言的呢，以及要怎样获取相对于屏幕的坐标呢？

目前View里面的变量还没有一个是相对于屏幕而言的，但是我们可以获取到相对于屏幕的坐标。一般来说，我们要获取View的坐标和高度 等，都必须等到View绘制完毕以后才能获取的到，在Activity 的 onCreate()方法 里面 是获取不到的，必须 等到View绘制完毕以后才能获取地到View的响应的坐标，一般来说，主要 有以下两种方法。

第一种方法，onWindowFocusChanged()方法里面进行调用

```java
      @Override
    public void onWindowFocusChanged(boolean hasFocus) {
     super.onWindowFocusChanged(hasFocus); 
     //确保只会调用一次
      if(first){
        first=false;
        final int[] location = new int[2];     
        mView.getLocationOnScreen(location);
        int x1 = location[0]  ;
        int y1 = location[1]  ;
        Log.i(TAG, "onCreate: x1=" +x1);
        Log.i(TAG, "onCreate: y1=" +y1);
      }
   }
```

第二种方法，在视图树绘制完成的时候进行测量

```java
        mView.getViewTreeObserver().addOnGlobalLayoutListener(new ViewTreeObserver
                .OnGlobalLayoutListener() {

            @Override
            public void onGlobalLayout() {
                //   移除监听器，确保只会调用一次，否则在视图树发挥改变的时候又会调用
                mView.getViewTreeObserver().removeGlobalOnLayoutListener(this);
                final int[] location = new int[2];
                mView.getLocationOnScreen(location);
                int x1 = location[0];
                int y1 = location[1];
                Log.i(TAG, "onCreate: x1=" + x1);
                Log.i(TAG, "onCreate: y1=" + y1);
            }
        });
```

## View 的 getＹ()， getTranslationY() 和 getTop() 之间的联

getＹ()

> Added in API level 14
> The visual y position of this view, in pixels.(返回的是View视觉上的图标，即我们眼睛看到位置的Y坐标，默认值跟getTop()相同，别急，下面会解释）

getTranslationY()

> Added in API level 14
> The vertical position of this view relative to its top position, in pixels.(竖直方向上相对于top的偏移量，默认值为0）

那 getY() 和 getTranslationY() 和 getTop () 到底有什么关系呢？

```java
@ViewDebug.ExportedProperty(category = "drawing")
public float getY() {
   return mTop + getTranslationY();
}

    @ViewDebug.ExportedProperty(category = "drawing")
    public float getTranslationY() {
        return mRenderNode.getTranslationY();
    }
    @ViewDebug.CapturedViewProperty
    public final int getTop() {
        return mTop;
    }
```

从以上的源码我们可以知道 getY()= getTranslationY()+ getTop ()，而 getTranslationY() 的默认值是0，除非我们通过 setTranlationY() 来改变它，这也就是我们上面上到的 getY 默认值跟 getTop()相同

那我们要怎样改变 top值 和 Y 值呢？ 很明显就是调用相应的set方法 ，即 setY() 和setTop() ，就可以改变他们 的值。

## View 的 getScroolY 和 View 的 scrollTo() 和 scrollBy()

getScrollY是一个比较特别的函数，因为它涉及一个值叫mScrollY，简单说，getScrollY一般得到的都是0，除非你调用过scrollTo或scrollBy这两个函数来改变它。

### scrollTo() 和 scrollBy()

从字面意思我们可以知道 scrollTo() 是滑动到哪里的意思 ，scrollBy()是相对当前的位置滑动了多少。当然这一点在源码中也是可以体现出来的

```java
public void scrollTo(int x, int y) {
    if (mScrollX != x || mScrollY != y) {
        int oldX = mScrollX;
        int oldY = mScrollY;
        mScrollX = x;
        mScrollY = y;
        invalidateParentCaches();
        onScrollChanged(mScrollX, mScrollY, oldX, oldY);
        if (!awakenScrollBars()) {
            postInvalidateOnAnimation();
        }
    }
}
public void scrollBy(int x, int y) {
    scrollTo(mScrollX + x, mScrollY + y);
}
```

有几点需要注意的是

- 不论是scrollTo或scrollBy，其实都是对View的内容进行滚动而不是对View本身，你可以做个小实验，一个LinearLayouy背景是黄色，里面放置一个子LinearLayout背景是蓝色，调用scrollTo或scrollBy，移动的永远是蓝色的子LinearLayout。
- 还有就是scrollTo和scrollBy函数的参数和坐标系是“相反的”，比如scrollTo(-100,0)，View的内容是向X轴正方向移动的，这个相反打引号是因为并不是真正的相反，具体可以看源码，关于这两个函数的源码分析大家可以看[Android——源码角度分析View的scrollBy()和scrollTo()的参数正负问题](http://blog.csdn.net/xplee0576/article/details/24242383?utm_source=tuicool&utm_medium=referral)，一目了然。

## View 的 width 和 height

```java
@ViewDebug.ExportedProperty(category = "layout")
public final int getHeight() {
    return mBottom - mTop;
}
```

我们可以看到 Android的 height 是由 mBottom 和 mTop 共同得出的，那我们要怎样设置Android的高度呢？有人会说直接在xml里面设置 android:height="" 不就OK了，那我们如果要动态设置height的高度呢，怎么办？你可能会想到 setWidth()方法？但是我们找遍了View的所有方法，都没有发现 setWidth()方法，那要怎样动态设置height呢？其实有两种方法

```java
 int width=50;
int height=100;
ViewGroup.LayoutParams layoutParams = view.getLayoutParams();
if(layoutParams==null){
    layoutParams=new ViewGroup.LayoutParams(width,height);
}else{
    layoutParams.height=height;
}
view.setLayoutParams(layoutParams);
```

第二种方法，单独地改变top或者bottom的值，这种方法不推荐使用

至于width，它跟height基本一样，只不过它是有mRight 和mLeft 共同决定而已。

需要注意的是，平时我们在执行动画的过程，不推荐使用LayoutParams来改变View的状态，因为改变LayoutParams会调用requestLayout()方法，会标记当前View及父容器，同时逐层向上提交，直到ViewRootImpl处理该事件，ViewRootImpl会调用三大流程，从measure开始，对于每一个含有标记位的view及其子View都会进行测量、布局、绘制，性能较差，源码体现如下：关于requestLayout ()方法的更多分析可以查看这一篇博客[Android View 深度分析requestLayout、invalidate与postInvalidate](http://blog.csdn.net/a553181867/article/details/51583060)

```java
public void setLayoutParams(ViewGroup.LayoutParams params) {
    if (params == null) {
        throw new NullPointerException("Layout parameters cannot be null");
    }
    mLayoutParams = params;
    resolveLayoutParams();
    if (mParent instanceof ViewGroup) {
        ((ViewGroup) mParent).onSetLayoutParams(this, params);
    }
    requestLayout();
}
```

因此我们如果在api 14 以后 ，在动画执行过程中，要改变View的状态，推荐使用setTranslationY()和setTranslationX（0等方法，而 尽量避免改变LayoutParams.因为性能嫌贵来说较差。

## event.getY 和 event.getRawY()

要区分于MotionEvent.getRawX() 和MotionEvent.getX();,

在public boolean onTouch(View view, MotionEvent event) 中，当你触到控件时，x,y是相对于该控件左上点（控件本身）的相对位置。 而rawx,rawy始终是相对于屏幕的位置。getX()是表示Widget相对于自身左上角的x坐标,而getRawX()是表示相对于屏幕左上角的x坐标值 (注意:这个屏幕左上角是手机屏幕左上角,不管activity是否有titleBar或是否全屏幕)。

![img](http://upload-images.jianshu.io/upload_images/2050203-8cf2b82342f5f76b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 扩展，怎样获取状态栏（StatusBar）和标题栏（titleBar）的高度

```java
     public void onWindowFocusChanged(boolean hasFocus) {
        super.onWindowFocusChanged(hasFocus);

        //屏幕
        DisplayMetrics dm = new DisplayMetrics();
        getWindowManager().getDefaultDisplay().getMetrics(dm);
        Log.e(TAG, "屏幕高:" + dm.heightPixels);

        //应用区域
        Rect outRect1 = new Rect();
        getWindow().getDecorView().getWindowVisibleDisplayFrame(outRect1);
        //这个也就是状态栏的 高度
        Log.e(TAG, "应用区顶部" + outRect1.top);

        Log.e(TAG, "应用区高" + outRect1.height());

        // 这个方法必须在有actionBar的情况下才能获取到状态栏的高度
        //View绘制区域
        Rect outRect2 = new Rect();
        getWindow().findViewById(Window.ID_ANDROID_CONTENT).getDrawingRect(outRect2);
        Log.e(TAG, "View绘制区域顶部-错误方法：" + outRect2.top);   //不能像上边一样由outRect2.top获取，这种方式获得的top是0，可能是bug吧
        Log.e(TAG, "View绘制区域高度：" + outRect2.height());

        int viewTop = getWindow().findViewById(Window.ID_ANDROID_CONTENT).getTop();   //要用这种方法
        Log.e(TAG, "View绘制区域顶部-正确方法：" + viewTop);

        int titleBarHeight=viewTop;

        Log.d(TAG, "onWindowFocusChanged: 标题栏高度titleBarHeight=" +titleBarHeight);

    }
```

这里我们需要注意的 是在ActionBar存在的情况下，通过这种方法我们才能够得出titleBar的高度，否则是无法得到的，因为viewTop 为0.

这篇博客到此为止，关于更多自定义View 的一些例子，可以看我以下的博客

[**常用的自定义View例子一(FlowLayout)**](http://blog.csdn.net/gdutxiaoxu/article/details/51765428)

[**自定义View常用例子二（点击展开隐藏控件，九宫格图片控件）**](http://blog.csdn.net/gdutxiaoxu/article/details/51772308)

[**常用的自定义View例子三（MultiInterfaceView多界面处理）**](http://blog.csdn.net/gdutxiaoxu/article/details/51804844)

[**常用的自定义控件四（QuickBarView）**](http://blog.csdn.net/gdutxiaoxu/article/details/51804865)