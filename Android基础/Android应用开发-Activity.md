Android有四大组件，分别是：Activity，BroadCastReceiver，Service，ContentProvider，本篇博客介绍的是其中的Activity。先来看下内容摘要

- 什么是Activity
- Activity的跳转
- Activity的生命周期
- Activity的启动模式
- 获取其他Activity 销毁时设置的数据

Activity 是Android 四大组件之一，用于展示界面。Activity 中所有操作都与用户密切相关，是一个负责与用户交互的组件，它上面可以显示一些控件也可以监听并处理用户的事件。一个Activity 通常就是一个单独的屏幕，Activity 之间通过Intent 进行通信。

# Activity的跳转

首先，创建一个Activity类。

![img](http://bbs.itheima.com/data/attachment/forum/201506/29/111654u4q191xxp36b4kzh.png.thumb.jpg) 

src/cn.itcast.createactivity/SecondActivity.java
```java
package cn.itcast.createactivity;
import android.app.Activity;
import android.os.Bundle;
public class SecondActivity extends Activity {
        //Activity显示在页面时，也就是创建时，onCreate方法就会被调用
        @Override
        protected void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                //多个Activity可以显示同一个布局文件，不过开发中很少有这样的需求
                setContentView(R.layout.activity_second);
        }
}
```
为secondactivity创建布局文件。

![img](http://bbs.itheima.com/data/attachment/forum/201506/29/111655vbcrrzi5n9bpinx5.png.thumb.jpg) 

res/layout/activity_second.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical" >

        <TextView 
             android:layout_width="wrap_content"
             android:layout_height="wrap_content"
             android:text="这是第二个Activity"
             android:textSize="20sp"/>
 
</LinearLayout>
```

在清单文件中配置Activity。AndroidManifest.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="cn.itcast.createactivity"
    android:versionCode="1"
    android:versionName="1.0" >

    <uses-sdk
        android:minSdkVersion="8"
        android:targetSdkVersion="17" />
 
     <application
         android:allowBackup="true"
         android:icon="@drawable/ic_launcher"
         android:label="@string/app_name"
         android:theme="@style/AppTheme" >
         <activity
             android:name="cn.itcast.createactivity.MainActivity"
             android:icon="@drawable/photo1"
             android:label="主啊" >
             <intent-filter>
                 <action android:name="android.intent.action.MAIN" />
                 <category android:name="android.intent.category.LAUNCHER" />
             </intent-filter>
         </activity>
         <activity
             android:name="cn.itcast.createactivity.SecondActivity"
                         android:icon="@drawable/photo2"
             android:label="第二个啊" >
             <intent-filter>
                 <action android:name="android.intent.action.MAIN" />
                 <category android:name="android.intent.category.LAUNCHER" />
             </intent-filter>
         </activity>
     </application>
</manifest>
```
说明：上面的MainActivity及SecondActivity的icon及label属性用于区分开两个Activity。intent-filter（意图过滤器）标签配置在应用程序入口Activity中，如上，在两个Activity中都配置intent-filter的情况，在实际开发中很少存在。

运行结果：

![img](http://bbs.itheima.com/data/attachment/forum/201506/29/112856dv38vr68lohee6oe.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201506/29/112857t02qjbsqxzq2xqqh.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201506/29/112859wbs3obnwv31rabt3.png.thumb.jpg) 

显示启动Activity
大多数的应用程序都有多个Activity，但是只有一个入口Activity，其他Activity的显示都是通过跳转实现的。

示例1：隐式启动拨号器

res/layout/activity_main.xml

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity" 
    android:orientation="vertical">
    <TextView 
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="这是主Activity"
        android:textSize="15sp"
        />
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="启动打电话Activity"
        android:onClick="click1" />
</LinearLayout>
```
src/cn.itcast.jump/MainActivity.java

```java
package cn.itcast.jump;

import android.app.Activity;
import android.content.Intent;
import android.net.Uri;
import android.os.Bundle;
import android.view.View;

public class MainActivity extends Activity {
 
         @Override
         protected void onCreate(Bundle savedInstanceState) {
                 super.onCreate(savedInstanceState);
                 setContentView(R.layout.activity_main);
         }
 
         //隐式跳转
         public void click1(View v){
                 //创建意图对象，隐式意图
                 Intent intent = new Intent();
                 intent.setAction(Intent.ACTION_CALL);
                 intent.setData(Uri.parse("tel:110"));
                 //启动新的Activity，也就是Activity的跳转
                 startActivity(intent);
         }
}
```
添加权限：

![img](http://bbs.itheima.com/data/attachment/forum/201506/29/120704z7mfnf10mnmh7n71.png.thumb.jpg) 

运行结果：

![img](http://bbs.itheima.com/data/attachment/forum/201506/29/120702rrt4gg6g7ge27rge.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201506/29/120703tr1g1d5ose6ms5dn.png.thumb.jpg) 

示例2：显示启动第二个Activity

res/layout/activity_second.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical" >

    <TextView 
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
         android:text="这是第二个Activity"
         android:textSize="18sp"
         />
 
</LinearLayout>
```

src/cn.itcast.jump/SecondActivity.java

```java
package cn.itcast.jump;

import android.app.Activity;
import android.os.Bundle;

public class SecondActivity extends Activity {

        @Override
        protected void onCreate(Bundle savedInstanceState) {
                 super.onCreate(savedInstanceState);
                 setContentView(R.layout.activity_second);
         }
}
```
res/layout/activity_main.xml
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
   xmlns:tools="http://schemas.android.com/tools"
   android:layout_width="match_parent"
   android:layout_height="match_parent"
   tools:context=".MainActivity" 
   android:orientation="vertical">
   
   <TextView 
       android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="这是主Activity"
        android:textSize="15sp"
        />

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="启动SecondActivity"
        android:onClick="click2" />

</LinearLayout>
```
src/cn.itcast.jump/MainActivity.java
```java
package cn.itcast.jump;

import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.view.View;

public class MainActivity extends Activity {

        @Override
        protected void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                setContentView(R.layout.activity_main);
        }

        //显式跳转
        public void click2(View v){
                //显式意图
                Intent intent = new Intent();
                //通过指定Activity类的字节码，指定要跳转到哪一个Activity
                intent.setClass(this, SecondActivity.class);
                startActivity(intent);
        }

}
```
在清单文件中配置第二个Activity：
![img](http://bbs.itheima.com/data/attachment/forum/201506/29/121509pw2338kfgogf5fur.png.thumb.jpg) 
​    
运行结果：

![img](http://bbs.itheima.com/data/attachment/forum/201506/29/121640slugzicqqlzuxxcq.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201506/29/121641ouls01440ssu1b88.png.thumb.jpg) 

示例3：显示启动拨号器

显式启动是通过指定目标Activity的包名和类名启动的。

res/layout/activity_main.xml
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity" 
    android:orientation="vertical">
    <TextView 
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="这是主Activity"
        android:textSize="15sp"
        />
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="显式启动拨号器"
        android:onClick="click3" />
</LinearLayout>
```
src/cn.itcast.jump/MainActivity.java
```java
package cn.itcast.jump;
import android.app.Activity;
import android.content.Intent;
import android.net.Uri;
import android.os.Bundle;
import android.view.View;
public class MainActivity extends Activity {
        @Override
        protected void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                setContentView(R.layout.activity_main);
        }
        //显式启动拨号器
        public void click3(View v){
                Intent intent = new Intent();
                //指定要启动哪个应用中的哪个Activity
                //第一个参数为应用包名，第二个参数为Activity全名
                intent.setClassName("com.android.dialer", "com.android.dialer.DialtactsActivity");
                startActivity(intent);
        }
}
```
运行结果：

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/133757kmeukgivyy9749gk.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/141628c0hc2bog24o34mcc.png.thumb.jpg) 

Android4.2版本模拟器中拨号器Activity为com.android.contacts.DialtactsActivity，Android4.2版本模拟器中拨号器Activity为com.android.dialer.DialtactsActivity。

在清单文件中，配置Activity标签的name属性时，如果Activity所在的包和应用包名一致，可以用点（.）代替包名，甚至可以连点都不写。

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/133807at3t2kic4n8i4rki.png.thumb.jpg) 

