上一篇文章我讲到[用事件分发的原理结合SwipeRefreshLayout写一个RecyclerView的上下拉](http://www.jianshu.com/p/4ef91430009c),里面有一个判断RecyclerView是否到达底部的方法isBottom。我的同事用了这个上下拉之后发现有些小bug，没考虑周全，譬如各个子项高度不统一的时候，然后我找到原因是因为这个判断上下拉的问题。所以，我就去网上查到几种判断RecyclerView到达底部的方法，发现各有千秋。以下的分析都以上一篇文章的SwipeRecyclerView为例。

```
1.lastVisibleItemPosition == totalItemCount - 1判断；
2.computeVerticalScrollRange()等三个方法判断；
3.canScrollVertically(1)判断；
4.利用RecyclerView的LinearLayoutManager几个方法判断。
```

其实，第2和第3种是属于同一种方法，在下面的分析会讲到。

一、首先，我们来介绍和分析一下第一种方法，也是网上最多人用的方法：

```java
public static boolean isVisBottom(RecyclerView recyclerView){  
  LinearLayoutManager layoutManager = (LinearLayoutManager) recyclerView.getLayoutManager();  
  //屏幕中最后一个可见子项的position
  int lastVisibleItemPosition = layoutManager.findLastVisibleItemPosition();  
  //当前屏幕所看到的子项个数
  int visibleItemCount = layoutManager.getChildCount();  
  //当前RecyclerView的所有子项个数
  int totalItemCount = layoutManager.getItemCount();  
  //RecyclerView的滑动状态
  int state = recyclerView.getScrollState();  
  if(visibleItemCount > 0 && lastVisibleItemPosition == totalItemCount - 1 && state == recyclerView.SCROLL_STATE_IDLE){   
     return true; 
  }else {   
     return false;  
  }
}
```

很明显，当屏幕中最后一个子项lastVisibleItemPosition等于所有子项个数totalItemCount - 1，那么RecyclerView就到达了底部。但是，我在这种方法中发现了极为极端的情况，就是当totalItemCount等于1，而这个子项的高度比屏幕还要高。

![item_recycleview](http://upload-images.jianshu.io/upload_images/2185296-4d48c063428a3237.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

看看效果图：

![](http://upload-images.jianshu.io/upload_images/2185296-3065121fc79c79ae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们可以发现这个子项没完全显示出来就已经被判断为拉到底部。当然，这种方法一般情况下都能满足开发者的需求，只是遇到了强迫症的我~

二、下面我们介绍第二种方法：

```java
public static boolean isSlideToBottom(RecyclerView recyclerView) {    
   if (recyclerView == null) return false; 
   if (recyclerView.computeVerticalScrollExtent() + recyclerView.computeVerticalScrollOffset() 
        >= recyclerView.computeVerticalScrollRange())   
     return true;  
   return false;
}
```

这种方法原理其实很简单，而且也是View自带的方法。

![](http://upload-images.jianshu.io/upload_images/2185296-78f18e9fabbcc7a2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这样就很清晰明了，computeVerticalScrollExtent()是当前屏幕显示的区域高度，computeVerticalScrollOffset() 是当前屏幕之前滑过的距离，而computeVerticalScrollRange()是整个View控件的高度。

这种方法经过测试，暂时还没发现有bug，而且它用的是View自带的方法，所以个人觉得比较靠谱。

三、下面讲讲第三种方法：

```
RecyclerView.canScrollVertically(1)的值表示是否能向上滚动，false表示已经滚动到底部
RecyclerView.canScrollVertically(-1)的值表示是否能向下滚动，false表示已经滚动到顶部
```

这种方法更简单，就通过简单的调用方法，就可以得到你想要的结果。我一讲过这种方法与第二种方法其实是同一种方法，那下面来分析一下，看看canScrollVertically的源码：

![](http://upload-images.jianshu.io/upload_images/2185296-667689a339975ccb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

是不是一目鸟然了，canScrollVertically方法的实现实际上运用到的是方法二的三个函数，只是这个方法Android已经帮我们封装好了，原理一模一样的。

本人现在也是运用了这种方法做判断的~懒人~工具类都省了~

四、最后一种方法其实是比较呆板的，就是利用LinearLayoutManager的几个方法，1.算出已经滑过的子项的距离，2.算出屏幕的高度，3.算出RecyclerView的总高度。然后用他们做比较，原理类似于方法二。

```java
public static int getItemHeight(RecyclerView recyclerView) {  
  int itemHeight = 0;  
  View child = null;  
  LinearLayoutManager layoutManager = (LinearLayoutManager) recyclerView.getLayoutManager();  
  int firstPos = layoutManager.findFirstCompletelyVisibleItemPosition(); 
  int lastPos = layoutManager.findLastCompletelyVisibleItemPosition();  
  child = layoutManager.findViewByPosition(lastPos);  
  if (child != null) {   
     RecyclerView.LayoutParams params = (RecyclerView.LayoutParams) child.getLayoutParams();   
     itemHeight = child.getHeight() + params.topMargin + params.bottomMargin;  
  }   
 return itemHeight;}
```

算出一个子项的高度

```java
public static int getLinearScrollY(RecyclerView recyclerView) {  
  int scrollY = 0;  
  LinearLayoutManager layoutManager = (LinearLayoutManager) recyclerView.getLayoutManager();  
  int headerCildHeight = getHeaderHeight(recyclerView);  
  int firstPos = layoutManager.findFirstVisibleItemPosition();  
  View child = layoutManager.findViewByPosition(firstPos);  
  int itemHeight = getItemHeight(recyclerView);  
  if (child != null) {   
     int firstItemBottom = layoutManager.getDecoratedBottom(child);   
     scrollY = headerCildHeight + itemHeight * firstPos - firstItemBottom;    
     if(scrollY < 0){    
         scrollY = 0;    
     }  
  }  
  return scrollY;
}
```

算出滑过的子项的总距离

```java
public static int getLinearTotalHeight(RecyclerView recyclerView) {    int totalHeight = 0;  
  LinearLayoutManager layoutManager = (LinearLayoutManager) recyclerView.getLayoutManager();  
  View child = layoutManager.findViewByPosition(layoutManager.findFirstVisibleItemPosition());  
  int headerCildHeight = getHeaderHeight(recyclerView);  
  if (child != null) {   
     int itemHeight = getItemHeight(recyclerView);    
     int childCount = layoutManager.getItemCount();    
     totalHeight = headerCildHeight + (childCount - 1) * itemHeight;  
  }  
  return totalHeight;
}
```

算出所有子项的总高度

```java
public static boolean isLinearBottom(RecyclerView recyclerView) {    
boolean isBottom = true;  
  int scrollY = getLinearScrollY(recyclerView);  
  int totalHeight = getLinearTotalHeight(recyclerView); 
  int height = recyclerView.getHeight();
 //    Log.e("height","scrollY  " + scrollY + "  totalHeight  " +  totalHeight + "  recyclerHeight  " + height);  
  if (scrollY + height < totalHeight) {    
    isBottom = false;  
  }  
  return isBottom;
}
```

高度作比较

虽然这种方法看上去比较呆板的同时考虑不很周全，但这种方法可以对RecylerView的LinearLayoutManager有深一步的理解，这也是我的师兄给我提供的一个借鉴的类，我非常感谢他！有兴趣的同学可以去下载源码的做进一步的研究，发现有更好玩的方法可以一起研究！
[源码下载链接](https://github.com/19snow93/SwipeRecyclerView)