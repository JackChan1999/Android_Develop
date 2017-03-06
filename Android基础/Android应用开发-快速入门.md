# **1. 网络制式的概念**
![Android](http://img.blog.csdn.net/20151020120029786)
![Android](http://img.blog.csdn.net/20151020120046039)
![Android](http://img.blog.csdn.net/20151020120059716)
![Android](http://img.blog.csdn.net/20151020120111481)
![Android](http://img.blog.csdn.net/20151020120124662)
![Android](http://img.blog.csdn.net/20151020120216137)

# **2. Android简单历史**
![Android](http://img.blog.csdn.net/20151020120141344)
![Android](http://img.blog.csdn.net/20151020120151005)

# **3. Android项目的目录结构**
* src：项目代码
* gen
  * buildConfig：应用是否可以debug
  * R.java：项目中所有资源文件的资源id
* Android.jar：Android的jar包，导入此包方可使用Android的api
* assets：存放资源文件，比方说mp3、视频文件
* bin：存放编译打包后的文件
* libs：导入第三方jar包
* res：存放资源文件，存放在此文件夹下的所有资源文件都会生成资源id
  * drawable：存放图片资源
  * layout：存放布局文件，把布局文件通过资源id指定给activity，界面就会显示出该布局文件定义的布局
  * menu：定义菜单的样式
  * values：字符串
     * Strings.xml：存放字符串资源，每个资源都会有一个资源id
     * dimens.xml：长度资源文件，用来定义长度资源
     * style.xml：样式和主题资源文件
* AndroidManifest.xml：配置文件

## **3.1 Android的配置文件（清单文件）**
* 指定应用的包名

```
package="com.itheima.helloworld"
```
data/data/com.itheima.helloworld(上面代码指定的包名)
应用生成的文件都会存放在此路径下

* Android的四大组件在使用前全部需要在清单文件中配置
* `<Application/>`的配置对整个应用生效
* `<activity/>`的配置对该activity生效

# **4. DDMS**

* Dalvik debug monitor service
* Dalvik调试监控服务
# **5. Dalvik VM和JVM的比较**

![dvm](http://img.blog.csdn.net/20151020120813503)

* sdk把所有的class文件都编译成一个dex文件
  ![Android](http://img.blog.csdn.net/20151020121325498)
## **5.1 ART**

![Android](http://img.blog.csdn.net/20151020121716903)

# **6. Android开发环境搭建**

![sdk](http://img.blog.csdn.net/20151020121902335)
# **7. 常用的adb指令**

###Android debug bridge：安卓调试桥
| 命令                  | 说明                           |
| ------------------- | ---------------------------- |
| adb start-server    | 启动adb进程                      |
| adb kill-server     | 杀死adb进程                      |
| adb devices         | 查看当前与开发环境连接的设备，此命令也可以启动adb进程 |
| adb install XXX.apk | 往模拟器安装apk                    |
| adb uninstall       | 包名：删除模拟器中的应用                 |
| adb shell           | 进入linux命令行                   |
| ps                  | 查看运行进程                       |
| ls                  | 查看当前目录下的文件结构                 |
| netstat -ano        | 查看占用端口的进程                    |
| ipconfig            | 查看IP地址                       |

# **8. 电话拨号器**

>功能：用户输入一个号码，点击拨打按钮，启动系统打电话的应用把号码拨打出去
## 8.1定义布局

- 组件必须设置宽高，否则不能通过编译

```xml
android:layout_width="wrap_content"
android:layout_height="wrap_content"
```
- 如果要在java代码中操作某个组件，则组件需要设置id，这样才能在代码中通过id拿到这个组件

```xml
android:id="@+id/et_phone"
```

## 8.2给按钮设置点击侦听

1. 给按钮设置侦听

```java
//通过id拿到按钮对象
Button bt_call = (Button) findViewById(R.id.bt_call);
//给按钮设置点击
bt_call.setOnClickListener(new MyListener());
```

## 8.3 得到用户输入的号码

```java
//得到用户输入的号码，先拿到输入框组件
EditText et_phone = (EditText) findViewById(R.id.et_phone);
String phone = et_phone.getText().toString();
```

## 8.4 把号码打出去

1. Android系统中基于动作机制，来调用系统的应用，你告诉系统你想做什么动作，系统就会把能做这个动作的应用给你，如果没有这个应用，会抛异常
2. 设置动作，通过意图告知系统

```java
//把号码打出去
//先创建一个意图对象
Intent intent = new Intent();
//设置动作，打电话
intent.setAction(Intent.ACTION_CALL);
intent.setData(Uri.parse("tel:" + phone));
//把意图告诉系统
startActivity(intent);
```
添加权限
```xml
<uses-permission android:name="android.permission.CALL_PHONE"/>
```

## 8.5 实现代码

```java
package com.itheima.dialer;

import android.net.Uri;
import android.os.Bundle;
import android.app.Activity;
import android.content.Intent;
import android.view.Menu;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;
import android.widget.EditText;

public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        //给按钮设置点击侦听
        //1.拿到按钮对象
        Button bt = (Button) findViewById(R.id.bt_call);
        //2.设置侦听
        bt.setOnClickListener(new MyListener());
    }

    class MyListener implements OnClickListener{

    	//按钮被点击时，此方法调用
		@Override
		public void onClick(View v) {
			//获取用户输入的号码
			EditText et = (EditText) findViewById(R.id.et_phone);
			String phone = et.getText().toString();
			
			//我们需要告诉系统，我们的动作：我要打电话
			//创建意图对象
			Intent intent = new Intent();
			//把动作封装至意图对象当中
			intent.setAction(Intent.ACTION_CALL);
			//设置打给谁
			intent.setData(Uri.parse("tel:" + phone));
			
			//把动作告诉系统
			startActivity(intent);
		}
    	
    }
    
}
```
布局文件
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity" 
    android:orientation="vertical"
    >

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="请输入号码" />
    <EditText 
        android:id="@+id/et_phone"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        />
    <Button
        android:id="@+id/bt_call" 
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="拨打"
        />

</LinearLayout>
```
# **9. 点击事件的四种写法**

## 9.1 定义一个MyListener实现onClickListener接口

```java
Button bt1 = (Button) findViewById(R.id.bt1);
bt1.setOnClickListener(new MyListener());
```

## 9.2 定义一个匿名内部类实现onClickListener接口

```java
Button bt2 = (Button) findViewById(R.id.bt2);
bt2.setOnClickListener(new OnClickListener() {
    @Override
    public void onClick(View v) {
        System.out.println("第二种");
    }
});
```

## 9.3 让当前activity实现onClickListener接口

```java
Button bt3 = (Button) findViewById(R.id.bt3);
bt3.setOnClickListener(this);
```

## 9.4 给Button节点设置onClick属性

android:onClick="click"

* 然后在activity中定义跟该属性值同名的方法

```java
public void click(View v){
    System.out.println("第四种");
}
```
## 9.5 按钮点击事件代码

```java
package com.itheima.clickevent;

import android.os.Bundle;
import android.app.Activity;
import android.view.Menu;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;

public class MainActivity extends Activity implements OnClickListener{

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        Button bt1 = (Button) findViewById(R.id.bt1);
        bt1.setOnClickListener(new OnClickListener() {
			
			@Override
			public void onClick(View v) {
				System.out.println("第一个按钮被点击了");
			}
		});
        Button bt2 = (Button) findViewById(R.id.bt2);
        bt2.setOnClickListener(this);
    }

	@Override
	public void onClick(View v) {
		System.out.println("第二个按钮被点击了");
		
	}

	//View：系统会把触发这个方法的那个组件的对象作为view对象传进来
	public void getScore(View v){
		//通过对view对象的判断，就可以知道用户点击的到底是哪一个按钮
		//拿到按钮的id
		int id = v.getId();
		switch (id) {
		case R.id.wangzhe:
			System.out.println("下辈子吧");
			break;
		case R.id.diamond:
			System.out.println("凑合凑合");
			break;
		case R.id.master:
			System.out.println("想想就好");
			break;
		}
	}
}
```
# **10. 短信发送器**

功能：用户输入号码和短信内容，点击发送按钮，调用短信api把短信发送给指定号码

## 10.1定义布局

* 输入框的提示
  android:hint="请输入号码"  

## 10.2 完成点击事件

* 先给Button组件设置onClick属性
  onClick="send"
* 在Activity中定义此方法
  public void send(View v){}
## 10.3 获取到用户输入的号码和内容

```java
EditText et_phone = (EditText) findViewById(R.id.et_phone);
EditText et_content = (EditText) findViewById(R.id.et_content);
String phone = et_phone.getText().toString();
String content = et_content.getText().toString();
```
## 10.4 调用发送短信的api

```java
//调用发送短信的api
SmsManager sm = SmsManager.getDefault();
//发送短信
sm.sendTextMessage(phone, null, content, null, null);
```

## 10.5 添加权限

```xml
<uses-permission android:name="android.permission.SEND_SMS"/>
```
## 10.6 如果短信过长，需要拆分

```java
List<String> smss = sm.divideMessage(content);
```
## 10.7 短信发送器代码

```java
package com.itheima.smssender;

import java.util.ArrayList;

import android.os.Bundle;
import android.app.Activity;
import android.telephony.SmsManager;
import android.view.Menu;
import android.view.View;
import android.widget.EditText;

public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }


    public void send(View v){
    	//拿到用户输入的号码和内容
    	EditText et_phone = (EditText) findViewById(R.id.et_phone);
    	EditText et_content = (EditText) findViewById(R.id.et_content);
    	
    	String phone = et_phone.getText().toString();
    	String content = et_content.getText().toString();
    	
    	//1.获取短信管理器
    	SmsManager sm = SmsManager.getDefault();
    	
    	//2.切割短信，把长短信分成若干个小短信
    	ArrayList<String> smss = sm.divideMessage(content);
    	
    	//3.for循环把集合中所有短信全部发出去
    	for (String string : smss) {
			
    		sm.sendTextMessage(phone, null, string, null, null);
		}
    }
    
}
```
布局文件
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
    android:orientation="vertical"
     >

    <EditText
        android:id="@+id/et_phone"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:inputType="phone"
        android:hint="请输入对方号码"
        />
    <EditText 
        android:id="@+id/et_content"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:lines="5"
        android:hint="请输入短信内容"
        android:gravity="top"
        />
    <Button 
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="发送"
        android:onClick="send"
        />

</LinearLayout>
```
# **11. 按钮点击事件**

1、 按钮设置onclick属性，实现click方法
```
  <Button
       android:onClick="click"
       android:layout_width="wrap_content"
       android:layout_height="wrap_content"/>
       
 public void click(){
 }
```

  2、 所在的类实现OnClickListener接口
  3、 内部类，调用setOnClickListener()方法
  4、 匿名内部类，调用setOnClickListener()方法
```Java
bt_dail.setOnClickListener(new OnClickListener() {
            
    @Override
    public void onClick(View v) {
        EditText et= (EditText) findViewById(R.id.et_dail);
        String phone = et.getText().toString().trim();
        Intent intent = new Intent();
        intent.setAction(Intent.ACTION_CALL);
        intent.setData(Uri.parse("tel:"+phone));
        startActivity(intent);
    }
});
```

# 12. Android项目目录结构
- src：项目的java代码
- gen
  * buildConfig：应用是否可以debug
  * R：保存项目中使用的资源的id
- Android.jar：导入这个包，应用才可以使用Android的api
- libs：存放第三方jar包
- assets：资源文件夹，存放视频或者音乐等较大的资源文件
- bin：存放应用打包编译后的文件
- res：资源文件夹，在这个文件夹中的所有资源，都会有资源id，读取时通过资源id就可以读取
  * 资源id不能出现中文
- layout:布局文件夹，保存布局文件，Android中所有布局文件都是xml文件
- menu：菜单配置文件夹，保存菜单的配置文件，决定菜单的样式
- values
  - strings:字符串资源文件，用来定义字符串资源的
  - dimens：长度资源文件，用来定义长度资源
  - style：样式和主题资源文件
- 清单文件
  - package：应用在系统中的唯一识别
  - versionCode:应用的版本号
  - 具有以下子节点的activity就是入口activity

```xml
<intent-filter>
    <action android:name="android.intent.action.MAIN" />
    <category android:name="android.intent.category.LAUNCHER" />
</intent-filter>
```
# 13. 安装路径

* 第三方应用保存路径：data/app
* 系统应用保存路径：system/app
* data/data/包名文件夹：系统为每一个应用提供的一个专属空间
# 14. DDMS
Dalvik debug monitor service

# 15. ADB

Android debug bridge
* 建立eclipse和Android设备之间的连接

## 15.1 ADB指令

* adb start-server：启动adb进程
* adb kill-server：杀死adb进程
* adb install E:\yyh.apk
* adb uninstall 应用包名
* adb devices:列出与开发环境建立连接的android设备的列表
* adb shell:进入Android命令行
* Android的指令：
    * ls：罗列出当前目录下的所有文件和文件夹
    * ps：罗列出当前系统运行的所有进程
* netstat -ano:查看系统的端口占用情况

# 案例1：电话拨号器

```java
public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        //给按钮设置点击侦听
        //1.拿到按钮对象
        Button bt = (Button) findViewById(R.id.bt_call);
        //2.设置侦听
        bt.setOnClickListener(new MyListener());
    }

    class MyListener implements OnClickListener{

        //按钮被点击时，此方法调用
        @Override
        public void onClick(View v) {
            //获取用户输入的号码
            EditText et = (EditText) findViewById(R.id.et_phone);
            String phone = et.getText().toString();
            
            //我们需要告诉系统，我们的动作：我要打电话
            //创建意图对象
            Intent intent = new Intent();
            //把动作封装至意图对象当中
            intent.setAction(Intent.ACTION_CALL);
            //设置打给谁
            intent.setData(Uri.parse("tel:" + phone));
            
            //把动作告诉系统
            startActivity(intent);
        }
    } 
}

```
# 案例2：按钮的点击事件

```java
public class MainActivity extends Activity implements OnClickListener{

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        Button bt1 = (Button) findViewById(R.id.bt1);
        bt1.setOnClickListener(new OnClickListener() {
            
            @Override
            public void onClick(View v) {
                System.out.println("第一个按钮被点击了");
            }
        });
        Button bt2 = (Button) findViewById(R.id.bt2);
        bt2.setOnClickListener(this);
    }

    @Override
    public void onClick(View v) {
        System.out.println("第二个按钮被点击了");
        
    }

    //View：系统会把触发这个方法的那个组件的对象作为view对象传进来
    public void getScore(View v){
        //通过对view对象的判断，就可以知道用户点击的到底是哪一个按钮
        //拿到按钮的id
        int id = v.getId();
        switch (id) {
        case R.id.wangzhe:
            System.out.println("下辈子吧");
            break;
        case R.id.diamond:
            System.out.println("凑合凑合");
            break;
        case R.id.master:
            System.out.println("想想就好");
            break;

        }
    }
}

```
# 案例3：短信发送器

```java
public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    public void send(View v){
        //拿到用户输入的号码和内容
        EditText et_phone = (EditText) findViewById(R.id.et_phone);
        EditText et_content = (EditText) findViewById(R.id.et_content);
        
        String phone = et_phone.getText().toString();
        String content = et_content.getText().toString();
        
        //1.获取短信管理器
        SmsManager sm = SmsManager.getDefault();
        
        //2.切割短信，把长短信分成若干个小短信
        ArrayList<String> smss = sm.divideMessage(content);
        
        //3.for循环把集合中所有短信全部发出去
        for (String string : smss) {
            
            sm.sendTextMessage(phone, null, string, null, null);
        }
    }
}
```

