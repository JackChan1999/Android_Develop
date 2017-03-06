# 广播
* 广播的概念
    * 现实：电台通过发送广播发布消息，买个收音机，就能收听
    * Android：系统在产生某个事件时发送广播，应用程序使用广播接收者接收这个广播，就知道系统产生了什么事件。
      Android系统在运行的过程中，会产生很多事件，比如开机、电量改变、收发短信、拨打电话、屏幕解锁
# IP拨号器
原理：接收拨打电话的广播，修改广播内携带的电话号码

定义广播接收者接收打电话广播

```java
public class CallReceiver extends BroadcastReceiver {

    //当广播接收者接收到广播时，此方法会调用
    @Override
    public void onReceive(Context context, Intent intent) {
        //拿到用户拨打的号码
        String number = getResultData();
        //修改广播内的号码
        setResultData("17951" + number);
    }
}
```

* 在清单文件中定义该广播接收者接收的广播类型

```xml
<receiver android:name="com.itheima.ipdialer.CallReceiver">
     <intent-filter >
         <action android:name="android.intent.action.NEW_OUTGOING_CALL"/>
     </intent-filter>
 </receiver>
```

* 接收打电话广播需要权限

```xml
<uses-permission android:name="android.permission.PROCESS_OUTGOING_CALLS"/>
```

* 即使广播接收者的进程没有启动，当系统发送的广播可以被该接收者接收时，系统会自动启动该接收者所在的进程

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
# 短信拦截器

系统收到短信时会产生一条广播，广播中包含了短信的号码和内容

* 定义广播接收者接收短信广播

```java
    public void onReceive(Context context, Intent intent) {
    //拿到广播里携带的短信内容
    Bundle bundle = intent.getExtras();
    Object[] objects = (Object[]) bundle.get("pdus");
    for(Object ob : objects ){
        //通过object对象创建一个短信对象
        SmsMessage sms = SmsMessage.createFromPdu((byte[])ob);
        System.out.println(sms.getMessageBody());
        System.out.println(sms.getOriginatingAddress());
    }
}
```

* 系统创建广播时，把短信存放到一个数组，然后把数据以pdus为key存入bundle，再把bundle存入intent
* 清单文件中配置广播接收者接收的广播类型，注意要设置优先级属性，要保证优先级高于短信应用，才可以实现拦截

```xml
<receiver android:name="com.itheima.smslistener.SmsReceiver">
    <intent-filter android:priority="1000">
        <action android:name="android.provider.Telephony.SMS_RECEIVED"/>
    </intent-filter>
</receiver>
```

* 添加权限

&lt;uses-permission android:name="android.permission.RECEIVE_SMS"/>

* 4.0以后广播接收者安装以后必须手动启动一次，否则不生效
* 4.0以后广播接收者如果被手动关闭，就不会再启动了

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
# 监听SD卡状态
* 清单文件中定义广播接收者接收的类型，监听SD卡常见的三种状态，所以广播接收者需要接收三种广播

```xml
 <receiver android:name="com.itheima.sdcradlistener.SDCardReceiver">
    <intent-filter >
        <action android:name="android.intent.action.MEDIA_MOUNTED"/>
        <action android:name="android.intent.action.MEDIA_UNMOUNTED"/>
        <action android:name="android.intent.action.MEDIA_REMOVED"/>
        <data android:scheme="file"/>
    </intent-filter>
</receiver>
```

* 广播接收者的定义

```java
public class SDCardReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        // 区分接收到的是哪个广播
        String action = intent.getAction();
            
        if(action.equals("android.intent.action.MEDIA_MOUNTED")){
            System.out.println("sd卡就绪");
        }
        else if(action.equals("android.intent.action.MEDIA_UNMOUNTED")){
            System.out.println("sd卡被移除");
        }
        else if(action.equals("android.intent.action.MEDIA_REMOVED")){
            System.out.println("sd卡被拔出");
        }
    }
}
```
# 勒索软件
* 接收开机广播，在广播接收者中启动勒索的Activity
* 清单文件中配置接收开机广播

```xml
<receiver android:name="com.itheima.lesuo.BootReceiver">
    <intent-filter >
        <action android:name="android.intent.action.BOOT_COMPLETED"/>
    </intent-filter>
</receiver>
```

* 权限

```java
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
```

* 定义广播接收者

```java
@Override
public void onReceive(Context context, Intent intent) {
    //开机的时候就启动勒索软件
    Intent it = new Intent(context, MainActivity.class);        
    context.startActivity(it);
}
```

* 以上代码还不能启动MainActivity，因为广播接收者的启动，并不会创建任务栈，那么没有任务栈，就无法启动activity
* 手动设置创建新任务栈的flag

```java
it.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
```
# 监听应用的安装、卸载、更新
原理：应用在安装卸载更新时，系统会发送广播，广播里会携带应用的包名

清单文件定义广播接收者接收的类型，因为要监听应用的三个动作，所以需要接收三种广播

