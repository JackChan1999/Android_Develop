## 前言

在上一节[sidhu眼中的CoordinatorLayout.Behavior（一）](https://segmentfault.com/a/1190000006657044)中，我们讲解了如何以通过Behavior来重写某个控件的触摸事件
可是我们只讲了如何将触摸事件抛出来，那怎么对这些数据进行处理呢？这就是我们今天要讲的内容了，**Behavior的嵌套滑动机制**

首先我们来理解一下什么是嵌套滑动，了解Android Design的大家想必已经非常熟悉这种交互了

![img](https://segmentfault.com/img/bVB7FB)

在一个控件滑动的同时，动态调整自身或其他控件的宽高或位置来达到更好的交互。
其实说白了，这个就是NestedScrollingParent和NestedScrollingChild的实际运用（不清楚的同学可以看这篇文章，[Android 嵌套滑动机制（NestedScrolling）](https://segmentfault.com/a/1190000002873657)）。仔细思考发现，实现嵌套滑动的关键，其实就是将自身的滑动事件告诉其他控件。现在大家知道我写第一篇文章的那个Behavior的用途了吧，哈哈

那我们就开始讲解今天的内容，同样，在文章的开始，我们先提出几个疑问：

- A发出消息，E也发出消息，怎样判断哪个是我们想要处理的滑动事件？
- A使B变化的同时，A能否也能一起变化？
- A使B发生改变的同时，B是否能发出消息，使C根据自己变化？

## A发出消息，E也发出消息，怎样判断哪个是我们想要处理的滑动事件？

我们再来看一下之前提过的Behavior的处理逻辑图
![img](https://segmentfault.com/img/bVB1iS)
当收到触摸事件后，CoordinatorLayout通知了所有的Behavior，换句话讲，就是Behavior会收到自己不需要处理的滑动事件。

我们来重写一下Behavior的onStartNestedScroll方法，会发现里面有个叫做target的View参数传进来，而这个target就是发出这个滑动事件的View。其对应的，就是当初写NestedScrollingChildHelper时，传入的那个View。而onStartNestedScroll方法是在target调用了startNestedScroll的时候才被调用的，也就是滑动事件发出的起始时间点，所以此时用这个taget来判断是否是我们所关心的那个view发出的最适合不过了。至于是用id（getId）还是类型（instanceof）来判断就看你自己喜欢了

在onStartNestedScroll这个方法里，return true则对后续的操作进行处理，return false则忽略掉，后续方法不会被调用。

## A使B变化的同时，A能否也能一起变化？

这个答案当然也是肯定的啦，你连target对象都拿到了，还有什么不能做，哈哈
说的多不如实战练一下。那我们接着昨天的写，网上很多都是根据滑动位移移到对应距离的例子，那我们不妨写一个其他的例子：根据手势的上滑下滑来用动画效果隐显头部。
大致效果是这样的

![img](https://segmentfault.com/img/bVB71Z)

蓝色部分用的是我们上一篇文章写的Behavior，而红色部分则用我们今天写的Behavior。

先看一下xml文件

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <View
        android:id="@+id/rel_head"
        android:layout_width="match_parent"
        android:layout_height="200dp"
        android:background="@color/colorAccent"
        app:layout_behavior=".HideHeadBehavior" />


    <View
        android:id="@+id/rel_body"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_marginTop="200dp"
        android:background="@color/colorPrimaryDark"
        app:layout_behavior=".TouchBehavior" />

</android.support.design.widget.CoordinatorLayout>
```

布局很简单，上面有个200dp高的红色部分，红色下面是蓝色部分

Activity更简单，因为它什么都不用做

```java
public class TouchTestActivity extends AppCompatActivity {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_touch_text);
    }

}
```

那重点是我们的Behavior了

```java
package com.mintmedical.mybehaviordemo;

import android.animation.ValueAnimator;
import android.content.Context;
import android.support.design.widget.CoordinatorLayout;
import android.util.AttributeSet;
import android.view.View;

public class HideHeadBehavior extends CoordinatorLayout.Behavior {

    private boolean isHeadHide = false;
    private boolean isAnimating = false;
    private final int SCROOL_VALUE = 50;
    private int childHeight;
    private final int animationDuration = 500;


    public HideHeadBehavior(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    public boolean onStartNestedScroll(CoordinatorLayout coordinatorLayout, View child, View directTargetChild, View target, int nestedScrollAxes) {
        if (target.getId() == R.id.rel_body) {
            if (childHeight == 0) {
                childHeight = child.getHeight();
            }
            return true;
        } else {
            return false;
        }
    }

    @Override
    public void onNestedPreScroll(CoordinatorLayout coordinatorLayout, View child, View target, int dx, int dy, int[] consumed) {
        super.onNestedPreScroll(coordinatorLayout, child, target, dx, dy, consumed);
        if (isAnimating) {
            return;
        }
        if (dy > SCROOL_VALUE && !isHeadHide) {
            hide(child, target);
        } else if (dy < -SCROOL_VALUE && isHeadHide) {
            show(child, target);
        }
    }

    public void hide(final View child, final View target) {
        isHeadHide = true;
        ValueAnimator valueAnimator = new ValueAnimator();
        valueAnimator.setIntValues(0, childHeight);
        valueAnimator.setDuration(animationDuration);
        valueAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                if (child.getBottom() > 0) {
                    int value = (int) animation.getAnimatedValue();
                    isAnimating = value != childHeight;
                    child.layout(child.getLeft(), -value, child.getRight(), -value + childHeight);
                    target.layout(target.getLeft(), -value + childHeight, target.getRight(), target.getBottom());
                }
            }
        });
        valueAnimator.start();
    }

    public void show(final View child, final View target) {
        isHeadHide = false;
        ValueAnimator valueAnimator = new ValueAnimator();
        valueAnimator.setIntValues(0, childHeight);
        valueAnimator.setDuration(animationDuration);
        valueAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                if (child.getBottom() < childHeight) {
                    int value = (int) animation.getAnimatedValue();
                    isAnimating = value != childHeight;
                    child.layout(child.getLeft(), value - childHeight, child.getRight(), value);
                    target.layout(target.getLeft(), value, target.getRight(), target.getBottom());
                }
            }
        });
        valueAnimator.start();
    }

}
```

代码简直非常简单，对嘛，写demo就是要简单，有时候看别人写的demo，无关业务一大堆，代码老长，看了半天才找到那句对我有用的代码。这个类里，排除两个值动画，只有两个方法

```java
@Override
public boolean onStartNestedScroll(CoordinatorLayout coordinatorLayout, View child, View directTargetChild, View target, int nestedScrollAxes) {
    if (target.getId() == R.id.rel_body) {
        if (childHeight == 0) {
            childHeight = child.getHeight();
        }
        return true;
    } else {
        return false;
    }
}
```

判断是否为我们关心的target对象，是返回true，否返回false，顺便获取了下child（也就是红色部分）的高度。

```java
@Override
public void onNestedPreScroll(CoordinatorLayout coordinatorLayout, View child, View target, int dx, int dy, int[] consumed) {
    super.onNestedPreScroll(coordinatorLayout, child, target, dx, dy, consumed);
    if (isAnimating) {
        return;
    }
    if (dy > SCROOL_VALUE && !isHeadHide) {
        hide(child, target);
    } else if (dy < -SCROOL_VALUE && isHeadHide) {
        show(child, target);
    }
}
```

接收到滑动数据后，假如动画结束，对滑动的值进行一个阈值和方向的判断，然后调用对应动画。
而在值动画里面，我们就仅仅不停改变Head和Body的Layout位置，实现动画效果
整个类就解释完了！快去试试在手机上的运行效果吧

## A使B发生改变的同时，B是否能发出消息，使C根据自己变化？

就得嘛得！不是还有一个问题嘛
这个问题的回答当然也是肯定的啦，其实聪明的你或许已经猜想到应该 怎么实现了，没错，就是根据第一篇文章依葫芦画瓢嘛。哈哈，赶紧试一试吧、不过到时候我写的会有点稍微不一样，因为下一篇要讲的是：Behavior的布局依赖
[sidhu眼中的CoordinatorLayout.Behavior（三）](https://segmentfault.com/a/1190000006666005)