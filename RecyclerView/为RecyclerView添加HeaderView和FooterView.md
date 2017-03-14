## 1. 概述

RecyclerView通过其高度的可定制性深受大家的青睐，也有非常多的使用者开始对它进行封装或者改造，从而满足越来越多的需求。

如果你对RecyclerView不陌生的话，你一定遇到过这样的情况，我想给RecyclerView加个headerView或者footerView，当你敲出`.addHeaderView`，你会发现并没有添加头部或者底部View的相关API。

那么本文主要的内容很明显了，完成以下工作：

- 如何为RecyclerView添加HeaderView(支持多个)
- 如何为RecyclerView添加FooterView(支持多个)
- 如何让HeaderView或者FooterView适配各种LayoutManager

恩，其实本来我是想偷个懒的，因为Loader写过一篇类似的文章，文章见文末参考链接。但是我发现被别的公众号推送了~~

那我只能考虑自己换种思路来解决这个问题，并且提供尽可能多的功能了~

本文首发于我的公众号，欢迎扫码关注（二维码见左侧栏）。

## 2. 思路

### 2.1 原理

对于添加headerView或者footerView的思路

其实HeaderView实际上也是Item的一种，只不过显示在顶部的位置，那么我们完全可以通过为其设置ItemType来完成。

有了思路以后，我们心里就妥了，最起码我们的内心中想想是可以实现的，接下来考虑一些细节。

### 2.2 一些细节

假设我们现在已经完成了RecyclerView的编写，忽然有个需求，需要在列表上加个HeaderView，此时我们该怎么办呢？

打开我们的Adapter，然后按照我们上述的原理，添加特殊的ViewType，然后修改代码完成。

这是比较常规的做法了，但是有个问题是，如果需要添加viewType，那么可能我们的Adapter需要修改的幅度就比较大了，比如`getItemType`、`getItemCount`、`onBindViewHolder`、`onCreateViewHolder`等，几乎所有的方法都要进行改变。

这样来看，出错率是非常高的。

况且一个项目中可能多个RecyclerView都需要在其列表中添加headerView。

这么来看，直接改Adapter的代码是非常不划算的，最好能够设计一个类，可以无缝的为原有的Adapter添加headerView和footerView。

本文的思路是通过类似装饰者模式，去设计一个类，增强原有Adapter的功能，使其支持`addHeaderView`和`addFooterView`。这样我们就可以不去改动我们之前已经完成的代码，灵活的去扩展功能了。

我希望的用法是这样的：

```
mHeaderAndFooterWrapper = new HeaderAndFooterWrapper(mAdapter);
t1.setText("Header 1");
TextView t2 = new TextView(this);
mHeaderAndFooterWrapper.addHeaderView(t2);12341234
```

在不改变原有的Adapter基础上去增强其功能。

## 3. 初步的实现

### 3.1 基本代码

```java
public class HeaderAndFooterWrapper<T> extends RecyclerView.Adapter<RecyclerView.ViewHolder>
{
    private static final int BASE_ITEM_TYPE_HEADER = 100000;
    private static final int BASE_ITEM_TYPE_FOOTER = 200000;

    private SparseArrayCompat<View> mHeaderViews = new SparseArrayCompat<>();
    private SparseArrayCompat<View> mFootViews = new SparseArrayCompat<>();

    private RecyclerView.Adapter mInnerAdapter;

    public HeaderAndFooterWrapper(RecyclerView.Adapter adapter)
    {
        mInnerAdapter = adapter;
    }

    private boolean isHeaderViewPos(int position)
    {
        return position < getHeadersCount();
    }

    private boolean isFooterViewPos(int position)
    {
        return position >= getHeadersCount() + getRealItemCount();
    }


    public void addHeaderView(View view)
    {
        mHeaderViews.put(mHeaderViews.size() + BASE_ITEM_TYPE_HEADER, view);
    }

    public void addFootView(View view)
    {
        mFootViews.put(mFootViews.size() + BASE_ITEM_TYPE_FOOTER, view);
    }

    public int getHeadersCount()
    {
        return mHeaderViews.size();
    }

    public int getFootersCount()
    {
        return mFootViews.size();
    }
}
```