## 隐式启动Activity

一个Activity既可以被显示启动，也可以被隐式启动。

示例1：隐式启动拨号器

res/layout/activity_main.xml
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
   xmlns:tools="http://schemas.android.com/tools"
   android:layout_width="match_parent"
   android:layout_height="match_parent"
   tools:context=".MainActivity" 
   android:orientation="vertical">

   <TextView 
       android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="这是主Activity"
        android:textSize="15sp"
        />

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="隐式启动拨号器"
        android:onClick="click4" />
</LinearLayout>
```

src/cn.itcast.jump/MainActivity.java

```java
package cn.itcast.jump;

import android.app.Activity;
import android.content.Intent;
import android.net.Uri;
import android.os.Bundle;
import android.view.View;

public class MainActivity extends Activity {

        @Override
        protected void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                setContentView(R.layout.activity_main);
        }

        //隐式启动拨号器
        public void click4(View v){
                Intent intent = new Intent();
                intent.setAction(Intent.ACTION_DIAL);
                startActivity(intent);
        }
}
```


![img](http://bbs.itheima.com/data/attachment/forum/201507/02/142059v9698j6d696lvzg6.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/142100t8t2tespjoeje9z3.png.thumb.jpg) 

示例2：隐式启动SecondActivity

res/layout/activity_main.xml
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
   xmlns:tools="http://schemas.android.com/tools"
   android:layout_width="match_parent"
   android:layout_height="match_parent"
   tools:context=".MainActivity" 
   android:orientation="vertical">

   <TextView 
       android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="这是主Activity"
        android:textSize="15sp"
        />

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="隐式启动SecondActivity"
        android:onClick="click5" />

</LinearLayout>
```

src/cn.itcast.jump/MainActivity.java

```java
package cn.itcast.jump;

import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.view.View;

public class MainActivity extends Activity {

         @Override
         protected void onCreate(Bundle savedInstanceState) {
                 super.onCreate(savedInstanceState);
                 setContentView(R.layout.activity_main);
         }
 
         //隐式启动SecondActivity
         public void click5(View v){
                 Intent intent = new Intent();
                 intent.setAction("a.b.c");
                 startActivity(intent);
         }
 
}
```
AndroidManifest.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="cn.itcast.jump"
    android:versionCode="1"
    android:versionName="1.0" >

    <uses-sdk
        android:minSdkVersion="8"
        android:targetSdkVersion="17" />
     <uses-permission android:name="android.permission.CALL_PHONE"/>
 
     <application
         android:allowBackup="true"
         android:icon="@drawable/ic_launcher"
         android:label="@string/app_name"
         android:theme="@style/AppTheme" >
         <activity
             android:name=".MainActivity"
             android:label="@string/app_name" >
             <intent-filter>
                 <action android:name="android.intent.action.MAIN" />
                 <category android:name="android.intent.category.LAUNCHER" />
             </intent-filter>
         </activity>
         <activity
             android:name=".SecondActivity"
             android:label="@string/app_name" >
             <intent-filter>
                 <action android:name="a.b.c" />
                 <category android:name="android.intent.category.DEFAULT"/>
             </intent-filter>
         </activity>
     </application>
 
</manifest>
```


运行结果：

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/164922dx8wwx0edzmxe0ec.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/164924ym20fbboqzt7qpf7.png.thumb.jpg) 

1、创建一个Activity，并且准备隐式启动它时，一定要在清单文件中为这个Activity加上意图过滤器标签（intent-fiter）。这是因为，当设置一个Action，把一个Action封装到意图中的时候（intent.setAction("a.b.c");），并且准备启动Activity的时候（startActivity(intent);），系统会在所有清单文件中去寻找是否有一个Activity标签的意图过滤器标签中action标签的android:name属性值与MainActivity.java中的intent中的setAction参数值相匹配。如果找到匹配的Activity，就启动显示。如果找不到，就报错。因此，清单文件中，Activity标签的意图过滤器标签中action标签的android:name属性值由我们自己指定，作用就是用于标示一个Activity并用于匹配。

2、intent匹配，这个匹配，指的是intent-filter中的所有子节点都需要匹配（action、data、category）。例如，在清单文件中，为Activity的intent-filter加上data标签，那么MainActivity中的intent也必须调用相应的方与之匹配。否则，报错。

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/171535irenza2raennkofr.png.thumb.jpg) 

src/cn.itcast.jump/MainActivity.java
```java
package cn.itcast.jump;
import android.app.Activity;
import android.content.Intent;
import android.net.Uri;
import android.os.Bundle;
import android.view.View;
public class MainActivity extends Activity {
        @Override
        protected void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                setContentView(R.layout.activity_main);
        }
        //隐式启动SecondActivity
        public void click5(View v){
                Intent intent = new Intent();
                intent.setAction("a.b.c");
                intent.setData(Uri.parse("itcast:hahahaha"));
                startActivity(intent);
        }
}
```
3、也可以为Activity定义多个意图过滤器，每个意图过滤器中也可以定义多个action和多个data，Intent的Action只要能够匹配任何一个就可以。

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/172228cf76x4p5k946hi55.png.thumb.jpg) 

4、category如果是DEFAULT，可以不用写匹配语句。但是，清单文件中的意图过滤器中必须要有category。否则，MainActivity.java中的intent调用addCategory方法默认匹配CATEGORY_DEFAULT，但清单文件中没有设置，就会报错。

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/172707jq3fuxgme3am1h1h.png.thumb.jpg) 

5、意图过滤器标签中的data属性表示Activity需要接收一个提交的数据，可以通过意图过滤器的data标签的android:mimetype定义提交的数据类型。此时，MainActivity中除了要通过intent.setData方法与data标签匹配，同时还需要intent.setType()语句与data标签的android:mimeType属性匹配。不过，setType方法与setData方法不能共存，否则，先执行的那个方法就会失效。不过，通过intent.setDataAndType()方法做到同时匹配。

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/174329lhhrxzbe9llurkxk.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/174330yj11o1por7vv7mzp.png.thumb.jpg) 

获取隐式意图传递的数据

src/cn.itcast.jump/SecondActivity.java
```java
package cn.itcast.jump;
import android.app.Activity;
import android.content.Intent;
import android.net.Uri;
import android.os.Bundle;
public class SecondActivity extends Activity {
        @Override
        protected void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                setContentView(R.layout.activity_second);
                //获取启动这个Activity的意图
                Intent intent = getIntent();
                Uri uri = intent.getData();
                System.out.println(uri.toString());
        }
}
```
运行结果：

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/180229rilu0uljultbfcfw.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/180230hddhobo8y1d2mcmd.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/180214jl2mmo9lmv96mb4c.png.thumb.jpg) 

## 显式启动和隐式启动的应用场景

启动同一个项目的Activity，使用显式启动。因为，显式启动的效率要比隐式启动高很多。隐式启动会去遍历所有应用程序的清单文件对Activity做匹配。而显式启动，则会根据指定的包名及类名去查找对应的应用程序的清单文件中是否存在Activity，而不用遍历所有应用程序的清单文件。如果通过显式启动去查找自己应用程序中的Activity，直接通过intent.setClass(上下文this，Activity类名)，遍历自己的清单文件查找即可。

启动不同项目的Activity，才会使用到隐式启动。原因是因为，第一，使用显式方式启动其他应用程序的Activity，需要使用包名和类名，而每个版本的应用程序，某个Activity的包名和类名不一定总是固定相同的。第二个原因：隐式启动可以为用户提供选择。

示例1：显式启动浏览器，不会给用户提供选择。

首先，为模拟器安装360浏览器。

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/183651zvgqx72udwdx2s2r.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/183652mjivt9ijvsz3rgij.png.thumb.jpg) 

 res/layout/activity_main.xml

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity" 
    android:orientation="vertical">

    <TextView 
        android:layout_width="wrap_content"
         android:layout_height="wrap_content"
         android:text="这是主Activity"
         android:textSize="15sp"
         />
 
     <Button
         android:layout_width="wrap_content"
         android:layout_height="wrap_content"
         android:text="显式启动浏览器"
         android:onClick="click5" />
 
</LinearLayout>
```
src/cn.itcast.jump/MainActivity.java
```java
package cn.itcast.jump;

import android.app.Activity;
import android.content.Intent;
import android.net.Uri;
import android.os.Bundle;
import android.view.View;

public class MainActivity extends Activity {
 
         @Override
         protected void onCreate(Bundle savedInstanceState) {
                 super.onCreate(savedInstanceState);
                 setContentView(R.layout.activity_main);
         }
 
         //显式启动浏览器
         public void click5(View v){
                 Intent intent = new Intent();
                 intent.setClassName("com.android.browser", "com.android.browser.BrowserActivity");
                 startActivity(intent);
         }
 
}
```

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/183944hcbumsmzpu2z26az.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/183945uiqxxu5duiv7ici1.png.thumb.jpg) 

