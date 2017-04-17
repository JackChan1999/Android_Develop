## 前言

Behavior是Android Design中推荐的布局概念，网上找了很多关于Behavior的资料，很多都是直接翻译的文档或者浅尝辄止，很多问题都没有讲明白，例如具体怎么自定义Behavior，怎么使非官方的控件也能实现这些功能等等。最近几天，我整理出了一些思路，跟大家分享下。

大家知道，Behavior的主要作用有三个，这也是我准备去分别介绍的、

1. 可以避免重写控件的触摸事件
2. 优秀的嵌套滑动机制
3. 轻松建立布局间的依赖关系

每篇文章，我都会提出几个相关的疑问，并带着大家一起去解决它，今天的疑问是以下几个、

- 如何用Behavior使我们自定义的控件能够对外传递触摸事件？
- 如何用对应的控件处理触摸事件？
- 如何绑定Behavior？

## 如何用Behavior使我们自定义的控件能够对外传递触摸事件？

这个问题之前困扰了我很长一段时间，网上的很多资料，都是讲的使用一些官方的控件（如RecyclerView，NestedScrollView等）来进行滑动或触摸事件的对外传递，难道除去这些官方的控件我们就不能对外传出触摸事件了吗?答案当然是否定的，**这关键的奥义就在于NestedScrollingChild接口！**

**使用Behavior，我们就必须使用CoordinatorLayout布局，并且我们所有希望带Behavior操作的控件都必须要是CoordinatorLayout的直接子View**
那我们看看CoordinatorLayout的源码

![img](https://segmentfault.com/img/bVB1aZ)

在最开始我们就看到，CoordinatorLayout是实现了NestedScrollingParent接口的，而通过一番查找资料发现，NestedScrollingParent接口，对应处理的就是NestedScrollingChild传递出的数据。看到这里，我们就有了一个大致思路流程了（强行带一波节奏，哈哈）：

![img](https://segmentfault.com/img/bVB1iS)

原来，CoordinatorLayout只是将传过来的数据进行了一次封装，分发给了自己子布局里面所有含有Behavior的控件，交由它们进行处理。（那么我们是不是可以自己实现NestedScrollingParent接口，然后直接处理NestedScrollingChild过来的数据，不使用CoordinatorLayout和Behavior呢？答案也是肯定的！）

说到这里，那我们先学习下NestedScrolling的使用，[推荐看这篇文章](https://segmentfault.com/a/1190000002873657)
有了以上知识，我们不妨写一个Behavior的例子，比如说我想让其他控件知道我某个控件的上滑手势（就如RecyclerView一样），那来看一下我们Behavior的代码

```java
public class TouchBehavior extends CoordinatorLayout.Behavior implements NestedScrollingChild {

    private NestedScrollingChildHelper childHelper;
    private float ox;
    private float oy;
    private int[] consumed = new int[2];
    private int[] offsetInWindow = new int[2];

    public TouchBehavior(Context context, AttributeSet attrs) {
        super(context, attrs);
    }
    @Override
    public boolean onTouchEvent(CoordinatorLayout parent, View child, MotionEvent ev) {
        if (childHelper == null) {
            childHelper = new NestedScrollingChildHelper(child);
            setNestedScrollingEnabled(true);
        }
        switch (ev.getAction()) {
            case MotionEvent.ACTION_DOWN:
                ox = ev.getX();
                oy = ev.getY();
                if (oy < child.getTop() || oy > child.getBottom() || ox < child.getLeft() || ox > child.getRight()) {
                    return true;
                }
                //开始滑动
                startNestedScroll(ViewCompat.SCROLL_AXIS_VERTICAL);
                break;
            case MotionEvent.ACTION_MOVE:

                float clampedX = ev.getX();

                float clampedY = ev.getY();

                int dx = (int) (clampedX - ox);
                int dy = (int) (clampedY - oy);

                if (Math.abs(dy) > Math.abs(dx)) {
                    //分发触屏事件给parent处理
                    dispatchNestedPreScroll(dx, -dy, consumed, offsetInWindow);
                }
                break;
            case MotionEvent.ACTION_UP:
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

如果学会了NestedScrolling的使用的话，可以发现，这个Behavior的代码很简单，排除了NestedScrollingChild部分的代码（诶？怎么是Behavior实现了NestedScrollingChild接口？其实这样的写跟View实现NestedScrollingChild接口是一样的啦），就只有一个onTouchEvent事件处理了，那我们接下来就仔细分析下里面的代码，首先看看这一段

```java
if (childHelper == null) {
    childHelper = new NestedScrollingChildHelper(child);
    setNestedScrollingEnabled(true);
}
```

在第一次接收到onTouch事件的时候，我们实例化了NestedScrollingChildHelper这个类，里面的参数是View类型，**而这个View，就是我们想要告诉外界，滑动数据是由哪个View发出来的的那个**，这里我们将Behavior绑定在了我们的控件上，所以child就是这个参数。

而setNestedScrollingEnabled这方法，则是NestedScrolling的开关，假如你想对外发出你的滑动触摸事件，那么就设置为true。（当然，有时候你是不用调用这句话的，比如说使用布局依赖的时候，这是后话了）

## 如何用对应的控件处理触摸事件？

然而有一点我们需要注意到的是：**onTouch事件是CoordinatorLayout分发下来的，所以这里的onTouchEvent并不是我们控件自己的onTouch事件**，也就是说，你假如手指不在我们的控件上滑动，也会触发onTouchEvent。
于是你就看到了我下面写的代码：

```
ox = ev.getX();
oy = ev.getY();
if (oy < child.getTop() || oy > child.getBottom() || ox < child.getLeft() || ox > child.getRight()) {
    return true;
}

```

对手势的位置进行过滤，不是我们控件范围内的，舍弃掉后面MOVE事件里面做的事情也很简单，大家应该也看的懂，就是将滑动的数据传递出去，我就不细细解释了

## 如何绑定Behavior？

我推荐的方法是用xml，就像这样

```xml
<View
    android:id="@+id/rel_body"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:layout_marginTop="200dp"
    android:background="@color/colorPrimaryDark"
    app:layout_behavior=".TouchBehavior" />
```

用app:layout_behavior这个属性，“.TouchBehavior”是缩略的写法，CoordinatorLayout会帮你补全的，你也可以写上完整路径使用xml的话，就要在Behavior里面写上这个构造函数

```java
public TouchBehavior(Context context, AttributeSet attrs) {
    super(context, attrs);
}
```

还有一个方法是自动绑定，限于是自定义的View，假如你是一个自定义的控件，则可以这样写

```java
@CoordinatorLayout.DefaultBehavior(TouchBehavior.class)
public class TouchFrameLayout extends FrameLayout {
}
```

我还在网上见过一种绑定的方式，是用代码绑定的

```java
TouchBehavior touchBehavior = new TouchBehavior();
CoordinatorLayout.LayoutParams params =(CoordinatorLayout.LayoutParams) yourView.getLayoutParams();
params.setBehavior(touchBehavior);
```

这种方法我尝试过发现，只有绑定一个Behavior的时候是好用的，但假如有多个Behavior的话，前面绑定的会被后面的取消掉，所以我在这里就不推荐了

好了，第一篇文章我就讲到这里啦，或许你还不知道写这样一个东西到底该怎么用，不急，我会在后面的文章中告诉你、
[sidhu眼中的CoordinatorLayout.Behavior（二）](https://segmentfault.com/a/1190000006665225)