首先我们编写一个Adapter的子类，我们叫做HeaderAndFooterWrapper，然后再其内部添加了addHeaderView，addFooterView等一些辅助方法。

这里你可以看到，对于多个HeaderView，讲道理我们首先想到的应该是使用`List<View>`，而这里我们为什么要使用`SparseArrayCompat<View>`呢？

SparseArrayCompat有什么特点呢？它类似于Map，只不过在某些情况下比Map的性能要好，并且只能存储key为int的情况。

并且可以看到我们对每个HeaderView，都有一个特定的key与其对应，第一个headerView对应的是`BASE_ITEM_TYPE_HEADER`，第二个对应的是`BASE_ITEM_TYPE_HEADER+1`；

为什么要这么做呢？

这两个问题都需要到复写onCreateViewHolder的时候来说明。

### 3.2 复写相关方法

```java
public class HeaderAndFooterWrapper<T> extends RecyclerView.Adapter<RecyclerView.ViewHolder>
{

    @Override
    public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType)
    {
        if (mHeaderViews.get(viewType) != null)
        {

            ViewHolder holder = ViewHolder.createViewHolder(parent.getContext(), mHeaderViews.get(viewType));
            return holder;

        } else if (mFootViews.get(viewType) != null)
        {
            ViewHolder holder = ViewHolder.createViewHolder(parent.getContext(), mFootViews.get(viewType));
            return holder;
        }
        return mInnerAdapter.onCreateViewHolder(parent, viewType);
    }

    @Override
    public int getItemViewType(int position)
    {
        if (isHeaderViewPos(position))
        {
            return mHeaderViews.keyAt(position);
        } else if (isFooterViewPos(position))
        {
            return mFootViews.keyAt(position - getHeadersCount() - getRealItemCount());
        }
        return mInnerAdapter.getItemViewType(position - getHeadersCount());
    }

    private int getRealItemCount()
    {
        return mInnerAdapter.getItemCount();
    }


    @Override
    public void onBindViewHolder(RecyclerView.ViewHolder holder, int position)
    {
        if (isHeaderViewPos(position))
        {
            return;
        }
        if (isFooterViewPos(position))
        {
            return;
        }
        mInnerAdapter.onBindViewHolder(holder, position - getHeadersCount());
    }

    @Override
    public int getItemCount()
    {
        return getHeadersCount() + getFootersCount() + getRealItemCount();
    }
}
```

- getItemViewType

由于我们增加了headerView和footerView首先需要复写的就是getItemCount和getItemViewType。

getItemCount很好理解；

对于getItemType，可以看到我们的返回值是mHeaderViews.keyAt(position)，这个值其实就是我们addHeaderView时的key，footerView是一样的处理方式，这里可以看出我们为每一个headerView创建了一个itemType。

- onCreateViewHolder

可以看到，我们分别判断viewType，如果是headview或者是footerview，我们则为其单独创建ViewHolder，这里的ViewHolder是我之前写的一个通用的库里面的类，文末有链接。当然，你也可以自己写一个ViewHolder的实现类，只需要将对应的headerView作为itemView传入ViewHolder的构造即可。

这个方法中，我们就可以解答之前的问题了：

- 为什么我要用SparseArrayCompat而不是List？
- 为什么我要让每个headerView对应一个itemType，而不是固定的一个？

对于headerView假设我们有多个，那么onCreateViewHolder返回的ViewHolder中的itemView应该对应不同的headerView，如果是List，那么不同的headerView应该对应着：list.get(0),list.get(1)等。

但是问题来了，该方法并没有position参数，只有itemType参数，如果itemType还是固定的一个值，那么你是没有办法根据参数得到不同的headerView的。

所以，我利用SparseArrayCompat，将其key作为itemType，value为我们的headerView，在onCreateViewHolder中，直接通过itemType，即可获得我们的headerView，然后构造ViewHolder对象。而且我们的取值是从100000开始的，正常的itemType是从0开始取值的，所以正常情况下，是不可能发生冲突的。

