Fragment系列文章：
1、[Fragment全解析系列（一）：那些年踩过的坑](http://www.jianshu.com/p/d9143a92ad94)
2、Fragment全解析系列（二）：正确的使用姿势
3、[Fragment之我的解决方案：Fragmentation](http://www.jianshu.com/p/38f7994faa6b)

本篇主要介绍一些Fragment使用技巧。

------

Fragment是可以让你的app纵享丝滑的设计，如果你的app想在现在基础上**性能大幅度提高**，并且**占用内存降低**，同样的界面Activity占用内存比Fragment要多，响应速度Fragment比Activty在中低端手机上快了很多，甚至能达到好几倍！如果你的app当前或以后有**移植**平板等平台时，可以让你节省大量时间和精力。

------

*简陋的目录*
**1、一些使用建议**
**2、add(), show(), hide(), replace()的那点事**
**3、关于FragmentManager你需要知道的**
**4、使用FragmentPagerAdapter+ViewPager的注意事项**
**5、是使用单Activity＋多Fragment的架构，还是多模块Activity＋多Fragment的架构？**

------

作为一个稳定的app，从后台且回到前台，一定会在任何情况都能恢复到离开前的页面，并且保证数据的完整性。

如果你没看过本系列的[第一篇](http://www.jianshu.com/p/fd71d65f0ec6)，为了方便后面文章的介绍，先规定一个“术语”，安卓app有一种特殊情况，就是 app运行在后台的时候，系统资源紧张的时候导致把app的资源全部回收（杀死app的进程），这时把app再从后台返回到前台时，app会重启。这种情况下文简称为：“内存重启”。（屏幕旋转等配置变化也会造成当前Activity重启，本质与“内存重启”类似）

# 1、一些使用建议

1、对Fragment传递数据，建议使用`setArguments(Bundle args)`，而后在`onCreate`中使用`getArguments()`取出，在 “内存重启”前，系统会帮你保存数据，不会造成数据的丢失。和Activity的Intent恢复机制类似。

2、使用`newInstance(参数)` 创建Fragment对象，优点是调用者只需要关系传递的哪些数据，而无需关心传递数据的Key是什么。

3、如果你需要在Fragment中用到宿主Activity对象，建议在你的基类Fragment定义一个Activity的全局变量，在`onAttach`中初始化，这不是最好的解决办法，但这可以有效避免一些意外Crash。详细原因参考[第一篇](http://www.jianshu.com/p/d9143a92ad94)的“getActivity()空指针”部分。

```
protected Activity mActivity;
@Override
public void onAttach(Activity activity) {
    super.onAttach(activity);
    this.mActivity = activity;
}
```

# 2、add(), show(), hide(), replace()的那点事

1、区别

```
show()
```

，

```
hide()
```

最终是让Fragment的View 

```
setVisibility
```

(true还是false)，不会调用生命周期；

`replace()`的话会销毁视图，即调用onDestoryView、onCreateView等一系列生命周期；

`add()`和 `replace()`不要在同一个阶级的FragmentManager里混搭使用。

**2、使用场景**
如果你有一个很高的概率会再次使用当前的Fragment，建议使用`show()`，`hide()`，可以提高性能。

在我使用Fragment过程中，大部分情况下都是用`show()`，`hide()`，而不是`replace()`。

注意：如果你的app有大量图片，这时更好的方式可能是replace，配合你的图片框架在Fragment视图销毁时，回收其图片所占的内存。

**3、onHiddenChanged的回调时机**
当使用`add()`+`show()，hide()`跳转新的Fragment时，旧的Fragment回调`onHiddenChanged()`，不会回调`onStop()`等生命周期方法，而新的Fragment在创建时是不会回调`onHiddenChanged()`，这点要切记。

**4、Fragment重叠问题**
使用`show()`，`hide()`带来的一个问题就是，如果你不做任何额外处理，在“内存重启”后，Fragment会重叠；（该BUG在support-v4 24.0.0+以上 官方已修复）

有些小伙伴可能就是为了避免Fragment重叠问题，而选择使用`replace()`，但是使用`show()`，`hide()`时，重叠问题很简单解决的：

- 如果你在用24.0.0+的版本，需要特殊处理，官方已经修复该BUG；
- 如果你在使用小于24.0.0以下的v4包，可以参考[9行代码让你App内的Fragment对重叠说再见](http://www.jianshu.com/p/c12a98a36b2b)

# 3、关于FragmentManager你需要知道的

FragmentManager栈视图:

（1）每个Fragment以及宿主Activity(继承自FragmentActivity)都会在创建时，初始化一个FragmentManager对象，处理好Fragment嵌套问题的关键，就是理清这些不同阶级的栈视图。

下面给出一个简要的**关系图**：

栈关系图.png

（2）对于宿主Activity，`getSupportFragmentManager()`获取的FragmentActivity的FragmentManager对象;

对于Fragment，`getFragmentManager()`是获取的是父Fragment(如果没有，则是FragmentActivity)的FragmentManager对象，而`getChildFragmentManager()`是获取自己的FragmentManager对象。

# 4、使用FragmentPagerAdapter+ViewPager的注意事项

- 使用FragmentPagerAdapter+ViewPager时，切换回上一个Fragment页面时（已经初始化完毕），不会回调任何生命周期方法以及`onHiddenChanged()`，只有`setUserVisibleHint(boolean isVisibleToUser)`会被回调，所以如果你想进行一些懒加载，需要在这里处理。
- 在给ViewPager绑定FragmentPagerAdapter时，
  `new FragmentPagerAdapter(fragmentManager)`的FragmentManager，一定要保证正确，如果ViewPager是Activity内的控件，则传递`getSupportFragmentManager()`，如果是Fragment的控件中，则应该传递`getChildFragmentManager()`。只要记住ViewPager内的Fragments是当前组件的子Fragment这个原则即可。
- 你不需要考虑在“内存重启”的情况下，去恢复的Fragments的问题，因为FragmentPagerAdapter已经帮我们处理啦。

# 5、是使用单Activity＋多Fragment的架构，还是多模块Activity＋多Fragment的架构？

单Activity＋多Fragment：

一个app仅有一个Activity，界面皆是Frament，Activity作为app容器使用。

优点：性能高，速度最快。参考：新版知乎 、google系app

缺点：逻辑比较复杂，尤其当Fragment之间联动较多或者嵌套较深时，比较复杂。

**多模块Activity＋多Fragment：**
一个模块用一个Activity，比如
1、登录注册流程：
LoginActivity + 登录Fragment + 注册Fragment + 填写信息Fragment ＋ 忘记密码Fragment
2、或者常见的数据展示流程：
DataActivity + 数据列表Fragment + 数据详情Fragment ＋ ...

优点：速度快，相比较单Activity+多Fragment，更易维护。

**我的观点：**
权衡利弊，我认为多模块Activity＋多Fragment是最合适的架构，开发起来不是很复杂，app的性能又很高效。

当然。**Fragment只是官方提供的灵活组件，请优先遵从你的项目设计！**真的特别复杂的界面，或者单个Activity就可以完成一个流程的界面，使用Activity可能是更好的方案。

# 最后

如果你读完了[第一篇](http://www.jianshu.com/p/d9143a92ad94)和这篇文章，那么我相信你使用多模块Activity+多Fragment的架构所遇到的坑，大部分都应该能找到解决办法。

但是如果流程较为复杂，比如Fragment A需要启动一个新的Fragment B并且关闭当前A,或者A启动B，B在获取数据后，想在返回到A时把数据交给A（类似Activity的`startActivityForResult`），又或者你保证在Fragment转场动画的情况下，使用`pop(tag\id)`从栈内退出多个Fragment，或者你甚至想Fragment有一个类似Activity的`SingleTask`启动模式，那么你可以参考[下一篇](http://www.jianshu.com/p/38f7994faa6b)，我的解决方案库，[Fragmentation](https://github.com/YoKeyword/Fragmentation)。它甚至提供了一个让你在开发时，可以随时查看所有阶级的栈视图的UI界面。