# 1. 前言

Google IO 2015的Design Support Library的控件进行效果展示。[代码](https://github.com/573842281/[Android](http://lib.csdn.net/base/android)-Blog-Source)

[就看天气](https://github.com/xcc3641/SeeWeather)这个应用已经集成了最新的效果，原作者已经开源，请支持并star

# 2. Design Support Library使用介绍

## 2.1 综述

支持Android 2.1以上设备。 

Gradle build script dependency： 

```gradle
compile 'com.android.support:design:22.2.0' //可修改版本号为自己匹配
```

Design Support Library包含8个控件，具体如下： 

| Widget Name                              | Description            |
| :--------------------------------------- | :--------------------- |
| android.support.design.widget.TextInputLayout | 强大带提示的MD风格的EditText    |
| android.support.design.widget.FloatingActionButton | MD风格的圆形按钮，来自于ImageView |
| android.support.design.widget.Snackbar   | 类似Toast，添加了简单的单个Action |
| android.support.design.widget.TabLayout  | 选项卡                    |
| android.support.design.widget.NavigationView | DrawerLayout的SlideMenu |
| android.support.design.widget.CoordinatorLayout | 超级FrameLayout          |
| android.support.design.widget.AppBarLayout | MD风格的滑动Layout          |
| android.support.design.widget.CollapsingToolbarLayout | 可折叠MD风格ToolbarLayout   |

下面详细说说这些控件的特性和使用注意项。 

## 2.2 TextInputLayout控件

![TextInputLayout](http://img.blog.csdn.net/20150603142024003)

在MD中，使用TextInputLayout将EditText进行了封装，提示信息会变成一个显示在EditText之上的floating label，这样用户就始终知道他们现在输入的是什么，而且过度动画是平滑的。还可以在下方通过setError设置Error提示，使用比较简单，所以不做过多说明，详情见Demo源码。 

注意项：TextInputLayout中至少嵌套一个EditText。 

## 2.3 FloatingActionButton控件

![FloatingActionButton](http://img.blog.csdn.net/20150603142131566)

一个负责显示界面基本操作的圆形按钮。Design library中的FloatingActionButton 实现了一个默认颜色为主题中colorAccent的悬浮操作按钮。除了一般大小的悬浮操作按钮，它还支持mini size（fabSize=”mini”）。FloatingActionButton继承自ImageView，你可以使用android:src或者 ImageView的任意方法，比如setImageDrawable()来设置FloatingActionButton里面的图标。 

无特别注意项，和普通控件类似。 

## 2.4 Snackbar控件

![Snackbar](http://img.blog.csdn.net/20150603142348085)

Snackbar为一个操作提供轻量级、快速的反馈。Snackbar显示在屏幕的底部（有MD动画效果浮现和消失），包含了文字信息与一个可选的操作按钮。在指定时间结束之后自动消失。另外，用户还可以在超时之前将它滑动删除。Snackbar被看作是比Toast更强大的快速反馈机制，你会发现他们的API非常相似。你应该注意到了make()方法中把一个View作为第一个参数（Snackbar试图找到一个合适的父亲以确保自己是被放置于底部）。 

无特殊注意项，和Toast类似。 

## 2.5 TabLayout控件

![TabLayout](http://img.blog.csdn.net/20150603142328036)

通过选项卡的方式切换View并不是MD中才有的新概念，它们和顶层导航模式或者组织app中不同分组内容（比如，不同风格的音乐）是同一个概念。 Design library的TabLayout 既实现了固定的选项卡（View的宽度平均分配），也实现了可滚动的选项卡（View宽度不固定同时可以横向滚动）。如果你使用ViewPager在 tab之间横向切换，你可以直接从PagerAdapter的getPageTitle() 中创建选项卡，然后使用setupWithViewPager()将两者联系在一起。它可以使tab的选中事件能更新ViewPager,同时 ViewPager 的页面改变能更新tab的选中状态。 

注意项：如果你使用ViewPager在tab之间横向切换，切记可以直接从PagerAdapter的getPageTitle() 中创建选项卡，然后使用setupWithViewPager()将两者联系在一起。 

## 2.6 NavigationView控件

![NavigationView](http://img.blog.csdn.net/20150603142539875)

抽屉导航是app识别度与内部导航的关键，保持这里设计上的一致性对app的可用性至关重要，尤其是对于第一次使用的用户。 NavigationView 通过提供抽屉导航所需的框架让实现更简单，同时它还能够直接通过菜单资源文件直接生成导航元素。把NavigationView作为 DrawerLayout的内容视图来使用。NavigationView处理好了和状态栏的关系，可以确保NavigationView在API21+ 设备上正确的和状态栏交互。 

注意项：你可以通过设置一个OnNavigationItemSelectedListener，使用其 setNavigationItemSelectedListener()来获得元素被选中的回调事件。它为你提供被点击的菜单元素，让你可以处理选择事件、改变复选框状态、加载新内容、关闭导航菜单，以及其他任何你想做的操作。你会注意到NavigationView的两个新自定义属性如下： 

| new attr         | description          |
| :--------------- | :------------------- |
| app:headerLayout | 控制头部的布局              |
| app:menu         | 导航菜单的资源文件（也可以在运行时配置） |

## 2.7 CoordinatorLayout控件

![CoordinatorLayout](http://img.blog.csdn.net/20150603142453883)

手势,及滚动布局，MD的手势有很多组成部分，包括touch ripples和meaningful transitions。Design library引入了CoordinatorLayout，一个从另一层面去控制子view之间触摸事件的布局，Design library中的很多控件都利用了它。一个很好的例子就是当你将FloatingActionButton作为一个子View添加进 CoordinatorLayout并且将CoordinatorLayout传递给 Snackbar.make()，在3.0及其以上的设备上，Snackbar不会显示在悬浮按钮的上面，而是FloatingActionButton 利用CoordinatorLayout提供的回调方法，在Snackbar以动画效果进入的时候自动向上移动让出位置，并且在Snackbar动画地消失的时候回到原来的位置，不需要额外的代码。 

CoordinatorLayout的另一个用例是ActionBar与滚动技巧。你可能已经在自己的布局中使用了Toolbar ，它允许你更加自由的自定义其外观与布局的其余部分融为一体。Design library把这种设计带到了更高的水平，使用AppBarLayout可以让你的Toolbar与其他View（比如TabLayout的选项卡）能响应被标记了ScrollingViewBehavior的View的滚动事件。 

注意项：

当用户滚动RecyclerView，AppBarLayout可以这样响应滚动事件： 

根据子view的滚动标志（scroll flag）来控制它们如何进入（滚入屏幕）与退出（滚出屏幕）。 

Flag包括： 

- scroll：所有想滚动出屏幕的View都需要设置这个flag，没有设置这个flag的View将被固定在屏幕顶部。 
- enterAlways：这个flag让任意向下的滚动都会导致该View变为可见，启用快速“返回模式”。 
- enterAlwaysCollapsed：当你的视图已经设置minHeight属性又使用此标志时，你的视图只能已最小高度进入，只有当滚动视图到达顶部时才扩大到完整高度。 
- exitUntilCollapsed：this flag causes the view to scroll off until it is ‘collapsed’ (its minHeight) before exiting。 

特别注意：所有使用scroll flag的View都必须定义在没有使用scroll flag的View前面，这样才能确保所有的View从顶部退出，留下固定的元素。 

PS一句：CoordinatorLayout还提供了layout_anchor和layout_anchorGravity属性一起配合使用，可以用于放置floating view，比如FloatingActionButton与其他View的相对位置。相见Demo中演示。 

## 2.8 AppBarLayout控件

![AppBarLayout](http://img.blog.csdn.net/20150603142727562)

这个没啥解释的，就是一个ViewGroup，配合ToolBar与CollapsingToolbarLayout等使用。就是一个纯容器类。 

无特殊注意项。 

## 2.9 CollapsingToolbarLayout控件

![CollapsingToolbarLayout](http://img.blog.csdn.net/20150603142632834)

可伸缩折叠的Toolbar （Collapsing Toolbar），直接添加Toolbar到AppBarLayout可以让你使用enterAlwaysCollapsed和 exitUntilCollapsedscroll标志，但是无法控制不同元素如何响应collapsing的细节。这里使用了 CollapsingToolbarLayout的app:layout_collapseMode=”pin”来确保Toolbar在view折叠的时候仍然被固定在屏幕的顶部。还可以做到更好的效果，当你让CollapsingToolbarLayout和Toolbar在一起使用的时候，title 会在展开的时候自动变得大些，而在折叠的时候让字体过渡到默认值。必须注意，在这种情况下你必须在CollapsingToolbarLayout上调用 setTitle()，而不是在Toolbar上。除了固定住View，你还可以使用 app:layout_collapseMode=”parallax”（以及使用 app:layout_collapseParallaxMultiplier=”0.7”来设置视差因子）来实现视差滚动效果（比如 CollapsingToolbarLayout里面的一个ImageView），这中情况和CollapsingToolbarLayout的 app:contentScrim=”?attr/colorPrimary”属性一起配合更完美。 

有一件事情必须注意，那就是CoordinatorLayout并不知道FloatingActionButton或者AppBarLayout的内部工作原理，它只是以Coordinator.Behavior的形式提供了额外的API，该API可以使子View更好的控制触摸事件与手势以及声明它们之间的依赖，并通过onDependentViewChanged()接收回调。 

可以使用CoordinatorLayout.DefaultBehavior(你的View.Behavior.class)注解或者在布局中使用app:layout_behavior=”com.example.app.你的View$Behavior”属性来定义view的默认行为。 framework让任意View和CoordinatorLayout结合在一起成为了可能。 

注意项：注意项上面描述部分已经声明，不需要额外说明。 

# 3. 总结

到此2015 Google IO的新suppory包控件完全介绍完毕。小伙伴们赶紧用起来吧。下载Demo请点击：[Design Support Library Demo](https://github.com/yanbober/Android-Blog-Source/tree/master/Support-Library-Demo)

提示：android.support.v4.widget.NestedScrollView也是新控件，NestedScrollView和ScrollView基本功能一致，只不过NestedScrollView可以兼容新的控件，如：
```xml
<?xml version="1.0" encoding="utf-8"?>  
<android.support.design.widget.CoordinatorLayout 
 xmlns:android="http://schemas.android.com/apk/res/android"  
   xmlns:app="http://schemas.android.com/apk/res-auto"  
   android:id="@+id/main_content"  
   android:layout_width="match_parent"  
   android:layout_height="match_parent"  
   android:fitsSystemWindows="true">  
 
    <!--第一部分：伸缩工具栏-->  
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
                android:src="@drawable/cheese_2"  
                app:layout_collapseMode="parallax" />  
  
            <android.support.v7.widget.Toolbar  
                android:id="@+id/toolbar"  
                android:layout_width="match_parent"  
                android:layout_height="?attr/actionBarSize"  
                app:layout_collapseMode="pin"  
                app:popupTheme="@style/ThemeOverlay.AppCompat.Light" />  
  
        </android.support.design.widget.CollapsingToolbarLayout>  
  
    </android.support.design.widget.AppBarLayout>  
  
    <!--第二部分：主要内容，NestedScrollView和ScrollView基本功能一致，只不过NestedScrollView可以兼容新的控件-->  
    <android.support.v4.widget.NestedScrollView  
        android:layout_width="match_parent"  
        android:layout_height="match_parent"  
        app:layout_behavior="@string/appbar_scrolling_view_behavior">  
  
        <LinearLayout  
            android:layout_width="match_parent"  
            android:layout_height="match_parent"  
            android:orientation="vertical"  
            android:paddingTop="24dp">  
  
            <!--卡片布局-->  
            <android.support.v7.widget.CardView  
                android:layout_width="match_parent"  
                android:layout_height="wrap_content"  
                android:layout_margin="@dimen/card_margin">  
  
                <LinearLayout                      
                    android:layout_width="match_parent"  
                    android:layout_height="wrap_content">  
  
                    <TextView  
                        android:layout_width="match_parent"  
                        android:layout_height="wrap_content"  
                        android:text="Info"  
                        android:textAppearance="@style/TextAppearance.AppCompat.Title" />  
  
                    <TextView  
                        android:layout_width="match_parent"  
                        android:layout_height="wrap_content"  
                        android:text="@string/cheese_ipsum" />  
                </LinearLayout>  
  
            </android.support.v7.widget.CardView>  
  
            <android.support.v7.widget.CardView  
                android:layout_width="match_parent"  
                android:layout_height="wrap_content"  
                android:layout_marginBottom="@dimen/card_margin"  
                android:layout_marginLeft="@dimen/card_margin"  
                android:layout_marginRight="@dimen/card_margin">  
  
                <LinearLayout                       
                    android:layout_width="match_parent"  
                    android:layout_height="wrap_content">  
  
                    <TextView  
                        android:layout_width="match_parent"  
                         android:layout_height="wrap_content"  
                         android:text="Friends"  
                         android:textAppearance="@style/TextAppearance.AppCompat.Title" />  
   
                     <TextView  
                         android:layout_width="match_parent"  
                         android:layout_height="wrap_content"  
                         android:text="@string/cheese_ipsum" />     
                 </LinearLayout>     
             </android.support.v7.widget.CardView>  
   
             <android.support.v7.widget.CardView  
                 android:layout_width="match_parent"  
                 android:layout_height="wrap_content"  
                 android:layout_marginBottom="@dimen/card_margin"  
                 android:layout_marginLeft="@dimen/card_margin"  
                 android:layout_marginRight="@dimen/card_margin">  
   
                 <LinearLayout                   
                     android:layout_width="match_parent"  
                     android:layout_height="wrap_content">  
   
                     <TextView  
                         android:layout_width="match_parent"  
                         android:layout_height="wrap_content"  
                         android:text="Related"  
                         android:textAppearance="@style/TextAppearance.AppCompat.Title" />  
   
                     <TextView  
                         android:layout_width="match_parent"  
                         android:layout_height="wrap_content"  
                         android:text="@string/cheese_ipsum" />    
                 </LinearLayout>    
             </android.support.v7.widget.CardView>  
         </LinearLayout>  
     </android.support.v4.widget.NestedScrollView>  
   
     <!--第三部分：漂浮按钮-->  
     <android.support.design.widget.FloatingActionButton  
         android:layout_width="wrap_content"  
         android:layout_height="wrap_content"  
         android:layout_margin="@dimen/fab_margin"  
         android:clickable="true"  
         android:src="@drawable/ic_discuss"  
         app:layout_anchor="@id/appbar"  
         app:layout_anchorGravity="bottom|right|end" />  
 </android.support.design.widget.CoordinatorLayout>  
```

补充：说说CollapsingToolbarLayout（来自：http://www.open-open.com/lib/view/open1438265746378.html）

CollapsingToolbarLayout作用是提供了一个可以折叠的Toolbar，它继承至FrameLayout，给它设置layout_scrollFlags，它可以控制包含在CollapsingToolbarLayout中的控件(如：ImageView、Toolbar)在响应layout_behavior事件时作出相应的scrollFlags滚动事件(移除屏幕或固定在屏幕顶端)。

使用CollapsingToolbarLayout：

```xml
<android.support.design.widget.AppBarLayout
    android:layout_width="match_parent"
    android:layout_height="256dp"
    android:fitsSystemWindows="true">

    <android.support.design.widget.CollapsingToolbarLayout
        android:id="@+id/collapsing_toolbar_layout"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:contentScrim="#30469b"
        app:expandedTitleMarginStart="48dp"
        app:layout_scrollFlags="scroll|exitUntilCollapsed">

        <ImageView
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:scaleType="centerCrop"
            android:src="@mipmap/bg"
            app:layout_collapseMode="parallax"
            app:layout_collapseParallaxMultiplier="0.7"  />

        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            app:layout_collapseMode="pin" />

    </android.support.design.widget.CollapsingToolbarLayout>

</android.support.design.widget.AppBarLayout>
```
我们在CollapsingToolbarLayout中设置了一个ImageView和一个Toolbar。并把这个CollapsingToolbarLayout放到AppBarLayout中作为一个整体。

1、在CollapsingToolbarLayout中：

我们设置了layout_scrollFlags:关于它的值我这里再说一下：

- scroll - 想滚动就必须设置这个。
- enterAlways - 实现quick return效果, 当向下移动时，立即显示View（比如Toolbar)。
- exitUntilCollapsed - 向上滚动时收缩View，但可以固定Toolbar一直在上面。
- enterAlwaysCollapsed - 当你的View已经设置minHeight属性又使用此标志时，你的View只能以最小高度进入，只有当滚动视图到达顶部时才扩大到完整高度。

其中还设置了一些属性，简要说明一下：

- contentScrim - 设置当完全CollapsingToolbarLayout折叠(收缩)后的背景颜色。
- expandedTitleMarginStart - 设置扩张时候(还没有收缩时)title向左填充的距离。

没扩张时候如图：

![Android5.0+(CollapsingToolbarLayout)](http://static.open-open.com/lib/uploadImg/20150730/20150730221533_599.jpg)

2、在ImageView控件中：

我们设置了：

- layout_collapseMode (折叠模式) - 有两个值:
  - pin -  设置为这个模式时，当CollapsingToolbarLayout完全收缩后，Toolbar还可以保留在屏幕上。
  - parallax - 设置为这个模式时，在内容滚动时，CollapsingToolbarLayout中的View（比如ImageView)也可以同时滚动，实现视差滚动效果，通常和layout_collapseParallaxMultiplier(设置视差因子)搭配使用。
- layout_collapseParallaxMultiplier(视差因子) - 设置视差滚动因子，值为：0~1。 

3、在Toolbar控件中：

我们设置了layout_collapseMode(折叠模式)：为pin。

综上分析：当设置了layout_behavior的控件响应起了CollapsingToolbarLayout中的layout_scrollFlags事件时，ImageView会有视差效果的向上滚动移除屏幕，当开始折叠时CollapsingToolbarLayout的背景色(也就是Toolbar的背景色)就会变为我们设置好的背景色，Toolbar也一直会固定在最顶端。效果如图

![Android5.0+(CollapsingToolbarLayout)](http://static.open-open.com/lib/uploadImg/20150730/20150730221534_501.jpg)

注意：使用CollapsingToolbarLayout时必须把title设置到CollapsingToolbarLayout上，设置到Toolbar上不会显示。

```java
mCollapsingToolbarLayout.setTitle("title"); // 该变title的字体颜色
mCollapsingToolbarLayout.setExpandedTitleColor(); // 扩张时候的title颜色
// 收缩后在Toolbar上显示时的title的颜色
mCollapsingToolbarLayout.setCollapsedTitleTextColor();
```
这个颜色的过度变化其实CollapsingToolbarLayout已经帮我们做好，它会自动的过度，比如我们把收缩后的title颜色设为绿色，效果如图：

![Android5.0+(CollapsingToolbarLayout)](http://static.open-open.com/lib/uploadImg/20150730/20150730221537_268.jpg)

没录好，反正效果出来了。接下来看看代码怎么实现吧

布局文件
```xml
<android.support.design.widget.CoordinatorLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <android.support.design.widget.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="256dp"
        android:fitsSystemWindows="true">

        <android.support.design.widget.CollapsingToolbarLayout
            android:id="@+id/collapsing_toolbar_layout"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:contentScrim="#30469b"
            app:expandedTitleMarginStart="48dp"
            app:layout_scrollFlags="scroll|exitUntilCollapsed">

            <ImageView
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:scaleType="centerCrop"
                android:src="@mipmap/bg"
                app:layout_collapseMode="parallax"
                app:layout_collapseParallaxMultiplier="0.7"/>

            <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                app:layout_collapseMode="pin"/>
        </android.support.design.widget.CollapsingToolbarLayout>

    </android.support.design.widget.AppBarLayout>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        app:layout_behavior="@string/appbar_scrolling_view_behavior">

        <android.support.v7.widget.RecyclerView
            android:id="@+id/recyclerView"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:scrollbars="none"/>
    </LinearLayout>

</android.support.design.widget.CoordinatorLayout>
```

代码文件： 

```java
Toolbar mToolbar = (Toolbar) findViewById(R.id.toolbar);
setSupportActionBar(mToolbar);
getSupportActionBar().setDisplayHomeAsUpEnabled(true);
mToolbar.setNavigationOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        onBackPressed();
    }
});

//使用CollapsingToolbarLayout必须把title设置到CollapsingToolbarLayout上，设置到Toolbar上则不会显示
CollapsingToolbarLayout mCollapsingToolbarLayout = (CollapsingToolbarLayout) findViewById(R.id.collapsing_toolbar_layout);
mCollapsingToolbarLayout.setTitle("CollapsingToolbarLayout");

//通过CollapsingToolbarLayout修改字体颜色
mCollapsingToolbarLayout.setExpandedTitleColor(Color.WHITE);//设置还没收缩时状态下字体颜色
mCollapsingToolbarLayout.setCollapsedTitleTextColor(Color.GREEN);//设置收缩后
```