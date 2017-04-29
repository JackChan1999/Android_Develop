# Toolbar作为ActionBar使用介绍

本文介绍了在Android中将Toolbar作为ActionBar使用的方法.
并且介绍了在Fragment和嵌套Fragment中使用Toolbar作为ActionBar使用时需要注意的事项.

## 使用support library的Toolbar

Android的ActionBar每个版本都会做一些改变, 所以原生的ActionBar在不同的系统上看起来可能会不一样.
使用support library版本的[Toolbar](https://developer.android.com/reference/android/support/v7/widget/Toolbar.html)可以让你的应用在多种设备类型上保持一致. support library中总是包含了最新的features.
Android从5.0 (API Level 21)开始提供[Material Design](https://developer.android.com/design/material/index.html), 使用v7版本的Toolbar后, 在任何Android 2.1(API Level 7)以上的机器上都可以看到Material Design风格的Toolbar.

## 在Activity中使用Toolbar

1.首先项目gradle中添加:

```
compile 'com.android.support:appcompat-v7:25.3.0'
```

2.确保Activity继承`AppCompatActivity`
3.在application设置中使用NoActionBar的主题:

```xml
<application
    android:theme="@style/Theme.AppCompat.Light.NoActionBar"
    />
```

4.把Toolbar写在布局中

```xml
<android.support.v7.widget.Toolbar
   android:id="@+id/my_toolbar"
   android:layout_width="match_parent"
   android:layout_height="?attr/actionBarSize"
   android:background="?attr/colorPrimary"
   android:elevation="4dp"
   android:theme="@style/ThemeOverlay.AppCompat.ActionBar"
   app:popupTheme="@style/ThemeOverlay.AppCompat.Light"/>
```

5.在Activity里面把Toolbar设置成为ActionBar
首先把Toolbar find出来, 然后调用[setSupportActionBar方法](https://developer.android.com/reference/android/support/v7/app/AppCompatActivity.html#setSupportActionBar(android.support.v7.widget.Toolbar))
把Toolbar设置为自己的ActionBar即可.

```java
public class ToolbarDemoActivity extends AppCompatActivity {

    @BindView(R.id.toolbar)
    Toolbar toolbar;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_toolbar_demo);
        ButterKnife.bind(this);
        setSupportActionBar(toolbar);
    }
}
```

然后就可以随意使用啦, 用[getSupportActionBar](https://developer.android.com/reference/android/support/v7/app/AppCompatActivity.html#getSupportActionBar())可以获取ActionBar类型的对象, 从而使用[ActionBar](https://developer.android.com/reference/android/support/v7/app/ActionBar.html)的方法.

### 添加Action Buttons

定义menu:

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <item
        android:id="@+id/action_android"
        android:icon="@drawable/ic_android_black_24dp"
        android:title="@string/action_android"
        app:showAsAction="always" />
    <item
        android:id="@+id/action_favourite"
        android:icon="@drawable/ic_favorite_black_24dp"
        android:title="@string/action_favourite"
        app:showAsAction="ifRoom" />
    <item
        android:id="@+id/action_settings"
        android:title="@string/action_settings"
        app:showAsAction="never" />
</menu>
```

然后在代码中inflate和处理它的点击事件:

```java
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    Log.i(TAG, "onCreateOptionsMenu()");
    getMenuInflater().inflate(R.menu.menu_activity_main, menu);
    return super.onCreateOptionsMenu(menu);
}

@Override
public boolean onOptionsItemSelected(MenuItem item) {
    switch (item.getItemId()) {
        case R.id.action_android:
            Log.i(TAG, "action android selected");
            return true;
        case R.id.action_favourite:
            Log.i(TAG, "action favourite selected");
            return true;
        case R.id.action_settings:
            Log.i(TAG, "action settings selected");
            return true;
        default:
            return super.onOptionsItemSelected(item);
    }
}
```

### 添加向上返回的action

添加向上返回parent的action:

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_toolbar_demo);
    ButterKnife.bind(this);
    setSupportActionBar(toolbar);

    // add a left arrow to back to parent activity,
    // no need to handle action selected event, this is handled by super
    getSupportActionBar().setDisplayHomeAsUpEnabled(true);
}
```

然后只需要在manifest中指定parent:

```xml
<activity
    android:name=".toolbar.ToolbarDemoActivity"
    android:parentActivityName=".MainActivity"></activity>
```

## 在Fragment中使用Toolbar

在Fragment中使用Toolbar的步骤和Activity差不多.
在Fragment布局中添加一个Toolbar, 然后find它, 然后调用Activity的方法来把它设置成ActionBar:

```
((AppCompatActivity) getActivity()).setSupportActionBar(toolbar);
```

注意此处有一个强转, 必须是AppCompatActivity才有这个方法.
但是此时运行到Fragment之后, 发现Toolbar上的文字和按钮全是Activity传过来的, 这是因为只有Activity的`onCreateOptionsMenu()`被调用了, 但是Fragment的并没有被调用.
在Fragment中加上这句:

```
setHasOptionsMenu(true);
```

此时Fragment的`onCreateOptionsMenu()`回调会被调到了, 但是inflate出的按钮和Activity中的actions加在一起显示出来了.
因为Activity的`onCreateOptionsMenu()`会在之前调用到.
于是Fragment中的写成这样:

```
@Override
public void onCreateOptionsMenu(Menu menu, MenuInflater inflater) {
    Log.e(TAG, "onCreateOptionsMenu()");
    menu.clear();
    inflater.inflate(R.menu.menu_parent_fragment, menu);
}
```

即先clear()一下, 这样按钮就只有Fragment中设置的自己的了, 不会有Activity中的按钮.

## 在嵌套的子Fragment中使用Toolbar

前面已经介绍过, Fragment可以嵌套使用: [Android Fragment使用(二) 嵌套Fragments (Nested Fragments) 的使用及常见错误](http://www.cnblogs.com/mengdd/p/5552721.html).
那么在前面的Fragment中再显示一个子Fragment, 并且又带有一个不一样的Toolbar, 还需要哪些处理呢?
首先, java代码中还是需要有:

```
setHasOptionsMenu(true)
((AppCompatActivity) getActivity()).setSupportActionBar(toolbar);
```

然后根据是否需要菜单按钮, 覆写onCreateOptionsMenu()方法来inflate自己的menu文件即可.
感觉和在普通的Fragment中使用Toolbar作为ActionBar并没有什么区别.
但是如果你的多个Fragment有不同的Toolbar菜单选项, 如果你没有懂得其中的原理, 可能就会出现一些混乱.
下面来解说一下相关的方法.

### onCreateOptionsMenu()方法的调用

一旦调用

```
((AppCompatActivity) getActivity()).setSupportActionBar(toolbar);
```

就会导致Activity`onCreateOptionsMenu()`方法的调用, 而Activity会根据其中Fragment是否设置了setHasOptionsMenu(true)来调用Fragment的`onCreateOptionsMenu()`方法, 调用顺序是树形的, 按层级调用, 中间如果有false则跳过.

假设当前Activity, Parent Fragment和Child Fragment中都设置了自己的Toolbar为ActionBar.
在打开Child fragment的时候, `onCreateOptionsMenu()`的调用顺序是.
`Activity -> Parent -> Child.` 此时parent和child fragment都设置了setHasOptionsMenu(true).

关于这个, 还有以下几种情况:

- 如果Parent的`setHasOptionsMenu(false)`, Child为true, 则Parent的`onCreateOptionsMenu()`不会调用, 打开Child的时候Activity -> Child.
- 如果Child的`setHasOptionsMenu(false)`, Parent为true, 则打开Child的时候仍然会调用Activity和Parent的onCreateOptionsMenu()方法.
- 如果Parent和Child都置为false, 打开Parent和Child Fragment的时候都会调用Activity的onCreateOptionsMenu()方法.

**仅仅是child Fragment的show() hide()的切换, activity和parent Fragment的onCreateOptionsMenu()也会重新进入.**
这一点我还没有想明白, 是项目中遇到的, 初步推测可能是menu的显隐变化invalidate了menu, 改天有空再试试.

上面的机制常常是导致Toolbar上面的按钮混淆错乱的原因.
举个例子:
如果我们现在Activity和Parent Fragment有不同的Toolbar按钮, 但是Child只有文字, 没有按钮.
很显然我们不需要给child写menu文件, 也不需要覆写child里的`onCreateOptionsMenu()`方法.
但是此时不管怎样, parent的`onCreateOptionsMenu()`方法都会被调用, 这样我们打开child的时候, toolbar上就神奇地出现了parent里的按钮.
这种情况如何解决呢?
可以在parent中加一个条件, 当没有child fragment的时候才做inflate的工作:

```
@Override
public void onCreateOptionsMenu(Menu menu, MenuInflater inflater) {
    Log.e(TAG, "onCreateOptionsMenu()");
    menu.clear();
    if (getChildFragmentManager().getBackStackEntryCount() == 0) {
        inflater.inflate(R.menu.menu_parent_fragment, menu);
    }
}
```

另外, 除了`setSupportActionBar()`之外, 如果我们想**主动触发** `onCreateOptionsMenu()`方法的调用, 可以用
`invalidateOptionsMenu()`方法.

### onOptionsItemSelected()方法的调用

在Activity和其中的Fragment都有options menu的时候, 需要注意menu item的id不要重复.
以为点击事件的分发也是从Activity开始分发下去的, 如果child fragment中有个选项的id和Activity中一个选项的id重复了, 则在Activity中就会将其处理, 不会继续分发.

### 有嵌套Fragment时 Back键处理

之前没有嵌套Fragment的情况下, 只要将Fragment加入到Back Stack中, 那么按下Back键的时候pop动作是系统自动做好的.
虽然在添加child fragment的时候将其加入到back stack中, 但是按back键的时候仍然是将parent fragment弹出, 只剩下Activity.
这是因为back键只检查第一层Fragment的back stack, 对于child fragment, 需要在其parent中自己处理.
比如这样处理:

在Activity中

```
@Override
public void onBackPressed() {
    Fragment fragment = getSupportFragmentManager().findFragmentById(android.R.id.content);
    if (fragment instanceof ToolbarFragment) {
        if (((ToolbarFragment) fragment).onBackPressed()) {
            return;
        }
    }
    super.onBackPressed();
}
```

其中ToolbarFragment是直接加在Activity中作为parent fragment的.
在parent fragment中(即ToolbarFragment中):

```
public boolean onBackPressed() {
    return getChildFragmentManager().popBackStackImmediate();
}
```

本文Demo地址: [Demo on github](https://github.com/mengdd/HelloActivityAndFragment)
其中的: ToolbarDemoActivity即为Toolbar Demo.
本文地址: [Android Fragment使用(四) Toolbar使用及Fragment中的Toolbar处理](http://www.cnblogs.com/mengdd/p/5590634.html)

# 参考资料

Developer Android:
[Training AppBar](https://developer.android.com/training/appbar/index.html)
[v7.widget.Toolbar Reference](https://developer.android.com/reference/android/support/v7/widget/Toolbar.html)
[v7.app.ActionBar](https://developer.android.com/reference/android/support/v7/app/ActionBar.html)

[Guides: action bar menu items and fragments](https://guides.codepath.com/android/Creating-and-Using-Fragments#actionbar-menu-items-and-fragments)