示例2：src/cn.itcast.jump/MainActivity.java
```java
package cn.itcast.jump;

import android.app.Activity;
import android.content.Intent;
import android.net.Uri;
import android.os.Bundle;
import android.view.View;

public class MainActivity extends Activity {
 
         @Override
         protected void onCreate(Bundle savedInstanceState) {
                 super.onCreate(savedInstanceState);
                 setContentView(R.layout.activity_main);
         }
 
         //隐式启动浏览器
         public void click5(View v){
                 Intent intent = new Intent();
                 //有多个Activity与这个intent匹配，那么就会弹出选择对话框
                 intent.setAction(Intent.ACTION_VIEW);
                 intent.setData(Uri.parse("http:www.baidu.com"));
                 startActivity(intent);
         }
 
}
```
运行结果：

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/184108kjllwjolakjry1yc.png.thumb.jpg) 

# Activity显式跳转时传递数据

1、intent.setData()语句只有在隐式跳转时使用。

2、显式跳转，没有必要在清单文件中配置intent-filter，配了也没用。

示例：res/layout/activity_main.xml

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity"
    android:orientation="vertical" >

    <TextView
        android:layout_width="wrap_content"
         android:layout_height="wrap_content"
         android:text="这是主Activity" />
 
     <EditText 
         android:id="@+id/et_malename"
         android:layout_width="wrap_content"
         android:layout_height="wrap_content"
         android:text="云鹤" 
         />
 
     <EditText 
         android:id="@+id/et_femalename"
         android:layout_width="wrap_content"
         android:layout_height="wrap_content"
         android:text="老兵" 
         />
 
     <Button
         android:layout_width="wrap_content"
         android:layout_height="wrap_content"
         android:text="计算姻缘"
         android:onClick="click" />
 
</LinearLayout>
```
src/cn.itcast.transmitdata/MainActivity.java
```java
package cn.itcast.transmitdata;

import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.EditText;

public class MainActivity extends Activity {
 
         @Override
         protected void onCreate(Bundle savedInstanceState) {
                 super.onCreate(savedInstanceState);
                 setContentView(R.layout.activity_main);
         }
 
         public void click(View v){
 
                 EditText et_maleName = (EditText) findViewById(R.id.et_malename);
                 EditText et_femaleName = (EditText) findViewById(R.id.et_femalename);
 
                 String maleName = et_maleName.getText().toString();
                 String femaleName = et_femaleName.getText().toString();
 
                 Intent intent = new Intent(this, SecondActivity.class);
 
                 //显式跳转，发送数据方法一
                 //把数据封装至意图对象中
                 //可以封装数据类型：八大基本数据类型、bundle对象、String、实现了序列化接口的对象
                 //intent.putExtra("malename", maleName);
                 //intent.putExtra("femalename", femaleName);
 
                 //显式跳转，发送数据方法二
                 //Bundle是Android中用来封装数据的一个API，类似于Map
                 //Bundle能封装的数据类型与intent一样
                 Bundle bundle = new Bundle();
                 bundle.putString("maleName",maleName);
                 bundle.putString("femaleName", femaleName);
 
                 //把bundle放入意图中
                 intent.putExtras(bundle);
                 startActivity(intent);
         }
}
```
res/layout/activity_second.xml
```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity" >

    <TextView
        android:id="@+id/tv"
        android:layout_width="wrap_content"
         android:layout_height="wrap_content"
         android:text="这是第二个Activity"/>
 
</RelativeLayout>
```
src/cn.itcast.transmitdata/SecondActivity.java
```java
package cn.itcast.transmitdata;

import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.widget.TextView;

public class SecondActivity extends Activity {

         @Override
         protected void onCreate(Bundle savedInstanceState) {
 
                 super.onCreate(savedInstanceState);
                 setContentView(R.layout.activity_second);
 
                 //获取到启动该Activity的意图对象
                 Intent intent = getIntent();
 
                 //显式跳转，接收数据方法一
                 //String maleName = intent.getStringExtra("maleName");
                 //String femaleName = intent.getStringExtra("femaleName");
 
                 //显式跳转，接收数据方法二
                 Bundle bundle = intent.getExtras();
                 String maleName = bundle.getString("maleName");
                 String femaleName = bundle.getString("femaleName");
 
                 String text = maleName + femaleName;
                 byte[] b = text.getBytes();
                 int total = 0;
 
                 for(byte c : b){
                         total += c;
                 }
 
                 //取绝对值，避免负数，模101，最大值100
                 total = Math.abs(total) % 101;
 
                 TextView tv = (TextView)findViewById(R.id.tv);
                 tv.setText(maleName + "和" + femaleName + "的姻缘值为：" + total + ",实乃天作之合");
         }
}
```
AndroidManifest.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="cn.itcast.transmitdata"
    android:versionCode="1"
    android:versionName="1.0" >

    <uses-sdk
        android:minSdkVersion="8"
        android:targetSdkVersion="17" />
 
     <application
         android:allowBackup="true"
         android:icon="@drawable/ic_launcher"
         android:label="@string/app_name"
         android:theme="@style/AppTheme" >
         <activity
             android:name="cn.itcast.transmitdata.MainActivity"
             android:label="@string/app_name" >
             <intent-filter>
                 <action android:name="android.intent.action.MAIN" />
 
                 <category android:name="android.intent.category.LAUNCHER" />
             </intent-filter>
         </activity>
         <activity android:name="cn.itcast.transmitdata.SecondActivity" />
     </application>
 
</manifest>
```
运行结果：

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/200027adz6vndo41nmxagk.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/200029mxw7fwi7k0iiq0w1.png.thumb.jpg) 

