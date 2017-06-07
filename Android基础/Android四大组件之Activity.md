Android有四大组件，分别是：Activity，BroadCastReceiver，Service，ContentProvider，本篇博客介绍的是其中的Activity。先来看下内容摘要

- 什么是Activity
- Activity的跳转
- Activity的生命周期
- Activity的启动模式
- 获取其他Activity 销毁时设置的数据

# **1. Activity 的创建**

Activity 是Android 四大组件之一，用于展示界面。Activity 中所有操作都与用户密切相关，是一个负责与用户交互的组件，它上面可以显示一些控件也可以监听并处理用户的事件。一个Activity 通常就是一个单独的屏幕，Activity 之间通过Intent 进行通信。
## **1.1 如何创建自定义的Activity**

### **1.1.1 自定义类继承Activity**

新建一个Android 工程，在src 目录下新建一个类，不妨起名MainActivity，让该类继承Android SDK提供的Activity。

### **1.1.2 在MyActivity 中覆写onCreate()方法**

onCreate() 方法是在当前Activity 被系统创建的时候调用的， 在该方法中一般要通过setContentView(R.layout.myactivity)方法给当前Activity 绑定布局文件或视图。因此为了让我们的MyActivity 能够展示界面需要在工程目录的res/layout/目录下创建一个布局文件。比如我创建的布局文件名（注意：Android 的资源文件名中是不允许有大写字母的，必须全小写，如果有多个单词可以用下划线分开）为myactivity.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
	xmlns:android="http://schemas.android.com/apk/res/android" 							
	android:layout_width="match_parent"
	android:layout_height="match_parent"
	android:orientation="vertical" >

<TextView
	android:layout_width="match_parent"
	android:layout_height="wrap_content"
	android:text="这是MyActivityA"/>

</LinearLayout>
```
必须要注意的就是在该方法中必须先调用父类的onCreate() 方法， 也就是super.onCreate(savedInstanceState);该句代码必须被调用一次。这是为什么呢？我们看看父类的onCreate()方法里面都干了些啥就知道了。如图1-1 所示，在父类的方法中执行了一堆业务逻辑，而且在最后一行用了一个成员变量mCalled 记录了当前方法是否被调用了。虽然我们现在还不看不同这些代码，但是我们已经知道既然父类帮我们实现了这么功能，那么我们子类就一定必须调用一次父类的该方法。

![这里写图片描述](http://img.blog.csdn.net/20160911134407182)

### **1.1.3在AndroidManifest.xml 中进行Activity 的注册**

Android 中共有四大组件，除了BroadCastReceiver 之外，其他三个组件如果要使用那么必须在AndroidManifest.xml 中进行注册（也叫声明）。

我们在上面创建的MyActivity 如何注册呢？其实如果你不会的话完全可以将eclipse 自动生成的MainActivity 的注册代码给拷贝过来，然后将android:name 的属性值修改成我们的类。

```xml
 <activity
	 android:name="com.itheima.android.activity.MyActivity"
	 android:label="@string/app_name" >
  <intent-filter>
	 <action android:name="android.intent.action.MAIN" />
	 <category android:name="android.intent.category.LAUNCHER" />
  </intent-filter>
 </activity>
