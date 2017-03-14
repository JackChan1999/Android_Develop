# **写在前面**

起深入浅出这名字的时候我是慎重又慎重的，生怕被人骂标题党，写的什么破玩意还敢说深入浅出。所以还是请大家不要抱着太高的期望，因为没有期望就没有失望，就像陈润说的，超预期嘛。全当看小说的心情来看这系列文章了。

这篇文章分三个部分，简单跟大家讲一下 RecyclerView 的常用方法与奇葩用法；工作原理与ListView比较；源码解析；

# **1. 常用方法**
RecyclerView 与 ListView、GridView 类似，都是可以显示同一种类型 View 的集合的控件。
首先看看最简单的用法，四步走：

## **1.1 接入 build.gradle 文件中加入**

```gradle
compile 'com.android.support:recyclerview-v7:24.0.0'
```

## **1.2 创建对象**

```java
RecyclerView recyclerview = (RecyclerView)
findViewById(R.id.recyclerview);
```

## **1.3 设置显示规则**

```java
recyclerview.setLayoutManager(new LinearLayoutManager(
this, LinearLayoutManager.VERTICAL, false));
```

RecyclerView 将所有的显示规则交给一个叫 LayoutManager 的类去完成了。
LayoutManager 是一个抽象类，系统已经为我们提供了三个默认的实现类，分别是 LinearLayoutManager、 GridLayoutManager 、 StaggeredGridLayoutManager，从名字我们就能看出来了，分别是，线性显示、网格显示、瀑布流显示。当然你也可以通过继承这些类来扩展实现自己的 LayougManager。

## **1.4 设置适配器**

```java
recyclerview.setAdapter(adapter);
```

适配器，同 ListView 一样，用来设置每个item显示内容的。 通常，我们写 ListView 适配器，都是首先继承 BaseAdapter，实现四个抽象方法，创建一个静态ViewHolder ， getView() 方法中判断 convertView 是否为空，创建还是获取 viewholder 对象。

而 RecyclerView 也是类似的步骤，首先继承RecyclerView.Adapter<VH>类，实现三个抽象方法，创建一个静态的 ViewHolder。不过 RecyclerView 的 ViewHolder 创建稍微有些限制，类名就是上面继承的时候泛型中声明的类名(好像反了，应该是上面泛型中的类名应该是这个holder的类名)；并且 ViewHolder 必须继承自RecyclerView.ViewHolder类。

```java
public class DemoAdapter extends RecyclerView.Adapter<DemoAdapter.VH> {
    private List<Data> dataList;
    private Context context;

    public DemoAdapter(Context context, ArrayList<Data> datas) {
        this.dataList = datas;
        this.context = context;
    }

    @Override
    public VH onCreateViewHolder(ViewGroup parent, int viewType) {
        return new VH(View.inflate(context, android.R.layout.simple_list_item_2, null));
    }

    @Override
    public void onBindViewHolder(VH holder, int position) {
        holder.mTextView.setText(dataList.get(position).getNum());
    }

    @Override
    public int getItemCount() {
        return dataList.size();
    }

    public static class VH extends RecyclerView.ViewHolder {
        TextView mTextView;
        public VH(View itemView) {
            super(itemView);
            mTextView = (TextView) itemView.findViewById(android.R.id.text1);
        }
    }
}
```

# **2. 更多方法**

除了常用方法，当然还有不常用的。

## **2.1 瀑布流与滚动方向**

前面已经介绍过，RecyclerView实现瀑布流，可以通过一句话设置：recycler.setLayoutManager(new StaggeredGridLayoutManager(2, VERTICAL))就可以了。

其中 StaggeredGridLayoutManager 第一个参数表示列数，就好像 GridView 的列数一样，第二个参数表示方向，可以很方便的实现横向滚动或者纵向滚动。


