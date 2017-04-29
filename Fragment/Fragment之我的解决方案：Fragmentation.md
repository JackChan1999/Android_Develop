Fragment系列文章：
1、[Fragment全解析系列（一）：那些年踩过的坑](http://www.jianshu.com/p/d9143a92ad94)
2、[Fragment全解析系列（二）：正确的使用姿势](http://www.jianshu.com/p/fd71d65f0ec6)
3、Fragment之我的解决方案：Fragmentation

附：[SwipeBackFragment的实现分析](http://www.jianshu.com/p/626229ca4dc2)

如果你通读了本系列的前两篇，我相信你可以写出大部分场景都能正常运行的Fragment了。如果你想了解更多，那么你可以看看我封装的这个库：Fragmentation。
本篇主要介绍这个库，解决了一些BUG，使用简单，提供实时查看栈视图等实用功能。

------

源码地址：[Github](https://github.com/YoKeyword/Fragmentation)，欢迎Star，Fork。

[Demo网盘下载（V_0.9.0）](https://pan.baidu.com/s/1c1Nku0C)
Demo演示：
单Activity + 多Fragment，项目中有3个Demo。

流式的单Activity＋多Fragment：

流式的单Activity＋多Fragment

类似微信交互方式的单Activity+多Fragment：（全页面支持滑动返回）

类似微信交互方式的单Activity+多Fragment

类似新版仿知乎交互方式的单Activity＋多Frgment：

类似新版仿知乎交互方式的单Activity＋多Frgment

# Fragmentation

为"单Activity ＋ 多Fragment的架构","多模块Activity + 多Fragment的架构"而生，帮你简化使用过程，轻松解决各种复杂嵌套等问题，修复了官方Fragment库存在的一些BUG。

![img](http://upload-images.jianshu.io/upload_images/937851-786e8de65830922a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 特性

1、**快速开发出各种嵌套设计的Fragment App**

2、**实时查看Fragment的(包括嵌套Fragment)栈视图的对话框和Log，方便调试**

3、**增加启动模式、startForResult等类似Activity方法**

4、**类似Android事件分发机制的Fragment回退方法：onBackPressedSupport()，轻松为每个Fragment实现Back按键事件**

5、**New！！！ 提供onSupportVisible()等生命周期方法，简化嵌套Fragment的开发过程； 提供统一的onLazyInitView()懒加载方法**

6、**提供靠谱的 Fragment转场动画 的解决方案**

7、**更强的兼容性, 解决多点触控、重叠等问题**

8、**支持SwipeBack滑动边缘退出(需要使用Fragmentation_SwipeBack库,详情README)**

通过logFragmentStackHierarchy(TAG)查看Log

# TODO

~~* 栈视图悬浮球／摇一摇唤出 栈视图~~

- Activity侧滑返回：更换实现方式
- Fragment侧滑返回：实现视觉差效果
- replace进一步的支持
- Fragment路由module

# 重大更新日志

### 0.10.X [详情点这里](https://github.com/YoKeyword/Fragmentation/wiki/Home)

1、添加可全局配置的Fragmentaion Builder类：

- 提供方便打开栈视图Dialog的方法：
  - bubble: 显示悬浮球，可点击唤出栈视图Dialog，可自由拖拽
  - shake: 摇一摇唤出栈视图Dialog
  - none: 默认不显示栈视图Dialog
- 根据是否是Debug环境，方便区别处理异常（"Can not perform this action after onSaveInstanceState!"）

2、兼容v4-25.1.1

### 0.9.X

1、解决多点触控问题，多项优化、兼容、Fix

2、对于25.1.0+的 v4包，完善了SharedElement！

### 0.8.X

1、提供onLazyInitView()懒加载，onSupportVisible()，onSupportInvisible()等生命周期方法，简化开发；

2、SupportActivity提供registerFragmentLifecycleCallbacks()来监控其下所有Fragment的生命周期；

3、自定义Tag

# 如何使用

**1. 项目下app的build.gradle中依赖：**

```
// appcompat v7包是必须的
compile 'me.yokeyword:fragmentation:0.10.1'
// 如果想使用SwipeBack 滑动边缘退出Fragment/Activity功能，请再添加下面的库
// compile 'me.yokeyword:fragmentation-swipeback:0.7.9'
```

**2. Activity继承SupportActivity：**

```
public class MainActivity extends SupportActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        ...
        if (savedInstanceState == null) {
            loadRootFragment(R.id.fl_container, HomeFragment.newInstance());  // 加载根Fragment
        }
        // 栈视图功能，大大降低Fragment的开发难度，建议在Application里初始化
        Fragmentation.builder()
                // 显示悬浮球 ; 其他Mode:SHAKE: 摇一摇唤出   NONE：隐藏
                .stackViewMode(Fragmentation.BUBBLE)
                .install();
    }
```

**3. Fragment继承SupportFragment：**

```
public class HomeFragment extends SupportFragment {

  private void xxx() {
        // 启动新的Fragment, 同时还有start(fragment,SINGTASK)、startForResult、startWithPop等启动方法
    start(DetailFragment.newInstance(HomeBean));
        // ... 其他方法请自行查看 API
  }
```

### [进一步使用，查看wiki](https://github.com/YoKeyword/Fragmentation/wiki)