```
上面用到的标签以及属性含义如下表
![activity](http://img.blog.csdn.net/20160911135452110)

![activity](http://img.blog.csdn.net/20160911135701312)

图1-2 Activity 的Label 属性

在【文件1-2】中，我将MainActivity 的所有东西都拷贝过来了，只不过是替换了类名。对于MainActivity来说正是配置了特定（action 为android:name="android.intent.action.MAIN"，category 为
android:name="android.intent.category.LAUNCHER"）的intent-filter，那么当应用安装好以后，才会在桌面创建一个图标，点击该图标打开MainActivity。现在我自定义的MyActivity 也配置了一模一样的intent-filter，那么系统也会为我的MyActivity 创建一个图标（如图1-3），如果用户点击了该图标，那么也可以直接打开我的MyActivity 界面。也就是说一个Android 工程可以创建多个图标，不过通常情况下一个Android 应用只要给一个入口的Activity 配置为入口Activity 即可。

![这里写图片描述](http://img.blog.csdn.net/20160911135839846)

图1-3 一个应用多个图标

# **2. Activity 的跳转**

一个Android 应用通常有多个界面，也就说有多个Activity，那么如何从一个Activity 跳转到另外一个Activity 呢？以及跳转的时候如何携带数据呢？

Activity 之间的跳转分为2 种：

**显式跳转**：在可以引用到另外一个Activity 的字节码，或者包名和类名的时候，通过字节码，或者包名+类名的方法实现的跳转叫做显示跳转。显示跳转多用于自己工程内部多个Activity 之间的跳转，因为在
自己工程内部可以很方便地获取到另外一个Activity 的字节码。

**隐式跳转**：隐式跳转不需要引用到另外一个Activity 的字节码，或者包名+类名，只需要知道另外一个Activity 在AndroidManifest.xml 中配置的intent-filter 中的action 和category 即可。言外之意，如果你想让
你的Activity 可以被隐式意图的形式启动起来，那么就必须为该Activity 配置intent-filter。

Activity 之间的跳转都是通过Intent 进行的。Intent 即意图，不仅用于描述一个的Activity 信息，同时也是一个数据的载体。

## **2.1 显式跳转**

为了方便演示，我们创建一个新的Android 工程《Activity 的跳转》。然后创建两个Activity 类，分别为FirstActivity，和SecondActivity，并在Android 工程的清单文件中声明这两个Activity 类。工程清单中添加Activity 配置如下所示。

```xml
<activity
	android:name="com.itheima.activitySkip.FirstActivity"
	android:label="@string/app_name" >
	<intent-filter>
		<action android:name="android.intent.action.MAIN" />
		<category android:name="android.intent.category.LAUNCHER" />
	</intent-filter>
</activity>

<activity android:name="com.itheima.activitySkip.SecondActivity"/>
```
在上面的清单文件中，我们将FirstActivity 配置成了主Activity，SecondActivity 则为普通Activity。这样FirstActivity 就是我们应用的入口Activity 了。FirstActivity 和SecondActivity 代码清单如下所示。

```java
package com.itheima.activitySkip;

import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.view.View;

public class FirstActivity extends Activity {

 @Override
 protected void onCreate(Bundle savedInstanceState) {
 	super.onCreate(savedInstanceState);
 	setContentView(R.layout.activity_first);
 	}
 	
 //绑定界面的Button 控件的点击事件，点击后实现跳转到SecondActivity
 public void skip2Second(View view){
 	//创建一个Intent 对象，并传递当前对象（Context 对象）和要跳转的Activity 类字节码
 	Intent intent = new Intent(this, SecondActivity.class);
 	//Intent 同时也是数据的载体，在跳转的时候可以携带数据
 	//通过intent 的putExtra(key,value)方法可以设置数据
 	//Intent 支持所有基本数据类型及其数组形式
 	intent.putExtra("name", "张三");
 	//启动第二个Activity
 	startActivity(intent);

 	}
}
```

```java
public class SecondActivity extends Activity {
    protected void onCreate(android.os.Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_second);
        /**
         * 获取Intent 对象，该对象是启动了当前Activity 的对象
         */
        Intent intent = getIntent();

        if (intent != null) {
            // 从Intent 中获取设置的数据，如果获取不到返回null
            //获取FirstActivity 中设置的数据
            String name = intent.getStringExtra("name");
            if (name!=null) {
                Toast.makeText(this, "name="+name, Toast.LENGTH_LONG).show();
            }
        }
    };
}
```
## **2.2 显式跳转用法总结**

**显示Intent 的写法**

 通过Intent 的带参构造函数
```java
Intent intent = new Intent(this, ThirdActivity.class);
```
通过Intent 的setClass 方法
```java
intent.setClass(this, SecondActivity.class);
```
**Intent 可以携带的数据类型**

- 八种基本数据类型boolean、byte、char、short、int、float、double、long 和String 以及这9 种数据类型的数组形式
- 实现了Serializable 接口的对象
- 实现了Android 的Parcelable 接口的对象以及其数组对象

## **2.3 隐式跳转**

在黑马Android 基础的第一天我们做了一个拨打电话的功能，其实那个实现的就是通过隐式意图跳转的功能。如果想让我们的SecondActivity 支持被隐式意图启动，那么就需要给SecondActivity 配置意图过滤器（intent-filter）。

1、配置intent-filter

SecondActivity 配置intent-filter

```xml
<activity android:name="com.itheima.activitySkip.SecondActivity">
    <!-- 配置意图过滤器-->
    <intent-filter >
        <!-- 在意图过滤器中设置action 和category，当有匹配的action 和category 的时候启动
        该Activity。这里使用Android 提供的默认category 即可-->
        <action android:name="com.itheima.activitySkip.SecondActivity"/>
        <category android:name="android.intent.category.DEFAULT"/>
    </intent-filter>
