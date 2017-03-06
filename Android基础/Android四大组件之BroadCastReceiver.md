# **1. 基本概念**
在Android 中，Broadcast 是一种广泛运用的在应用程序之间传输信息的机制。而BroadcastReceiver 是对发送出来的Broadcast 进行过滤接受并响应的一类组件，是Android 四大组件之一。

广播接收者（BroadcastReceiver）用于接收广播的，广播的发送是通过调用sendBroadcast（Intent），sendOrderedBroadcast（Intent）来实现的。通常一个广播可以被多个广播接收者所接收。

广播被分为两种不同的类型：“普通广播（Normal Broadcasts）”也叫无序广播和“有序广播（OrderedBroadcasts）”。

1、普通广播是完全异步（就是不会被某个广播接收者终止）的，可以在同一时刻（逻辑上）被所有接收者接收到（其实被接收者接收到也是由顺序的，接收者配置的优先级越高，越先接收到，也就是说广
播接收者的优先级对于无序广播也是有用的），消息传递的效率比较高，但缺点是：接收者不能将处理结果传递给下一个接收者，并且无法终止广播的传播。

2、有序广播是按照接收者声明的优先级别，被接收者依次接收广播。如：A 接收者的级别高于B，B的级别高于C，那么，广播先传给A，再传给B，最后传给C 。在传递的过程中如果有某个接收者终止（abortBroadCast）了该广播，那么后面的接收者就接收不到该广播。

3、广播接收者属于四大组件之一，因此通常需要AndroidManifest.xml 中进行注册，优先级别声明在intent-filter 元素的android:priority 属性中，数越大优先级别越高，取值范围:-1000 到1000，优先级别也可以调用IntentFilter 对象的setPriority()进行设置。

4、有序广播的接收者可以终止广播的传播，广播的传播一旦终止，后面的接收者就无法接收到广播，有序广播的接收者可以将数据传递给下一个接收者，如：A 得到广播后，可以往它的结果对象中存入数据，当广播传给B 时，B 可以从A 的结果对象中得到A 存入的数据。

5、Context.sendBroadcast() 发送的是普通广播，所有订阅者都有机会获得并进行处理。

6、Context.sendOrderedBroadcast() 发送的是有序广播，系统会根据接收者声明的优先级别按顺序逐个执行接收者，前面的接收者有权终止广播(BroadcastReceiver.abortBroadcast())，如果广播被前面的接收者终
止， 后面的接收者就再也无法获取到广播。对于有序广播， 前面的接收者可以将数据通过setResultExtras(Bundle)方法存放进结果对象，然后传给下一个接收者，下一个接收者通过代码：Bundle bundle = getResultExtras(true))可以获取上一个接收者存入在结果对象中的据。

# **2. Android 系统常见的广播**
Android 为了将系统运行时的各种“事件”通知给其他应用（或者说通知给我们程序员，让我们程序员好做出相应的反应。举个生活中的例子：比如我们坐火车，当前方到达某站的时候，火车乘务员会给所有乘客发送即将到站的广播，这样乘客收到广播后就可以提前准备下车），因此内置了多种广播，比如：系统电量的改变、屏幕的锁屏、网络状态的改变、接收到新的短信、拨打电话事件、sdcard 的挂载和移除、应用的安装和卸载等等。比如我们开发的在线播放视频类的APP，那么我们就有必要监听网络转态改变的事件广播，如果用户的网络状态从wifi 改变为了4G 上网，那么应该提示用户是否使用4G 网络继续播放视频，如果不提示用户，那么就可能导致用户流量被大量使用，一会儿功夫，用户可能就要停机了。

接下来我们会用4 个案例来演示广播接收者的使用。

## **2.1 案例-IP 拨号器**
**需求分析**

什么是IP 拨号服务？我们为什么要用IP 服务？所谓的IP 拨号就是通过接入数据网络来传播语音信息。IP 拨号的目的在于转接至其他频道，减少话费等用处。移动17951，联通17911，打长途时在电话号码前加上这个就便宜了，如果你的手机上有这个键的话，那么打电话时输入长途电话号码后，直接按那个键就拨出去了，它会自动加上IP。通俗的说就是打长途便宜。

