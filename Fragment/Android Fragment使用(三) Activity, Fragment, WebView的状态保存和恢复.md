# Android中的状态保存和恢复

Android中的状态保存和恢复, 包括Activity和Fragment以及其中View的状态处理.
Activity的状态除了其中的View和Fragment的状态之外, 还需要用户手动保存一些成员变量.
Fragment的状态有它自己的实例状态和其中的View状态, 因为其生命周期的灵活性和实际需要的不同, 情况会多一些.
根据源码, 列出了Fragment中实例状态和View状态保存和恢复的几个入口, 便于分析查看.
最后专门讲了WebView状态保存和恢复, 问题及处理.
还有一个工具类icepick的介绍.

# Activity的状态保存和恢复

作为热身, 先来讲一下Activity的状态保存和恢复.

## 什么时候需要恢复Activity

关于Activity的销毁和重建, 之前有这么一篇博文: [Activity的重新创建](http://www.cnblogs.com/mengdd/archive/2012/12/17/2822291.html)
总结来说, 就是Activity的销毁, 分为彻底销毁和留下数据的销毁两种.

**彻底销毁**是指用户主动去关闭或退出这个Activity. 此时是不需要状态恢复的, 因为下次回来又是重新创建全新的实例.
**留下数据的销毁**是指系统销毁了activity, 但是当用户返回来时, 会重新创建它, 让用户觉得它一直都在.

屏幕旋转重建可以归结为第二种情况, 打开Do not keep activities开关, 切换activities也是会出现第二种情况.
打开**Do not keep activities**开关就是为了模拟内存不足时的系统行为, 这里有一篇[分析](http://www.cnblogs.com/mengdd/p/4528417.html)

## 如何恢复

实际上系统已经帮我们做好了View层面基本的恢复工作, 主要是依靠下面两个方法:

```
    @Override
    protected void onSaveInstanceState(Bundle outState) {
        super.onSaveInstanceState(outState);
        // 在onStop()之前调用, 文档中说并不保证在onPause()的之前还是之后
        // 我的试验中一般是在onPause()之后
    }

    @Override
    protected void onRestoreInstanceState(Bundle savedInstanceState) {
        super.onRestoreInstanceState(savedInstanceState);
        // 在onStart() 之后
    }
```

Bundle其中包含了activity中的view和fragment的各种信息, 所以调用基类的方法就可以完成基本的view层面的恢复工作.
**注意这两个方法并不是activity的生命周期回调, 对于activity来说它们不是一定会发生的.**
**另外需要注意的是, View必须要有id才能被恢复.**

举一个实例来说明:
Activity A start B, 那么A的`onSaveInstanceState()`会在onStop()之前调用, 以防A被系统销毁.
但是在B中按下back键finish()了自己后, B被销毁的过程中, 并没有调用`onSaveInstanceState()`, 是因为B并没有被压入task的back stack中,
也即系统知道B并不需要储存自己的状态.
正常情况下, 返回到A, A没有被销毁, 也不会调用`onRestoreInstanceState()`, 因为所有的状态都还在, 并不需要重建.

如果我们打开了**Do not keep activities**开关, 模拟系统内存不足时的行为, 从A到B, 可以看到当B resume的时候A会一路走到onDestroy(),
而关掉B之后, A会从onCreate()开始走, 此时onCreate()的参数bundle就不为空了, onStart()之后会调用`onRestoreInstanceState()`方法, 其参数bundle中内容类似于如下:

```
Bundle[{android:viewHierarchyState=Bundle[mParcelledData.dataSize=272]}]
```

其中包含了View的状态, 如果有Fragment, 也会包含Fragment的状态, 其实质是保存了FragmentManagerState, 内容类似于如下:

```
Bundle[{android:viewHierarchyState=Bundle[{android:views={16908290=android.view.AbsSavedState$1@bc382e7, 2131492950=CompoundButton.SavedState{4034f96 checked=true}, 2131492951=android.view.AbsSavedState$1@bc382e7}}], android:fragments=android.app.FragmentManagerState@bacc717}]
```

对于上面的例子来说, B什么时候会调用`onSaveInstanceState()`呢?
当从A打开B之后, 按下Home键, B就会调用`onSaveInstanceState()`.
因为这时候系统不知道用户什么时候会返回, 有可能会把B也销毁了, 所以保存一下它的状态.
如果下次回来它没有被重建, `onRestoreInstanceState()`就不会被调用, 如果它被重建了, `onRestoreInstanceState()`才会被调用.

### Activity保存方法的调用时机

**activity的onSaveInstanceState()和onRestoreInstanceState()方法在如下情形下会调用:**

1. 屏幕旋转重建: 先save再restore.
2. 启动另一个activity: 当前activity在离开前会save, 返回时如果因为被系统杀死需要重建, 则会从onCreate()重新开始生命周期, 调用onRestoreInstanceState(); 如果没有重建, 则不会调用onCreate(), 也不会调用onRestoreInstanceState(), 生命周期从onRestart()开始, 接着onStart()和onResume().
3. 按Home键的情形和启动另一个activity一样, 当前activity在离开前会save, 用户再次点击应用图标返回时, 如果重建发生, 则会调用onCreate()和onRestoreInstanceState(); 如果activity不需要重建, 只是onRestart(), 则不会调用onRestoreInstanceState().

### Activity恢复方法的调用时机

**activity的onSaveInstanceState()和onRestoreInstanceState()方法在如下情形下不会调用:**

1. 用户主动finish()掉的activity不会调用onSaveInstanceState(), 包括主动按back退出的情况.
2. 新建的activity, 从onCreate()开始, 不会调用onRestoreInstanceState().

## Activity中还需要手动恢复什么

如上, 系统已经为我们恢复了activity中的各种view和fragment, 那么我们自己需要保存和恢复一些什么呢?
答案是**成员变量值**.

因为系统并不知道你的各种成员变量有什么用, 哪些值需要保存, 所以需要你自己覆写上面两个方法, 然后把自己需要保存的值加进bundle里面去. 具体例子, 这里[Activity的重新创建](http://www.cnblogs.com/mengdd/archive/2012/12/17/2822291.html)有, 我就不重复了.
重要的是不要忘记调用super的方法, 那里有系统帮我们恢复的工作.

# 工具类Icepick介绍

在介绍下面的内容之前, 先介绍一个小工具: [Icepick](https://github.com/frankiesardo/icepick)
这个工具的作用是, 在你想保存和重建自己的成员变量数据时, 帮你省去那些put和get方法的调用, 你也不用为每一个字段起一个常量key.
你需要做的就是简单地在你想要保存状态的字段上面加上一个`@State` 注解.
然后在保存和恢复的时候分别加上一句话:

```
  @Override
  public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    Icepick.restoreInstanceState(this, savedInstanceState);
  }

  @Override
  public void onSaveInstanceState(Bundle outState) {
    super.onSaveInstanceState(outState);
    Icepick.saveInstanceState(this, outState);
  }
```

然后你的成员变量就有了它应该有的值了, DONE!

# Fragment的状态保存和恢复

Fragment的状态比Activity的要复杂一些, 因为它的生命周期状态比较多.

## Fragment状态保存和恢复的相关方法

按照上面的思路, 我先去查找Fragment中保存和恢复的回调方法了.
Fragment的状态保存回调是这个方法:

```
    public void onSaveInstanceState(Bundle outState) {
        // may be called any time before onDestroy()
    }
```

这个方法和之前activity的情况大体是类似的, 它不是生命周期的回调, 所以只在有需要的时候会调到.
onSaveInstanceState()在activity调用onSaveInstanceState()的时候发生, 用于保存实例状态.(看它的方法名: instance state).
`onSaveInstanceState()`方法保存的bundle会返回给几个生命周期回调: `onCreate()`, `onCreateView()`, `onViewCreated()`和`onActivityCreated()`.

Fragment并没有对应的onRestoreInstanceState()方法.
也即没有实例状态的恢复回调.

Fragment只有一个onViewStateRestored()的回调方法:

```
    public void onViewStateRestored(@Nullable Bundle savedInstanceState) {
        // 在onActivityCreated()和onStart()之间调用
        mCalled = true;
    }
```

onViewStateRestored()每次新建Fragment都会发生.
它并不是实例状态恢复的方法, 只是一个View状态恢复的回调.

**这里需要注意, Fragment的状态分两个类型: 实例状态和View状态**.
这里有个最佳实践: [The Real Best Practices to Save/Restore Activity's and Fragment's state](https://inthecheesefactory.com/blog/fragment-state-saving-best-practices/en)
**不要把Fragment的实例状态和View状态混在一起处理.**

在这里我先上个结论, 把查看源码中Fragment状态保存和恢复的相关方法列出来:

Fragment状态保存入口:
![Fragment状态保存](http://images2015.cnblogs.com/blog/325852/201606/325852-20160613220156526-879299857.png)

Fragment的状态保存入口有三个:

1. Activity的状态保存, 在Activity的`onSaveInstanceState()`里, 调用了FragmentManger的`saveAllState()`方法, 其中会对mActive中各个Fragment的实例状态和View状态分别进行保存.
2. FragmentManager还提供了public方法: `saveFragmentInstanceState()`, 可以对单个Fragment进行状态保存, 这是提供给我们用的, 后面会有例子介绍这个. 其中调用的`saveFragmentBasicState()`方法即为情况一中所用, 图中已画出标记.
3. FragmentManager的`moveToState()`方法中, 当状态回退到`ACTIVITY_CREATED`, 会调用`saveFragmentViewState()`方法, 保存View的状态.

`moveToState()`方法中有很长的switch case, 中间不带break, 基本是根据新状态和当前状态的比较, 分为正向创建和反向销毁两个方向, 一路沿着多个case走下去.

Fragment状态恢复入口:
![Fragment状态恢复](http://images2015.cnblogs.com/blog/325852/201606/325852-20160613220231588-704818073.png)

三个恢复的入口和三个保存的入口刚好对应.

1. 在Activity重新创建的时候, 恢复所有的Fragment状态.
2. 如果调用了FragmentManager的方法: `saveFragmentInstanceState()`, 返回值得到的状态可以用Fragment的`setInitialSavedState()`方法设置给新的Fragment实例, 作为初始状态.
3. FragmentManager的`moveToState()`方法中, 当状态正向创建到`CREATED`时, Fragment自己会恢复View的状态.

这三个入口分别对应的情况是:
入口1对应系统销毁和重建新实例.
入口2对应用户自定义销毁和创建新Fragment实例的状态传递.
入口3对应同一Fragment实例自身的View状态重建.

## Fragment状态保存恢复和Activity的联系

这里对应的是入口1的情况.
当Activity在做状态保存和恢复的时候, 在它其中的fragment自然也需要做状态保存和恢复.
所以Fragment的onSaveInstanceState()在activity调用onSaveInstanceState()的时候一定会发生.
同样的, 如果Fragment中有一些成员变量的值在此时需要保存, [也可以用@State标记](mailto:%E4%B9%9F%E5%8F%AF%E4%BB%A5%E7%94%A8@State%E6%A0%87%E8%AE%B0), 处理方法和上面一样.
也即, 在Activity需要保存状态的时候, 其中的Fragments的**实例状态**自动被处理保存.

## Fragment同一实例的View状态恢复

这里对应的是入口3的情况.
前面介绍过, activity在保存状态的时候, 会将所有View和Fragment的状态都保存起来等待重建的时候使用.
但是如果是单个Activity对应多个Fragments的架构, Activity永远是resume状态, 多个Fragments在切换的过程中, 没有activity的帮助, 如何保存自己的状态?

首先, 取决于你的多个Fragments是如何初始化的.
我做了一个实验, 在activity的onCreate()里面初始化两个Fragment:

```
private void initFragments() {
    tab1Fragment = getFragmentManager().findFragmentByTag(Tab1Fragment.TAG);
    if (tab1Fragment == null) {
        tab1Fragment = new Tab1Fragment();
    }
    tab2Fragment = getFragmentManager().findFragmentByTag(Tab2Fragment.TAG);
    if (tab2Fragment == null) {
        tab2Fragment = new Tab2Fragment();
    }
}
```

然后点击两个按钮来切换它们, replace(), 并且不加入到back stack中:

```
@OnClick(R.id.tab1)
void onTab1Clicked() {
    getFragmentManager().beginTransaction()
            .replace(R.id.content_container, tab1Fragment, Tab1Fragment.TAG)
            .commit();
}

@OnClick(R.id.tab2)
void onTab2Clicked() {
    getFragmentManager().beginTransaction()
            .replace(R.id.content_container, tab2Fragment, Tab2Fragment.TAG)
            .commit();

}
```

可以看到, 每一次的切换, 都是一个Fragment的完全destroy, detach和另一个fragment的attach, create,
但是当我在这两个fragment中各自加上EditText, 发现只要EditText有id, 切换过程中EditText的内容是被保存的.
这是谁在什么时候保存并恢复的呢?
我在TextChange的回调里打了断点, 发现调用栈如下:
![Fragment restore view](http://images2015.cnblogs.com/blog/325852/201606/325852-20160613220310635-370005507.png)
在`FragmentManagerImpl`中, `moveToState()`方法的case Fragment.CREATED中:
调用了: `f.restoreViewState(f.mSavedFragmentState);`
此时我没有做任何保存状态的处理, 但是断点中可以看出:
![Fragment states](http://images2015.cnblogs.com/blog/325852/201606/325852-20160613220336651-1916824950.png)
虽然mSavedFragmentState是null, 但是mSavedViewState却有值.
所以这个View状态保存和恢复对应的入口即是上面两个图中的入口三.

这是因为我的两个fragment只new了一次, 然后保存了成员变量, 即便是Fragment重新onCreate(), 但是对应的实例仍然是同一个.
这和Activity是不同的, 因为你是无法new一个Activity的.

在上面的例子中, 如果不保存Fragment的引用, 每次都new Fragment, 那么View的状态是不会被保存的, 因为不同实例间的状态传递只有在系统销毁恢复的情况下才会发生(入口一).
如果我们需要在不同的实例间传递状态, 就需要用到下面的方法.

## 不同Fragment实例间的状态保存和恢复

这里对应的是入口2, 不同于入口1和3, 它们是自动的, 入口2是用户主动保存和恢复的情形.
自己主动保存Fragment的状态, 可以调用FragmentManager的这个方法:

```
public abstract Fragment.SavedState saveFragmentInstanceState(Fragment f);
```

它的实现是这样的:

```
@Override
public Fragment.SavedState saveFragmentInstanceState(Fragment fragment) {
    if (fragment.mIndex < 0) {
        throwException(new IllegalStateException("Fragment " + fragment
                + " is not currently in the FragmentManager"));
    }
    if (fragment.mState > Fragment.INITIALIZING) {
        Bundle result = saveFragmentBasicState(fragment);
        return result != null ? new Fragment.SavedState(result) : null;
    }
    return null;
}
```

返回的数据类型是: Fragment.SavedState, 这个state可以通过Fragment的这个方法设置给自己:

```
public void setInitialSavedState(SavedState state) {
    if (mIndex >= 0) {
        throw new IllegalStateException("Fragment already active");
    }
    mSavedFragmentState = state != null && state.mState != null
            ? state.mState : null;
}
```

但是注意只能在Fragment被加入之前设置, 这是一个初始状态.
利用这两个方法可以更加自由地保存和恢复状态, 而不依赖于Activity.
这样处理以后, 不必保存Fragment的引用, 每次切换的时候虽然都new了新的实例, 但是旧的实例的状态可以设置给新实例.

例子代码:

```
@State
SparseArray<Fragment.SavedState> savedStateSparseArray = new SparseArray<>();

void onTab1Clicked() {
    // save current tab
    Fragment tab2Fragment = getSupportFragmentManager().findFragmentByTag(Tab2Fragment.TAG);
    if (tab2Fragment != null) {
        saveFragmentState(1, tab2Fragment);
    }

    // restore last state
    Tab1Fragment tab1Fragment = new Tab1Fragment();
    restoreFragmentState(0, tab1Fragment);

    // show new tab
    getSupportFragmentManager().beginTransaction()
            .replace(R.id.content_container, tab1Fragment, Tab1Fragment.TAG)
            .commit();
}

private void saveFragmentState(int index, Fragment fragment) {
    Fragment.SavedState savedState = getSupportFragmentManager().saveFragmentInstanceState(fragment);
    savedStateSparseArray.put(index, savedState);
}

private void restoreFragmentState(int index, Fragment fragment) {
    Fragment.SavedState savedState = savedStateSparseArray.get(index);
    fragment.setInitialSavedState(savedState);
}
```

注意这里用了SparseArray来存储Fragment的状态, 并且加上了`@State`, 这样在Activity重建的时候其中的内容也能够被恢复.

## Back stack中的fragment

有一点很特殊的是, 当Fragment从back stack中返回, 实际上是经历了一次View的销毁和重建, 但是它本身并没有被重建.
即View状态需要重建, 实例状态不需要重建.

举个例子说明这种情形: Fragment被另一个Fragment replace(), 并且压入back stack中, 此时它的View是被销毁的, 但是它本身并没有被销毁.
也即, 它走到了onDestroyView(), 却没有走`onDestroy()`和`onDetact()`.
等back回来的时候, 它的view会被重建, 重新从onCreateView()开始走生命周期.
在这整个过程中, 该Fragment中的成员变量是保持不变的, 只有View会被重新创建.
在这个过程中, instance state的saving并没有发生.

**所以, 很多时候Fragment还需要考虑的是在没有Activity帮助的情形下(Activity并没有可能重建的情形), 自身View状态的保存.**
此时要注意一些不容易发现的错误, 比如List的新实例需要重新setAdapter等.

## Fragment setRetainInstance

Fragment有一个相关方法:
[setRetainInstance](https://developer.android.com/reference/android/app/Fragment.html#setRetainInstance(boolean))
这个方法设置为true的时候表示, 即便activity重建了, 但是fragment的实例并不被重建.
注意此方法只对没有放在back stack中的fragment生效.
什么时候要用这个方法呢? 处理configuration change的时候:
[Handling Configuration Changes with Fragments](http://www.androiddesignpatterns.com/2013/04/retaining-objects-across-config-changes.html)
这样, 当屏幕旋转, Activity重建, 但是其中的fragment和fragment正在执行的任务不必重建.
更多解释可以参见:
[http://stackoverflow.com/questions/11182180/understanding-fragments-setretaininstanceboolean](http://stackoverflow.com/questions/11182180/understanding-fragments-setretaininstanceboolean)
[http://stackoverflow.com/questions/11160412/why-use-fragmentsetretaininstanceboolean](http://stackoverflow.com/questions/11160412/why-use-fragmentsetretaininstanceboolean)

注意这个方法只是针对**configuration change**, 并不影响用户主动关闭和系统销毁的情况:
当activity被用户主动finish, 其中的所有fragments仍然会被销毁.
当activity不在最顶端, memory不够了, 系统仍然可能会销毁activity和其中的fragments.

# View的状态保存和恢复

View的状态保存和恢复主要是依赖于下面几个方法:
保存: `saveHierarchyState()` -> `dispatchSaveInstanceState()` -> `onSaveInstanceState()`
恢复: `restoreHierarchyState()` -> `dispatchRestoreInstanceState()` -> `onRestoreInstanceState()`
还有两个重要的前提条件是View要有id, 并且`setSavedEnabled()`为true.(这个值默认为true).
在系统的widget里(比如TextView, EditText, Checkbox等), 这些都是已经被处理好的, 我们只需要给View赋予id, Activity和Fragment重建的时候会自动恢复其中的状态. (这里的Fragment恢复对应入口一和入口三, 入口二属于跨实例新建的情况).

但是如果你要使用第三方的自定义View, 就需要确认一下它们内部是否有状态保存和恢复的代码.
如果不行你就需要继承该自定义View, 然后实现这两个方法:

```
//
// Assumes that SomeSmartButton is a 3rd Party view that
// View State Saving/Restoring are not implemented internally
//
public class SomeBetterSmartButton extends SomeSmartButton {

    ...

    @Override
    public Parcelable onSaveInstanceState() {
        Bundle bundle = new Bundle();
        // Save current View's state here
        return bundle;
    }

    @Override
    public void onRestoreInstanceState(Parcelable state) {
        super.onRestoreInstanceState(state);
        // Restore View's state here
    }

    ...

}
```

# WebView的状态保存和恢复

WebView的状态保存和恢复不像其他原生View一样是自动完成的.
WebView不是继承自View的.
如果我们把WebView放在布局里, 不加处理, 那么Activity或Fragment重建的过程中, WebView的状态就会丢失, 变成初始状态.

在Fragment的onSaveInstanceState()里面可以加入如下代码来保存WebView的状态:

```
@Override
public void onSaveInstanceState(Bundle outState) {
    super.onSaveInstanceState(outState);
    webView.saveState(outState);
}
```

然后在初始化的时候, 增加判断, 不必每次都打开初始链接:

```
if (savedInstanceState != null) {
    webView.restoreState(savedInstanceState);
} else {
    webView.loadUrl(TEST_URL);
}
```

这样处理以后, 在重新建立的时候, WebView的状态就能恢复到离开前的页面.
不论WebView是放在Activity里还是Fragment里, 这个方法都适用.

但是Fragment还有另一种情况, 即Fragment被压入back stack, 此时它没有被destroy(), 所以没有调用onSavedInstanceState()这个方法.
这种情况返回的时候, 会从onCreateView()开始, 并且savedInstanceState为null, 于是其中WebView之前的状态在此时丢失了.
解决这种情况可以利用Fragment实例并未销毁的条件, 增加一个成员变量bundle, 保存WebView的状态, 最终解决如下:

```
private Bundle webViewState;

@Override
public void onViewCreated(View view, Bundle savedInstanceState) {
    super.onViewCreated(view, savedInstanceState);
    ButterKnife.bind(this, view);

    initWebView();
    if (webViewState != null) {
        //Fragment实例并未被销毁, 重新create view
        webView.restoreState(webViewState);
    } else if (savedInstanceState != null) {
        //Fragment实例被销毁重建
        webView.restoreState(savedInstanceState);
    } else {
        //全新Fragment
        webView.loadUrl(TEST_URL);
    }
}

@Override
public void onPause() {
    super.onPause();
    webView.onPause();

    //Fragment不被销毁(Fragment被加入back stack)的情况下, 依靠Fragment中的成员变量保存WebView状态
    webViewState = new Bundle();
    webView.saveState(webViewState);
}

@Override
public void onSaveInstanceState(Bundle outState) {
    super.onSaveInstanceState(outState);
    //Fragment被销毁的情况, 依靠outState保存WebView状态
    if (webView != null) {
        webView.saveState(outState);
    }
}
```

本文完整例子相关实验代码可见:
[HelloActivityAndFragment](https://github.com/mengdd/HelloActivityAndFragment)
中的State Restore Demo.

本文地址: [Android Fragment使用(三) Activity, Fragment, WebView的状态保存和恢复](http://www.cnblogs.com/mengdd/p/5582244.html)

# 参考资料

Developer Android:
[Android Fragment Reference](https://developer.android.com/reference/android/app/Fragment.html)
[Android FragmentManager Reference](https://developer.android.com/reference/android/app/FragmentManager.html)

Posts:
[Recreating an Activity](https://developer.android.com/training/basics/activity-lifecycle/recreating.html)
[Activity的重新创建](http://www.cnblogs.com/mengdd/archive/2012/12/17/2822291.html)
[从源码角度剖析Fragment核心知识点](http://www.jianshu.com/p/180d2cc0feb5)
[Fragment源码阅读笔记](http://www.jianshu.com/p/bd4a8be309c8)
[The Real Best Practices to Save/Restore Activity's and Fragment's state](https://inthecheesefactory.com/blog/fragment-state-saving-best-practices/en)
[Android中保存和恢复Fragment状态的最好方法](http://jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0327/2648.html)

[Handling Configuration Changes with Fragments](http://www.androiddesignpatterns.com/2013/04/retaining-objects-across-config-changes.html)
[Saving Android View state correctly](http://trickyandroid.com/saving-android-view-state-correctly/)

Tools:
[icepick](https://github.com/frankiesardo/icepick)