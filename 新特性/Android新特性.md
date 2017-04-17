# 1. Android5.0新特性

## 1. 性能提升

- 全面由 Dalvik 虚拟机转用 Android RunTime（ART）虚拟机

- 64位CPU支持

- NDK开发工具

- 改进的通知栏

- 支持64位处理器。

- 支持Bluetooth 4.1。

- 改良的通知界面及新增Priority Mode。

- 强化网络及传输连接性，包括Wi-Fi、蓝牙及NFC。

- 强化多媒体功能，例如支持RAW格式拍摄。

- 改善Android TV的支持。

- 提供低视力的设置，以协助色弱人士。

- 预载省电及充电预测功能。

- 新增自动内容加密功能。

- 支持心跳传感器。

- 新增多人设备分享功能，可在其他设备登录自己账号，并获取用户的联系人、日历等Google云数据。

- 改善Google Now功能。

[http://zh.wikipedia.org/wiki/Android](http://zh.wikipedia.org/wiki/Android)

[https://developer.android.com/about/versions/android-5.0-changes.html](https://developer.android.com/about/versions/android-5.0-changes.html)

## 2. MaterialDesign设计风格

### 2.1.UI控件，DesignLibrary

### 2.2.全新的动画

## 3. 支持多种设备

## 4. 支持64位ART虚拟机


[http://zh.wikipedia.org/wiki/Android%E6%AD%B7%E5%8F%B2%E7%89%88%E6%9C%AC](http://zh.wikipedia.org/wiki/Android%E6%AD%B7%E5%8F%B2%E7%89%88%E6%9C%AC)


[http://en.wikipedia.org/wiki/Android_version_history](http://en.wikipedia.org/wiki/Android_version_history)

## 5. 主题样式

# 2. Android6.0新特性

[http://bbs.itheima.com/forum.php?mod=viewthread&tid=262734&highlight=android6.0](http://bbs.itheima.com/forum.php?mod=viewthread&tid=262734&highlight=android6.0)

## 1. 大量漂亮流畅的动画

## 2. 支持快速充电的切换

## 3. 支持文件夹拖拽应用

## 4. 相机新增专业模式

## 5. DataBinding

## 6. Runtime Permission

## 7. Direct Share

### 7.1.ChooserTargetService

```xml
<service

   android:name=".SampleChooserTargetService"

   android:label="@string/app_name"

   android:permission="android.permission.BIND_CHOOSER_TARGET_SERVICE">

   <intent-filter>

       <actionandroid:name="android.service.chooser.ChooserTargetService" />

   </intent-filter>

</service>
```

## 8. OkHttp

Android6.0已经移除了ApacheHTTP Client,推荐使用HttpURLConnection，如果还相继续使用俩种HTTP请求方式，OkHttp会是一个不错的选择。

## 9. Text Selection

```java
private ActionMode.Callback2
callback2 = new ActionMode.Callback2()
{
    @Override
    public boolean onCreateActionMode(ActionMode mode, Menu menu) {
        MenuInflater menuInflater =
mode.getMenuInflater();
        menuInflater.inflate(R.menu.text,menu);
        return true;
    }
    @Override
    public boolean onPrepareActionMode(ActionMode mode, Menu menu) {
        return false;
    }
    @Override
    public boolean onActionItemClicked(ActionMode mode, MenuItem item) {
        return false;
    }
    @Override
    public void onDestroyActionMode(ActionMode
mode) {
    }
    //控制这个浮动菜单的位置
    @Override
    public void onGetContentRect(ActionMode mode, View view, Rect outRect) {
        super.onGetContentRect(mode, view, outRect);
    }
};
```

# 3. Android7.0新特性

## 1. JIT

即使编译

## 2. 支持Vulkan API

## 3. 分屏多任务

## 4. 可恢复的Notification

Notification.Action

### 4.1.通知机制

分组显示

## 5. 增强的Java8语言模式

## 6. 夜间模式

## 7. 流量节省

## 8. ICU4J

国际化

## 9. Direct Boot

直接启动

## 10. VR支持

# 4. MaterialDesign

## 1. 主题样式

android:Theme.Material

### 1.1.主题配色

![img](file:///C:/Users/ALLENI~1/AppData/Local/Temp/msohtmlclip1/01/clip_image002.png)


| 属性                                  | 说明                                   |
| :---------------------------------- | :----------------------------------- |
| android:colorPrimaryDark            | 应用的主要暗色调，statusBarColor默认使用该颜色       |
| android:statusBarColor              | 状态栏颜色，默认使用colorPrimaryDark           |
| android:colorPrimary                | 应用的主要色调，actionBar默认使用该颜色             |
| android:windowBackground            | 窗口背景颜色                               |
| android:navigationBarColor          | 底部栏颜色                                |
| android:colorForeground             | 应用的前景色，ListView的分割线，switch滑动区默认使用该颜色 |
| android:colorBackground             | 应用的背景色，popMenu的背景默认使用该颜色             |
| android:colorAccent                 | 一般控件的选种效果默认采用该颜色                     |
| android:colorControlNormal          | 控件的默认色调                              |
| android:colorControlHighlight       | 控件按压时的色调                             |
| android:colorControlActivated       | 控件选中时的颜色，默认使用colorAccent             |
| android:colorButtonNormal           | 默认按钮的背景颜色                            |
| android:textColor                   | Button ，TextView的文字颜色                |
| android:textColorPrimaryDisableOnly | RadioButton checkbox等控件的文字           |
| android:textColorPrimary            | 应用的主要文字颜色，actionBar的标题文字默认使用该颜色      |

### 1.2.设置主题

setTheme(R.style.xx) 必须在setContentView()之前调用

解决方法：重启自己

```java
public void reload(){

finish();//开始前先结束自己

overridePendingTransition(0,0);
    Intent intent = new Intent(MainActivity.this, MainActivity.class);
    intent.putExtra("themeId",R.style.*AppTheme*);
    startActivity(intent);

overridePendingTransition(0,0);
}



protected void onCreate(Bundle
savedInstanceState) {
    super.onCreate(savedInstanceState);
    int themeId =
getIntent().getIntExtra("themeId",-1);
if (themeId != -1){
        setTheme(themeId);
    }
    setContentView(R.layout.*activity_main*);

}
```

## 2. 阴影和剪裁

### 2.1.阴影

elevation：海拔，控制阴影，还控制层次关系

translationZ

注意：要设置一定的margin，才能看到阴影效果

### 2.2.View的轮廓

设置背景：shape，color，图片

android:outlineProvider

| 属性           | 说明                      |
| :----------- | :---------------------- |
| none         | 即使设置了Z属性，也不会显示阴影        |
| background   | 会按照背景来设置阴影形状            |
| bounds       | 会按照View的大小来描绘阴影         |
| paddedBounds | 和bounds类似，不过阴影会稍微向右偏移一点 |



代码设置
```java
ViewOutlineProvider
viewOutlineProvider = new ViewOutlineProvider()
{
    public void getOutline(View view, Outline outline)
{
        // 可以指定圆形，矩形，圆角矩形，path
        outline.setOval(0, 0, view.getWidth(), view.getHeight());
    }
};
View.setOutlineProvider(viewOutlineProvider );
```


outline.canClip()判断一个轮廓是否支持剪裁



注意：如果采用图片作为背景，即使在xml布局中指定android:outlineProvider为background也不会显示阴影，只有通过代码中指定轮廓来显示。

### 2.3.View的剪裁

setClipToOutline(true)

## 3. 图片和颜色

### 3.1.Tint着色

tint属性一个颜色值，可以对图片做颜色渲染，我们可以给view的背景设置tint色值，给ImageView的图片设置tint色值，也可以给任意Drawable或者NinePatchDrawable设置tint色值。



在应用的主题中也可以通过设置 android:tint 来给主题设置统一的颜色渲染。



tint的渲染模式（android:tintMode）有总共有16种，xml文件中可以使用6种，代码中我们可以设置16种，渲染模式决定了渲染颜色和原图颜色的取舍和合成规则：



setTint()，android:tint和android:initMode属性设置染色的颜色和模式

![img](file:///C:/Users/ALLENI~1/AppData/Local/Temp/msohtmlclip1/01/clip_image003.png)

### 3.2.Selector
```xml
<?xml version="1.0" encoding="utf-8"?>
<bitmap
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:src="@drawable/progress_normal"
    android:tintMode="multiply"
    android:tint="#29aeba"/>



<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:drawable="@drawable/tint_bitmap"
android:state_pressed="true"/>
    <item android:drawable="@drawable/ring"
/>
</selector>
```
### 3.3.Palette调色板

| 颜色            | 说明    |
| ------------- | ----- |
| Vibrant       | 鲜艳的   |
| Vibrant dark  | 鲜艳的暗色 |
| Vibrant light | 鲜艳的亮色 |
| Muted         | 柔和的   |
| Muted dark    | 柔和的暗色 |
| Muted light   | 柔和的亮色 |

Palette

| 方法声明                           | 功能描述        |
| ------------------------------ | ----------- |
| from(bitmap)                   | 从bitmap上取颜色 |
| generate()                     | 同步          |
| generate(PaletteAsyncListener) | 异步          |
| Palette.Swatch                 | 样色          |
| getVibrantSwatch()             | 获取鲜艳的颜色     |
| getDarkVibrantSwatch()         | 获取鲜艳的暗色     |
| getLightVibrantSwatch()        | 获取鲜艳的亮色     |
| getMutedSwatch()               | 获取柔和的颜色     |
| getDarkMutedSwatch()           | 获取柔和的暗色     |
| getLightMutedSwatch()          | 获取柔和的亮色     |
| getBodyTextColor()             |             |
| getTitleTextColor()            | 获取标题颜色      |
| getRgb()                       |             |
| getPopulation()                |             |
| getVibrantColor()              |             |
| getMutedColor()                |             |


```java
Bitmap bitmap =
BitmapFactory.*decodeResource*(getResources(),R.mipmap.*ic_launcher*);
Palette.*from*(bitmap).generate(new Palette.PaletteAsyncListener()
{
    @Override
    public void onGenerated(Palette
palette) {
        Palette.Swatch vibrant =
palette.getVibrantSwatch();
        Palette.Swatch
darkVibrant = palette.getDarkVibrantSwatch();
        Palette.Swatch
lightVibrant = palette.getLightVibrantSwatch();
        Palette.Swatch
muted = palette.getMutedSwatch();
        Palette.Swatch
darkMuted = palette.getDarkMutedSwatch();
        Palette.Swatch
lightMuted = palette.getLightMutedSwatch();
        int bodyTextColor =
vibrant.getBodyTextColor();
        int titleTextColor =
vibrant.getTitleTextColor();
        int rgb =
vibrant.getRgb();
        int population =
vibrant.getPopulation();
        int vibrantColor =
palette.getVibrantColor(Color.*WHITE*);
        int mutedColor =
palette.getMutedColor(Color.*WHITE*);
    }
});
```

### 3.4.Vector矢量图

（有线段和曲线组成，占用空间小，与分辨率无关，颜色不丰富），位图（像素点描述）

## 4. 全新的动画

### 4.1.Touch feedback触摸反馈

| 属性                                       | 说明                         |
| ---------------------------------------- | -------------------------- |
| ?android:attr/selectableItemBackground   | 有界水波纹                      |
| ?android:attr/selectableItemBackgroundBorderless | 无界水波纹                      |
| android:colorControlHighlight            | 水波纹颜色                      |
| android:colorAccent                      | 可以修改checkbox的选中颜色          |
| RippleDrawable                           | 水波纹效果，对应xml中的<  ripple >标签 |
| RippleDrawableColorStateList             |                            |


```xml
<ripple xmlns:android="http://schemas.android.com/apk/res/android"
        android:color="?android:attr/colorControlHighlight">
    <item>
        <inset
            android:insetLeft="4dp"
            android:insetTop="6dp"
            android:insetRight="4dp"
            android:insetBottom="6dp">
            <shape android:shape="rectangle">
                <corners android:radius="2dp" />
                <solid android:color="?android:attr/colorButtonNormal"
/>
                <padding android:left="8dp"
                         android:top="4dp"
                         android:right="8dp"
                         android:bottom="4dp" />
            </shape>
        </inset>
    </item>
</ripple>
```


?android:attr/selectableItemBackground
```xml
<ripple xmlns:android="http://schemas.android.com/apk/res/android"
        android:color="?android:attr/colorControlHighlight">
    <item android:id="@android:id/mask">
        <color android:color="@android:color/white"/>
    </item>
</ripple>
```


?android:attr/selectableItemBackgroundBorderless
```xml
<ripple xmlns:android="http://schemas.android.com/apk/res/android"
        android:color="?android:attr/colorControlHighlight"/>



<?xml version="1.0" encoding="utf-8"?>
<ripple xmlns:android="http://schemas.android.com/apk/res/android"
        android:color="@color/lighter_gray">
    <item android:drawable="@color/white"/>
</ripple>
```


代码设置
```java
RippleDrawableColorStateList
stateList      =
getResources().getColorStateList(R.color.tint_state_color);
RippleDrawable  rippleDrawable = new RippleDrawable(stateList,null,null);
view.setBackground(rippleDrawable);
```
### 4.2.Circular Reveal圆形展示

RevealAnimator，隐藏或者显示一个view，改变view的大小等过渡效果，填充效果（Reveal Effect）



沿着中心的缩小的动画
```java
Animator
animator = ViewAnimationUtils.*createCircularReveal*(
        view, //操作的视图
        view.getWidth() / 2, //动画开始的中心点X
        view.getHeight() / 2, //动画开始的中心点Y
        view.getWidth(), //动画开始半径
        0 //动画结束半径
);
animator.setInterpolator(new LinearInterpolator());
animator.setDuration(1000);
animator.start();
```


从左上角扩展的圆形动画
```java
Animator
animator = ViewAnimationUtils.*createCircularReveal*(
        view,0,0,0,(float) Math.*hypot*(view.getWidth(), view.getHeight()));
animator.setDuration(1000);
animator.start();
```
### 4.3.Curved motion曲线运动

Android5.0之前的实现方式
```java
public class ArcTranslateAnimation
extends Animation {
    private float mFromXValue,mToXValue,mFromYValue,mToYValue;
    private float mFromXDelta,mToXDelta,mFromYDelta,mToYDelta;
    private PointF mStart,mControl,mEnd;
    public ArcTranslateAnimation(float fromXValue, float toXValue, float fromYValue, float toYValue) {
        mFromXValue = fromXValue;
        mToXValue = toXValue;
        mFromYValue = fromYValue;
        mToYValue = toYValue;
    }
    protected void applyTransformation(float interpolatedTime, Transformation
t) {
        float dx =
calcBezier(interpolatedTime, mStart.x, mControl.x, mEnd.x);
        float dy =
calcBezier(interpolatedTime, mStart.y, mControl.y, mEnd.y);
        t.getMatrix().setTranslate(dx, dy);
    }
    public void initialize(int width, int height, int parentWidth, int parentHeight) {
        super.initialize(width, height, parentWidth, parentHeight);
        mFromXDelta = resolveSize(*ABSOLUTE*, mFromXValue, width, parentWidth);
        mToXDelta = resolveSize(*ABSOLUTE*, mToXValue, width, parentWidth);
        mFromYDelta = resolveSize(*ABSOLUTE*, mFromYValue, height, parentHeight);
        mToYDelta = resolveSize(*ABSOLUTE*, mToYValue, height, parentHeight);
        mStart = new PointF(mFromXDelta, mFromYDelta);
        mEnd = new PointF(mToXDelta, mToYDelta);
        mControl = new PointF(mFromXDelta, mToYDelta);
    }
    private long calcBezier(float interpolatedTime, float p0, float p1, float p2) {
        return Math.*round*((Math.*pow*((1 -
interpolatedTime), 2) * p0)
                + (2 * (1 -
interpolatedTime) * interpolatedTime * p1) +
                (Math.*pow*(interpolatedTime, 2) * p2);
    }
}
```


ObjectAnimator新增了path方式来构建动画，并且可以同时对x，y两个属性做动画，我们只用指定一个曲线的path，即可作出曲线的动画，可以用quadTo/cubicTo绘制贝塞尔曲线，也可以使用arcTo绘制普通的弧线 新增了PathInterpolator动画插入器，新的基于贝塞尔曲线或路径对象的插入器。这个插入器指定了一个1x1正方形运动曲线，它使用(0,0)为锚点，(1,1)为控制点，作为构造函数的参数。
```java
Path path = new Path();
path.moveTo(100,100);
path.quadTo(1000,300,300,700);
ObjectAnimator mAnimator = ObjectAnimator.ofFloat(curved, View.*X*, View.*Y*, path);
Path p = new Path();
p.lineTo(0.6f, 0.9f);
p.lineTo(0.75f, 0.2f);
p.lineTo(1f, 1f);
mAnimator.setInterpolator(new PathInterpolator(p));
mAnimator.setDuration(3000);
mAnimator.start();
```
### 4.4.View state changes视图状态变化

Android L在原有的图片选择器和颜色选择器上进行了增强，不仅是控件能根据不同的状态显示不同的背景图片，还能在两种状态切换时指定一个动画，来增加过渡效果，吸引用户眼球，以突出重点内容。



StateListAnimator类和图片选择器，颜色选择器类似，可以根据view的状态改变呈现不同的动画效果，通过xml我们可以构建对应不同状态的动画合集，其使用方式也非常简单，在对应的状态指定一个属性动画即可


```
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_pressed="true">
        <set>
            <objectAnimator android:propertyName="translationZ"
                            android:duration="200"
                            android:valueTo="20dp"
                            android:valueType="floatType"/>
        </set>
    </item>
    <item android:state_enabled="true"
android:state_pressed="false">
        <set>
            <objectAnimator android:propertyName="translationZ"
                            android:duration="200"
                            android:valueTo="0"
                            android:valueType="floatType"/>
        </set>
    </item>
</selector>
```


在xml中通过android:stateListAnimator来指定状态动画，代码中可以通过AnimationInflater.loadStateListAnimator()加载动画，并使用view.setStateListAnimator()将其指定给View。



可以在状态切换的过程指定多个属性动画的合集，继承了Material主题后，按钮默认拥有了z属性动画。如果想取消这种默认状态，可以把状态动画指定为null。



除了StateListAnimator类指定状态切换的属性动画外，还可以通过AnimatedStateListDrawable来指定状态切换的帧动画
```
<animated-selector
xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:id="@+id/pressed"
android:drawable="@drawable/btn_check_15"
          android:state_pressed="true"/>
    <item android:id="@+id/normal"  android:drawable="@drawable/btn_check_0"/>
    <transition android:fromId="@+id/normal"
android:toId="@+id/pressed">
        <animation-list>
            <item android:duration="20" android:drawable="@drawable/btn_check_0"/>
            <item android:duration="20" android:drawable="@drawable/btn_check_1"/>
            <item android:duration="20" android:drawable="@drawable/btn_check_2"/>
        </animation-list>
    </transition>
</animated-selector>
```
### 4.5.Vector Drawables矢量图动画

前面我们学习了矢量图，AnimatedVectorDrawable类让你能使一个矢量图动起来。矢量图动画比帧动画更平滑的展现图片的变化过程，并且无论在内存占用，还是包体积占用上都要优于帧动画。通常定义一个矢量图动画需要三步：



在drawable资源目录下定义一个矢量图
```
<vector xmlns:android="http://schemas.android.com/apk/res/android"
        android:height="64dp"
        android:width="64dp"
        android:viewportHeight="600"
        android:viewportWidth="600">
    <group
        android:name="rotationGroup"
        android:pivotX="300.0"
        android:pivotY="300.0"
        android:rotation="45.0"
\>
        <path
            android:name="v"
            android:fillColor="#000000"
            android:pathData="M300,70 l
0,-70 70,70 0,0 -70,70z" />
    </group>
</vector>
```


在anim下定义一个objectAnimator，并在动画中修改矢量图的path
```
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <objectAnimator
        android:duration="3000"
        android:propertyName="pathData"
        android:valueFrom="M300,70 l
0,-70 70,70 0,0 -70,70z"
        android:valueTo="M300,70 l
0,-70 70,0  0,140 -70,0z"
        android:valueType="pathType"
/>
</set>
```

在drawable下定义一个animated-vector，并把drawable指向矢量图，把target中的动画指定为之前定义的objectAnimator
```
<animated-vector
xmlns:android="http://schemas.android.com/apk/res/android"
                   android:drawable="@drawable/vector_drawable"
\>
    <target
        android:name="v"
        android:animation="@anim/vector_anim"
/>
</animated-vector>
```
### 4.6.Activity transitions活动转场

l  XML设置

android:windowContentTransitions

android:windowEnterTransition进入

android:windowExitTransition退出

android:windowReturnTransition返回

android:windowReenterTransition重新进入



l  代码设置
```java
requestFeature(Window.FEATURE_CONTENT_TRANSITIONS)



setEnterTransition() 进入

setExitTransition() 退出

setReturnTransition() 返回

setReenterTransition() 重新进入



setSharedElementEnterTransition()

setSharedElementExitTransition()

setSharedElementReturnTransition()

setSharedElementReenterTransition()

setAllowEnterTransitionOverlap()



view.setTransitionName()
```


| Visibility 进入退出              |                  |
| ---------------------------- | ---------------- |
| Fade                         | 淡入淡出，改变视图的透明度    |
| Slide                        | 滑动，从屏幕边缘移入或移出视图  |
| Explode                      | 爆炸，从屏幕中心移入或移出视图  |
| 共享转场动画                       |                  |
| ChangeClipBounds             | 裁剪目标视图边界         |
| ChangeBounds                 | 改变目标视图的布局边界      |
| ChangeTransform              | 改变目标视图的缩放比例和旋转角度 |
| ChangeImageTransform         | 改变目标图片的大小和缩放比例   |
| TransitionSet                |                  |
| addTransition()              |                  |
| ChangeScroll                 |                  |
| ActivityOptions              |                  |
| makeSceneTransitionAnimation |                  |
| toBundle()                   |                  |
| Pair.create()                |                  |



在Android L之前，我们可以在startActivity之后调用overridePendingTransition来指定Activity的转场动画。现在Android L给我们带来了更绚丽的转场动画。



新的转场动画分为两大类，一种是普通的过渡动画，另一种是共享元素的过渡动画。 要想使用新的转场动画，可以继承Material Design主题后在style风格中指定：

<style name="DefaultTheme"
parent="android:Theme.Material">
​    <!-- 允许使用transitions -->
​    <item name="android:windowContentTransitions">true</item>
​    <!-- 指定进入、退出、返回、重新进入时的transitions -->
​    <item name="android:windowEnterTransition">@transition/explode</item>
​    <item name="android:windowExitTransition">@transition/explode</item>
​    <item name="android:windowReturnTransition">@transition/explode</item>
​    <item name="android:windowReenterTransition">@transition/explode</item>
​    <!-- 指定进入、退出、返回、重新进入时的共享transitions -->
​    <item name="android:windowSharedElementEnterTransition">@transition/change</item>
​    <item name="android:windowSharedElementExitTransition">@transition/change</item>
​    <item name="android:windowSharedElementReturnTransition">@transition/change</item>
​    <item name="android:windowSharedElementReenterTransition">@transition/change</item>
</style>



也可以在activity的oncreate方法中进行代码设置：

// 允许使用transitions
getWindow().requestFeature(Window.*FEATURE_CONTENT_TRANSITIONS*);
// 指定进入、退出、返回、重新进入时的transitions
getWindow().setEnterTransition(new Explode());
getWindow().setExitTransition(new Explode());
getWindow().setReturnTransition(new Explode());
getWindow().setReenterTransition(new Explode());
// 指定进入、退出、返回、重新进入时的共享transitions
getWindow().setSharedElementEnterTransition(new ChangeTransform());
getWindow().setSharedElementExitTransition(new ChangeTransform());
getWindow().setSharedElementReturnTransition(new ChangeTransform());
getWindow().setSharedElementReenterTransition(new ChangeTransform());



**普通转场动画**

** **

所有继承自Visibility类都可以作为进入、退出的过度动画。如果我们想自定义进入和退出时的动画效果，只需要继承Visibility，重载onAppear和onDisappear方法来定义进入喝退出的动画。系统提供了三种默认方式



想在xml中指定自定义的进入、退出的过度动画需要先对动画进行定义：

<transition class="my.app.transition.CustomTransition"/>



注意：其中CustomTransition是你自定义的动画，它必须继承自Visibility。

想以普通转场动画的方式启动一个Activity，必须在startActivity函数中传递一个ActivityOptions的Bundle对象：

ActivityOptions
options = ActivityOptions.*makeSceneTransitionAnimation*(activity);
startActivity(intent, options.toBundle());



如果想让返回也具备转场效果，那么在返回的Activity中不要再调用finish()函数，而是应该使用finishAfterTransition()来结束一个Activity，该函数会等待动画执行完毕才结束该Activity。



**共享转场动画**

** **

当两个Activity具备某些相遇的元素时，共享转场动画将是一个非常好的选择。使用转场动画需要将相同的元素通过android:transitionName或者view.setTransitionName()设置为相同的名称，这样系统才能区分出相同的元素。



共享元素：两个Activity都有的的元素，一个共字元素过渡动画决定两个Activities之间的过渡，怎么共享它们的视图



通过下面的函数启动一个共享元素动画

ActivityOptions
options = ActivityOptions.makeSceneTransitionAnimation(activity, view, "name");
startActivity(intent, options.toBundle());



如果有多个共享元素，则可以通过Pair进行包装处理：

ActivityOptions
options = ActivityOptions.makeSceneTransitionAnimation(activity,Pair.create(view1, "name1"),Pair.create(view2, "name2"));
startActivity(intent, options.toBundle());

返回时如果需要具备转场动画，那么也需要用finish()函数替代finishAfterTransition()来结束一个Activity。



共享转场动画通常可以根据指定的元素判断出合适的转场动画效果，不需要我们做额外的处理，也可以通过之前学习的方法进行指定共享元素转场动画效果。



**组合转场动画**



我们可以把多个转场动画进行组合，作出更具个性的转场效果，在资源文件中通过以下方式：

<transitionSet
xmlns:android="http://schemas.android.com/apk/res/android">
​    <explode/>
​    <transition class="my.app.transition.CustomTransition"/>
​    <<changeImageTransform/>
</transitionSet>



代码中我们可以通过TransitionSet类组合多个转场动画：

TransitionSet
transitionSet = new TransitionSet();
transitionSet.addTransition(new Fade());
transitionSet.addTransition(new ChangeBounds());



组合可以同时针对普通转场动画和共享元素转场动画。



转场动画也可以像普通动画一样设置持续时间，延期执行时间，速率插入器，以及动画的监听等。



转场动画通常是对整个布局起作用，如果我们想对某个特定的view实施转场动画，可以把该view设置为转场动画的target，这样转场动画将只对特定的view起作用。共享元素的动画的target需要指定为transitionName

## 5. 新增的Widget

### 5.1.RoundedBitmapDrawable

```java
Bitmap src = BitmapFactory.decodeResource(getResources(),imageId);

RoundedBitmapDrawable roundedDrawable = RoundedBitmapDrawableFactory

        .create(getResources(), src);

roundedDrawable.setCornerRadius( 100 ); //设置圆角半径

roundedDrawable.setAntiAlias( true ); //设置反走样

imageview.setImageDrawable(roundedBitmapDrawable); //显示圆角图片
```

### 5.2.CardView

阴影效果：cardElevation，代码设置：setCardElevation()

圆角：cardCornerRadius，代码设置：setRadius()

背景：carBackgroundColor，代码设置：setBackgroundColor()

### 5.3.RecyclerView

## 6. CoordinatorLayout

```java
class CoordinatorLayout extends ViewGroup implements NestedScrollingParent
```

CoordinatorLayout 的神奇之处就在于Behavior对象。怎么理解呢？CoordinatorLayout使得子view之间知道了彼此的存在，一个子view的变化可以通知到另一个子view，CoordinatorLayout 所做的事情就是当成一个通信的桥梁，连接不同的view，使用 Behavior 对象进行通信。

比如：在CoordinatorLayout中使用AppBarLayout，如果AppBarLayout的子View（如ToolBar、TabLayout）标记了app:layout_scrollFlags滚动事件，那么在CoordinatorLayout布局里其它标记了app:layout_behavior的子View（LinearLayout、RecyclerView、NestedScrollView等）就能够响应（如ToolBar、TabLayout）控件被标记的滚动事件。

AppBarLayout来让Toolbar响应滚动事件

当CoordinatorLayout发现RecyclerView中定义了这个属性，它会搜索自己所包含的其他view，看看是否有view与这个behavior相关联。AppBarLayout.ScrollingViewBehavior描述了RecyclerView与AppBarLayout之间的依赖关系。RecyclerView的任意滚动事件都将触发AppBarLayout或者AppBarLayout里面view的改变。AppBarLayout里面定义的view只要设置了app:layout_scrollFlags属性，就可以在RecyclerView滚动事件发生的时候被触发

CoordinatorLayout的工作原理是搜索定义了CoordinatorLayout.Behavior 的子view，不管是通过在xml中使用app:layout_behavior标签还是通过在代码中对view类使用@DefaultBehavior修饰符来添加注解。当滚动发生的时候，CoordinatorLayout会尝试触发那些声明了依赖的子view。

CoordinatorLayout是design库中的一个继承自viewgroup控件，功能十分强大，而这最主要的功臣就是behavior，这个behavior是个什么呢，它算是CoordinatorLayout子view的一种插件，可以管理子view的拖，刷，拉等等一系列手势操作，CoordinatorLayout是统筹全局的管理者，组织众多子View相互协调，当一个子View位置或者滚动状态发生变化会及时通知给其他子view，CoordinatorLayout所做的事情就是变成一个通信的桥梁，连接不同的view，使用Behavior对象进行通信。在XML中我们常常只设置app:layout_behavior属性来实现不同的滚动策略，这里CoordinatorLayout通过反射来实现behavior的实例化

behavior是CoordinatorLayout中的一个内部类，它的实例化是同样内部类中的LayoutParams来实现的。

设置了layout_scrollFlags标志的View必须在没有设置的View的之前定义，这样可以确保设置过的View都从上面移出, 只留下那些固定的View在下面。

### 视图间的依赖

layout_anchor：固定在那个布局区域

layout_anchorGravity：固定在布局的那个重心位置

### 6.1.代码设置behavior
```java
CoordinatorLayout.LayoutParams
params;
SwipeDismissBehavior<TextView> behavior = new SwipeDismissBehavior<>();
behavior.setListener(new SwipeDismissBehavior.OnDismissListener()
{
    @Override
    public void onDismiss(View view) {
    }
    @Override
    public void onDragStateChanged(int state) {
    }
});
params.setBehavior(behavior);
```

### 6.2.自定义behavior
```java
class MyBehavior extends CoordinatorLayout.Behavior{
    public MyBehavior(Context
context, AttributeSet attrs) {
        super(context, attrs);
    }
    @Override
    public boolean onStartNestedScroll(CoordinatorLayout
coordinatorLayout, View child, View
            directTargetChild, View target, int nestedScrollAxes)
{
        return super.onStartNestedScroll(coordinatorLayout, child, directTargetChild, target,
                nestedScrollAxes);
    }
    @Override
    public void onNestedPreScroll(CoordinatorLayout
coordinatorLayout, View child, View
            target, int dx, int dy, int[] consumed) {
        super.onNestedPreScroll(coordinatorLayout, child, target, dx, dy, consumed);
    }
}
```
## 7. AppBarLayout

是继承LinerLayout实现的一个ViewGroup容器组件，它的作用是把AppBarLayout包裹的内容都作为AppBar

```
// 布局文件设置
app:layout_scrollFlags="scroll|enterAlways"
// 代码设置
AppBarLayout.LayoutParams params;
params.setScrollFlags();
```

| 标记                   | 说明                                       |
| :------------------- | :--------------------------------------- |
| scroll               | 所有想滚动出屏幕的View都需要设置这个flag，没有设置这个flag的View将被固定在屏幕顶部 |
| enterAlways          | 这个flag让任意向下的滚动都会导致该View变为可见，启用快速“返回模式”   |
| enterAlwaysCollapsed | 这个flag定义的是何时进入，假设你定义了一个最小高度（minHeight）同时enterAlways也定义了，那么view将在到达这个最小高度的时候开始显示，并且从这个时候开始慢慢展开，当滚动到顶部的时候展开完 |
| exitUntilCollapsed   | 这个flag定义何时退出，当你定义了一个minHeight，这个view将在滚动到达这个最小高度的时候消失。 |

### 7.1.ScrollingViewBehavior

```
// 布局文件设置
app:layout_behavior="@string/appbar_scrolling_view_behavior"

// 通过注解方式设置
@CoordinatorLayout.DefaultBehavior(AppBarLayout.ScrollingViewBehavior.class)
RecyclerView mRecyclerView;

// 代码设置
CoordinatorLayout.LayoutParams params = (CoordinatorLayout.LayoutParams)mRecyclerView.getLayoutParams();
params.setBehavior(new AppBarLayout.ScrollingViewBehavior());
```

AppBarLayout. 用来通知AppBarLayout这个特殊的view何时发生了滚动事件，这个behavior需要设置在触发事件（滚动）的view之上。


```
app:contentScrim=”?attr/colorPrimary” 收缩的时候自动的变化到普通的颜色
```


## 8. CollapsingToolbarLayout

| 方法声明                                  | 功能描述                     |
| :------------------------------------ | :----------------------- |
| app:layout_collapseMode               | pin固定在顶部  parallax视差特效   |
| app:layout_collapseParallaxMultiplier | 视差因子                     |
| app:contentScrim=＂?attr/colorPrimary＂ | 折叠后的背景色                  |
| setExpandedTitleColor()               | 扩张时候的title颜色             |
| setCollapsedTitleTextColor()          | 收缩后在Toolbar上显示时的title的颜色 |



| 属性                               | 说明        |
| :------------------------------- | :-------- |
| app:collapsedTitleTextAppearance | 折叠时的文本    |
| app:expandedTitleTextAppearance  | 展开时的文本    |
| app:expandedTitleMarginStart     | 展开时标题的外边距 |
| app:expandedTitleMarginEnd       |           |
| app:expandedTitleMarginBottom    |           |

使用CollapsingToolbarLayout时必须把title设置到CollapsingToolbarLayout上，设置到Toolbar上不会显示，mCollapsingToolbarLayout.setTitle("")

setTitle()：title 会在展开的时候自动变得大些，而在折叠的时候让字体过渡到默认值

不用CollapsingToolbarLayout包裹ToolBar的话，ToolBar会滑出屏幕

## 9. ToolBar

如果你要将Toolbar当作actionbar来使用，你首先要去掉actionbar，最简单的方法是使用Theme.AppCompat.NoActionBar主题。或者是设置主题的属性android:windowNoTitle为true。然后在Activity的onCreate中调用setSupportActionBar(toolbar)，原本应该出现在ActionBar上的menu会自动出现在actionbar上。

| 方法声明                         | 功能描述                                     |
| :--------------------------- | :--------------------------------------- |
| setSupportActionBar(toolbar) | 把ToolBar设置作为ActionBar                    |
| setOnMenuItemClickListener() | 设置菜单的点击监听                                |
| setLogo()                    | 设置图标                                     |
| setTitle()                   | 设置标题                                     |
| setSubtitle()                | 设置副标题                                    |
| android:background           | "?attr/colorPrimary"                     |
| android:layout_height        | "?attr/actionBarSize"  指定ToolBar收缩后的高度和actionBar保持一致 |


```java
toolbar.setOnMenuItemClickListener(new Toolbar.OnMenuItemClickListener()
{
    @Override
    public boolean onMenuItemClick(MenuItem item)
{
        // 处理menu事件
        return true;
    }
});
// 创建一个menu添加到toolbar上
toolbar.inflateMenu(R.menu.your_toolbar_menu);
```
## 10. TabLayout

### 10.1.常用方法

| 方法声明                                  | 功能描述                                     |
| ------------------------------------- | ---------------------------------------- |
| addTab()                              |                                          |
| newTab()                              |                                          |
| setOnTabSelectedListener()            |                                          |
| setupWithViewPager(viewPager)         |                                          |
| setTabsFromPagerAdapter(PagerAdapter) |                                          |
| app:tabMode                           | fixed：不可滚动  scrollable：可滚动               |
| app:tabGravity                        | fill分配所有的可用空间给每个tab  center所有的 tab 在屏幕的中间 |

### 10.2.TabLayoutOnPageChangeListener
```java
ViewPager
viewPager = ...;
TabLayout tabLayout = ...;
viewPager.addOnPageChangeListener(new TabLayoutOnPageChangeListener(tabLayout));
```
### 10.3.TabLayout.Tab

setText()，setIcon()

## 11. NestedScrollView

## 12. DrawerLayout

setDrawerListener()

addDrawerListener(mToggle)

closeDrawers()

## 13. ActionBarDrawerToggle
```java
ActionBarDrawerToggle(MainActivity.this，mDrawerLayout，mToolbar，R.string.open，R.string.close)

syncState()
```
## 14. NavigationView侧拉菜单

- app:headerLayout="@layout/nav_header"头布局
- app:menu="@menu/navigation_drawer_items"菜单
- setNavigationItemSelectedListener()

## 15. FloatingActionButton

默认颜色为主题中colorAccent

## 16. Snackbar

## 17. SwipeRefreshLayout

setColorSchemeResources()

## 18. TextInputLayout
```
<android.support.design.widget.TextInputLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content">
    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Username"
/>
</android.support.design.widget.TextInputLayout>
```
## 19. BottomSheetDialog
```java
BottomSheetDialog
dialog = new BottomSheetDialog(this);
dialog.setContentView();
dialog.show();
dialog.dismiss();
```
## 20. Menu

```java
public boolean onCreateOptionsMenu(Menu menu) {

    getMenuInflater().inflate(R.menu.xx, menu);

    return true;

}
```



```java
public boolean onOptionsItemSelected(MenuItem item) {

    switch (item.getItemId()) {

        case R.id.home:

            mDrawerLayout.openDrawer(GravityCompat.START);

            return true;

    }

    return super.onOptionsItemSelected(item);

}
```

## 21. Demo
```
<?xml version="1.0" encoding="utf-8"?>
<android.support.v4.widget.DrawerLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/drawer_layout"
    android:layout_height="match_parent"
    android:layout_width="match_parent"
    android:fitsSystemWindows="true">
    <include layout="@layout/include_list_viewpager"/>
<android.support.design.widget.NavigationView
        android:id="@+id/nav_view"
        android:layout_height="match_parent"
        android:layout_width="wrap_content"
        android:layout_gravity="start"
        android:fitsSystemWindows="true"
        app:headerLayout="@layout/nav_header"
        app:menu="@menu/drawer_view"/>
</android.support.v4.widget.DrawerLayout>
```


```
<?xml version="1.0" encoding="utf-8"?>

<android.support.design.widget.CoordinatorLayout

    android:id="@+id/main_content"

    xmlns:android="http://schemas.android.com/apk/res/android"

    xmlns:app="http://schemas.android.com/apk/res-auto"

    android:layout_width="match_parent"

    android:layout_height="match_parent">



    <android.support.design.widget.AppBarLayout

        android:id="@+id/appbar"

        android:layout_width="match_parent"

        android:layout_height="wrap_content"

        android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar">



        <android.support.v7.widget.Toolbar

            android:id="@+id/toolbar"

            android:layout_width="match_parent"

            android:layout_height="?attr/actionBarSize"

            android:background="?attr/colorPrimary"

            app:layout_scrollFlags="scroll|enterAlways|snap"

            app:popupTheme="@style/ThemeOverlay.AppCompat.Light"/>



        <android.support.design.widget.TabLayout

            android:id="@+id/tabs"

            android:layout_width="match_parent"

            android:layout_height="wrap_content"/>



    </android.support.design.widget.AppBarLayout>



    <android.support.v4.view.ViewPager

        android:id="@+id/viewpager"

        android:layout_width="match_parent"

        android:layout_height="match_parent"

        app:layout_behavior="@string/appbar_scrolling_view_behavior"/>



    <android.support.design.widget.FloatingActionButton

        android:id="@+id/fab"

        android:layout_width="wrap_content"

        android:layout_height="wrap_content"

        android:layout_gravity="end|bottom"

        android:layout_margin="@dimen/fab_margin"

        android:src="@drawable/ic_done"/>



</android.support.design.widget.CoordinatorLayout>
```



```
<?xml version="1.0" encoding="utf-8"?>

<android.support.design.widget.CoordinatorLayout

    android:id="@+id/main_content"

    xmlns:android="http://schemas.android.com/apk/res/android"

    xmlns:app="http://schemas.android.com/apk/res-auto"

    android:layout_width="match_parent"

    android:layout_height="match_parent"

    android:fitsSystemWindows="true">



    <android.support.design.widget.AppBarLayout

        android:id="@+id/appbar"

        android:layout_width="match_parent"

        android:layout_height="@dimen/detail_backdrop_height"

        android:fitsSystemWindows="true"

        android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar">



        <android.support.design.widget.CollapsingToolbarLayout

            android:id="@+id/collapsing_toolbar"

            android:layout_width="match_parent"

            android:layout_height="match_parent"

            android:fitsSystemWindows="true"

            app:contentScrim="?attr/colorPrimary"

            app:expandedTitleMarginEnd="64dp"

            app:expandedTitleMarginStart="48dp"

            app:layout_scrollFlags="scroll|exitUntilCollapsed">



            <ImageView

                android:id="@+id/backdrop"

                android:layout_width="match_parent"

                android:layout_height="match_parent"

                android:fitsSystemWindows="true"

                android:scaleType="centerCrop"

                app:layout_collapseMode="parallax"/>



            <android.support.v7.widget.Toolbar

                android:id="@+id/toolbar"

                android:layout_width="match_parent"

                android:layout_height="?attr/actionBarSize"

                app:layout_collapseMode="pin"

                app:popupTheme="@style/ThemeOverlay.AppCompat.Light"/>



        </android.support.design.widget.CollapsingToolbarLayout>



    </android.support.design.widget.AppBarLayout>



    <android.support.v4.widget.NestedScrollView

        android:layout_width="match_parent"

        android:layout_height="match_parent"

        app:layout_behavior="@string/appbar_scrolling_view_behavior">

        ...

    </android.support.v4.widget.NestedScrollView>



    <android.support.design.widget.FloatingActionButton

        android:layout_width="wrap_content"

        android:layout_height="wrap_content"

        android:layout_margin="@dimen/fab_margin"

        android:clickable="true"

        android:src="@drawable/ic_discuss"

        app:layout_anchor="@id/appbar"

        app:layout_anchorGravity="bottom|right|end"/>



</android.support.design.widget.CoordinatorLayout>
```

# 5. NestedScrolling嵌套滚动机制

Touch 事件在进行分发的时候，由父View 向它的子 View 传递，一旦某个子 View 开始接收进行处理，那么接下来所有事件都将由这个 View 来进行处理，它的 ViewGroup 将不会再接收到这些事件，直到下一次手指按下。而嵌套滚动机制（NestedScrolling）就是为了弥补这一机制的不足，为了让子 View 能和父 View 同时处理一个 Touch 事件

## 5.1 NestedScrollingParent
```java
public interface
NestedScrollingParent {
    *
    *public boolean onStartNestedScroll(View child, View target, int nestedScrollAxes);
*    *public void onNestedScrollAccepted(View child, View target, int nestedScrollAxes);
     *
         *public void onStopNestedScroll(View target);
*    *public void onNestedScroll(View target, int dxConsumed, int dyConsumed,
                 int dxUnconsumed, int dyUnconsumed);
     *
         *public void onNestedPreScroll(View target, int dx, int dy, int[] consumed);
     *
         *public boolean onNestedFling(View target, float velocityX, float velocityY, boolean consumed);
     *
         *public boolean onNestedPreFling(View target, float velocityX, float velocityY);
     *
         *public int getNestedScrollAxes();
     }
```
## 2. NestedScrollingChild

NestedScrollingParent接口，对应处理的就是NestedScrollingChild传递出的数据，让控件通知CoordinatorLayout自己的状态

## 3. NestedScrollingChildHelper

| 方法声明                        | 功能描述 |
| --------------------------- | ---- |
| setNestedScrollingEnabled() |      |
| isNestedScrollingEnabled()  |      |
| startNestedScroll()         |      |
| stopNestedScroll()          |      |
| hasNestedScrollingParent()  |      |
| dispatchNestedScroll()      |      |
| dispatchNestedPreScroll()   |      |
| dispatchNestedFling()       |      |
| dispatchNestedPreFling()    |      |


```java
public class TouchBehavior extends CoordinatorLayout.Behavior
implements NestedScrollingChild {
    private NestedScrollingChildHelper
childHelper;
    private float                      ox;
    private float                      oy;
    private int[] consumed = new int[2];
    private int[] offsetInWindow = new int[2];
    public TouchBehavior(Context
context, AttributeSet attrs) {
        super(context, attrs);
    }
    @Override
    public boolean onTouchEvent(CoordinatorLayout
parent, View child, MotionEvent ev) {
if (childHelper == null) {
            childHelper = new NestedScrollingChildHelper(child);
            setNestedScrollingEnabled(true);
        }
        switch (ev.getAction())
{
            case MotionEvent.*ACTION_DOWN*:
                ox = ev.getX();
                oy = ev.getY();
                if (oy <
child.getTop() || oy > child.getBottom() || ox <
child.getLeft() || ox > child.getRight()) {
                    return true;
                }
                //开始滑动
                startNestedScroll(ViewCompat.*SCROLL_AXIS_VERTICAL*);
                break;
            case MotionEvent.*ACTION_MOVE*:
                float clampedX =
ev.getX();
                float clampedY =
ev.getY();
                int dx = (int) (clampedX - ox);
                int dy = (int) (clampedY - oy);
                if (Math.*abs*(dy)
\> Math.*abs*(dx)) {
                    //分发触屏事件给parent处理
                    dispatchNestedPreScroll(dx, -dy, consumed, offsetInWindow);
                }
                break;
            case MotionEvent.*ACTION_UP*:
                stopNestedScroll();
                break;
        }
        return true;
    }
    @Override
    public void setNestedScrollingEnabled(boolean enabled) {
        childHelper.setNestedScrollingEnabled(enabled);
    }
    @Override
    public boolean isNestedScrollingEnabled() {
        return childHelper.isNestedScrollingEnabled();
    }
    @Override
    public boolean startNestedScroll(int axes) {
        return childHelper.startNestedScroll(axes);
    }
    @Override
    public void stopNestedScroll() {
        childHelper.stopNestedScroll();
    }
    @Override
    public boolean hasNestedScrollingParent() {
        return childHelper.hasNestedScrollingParent();
    }
    @Override
    public boolean dispatchNestedScroll(int dxConsumed, int dyConsumed, int dxUnconsumed, int dyUnconsumed, int[]
offsetInWindow) {
        return childHelper.dispatchNestedScroll(dxConsumed, dyConsumed, dxUnconsumed, dyUnconsumed, offsetInWindow);
    }
    @Override
    public boolean dispatchNestedPreScroll(int dx, int dy, int[] consumed, int[]
offsetInWindow) {
        return childHelper.dispatchNestedPreScroll(dx, dy, consumed, offsetInWindow);
    }
    @Override
    public boolean dispatchNestedFling(float velocityX, float velocityY, boolean consumed) {
        return childHelper.dispatchNestedFling(velocityX, velocityY, consumed);
    }
    @Override
    public boolean dispatchNestedPreFling(float velocityX, float velocityY) {
        return childHelper.dispatchNestedPreFling(velocityX, velocityY);
    }
}
```
## 4. NestedScrollingParentHelper

## 5. 对外传递触摸事件

# 6. Behavior机制

CoordinatorLayout 的职责充当了一个第三方的角色，通知各个子View之间状态的变换，Behavior 是 CoordinatorLayout 用来和各个子 View 通信用的代理类，CoordinatorLayout只是将传过来的数据进行了一次封装，分发给了自己子布局里面所有含有Behavior的控件，交由它们进行处理。

NestedScrollingParent，NestedScrollingChild

## 1. 常用方法

| 方法声明                     | 功能描述                   |
| :----------------------- | :--------------------- |
| layoutDependsOn()        | 布局依赖，依赖哪些view          |
| onDependentViewChanged() | 所依赖的view发生变化时的回调       |
| onStartNestedScroll()    | 是否拦截滚动事件，如拦截纵向或横向的滚动事件 |
| onNestedPreScroll()      | 接收滑动数据                 |
| onNestedPreFling()       | 滑动回调                   |
| setTag()                 |                        |
| getTag()                 |                        |
| onSaveInstanceState()    |                        |
| onRestoreInstanceState() |                        |
| onDependentViewRemoved() | 依赖视图被移除时的回调            |

## 2. 观察一个view的状态变化
```java
class DependentBehavior
extends CoordinatorLayout.Behavior<View>{
    public DependentBehavior(Context
context, AttributeSet attrs) {
        super(context, attrs);
    }
    */\**
     *
     * **@param ****parent
     *** \**@param ****child  **设置**Behavior**的**View
     \* **@param ****dependency **所依赖的**View
     \* **@return
     ***/
    *@Override
    public boolean layoutDependsOn(CoordinatorLayout
parent, View child, View dependency) {
        int offset =
dependency.getTop() - child.getTop();
        ViewCompat.*offsetTopAndBottom*(child, offset);
        return true;
    }
    @Override
    public boolean onDependentViewChanged(CoordinatorLayout
parent, View child, View
            dependency) {
        return dependency instanceof TextView;
    }
}
```
## 3. 监听CoordinatorLayout里的滑动状态
```java
class ScrollBehavior extends CoordinatorLayout.Behavior<View>{
    public ScrollBehavior(Context
context, AttributeSet attrs) {
        super(context, attrs);
    }
    @Override
    public boolean onStartNestedScroll(CoordinatorLayout
coordinatorLayout, View child, View
            directTargetChild, View target, int nestedScrollAxes)
{
        return (nestedScrollAxes
& ViewCompat.*SCROLL_AXIS_VERTICAL*) != 0;
    }
    @Override
    public void onNestedPreScroll(CoordinatorLayout
coordinatorLayout, View child, View
            target, int dx, int dy, int[] consumed) {
        int leftScrolled =
target.getScrollY();
        child.setScrollY(leftScrolled);
    }
@Override
    public boolean onNestedPreFling(CoordinatorLayout
coordinatorLayout, View child, View
            target, float velocityX, float velocityY) {
        ((NestedScrollView)
child).fling((int)velocityY);
        return true;
    }
}
```


## 4. 关联Behavior

通过代码绑定Behavior

```
CoordinatorLayout.LayoutParams params = (CoordinatorLayout.LayoutParams) mRecyclerView.getLayoutParams();

params.setBehavior(new AppBarLayout.ScrollingViewBehavior());
```

在XML中绑定Behavior

app:layout_behavior="@string/appbar_scrolling_view_behavior"

AppBarLayout.ScrollingViewBehavior

app:layout_behavior=".TouchBehavior"

自动绑定Behavior

@CoordinatorLayout.DefaultBehavior(FancyFrameLayoutBehavior.class)
public class FancyFrameLayout extends FrameLayout {
}

## 5. Behavior的实例化

app:layout_behavior="@string/appbar_scrolling_view_behavior"

Android.support.design.widget.AppBarLayout$ScrollingViewBehavior

我们在view中可以通过app:layout_behavior然后指定一个字符串来表示使用哪个behavior，在CoordinatorLayout中利用反射机制来完成的behavior的实例化

```xml
<string name="appbar_scrolling_view_behavior" translatable="false">
	android.support.design.widget.AppBarLayout$ScrollingViewBehavior
</string>
```

NestedScrollingChildHelper

NestedScrollView.onInterceptTouchEvent → NestedScrollingChildHelper.onStartNestedScroll → CoordinatorLayout.onStartNestedScroll

## 6. Behavior的事件分发过程

发到CoordinatorLayout 上的触摸事件会发到Behavior 中

onInterceptTouchEvent，onTouchEvent

## 7. SwipeDismissBehavior

用来实现滑动删除
```java
SwipeDismissBehavior<View>
swipe = new SwipeDismissBehavior();
swipe.setSwipeDirection(SwipeDismissBehavior.*SWIPE_DIRECTION_ANY*);
swipe.setListener(
        new SwipeDismissBehavior.OnDismissListener()
{
            @Override public void onDismiss(View view) {
            }
            @Override
            public void onDragStateChanged(int state) {}
        });
CoordinatorLayout.LayoutParams coordinatorParams =
        (CoordinatorLayout.LayoutParams)
textView.getLayoutParams();
coordinatorParams.setBehavior(swipe);
```


## 8. BottomSheetBehavior

app:layout_behavior="android.support.design.widget.BottomSheetBehavior"


```java
BottomSheetBehavior
behavior =
BottomSheetBehavior.from(findViewById(R.id.scroll));
if(behavior.getState() == BottomSheetBehavior.STATE_EXPANDED) {
behavior.setState(BottomSheetBehavior.STATE_COLLAPSED);
}else {
behavior.setState(BottomSheetBehavior.STATE_EXPANDED);
}



behavior.setBottomSheetCallback(new BottomSheetBehavior.BottomSheetCallback()
{
    @Override
    public void onStateChanged(@NonNull View bottomSheet, int newState) {
    }
    @Override
    public void onSlide(@NonNull View bottomSheet, float slideOffset) {
    }
});
```