使用 demo 可以查看：[Github 【RecyclerView简单使用】](https://github.com/kymjs/RecyclerViewDemo/blob/master/RecyclerViewDemo/sample/src/main/java/com/kymjs/sample/Demo1Activity.java)

## **2.2 添加删除 item 的动画**

同 ListView 每次修改了数据源后，都要调用 notifyDataSetChanged() 刷新每项 item 类似，只不过RecyclerView 还支持局部刷新 notifyItemInserted(index);、 notifyItemRemoved(position)、notifyItemChanged(position)。

在添加或删除了数据后，RecyclerView 还提供了一个默认的动画效果，来改变显示。同时，你也可以定制自己的动画效果：模仿 DefaultItemAnimator 或直接继承这个类，实现自己的动画效果，并调用recyclerview.setItemAnimator(new DefaultItemAnimator()); 设置上自己的动画。

使用 demo 可以查看：[Github 【RecyclerView默认动画】](https://github.com/kymjs/RecyclerViewDemo/blob/master/RecyclerViewDemo/sample/src/main/java/com/kymjs/sample/Demo3Activity1.java)

## **2.3 LayoutManager的常用方法**

| 方法                                       | 功能描述                          |
| :--------------------------------------- | :---------------------------- |
| findFirstVisibleItemPosition()           | 返回当前第一个可见 Item 的 position     |
| findFirstCompletelyVisibleItemPosition() | 返回当前第一个完全可见 Item 的 position   |
| findLastVisibleItemPosition()            | 返回当前最后一个可见 Item 的 position    |
| findLastCompletelyVisibleItemPosition()  | 返回当前最后一个完全可见 Item 的 position. |
| scrollBy()                               | 滚动到某个位置。                      |

## **2.4adapter封装**

其实很早之前写过一篇关于 RecyclerView 适配器的封装，所以这不再赘述了，传送门：RecyclerView的通用适配器
使用 demo 可以查看：[Github 【RecyclerView通用适配器演示】](https://github.com/kymjs/RecyclerViewDemo/blob/master/RecyclerViewDemo/sample/src/main/java/com/kymjs/sample/Demo4Activity.java)

# **3. 吐槽**
## **OnItemTouchListener 什么鬼？**
用习惯了 ListView 的 OnItemClickListener ，RecyclerView 你的 OnItemClickListener 呢？
Tell me where do I find, something like ListView listener ?

好吧，翻遍了 API 列表，就找到了个 OnItemTouchListener ，这特么什么鬼，我干嘛要对每个 item 监听触摸屏事件。

万万没想到，最终我还是在 Google IO 里面的介绍找到了原因。原来是 Google 的工程师分不清究竟是改给 listview 的 item 添加点击事件，还是应该给每个 item 的 view 添加点击事件，索性就不给OnItemClickListener 了，然后在 support demo 里面，你就会发现，RecyclerView 的 item 点击事件都是写在了 adapter 的 ViewHolder 里面。

当然，除了 support demo 包里面使用的在 ViewHolder 里面设置点击事件以外，我还写好了一个RecyclerView 使用的 OnItemClickListener 代码请见：RecyclerItemClickListener.java

需要一提的是，网上有很多这种类似的 ItemClickListener ,在使用的时候一定注意一个问题，就是循环引用问题。比如 listener 里面持有了一个 recyclerview, 而这个 recyclerview 在调用 setListener() 的时候又持有了一个 listener。尽管 Java 虚拟机现在可以解决这种问题了，但作为代码编写者，这种写法还是应该尽量避免的。

## **divider 跑哪了？**

在ListView中设置 divider 非常简单，只需要在 XML 文件中设置就可以了，同时还可以设置 divider 高度。

```xml
android:divider="@android:color/black"
android:dividerHeight="2dp"
```

而在RecyclerView里面，想实现这两种需求，稍微复杂一点，需要自己继承RecyclerView.ItemDecoration来实现想要实现的方法。

虽说这样写灵活多了，但是要额外写一个类去做难免麻烦，这里大家可以看我已经实现好的一个封装，包括显示纯色divider、显示图片divider、divider的上下左右的间距、宽高设置 应该可以满足基本需求了： [Divider.java](https://github.com/kymjs/CoreModule/blob/master/CoreModule/recycler/src/main/java/com/kymjs/recycler/Divider.java)
使用 demo 可以查看：[Github 【自定义 Divider 使用】](https://github.com/kymjs/RecyclerViewDemo/blob/master/RecyclerViewDemo/sample/src/main/java/com/kymjs/sample/Demo4Activity.java)

# 4. 五虎上将工作原理

借用 Google IO 视频中的一张截图，视频的完整地址可查看: [RecyclerView ins and outs - Google I/O 2016](http://v.youku.com/v_show/id_XMTU4MTQ1ODg2NA==.html?f=27314446)

![RecyclerView](http://files.colabug.com/forum/201607/10/102218e999davnmje9m4l9.png)

其实上图中并没有写完整，大 boss RecyclerView 应该有这五虎上将：

| 类名                          | 作用                             |
| :-------------------------- | :----------------------------- |
| RecyclerView.LayoutManager  | 负责Item视图的布局的显示管理               |
| RecyclerView.ItemDecoration | 给每一项Item视图添加子View,例如可以进行画分隔线之类 |
| RecyclerView.ItemAnimator   | 负责处理数据添加或者删除时候的动画效果            |
| RecyclerView.Adapter        | 为每一项Item创建视图                   |
| RecyclerView.ViewHolder     | 承载Item视图的子布局                   |

## LayoutManager工作原理

```
java.lang.Object  
   ↳ android.view.View  
        ↳ android.view.ViewGroup  
            ↳ android.support.v7.widget.RecyclerView
```

首先是 RecyclerView 继承关系，可以看到，与 ListView 不同，他是一个 ViewGroup。既然是一个 View，那么就不可少的要经历 onMeasure()、onLayout()、onDraw() 这三个方法。 实际上，RecyclerView 就是将onMeasure()、onLayout() 交给了 LayoutManager 去处理，因此如果给 RecyclerView 设置不同的 LayoutManager 就可以达到不同的显示效果，因为onMeasure()、onLayout()都不同了嘛。

## **ItemDecoration 工作原理**

ItemDecoration 是为了显示每个 item 之间分隔样式的。它的本质实际上就是一个 Drawable。当 RecyclerView 执行到 onDraw() 方法的时候，就会调用到他的 onDraw()，这时，如果你重写了这个方法，就相当于是直接在 RecyclerView 上画了一个 Drawable 表现的东西。 而最后，在他的内部还有一个叫getItemOffsets()的方法，从字面就可以理解，他是用来偏移每个 item 视图的。当我们在每个 item 视图之间强行插入绘画了一段 Drawable，那么如果再照着原本的逻辑去绘 item 视图，就会覆盖掉 Decoration 了，所以需要getItemOffsets()这个方法，让每个 item 往后面偏移一点，不要覆盖到之前画上的分隔样式了。

## **ItemAnimator**

每一个 item 在特定情况下都会执行的动画。说是特定情况，其实就是在视图发生改变，我们手动调用notifyxxxx()的时候。通常这个时候我们会要传一个下标，那么从这个标记开始一直到结束，所有 item 视图都会被执行一次这个动画。

## **Adapter工作原理**

首先是适配器，适配器的作用都是类似的，用于提供每个 item 视图，并返回给 RecyclerView 作为其子布局添加到内部。
但是，与 ListView 不同的是，ListView 的适配器是直接返回一个 View，将这个 View 加入到 ListView 内部。而 RecyclerView 是返回一个 ViewHolder 并且不是直接将这个 holder 加入到视图内部，而是加入到一个缓存区域，在视图需要的时候去缓存区域找到 holder 再间接的找到 holder 包裹的 View。

## **ViewHolder**

每个 ViewHolder 的内部是一个 View，并且 ViewHolder 必须继承自RecyclerView.ViewHolder类。 这主要是因为 RecyclerView 内部的缓存结构并不是像 ListView 那样去缓存一个 View，而是直接缓存一个 ViewHolder ，在 ViewHolder 的内部又持有了一个 View。既然是缓存一个 ViewHolder，那么当然就必须所有的 ViewHolder 都继承同一个类才能做到了。

## **缓存与复用的原理**

还是一张截图

![这里写图片描述](http://files.colabug.com/forum/201607/10/102218yp5cb54w5ccc6fs1.png)

RecyclerView 的内部维护了一个二级缓存，滑出界面的 ViewHolder 会暂时放到 cache 结构中，而从 cache 结构中移除的 ViewHolder，则会放到一个叫做 RecycledViewPool 的循环缓存池中。

顺带一说，RecycledView 的性能并不比 ListView 要好多少，它最大的优势在于其扩展性。但是有一点，在 RecycledView 内部的这个第二级缓存池 RecycledViewPool 是可以被多个 RecyclerView 共用的，这一点比起直接缓存 View 的 ListView 就要高明了很多，但也正是因为需要被多个 RecyclerView 公用，所以我们的 ViewHolder 必须继承自同一个基类(即RecyclerView.ViewHolder)。

默认的情况下，cache 缓存 2 个 holder，RecycledViewPool 缓存 5 个 holder。对于二级缓存池中的 holder 对象，会根据 viewType 进行分类，不同类型的 viewType 之间互不影响。

# **5. 源码解析**
## **onMeasure**

既然是一个 View,我们先从onMeasure()开始看。
之前我们就说了 RecyclerView 的 measure 和 layout 都是交给了 LayoutManager 去做的，来看一下为什么：

```java
if (mLayout.mAutoMeasure) {
    final int widthMode = MeasureSpec.getMode(widthSpec);
    final int heightMode = MeasureSpec.getMode(heightSpec);
    final boolean skipMeasure = widthMode == MeasureSpec.EXACTLY
            && heightMode == MeasureSpec.EXACTLY;
    mLayout.onMeasure(mRecycler, mState, widthSpec, heightSpec);
} else {
    mLayout.onMeasure(mRecycler, mState, widthSpec, heightSpec);
}
```

不论是否启用 mAutoMeasure 最终都会执行到 mLayout.onMeasure() 方法中，而这个 mLayout 就是一个 LayoutManager 对象。

我们挑选 LinearLayoutManager 来看
发现它并没有onMeasure()方法，LinearLayoutManager 直接继承自 LayoutManager，所以又回到了父类 LayoutManager 中。

```java
void defaultOnMeasure(int widthSpec, int heightSpec) {
    // calling LayoutManager here is not pretty but that API is already public and it is better
    // than creating another method since this is internal.
    final int width = LayoutManager.chooseSize(widthSpec,
            getPaddingLeft() + getPaddingRight(),
            ViewCompat.getMinimumWidth(this));
    final int height = LayoutManager.chooseSize(heightSpec,
            getPaddingTop() + getPaddingBottom(),
            ViewCompat.getMinimumHeight(this));

    setMeasuredDimension(width, height);
}
```

有一句非常奇葩的注释：在这里直接调用 LayoutManager 静态方法并不完美，因为本身就是在类内部，更好的办法调用一个单独的方法。但反正这段代码也已经公开了，你们自己看着办。。。。。。

如果这不是历史遗留问题，那肯定是临时工写的，你写的时候都意识到这问题了，你还把一大堆类都写在一个类里面，造成了 RecyclerView 一个类有一万多行代码。我猜你是为了类之间跨类调用方便一点，可是你就不能设置一个包访问权限，所有类成员方法都包内调用吗，一个类干了六个类的活，网上居然还有人说这是高内聚的表现。

接着是chooseSize()方法，很简单，直接根据测量值和模式返回了最适大小。

```java
public static int chooseSize(int spec, int desired, int min) {
    final int mode = View.MeasureSpec.getMode(spec);
    final int size = View.MeasureSpec.getSize(spec);
    switch (mode) {
        case View.MeasureSpec.EXACTLY:
            return size;
        case View.MeasureSpec.AT_MOST:
            return Math.min(size, Math.max(desired, min));
        case View.MeasureSpec.UNSPECIFIED:
        default:
            return Math.max(desired, min);
    }
}
```

紧接着是对子控件 measure ，调用了：dispatchLayoutStep2() 调用了相同的方法，子控件的 measure 在 layout 过程中讲解

## **onLayout**

然后我们来看 layout 过程. 在onLayout()方法中间接的调用到了这么一个方法：dispatchLayoutStep2()，在它之中又调用到了mLayout.onLayoutChildren(mRecycler, mState);
我们重点看这个onLayoutChildren()方法。

这个方法在 LayoutManager 中的实现是空的，那么想必是在子类中实现了吧。还是找LinearLayoutManager ，跟上面 measure 过程一样，调用了dispatchLayoutStep2() 跟进去发现这么一个方法：

fill(recycler, mLayoutState, state, false);
onLayoutChildren() 中有一个非常重要的方法：fill()
recycler，是一个全局的回收复用池，用于对每个itemview回收以及复用提供支持。稍后会详细讲这个。

```java
while ((layoutState.mInfinite || remainingSpace > 0) && layoutState.hasMore(state)) {
    layoutChunk(recycler, state, layoutState, layoutChunkResult);
    layoutState.mOffset += layoutChunkResult.mConsumed * layoutState.mLayoutDirection;

    if (layoutState.mScrollingOffset != LayoutState.SCOLLING_OFFSET_NaN) {
        layoutState.mScrollingOffset += layoutChunkResult.mConsumed;
        if (layoutState.mAvailable < 0) {
            layoutState.mScrollingOffset += layoutState.mAvailable;
        }
        recycleByLayoutState(recycler, layoutState);
    }
}
```

fill() 作用就是根据当前状态决定是应该从缓存池中取 itemview 填充 还是应该回收当前的 itemview。

其中，layoutChunk() 负责从缓存池 recycler 中取 itemview，并调用View.addView() 将获取到的 ItemView 添加到 RecyclerView 中去，并调用 itemview 自身的 layout 方法去布局 item 位置。

同时在这里，还调用了measureChildWithMargins()来测绘子控件大小以及设置显示位置。这一步，我们到下面的 draw 过程还要讲。

而这全部的添加逻辑都放在一个 while 循环里面，不停的添加 itemview 到 recyclerview 里面，直到塞满所有可见区域为止。

## **onDraw**

```java
@Override
public void onDraw(Canvas c) {
    super.onDraw(c);
    final int count = mItemDecorations.size();
    for (int i = 0; i < count; i++) {
        mItemDecorations.get(i).onDraw(c, this, mState);
    }
}
```

在 onDraw() 中，除了绘制自己以外，还多调了一个mItemDecorations 的 onDraw() 方法，这个mItemDecorations 就是前面吐槽的分隔线的集合。

之前在讲 RecyclerView 的五虎上将的时候就讲过这个 ItemDecoration。 当时我们还重写了一个方法叫getItemOffsets()目的是为了不让 itemview 挡住分隔线。那他是在哪调用的呢？

还记得 layout 时说的那个measureChildWithMargins()吗，就是在这里：

```java
public void measureChildWithMargins(View child, int widthUsed, int heightUsed) {
    final Rect insets = mRecyclerView.getItemDecorInsetsForChild(child);
    widthUsed += insets.left + insets.right;
    heightUsed += insets.top + insets.bottom;

    if (shouldMeasureChild(child, widthSpec, heightSpec, lp)) {
        child.measure(widthSpec, heightSpec);
    }
}
```

在 itemview measure 的时候，会把偏移量也计算进来，也就是说：其实 ItemDecoration 的宽高是计算在 itemview 中的，只不过 itemview 本身绘制区域没有那么大，留出来的地方正好的透明的，于是就透过 itemview 显示出了 ItemDecoration。那么就很有意思了，如果我故意在 ItemDecoration 的偏移量中写成0，那么 itemview 就会挡住 ItemDecoration，而在 itemview 的增加或删除的时候，会短暂的消失(透明)，这时候就又可以透过 itemview 看到 ItemDecoration 的样子。使用这种组合还可以做出意想不到的动画效果。

## **滚动**

前面我们已经完整的走完了 RecyclerView 的绘制流程。接下来我们再看看它在滚动的时候代码又是怎么调用的。

说到滚动，自然要看 onTouch() 方法的 MOVE 状态。

```java
case MotionEvent.ACTION_MOVE: {
    final int index = MotionEventCompat.findPointerIndex(e, mScrollPointerId);
    final int x = (int) (MotionEventCompat.getX(e, index) + 0.5f);
    final int y = (int) (MotionEventCompat.getY(e, index) + 0.5f);
    int dx = mLastTouchX - x;
    int dy = mLastTouchY - y;

    if (dispatchNestedPreScroll(dx, dy, mScrollConsumed, mScrollOffset)) {
 		...
    }

    if (mScrollState != SCROLL_STATE_DRAGGING) {
		...
        if (startScroll) {
            setScrollState(SCROLL_STATE_DRAGGING);
        }
    }

    if (mScrollState == SCROLL_STATE_DRAGGING) {
        mLastTouchX = x - mScrollOffset[0];
        mLastTouchY = y - mScrollOffset[1];

        if (scrollByInternal(
                canScrollHorizontally ? dx : 0,
                canScrollVertically ? dy : 0,
                vtev)) {
            getParent().requestDisallowInterceptTouchEvent(true);
        }
    }
} break;
```

看到这段代码的时候，特意去搜了一下，MotionEventCompat 这个类是干嘛的。 他是 v4 包里面提供的一个工具类，用于兼容低版本的触摸屏手势。平时用的时候更多的是用它来处理多点触控的情况，当成MotionEvent就可以了。

dispatchNestedPreScroll() 用于处理嵌套逻辑，例如在 ScrollView 里面放一个 RecyclerView ，如果是以前用 ListView ，还得要把高度写死，禁止 ListView 的复用和滚动逻辑，而 RecyclerView 则完全不需要更多处理，直接用就是了。而且有一个非常好的地方，如果放到 ScrollView 里面，ListView 的 ItemView 是不会复用的，而 RecyclerView 因为是全局公用一套缓存池，虽说嵌套到 ScrollView 效率会低很多，但比起 ListView 嵌套要好很多，之后讲缓存池的时候，我们继续讲。

再之后，如果在相应方向上手指move的距离达到最大值，则认为需要滚动，并设置为滚动状态(SCROLL_STATE_DRAGGING)，这个最大距离默认是 8 个像素。

接着走出 if 块，如果是滚动状态，则调用滚动方法scrollByInternal()执行相应方向的滚动。滚动的距离当然就是手指移动的距离。跟进去看，果然是调用了LinearLayoutManager.scrollBy()方法，又印证了前面【更多操作】里面讲 LayoutManager 可以滚动 RecyclerView 的方法。

以上就是滚动的逻辑了。 但是没完，就像 ListView，在手指划过以后，手指离开了屏幕，相关性一样，View 自己依旧可以自己滚动一段距离。
既然手指离开了屏幕，那就去 UP 或者 CANCEL 状态去找。

```java
case MotionEvent.ACTION_CANCEL: {
    cancelTouch();
} break;

case MotionEvent.ACTION_UP: {
    mVelocityTracker.addMovement(vtev);
    mVelocityTracker.computeCurrentVelocity(1000, mMaxFlingVelocity);
    final float yvel = canScrollVertically ?
            -VelocityTrackerCompat.getYVelocity(mVelocityTracker, mScrollPointerId) : 0;
    if (!((xvel != 0 || yvel != 0) && fling((int) xvel, (int) yvel))) {
        setScrollState(SCROLL_STATE_IDLE);
    }
    resetTouch();
} break;
```

ACTION_CANCEL 里面只有一个 cancelTouch() ,那么自然是在 UP 状态里面实现的惯性滚动。

看到了一个 mVelocityTracker 对象，大概原理也就清楚了，惯性滚动多长，肯定是跟手指移动的速度有关了。

再往下，跟进fling()方法里面看：调用了mViewFlinger.fling(velocityX, velocityY);
再进：

```java
mScroller.fling(0, 0, velocityX, velocityY,
                    Integer.MIN_VALUE, Integer.MAX_VALUE, Integer.MIN_VALUE, Integer.MAX_VALUE);
```

原来是调用了 Scroller 类的fling()方法，再仔细看一下，发现是ScrollerCompat 看名字，估计又是用来兼容旧版本的 support 包里面的 Scroller 类。关于这个Scroller类，他是一个可以用来实现平滑滚动效果的类，其实内部实现也是通过一点一点移动 view，利用了人眼的视觉暂留。

## **回收与复用**
前面讲 layout、滚动的时候，都出现了一个东西，叫 Recycler,现在我们就来看看他到底是个什么。

```java
public final class Recycler {
final ArrayList<ViewHolder> mAttachedScrap = new ArrayList<>();
private ArrayList<ViewHolder> mChangedScrap = null;

final ArrayList<ViewHolder> mCachedViews = new ArrayList<ViewHolder>();

private final List<ViewHolder>
        mUnmodifiableAttachedScrap = Collections.unmodifiableList(mAttachedScrap);

private RecycledViewPool mRecyclerPool;

private ViewCacheExtension mViewCacheExtension;
```

这么多的集合，还有什么Pool，ViewCache。看来他就是一个超大型的缓存器了。
事实上他确实就是一个超大型的缓存器，拥有三级缓存(如果算上创建的那一次，应该是四级了)，这么大的缓存系统，究竟是如何完成的？

### **第一级缓存**

就是上面的一系列 mCachedViews。如果仍依赖于 RecyclerView （比如已经滑动出可视范围，但还没有被移除掉），但已经被标记移除的 ItemView 集合会被添加到 mAttachedScrap 中。然后如果 mAttachedScrap 中不再依赖时会被加入到 mCachedViews 中。 mChangedScrap 则是存储 notifXXX 方法时需要改变的 ViewHolder 。

### **第二级缓存**

ViewCacheExtension 是一个抽象静态类，用于充当附加的缓存池，当 RecyclerView 从第一级缓存找不到需要的 View 时，将会从 ViewCacheExtension 中找。不过这个缓存是由开发者维护的，如果没有设置它，则不会启用。通常我们也不会去设置他，系统已经预先提供了两级缓存了，除非有特殊需求，比如要在调用系统的缓存池之前，返回一个特定的视图，才会用到他。

### **第三级缓存**

最强大的缓存器。之前讲了，与 ListView 直接缓存 ItemView 不同，从上面代码里我们也能看到，RecyclerView 缓存的是 ViewHolder。而 ViewHolder 里面包含了一个 View 这也就是为什么在写 Adapter 的时候 必须继承一个固定的 ViewHolder 的原因。首先来看一下 RecycledViewPool：

```java
public static class RecycledViewPool {

 // 根据 viewType 保存的被废弃的 ViewHolder 集合，以便下次使用
 private SparseArray<ArrayList<ViewHolder>> mScrap = new SparseArray<ArrayList<ViewHolder>>();
  /**
   * 从缓存池移除并返回一个 ViewHolder
   */
  public ViewHolder getRecycledView(int viewType) {
    final ArrayList<ViewHolder> scrapHeap = mScrap.get(viewType);
    if (scrapHeap != null && !scrapHeap.isEmpty()) {
      final int index = scrapHeap.size() - 1;
      final ViewHolder scrap = scrapHeap.get(index);
      scrapHeap.remove(index);
      return scrap;
    }
      return null;
    }

  public void putRecycledView(ViewHolder scrap) {
    final int viewType = scrap.getItemViewType();
    final ArrayList scrapHeap = getScrapHeapForType(viewType);
    if (mMaxScrap.get(viewType) <= scrapHeap.size()) {
      return;
    }
    scrap.resetInternal();
    scrapHeap.add(scrap);
  }

  /**
   * 根据 viewType 获取对应缓存池
   */
  private ArrayList<ViewHolder> getScrapHeapForType(int viewType) {
    ArrayList<ViewHolder> scrap = mScrap.get(viewType);
      if (scrap == null) {
        scrap = new ArrayList<>();
        mScrap.put(viewType, scrap);
          if (mMaxScrap.indexOfKey(viewType) < 0) {
            mMaxScrap.put(viewType, DEFAULT_MAX_SCRAP);
          }
      }
    return scrap;
  }
}
```

从名字来看，他是一个缓存池，实现上，是通过一个默认为 5 大小的 ArrayList 实现的。这一点，同 ListView 的 RecyclerBin 这个类一样。很奇怪为什么不用 LinkedList 来做，按理说这种不需要索引读取的缓存池，用链表是最合适的。
然后每一个 ArrayList 又都是放在一个 Map 里面的，SparseArray 这个类我们在讲性能优化的时候已经多次提到了，就是两个数组，用来替代 Map 的。

把所有的 ArrayList 放在一个 Map 里面，这也是 RecyclerView 最大的亮点，这样根据 itemType 来取不同的缓存 Holder，每一个 Holder 都有对应的缓存，而只需要为这些不同 RecyclerView 设置同一个 Pool 就可以了。
这一点我们在 Pool 的 setter 方法上可以看到注释：

```java
/**
 * Recycled view pools allow multiple RecyclerViews to share a common pool of scrap views.
 * This can be useful if you have multiple RecyclerViews with adapters that use the same
 * view types, for example if you have several data sets with the same kinds of item views
 * displayed by a {@link android.support.v4.view.ViewPager ViewPager}.
 *
 * @param pool Pool to set. If this parameter is null a new pool will be created and used.
 */
public void setRecycledViewPool(RecycledViewPool pool) {
    mRecycler.setRecycledViewPool(pool);
}
```

在类似 ViewPager 这种视图中，所有 RecyclerView 的 holder 是共存于同一个 Pool 中的。

写了这么多累死我了，就这样吧，最后发一个 demo 地址：[RecyclerViewDemo](https://github.com/kymjs/RecyclerViewDemo)
和一份内部分享的 PPT 地址：[RecyclerView PPT](https://pan.baidu.com/s/1c2jvOly)
