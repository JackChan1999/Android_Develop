## 前言

看过前两篇文章的同学
[sidhu眼中的CoordinatorLayout.Behavior（一）](https://segmentfault.com/a/1190000006657044)
[sidhu眼中的CoordinatorLayout.Behavior（二）](https://segmentfault.com/a/1190000006665225)
应该知道今天要讲的内容了——Behavior的布局依赖
其实这个内容挺少的，我都想直接贴代码然大家自己体会了……额，开玩笑的，不过内容真的少，我也不浪费大家时间了，疑问我不提了，直入主题

## 主题

（有木有直入主题，哈哈~）
我将上次的例子做了下修改
xml：

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

    <View
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:layout_marginTop="250dp"
        app:layout_behavior=".MoveWithHeadBehavior"
        android:background="@color/colorAccent" />

</android.support.design.widget.CoordinatorLayout>
```

相比于之前的布局我们可以看到，就多了一个小方块在布局里面，至于我想实现的效果可以看下面效果图

![img](https://segmentfault.com/img/bVB7Ej)

让小方块可以随着上面的head做同步的位移就如我上篇所说的，使用原理还是实现NestedScrollingChild接口，废话不多说，上代码（没错，我就是这样的人，一言不合就上代码）

```java
package com.mintmedical.mybehaviordemo;

import android.animation.ValueAnimator;
import android.content.Context;
import android.support.design.widget.CoordinatorLayout;
import android.support.v4.view.NestedScrollingChild;
import android.support.v4.view.NestedScrollingChildHelper;
import android.util.AttributeSet;
import android.view.View;

/**
 * Created by SidHu on 2016/8/17.
 */
public class HideHeadBehavior extends CoordinatorLayout.Behavior implements NestedScrollingChild {

    private boolean isHeadHide = false;
    private boolean isAnimating = false;
    private final int SCROOL_VALUE = 50;
    private int childHeight;
    private final int animationDuration = 500;

    private NestedScrollingChildHelper childHelper;

    public HideHeadBehavior(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    public boolean onStartNestedScroll(CoordinatorLayout coordinatorLayout, View child, View directTargetChild, View target, int nestedScrollAxes) {
        if (target.getId() == R.id.rel_body) {
            if (childHeight == 0) {
                childHeight = child.getHeight();
            }
            if (childHelper == null) {
                childHelper = new NestedScrollingChildHelper(child);
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
    public boolean dispatchNestedScroll(int dxConsumed, int dyConsumed, int dxUnconsumed, int dyUnconsumed, int[] offsetInWindow) {
        return childHelper.dispatchNestedScroll(dxConsumed, dyConsumed, dxUnconsumed, dyUnconsumed, offsetInWindow);
    }

    @Override
    public boolean dispatchNestedPreScroll(int dx, int dy, int[] consumed, int[] offsetInWindow) {
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

我将head实现了NestedScrollingChild接口，然后就没有做其他事情了。（这也说明了，想让控件通知CoordinatorLayout自己的状态其实只要实现了NestedScrollingChild接口就够了，假如你不需要关心滑动手势，就像小方块只关心head的位移一样，那你startNestedScroll之类的这样方法都不用要了）

那我们看一下小方块的Behavior

```java
package com.mintmedical.mybehaviordemo;

import android.content.Context;
import android.support.design.widget.CoordinatorLayout;
import android.util.AttributeSet;
import android.view.View;

/**
 * Created by SidHu on 2016/8/18.
 */
public class MoveWithHeadBehavior extends CoordinatorLayout.Behavior{

    private int lastBottom = -1;

    public MoveWithHeadBehavior(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    public boolean layoutDependsOn(CoordinatorLayout parent, View child, View dependency) {
        return dependency.getId() == R.id.rel_head;
    }

    @Override
    public boolean onDependentViewChanged(CoordinatorLayout parent, View child, View dependency) {
        if (lastBottom == -1) {
            lastBottom = dependency.getBottom();
        }
        if (dependency.getBottom() != lastBottom) {
            int d = dependency.getBottom()-lastBottom;
            lastBottom = dependency.getBottom();
            child.offsetTopAndBottom(d);
        }
        return super.onDependentViewChanged(parent, child, dependency);
    }
}
```

代码也是非常简单，布局依赖最主要的关系这两个方法，一个是判断是不是自己关心的target View（跟滑动的时候简直一毛一样），一个是被关心的target View变化以后的回调，代码我就不解释啦，也是很简单（大家有什么问题可以在评论里面问我啊）。

好了，关于Behavior的使用我就先介绍到这了