</activity>
```
我们给SecondActivity 配置了intent-filter，该标签下的action 子标签是我们自定义的，而category 则必须使用系统提供的。通过我们使用android:name="android.intent.category.DEFAULT"。配置成这样的category 的好处就是在启动SecondActivity 的时候category 可以设置也可以不设置。

2、通过隐式意图启动SecondActivity

使用隐式意图启动SecondActivity 核心代码

```java
//创建一个Intent 对象
Intent intent = new Intent();
//设置Action，次Action 必须和intent-filter 中的action 一模一样
intent.setAction("com.itheima.activitySkip.SecondActivity");
//对于android.intent.category.DEFAULT 类型的信息为Android 系统默认的信息，省略也可以
intent.addCategory("android.intent.category.DEFAULT");
//启动Activity
startActivity(intent);
```

## **2.4 案例-隐式意图打开浏览器**

自己将Android 应用的源码安装在： D:\AndroidSource_GB\AndroidSource_GB 中， 打开D:\AndroidSource_GB\AndroidSource_GB\packages\apps\Browser 目录，然后打开AndroidManifest.xml 清单文件，找到用于启动浏览器的intent-filter。大家可能发现有很多个intent-filter，这么多intent-filter 只要能匹配上一个则就可以将该Activity 启动起来。

Browser 浏览器的意图过滤器

```java
<intent-filter>
    <action android:name="android.intent.action.VIEW"/>

    <category android:name="android.intent.category.DEFAULT"/>
    <category android:name="android.intent.category.BROWSABLE"/>

    <data android:scheme="http"/>
    <data android:scheme="https"/>
    <data android:scheme="about"/>
    <data android:scheme="javascript"/>
</intent-filter>
```
观察上面的intent-flter，发现有一个data 标签，该标签中有scheme 属性。这个标签是用于传递数据的。如果某个intent-filter 有data 标签，那么在通过该intent-filter 启动Activity 的时候就必须设置data，data 标签的scheme 属性相当于数据的的前缀或约束，设置数据时必须在数据前加上该前缀。

下面就给出一个打来浏览器的核心代码

```java
//跳转到浏览器界面
    public void skip2Browser(View view){
        //创建一个Intent 对象
        Intent intent = new Intent();
        //设置Action
        intent.setAction("android.intent.action.VIEW");
        //设置category
        intent.addCategory("android.intent.category.BROWSABLE");
        //设置参数
        intent.setData(Uri.parse("http://www.itheima.com"));
        //启动Activity
        startActivity(intent);
    }
```

启动后的界面如图所示

![](http://img.blog.csdn.net/20170102150728679?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXhpMjk1MzA5MDY2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## **2.5 通过隐式意图给自己的Activity 传递数据**

首先Intent 本身就是数据的载体，因此不管是显示意图跳转还是隐式意图跳转都可以直接通过Intent的putXXX()方法设置数据。除此之外，隐式意图还可以根据intent-filter 中的data 标签然后通过Intent 的setData(Uri)方法设置数据。后者是本节要去演示的内容。

1、修改AndroidManifest.xml 文件中SecondActivity 的intent-filter 参数

给SecondActivity 配置intent-filter
```xml
<activity android:name="com.itheima.activitySkip.SecondActivity">
    <!-- 配置意图过滤器-->
    <intent-filter >
        <!-- 在意图过滤器中设置action 和category，当有匹配的action 和category
        的时候启动该Activity -->
        <action android:name="com.itheima.activitySkip.SecondActivity"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <!-- 设置数据协议-->
        <data android:scheme="money"/>
        <!-- 配置数据的mimeType:必须为xxx/xxx 的格式，否则会报异常-->
        <data android:mimeType="data/mymime"/>
    </intent-filter>
