翻译原文出处：[Android’s matryoshka problem]()

[Android](http://lib.csdn.net/base/android)有碎片化的问题，当然本文说的碎片化不是指的系统版本碎片化的问题，而是Fragment组件碎片化的问题。

很久之前，在Android 3.1系统发布的时候，Google推出了使用Fragment来更加容易地开发平板和手机应用，虽然Activity还是页面结构的主体，但是却可以在其基础上使用多个Fragment来构建页面，这些Fragment都是有各自的生命周期的。

最常见的是列表和详情页面使用Fragment，如果在手机设备上，这个两个一般都是在独立的Activity页面中，但是在平板上这两个Fragment往往都是嵌套在一个Activity中。

当然，在开发过程中，正常情况下都是没有问题的。

**1、特殊的Fragment**

如果想要使用Fragment来开发应用并且适配低版本系统，必须要使用Google提供的Support Library（V4）.

Support Library这个兼容库设计地有些莫名其妙，它为了提供向后兼容的特性，替换了整个Fragment框架。比如，运行在3.1之后的系统系统上，使用的Fragment还是Support Library所提供的，而不是基于系统自身的（这一点在讲到后面的时候非常重要）。

**2、Fragment的嵌套**

使用Fragment时最繁琐复杂的就是多个Fragment之间的相互通信，必须通过Activity作为中间者传递。嵌套的Fragment一开始是不支持的，因为会导致了各式各样的bug。直到API 17，也就是Jelly Bean 4.2，终于开始支持嵌套的Fragments，并且这个功能也被添加到了Support Library里面。使用Fragment来搭建页面的梦想实现的一天终于到来了，这种方式有一个巨大的好处，就是解放Activity，使用多个Fragment组件来承载UI和逻辑。

梦想很美好，现实很残酷！

**3、Fragment嵌套BUG之一：突变的动画效果**

**问题：** 交互体验做到极致的APP，都会使UI具有平滑顺畅的动画效果。FragmentManager是允许通过设置转场过渡动画的。但是，退出动画会导致嵌套的Fragment在动画刚刚开始时就瞬间消失。

**原因：** Fragments有一个嵌套的生命周期，导致嵌套的Fragment会在其宿主Fragment前执行相应的生命周期，比如onStop。由于宿主Fragment的FragmentManager无法识别嵌套的Fragment，在动画开始执行的时候，嵌套的Fragment的视图树会直接跳过动画阶段，但是宿主Fragment的动画却还在执行。所以宿主Fragment和嵌套Fragment动画的步调是完全不一致的。

**解决：** 参考Stack Overflow上的一个解决方案：[http://stackoverflow.com/questions/14900738/nested-fragments-disappear-during-transition-animation](http://stackoverflow.com/questions/14900738/nested-fragments-disappear-during-transition-animation) 原理是缓存宿主Fragment的当前可见状态，但是这个会导致页面重绘，可能衍生出其它的问题。

**4、Fragment嵌套BUG之一：被继承的setRetainInstance**

Fragments可以设置成保持状态。比如，当屏幕旋转导致Activity销毁和重启时，可以不用重新创建Fragment。

**问题：** 嵌套的Fragment会继承宿主Fragment的retain instance状态。

**原因：** 不明

**解决：** 尚无解决方案。

这个看起来是个很小的点，但是却可能产生很大的问题。虽然个人倾向于让所有fragments重新创建来保证其状态不出错（尤其是有复杂View的场景下），但是如果遇到不存在或简单View的场景是，比如网络请求或者多个组件调用，可能会设置一个回调监听器，而这个监听器是不需要重复创建的。上面所说的这种Fragment如果被嵌套在一个需要重新创建的Fragment里面，由于setRetainInstance 的继承性，会导致这个Fragment也跟着被重新创建。我的解决方式是使用静态实例和弱引用来持有这个Fragment，保证其不需要重新创建，有点坑。。。

**5、Fragment嵌套BUG之一：错乱的onActivityResult传递**

这是最让人头疼的问题了，而且我们会经常遇到，比如在嵌套的Fragment里面启动Activity。

**问题：** onActivityResult回调不会走到嵌套的Fragment里面。

**原因：** Support Library（V4）会修改了requestCode，使其中包含了一个Fragment 16位的索引值。这个索引值是与FragmentManager相关联的，Activity会根据这个索引值在自身的FragmentManager里面搜索Fragment来分发onActivityResult，但是只能搜寻到宿主Fragment，而宿主Fragment却不会向其内部嵌套的Fragment分发。这样就导致嵌套的Fragment永远收不到onActivityResult回调。

**解决：** 宿主Fragment向其内部嵌套的Fragment发送onActivityResult回调。

**补充：** 使用系统自带的Fragment不会出现这种问题，谷歌还是很牛逼的哈！

------

[测试](http://lib.csdn.net/base/softwaretest)源码仓库：[https://github.com/BurntBrunch/android-fragment-bugs](https://github.com/BurntBrunch/android-fragment-bugs)

结束，谢谢观赏！