# Activity的生命周期

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/202656prr6tqfnz68t6e6m.png.thumb.jpg) 

- onCreate：Activity创建了，但是还没有显示到界面上
- onStart：Activity显示在屏幕上，但是还没有获得焦点
- onResume：Activity获得焦点
- onPause：失去焦点，但是还显示在屏幕上
- onStop：失去焦点，并且不可见了
- onDestroy：摧毁

示例：res/layout/activity_main.xml
```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity" >

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
         android:text="这是主Activity" />
 
</RelativeLayout>
```
src/cn.itcast.ifecycle/MainActivity.java
```java
package cn.itcast.lifecycle;

import android.os.Bundle;
import android.app.Activity;
import android.view.Menu;

public class MainActivity extends Activity {

        @Override
         protected void onCreate(Bundle savedInstanceState) {
                 super.onCreate(savedInstanceState);
                 setContentView(R.layout.activity_main);
                 System.out.println("主create");
         }
 
         @Override
         protected void onStart() {
                 super.onStart();
                 System.out.println("主start");
         }
 
         @Override
         protected void onResume() {
                 super.onResume();
                 System.out.println("主resume");
         }
 
         @Override
         protected void onPause() {
                 super.onPause();
                 System.out.println("主pause");
         }
 
         @Override
         protected void onStop() {
                 super.onStop();
                 System.out.println("主stop");
         }
 
         @Override
         protected void onDestroy() {
                 super.onDestroy();
                 System.out.println("主destroy");
         }
 
         @Override
         protected void onRestart() {
                 super.onRestart();
                 System.out.println("主restart");
         }
}
```

运行结果：

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/204100kqvvv2k2vrrvkv7w.png.thumb.jpg)

点击返回键，摧毁Activity。

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/204101wmh0d50bppkdpcw3.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/204102ztwemz7dzfaxxdwf.png.thumb.jpg) 

重新在模拟器上运行应用程序，点击HOME键，不摧毁Activity，再点击应用图标，恢复Activity的显式状态，结果如下。

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/204700l51f2bbmez52tnb5.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/204928kkuunljdndpsc0kb.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/204930p8i6kkyvahcizfim.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/204701f01uwr1hq1tku0nz.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/204702mf6zvyvlv8vhk82y.png.thumb.jpg) 

跳转时的生命周期 示例1：res/layout/activity_main.xml
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity" 
    android:orientation="vertical">

    <TextView
        android:layout_width="wrap_content"
         android:layout_height="wrap_content"
         android:text="这是主Activity" />
 
     <Button
         android:layout_width="wrap_content"
         android:layout_height="wrap_content"
         android:text="跳转到第二个" 
         android:onClick="click"/>
 
</LinearLayout>
```
src/cn.itcast.lifecycle/MainActivity.java
```java
package cn.itcast.lifecycle;

import android.os.Bundle;
import android.app.Activity;
import android.content.Intent;
import android.view.Menu;
import android.view.View;

public class MainActivity extends Activity {
 
         @Override
         protected void onCreate(Bundle savedInstanceState) {
                 super.onCreate(savedInstanceState);
                 setContentView(R.layout.activity_main);
                 System.out.println("主create");
         }
 
         @Override
         protected void onStart() {
                 super.onStart();
                 System.out.println("主start");
         }
 
         @Override
         protected void onResume() {
                 super.onResume();
                 System.out.println("主resume");
         }
 
         @Override
         protected void onPause() {
                 super.onPause();
                 System.out.println("主pause");
         }
 
         @Override
         protected void onStop() {
                 super.onStop();
                 System.out.println("主stop");
         }
 
         @Override
         protected void onDestroy() {
                 super.onDestroy();
                 System.out.println("主destroy");
         }
 
         @Override
         protected void onRestart() {
                 super.onRestart();
                 System.out.println("主restart");
         }
 
         public void click(View v) {
                 Intent intent = new Intent(this, SecondActivity.class);
                 startActivity(intent);
         }
}
```
res/layout/activity_second.xml
```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity" >

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
         android:text="这是第二个Activity" />
 
</RelativeLayout>
```
src/cn.itcast.lifecycle/SecondActivity.java
```java
package cn.itcast.lifecycle;

import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.view.View;

public class SecondActivity extends Activity {

         @Override
         protected void onCreate(Bundle savedInstanceState) {
                 super.onCreate(savedInstanceState);
                 setContentView(R.layout.activity_second);
                 System.out.println("二create");
         }
 
         @Override
         protected void onStart() {
                 super.onStart();
                 System.out.println("二start");
         }
 
         @Override
         protected void onResume() {
                 super.onResume();
                 System.out.println("二resume");
         }
 
         @Override
         protected void onPause() {
                 super.onPause();
                 System.out.println("二pause");
         }
 
         @Override
         protected void onStop() {
                 super.onStop();
                 System.out.println("二stop");
         }
 
         @Override
         protected void onDestroy() {
                 super.onDestroy();
                 System.out.println("二destroy");
         }
 
         @Override
         protected void onRestart() {
                 super.onRestart();
                 System.out.println("二restart");
         }
 
}
```
AndroidManifest.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="cn.itcast.lifecycle"
    android:versionCode="1"
    android:versionName="1.0" >

    <uses-sdk
        android:minSdkVersion="8"
        android:targetSdkVersion="17" />
 
     <application
         android:allowBackup="true"
         android:icon="@drawable/ic_launcher"
         android:label="@string/app_name"
         android:theme="@style/AppTheme" >
         <activity
             android:name="cn.itcast.lifecycle.MainActivity"
             android:label="@string/app_name" >
             <intent-filter>
                 <action android:name="android.intent.action.MAIN" />
 
                 <category android:name="android.intent.category.LAUNCHER" />
             </intent-filter>
         </activity>
         <activity
             android:name="cn.itcast.lifecycle.SecondActivity">
         </activity>
     </application>
 
</manifest>
```
运行结果：点击“跳转按钮”，观察两个Activity的生命周期。

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/214722ku5dzwpwwacnmw9m.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/214715ftswitqgvz4sc61g.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/214717wu7zizjtk0z44njk.png.thumb.jpg) 

点击“返回按钮”，观察两个Activity的生命周期。

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/214719o0n7ne7ekekdmmg3.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/214720xfppgkgdgdggdccw.png.thumb.jpg) 

 如果刚才不点击“返回按钮”，而是点击“HOME按钮”，再观察两个Activity的生命周期。

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/215443ipize67iosie2ipe.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/215445owoei3oesklwke1j.png.thumb.jpg) 