</activity>
```

多了两个data 标签，这两个标签分别配置了scheme 和mimeType属性。那么对于这样的intent-filter 我们该如何设置数据呢？

2、编写跳转的核心代码

跳转功能的核心代码

```java
// 发送数据给SecondActivity
public void sendData2Second(View view) {
    // 创建一个Intent 对象
    Intent intent = new Intent();
    // 设置Action
    intent.setAction("com.itheima.activitySkip.SecondActivity");
    // 对于android.intent.category.DEFAULT 类型的信息为Android 系统默认的信息，省略也可以
    intent.addCategory("android.intent.category.DEFAULT");
    //这里必须调用setDataAndType 方法同时将data 和type 设置出来
    intent.setDataAndType(Uri.parse("money:转账100 元。"), "data/mymime");
    // 启动Activity
    startActivity(intent);
}
```

注意：上面的代码是根据SecondActivity 的intent-fiter 的配置文件编写的，配置文件中的data 标签声明了scheme 和mimeType 属性，因此这里要想成功跳转就必须使用setDataAndType()这样的方法传递值。除此之外我们发现Intent 还有seteData()和setType()两个方法，这两个方法一看便知道是设置data 和设置mimeType 的，那么我们为何不使用这两个方法呢？

我们查看Intent 的setData 和setType 方法的源码以及setDataAndType，见setData 和setType 源码

```java
public Intent setData(Uri data) {
        mData = data;
        mType = null;
        return this;
    }

    public Intent setType(String type) {
        mData = null;
        mType = type;

        return this;
    }

    public Intent setDataAndType(Uri data, String type) {
        mData = data;
        mType = type;
        return this;
    }
```

从上面的源码我们发现setData()的时候给data 赋值了，但是把mimeType 的值给设置为null 了，setType的时候给mimeType 赋值了，但是把data 给设置为null 了，也就是说这两个方法是互斥的，而setDataAndType方法则同时设置了data 和mimeType，因此当我们需要同时设置data 和mimeType 的时候只能使用setDataAndType 方法。

3、在SecondActivity 中接收数据

在第二步中我们的sendData2Second 方法给Intent 设置了数据，那么这些数据如何在SecondActivity 中获取呢？

获取Intent 中的数据

```java
public class SecondActivity extends Activity {
    private static final String TAG = "SecondActivity";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_second);
        //获取Intent 对象
        Intent intent = getIntent();
        if (intent!=null) {
            //获取setData 方法设置的值
            Uri data = intent.getData();
            //获取data 中的scheme
            String scheme = data.getScheme();
            //获取mimeType
            String mimeType = intent.getType();
            Log.d(TAG, "data:"+data.toString());
            Log.d(TAG, "scheme:"+scheme);
            Log.d(TAG, "mimeType:"+mimeType);
        }
    }
}
```

执行完上面的代码后日志成功输入了我们的数据，如图所示

![](http://img.blog.csdn.net/20170102151626816?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXhpMjk1MzA5MDY2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

# **3. Activity 的生命周期**

学到这里我们有必要对Activity 有一个全新的认识。Activity 已经不能再简单的认为就是一个普通的类，其生命周期就是类的加载，对象的的创建和对象的卸载那么简单。Activity 从创建到销毁，整个生命周期是一个非常复杂的过程，该过程由Android 系统负责维护。

## **3.1 Activity 的3 种状态**

我们人类有婴幼儿时期、青少年时期、中老年时期，Activity 一样也有3 种状态：Resumed、Paused、Stopped，这三种状态是Android 官方给出的，我们翻译过来可以理解为：激活或运行状态、暂停状态、停止状态。这3 种状态的特点如下：

**Resumed 状态**：Activity 位于前端位置，并且获取到了用户的焦点。也就是当前Activity 完全可见也可用。
注意：在Android 中目前只允许同时只能有一个Activity 位于前端位置。

**Paused 状态：**如果另外一个Activity 位于前端位置并且获取了焦点，但是该Activity 还依然可见，那么该Activity 就处于了Paused 状态。比如如果另外一个Activity 虽然位于前端，但是是透明的或者没有
占满整个屏幕，那么就会出现上面的这种情况。位于Paused 状态的Activity 依然是“存活”着的，但是如果系统内存极端的不足，那么就有可能被系统“杀死”以便释放内存。

**Stopped 状态：**当另外一个Activity 完全将该Activity 遮盖住的情况下，那么该Activity 就处于停止状态了。位于停止状态的Activity 依然“活着”，但是它已经对用户完全不可见了，因此只要系统需要释放内存就会将该Activity“杀死”。我们编写的代码要根据Activity 的不同状态让其做不同的工作，比如如果我们的Activity 是用于播放视频的，那么当其位于Resumed 状态时我们可以让视频正常的播放，当Activity 位于Paused 状态的时候，我们也应该让我们的视频暂停播放，当Activity 位于Stopped 状态时我们应该停止播放视频并视频资源。那么如何让我们的程序员能够感知到Activity 的状态变化呢？Android 系统为了将Activity 状态的变化通知给我们在Activity 中提供了7 个回调方法。我们只需要在不同的回调方法中完成不同的工作即可。

## **3.2 Activity 生命周期的7 个回调方法**

![Activity 生命周期](http://img.blog.csdn.net/20160911143940786)

**Android 官方给出的Activity 生命周期图**
![Activity 生命周期](http://android.xsoftlab.net/images/activity_lifecycle.png)

## **3.3 观察Activity 生命周期方法的调用**

为了观察Activity 的生命周期，我们新创建一个Android 工程《Activity 的生命周期》。创建两个Activity，FirstActivity 和SecondActivity，在这两个Activity 中分别覆写Activity 生命周期的7 个回调方法。

```java
package com.itheima.activity.lifecycle;
public class FirstActivity extends Activity {
        private static final String TAG = "MyTag";

        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            Log.d(TAG, "FirstActivity：onCreate");
        }
        @Override
        protected void onStart() {
            super.onStart();
            Log.d(TAG, "FirstActivity：onStart");
        }

        @Override
        protected void onResume() {
            super.onResume();
            Log.d(TAG, "FirstActivity：onResume");
        }
        @Override
        protected void onPause() {
            super.onPause();
            Log.d(TAG, "FirstActivity：onPause");
        }
        @Override
        protected void onStop() {
            super.onStop();
            Log.d(TAG, "FirstActivity：onStop");
        }
        @Override
        protected void onRestart() {

            super.onRestart();
            Log.d(TAG, "FirstActivity：onRestart");
        }
        @Override
        protected void onDestroy() {
            super.onDestroy();
            Log.d(TAG, "FirstActivity：onDestroy");
        }
        /**
         * 跳转到SecondActivity
         * @param view
         */
        public void click1(View view){
            Intent intent = new Intent(this, SecondActivity.class);
            startActivity(intent);
        }
        /**
         * 调用本Activity 中的finish()方法
         * @param view
         */
        public void click2(View view){
            //调用该方法用于销毁当前Activity
            finish();
        }
        /**
         * 跳转到FirstActivity
         * @param view
         */
        public void click3(View view){
            startActivity(new Intent(this, FirstActivity.class));
        }

    }
