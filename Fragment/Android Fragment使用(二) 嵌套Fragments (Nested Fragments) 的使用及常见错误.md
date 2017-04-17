# 嵌套Fragment的使用及常见错误

嵌套Fragments (Nested Fragments), 是在Fragment内部又添加Fragment.
使用时, 主要要依靠宿主Fragment的 `getChildFragmentManager()` 来获取FragmentManger.
虽然看起来和在activity中添加fragment差不多, 但因为fragment生命周期及管理恢复模式不同, 其中有一些需要特别注意的地方.
本文内容还包括了从Fragment迁移到v4.Fragment代码中需要改动的一些地方.

## 嵌套Fragments

嵌套Fragments [Nested Fragments](https://developer.android.com/about/versions/android-4.2.html#NestedFragments) 是Android 4.2 API 17 引入的.
目的: 进一步增强动态复用.
如果要在Android 4.2之前使用, 可以用support library v4的版本, 后面会有详细的迁移过程介绍.

### 嵌套Fragment的动态添加

在宿主fragment里调用[getChildFragmentManager()](https://developer.android.com/reference/android/app/Fragment.html#getChildFragmentManager())
即可用它来向这个fragment内部添加fragments.

```
Fragment videoFragment = new VideoPlayerFragment();
FragmentTransaction transaction = getChildFragmentManager().beginTransaction();
transaction.add(R.id.video_fragment, videoFragment).commit();
```

同样, 对于内部的fragment来说, [getParentFragment()](https://developer.android.com/reference/android/app/Fragment.html#getParentFragment()) 方法可以获取到fragment的宿主fragment.

### getChildFragmentManager() 和 getFragmentManager()

`getChildFragmentManager()`是fragment中的方法, 返回的是管理当前fragment内部子fragments的manager.
`getFragmentManager()`在activity和fragment中都有.
在activity中, 如果用的是v4 support库, 方法应该用`getSupportFragmentManager()`, 返回的是管理activity中fragments的manager.
在fragment中, 还叫getFragmentManager(), 返回的是把自己加进来的那个manager.

也即, 如果fragment在activity中, fragment.getFragmentManager()得到的是activity中管理fragments的那个manager.
如果fragment是嵌套在另一个fragment中, fragment.getFragmentManager()得到的是它的parent的getChildFragmentManager().

总结就是: **getFragmentManager()是本级别管理者, getChildFragmentManager()是下一级别管理者**.
这实际上是一个树形管理结构.

## 使用Support library

### 为什么要使用support library? 有两种原因:

1. 要在API level11之前使用fragment.
2. 要在API Level 17之前使用`getChildFragmentManager()`, 即使用嵌套Fragment.

### 迁移到support library需要改动哪些地方?

把Fragment迁移到v4版本, 需要改动如下地方:

```
import android.app.Fragment; -> import android.support.v4.app.Fragment;
Activity -> FragmentActivity / AppCompatActivity
activity.getFragmentManager() -> getSupportFragmentManager()

Loader, LoaderManager, LoaderCursor也需要改成v4包的.
activity.getLoaderManager() -> getSupportLoaderManager()
```

Fragment中onTrimMemory()方法不见了
以前是这个方法

```
    @Override
    public void onTrimMemory(int level) {
        super.onTrimMemory(level);
        imageLoader.trimMemory(level);
    }
```

v4版本需要改成这个

```
   @Override
    public void onLowMemory() {
        super.onLowMemory();
        imageLoader.trimMemory(ComponentCallbacks2.TRIM_MEMORY_COMPLETE);
    }
```

## 嵌套Fragment使用常见错误

### 错误情形1: 把嵌套Fragment放在布局里

把嵌套Fragment放在布局里 -> `InflateException in Binary XML`

看起来嵌套fragment的使用除了要用`getChildFragmentManager()`以外, 其他跟之前似乎没什么区别.
如果嵌套的fragment不需要太多控制, 固定地占据了一块地方, 你可能想当然地为了省事就把它放进了xml布局文件里, 写个标签.
运行一下初看起来似乎没什么错, run一下也能显示出来, 但是千万不要这样做, 多玩两下更复杂的你就知道了.

上面[官网介绍时](https://developer.android.com/about/versions/android-4.2.html#NestedFragments)就有这么一句:

```
Note: You cannot inflate a layout into a fragment when that layout includes a <fragment>.
Nested fragments are only supported when added to a fragment dynamically.
```

人家这么说肯定是有原因的哇, 下面我来告诉你我知道的问题:
如果Fragment被嵌套写在了布局里, inflate到这个标签的时候就相当于将它加进了FragmentManager里.
如果嵌套的parent fragment因为需要重建View而重新走了`onCreateView()`方法, 再次inflate, 此时就会抛出异常: `InflateException in Binary XML`

之前为什么可以呢? 非嵌套的情况, fragment直接加在activity里, 如果需要重新inflate, 必定是在onCreate()里, activity是重新建的, 所以没有问题, 因为不存在fragmentManager中已经持有同一个fragment的问题.

举一个例子:
在嵌套的情况下, 如果FragmentE布局里有FragmentA, 这时候我们需要叠加一个FragmentD.
用了`replace()`, 并且`addToBackStack()`.
当D显示的时候, E实际上View是被销毁的, 然后back回来, 重建View, 即FragementE需要重新从onCreateView()开始走生命周期, 走到inflate的时候又看到了fragmentA的标签.
但是这时候A实际上还在FragmentManager里面, 所以就会抛出如下的异常:
`android.view.InflateException: Binary XML file line # XX: Binary XML file line #XX: Error inflating class fragment`
崩溃的位置就在parent fragment(FragmentE) inflate的时候.
打印具体的异常栈信息可以看到:

```
at com.example.ddmeng.helloactivityandfragment.fragment.FragmentE.onCreateView(FragmentE.java:35)
at android.app.Fragment.performCreateView(Fragment.java:2220)
at android.app.FragmentManagerImpl.moveToState(FragmentManager.java:973)
at android.app.FragmentManagerImpl.moveToState(FragmentManager.java:1148)
at android.app.FragmentManagerImpl.popBackStackState(FragmentManager.java:1587)
at android.app.FragmentManagerImpl.popBackStackImmediate(FragmentManager.java:578)
at android.support.v4.app.BaseFragmentActivityEclair.onBackPressedNotHandled(BaseFragmentActivityEclair.java:27)
at android.support.v4.app.FragmentActivity.onBackPressed(FragmentActivity.java:189)
 Caused by: java.lang.IllegalArgumentException: Binary XML file line #16: Duplicate id 0x7f0c0059, tag null, or parent id 0xffffffff with another fragment for com.example.ddmeng.helloactivityandfragment.fragment.FragmentA
at android.app.FragmentManagerImpl.onCreateView(FragmentManager.java:2205)
```

[实验例子代码](https://github.com/mengdd/HelloActivityAndFragment)

#### Solution 1: 动态添加child fragment

解决上面的问题有各种方法, 最常规的做法是, 使用动态添加:

```
Fragment fragmentA = getChildFragmentManager().findFragmentByTag(NESTED_FRAGMENT_TAG);
if (fragmentA == null) {
    Log.i(LOG_TAG, "add new FragmentA !!");
    fragmentA = new FragmentA();
    FragmentTransaction fragmentTransaction = getChildFragmentManager().beginTransaction();
    fragmentTransaction.add(R.id.fragment_container, fragmentA, NESTED_FRAGMENT_TAG).commit();
} else {
    Log.i(LOG_TAG, "found existing FragmentA, no need to add it again !!");
}
```

#### Solution 2: 在异常之前remove child fragment

如果你的子fragment非要加在布局里不可, 而你的程序确实会有重建父fragment view的情形.
为了避免上面的异常, 你也可以这样做(tricky and not recommended):

```
public void removeChildFragment(Fragment parentFragment) {
    FragmentManager fragmentManager = parentFragment.getChildFragmentManager();
    Fragment child = fragmentManager.findFragmentById(R.id.child);
    if (child != null) {
        fragmentManager.beginTransaction()
        .remove(child)
        .commitAllowingStateLoss();
    }
}
```

在parentFragment的`onCreateView()`方法中inflate之前和`onSaveInstanceState()`方法中做save工作之前调用它.
这两个地方是发生异常的地方, 只要在其之前remove就好.

### 错误情形2: 把fragment放在一个动态布局里

把fragment放在一个动态布局里 -> `java.lang.IllegalArgumentException: No view found for id`

发现这个错误是因为项目中的一个子Fragment是添加在RecyclerView里面的一块的.
RecyclerView要等到Loader的数据取到了之后再populate每一块的布局.
还是上面的流程, 启动父fragment, load数据, 添加子fragment, 这都没有问题.
但是一旦如果是上面的`replace()`加`addToBackStack()` , 并且再次返回, 就会出现异常.

因为当重建View的时候, fragmentManager其中是持有child fragment的, 但是找不到它的container, 于是就会抛出异常.
我也同样做了一个小实验, 在我的demo程序里:
[HelloActivityAndFragment](https://github.com/mengdd/HelloActivityAndFragment)
Nested Fragment in Dynamic Container:
在Fragment F中, 先添加一个FrameLayout, 再把child fragment A加进去.
然后在Activity中, 用D replace F, 按back键返回, 就会有crash:

```
     java.lang.IllegalArgumentException: No view found for id 0x7f0c0062 (com.example.ddmeng.helloactivityandfragment:id/frame_container) for fragment FragmentA{b37763 #0 id=0x7f0c0062 FragmentA}
         at android.app.FragmentManagerImpl.moveToState(FragmentManager.java:965)
         at android.app.FragmentManagerImpl.moveToState(FragmentManager.java:1148)
         at android.app.FragmentManagerImpl.moveToState(FragmentManager.java:1130)
         at android.app.FragmentManagerImpl.dispatchActivityCreated(FragmentManager.java:1953)
         at android.app.Fragment.performActivityCreated(Fragment.java:2234)
         at android.app.FragmentManagerImpl.moveToState(FragmentManager.java:992)
         at android.app.FragmentManagerImpl.moveToState(FragmentManager.java:1148)
         at android.app.BackStackRecord.popFromBackStack(BackStackRecord.java:1670)
         at android.app.FragmentManagerImpl.popBackStackState(FragmentManager.java:1587)
         at android.app.FragmentManagerImpl.popBackStackImmediate(FragmentManager.java:578)
         at android.app.Activity.onBackPressed(Activity.java:2503)
```

这是因为返回的时候FragmentManager找不到对应的container了.
所以应该避免这种做法, 尽量把fragment加进parent的根布局里, 而不是某个动态添加的布局.

## 其他

关于嵌套fragments的情况, 可能和ViewPager结合使用的情形比较多.
这个感觉说来话长了, 以为有很多系统帮忙做的事情, 改天有空再说吧.

这里有个大哥写了个工具类[Fragmentation](https://github.com/YoKeyword/Fragmentation).
他也有几篇博文分析遇到的坑和原因(见上面repo的README给出的链接), 里面有一些back stack的问题, 还有动画什么的, 大家有兴趣可以看看.

# 参考资料

[Guide: Nested Fragments](https://guides.codepath.com/android/Creating-and-Using-Fragments#nesting-fragments-within-fragments)

[相关Demo](https://github.com/mengdd/HelloActivityAndFragment)