```xml
<receiver android:name="com.itheima.app.AppReceiver">
    <intent-filter >
        <action android:name="android.intent.action.PACKAGE_ADDED"/>
        <action android:name="android.intent.action.PACKAGE_REPLACED"/>
        <action android:name="android.intent.action.PACKAGE_REMOVED"/>
        <data android:scheme="package"/>
    </intent-filter>
</receiver>
```

* 广播接收者的定义

```java
public void onReceive(Context context, Intent intent) {
    //区分接收到的是哪种广播
    String action = intent.getAction();
    //获取广播中包含的应用包名
    Uri uri = intent.getData();
    if(action.equals("android.intent.action.PACKAGE_ADDED")){
        System.out.println(uri + "被安装了");
    }
    else if(action.equals("android.intent.action.PACKAGE_REPLACED")){
        System.out.println(uri + "被更新了");
    }
    else if(action.equals("android.intent.action.PACKAGE_REMOVED")){
        System.out.println(uri + "被卸载了");
    }
}
```
# 广播的两种类型
* 无序广播：所有跟广播的intent匹配的广播接收者都可以收到该广播，并且是没有先后顺序（同时收到）
* 有序广播：所有跟广播的intent匹配的广播接收者都可以收到该广播，但是会按照广播接收者的优先级来决定接收的先后顺序
    * 优先级的定义：-1000~1000
    * 最终接收者：所有广播接收者都接收到广播之后，它才接收，并且一定会接收
    * abortBroadCast：阻止其他接收者接收这条广播，类似拦截，只有有序广播可以被拦截
# Service
* 就是默默运行在后台的组件，可以理解为是没有前台的activity，适合用来运行不需要前台界面的代码
* 服务可以被手动关闭，不会重启，但是如果被自动关闭，内存充足就会重启
* startService启动服务的生命周期
    * onCreate-onStartCommand-onDestroy
* 重复的调用startService会导致onStartCommand被重复调用

# 进程优先级
1. 前台进程：拥有前台activity（onResume方法被调用）
2. 可见进程：拥有可见activity（onPause方法被调用）
3. 服务进程：不到万不得已不会被回收，而且即便被回收，内存充足时也会被重启
4. 后台进程：拥有后台activity（activity的onStop方法被调用了），很容易被回收
5. 空进程：没有运行任何activity，很容易被回收

# 电话窃听器
* 电话状态：空闲、响铃、接听
* 获取电话管理器，设置侦听

```java
 TelephonyManager tm = (TelephonyManager)getSystemService(TELEPHONY_SERVICE);
tm.listen(new MyPhoneStateListener(),PhoneStateListener.LISTEN_CALL_STATE);
```

* 侦听对象的实现

```java
 class MyPhoneStateListener extends PhoneStateListener{

     //当电话状态改变时，此方法调用
     @Override
     public void onCallStateChanged(int state, String incomingNumber) {
         // TODO Auto-generated method stub
         super.onCallStateChanged(state, incomingNumber);
         switch (state) {
         case TelephonyManager.CALL_STATE_IDLE://空闲
             if(recorder != null){
                 recorder.stop();
                 recorder.release();
             }
             break;
         case TelephonyManager.CALL_STATE_OFFHOOK://摘机
             if(recorder != null){
                 recorder.start();
             }
             break;
         case TelephonyManager.CALL_STATE_RINGING://响铃
             recorder = new MediaRecorder();
             //设置声音来源
             recorder.setAudioSource(MediaRecorder.AudioSource.MIC);
             //设置音频文件格式
             recorder.setOutputFormat(MediaRecorder.OutputFormat.THREE_GPP);
             recorder.setOutputFile("sdcard/haha.3gp");
             //设置音频文件编码
             recorder.setAudioEncoder(MediaRecorder.AudioEncoder.AMR_NB);
             try {
                 recorder.prepare();
             } catch (IllegalStateException e) {
                 // TODO Auto-generated catch block
                 e.printStackTrace();
             } catch (IOException e) {
                 // TODO Auto-generated catch block
                 e.printStackTrace();
             }
             break;
         }
     }
 }
```

# 广播接收者
* 现实中：电台要发布消息，通过广播把消息广播出去，使用收音机，就可以收听广播，得知这条消息
* Android中：系统在运行过程中，会产生会多事件，那么某些事件产生时，比如：电量改变、收发短信、拨打电话、屏幕解锁、开机，系统会发送广播，只要应用程序接收到这条广播，就知道系统发生了相应的事件，从而执行相应的代码。使用广播接收者，就可以收听广播

## 创建广播接收者
1、 定义java类继承BroadcastReceiver
2、 在清单文件中定义receiver节点，定义name属性，指定广播接收者java类的全类名
3、 在intent-filter的节点中，指定action子节点，action的值必须跟要接受的广播中的action匹配，比如，如果要接受打电话广播，
那么action的值必须指定为

```xml
<action android:name="android.intent.action.NEW_OUTGOING_CALL"/>
```