然后再点击“应用程序图标”，再观察两个Activity的生命周期。

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/215654cajmmmomnntvmelq.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/215658su44bsjswwbfhdaf.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/215702j0l0yvzmm2qyy204.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/215706padu8jb0waodd3z0.png.thumb.jpg) 

示例2：基于上面示例1的基础上，修改SecondActivity的样式为透明，也就是当SecondActivity显示出来时，同时MainActivity也可以显示出来，只是MainActivity的焦点丢失了，SecondActivity获取了焦点。此时，MainActivity的状态是pause状态，而不会是stop状态。

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/220507iwk0ekzsexe5m60k.png.thumb.jpg) 

重新执行应用程序，点击跳转按钮，可以看到MainActivity并没有处于Stop状态，而是处于Pause状态。

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/220624ya61sel1zjlazjxk.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/220625r152e0p8gv51e8c9.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/220626ztwniwtwosgmsswx.png.thumb.jpg) 

然后点击“返回按钮”，再观察两个Activity的生命周期。

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/220912o50iimbemi5beieb.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/220910izrvphvr5vdrpzxz.png.thumb.jpg) 

PS：当Activity销毁或者进入后台，或者失去焦点，进程都不会被杀死，直到系统内存不足时，系统才会去杀死这些进程。选择被杀死的目标时，会使用LRU（Least Recently Used，最近最少使用）算法。


# Activity的启动模式

一个应用程序可以启动多个Activity，启动多个Activity后，所有的Activity对象都会保存在内存的任务栈（Activity task stack）中，也就保存了Activity的启动顺序，即后进先出的顺序，这是Activity默认的启动模式。但Activity有4种启动模式，其中3种会改变默认的后进先出的顺序。

示例：res/layout/activity_main.xml
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity" 
    android:orientation="vertical">

    <TextView
        android:layout_width="wrap_content"
         android:layout_height="wrap_content"
         android:text="这是主Activity" 
         android:textSize="20sp"/>
 
     <Button
         android:layout_width="wrap_content"
         android:layout_height="wrap_content"
         android:text="启动主Activity" 
         android:onClick="click1"/>
 
     <Button
         android:layout_width="wrap_content"
         android:layout_height="wrap_content"
         android:text="启动二Activity" 
         android:onClick="click2"/>
 
</LinearLayout>
```
src/cn.itcast.launchmode/MainActivity.java
```java
package cn.itcast.launchmode;

import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.view.View;

public class MainActivity extends Activity {

         @Override
         protected void onCreate(Bundle savedInstanceState) {
                 super.onCreate(savedInstanceState);
                 setContentView(R.layout.activity_main);
         }
 
         public void click1(View v){
                 Intent intent = new Intent(this,MainActivity.class);
                 startActivity(intent);
         }
 
         public void click2(View v){
                 Intent intent = new Intent(this,SecondActivity.class);
                 startActivity(intent);
         }
}
```
res/layout/activity_second.xml
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity" 
    android:orientation="vertical">

    <TextView
        android:layout_width="wrap_content"
         android:layout_height="wrap_content"
         android:text="这是第二个Activity" 
         android:textSize="20sp"/>
 
     <Button
         android:layout_width="wrap_content"
         android:layout_height="wrap_content"
         android:text="启动主Activity" 
         android:onClick="click1"/>
 
     <Button
         android:layout_width="wrap_content"
         android:layout_height="wrap_content"
         android:text="启动二Activity" 
         android:onClick="click2"/>
 
</LinearLayout>
```
src/cn.itcast.launchmode/SecondActivity.java
```java
package cn.itcast.launchmode;

import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.view.View;

public class SecondActivity extends Activity {

         @Override
         protected void onCreate(Bundle savedInstanceState) {
                 super.onCreate(savedInstanceState);
                 setContentView(R.layout.activity_second);
         }
 
         public void click1(View v){
                 Intent intent = new Intent(this,MainActivity.class);
                 startActivity(intent);
         }
 
         public void click2(View v){
                 Intent intent = new Intent(this,SecondActivity.class);
                 startActivity(intent);
         }
}
```
AndroidManifest.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="cn.itcast.launchmode"
    android:versionCode="1"
    android:versionName="1.0" >

    <uses-sdk
        android:minSdkVersion="8"
        android:targetSdkVersion="17" />
 
     <application
         android:allowBackup="true"
         android:icon="@drawable/ic_launcher"
         android:label="@string/app_name"
         android:theme="@style/AppTheme" >
         <activity
             android:name="cn.itcast.launchmode.MainActivity"
             android:label="@string/app_name" >
             <intent-filter>
                 <action android:name="android.intent.action.MAIN" />
 
                 <category android:name="android.intent.category.LAUNCHER" />
             </intent-filter>
         </activity>
         <activity
             android:launchMode="standard"
             android:name="cn.itcast.launchmode.SecondActivity">
         </activity>
     </application>
 