```

```java
package com.itheima.activity.lifecycle;
    public class SecondActivity extends Activity {
        private static final String TAG = "MyTag";
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_second);
            Log.d(TAG, "SecondActivity：onCreate");
        }
        @Override
        protected void onStart() {
            super.onStart();
            Log.d(TAG, "SecondActivity：onStart");
        }

        @Override
        protected void onResume() {
            super.onResume();
            Log.d(TAG, "SecondActivity：onResume");
        }
        @Override
        protected void onPause() {
            super.onPause();
            Log.d(TAG, "SecondActivity：onPause");
        }
        @Override
        protected void onStop() {
            super.onStop();
            Log.d(TAG, "SecondActivity：onStop");
        }
        @Override
        protected void onRestart() {
            super.onRestart();
            Log.d(TAG, "SecondActivity：onRestart");
        }
        @Override
        protected void onDestroy() {
            super.onDestroy();
            Log.d(TAG, "SecondActivity：onDestroy");
        }
        /**
         * 跳转到FirstActivity
         * @param view
         */
        public void click1(View view){
            startActivity(new Intent(this, FirstActivity.class));
        }
        /**
         * 跳转到当前Activity 也就是SecondActivity
         * @param view
         */
        public void click2(View view){
            startActivity(new Intent(this, SecondActivity.class));
        }
    }
