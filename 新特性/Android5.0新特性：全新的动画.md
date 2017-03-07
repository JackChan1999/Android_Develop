在Material Design设计中，为用户与app交互反馈他们的动作行为和提供了视觉上的连贯性。Material主题为控件和Activity的过渡提供了一些默认的动画，在android  L上，允许自定义这些动画：

- Touch feedback  触摸反馈
- Circular Reveal  圆形展示
- Curved motion       曲线运动
- View state changes  视图状态变化
- Vector Drawables  矢量图动画
- Activity transitions  活动转场

# **1. 触摸反馈**
触摸反馈是指用户在触摸控件时的一种可视化交互，在Android L之前，通常是通过press色变来凸显，但是因为是瞬间变化的效果，不如动画生动。

在Android L使用了RippleDrawable类，用一个水波纹扩散效果在两种不同的状态间过渡。

使用Material Design样式的应用，button默认带有该效果。除了默认的效果外，系统还提供了另外两种效果，我们只把button的背景指定为：

- ?android:attr/selectableItemBackground` 用于有界Ripple动画
- ?android:attr/selectableItemBackgroundBorderless`用于越出视图边界的动画。它会被绘制在最近的且不是全屏的父视图上

> Note：selectableItemBackgroundBorderless 是 API level 21 新加入的属性

任何view处于可点击状态，都可以使用RippleDrawable来达到水波纹特效。

我们也可以通过设置RippleDrawable的颜色属性来调节动画颜色，系统默认的颜色为主题的一个属性颜色：android:colorControlHighlight，所以我们可以通过修改该颜色值来统一修改默认的水波纹颜色。android:colorAccent可以修改checkbox的选中颜色，更多颜色设置请参考主题。

系统的三种触摸反馈都是通过xml构建的，内容如下：
默认：

```xml
<ripplexmlns:android="http://schemas.android.com/apk/res/android"
        android:color="?android:attr/colorControlHighlight">
    <item>
        <inset
            android:insetLeft="4dp"
            android:insetTop="6dp"
            android:insetRight="4dp"
            android:insetBottom="6dp">
            <shape android:shape="rectangle">
                <corners android:radius="2dp"/>
                <solid android:color="?android:attr/colorButtonNormal"/>
                <padding android:left="8dp"
                         android:top="4dp"
                         android:right="8dp"
                         android:bottom="4dp"/>
            </shape>
        </inset>
    </item>
</ripple>
```

?android:attr/selectableItemBackground

```xml
<ripplexmlns:android="http://schemas.android.com/apk/res/android"
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
```
代码设置

```java
RippleDrawableColorStateList stateList      = getResources().getColorStateList(R.color.tint_state_color);
RippleDrawable  rippleDrawable = new RippleDrawable(stateList,null,null);
view.setBackground(rippleDrawable);
```

# **2. 圆形展示**

我们通常会显示或者隐藏一个view，在Android L之前，这是一个生硬瞬间变化动作，现在，有了一个新的api为此效果提供一个圆形的显示或者隐藏的动画效果。

RevealAnimator和之前的动画使用没什么区别，同样可以设置监听器和加速器来实现各种各样的特效，该动画主要用在隐藏或者显示一个view，改变view的大小等过渡效果。

通过`ViewAnimationUtils.createCircularReveal`来创建一个动画，该api接受5个参数

- view ：操作的视图
- centerX： 动画开始的中心点X
- centerY ：动画开始的中心点Y
- startRadius： 动画开始半径
- startRadius ：动画结束半径

沿着中心的缩小的动画

```java
Animatoranimator = ViewAnimationUtils.createCircularReveal(
        view, //操作的视图
        view.getWidth() / 2,//动画开始的中心点X
        view.getHeight() / 2,//动画开始的中心点Y
        view.getWidth(),//动画开始半径
        0 //动画结束半径
);
animator.setInterpolator(newLinearInterpolator());
animator.setDuration(1000);
animator.start();
```
从左上角扩展的圆形动画

```java
Animatoranimator = ViewAnimationUtils.createCircularReveal(
        view,0,0,0,(float) Math.hypot(view.getWidth(), view.getHeight()));
animator.setDuration(1000);
animator.start();
```