* 因为打电话广播中所包含的action，就是"android.intent.action.NEW_OUTGOING_CALL"，所以我们定义广播接收者时，
  action必须与其匹配，才能收到这条广播
* 即便广播接收者所在进程已经被关闭，当系统发出的广播中的action跟该广播接收者的action匹配时，系统会启动该广播接收者所在的进程，
  并把广播发给该广播接收者

### 短信防火墙
* 系统发送短信广播时，是怎么把短信内容存入广播的，我们就只能怎么取出来
* 如果短信过长，那么发送时会拆分成多条短信发送，那么短信广播中就会包含多条短信
* 4.0之后，广播接收者所在进程如果从来没启动过，那么广播接收者不会生效
* 4.0之后，如果系统自动关闭广播接收者所在进程，在广播中的action跟该广播接收者的action匹配时，系统会启动该广播接收者所在的进程，但是如果是用户手动关闭该进程，
  那么该进程会进入冻结状态，再也不会启动了，直到用户下一次手动启动该进程

### 广播的分类
#### 无序广播
* 所有与广播中的action匹配的广播接收者都可以收到这条广播，并且是没有先后顺序，视为同时收到
#### 有序广播
* 所有与广播中的action匹配的广播接收者都可以收到这条广播，但是是有先后顺序的，按照广播接收者的优先级排序
# 服务
* Service
* 运行于后台的一个组件，用来运行适合运行在后台的代码，服务是没有前台界面，可以视为没有界面的activity
## 进程优先级
1. 前台进程：拥有一个正在与用户交互的Activity（onResume方法被调用）的进程
2. 可见进程：拥有一个可见但是没有焦点的Activity（onPause方法被调用）
3. 服务进程：拥有一个通过startService方法启动的服务
4. 后台进程：拥有一个不可见的Activity（onStop方法被调用）的进程
5. 空进程：没有拥有任何活动的应用组件的进程

## 电话录音机
### 电话的状态
* 空闲状态
* 响铃状态
* 摘机状态

### 录音机
* 音频文件的编码和格式不是一一对应的

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
# 案例7：电话录音机

```java
public class RecorderService extends Service {
    private MediaRecorder recorder;
    @Override
    public IBinder onBind(Intent intent) {
        // TODO Auto-generated method stub
        return null;
    }

    @Override
    public void onCreate() {
        // TODO Auto-generated method stub
        super.onCreate();
        //拿到电话管理器
        TelephonyManager tm = (TelephonyManager) getSystemService(TELEPHONY_SERVICE);
        //监听电话状态
        //events:决定PhoneStateListener侦听什么内容
        tm.listen(new MyListener(), PhoneStateListener.LISTEN_CALL_STATE);
    }
    
    class MyListener extends PhoneStateListener{

    

        //一旦电话状态改变，此方法调用
        @Override
        public void onCallStateChanged(int state, String incomingNumber) {
            // TODO Auto-generated method stub
            super.onCallStateChanged(state, incomingNumber);
            
            switch (state) {
            case TelephonyManager.CALL_STATE_IDLE:
                System.out.println("空闲");
                if(recorder != null){
                    recorder.stop();
                    recorder.release();
                    recorder = null;
                }
                break;
            case TelephonyManager.CALL_STATE_RINGING:
                System.out.println("响铃");
                if(recorder == null){
                    recorder = new MediaRecorder();
                    recorder.setAudioSource(MediaRecorder.AudioSource.MIC);
                    recorder.setOutputFormat(MediaRecorder.OutputFormat.THREE_GPP);
                    recorder.setOutputFile("sdcard/luyin.3gp");
                    recorder.setAudioEncoder(MediaRecorder.AudioEncoder.AMR_NB);
                    try {
                        recorder.prepare();
                    } catch (Exception e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                    }
                }
                break;
            case TelephonyManager.CALL_STATE_OFFHOOK:
                System.out.println("摘机");
                //开始录音
                if(recorder != null){
                    recorder.start();
                }
                break;

            }
        }
        
    }
    
}

```
```java
public class BootReceiver extends BroadcastReceiver {

    @Override
    public void onReceive(Context context, Intent intent) {
        //启动录音机服务
        Intent it = new Intent(context, RecorderService.class);
        context.startService(it);
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
        Intent intent = new Intent(this, RecorderService.class);
        startService(intent);
    }
}
```

## 拦截短信的广播

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
## 注册静态广播

```java
receiver = new InnerSmsReceiver();
IntentFilter filter = new IntentFilter("android.provider.Telephony.SMS_RECEIVED");
// 设置优先级
filter.setPriority(2147483647);
// 注册一个短信监听的广播
registerReceiver(receiver, filter);
```
## 反注册广播，防止内存泄露

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
## 注册广播并设置优先级

```xml
<!-- 拦截黑名单信息-->
<receiver
    android:name="com.itheima.mobilesafe_sh2.receiver.InnerSmsReceiver " >
    <intent-filter android:priority="1000" >
        <action android:name="android.provider.Telephony.SMS_RECEIVED"/>
    </intent-filter>
</receiver>
```