```
FirstActivity 的效果图下图所示
![activity](http://img.blog.csdn.net/20160914104740803)
FirstActivity 和SecondActivity 都必须在AndroidManifest.xml 中进行注册，这里就不给出清单文件代码了，比较简单。

**1、启动一个新的Activity 时的调用情况**
当新启动FirstActivity 时生命周期如下图所示，调用顺序为：onCreate->onStart->onResume，下面我分别以多种情况来分析Activity 的生命周期调用情况。

![这里写图片描述](http://img.blog.csdn.net/20160914104902159)

**2、点击back 键，关闭Activity 时的调用情况**
在上面的基础上点击back 键，生命周期新增了onPause->onStop->onDestory
![这里写图片描述](http://img.blog.csdn.net/20160914105038005)

## **3.4 横竖屏切换时Activity 的生命周期**

Android 手机在横竖屏切换时，默认情况下会把Activity 先销毁再创建，其生命周期如图1-14。模拟器横竖屏切换的快捷键是Ctrl+F11。

在类似手机游戏、手机影音这一类的应用中，这个体验是非常差的。不让Activity 在横竖屏切换时销毁，只需要在清单文件声明Activity 时配置&lt;activity>节点的几个属性即可，其方式如下：
![activity](http://img.blog.csdn.net/20160914105229836)

**4.0 以下版本：**

```xml
android:configChanges="orientation|keyboardHidden"
```
4.0 以上版本：

```xml
android:configChanges="orientation|screenSize"
```
4.0 以上版本：

```xml
android:configChanges="orientation|keyboardHidden|screenSize"
```
将该参数配置到FirstActivity 中后在切换屏幕方向，那么Activity 就不会再销毁和重新创建了。但是配置了该参数仅仅是不让Activity 销毁和重建，Activity 界面依然会跟着屏幕方向重新调整，那么如何固定Activity 的方向呢？

## **3.5 固定Activity 的方向**

想固定Activity 的方向其实比较简单，有两种方法：

### **3.5.1 通过配置文件**

在AndroidManifest.xml 中的activity 节点中添加如下属性。

```xml
android:screenOrientation="portrait"
```
该属性通常有两个常量值，portrait：垂直方向，landscape：水平方向。

### **3.5.2  通过代码**

```
在Activity 的onCreate 方法中执行如下方法。
//垂直方向
setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_PORTRAIT);
//水平方向
setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_LANDSCAPE);
```

# **6. Activity 的启动模式**

## **6.1 Activity 的任务栈**

Task Stack（任务栈）是一个具有栈结构的容器，可以放置多个Activity 实例。启动一个应用，系统就会为之创建一个task，来放置根Activity；默认情况下，一个Activity 启动另一个Activity 时，两个Activity是放置在同一个task 中的，后者被压入前者所在的task 栈，当用户按下back 键，后者从task 中被弹出，前者又显示在屏幕前，特别是启动其他应用中的Activity 时，两个Activity 对用户来说就好像是属于同一个应用；系统task 和应用task 之间是互相独立的，当我们运行一个应用时，按下Home 键回到主屏，启动另一个应用，这个过程中，之前的task 被转移到后台，新的task 被转移到前台，其根Activity 也会显示到幕前，过了一会之后，再次按下Home 键回到主屏，再选择之前的应用，之前的task 会被转移到前台，系统仍然保留着task 内的所有Activity 实例，而那个新的task 会被转移到后台，如果这时用户再做后退等动作，就是针对该task 内部进行操作了。任务栈的设计是为了提高用户体验，但是也有其不足的地方。任务栈的缺点如下：

- 每开启一次页面都会在任务栈中添加一个Activity，而只有任务栈中的Activity 全部清除出栈时，任务栈被销毁，程序才会退出，这样的设计在某种程度上可能造成了用户体验差，需要点击多次返回才可以把程序退出了。
- 每开启一次页面都会在任务栈中添加一个Activity 还会造成数据冗余, 重复数据太多, 会导致内存溢出的问题(OOM)。

为了解决任务栈产生的问题，Android 为Activity 设计了启动模式，那么下面的内容将介绍Android 中Activity 的启动模式，这也是最重要的内容之一。
## **6.2 Activity 的启动模式**

启动模式（launchMode）在多个Activity 跳转的过程中扮演着重要的角色，它可以决定是否生成新的Activity 实例，是否重用已存在的Activity 实例，是否和其他Activity 实例共用一个task。

Activity 一共有以下四种launchMode：standard、singleTop、singleTask、singleInstance。

我们可以在AndroidManifest.xml 配置<activity>的android:launchMode 属性为以上四种之一即可。

下面我们结合实例一一介绍这四种lanchMode。

当activity设置为singleTask /SingleTop的时候 如果调用startActivity没有创建新的实例 还需要传递数据并且更新目标activity的界面，可以通过重写onNewIntent来实现这个需求