# **3. 曲线运动**
曲线动画在Android L之前我们可以通过继承位移动画重载applyTransformation函数来实现运动轨迹算法，但是操作起来比较繁琐：

 通过继承位移动画，来改写applyTransformation来修改位移的轨迹       

```java
public classArcTranslateAnimationextendsAnimation {
    private float mFromXValue,mToXValue,mFromYValue,mToYValue;
    private float mFromXDelta,mToXDelta,mFromYDelta,mToYDelta;
    private PointF mStart,mControl,mEnd;
    public ArcTranslateAnimation(floatfromXValue, floattoXValue, floatfromYValue, floattoYValue) {
        mFromXValue = fromXValue;
        mToXValue = toXValue;
        mFromYValue = fromYValue;
        mToYValue = toYValue;
    }
    protected void applyTransformation(floatinterpolatedTime,Transformationt) {
        float dx =calcBezier(interpolatedTime,mStart.x,mControl.x,mEnd.x);
        float dy =calcBezier(interpolatedTime,mStart.y,mControl.y,mEnd.y);
        t.getMatrix().setTranslate(dx,dy);
    }
    public void initialize(intwidth, intheight, intparentWidth, intparentHeight) {
        super.initialize(width,height,parentWidth,parentHeight);
        mFromXDelta = resolveSize(ABSOLUTE,mFromXValue,width,parentWidth);
        mToXDelta = resolveSize(ABSOLUTE,mToXValue,width, parentWidth);
        mFromYDelta = resolveSize(ABSOLUTE,mFromYValue,height,parentHeight);
        mToYDelta = resolveSize(ABSOLUTE,mToYValue,height, parentHeight);
        mStart =newPointF(mFromXDelta,mFromYDelta);
        mEnd =newPointF(mToXDelta,mToYDelta);
        mControl =newPointF(mFromXDelta,mToYDelta);
    }
    private long calcBezier(floatinterpolatedTime, floatp0, floatp1, float p2) {
        return Math.round((Math.pow((1- interpolatedTime),2) * p0)
                + (2 * (1-interpolatedTime) * interpolatedTime * p1) +
                (Math.pow(interpolatedTime,2) * p2);
    }
}
```
现在我们有了更简单的实现方式。

ObjectAnimator新增了path方式来构建动画，并且可以同时对x，y两个属性做动画，我们只用指定一个曲线的path，即可作出曲线的动画，可以用quadTo/cubicTo绘制贝塞尔曲线，也可以使用arcTo绘制普通的弧线

新增了PathInterpolator动画插入器，新的基于贝塞尔曲线或路径对象的插入器。这个插入器指定了一个`1x1`正方形运动曲线，它使用(0,0)为锚点，(1,1)为控制点，作为构造函数的参数。
视图状态变化

Android L在原有的图片选择器和颜色选择器上进行了增强，不仅是控件能根据不同的状态显示不同的背景图片，还能在两种状态切换时指定一个动画，来增加过渡效果，吸引用户眼球，以突出重点内容。

StateListAnimator类和图片选择器，颜色选择器类似，可以根据view的状态改变呈现不同的动画效果，通过xml我们可以构建对应不同状态的动画合集，其使用方式也非常简单，在对应的状态指定一个属性动画即可：
```xml
<selectorxmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_pressed="true">
        <set>
            <objectAnimator android:propertyName="translationZ"
                            android:duration="200"
                            android:valueTo="20dp"
                            android:valueType="floatType"/>
        </set>
    </item>
    <item android:state_enabled="true"android:state_pressed="false">
        <set>
            <objectAnimator android:propertyName="translationZ"
                            android:duration="200"
                            android:valueTo="0"
                            android:valueType="floatType"/>
        </set>
    </item>
</selector>
```
在xml中通过`android:stateListAnimator`来指定状态动画，代码中可以通过AnimationInflater.loadStateListAnimator()加载动画，并使用view.setStateListAnimator()将其指定给View。

可以在状态切换的过程指定多个属性动画的合集，继承了Material主题后，按钮默认拥有了z属性动画。如果想取消这种默认状态，可以把状态动画指定为null。