</manifest>
```
从上面的清单文件可以看到，当SecondActivity的Activity声明中的android:launchMode属性值为“standard”时，为标准模式，也就是后劲模式的栈模式。

运行界面：

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/232958rl4k583tp565t486.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/233000dv4jgv161jjqkqwv.png.thumb.jpg) 

1、当将SecondActivity的Activity声明中的android:launchMode属性值修改为“singleTop”。

通过点击按钮，操作顺序为：

1. 运行应用程序，显示主Activity

2. 再启动主Activity

3. 启动二Activity

4. 启动主Activity

5. 启动二Activity

6. 再点击按钮启动二Activity

可以看到没有任何效果，SingleTop的效果就是如果栈顶已经是二Activity了，就不会启动二Activity了。

## SingleTop
如果该Activity已经在栈顶，就不会再去创建了，如果不在栈顶，就会再去创建。

SingleTop的应用主要针对于可以隐式启动的一些应用程序，例如浏览器、拨号器等等，以防止恶意不断重复启动这些应用程序，造成无限退回才能最终退出的情况。

2、当将SecondActivity的Activity声明中的android:launchMode属性值修改为“singleTask”。

通过点击按钮，操作顺序为：

1. 运行应用程序，显示主Activity

2. 启动二Activity

3. 再启动二Activity（无效果，与SingleTop效果相同）

4. 启动主Activity

5. 再启动主Activity

6. 启动二Activity（有效果，返回到上次启动的二Activity，也就是将栈中的最近的两个主Activity都删除了，这时候栈中就只有一个二Activity和一个主Activity了）。

可以看到，SingleTask的效果就是第一次可以启动二Activity，以后再也不会启动了，以后都会返回到之前已经启动的二Activity，保证栈中只有一个二Activity的实例。

## SingleTask
如果该Activity在栈中没有实例，那么启动时会被创建，如果已经有实例了，启动时，会直接返回至该实例，也就是杀死该实例上方的所有Activity，让该Activity成为栈顶Activity。保证栈中只有一个该Activity的实例。

SingleTask的应用：浏览器的网页切换比较耗费资源，因为浏览器上只能显示一个网页，所以，页面切换来切换去都只是一个Activity。

3、当将SecondActivity的Activity声明中的android:launchMode属性值修改为“singleInstance”（单例）。

通过点击按钮，操作顺序为：

1. 运行应用程序，显示主Activity
2. 再启动主Activity
3. 启动二Activity（效果夸张，因为二Activity进入了一个不同于主Activity的栈，主Activity所在的栈进入后台）
4. 启动二Activity（无效果，因为是单例）
5. 再启动主Activity（效果夸张，因为切换到了主Activity所在的栈，二Activity所在的栈进入后台）
6. 启动二Activity（效果夸张，切换到二Activity所在的栈）
7. 点击返回键（效果夸张，切换到主Activity所在的栈）
8. 不断点击返回键（主Activity以此退栈）

如果操作顺序为：

1. 运行应用程序，显示主Activity
2. 再启动主Activity
3. 启动二Activity（效果夸张，因为二Activity进入了一个不同于主Activity的栈，主Activity所在的栈进入后台）
4. 启动二Activity（无效果，因为是单例）
5. 再启动主Activity（效果夸张，因为切换到了主Activity所在的栈，二Activity所在的栈进入后台）
6. 点击返回键（显示的竟然还是主Activity，因为返回键不会切换栈，也就是不会切换到二Activity所在的栈，所以依然在主Activity所在的栈操作）
7. 不断点击返回键（直到主Activity全部退出）
8. 最终才显示二Activity-->点击返回键（退出应用程序）

## SingleInstance
单例模式，Activity会再一个独立的任务栈中创建，且只会被创建一次，对该Activity的任何跳转操作，都是把该Activity的任务栈，显示至前台。SingleTask保证应用的栈中只有一个该Activity的实例，而SingleInstance保证整个系统的内存中只有一个该Activity的实例。

SingleInstance在实际开发中几乎不会被使用，只有在系统级应用中会被使用。例如：来电页面，因为任何时候，用户只能接一个电话。

## 横竖屏切换
手机中有一个重力传感器，可以检测手机的朝向，以控制手机横竖屏切换。在模拟器中，可以通过按下Ctrl+F11按钮，使模拟器横竖屏切换。

横竖屏切换时，会加载另一布局布局，当前Activity会销毁，然后重建。所以，不要在Activity的onCreate方法中对成员变量重新赋值，并且应该在Activity销毁时调用的onStop和onDestroy方法中保存所有的数据，在onCreate和onStart方法中将所有保存的数据重新赋值。

在清单文件文件中作如下的配置，横竖屏切换，不会再销毁重建Activity了，也不会加载另一套布局文件了。

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/235311zfsfstkpdfkpvvvp.png.thumb.jpg) 

当方向改变的时候，屏幕尺寸变化的时候（横竖屏切换，宽变高，高变宽），软键盘弹出，系统不要做任何的处理。

在清单文件文件中作如下的配置，可以写死横竖屏方向。

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/235637emm01ql1ed2alk5o.png.thumb.jpg) 

用代码方式写死横竖屏方向。

![img](http://bbs.itheima.com/data/attachment/forum/201507/02/235842z1c3janj3jzn31qy.png.thumb.jpg) 

PS :如果配置文件和代码都确定了横竖屏方向，那么因为配置文件先被解读，代码后被执行，因此以代码为准。

定义短信发送界面和选择联系人界面

代码：res/layout/activity_main.xml
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context=".MainActivity" 
     android:orientation="vertical">
 
     <LinearLayout 
         android:layout_width="match_parent"
         android:layout_height="wrap_content"
         android:orientation="horizontal"
         >
 
         <EditText 
             android:id="@+id/et_name"
                 android:layout_weight="1"
                 android:layout_width="0dp"
                 android:layout_height="wrap_content"
             />
 
         <Button 
             android:layout_width="wrap_content"
             android:layout_height="wrap_content"
             android:text="获取联系人"
             android:onClick="click"
             />
 
     </LinearLayout>
 
     <EditText 
         android:id="@+id/et_content"
         android:layout_weight="1"
         android:layout_width="match_parent"
         android:layout_height="0dp"
         />
 
     <Button 
             android:layout_width="wrap_content"
             android:layout_height="wrap_content"
             android:text="发送"
             />
 
</LinearLayout>
```
src/cn.itcast.transmitdataagain/MainActivity.java
```java
package cn.itcast.transmitdataagain;

import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.view.View;

public class MainActivity extends Activity {

         @Override
         protected void onCreate(Bundle savedInstanceState) {
                 super.onCreate(savedInstanceState);
                 setContentView(R.layout.activity_main);
         }
 
         public void click(View v){
                 Intent intent = new Intent(this, ContactsActivity.class);
                 startActivity(intent);
         }
}
```
res/layout/activity_contacts.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical" >

        <ListView 
            android:id="@+id/lv"
            android:layout_width="match_parent"
             android:layout_height="match_parent"
             ></ListView>
</LinearLayout>
```
res/layout/item_listview.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical" >

        <TextView 
            android:id="@+id/tv_name"
            android:layout_width="wrap_content"
             android:layout_height="wrap_content"
             android:textSize="18sp"
             />
 
</LinearLayout>
```
src/cn.itcast.transmitdataagain/ContactsActivity.java
```java
package cn.itcast.transmitdataagain;

import android.app.Activity;
import android.os.Bundle;
import android.widget.ArrayAdapter;
import android.widget.ListView;
import android.widget.TextView;

public class ContactsActivity extends Activity {
 
         @Override
         protected void onCreate(Bundle savedInstanceState) {
                 super.onCreate(savedInstanceState);
                 setContentView(R.layout.activity_contacts);
 
                 String[] names = new String[]{
                                 "凤姐",
                                 "凤姐夫",
                                 "凤姐的闺女"
                 };
                 ListView lv = (ListView) findViewById(R.id.lv);
                 lv.setAdapter(new ArrayAdapter<String>(this, R.layout.item_listview, //要填充成listview条目的布局文件的资源id
                                 R.id.tv_name, //字符串显示在哪个组件中
                                 names));
         }
 
}
```
AndroidManifest.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="cn.itcast.transmitdataagain"
    android:versionCode="1"
    android:versionName="1.0" >

    <uses-sdk
        android:minSdkVersion="8"
        android:targetSdkVersion="17" />
 
     <application
         android:allowBackup="true"
         android:icon="@drawable/ic_launcher"
         android:label="@string/app_name"
         android:theme="@style/AppTheme" >
         <activity
             android:name="cn.itcast.transmitdataagain.MainActivity"
             android:label="@string/app_name" >
             <intent-filter>
                 <action android:name="android.intent.action.MAIN" />
 
                 <category android:name="android.intent.category.LAUNCHER" />
             </intent-filter>
         </activity>
         <activity android:name="cn.itcast.transmitdataagain.ContactsActivity" />
     </application>
 
</manifest>
```
运行结果：

![img](http://bbs.itheima.com/data/attachment/forum/201507/03/114746z0rgy2b379s0hq8b.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201507/03/114747mhrbej8qhyhqj3zg.png.thumb.jpg) 

联系人界面销毁时传递数据

示例：src/cn.itcast.transmitdataagain/ContactsActivity.java
```java
package cn.itcast.transmitdataagain;