例如手机拨打长途电话：
移动拨区号+电话号=0.25/分市话+0.7/分长途=0.95/分；
移动拨17951+区号+电话号=0.25/分市话+0.3/分长途=0.55/分。

![BroadcastReceiver](http://img.blog.csdn.net/20160911145040840)

了解了IP 拨号的用途之后，接下来，我们通过程序在用户拨出去的号码前自动加上一个IP 号码，为用户省钱。

之所以能实现这样的功能，是因为拨号的时候Android 系统会发送一个有序广播，该广播中携带了用户拨打的号码，我们通过注册广播接收者就可以获取到该广播，同时将该广播中的数据进行修改。从而实现了用户号码自动加IP 号的功能。

为了能让用户自己决定IP 号码，我们需要一个界面（如图1-2），让那个用户输入IP 号码，然后将该IP 号码保存到SharedPreferences 中。

![IP 拨号器界面](http://img.blog.csdn.net/20160911145220509)

**布局文件如下**

```java
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              xmlns:tools="http://schemas.android.com/tools"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:orientation="vertical">

    <EditText
        android:id="@+id/et_ip"
        android:hint="请输入IP 号码，默认17951"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
    <Button
        android:onClick="saveIP"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="保存"
        />

</LinearLayout>

```
**实现代码**
在该案例中总共用到了两个类一个是MainActivity.java 负责让用户输入IP 号码，另外一个是自定义的广播接收者IPCallerReceiver 负责监听用户的拨打电话事件。

```java
    import android.os.Bundle;
    import android.app.Activity;
    import android.content.SharedPreferences;
    import android.text.TextUtils;
    import android.view.View;
    import android.widget.EditText;
    import android.widget.Toast;

    //保存用户的IP 号码
    public class MainActivity extends Activity {

        private EditText          et_ip;
        private SharedPreferences sp;
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            //文本编辑控件
            et_ip = (EditText) findViewById(R.id.et_ip);
            //获取sp 对象
            sp = getSharedPreferences("config", MODE_PRIVATE);
        }
        //保存IP 号码
        public void saveIP(View view){
            String ipNum = et_ip.getText().toString().trim();
            //如果为空则保存默认值
            if (TextUtils.isEmpty(ipNum)) {
                sp.edit().putString("ip", "17951").commit();
            }else {
                sp.edit().putString("ip", ipNum).commit();
            }
            Toast.makeText(this, "IP 号码保存成功", Toast.LENGTH_SHORT).show();
        }

    }
```
编写自定义广播接收者需要自定义一个类然后继承系统提供的BroadCastReceiver 类，然后覆写抽象方法onReceive。

```java
    package com.itheima.android.ipcaller;

    import android.content.BroadcastReceiver;
    import android.content.Context;
    import android.content.Intent;
    import android.content.SharedPreferences;
    import android.text.TextUtils;
    import android.util.Log;

    //自定义广播接收者
    public class IPCallerReceiver extends BroadcastReceiver {

        @Override
        public void onReceive(Context context, Intent intent) {
            //获取数据
            String resultData = getResultData();
            Log.d("tag", "接收到广播："+resultData);
            //从SharedPreferences 中获取用户保存的IP 号码
            SharedPreferences sp =
                    context.getSharedPreferences("config", Context.MODE_PRIVATE);
            String ipNum = sp.getString("ip", "17951");
            if (!TextUtils.isEmpty(ipNum)) {
                //修改数据
                resultData = ipNum+resultData;
            }
            //将修改后的数据设置出去
            setResultData(resultData);
        }

    }
```

**在清单文件中进行注册**

广播是Android 四大组件之一，因此需要在AndroidManifest.xml 中进行注册。同时监听用户的拨打电话行为也属于侵犯用户隐私的行为，因此需要添加权限。

**注册广播**

```java
    <receiver android:name="com.itheima.android.ipcaller.IPCallerReceiver">
        <intent-filter >
            <action android:name="android.intent.action.NEW_OUTGOING_CALL"></action>
        </intent-filter>
    </receiver>
```
大家可以发现广播接收者的注册也需要通过intent-filter 来监听特定的广播，如果是监听Android 系统的，那么在action 中就需要配置系统提供的常量。如果监听自定义发送的广播，那么就需要配置自定义广播设置的action。

**声明权限**

```java
<uses-permission android:name="android.permission.PROCESS_OUTGOING_CALLS"/>
```

## **2.2 案例-短信监听器**
系统接收到短信时会将该事件以有序广播（部分自定义的ROM 可能已经修改了这个策略，比如：小米的MIUI 系统） 的形式发送出去， 因此我们只需要自定义一个BroadCastReceiver 监听该广播（android.provider.Telephony.SMS_RECEIVED）即可监听到短信的到来。由于该广播是有序的，因此如果将我们自定义的BroadCastReceiver 配置了较高的优先级，那么我们就能先于系统短信app 接收到该广播，
然后终止该广播，从而就实现了短信拦截功能。

通过该案例我们可以学到：

- 什么是有序广播？
- 如何终止有序广播
- 如何从广播中获取短信
- 广播的优先级概念

在该案例中我们要做一个类似短信黑名单的应用，主界面提供一个EditText 和一个Button，让用户输入一个“黑名单”，点击保存之后，如果该号码发短信过来，那么我们的应用就将其拦截

![BroadcastReceiver](http://img.blog.csdn.net/20161021123011029)

## **2.3 案例-监听应用的安装和卸载**
在Android 系统中，安装应用和卸载应用事件也都会发送特定的广播，我们可以通过监听这些广播间接获取到用户新安装了什么软件，卸载了哪些软件，进而可以统计用户的偏好，或统计某个软件的存留率。

需求很简单，监听应用的安装和卸载，并将其报名打印出来即可。
该应用不需要界面，代码也很简单，只需要一个自定义广播接收这就可以了。
# **3. 发送无序广播**
# **4. 发送有序广播**
# **5. 特殊的广播接收者-锁屏与解屏**
# 拦截短信的广播

```java
private class InnerSmsReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        System.out.println("InnerSmsReceiver");
        // 获取到短信
        Object[] objects = (Object[]) intent.getExtras().get("pdus");
        for (Object obj : objects) {
            SmsMessage smsMessage = SmsMessage.createFromPdu((byte[]) obj);
            // 获取到短信内容
            String body = smsMessage.getMessageBody();
            // 获取到电话号码
            String phone = smsMessage.getDisplayOriginatingAddress();
            // 根据电话号码查询拦截的模式
            String mode = dao.findNumberMode(phone);
            /**
             * 黑名单的拦截模式1 全部拦截(电话拦截+ 短信拦截) 2 电话拦截3 短信拦截
             */
            if ("1".equals(mode) || "3".equals(mode)) {
                System.out.println("被哥拦截了");
                //往短信拦截数据库里面添加数据
                abortBroadcast();
            }
            /**
             * 根据内容拦截(智能拦截)
             */
            if (body.contains("xue sheng mei")) {
                System.out.println("学生妹被拦截了");
                abortBroadcast();
            }
        }
    }
}
```
# 注册静态广播

```java
receiver = new InnerSmsReceiver();
IntentFilter filter = new IntentFilter("android.provider.Telephony.SMS_RECEIVED");
// 设置优先级
filter.setPriority(2147483647);
// 注册一个短信监听的广播
registerReceiver(receiver, filter);
```
# 反注册广播，防止内存泄露

```java
public void onDestroy() {
    super.onDestroy();
    // 反注册
    unregisterReceiver(receiver);
    receiver = null;
    // 当不用了。设置为null
    mTelephonyManager.listen(listener, PhoneStateListener.LISTEN_NONE);
    listener = null;
}
```
# 注册广播并设置优先级

```xml
<!-- 拦截黑名单信息-->
<receiver
    android:name="com.itheima.mobilesafe_sh2.receiver.InnerSmsReceiver " >
    <intent-filter android:priority="1000" >
        <action android:name="android.provider.Telephony.SMS_RECEIVED"/>
    </intent-filter>
</receiver>
```
# 案例1：IP拨号器

```java
public class CallReceiver extends BroadcastReceiver {

    //接收到广播时就会调用
    @Override
    public void onReceive(Context context, Intent intent) {
        //添加IP线路
        //在打电话广播中，会携带拨打的电话的号码，通过以下代码获取到
        String number = getResultData();

        if(number.startsWith("0")){
            SharedPreferences sp = context.getSharedPreferences("ip", Context.MODE_PRIVATE);
            String ipNumber = sp.getString("ipNumber", "");

            //把IP线路号码添加至用户拨打号码的前面
            number = ipNumber + number;

            //把新的号码重新放入广播中
            setResultData(number);

            abortBroadcast();
        }        
    }
}
```

```java
public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    public void click(View v){
        EditText et = (EditText) findViewById(R.id.et);
        SharedPreferences sp = getSharedPreferences("ip", MODE_PRIVATE);
        sp.edit().putString("ipNumber", et.getText().toString()).commit();
    }
}
```
# 案例2：短信防火墙

```java
public class SmsReceiver extends BroadcastReceiver {

    @Override
    public void onReceive(Context context, Intent intent) {
        //拿到短信的信息
        //短信内容封装在intent中
        Bundle bundle = intent.getExtras();
        //以pdus为键，取出一个object数组，数组中的每一个元素，都是一条短信
        Object[] objects = (Object[]) bundle.get("pdus");

        //拿到广播中的所有短信
        for (Object object : objects) {
            //通过pdu来构造短信
            SmsMessage sms = SmsMessage.createFromPdu((byte[])object);
            if(sms.getOriginatingAddress().equals("138438")){
                //阻止其他广播接收者收到这条广播
                abortBroadcast();
//              SmsManager.getDefault().sendTextMessage(sms.getOriginatingAddress(), null, "你是个好人", null, null);
            }
//          System.out.println(sms.getMessageBody());            
        }
    }
}

```
# 案例3：监听SD卡状态

```java
public class SDStatusReceiver extends BroadcastReceiver {

    @Override
    public void onReceive(Context context, Intent intent) {
        //判断收到的到底是什么广播
        String action = intent.getAction();
        if("android.intent.action.MEDIA_MOUNTED".equals(action)){
            Toast.makeText(context, "SD卡可用", 0).show();
        }
        else if("android.intent.action.MEDIA_REMOVED".equals(action)){
            Toast.makeText(context, "SD卡拔出", 0).show();
        }
        else if("android.intent.action.MEDIA_UNMOUNTED".equals(action)){
            Toast.makeText(context, "SD卡不可用", 0).show();
        }
    }
}
```
# 案例4：手机勒索软件

```java
public class BootReceiver extends BroadcastReceiver {

    @Override
    public void onReceive(Context context, Intent intent) {
        // 启动Activity，实现开机自动启动勒索软件
        Intent it = new Intent(context, MainActivity.class);
        //创建任务栈存放启动的Activity
        it.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        context.startActivity(it);
    }
}
```
# 案例5：监控应用的状态

```java
public class APPStatusReceiver extends BroadcastReceiver {

    @Override
    public void onReceive(Context context, Intent intent) {
        // TODO Auto-generated method stub
        String action = intent.getAction();
        Uri uri = intent.getData();
        if("android.intent.action.PACKAGE_ADDED".equals(action)){
            Toast.makeText(context, uri.toString() + "被安装了", 0).show();
        }
        if("android.intent.action.PACKAGE_REPLACED".equals(action)){
            Toast.makeText(context, uri.toString() + "被升级了", 0).show();
        }
        if("android.intent.action.PACKAGE_REMOVED".equals(action)){
            Toast.makeText(context, uri.toString() + "被卸载了", 0).show();
        }
    }
}
```
# 案例6：发送自定义广播

```java
public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    public void click(View v){
        //发送自定义广播
        Intent intent = new Intent();
        //广播中的action也是自定义的
        intent.setAction("com.itheima.zdy");
        sendBroadcast(intent);
    }
}
```