除了StateListAnimator类指定状态切换的属性动画外，还可以通过AnimatedStateListDrawable来指定状态切换的帧动画：
```xml
<animated-selectorxmlns:android="http://schemas.android.com/apk/res/android">
    <item android:id="@+id/pressed"android:drawable="@drawable/btn_check_15"
          android:state_pressed="true"/>
    <item android:id="@+id/normal" android:drawable="@drawable/btn_check_0"/>
    <transition android:fromId="@+id/normal"android:toId="@+id/pressed">
        <animation-list>
            <item android:duration="20"android:drawable="@drawable/btn_check_0"/>
            <item android:duration="20"android:drawable="@drawable/btn_check_1"/>
            <item android:duration="20"android:drawable="@drawable/btn_check_2"/>
        </animation-list>
    </transition>
</animated-selector>
```

# **4. 矢量图动画**

前面我们学习了矢量图，AnimatedVectorDrawable类让你能使一个矢量图动起来。矢量图动画比帧动画更平滑的展现图片的变化过程，并且无论在内存占用，还是包体积占用上都要优于帧动画。通常定义一个矢量图动画需要三步：

在drawable资源目录下定义一个矢量图

```xml
<vectorxmlns:android="http://schemas.android.com/apk/res/android"
        android:height="64dp"
        android:width="64dp"
        android:viewportHeight="600"
        android:viewportWidth="600">
    <group
        android:name="rotationGroup"
        android:pivotX="300.0"
        android:pivotY="300.0"
        android:rotation="45.0">
        <path
            android:name="v"
            android:fillColor="#000000"
            android:pathData="M300,70 l0,-70 70,70 0,0 -70,70z"/>
    </group>
</vector>
```
在anim下顶一个objectAnimator，并在动画中修改矢量图的path

```xml
<setxmlns:android="http://schemas.android.com/apk/res/android">
    <objectAnimator
        android:duration="3000"
        android:propertyName="pathData"
        android:valueFrom="M300,70 l0,-70 70,70 0,0 -70,70z"
        android:valueTo="M300,70 l0,-70 70,0  0,140 -70,0z"
        android:valueType="pathType"/>
</set>
```
在drawable下定义一个animated-vector，并把drawable指向矢量图，把target中的动画指定为之前定义的objectAnimator

```xml
<animated-vectorxmlns:android="http://schemas.android.com/apk/res/android"
                 android:drawable="@drawable/vector_drawable">
    <target
        android:name="v"
        android:animation="@anim/vector_anim"/>
</animated-vector>
```