import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.AdapterView;
import android.widget.AdapterView.OnItemClickListener;
import android.widget.ArrayAdapter;
 import android.widget.ListView;
 
 public class ContactsActivity extends Activity {
 
         @Override
         protected void onCreate(Bundle savedInstanceState) {
                 super.onCreate(savedInstanceState);
                 setContentView(R.layout.activity_contacts);
 
                 final String[] names = new String[]{
                                 "凤姐",
                                 "凤姐夫",
                                 "凤姐的闺女"
                 };
                 ListView lv = (ListView) findViewById(R.id.lv);
                 lv.setAdapter(new ArrayAdapter<String>(this, R.layout.item_listview, 
                                 R.id.tv_name, 
                                 names));
 
                 //设置条目侦听，可以理解为给条目设置侦听
                 lv.setOnItemClickListener(new OnItemClickListener() {
 
                         //当listView的条目被点击时，此方法调用
                         //position：点击的条目的索引
                         @Override
                         public void onItemClick(AdapterView<?> parent, View view, int position,
                                         long id) {
 
                                 //把要传递的数据封装至intent对象中
                                 Intent data = new Intent();
                                 data.putExtra("name", names[position]);
 
                                 //第一个参数：结果码，0，表示不用
                                 //第二个参数：意图，当前Activity销毁时，这个意图就会传递给上一个Activity（启动当前Activity的Activity）
                                 setResult(0, data);
 
                                 //销毁当前Activity
                                 finish();
                         }
                 });
         }
}
```
src/cn.itcast.transmitdataagain/MainActivity.java
```java
package cn.itcast.transmitdataagain;

import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.EditText;

public class MainActivity extends Activity {
 
         @Override
         protected void onCreate(Bundle savedInstanceState) {
                 super.onCreate(savedInstanceState);
                 setContentView(R.layout.activity_main);
         }
 
         public void click(View v){
                 Intent intent = new Intent(this, ContactsActivity.class);
                 //为了结果去启动Activity，那么当ContactsActivity销毁时，系统才会调用onActivityResult
                 //第二个参数：请求码，整型
                 startActivityForResult(intent, 0);
         }
 
         //当某个Activity销毁时，把意图传递给MainActivity时，此方法调用
         @Override
         protected void onActivityResult(int requestCode, int resultCode, Intent data) {
                 super.onActivityResult(requestCode, resultCode, data);
 
                 //从intent取出传递过来的数据
                 String name = data.getStringExtra("name");
                 EditText et_name = (EditText) findViewById(R.id.et_name);
 
                 //把数据回显至输入框中
                 et_name.setText(name);
         }
}
```
运行结果：

![img](http://bbs.itheima.com/data/attachment/forum/201507/03/120841jd6v18k8806fdzl7.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201507/03/120842d6gsmfgva9148v1g.png.thumb.jpg) 

点击3个Item中的“凤姐”，结果如下：

![img](http://bbs.itheima.com/data/attachment/forum/201507/03/120843dup2it0sstjfvf1f.png.thumb.jpg) 

通过请求码判断数据来自哪个Activity

示例：res/layout/activity_main.xml
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context=".MainActivity" 
     android:orientation="vertical">
 
     <LinearLayout 
         android:layout_width="match_parent"
         android:layout_height="wrap_content"
         android:orientation="horizontal"
         >
 
         <EditText 
             android:id="@+id/et_name"
                 android:layout_weight="1"
                 android:layout_width="0dp"
                 android:layout_height="wrap_content"
             />
 
         <Button 
             android:layout_width="wrap_content"
             android:layout_height="wrap_content"
             android:text="获取联系人"
             android:onClick="click"
             />
 
     </LinearLayout>
 
     <EditText 
         android:id="@+id/et_content"
         android:layout_weight="1"
         android:layout_width="match_parent"
         android:layout_height="0dp"
         />
 
         <Button 
                 android:layout_width="wrap_content"
                 android:layout_height="wrap_content"
                 android:text="选择快捷短信"
                 android:onClick="click2"
         />
 
     <Button 
             android:layout_width="wrap_content"
             android:layout_height="wrap_content"
             android:text="发送"
             />
 
</LinearLayout>
```
src/cn.itcast.transmitdataagain/SmsActivity.java
```java
package cn.itcast.transmitdataagain;

import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.AdapterView;
import android.widget.ArrayAdapter;
import android.widget.ListView;
 import android.widget.AdapterView.OnItemClickListener;
 
 public class SmsActivity extends Activity {
 
         @Override
         protected void onCreate(Bundle savedInstanceState) {
                 super.onCreate(savedInstanceState);
                 setContentView(R.layout.activity_contacts);
 
                 final String[] contents = new String[]{
                                 "人在塔在",
                                 "云鹤正在跪键盘，一会儿他会回复你",
                                 "云鹤正在捡肥皂，一会儿他会回复你"
                 };
                 ListView lv = (ListView) findViewById(R.id.lv);
                 lv.setAdapter(new ArrayAdapter<String>(this, R.layout.item_listview, 
                                 R.id.tv_name, 
                                 contents));
 
                 lv.setOnItemClickListener(new OnItemClickListener() {
 
                         @Override
                         public void onItemClick(AdapterView<?> parent, View view, int position,
                                         long id) {
 
                                 Intent data = new Intent();
                                 data.putExtra("content", contents[position]);
 
                                 setResult(0, data);
 
                                 finish();
                         }
                 });
         }
}
```
src/cn.itcast.transmitdataagain/MainActivity.java
```java
package cn.itcast.transmitdataagain;

import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.EditText;

public class MainActivity extends Activity {
 
         static final int CONTACTS_SELECT = 10;
         static final int SMS_SELECT = 20;
 
         @Override
         protected void onCreate(Bundle savedInstanceState) {
                 super.onCreate(savedInstanceState);
                 setContentView(R.layout.activity_main);
         }
 
         public void click(View v){
                 Intent intent = new Intent(this, ContactsActivity.class);
                 startActivityForResult(intent, CONTACTS_SELECT);
         }
 
         public void click2(View v){
                 Intent intent = new Intent(this, SmsActivity.class);
                 startActivityForResult(intent, SMS_SELECT);
         }
 
         @Override
         protected void onActivityResult(int requestCode, int resultCode, Intent data) {
                 super.onActivityResult(requestCode, resultCode, data);
 
                 if(requestCode == CONTACTS_SELECT){
                         String name = data.getStringExtra("name");
                         EditText et_name = (EditText) findViewById(R.id.et_name);
                         et_name.setText(name);
                 }else if(requestCode == SMS_SELECT){
                         String content = data.getStringExtra("content");
                         EditText et_content = (EditText) findViewById(R.id.et_content);
                         et_content.setText(content);
                 }
         }
}
```
AndroidManifest.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="cn.itcast.transmitdataagain"
    android:versionCode="1"
    android:versionName="1.0" >

    <uses-sdk
        android:minSdkVersion="8"
        android:targetSdkVersion="17" />
 
     <application
         android:allowBackup="true"
         android:icon="@drawable/ic_launcher"
         android:label="@string/app_name"
         android:theme="@style/AppTheme" >
         <activity
             android:name="cn.itcast.transmitdataagain.MainActivity"
             android:label="@string/app_name" >
             <intent-filter>
                 <action android:name="android.intent.action.MAIN" />
 
                 <category android:name="android.intent.category.LAUNCHER" />
             </intent-filter>
         </activity>
         <activity android:name="cn.itcast.transmitdataagain.ContactsActivity" />
         <activity android:name="cn.itcast.transmitdataagain.SmsActivity" />
     </application>
 
