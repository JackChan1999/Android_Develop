##### 我们先来看下UC浏览器首页的滑动动画和我最终实现的动画效果

![UC浏览器首页效果图](http://img.blog.csdn.net/20160526232024470) 
![UC浏览器首页效果图](http://img.blog.csdn.net/20160526232223237) 
![UC浏览器首页效果图](http://img.blog.csdn.net/20160526232326059) 
![UC浏览器首页效果图](http://img.blog.csdn.net/20160526232403427)

##### 使用方式

```xml
<cn.ittiger.ucpage.view.UCIndexView
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:ucindexview="http://schemas.android.com/apk/res-auto"
        android:id="@+id/ucindexview"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        ucindexview:pageHeadViewHeight="@dimen/dp_40"
        ucindexview:isPageHeadViewFixed="false"
        ucindexview:isContentHeadViewEnable="true"
        ucindexview:isPullRestoreEnable="false"
        ucindexview:contentHeadViewHeight="@dimen/dp_40"
        ucindexview:pageNavigationViewHeight="@dimen/dp_200"
        ucindexview:pageHeadViewLayoutId="@layout/page_head_view_layout"
        ucindexview:pageNavigationViewLayoutId="@layout/page_navigation_view_layout"
        ucindexview:contentHeadViewLayoutId="@layout/content_head_view_layout"
        ucindexview:contentViewLayoutId="@layout/content_view_layout">

    </cn.ittiger.ucpage.view.UCIndexView>
```

使用方式只需要如此布局即可，没有其他的任何代码操作，非常简单。几个重要属性如下： 
1. `isPageHeadViewFixed：`设置`PageHeadView`是否固定显示(效果如上图2) 
2. `isContentHeadViewEnable：`设置`ContentHeadView`是否启用(效果如上图3) 
3. `isPullRestoreEnable：`设置是否可以通过下拉手势恢复到初始状态，UC首页下拉不能恢复初始状态 
4. `contentHeadViewHeight：`设置`contentHeadView`视图的高度 
5. `pageNavigationViewHeight：`设置`PageNavigationView`视图的高度 
6. `pageHeadViewLayoutId：`设置`PagetHeadView`视图的内容布局，如图中的文字`UC头条`布局 
7. `pageNavigationViewLayoutId：`设置`PageNavigationView`视图的内容布局，如图中的文字`网址导航`布局 
8. `contentHeadViewLayoutId：`设置`ContentHeadViev`视图的内容布局，如图中的文字`新闻头部导航`布局 
9. `contentViewLayoutId：`设置`ContentView`视图的内容布局，如图中的文字`新闻内容区`布局

------

##### github地址

实现及Demo地址：[https://github.com/huyongli/UCIndexAnimation](https://github.com/huyongli/UCIndexAnimation)

------

### 下面来讲讲我的实现过程

##### 首先来分析下UC首页这个动画中涉及到的元素和要点

1. 向上滑动过程顶部的`UC头条`会慢慢的显示出来，我把这部分视图称为：`PageHeadView`(页面头部视图)，很明显其初始化时是在屏幕外部的
2. 向上滑动过程中`UC漫站`上面会有个`新闻Tab菜单`头部慢慢的显示出来，我把这部分视图称为：`ContentHeadView`(新闻头部视图)，初始时是被`ContentView`遮盖，而后慢慢滑出，显然它的滑动速度是比`ContentView`快的
3. 向上滑动过程中`天气、搜索、网站导航`会稍微往上滑动一段距离最终隐藏，我把这部分视图称为：`PageNavigationView`(页面导航视图)，随着不断的滑动，会慢慢的被其余三个视图共同遮盖掉
4. `UC漫站`整个新闻内容视图慢慢的向上滑动直至跟`UC头条`相接，我把这部分称为：`ContentView`(新闻内容视图)
5. 根据动画效果来看，上面所说的四个不同的视图部分都是同时停止滑动，但是他们滑动的距离明显是不相同的 
   `PageHeadView`滑动的距离为其自身高度，`ContentHeadView`的滑动距离为其自身高度与`ContentView`滑动的距离之和，而其相对`ContentView`视图的滑动距离为其自身高度`ContentView`滑动距离为其自身初始距离顶部的边距减去 `PageHeadView`和`ContentHeadView`两者的高度`PageNavigationView`的滑动距离很小
6. 上面的分析把UC首页整体划分成四个不同的View部分，另外其首页中只有`ContentView`、`PageNavigationView`两个视图会处理滑动事件

------

##### 实现技术点和细节

根据上面的分析我们知道有三个视图是从被遮盖到显示或者是从显示到遮盖，剩下的`ContentView`一直是遮盖其他的视图。根据视图遮盖很容易联想到`FrameLayout`布局，多个`FrameLayout`布局叠加在一起就类似PS中的图层叠加，滑动可以看成是不同层级的`FrameLayout`的marginTop不断变化的过程。

##### 实现类结构图

我的实现中先要说明两点： 
\1. 我把上滑称为`展示状态`或者叫`Show状态` 
\2. 下滑称为`恢复初始化状态`或者叫`Hide状态`

下图是我的实现类结构图： 
![这里写图片描述](http://img.blog.csdn.net/20160527182733173)

##### 主要类实现解析

1. `UCIndexView：`模仿UC首页的最终实现，使用时在布局文件中使用此类即可
2. `PageHeadView、PageNavigationView、ContentView、ContentHeadView：`这四个类的作用见我上面的分析
3. `MoveView：`页面四个视图的基类，主要包含滑动过程中的一些基本属性和方法 
   `mNeedMoveHeight：`视图需要滑动的距离(`ContentHeadView`的此属性设置为相对`ContentView`的滑动距离)，此属性主要用来计算各自视图在滑动过程中的步长`mShowStopMarginTop：`视图进行`Show`操作时，当视图的`marginTop`值等于该值时，结束`Show`操作。此值用来确定上滑过程中，视图滑动结束时的位置。`mHideStopMarginTop：`视图进行`Hide`操作时，当视图的`marginTop`值等于该值时，结束`Hide`操作。此值用来确定下滑过程中，视图滑动结束时的位置。通常此值为视图初始化成功后的`marginTop`值。`getMarginTop()：`获取视图当前的`marginTop`值`updateMarginTop(flaot step)：`根据当前的滑动步长更新视图的`marginTop`值`isHideFinish()：`判断当前视图的`Hide`操作是否完成`isShowFinish()：`判断当前视图的`Show`操作是否完成`getShowMoveStep(float step)：`获取视图`Show`过程中的实际可滑动步长，有可能计算得到的滑动步长超过了视图的可滑动距离`getHideMoveStep(float step)：`获取视图`Hide`过程中的实际可滑动步长，有可能计算得到的滑动步长超过了视图的可滑动距离
4. `TouchMoveView：`用来处理手指触摸事件从而滑动的视图基类

##### 滑动事件处理

直接上代码：

```
@Override
    public void onTouchMoveEvent(MotionEvent event) {

        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                mLastTouchY = event.getRawY();
                break;
            case MotionEvent.ACTION_MOVE:
                //手指滑动过程中的滑动步长
                mDelY = event.getRawY() - mLastTouchY;
                viewMove(mDelY, mIsPullRestoreEnable);
                mLastTouchY = event.getRawY();
                break;
            case MotionEvent.ACTION_UP:
                int offset = 0;
                if(mDelY > 0) {//hide，下拉
                    if(!mIsPullRestoreEnable) {//当前不允许下拉恢复
                        return;
                    }
                    offset = mContentView.getHideOffset();
                } else {//show， 上拉
                    offset = mContentView.getShowOffset();
                }
                if(offset <= mPageHeadView.getNeedMoveHeight() / 2) {//没有滑过二分之一高度
                    slip(-mDelY, mIsPullRestoreEnable);
                } else {
                    slip(mDelY, mIsPullRestoreEnable);
                }
                break;
        }
    }
```

```
/**
     * 对所有视图进行滑动操作
     * @param delY                  当前的滑动步长
     * @param isPullRestoreEnable   是否允许下拉恢复
     */
    private void viewMove(float delY, boolean isPullRestoreEnable) {

        float step = Math.abs(delY);
        //根据滑动距离的比例计算PageHeadView的滑动步长
        float pageHeadViewStep = step * mPageHeadView.getNeedMoveHeight() / mContentView.getNeedMoveHeight();
        //手指滑动距离作为ContentView的滑动步长
        float contentViewStep = step;
        //ContentHeadView初始不固定显示时，其实际滑动步长为ContentView的滑动步长加上其相对ContentView的滑动步长
        float contentHeadViewStep = mIsContentHeadViewEnable ? step + step * mContentHeadView.getNeedMoveHeight() / mContentView.getNeedMoveHeight() : 0;
        float pageNavigationViewStep = step * mPageNavigationView.getNeedMoveHeight() / mContentView.getNeedMoveHeight();

        if(delY > 0) {//下滑
            if(!isPullRestoreEnable) {//当前不允许下拉恢复
                return;
            }
            if(!isHideFinish()) {//恢复状态是否已完成
                if(mIsPageHeadViewFixed == false) {
                    mPageHeadView.onHideAnimation(pageHeadViewStep);
                }
                mContentView.onHideAnimation(contentViewStep);
                if(mIsContentHeadViewEnable) {
                    mContentHeadView.onHideAnimation(contentHeadViewStep);
                }
                mPageNavigationView.onHideAnimation(pageNavigationViewStep);
            }
        } else {//上滑
            if(!isShowFinish()) {//展示状态是否已完成
                if(mIsPageHeadViewFixed == false) {//PageHeadView没有被固定时才进行滑动
                    mPageHeadView.onShowAnimation(pageHeadViewStep);
                }
                mContentView.onShowAnimation(contentViewStep);
                if(mIsContentHeadViewEnable) {//ContentHeadView启用时才进行滑动
                    mContentHeadView.onShowAnimation(contentHeadViewStep);
                }
                mPageNavigationView.onShowAnimation(pageNavigationViewStep);
            }
        }
    }
```

```
/**
 * 手指松开屏幕后，视图自动滑动
 * 每隔 mAutoSlipTimeStep 长时间滑动 mAutoSlipStep 距离
 * @param delY  当前的滑动距离
 *  @param isPullRestoreEnable   是否允许下拉恢复
 */
private void slip(float delY, final boolean isPullRestoreEnable) {

    if(delY > 0) {//当前滑动为向下滑动，即处于恢复状态
        if(isHideFinish()) {//已经恢复结束
            return;
        }
        postDelayed(new Runnable() {
            @Override
            public void run() {

                viewMove(mAutoSlipStep, isPullRestoreEnable);
                slip(mAutoSlipStep, isPullRestoreEnable);//准备下一次滑动
            }
        }, mAutoSlipTimeStep);
    } else {//当前滑动为向上滑动，即处于展示状态
        if(isShowFinish()) {//已经展示结束
            return;
        }
        postDelayed(new Runnable() {
            @Override
            public void run() {

                viewMove(-mAutoSlipStep, isPullRestoreEnable);
                slip(-mAutoSlipStep, isPullRestoreEnable);//准备下一次滑动
            }
        }, mAutoSlipTimeStep);
    }
}
```

上面的代码中我把手指每次的滑动距离当做`ContentView`的滑动步长，再根据`ContentView`的滑动步长计算其他视图的当前滑动步长，计算具体方式可以看代码中的注释。

其实上面代码里的实现步骤还是比较清晰简单的，主要是如下几个步骤：

1. 在MotionEvent.MOVE中先得到当前手指滑动的距离，此距离作为ContentView的此次滑动步长 
   根据ContentView的滑动步长计算其他三个视图的滑动步长
2. 根据当前是上滑还是下滑，判断是否滑动结束，没有结束则按照计算得到的步长继续对View进行滑动处理
3. 当手指松开后，在MotionEvent.UP中判断ContentView的滑动距离是否达到了PageHeadView高度的一半，从而决定是继续同方向滑动至结束还是恢复到原状态
4. 手指松开后，最终调用slip方法开始自动循环滑动处理，直至滑动结束