# **5. Transition转场动画**
![Transition](http://hukai.me/android-training-course-in-chinese/material/ContactsAnim.gif)

Android KitKat 4.4.2 (API 19) 开始的 Transitions 功能让开发者无需学习动画就可以轻松实现 layout 切换时的动画效果，为用户带来更丰富的体验。现在 Android Support Library 24.2.0 (新增的 android.support.transition 包)又将这一功能带到了所有 Android 4.0 ICS(API 14) 及以上系统

在Android L之前，我们可以在startActivity之后调用overridePendingTransition来指定Activity的转场动画。现在Android L给我们带来了更绚丽的转场动画。

新的转场动画分为两大类，一种是普通的过渡动画，另一种是共享元素的过渡动画。

要想使用新的转场动画，可以继承Material Design主题后在style风格中指定：

```xml
<stylename="DefaultTheme"parent="android:Theme.Material">
    <!-- 允许使用transitions -->
    <item name="android:windowContentTransitions">true</item>
    <!-- 指定进入、退出、返回、重新进入时的transitions -->
    <item name="android:windowEnterTransition">@transition/explode</item>
    <item name="android:windowExitTransition">@transition/explode</item>
    <item name="android:windowReturnTransition">@transition/explode</item>
    <item name="android:windowReenterTransition">@transition/explode</item>
    <!-- 指定进入、退出、返回、重新进入时的共享transitions -->
    <item name="android:windowSharedElementEnterTransition">@transition/change</item>
    <item name="android:windowSharedElementExitTransition">@transition/change</item>
    <item name="android:windowSharedElementReturnTransition">@transition/change</item>
    <item name="android:windowSharedElementReenterTransition">@transition/change</item>
</style>
```
也可以在activity的oncreate方法中进行代码设置：

```java
// 允许使用transitions
getWindow().requestFeature(Window.FEATURE_CONTENT_TRANSITIONS);
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
```

## **5.1 普通转场动画**

所有继承自visibility类都可以作为进入、退出的过度动画。如果我们想自定义进入和退出时的动画效果，只需要继承Visibility，重载onAppear()和onDisappear()方法来定义进入喝退出的动画。系统提供了三种默认方式：

| Transition | 说明                |
| :--------- | :---------------- |
| Explode    | 爆炸效果，从屏幕中心移入或移出视图 |
| Slide      | 滑动效果，从屏幕边缘移入或移出视图 |
| Fade       | 谈入谈出效果，改变视图的透明度   |

想在xml中指定自定义的进入、退出的过度动画需要先对动画进行定义：

```xml
<transitionclass="my.app.transition.CustomTransition"/>
```
注意：其中CustomTransition是你自定义的动画，它必须继承自Visibility。

想以普通转场动画的方式启动一个Activity，必须在startActivity函数中传递一个ActivityOptions的Bundle对象：

```java
ActivityOptions options = ActivityOptions.makeSceneTransitionAnimation(activity);
startActivity(intent,options.toBundle());
```
如果想让返回也具备转场效果，那么在返回的Activity中不要再调用finish()函数，而是应该使用finishAfterTransition()来结束一个Activity，该函数会等待动画执行完毕才结束该Activity

要尽早开始入场切换，可以在被调用的Activity上使用Window.setAllowEnterTransitionOverlap() ，它可以使你拥有更戏剧性的入场切换

## **5.2 共享转场动画**

当两个Activity具备某些相遇的元素时，共享转场动画将是一个非常好的选择。使用转场动画需要将相同的元素通过android:transitionName或者view.setTransitionName()设置为相同的名称，这样系统才能区分出相同的元素。

共享转场动画支持以下共享元素：
| Transition           | 说明                |
| :------------------- | :---------------- |
| ChangeBounds         | 对目标视图的大小进行动画      |
| ChangeClipBounds     | 对目标视图的剪裁大小进行动画    |
| ChangeTransform      | 对目标视图进行缩放、旋转、位移动画 |
| ChangeImageTransform | 对目标图片进行缩放         |

通过下面的函数启动一个共享元素动画：

```java
ActivityOptions options = ActivityOptions.makeSceneTransitionAnimation(activity,view, "name");
startActivity(intent,options.toBundle());
```
 如果有多个共享元素，则可以通过Pair进行包装处理：

```java
ActivityOptions options = ActivityOptions.makeSceneTransitionAnimation(activity,Pair.create(view1,"name1"),Pair.create(view2,"name2"));
startActivity(intent,.toBundle());
```
返回时如果需要具备转场动画，那么也需要用finish()函数替代finishAfterTransition()来结束一个Activity。

共享转场动画通常可以根据指定的元素判断出合适的转场动画效果，不需要我们做额外的处理，也可以通过之前学习的方法进行指定共享元素转场动画效果。
## **5.3 组合转场动画**
我们可以把多个转场动画进行组合，作出更具个性的转场效果，在资源文件中通过以下方式：

```java
<transitionSetxmlns:android="http://schemas.android.com/apk/res/android">
    <explode/>
    <transition class="my.app.transition.CustomTransition"/>
    <<changeImageTransform/>
</transitionSet>
```
代码中我们可以通过`TransitionSet`类组合多个转场动画：

```java
TransitionSet transitionSet = new TransitionSet();
transitionSet.addTransition(new Fade());
transitionSet.addTransition(newChangeBounds());
```

组合可以同时针对普通转场动画和共享元素转场动画。

转场动画也可以像普通动画一样设置持续时间，延期执行时间，速率插入器，以及动画的监听等。

转场动画通常是对整个布局起作用，如果我们想对某个特定的view实施转场动画，可以把该view设置为转场动画的target，这样转场动画将只对特定的view起作用。共享元素的动画的target需要指定为transitionName