</manifest>
```
运行结果：

![img](http://bbs.itheima.com/data/attachment/forum/201507/03/143027nqj2ndj8yj3drv52.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201507/03/143029db26qy4c1445aba8.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201507/03/143031baf9xr88o9azf2ja.png.thumb.jpg) 

请求码和结果码共同作用

通过结果码也可以实现和上面同样的效果。

示例：src/cn.itcast.transmitdataagain/ContactsActivity.java

![img](http://bbs.itheima.com/data/attachment/forum/201507/03/143521b0qznn9f57tiz73e.png.thumb.jpg) 

src/cn.itcast.transmitdataagain/SmsActivity.java

![img](http://bbs.itheima.com/data/attachment/forum/201507/03/143521duk3ycuhkmkc3vf1.png.thumb.jpg) 

src/cn.itcast.transmitdataagain/MainActivity.java

![img](http://bbs.itheima.com/data/attachment/forum/201507/03/143522xvwuyvpyfavloago.png.thumb.jpg) 

请求码和结果码结合，不但可以区分出数据的来自哪个Activity，还可以区分出是什么数据（一个Activity可能传来不同类型的数据）。

- requestCode：判断来自哪一个Activity。
- resultCode：判断传递过来的数据属于什么类型。

示例：res/layout/activity_main.xml
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context=".MainActivity" 
     android:orientation="vertical">
 
     <LinearLayout 
         android:layout_width="match_parent"
         android:layout_height="wrap_content"
         android:orientation="horizontal"
         >
 
         <EditText 
             android:id="@+id/et_name"
                 android:layout_weight="1"
                 android:layout_width="0dp"
                 android:layout_height="wrap_content"
             />
 
         <Button 
             android:layout_width="wrap_content"
             android:layout_height="wrap_content"
             android:text="获取联系人"
             android:onClick="click"
             />
 
     </LinearLayout>
 
     <EditText 
         android:id="@+id/et_content"
         android:layout_weight="1"
         android:layout_width="match_parent"
         android:layout_height="0dp"
         />
 
         <Button 
                 android:layout_width="wrap_content"
                 android:layout_height="wrap_content"
                 android:text="选择快捷短信"
                 android:onClick="click2"
         />
 
     <Button 
             android:layout_width="wrap_content"
             android:layout_height="wrap_content"
             android:text="发送"
             />
 
     <Button 
             android:layout_width="wrap_content"
             android:layout_height="wrap_content"
             android:text="机主信息"
             android:onClick="click3"
             />
 
</LinearLayout>
```
res/layout/activity_master.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical" >

        <TextView 
            android:id="@+id/tv_mastername"
            android:layout_width="wrap_content"
             android:layout_height="wrap_content"
             android:text="云鹤"
             />
 
         <TextView 
             android:id="@+id/tv_mastercontent"
             android:layout_width="wrap_content"
             android:layout_height="wrap_content"
             android:text="请把手机还给我"
             />
</LinearLayout>
```
src/cn.itcast.transmitdataagain/MasterActivity.java
```java
package cn.itcast.transmitdataagain;

import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.TextView;

 public class MasterActivity extends Activity {
 
         static final int MASTER_NAME = 1;
         static final int MASTER_CONTENT = 2;
 
         @Override
         protected void onCreate(Bundle savedInstanceState) {
                 super.onCreate(savedInstanceState);
                 setContentView(R.layout.activity_master);
 
                 TextView tv_mastername = (TextView) findViewById(R.id.tv_mastername);
                 TextView tv_mastercontent = (TextView) findViewById(R.id.tv_mastercontent);
 
                 tv_mastername.setOnClickListener(new OnClickListener() {
 
                         @Override
                         public void onClick(View v) {
                                 Intent data = new Intent();
                                 data.putExtra("name", "云鹤");
                                 setResult(MASTER_NAME, data);
                                 finish();
                         }
                 });
 
                 tv_mastercontent.setOnClickListener(new OnClickListener() {
 
                         @Override
                         public void onClick(View v) {
                                 Intent data = new Intent();
                                 data.putExtra("content", "请把我的手机还给我");
                                 setResult(MASTER_CONTENT, data);
                                 finish();
                         }
                 });
         }
}
```

src/cn.itcast.transmitdataagain/MainActivity.java
```java
package cn.itcast.transmitdataagain;

import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.EditText;

public class MainActivity extends Activity {
 
         static final int CONTACTS_SELECT = 10;
         static final int SMS_SELECT = 20;
         static final int MASTER_SELECT = 30;
 
         @Override
         protected void onCreate(Bundle savedInstanceState) {
                 super.onCreate(savedInstanceState);
                 setContentView(R.layout.activity_main);
         }
 
         public void click(View v){
                 Intent intent = new Intent(this, ContactsActivity.class);
                 startActivityForResult(intent, CONTACTS_SELECT);
         }
 
         public void click2(View v){
                 Intent intent = new Intent(this, SmsActivity.class);
                 startActivityForResult(intent, SMS_SELECT);
         }
 
         public void click3(View v){
                 Intent intent = new Intent(this, MasterActivity.class);
                 startActivityForResult(intent, MASTER_SELECT);
         }
 
         @Override
         protected void onActivityResult(int requestCode, int resultCode, Intent data) {
                 super.onActivityResult(requestCode, resultCode, data);
 
                 if(resultCode == CONTACTS_SELECT){
                         String name = data.getStringExtra("name");
                         EditText et_name = (EditText) findViewById(R.id.et_name);
                         et_name.setText(name);
                 }else if(resultCode == SMS_SELECT){
                         String content = data.getStringExtra("content");
                         EditText et_content = (EditText) findViewById(R.id.et_content);
                         et_content.setText(content);
                 }else if(resultCode == MASTER_SELECT){
                         if(requestCode == MasterActivity.MASTER_NAME){
                                 String name = data.getStringExtra("name");
                                 EditText et_name = (EditText) findViewById(R.id.et_name);
                                 et_name.setText(name);
                         }else if(requestCode == MasterActivity.MASTER_CONTENT){
                                 String content = data.getStringExtra("content");
                                 EditText et_content = (EditText) findViewById(R.id.et_content);
                                 et_content.setText(content);
                         }
                 }
         }
}
```
AndroidManifest.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="cn.itcast.transmitdataagain"
    android:versionCode="1"
    android:versionName="1.0" >

    <uses-sdk
        android:minSdkVersion="8"
        android:targetSdkVersion="17" />
 
     <application
         android:allowBackup="true"
         android:icon="@drawable/ic_launcher"
         android:label="@string/app_name"
         android:theme="@style/AppTheme" >
         <activity
             android:name="cn.itcast.transmitdataagain.MainActivity"
             android:label="@string/app_name" >
             <intent-filter>
                 <action android:name="android.intent.action.MAIN" />
 
                 <category android:name="android.intent.category.LAUNCHER" />
             </intent-filter>
         </activity>
         <activity android:name="cn.itcast.transmitdataagain.ContactsActivity" />
         <activity android:name="cn.itcast.transmitdataagain.SmsActivity" />
         <activity android:name="cn.itcast.transmitdataagain.MasterActivity" />
     </application>
 
</manifest>
```

运行结果：

![img](http://bbs.itheima.com/data/attachment/forum/201507/03/150306snrnnnv55vpd555d.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201507/03/150308t65h29jdgpwa9hqd.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201507/03/150309xeuj7eb7cx0b70qz.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201507/03/150311pcw87og4s2r942s7.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201507/03/150312cgdzrlr9sq26qbg4.png.thumb.jpg) 