> 需要说明的是，这里的意思并非是一定不能用List，通过一些特殊的处理，List也能达到上述我描述的效果。

- onBindViewHolder

onBindViewHolder比较简单，发现是HeaderView或者FooterView直接return即可，因为对于头部和底部我们仅仅做展示即可，对于事件应该是在addHeaderView等方法前设置。

这样就初步完成了我们的装饰类，我们分别添加两个headerView和footerView:

我们看看运行效果：

![img](http://img.blog.csdn.net/20160707215354713)

感觉还是不错的，再不改动原来的Adapter的基础上，我们完成了初步的实现。

大家都知道RecyclerView比较强大，可以设置不同的LayoutManager，那么我们换成GridLayoutMananger再看看效果。

![img](http://img.blog.csdn.net/20160707215502338)

好像发现了不对的地方，我们的headerView果真被当成普通的Item处理了，不过由于我们的编写方式，出现上述情况是可以理解的。

那么我们该如何处理呢？让每个headerView独立的占据一行？

## 4. 进一步的完善

好在RecyclerView里面为我们提供了一些方法。

### 4.1 针对GridLayoutManager

在我们的HeaderAndFooterWrapper中复写onAttachedToRecyclerView方法，如下：

```java
@Override
public void onAttachedToRecyclerView(RecyclerView recyclerView)
{
    innerAdapter.onAttachedToRecyclerView(recyclerView);

    RecyclerView.LayoutManager layoutManager = recyclerView.getLayoutManager();
    if (layoutManager instanceof GridLayoutManager)
    {
        final GridLayoutManager gridLayoutManager = (GridLayoutManager) layoutManager;
        final GridLayoutManager.SpanSizeLookup spanSizeLookup = gridLayoutManager.getSpanSizeLookup();

        gridLayoutManager.setSpanSizeLookup(new GridLayoutManager.SpanSizeLookup()
        {
            @Override
            public int getSpanSize(int position)
            {
               int viewType = getItemViewType(position);
              if (mHeaderViews.get(viewType) != null)
              {
                  return layoutManager.getSpanCount();
              } else if (mFootViews.get(viewType) != null)
              {
                  return layoutManager.getSpanCount();
              }
              if (oldLookup != null)
                  return oldLookup.getSpanSize(position);
              return 1;
            }
        });
        gridLayoutManager.setSpanCount(gridLayoutManager.getSpanCount());
    }
}
```

当发现layoutManager为GridLayoutManager时，通过设置SpanSizeLookup，对其getSpanSize方法，返回值设置为layoutManager.getSpanCount();

现在看一下运行效果：

![img](http://img.blog.csdn.net/20160707215652536)

哈，终于正常了。

### 4.2 对于StaggeredGridLayoutManager

在刚才的代码中我们好像没有发现StaggeredGridLayoutManager的身影，StaggeredGridLayoutManager并没有setSpanSizeLookup这样的方法，那么该如何处理呢？

依然不复杂，重写onViewAttachedToWindow方法，如下：

```java
@Override
public void onViewAttachedToWindow(RecyclerView.ViewHolder holder)
{
    mInnerAdapter.onViewAttachedToWindow(holder);
    int position = holder.getLayoutPosition();
    if (isHeaderViewPos(position) || isFooterViewPos(position))
    {
        ViewGroup.LayoutParams lp = holder.itemView.getLayoutParams();

        if (lp != null
                && lp instanceof StaggeredGridLayoutManager.LayoutParams)
        {

            StaggeredGridLayoutManager.LayoutParams p = 
                    (StaggeredGridLayoutManager.LayoutParams) lp;

            p.setFullSpan(true);
        }
    }
}
```

这样就完成了对StaggeredGridLayoutManager的处理，效果图就不贴了。

到此，我们就完成了整个HeaderAndFooterWrapper的编写，可以在不改变原Adapter代码的情况下，为其添加一个或者多个headerView或者footerView，以及完成了如何让HeaderView或者FooterView适配各种LayoutManager。

源码地址： 
[https://github.com/hongyangAndroid/baseAdapter](https://github.com/hongyangAndroid/baseAdapter)