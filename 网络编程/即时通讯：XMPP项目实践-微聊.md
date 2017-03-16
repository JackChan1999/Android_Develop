# 即时通讯系列阅读

1. [即时通讯基础](http://blog.csdn.net/axi295309066/article/details/62427091)

2. [即时通讯：XMPP基础](http://blog.csdn.net/axi295309066/article/details/62427206)

3. [即时通讯：XMPP项目实践-微聊](http://blog.csdn.net/axi295309066/article/details/62427366)

# 1. 项目简介

做一个类似QQ 的通讯工具，要求有注册、登录、添加好友、添加分组、聊天、退出登录等功能。我用8 张运行效果图来展示我们将要实现的功能。

![](http://upload-images.jianshu.io/upload_images/3981391-725c1cc8d49c8230.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) ![](http://upload-images.jianshu.io/upload_images/3981391-8c42c0e0714b520f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

注意，服务器用的是Openfire，我们可以用Spark 作为另外一个客户端进行测试。
闪屏页进来以后是登录界面，要求记住上次登陆的账号和密码，在界面的右下角有新用户按钮，点击后进入注册界面。注册只需要输入账号和密码即可。账号不能和其他人重复，否则注册不成功。

![](http://upload-images.jianshu.io/upload_images/3981391-697cc0745674e053.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) ![](http://upload-images.jianshu.io/upload_images/3981391-b387a80d2b178e36.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

主界面是一个Activity 绑定3 个Fragment 实现，分别是消息、联系人、动态。

其中消息界面是一个ListView 展出了最近的联系人。点击其中的一个条目可以进入聊天界面。

联系人界面主要是一个ExpandableListView。ExpandableListView 列出了用户组，以及各个组中的好友。点击任意好友可以进入聊天界面。在ExpandableListView 上面有两个图标，分别是新朋友、新群组。点击新朋友弹出一个自定义对话框，在该对话框中输入对方好友的名称，等待对方同意了即可添加为好友。

点击新群组也弹出一个自定义对话框，在该对话框中输入分组名称，则可以创建一个分组。

如果好友不存则添加好友失败，如果分组不存在则创建分组失败。

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3981391-497dd217c7ba0be0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) ![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3981391-119ac722da7f1652.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3981391-8929ccde032961d5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) ![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3981391-1b05000a6b9afd21.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
动态界面主要展示了当前用户的信息，最下面有个退出按钮，点击后退出当前登录，并跳转到登录界面。

聊天界面是一个ListView，该ListView 的条目有两类布局，分别用来表示好友的消息和自己的消息。在最下方的输入框输入文本内容，然后点击发送可以将该消息发送给好友。好友有消息过来也可以直接显示在该界面。

# 2. 项目搭建

1、[下载asmak.jar](http://asmack.freakempire.de/)，是个德国网站

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3981391-a4aa7e1050118fdd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

asmack 的版本号从0.x 到现在的4.x 变化比较大。不通过版本差异也比较大。本次我写项目用的是0.8.x 的。

用的是13 年的版本。因为该api 在网上能查到的资料比较多。如果是下一次我再写这个项目我就决定用15 年发布的最新版本的了。

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3981391-da0d4ac5d05df80a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我想看看asmack 公司官网，吧asmack 去掉，想着就是贵公司的网址呢，却得到这样的界面。

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3981391-3f09363e9bbd38de.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2、给工程添加jar
简单极了，只需把下载好的jar 包添加到Android 工程的libs 目录下即可。注意如果eclipse 没有自动将该包添加到环境变量中，我们需要手动添加一下。

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3981391-c200c889bde76496.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3、项目目录结构图

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3981391-b9cfeba7d05297f2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) ![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3981391-ca71a808f7d71f6b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 3. 项目实现

## 3.1 Spark 客户端的下载和安装闪屏界面

新颖的闪屏界面，马老师，学习的目的，我借来用一用哈希望您不用太在意。

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3981391-5b64e495eef32c19.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我要做的效果是闪屏等待3 秒，然后进入主界面。但是每次都让用户等待3 秒，对于急性子来讲估计会很抓
狂。那怎么办呢，只要是这个界面用户触摸屏幕则立即进入主界面。布局太简单了，不给了。直接给代码。

```java
/**
   * 闪屏页面默认等待3s 触摸时直接进入主界面
   *
   * @author wzy 2015-8-14
   *
   */
  public class SplashActivity extends Activity {

      private static final long DURATION = 3000;
      /**
       * 保证变量的修改是可见的，但是无法保证变量的原子性
       */
      private volatile boolean isEntered = false;
      private Thread splashThread = new Thread(new Runnable() {

          @Override
          public void run() {
              SystemClock.sleep(DURATION);
              enter();
          }
      });

      @Override


      protected void onCreate(Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
          setContentView(R.layout.activity_splash);
          splashThread.start();
      }

      private synchronized void enter() {
          if (!isEntered) {
              isEntered = true;
              startActivity(new Intent(SplashActivity.this, LoginActivity.class));
              finish();
          }
      }

      @Override
      public boolean onTouchEvent(MotionEvent event) {
          enter();
          return true;
      }
  }
```

## 3.2 登录

登录界面布局很简单。如果写不出来就没必要往下看了。登录的核心代码：
mXmppConnection.login(name, pwd);

activity_login.xml
```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
                xmlns:tools="http://schemas.android.com/tools"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:background="@drawable/login_background" >
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@drawable/login_background"
        android:orientation="vertical" >
        <ImageView
            android:id="@+id/iv_touxiang"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center_horizontal"


            android:layout_marginTop="40dp"
            android:contentDescription="@null"
            android:src="@drawable/login_image" />
        <EditText
            android:id="@+id/et_name"
            android:layout_width="match_parent"
            android:layout_height="40dp"
            android:layout_marginLeft="5dp"
            android:layout_marginRight="5dp"
            android:layout_marginTop="30dp"
            android:background="#FFFFFF"
            android:hint="请输入账号"
            android:paddingLeft="5dp"
            android:textSize="20sp" />
        <View
            android:layout_width="match_parent"
            android:layout_height="1dp"
            android:background="#55AABBCC" />
        <EditText
            android:id="@+id/et_pwd"
            android:layout_width="match_parent"
            android:layout_height="40dp"
            android:layout_marginLeft="5dp"
            android:layout_marginRight="5dp"
            android:background="#FFFFFF"
            android:hint="请输入密码"
            android:inputType="textPassword"
            android:paddingLeft="5dp"
            android:textSize="20sp" />
        <Button
            android:id="@+id/btn_login"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_margin="15dp"
            android:background="@color/title_layout"
            android:onClick="login"
            android:text="登录"
            android:textColor="#FFFFFF"
            android:textSize="20sp" />
    </LinearLayout>
    <Button
        android:id="@+id/register"
        android:layout_width="100dp"
        android:layout_height="40dp"
        android:layout_alignParentBottom="true"


        android:layout_alignParentRight="true"
        android:layout_marginRight="20dp"
        android:background="@drawable/register_user_btn"
        android:text="新用户"
        android:onClick="gotoRegist"
        android:textColor="@color/blue"
        android:textSize="16sp" />
</RelativeLayout>
```
LoginActivity 和RegistActivity 都继承自BaseActivity。因此这里先把BaseActivity 的代码列出。

BaseActivity.java
```java
public class XmppConnectionManager {
     private static XmppConnectionManager instance = null;
     //私有化构造函数
     private XmppConnectionManager() {
     }
     /**
      * 获取该对象
      * @return
      */
     public static XmppConnectionManager getInstance() {
         if (instance == null) {
             instance = new XmppConnectionManager();
         }
         return instance;
     }
     /**
      * 执行初始化脚本
      * @return
      */
     public XMPPConnection init() {
         /**
          * 创建连接配置对象<br>
          * 第一个参数是Openfire 服务器地址<br>
          * 第二个参数是Openfire 服务器断开号，默认是5222<br>
          * 我们可以把这两个参数配置的清单文件中，也可以写死在代码中
          */
         ConnectionConfiguration connectionConfig = new
                 ConnectionConfiguration(Const.XMPP_HOST,Const.XMPP_PORT);
         /**


          * 不使用SAL 安全验证
          */
         connectionConfig.setSASLAuthenticationEnabled(false);
         /**
          * 设置TLS 安全模式
          */
         connectionConfig.setSecurityMode(ConnectionConfiguration.SecurityMode.enabled);
         // 允许自动连接
         connectionConfig.setReconnectionAllowed(true);
         // 允许登陆成功后更新在线状态
         connectionConfig.setSendPresence(true);
         //设置为debug 模式，该模式可以在控制台看到接收和发送的xmpp 协议
         connectionConfig.setDebuggerEnabled(true);
         // 收到好友邀请后同意添加为好友的模式，有三种，manual 表示需要经过同意,accept_all 表示不经同意自动
         为好友，reject_all 拒绝加为好友邀请
         Roster.setDefaultSubscriptionMode(Roster.SubscriptionMode.accept_all);
         /**
          * 该配置时为了解决asmack 的一个bug 或者说弥补一个不足之处。不用细细追究。
          */
         configure(ProviderManager.getInstance());
         //创建一个连接对象，参数为配置对象
         XMPPConnection connection = new XMPPConnection(connectionConfig);
         return connection;
     }

     public void configure(ProviderManager pm) {

         // Private Data Storage
         pm.addIQProvider("query", "jabber:iq:private", new
                 PrivateDataManager.PrivateDataIQProvider());

         // Time
         try {
             pm.addIQProvider("query", "jabber:iq:time",
                     Class.forName("org.jivesoftware.smackx.packet.Time"));
         } catch (ClassNotFoundException e) {
             Log.w("TestClient", "Can't load class for org.jivesoftware.smackx.packet.Time");
         }

         // Roster Exchange
         pm.addExtensionProvider("x", "jabber:x:roster", new RosterExchangeProvider());
         // Message Events
         pm.addExtensionProvider("x", "jabber:x:event", new MessageEventProvider());
         // Chat State


         pm.addExtensionProvider("active", "http://jabber.org/protocol/chatstates", new
                 ChatStateExtension.Provider());
         pm.addExtensionProvider("composing", "http://jabber.org/protocol/chatstates", new
                 ChatStateExtension.Provider());
         pm.addExtensionProvider("paused", "http://jabber.org/protocol/chatstates", new
                 ChatStateExtension.Provider());
         pm.addExtensionProvider("inactive", "http://jabber.org/protocol/chatstates", new
                 ChatStateExtension.Provider());
         pm.addExtensionProvider("gone", "http://jabber.org/protocol/chatstates", new
                 ChatStateExtension.Provider());
         // XHTML
         pm.addExtensionProvider("html", "http://jabber.org/protocol/xhtml-im", new
                 XHTMLExtensionProvider());
         // Group Chat Invitations
         pm.addExtensionProvider("x", "jabber:x:conference", new
                 GroupChatInvitation.Provider());
         // Service Discovery # Items
         pm.addIQProvider("query", "http://jabber.org/protocol/disco#items", new
                 DiscoverItemsProvider());
         // Service Discovery # Info
         pm.addIQProvider("query", "http://jabber.org/protocol/disco#info", new
                 DiscoverInfoProvider());
         // Data Forms
         pm.addExtensionProvider("x", "jabber:x:data", new DataFormProvider());
         // MUC User
         pm.addExtensionProvider("x", "http://jabber.org/protocol/muc#user", new
                 MUCUserProvider());
         // MUC Admin
         pm.addIQProvider("query", "http://jabber.org/protocol/muc#admin", new
                 MUCAdminProvider());
         // MUC Owner
         pm.addIQProvider("query", "http://jabber.org/protocol/muc#owner", new
                 MUCOwnerProvider());
         // Delayed Delivery
         pm.addExtensionProvider("x", "jabber:x:delay", new DelayInformationProvider());
         // Version
         try {
             pm.addIQProvider("query", "jabber:iq:version",
                     Class.forName("org.jivesoftware.smackx.packet.Version"));
         } catch (ClassNotFoundException e) {
             // Not sure what's happening here.
         }
         // VCard
         pm.addIQProvider("vCard", "vcard-temp", new VCardProvider());
         // Offline Message Requests


         pm.addIQProvider("offline", "http://jabber.org/protocol/offline", new
                 OfflineMessageRequest.Provider());
         // Offline Message Indicator
         pm.addExtensionProvider("offline", "http://jabber.org/protocol/offline", new
                 OfflineMessageInfo.Provider());
         // Last Activity
         pm.addIQProvider("query", "jabber:iq:last", new LastActivity.Provider());
         // User Search
         pm.addIQProvider("query", "jabber:iq:search", new UserSearch.Provider());
         // SharedGroupsInfo
         pm.addIQProvider("sharedgroup",
                 "http://www.jivesoftware.org/protocol/sharedgroup", new SharedGroupsInfo.Provider());
         // JEP-33: Extended Stanza Addressing
         pm.addExtensionProvider("addresses", "http://jabber.org/protocol/address", new
                 MultipleAddressesProvider());
         // FileTransfer
         pm.addIQProvider("si", "http://jabber.org/protocol/si", new
                 StreamInitiationProvider());
         pm.addIQProvider("query", "http://jabber.org/protocol/bytestreams", new
                 BytestreamsProvider());
         // Privacy
         pm.addIQProvider("query", "jabber:iq:privacy", new PrivacyProvider());
         pm.addIQProvider("command", "http://jabber.org/protocol/commands", new
                 AdHocCommandDataProvider());
         pm.addExtensionProvider("malformed-action",
                 "http://jabber.org/protocol/commands", new
                         AdHocCommandDataProvider.MalformedActionError());
         pm.addExtensionProvider("bad-locale", "http://jabber.org/protocol/commands", new
                 AdHocCommandDataProvider.BadLocaleError());
         pm.addExtensionProvider("bad-payload", "http://jabber.org/protocol/commands", new
                 AdHocCommandDataProvider.BadPayloadError());
         pm.addExtensionProvider("bad-sessionid", "http://jabber.org/protocol/commands",
                 new AdHocCommandDataProvider.BadSessionIDError());
         pm.addExtensionProvider("session-expired","http://jabber.org/protocol/commands",
                 new AdHocCommandDataProvider.SessionExpiredError());
     }
 }
```
LoginActivity.java

```java
import org.jivesoftware.smack.XMPPException;
import com.itheima.qq.MainActivity;
import com.itheima.qq.QQApplication;
import com.itheima.qq.R;


import android.content.Intent;
import android.content.SharedPreferences;
import android.content.SharedPreferences.Editor;
import android.os.Bundle;
import android.os.Handler;
import android.os.Looper;
import android.os.Message;
import android.text.TextUtils;
import android.view.View;
import android.widget.EditText;
import android.widget.Toast;

import static android.app.Activity.RESULT_CANCELED;
import static android.app.Activity.RESULT_OK;
import static android.content.Context.MODE_PRIVATE;

public class LoginActivity extends BaseActivity {
    private EditText et_name;
    private EditText et_pwd;
    private String name;
    private String pwd;
    private SharedPreferences sp;
    private Handler handler = new Handler() {
        public void handleMessage(android.os.Message msg) {
            switch (msg.what) {
                case RESULT_OK:
                    Toast.makeText(LoginActivity.this, msg.obj + " 登录成功", 0).show();
                    //获取自定义的Application，并将连接对象保存在Application 中
                    QQApplication application = (QQApplication) getApplication();
                    application.setXmppConnection(mXmppConnection);
                    //进入主界面
                    Intent intent = new Intent(LoginActivity.this, MainActivity.class);
                    startActivity(intent);
                    //关闭登陆页面
                    finish();
                    break;
                case RESULT_CANCELED:
                    Toast.makeText(LoginActivity.this, "登录失败。" + msg.obj, 0).show();
                    break;

                default:
                    break;
            }
        };
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_login);
        sp = getSharedPreferences("config", MODE_PRIVATE);
        initView();
    }

    private void initView() {
        et_name = (EditText) findViewById(R.id.et_name);
        et_pwd = (EditText) findViewById(R.id.et_pwd);
        // 如果sp 中记录有历史用户名和密码则填充到界面
        String name = sp.getString("name", "");
        String pwd = sp.getString("pwd", "");
        if (!TextUtils.isEmpty(name)) {
            et_name.setText(name);
        }
        if (!TextUtils.isEmpty(pwd)) {
            et_pwd.setText(pwd);
        }
    }

    /**
     * 登录Button 绑定的按钮事件
     *
     * @param view
     */
    public void login(View view) {
        // 首先获取到用户输入的用户名和密码
        name = et_name.getText().toString();
        pwd = et_pwd.getText().toString();
        // 保存到sp 中
        Editor editor = sp.edit();
        editor.putString("name", name);
        editor.putString("pwd", pwd);
        // 一定记得提交
        editor.commit();
        /**
         * 开启一个子线程进行登录，因为登录肯定要连接网络，网络操作必须在子线程中
         */
        new Thread(new Runnable() {

            @Override
            public void run() {
                try {
                    //连接服务器
                    if (!mXmppConnection.isConnected()) {


                        mXmppConnection.connect();
                    }
                } catch (XMPPException e1) {
                    e1.printStackTrace();
                    Looper.prepare();
                    Toast.makeText(LoginActivity.this, "连接服务器失败。", 0).show();
                    Looper.loop();
                    return;
                }
                try {
                    //登录，登录其实也是授权的过程
                    if (!mXmppConnection.isAuthenticated()) {
                        mXmppConnection.login(name, pwd);//登录的关键代码
                    }
                    //如果授权成功则发送handler 消息
                    if (mXmppConnection.isAuthenticated()) {
                        Message message = Message.obtain();
                        message.what = RESULT_OK;
                        //通过连接获取当前登录的用户名
                        message.obj = mXmppConnection.getUser();
                        handler.sendMessage(message);
                    }
                } catch (XMPPException e) {
                    e.printStackTrace();
                    Message message = Message.obtain();
                    message.what = RESULT_CANCELED;
                    message.obj = e;
                    handler.sendMessage(message);
                }
            }
        }).start();

    }
    /**
     * 绑定的界面中button 事件<br>
     * 跳转到注册界面
     *
     * @param view
     */
    public void gotoRegist(View view) {
        //跳转到注册界面
        Intent registIntent = new Intent(this, RegistActivity.class);
        startActivity(registIntent);
    }
}
```
在上面的代码中我们将登陆成功后获取的connection 对象设置到了Application 中，这里用到了自定义的
Application，并且在清单文件中已经配置。

QQApplication.java

```java
public class QQApplication extends Application {
     private XMPPConnection mXmppConnection;
     @Override
     public void onCreate() {
         super.onCreate();
     }
     public XMPPConnection getXmppConnection(){
         return mXmppConnection;
     }
     public void setXmppConnection(XMPPConnection xmppConnection){
         this.mXmppConnection = xmppConnection;
     }
 }
```
AndroidManifest.xml
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          package="com.itheima.qq"
          android:versionCode="1"
          android:versionName="1.0" >
    <uses-sdk
        android:minSdkVersion="8"
        android:targetSdkVersion="21" />
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <application
        android:name="com.itheima.qq.QQApplication"
        android:allowBackup="true"
        android:icon="@drawable/qq"
        android:label="@string/app_name"
        android:theme="@style/AppTheme" >
        <activity
            android:name="com.itheima.qq.activity.SplashActivity"
            android:label="@string/app_name"
            android:theme="@android:style/Theme.NoTitleBar.Fullscreen" >
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <activity android:name="com.itheima.qq.MainActivity" />
        <activity android:name="com.itheima.qq.activity.LoginActivity" />
        <activity android:name="com.itheima.qq.activity.RegistActivity" />
        <activity android:name="com.itheima.qq.activity.ChatActivity" />
        <service android:name="com.itheima.qq.ChatService" />
    </application>

</manifest>
```
清单文件中高亮显示的两个部分，因为需要连接服务器，因此需要联网权限。

## 3.3 注册

注册功能核心代码：
```java
//创建注册对象，用于封装注册参数
  Registration registration = new Registration();
  //设置注册属于设置属性，因此这里设置类型为set
  registration.setType(Type.SET);
  //设置用户名和密码
  registration.setUsername(name);
  registration.setPassword(pwd);
  /**

   * 注册信息封装好之后其实就可以发送了可以直接调用<br>
   * mXmppConnection.sendPacket(registration);方法<br>
   * 但是上面的方法并没有返回值，注册是否成功我们不清楚<br>
   * 因此我们需要开启一个数据包收集器来手机服务返回的信息
   *
   */
  //定义一个数据包过滤器
  /**
   * AndFilter 是一个组合过滤器，形参是可变参数，可以传递多种PacketFilter 的子类
   <br>
   * 我们需要要过滤的原则是根据注册数据包的id 和数据包类型<br>
   *
   */
  PacketFilter filter = new AndFilter(new
          PacketIDFilter(registration.getPacketID()), new PacketTypeFilter(IQ.class));
  //创建一个数据包收集器，形参为数据包过滤器
  PacketCollector collector = mXmppConnection.createPacketCollector(filter);
  // 这个api 并没有提供最简单的regist 方法。而是用了很负责的api，用户体验不佳。
  mXmppConnection.sendPacket(registration);
  /**
   * 上面的代码已经发送过注册数据包了，接下来我们就可以收集服务器的返回值了<br>
   * 形参为网络超时时间默认5s
   */
  Packet packet = collector.nextResult(SmackConfiguration.getPacketReplyTimeout());
  /**
   * 将数据包强转为IQ，因为我们的过滤器已经限定了是IQ 类型的数据包才收集，因此我们
   可以大胆的强转
   */
  IQ result = (IQ) packet;
  //收集到数据后就可以将收集器关闭了，不然可能把其他符合条件的数据也收集来了
  collector.cancel();
  //如果IQ 为空则代表请求失败这里代码我写的不太好了，应该先判断packet 是否为空，
  不为空再强转，这样才能避免空指针
  if (result == null) {
      Message message = Message.obtain();
      message.what = RESULT_CANCELED;
      handler.sendMessage(message);
      return;
      //如果返回的数据类型为IQ.Type.Result 则代表成功
  } else if (result.getType().equals(IQ.Type.RESULT)) {
      Message message = Message.obtain();
      message.what = RESULT_OK;
      handler.sendMessage(message);
      return;
  } else {
      Message message = Message.obtain();
      //否则代表失败，那么我们就收集失败码
      int errorCode = result.getError().getCode();
      message.what = RESULT_CANCELED;
      //如果失败码是409，那么代表用户已经被注册
      if (409 == errorCode) {
          message.obj = "该用户名已经被注册，请换一个名字吧";
          handler.sendMessage(message);
          return;
      } else {
          message.obj = result.getError();
          handler.sendMessage(message);
          return;
      }
  }
}
```
activity_regist.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
                xmlns:tools="http://schemas.android.com/tools"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:background="@drawable/login_background" >

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@drawable/login_background"
        android:orientation="vertical" >

        <ImageView
            android:id="@+id/iv_touxiang"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center_horizontal"
            android:layout_marginTop="40dp"
            android:contentDescription="@null"
            android:src="@drawable/login_default_avatar" />

        <EditText
            android:id="@+id/et_name"
            android:layout_width="match_parent"
            android:layout_height="40dp"
            android:layout_marginLeft="5dp"
            android:layout_marginRight="5dp"
            android:layout_marginTop="30dp"
            android:background="#FFFFFF"
            android:hint="请输入账号"
            android:paddingLeft="5dp"
            android:textSize="20sp" />

        <View
            android:layout_width="match_parent"
            android:layout_height="1dp"
            android:background="#55AABBCC" />

        <EditText
            android:id="@+id/et_pwd"
            android:layout_width="match_parent"
            android:layout_height="40dp"
            android:layout_marginLeft="5dp"


            android:layout_marginRight="5dp"
            android:background="#FFFFFF"
            android:hint="请输入密码"
            android:inputType="textPassword"
            android:paddingLeft="5dp"
            android:textSize="20sp" />

        <Button
            android:id="@+id/btn_regist"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_margin="15dp"
            android:background="@color/title_layout"
            android:onClick="regist"
            android:text="注册"
            android:textColor="#FFFFFF"
            android:textSize="20sp" />
    </LinearLayout>

</RelativeLayout>
```
RegistActivity.java
```java
import org.jivesoftware.smack.PacketCollector;
import org.jivesoftware.smack.SmackConfiguration;
import org.jivesoftware.smack.filter.AndFilter;
import org.jivesoftware.smack.filter.PacketFilter;
import org.jivesoftware.smack.filter.PacketIDFilter;
import org.jivesoftware.smack.filter.PacketTypeFilter;
import org.jivesoftware.smack.packet.IQ.Type;
import org.jivesoftware.smack.packet.IQ;
import org.jivesoftware.smack.packet.Packet;
import org.jivesoftware.smack.packet.Registration;
import com.itheima.qq.R;
import android.content.SharedPreferences;
import android.content.SharedPreferences.Editor;
import android.os.Bundle;
import android.os.Handler;
import android.os.Looper;
import android.os.Message;
import android.view.View;
import android.widget.EditText;
import android.widget.Toast;

import static android.app.Activity.RESULT_CANCELED;

public class RegistActivity extends BaseActivity {
    private EditText et_name;
    private EditText et_pwd;
    private String name;
    private String pwd;
    private SharedPreferences sp;
    private static final int RESULT_NO_RESPONSE = 1;
    private Handler handler = new Handler() {
        public void handleMessage(android.os.Message msg) {
            switch (msg.what) {
                case RESULT_NO_RESPONSE:
                    Toast.makeText(RegistActivity.this,"服务器未响应，请稍后再试", 0).show();
                    break;
                case RESULT_OK:
                    Toast.makeText(RegistActivity.this, "注册成功", 0).show();
                    finish();
                    break;
                case RESULT_CANCELED:
                    Toast.makeText(RegistActivity.this, "注册失败。" + msg.obj, 0).show();
                    break;

                default:
                    break;
            }
        };
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_regist);
        sp = getSharedPreferences("config", MODE_PRIVATE);
        initView();
    }

    private void initView() {
        //获取到注册名和密码
        et_name = (EditText) findViewById(R.id.et_name);
        et_pwd = (EditText) findViewById(R.id.et_pwd);
    }
    /**
     * 绑定注册按钮事件
     * @param view
     */


    public void regist(View view) {
        //获取用户的数据
        name = et_name.getText().toString();
        pwd = et_pwd.getText().toString();
        //保存到sp 中
        Editor editor = sp.edit();
        editor.putString("name", name);
        editor.putString("pwd", pwd);
        editor.commit();
        //因为注册需要联网，因此放在子线程中
        new Thread(new Runnable() {

            @Override
            public void run() {
                try {
                    //如果没有连接服务器则连接服务器
                    if (!mXmppConnection.isConnected()) {
                        mXmppConnection.connect();
                    }
                } catch (Exception e1) {
                    e1.printStackTrace();
                    Looper.prepare();
                    Toast.makeText(RegistActivity.this, "连接服务器失败。", 0).show();
                    Looper.loop();
                    return;
                }
                try {
                    //创建注册对象，用于封装注册参数
                    Registration registration = new Registration();
                    //设置注册属于设置属性，因此这里设置类型为set
                    registration.setType(Type.SET);
                    //设置用户名和密码
                    registration.setUsername(name);
                    registration.setPassword(pwd);
                    /**
                     * 注册信息封装好之后其实就可以发送了可以直接调用<br>
                     * mXmppConnection.sendPacket(registration);方法<br>
                     * 但是上面的方法并没有返回值，注册是否成功我们不清楚<br>
                     * 因此我们需要开启一个数据包收集器来手机服务返回的信息
                     *
                     */
                    //定义一个数据包过滤器
                    /**
                     * AndFilter 是一个组合过滤器，形参是可变参数，可以传递多种PacketFilter 的子类


                     * 我们需要要过滤的原则是根据注册数据包的id 和数据包类型
                     *
                     */
                    PacketFilter filter = new AndFilter(new
                            PacketIDFilter(registration.getPacketID()),new PacketTypeFilter(IQ.class));
                    //创建一个数据包收集器，形参为数据包过滤器
                    PacketCollector collector = mXmppConnection.createPacketCollector(filter);
                    // 这个api 并没有提供最简单的regist 方法。而是用了很负责的api，用户体验不佳。
                    mXmppConnection.sendPacket(registration);
                    /**
                     * 上面的代码已经发送过注册数据包了，接下来我们就可以收集服务器的返回值了
                     * 形参为网络超时时间默认5s
                     */
                    Packet packet =
                            collector.nextResult(SmackConfiguration.getPacketReplyTimeout());
                    /**
                     * 将数据包强转为IQ，因为我们的过滤器已经限定了是IQ 类型的数据包才收集，因此我们
                     可以大胆的强转
                     */
                    IQ result = (IQ)packet;
                    //收集到数据后就可以将收集器关闭了，不然可能把其他符合条件的数据也收集来了
                    collector.cancel();
                    //如果IQ 为空则代表请求失败这里代码我写的不太好了，应该先判断packet 是否为空，
                    不为空再强转，这样才能避免空指针
                    if (result==null) {
                        Message message = Message.obtain();
                        message.what = RESULT_CANCELED;
                        handler.sendMessage(message);
                        return;
                        //如果返回的数据类型为IQ.Type.Result 则代表成功
                    }else if (result.getType().equals(IQ.Type.RESULT)) {
                        Message message = Message.obtain();
                        message.what = RESULT_OK;
                        handler.sendMessage(message);
                        return;
                    }else {
                        Message message = Message.obtain();
                        //否则代表失败，那么我们就收集失败码
                        int errorCode = result.getError().getCode();
                        message.what = RESULT_CANCELED;
                        //如果失败码是409，那么代表用户已经被注册
                        if (409==errorCode) {


                            message.obj = "该用户名已经被注册，请换一个名字吧";
                            handler.sendMessage(message);
                            return;
                        }else {
                            message.obj = result.getError();
                            handler.sendMessage(message);
                            return;
                        }
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                    Message message = Message.obtain();
                    message.what = RESULT_CANCELED;
                    message.obj = e;
                    handler.sendMessage(message);
                }
            }
        }).start();
    }
}
```
## 3.4 主界面

登录之后就进入主界面了。

activity_main.xml
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              xmlns:tools="http://schemas.android.com/tools"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:orientation="vertical" >
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="60dp"
        android:background="@color/title_layout"
        android:gravity="center"
        android:orientation="horizontal" >
        <TextView
            android:id="@+id/tv_title"
            style="@style/TitleStyle"
            android:padding="5dp"
            android:text="@string/title_message" />


    </LinearLayout>

    <FrameLayout
        android:id="@+id/fl_content"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1" >
    </FrameLayout>

    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="@dimen/activity_main_tab_height"
        android:gravity="center_horizontal"
        android:orientation="horizontal" >

        <View
            android:layout_width="match_parent"
            android:layout_height="1dp"
            android:layout_alignParentTop="true"
            android:background="#55000000" />

        <RadioGroup
            android:layout_alignParentBottom="true"
            android:id="@+id/rg_group"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:gravity="center_horizontal"
            >

            <RadioButton
                android:id="@+id/session_rbn"
                android:layout_width="@dimen/message_icon_width"
                android:layout_height="@dimen/message_icon_height"
                android:layout_marginBottom="@dimen/activity_main_tab_margin_top_and_bottom"
                android:layout_marginTop="@dimen/activity_main_tab_margin_top_and_bottom"
                android:layout_marginRight="30dp"
                android:background="@drawable/message_icon"
                android:button="@null" />
            <RadioButton
                android:id="@+id/contact_rbn"
                android:layout_width="@dimen/contact_icon_width"
                android:layout_height="@dimen/contact_icon_height"
                android:layout_marginBottom="@dimen/activity_main_tab_margin_top_and_bottom"
                android:layout_marginTop="@dimen/activity_main_tab_margin_top_and_bottom"


                android:background="@drawable/contact_icon"
                android:button="@null" />
            <RadioButton
                android:id="@+id/dongtai_rbn"
                android:layout_width="@dimen/dongtai_icon_width"
                android:layout_height="@dimen/dongtai_icon_height"
                android:layout_marginBottom="@dimen/activity_main_tab_margin_top_and_bottom"
                android:layout_marginTop="@dimen/activity_main_tab_margin_top_and_bottom"
                android:background="@drawable/dongtai_icon"
                android:layout_marginLeft="30dp"
                android:button="@null" />
        </RadioGroup>
    </RelativeLayout>

</LinearLayout>
```
MainActivity.java
```java
import com.itheima.qq.fragment.ContactFragment;
import com.itheima.qq.fragment.DongtaiFragment;
import com.itheima.qq.fragment.SessionFrament;
import android.content.Intent;
import android.os.Bundle;
import android.support.v4.app.FragmentActivity;
import android.support.v4.app.FragmentManager;
import android.support.v4.app.FragmentTransaction;
import android.widget.RadioButton;
import android.widget.RadioGroup;
import android.widget.RadioGroup.OnCheckedChangeListener;
import android.widget.TextView;
public class MainActivity extends FragmentActivity implements OnCheckedChangeListener {
    public static final int COUNT = 3;
    private RadioButton session_rbn;
    private RadioButton contact_rbn;
    private RadioButton dongtai_rbn;
    private TextView title_tv;
    private RadioGroup radioGroup;
    private FragmentManager fragmentManager;
    private SessionFrament sessionFragment;
    private ContactFragment contactFragment;
    private static final String TAG_SESSION_FRAGMENT = "SessionFragment";
    private static final String TAG_CONTACT_FRAGMENT = "ContactFragment";
    private static final String TAG_DONGTAI_FRAGMENT = "DongtaiFragment";
    private DongtaiFragment dongtaiFragment;



    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    @Override
    protected void onResume() {
        super.onResume();
        //初始化控件
        initView();
        //初始化监听器监听切换Fragment 事件
        initListener();
        //初始化Fragment
        initData();
        //开启聊天服务进程，监听聊天消息
        startChatService();
    }

    private void initListener() {
        radioGroup.setOnCheckedChangeListener(this);
    }

    private void startChatService() {
        Intent intent = new Intent(this, ChatService.class);
        startService(intent);
    }

    private void initData() {
        fragmentManager = getSupportFragmentManager();
        sessionFragment = new SessionFrament();
        contactFragment = new ContactFragment();
        dongtaiFragment = new DongtaiFragment();
        //默认选中消息Fragment
        radioGroup.check(R.id.session_rbn);

    }

    private void initView() {
        title_tv = (TextView) findViewById(R.id.tv_title);
        session_rbn = (RadioButton) findViewById(R.id.session_rbn);
        contact_rbn = (RadioButton) findViewById(R.id.contact_rbn);
        dongtai_rbn = (RadioButton) findViewById(R.id.dongtai_rbn);
        radioGroup = (RadioGroup) findViewById(R.id.rg_group);
    }

    @Override
    public void onCheckedChanged(RadioGroup group, int checkedId) {
        // 把其他选中状态取消
        session_rbn.setChecked(checkedId == R.id.session_rbn);
        contact_rbn.setChecked(checkedId == R.id.contact_rbn);
        dongtai_rbn.setChecked(checkedId == R.id.dongtai_rbn);
        if (checkedId == R.id.session_rbn) {
            title_tv.setText(getResources().getString(R.string.title_message));
            FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
            fragmentTransaction.replace(R.id.fl_content, sessionFragment,
                    TAG_SESSION_FRAGMENT);
            fragmentTransaction.commit();
        } else if (checkedId == R.id.contact_rbn) {
            title_tv.setText(getResources().getString(R.string.title_contact));
            FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
            fragmentTransaction.replace(R.id.fl_content, contactFragment,
                    TAG_CONTACT_FRAGMENT);
            fragmentTransaction.commit();
        } else if (checkedId == R.id.dongtai_rbn) {
            title_tv.setText(getResources().getString(R.string.title_dongtai));
            FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
            fragmentTransaction.replace(R.id.fl_content, dongtaiFragment,
                    TAG_DONGTAI_FRAGMENT);
            fragmentTransaction.commit();
        }
    }
}
```

## 3.5 退出登录

退出登录的功能是在动态中实现的。消息、联系人、动态都是Fragment 实现的。核心代码：
```java
if (xmppConnection.isConnected()) {
      if (xmppConnection.isAuthenticated()) {
          try {
              xmppConnection.disconnect();
          } catch (Exception e) {
              e.printStackTrace();
              Looper.prepare();
              Toast.makeText(application, "注销失败"+e, 0).show();


              Looper.loop();
              getActivity().finish();
              return ;
          }
      }
  }

  DongtaiFragment.java
  public class DongtaiFragment extends Fragment {
      private TextView       tv_name;
      private RelativeLayout rl_logout;

      @Override
      public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle
              savedInstanceState) {
          View view = initView();
          return view;
      }

      private View initView() {
          View view = View.inflate(getActivity(), R.layout.fragment_dongtai2, null);
          SharedPreferences sp = getActivity().getSharedPreferences("config",
                  Context.MODE_PRIVATE);
          String name = sp.getString("name", "");
          tv_name = (TextView) view.findViewById(R.id.tv_name);
          rl_logout = (RelativeLayout) view.findViewById(R.id.rl_logout);
          if (!TextUtils.isEmpty(name)) {
              tv_name.setText(name);
          }
          rl_logout.setOnClickListener(new OnClickListener() {
              @Override
              public void onClick(View v) {
                  final QQApplication application = (QQApplication)
                          getActivity().getApplication();
                  final XMPPConnection xmppConnection = application.getXmppConnection();
                  new Thread(new Runnable() {
                      @Override
                      public void run() {
                          if (xmppConnection.isConnected()) {
                              if (xmppConnection.isAuthenticated()) {
                                  try {
                                      xmppConnection.disconnect();
                                  } catch (Exception e) {
                                      e.printStackTrace();


                                      Looper.prepare();
                                      Toast.makeText(application, "注销失败"+e, 0).show();
                                      Looper.loop();
                                      getActivity().finish();
                                      return ;
                                  }
                                  Looper.prepare();
                                  Toast.makeText(application, "注销成功", 0).show();
                                  getActivity().startActivity(new Intent(application,
                                          LoginActivity.class));
                                  getActivity().finish();
                                  Looper.loop();
                              }
                          }
                      }
                  }).start();
              }
          });
          return view;
      }
  }
```
## 3.6 获取联系人功能

联系人界面对应的是ContactFragment，这个界面包含了获取联系人，添加新朋友，添加新组群等三个功能。
我先将上面三个功能的核心代码列出来，然后在把ContactFragment.java 代码给列出出来。
获取到分组，然后每个分组里面有联系人。界面用的是一个ExpendableListView。

获取联系人核心代码：
```java
thread = new Thread(new Runnable() {
     @Override
     public void run() {
         FragmentActivity activity = getActivity();
         if (activity==null) {
             return ;
         }
         QQApplication application = (QQApplication) activity.getApplication();
         xmppConnection = application.getXmppConnection();
         Roster roster = xmppConnection.getRoster();
         Collection<RosterGroup> groups = roster.getGroups();
         Iterator<RosterGroup> iterator = groups.iterator();
         rosterGroups.clear();


         while (iterator.hasNext()) {
             RosterGroup rosterGroup = (RosterGroup) iterator.next();
             rosterGroups.add(rosterGroup);
         }
         handler.sendEmptyMessage(1);
     }
 });
```

添加新朋友核心代码：

 ```java
 public static boolean addUsers(Roster roster, String userName, String name, String groupName) {
     try {
         roster.createEntry(userName, name, new String[]{groupName});
         return true;
     } catch (Exception e) {
         e.printStackTrace();
         Log.e("XmppUtils", "添加好友异常：" + e.getMessage());
     }
 }
 ```
添加新群组核心代码：
```java
public static RosterGroup addGroup(Roster roster, String groupName) {
     try {
         return roster.createGroup(groupName);
     } catch (Exception e) {
         e.printStackTrace();
         Log.e("XmppUtils", "创建分组异常：" + e.getMessage());
         return null;
     }
 }
```
联系人布局ragment_contact.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:orientation="vertical">

    <LinearLayout
        android:id="@+id/rl_roster"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">

        <LinearLayout
            android:id="@+id/ll_new_friend"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:gravity="center_horizontal"
            android:orientation="vertical">

            <ImageView
                android:layout_width="40dp"
                android:layout_height="40dp"
                android:src="@drawable/new_friend"/>

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="新朋友"/>
        </LinearLayout>

        <LinearLayout
            android:id="@+id/ll_new_group"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:gravity="center_horizontal"
            android:orientation="vertical">

            <ImageView
                android:layout_width="40dp"
                android:layout_height="40dp"
                android:src="@drawable/new_group"/>

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="新群组"/>
        </LinearLayout>
    </LinearLayout>

    <View
        android:layout_width="match_parent"
        android:layout_height="0.5dp"
        android:background="@color/devide_line"/>

    <ExpandableListView
        android:id="@+id/lv_roster"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:cacheColorHint="#00000000"
        android:childDivider="@color/devide_line"
        android:groupIndicator="@null"
        android:listSelector="#00000000"
        />
</LinearLayout>
```
联系人代码ContactFragment.java
```java
public class ContactFragment extends Fragment {
    private ExpandableListView lv_roster;
    private MyAdapter          adapter;
    private XMPPConnection     xmppConnection;
    private ArrayList<RosterGroup> rosterGroups = new ArrayList<RosterGroup>();
    private LinearLayout ll_new_friend;
    private LinearLayout ll_new_group;
    private Thread       thread;
    private Handler handler = new Handler(){
        @Override
        public void handleMessage(android.os.Message msg) {
            if (msg.what==0) {
                initData();
            }else if (msg.what==1) {
                adapter.notifyDataSetChanged();
            }
        }
    };

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle
            savedInstanceState) {
        View view = initView();
        initData();
        return view;
    }

    private View initView() {
        View view = View.inflate(getActivity(), R.layout.fragment_contact, null);
        ll_new_friend = (LinearLayout) view.findViewById(R.id.ll_new_friend);
        ll_new_group = (LinearLayout) view.findViewById(R.id.ll_new_group);
        ll_new_friend.setOnClickListener(new OnClickListener() {

            @Override
            public void onClick(View v) {
                //创建一个自定义布局的Dialog
                AlertDialog.Builder builder = new AlertDialog.Builder(getActivity());
                View view = View.inflate(getActivity(), R.layout.dialog_new_friend, null);
                builder.setView(view);
                builder.setTitle("添加新朋友");
                Button button = (Button) view.findViewById(R.id.btn_add);
                final EditText et_name = (EditText) view.findViewById(R.id.et_name);
                final AlertDialog dialog = builder.create();
                dialog.setCanceledOnTouchOutside(false);
                button.setOnClickListener(new OnClickListener() {

                    @Override
                    public void onClick(View v) {
                        String name = et_name.getText().toString();
                        addNewFriend(name,new AddListener() {
                            @Override
                            public void onAddSuccess() {
                                handler.sendEmptyMessage(0);
                                //添加成功后取消Dialog
                                dialog.dismiss();
                            }

                            @Override
                            public void onAddFailure(String msg) {
                                Looper.prepare();
                                Toast.makeText(getActivity(), "添加新朋友失败。"+msg, 0).show();
                                Looper.loop();
                            }
                        });
                    }

                });
                dialog.show();
            }
        });
        ll_new_group.setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View v) {
                AlertDialog.Builder builder = new AlertDialog.Builder(getActivity());
                View view = View.inflate(getActivity(), R.layout.dialog_new_group, null);
                builder.setView(view);
                builder.setTitle("添加新群组");
                final AlertDialog dialog = builder.create();
                Button button = (Button) view.findViewById(R.id.btn_add);
                final EditText et_name = (EditText) view.findViewById(R.id.et_name);
                dialog.setCanceledOnTouchOutside(false);
                button.setOnClickListener(new OnClickListener() {

                    @Override
                    public void onClick(View v) {
                        String name = et_name.getText().toString();
                        addNewGroup(name,new AddListener() {
                            @Override
                            public void onAddSuccess() {
                                dialog.dismiss();
                                handler.sendEmptyMessage(0);
                            }

                            @Override
                            public void onAddFailure(String message) {
                                Looper.prepare();
                                Toast.makeText(getActivity(), "添加新分组失败。"+message, 0).show();
                                Looper.loop();

                            }
                        });

                    }
                });
                dialog.show();
            }
        });
        lv_roster = (ExpandableListView) view.findViewById(R.id.lv_roster);
        lv_roster.setSmoothScrollbarEnabled(false);
        //点击子ListView 的条目，其实也就是点击联系人的时候跳转到聊天界面
        lv_roster.setOnChildClickListener(new OnChildClickListener() {
            @Override
            public boolean onChildClick(ExpandableListView parent, View v, int groupPosition,
                                        Int childPosition, long id) {
                RosterGroup rosterGroup = rosterGroups.get(groupPosition);
                String user = new
                        ArrayList<>(rosterGroup.getEntries()).get(childPosition).getUser();
                Intent chatIntent = new Intent(getActivity(), ChatActivity.class);
                chatIntent.putExtra("user", user);
                startActivity(chatIntent);
                return true;
            }
        });
        return view;
    }
    public void initData() {
        adapter = new MyAdapter();
        lv_roster.setAdapter(adapter);
        //在子线程中请求联系人
        thread = new Thread(new Runnable() {
            @Override
            public void run() {
                FragmentActivity activity = getActivity();
                if (activity==null) {
                    return ;
                }
                QQApplication application = (QQApplication) activity.getApplication();
                xmppConnection = application.getXmppConnection();
                //获取到花名册对象
                Roster roster = xmppConnection.getRoster();
                //获取到所有的分组
                Collection<RosterGroup> groups = roster.getGroups();
                Iterator<RosterGroup> iterator = groups.iterator();
                rosterGroups.clear();
                while (iterator.hasNext()) {
                    RosterGroup rosterGroup = (RosterGroup) iterator.next();
                    rosterGroups.add(rosterGroup);
                }
                handler.sendEmptyMessage(1);
            }
        });
        if (thread.isAlive()) {
            return ;
        }else {
            thread.start();
        }
    }
    //添加新朋友
    private void addNewFriend(final String name,final AddListener listener) {
        final Roster roster = xmppConnection.getRoster();
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    if(!XmppUtils.searchUsers(xmppConnection, name)){
                        if (listener!=null) {
                            listener.onAddFailure(name+"不存在");
                        }
                        return ;
                    }
                    //先判断该用户是否存在
                    XmppUtils.addGroup(roster, "我的好友");//先默认创建一个分组
                    XmppUtils.addUsers(roster,name+"@"+xmppConnection.getServiceName(),
                            name,"我的好友");
                    if (listener!=null) {
                        listener.onAddSuccess();
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                    if(listener!=null){
                        listener.onAddFailure(e.toString());
                    }
                }
            }
        }).start();
    }
    //添加新群组
    private void addNewGroup(final String name,final AddListener listener) {
        final Roster roster = xmppConnection.getRoster();
        new Thread(new Runnable() {
            @Override
            public void run() {
                RosterGroup group = XmppUtils.addGroup(roster, name);
                if(group==null){
                    if (listener!=null) {
                        listener.onAddFailure("创建分组失败。");
                    }
                }
                try {
                    /**
                     * 书写格式，注意书写格式！！！
                     */
                    RosterEntry rosterEntry = roster.getEntry("admin");
                    if (rosterEntry==null) {
                        rosterEntry =
                                roster.getEntry("admin@"+xmppConnection.getServiceName());
                    }
                    if (rosterEntry!=null) {
                        group.addEntry(rosterEntry);


                        if (listener!=null) {
                            listener.onAddSuccess();
                        }
                    }else {
                        if (listener!=null) {
                            listener.onAddFailure("创建分组失败。");
                        }
                    }

                } catch (Exception e) {
                    e.printStackTrace();
                    if (listener!=null) {
                        listener.onAddFailure(e.toString());
                    }
                }
            }
        }).start();
    }

    class MyAdapter extends BaseExpandableListAdapter {

        @Override
        public int getGroupCount() {
            return rosterGroups.size();
        }

        @Override
        public int getChildrenCount(int groupPosition) {
            RosterGroup rosterGroup = rosterGroups.get(groupPosition);
            //根据组群获取该群组下的联系人数量
            return rosterGroup.getEntryCount();
        }
        @Override
        public Object getGroup(int groupPosition) {
            return rosterGroups.get(groupPosition);
        }

        @Override
        public Object getChild(int groupPosition, int childPosition) {

            return rosterGroups.get(groupPosition).getEntries();
        }

        @Override


        public long getGroupId(int groupPosition) {
            return groupPosition;
        }

        @Override
        public long getChildId(int groupPosition, int childPosition) {
            return groupPosition * 100000 + childPosition;
        }

        @Override
        public boolean hasStableIds() {
            return false;
        }

        @Override
        public View getGroupView(int groupPosition, boolean isExpanded, View convertView,
                                 ViewGroup parent) {
            if (convertView == null) {
                convertView = View.inflate(getActivity(), R.layout.list_item_roaster_group, null);
            }
            ImageView iv_indicator = (ImageView)
                    convertView.findViewById(R.id.iv_indicator);
            TextView tv_groupName = (TextView)
                    convertView.findViewById(R.id.roast_group_name);
            TextView tv_roaster_count = (TextView)
                    convertView.findViewById(R.id.tv_roaster_count);
            if (isExpanded) {
                iv_indicator.setBackgroundResource(R.drawable.indicator_expanded);
            } else {
                iv_indicator.setBackgroundResource(R.drawable.indicator_unexpanded);
            }
            RosterGroup rosterGroup = rosterGroups.get(groupPosition);
            String name = rosterGroup.getName();
            tv_groupName.setText(name);
            int entryCount = rosterGroup.getEntryCount();
            tv_roaster_count.setText(entryCount + "");
            return convertView;
        }

        @Override
        public View getChildView(int groupPosition, int childPosition, boolean isLastChild,
                                 View convertView, ViewGroup parent) {
            if (convertView == null) {
                convertView = View.inflate(getActivity(), R.layout.list_item_roster, null);
            }
            TextView tv_name = (TextView) convertView.findViewById(R.id.tv_name);
            RosterGroup rosterGroup = rosterGroups.get(groupPosition);
            List<RosterEntry> list = new ArrayList<RosterEntry>(rosterGroup.getEntries());
            RosterEntry rosterEntry = list.get(childPosition);
            String name = rosterEntry.getUser();
            tv_name.setText(name);
            return convertView;
        }

        @Override
        public boolean isChildSelectable(int groupPosition, int childPosition) {
            return true;
        }

    }
}
```
## 3.7 聊天功能
聊天界面是一个ListView，这个ListView 有两个布局一个是我发送的消息，另外一个是好友发送的消息。

聊天功能核心代码：
```java
if (chatManager==null) {
     //获取聊天管理器
     chatManager = xmppConnection.getChatManager();
 }
 if (chat==null) {
     //创建聊天，并制定消息监听器用于监听好友发送的消息
     chat = chatManager.createChat(user, messageListener);
 }

 chat.sendMessage(msg);

 private MessageListener messageListener = new MessageListener() {

     @Override
     public void processMessage(Chat chat, Message message) {
         String body = message.getBody();
         if (TextUtils.isEmpty(body)) {
             return ;


         }
         com.itheima.qq.bean.Message message2 =new com.itheima.qq.bean.Message();
         message2.setBody(body);
         message2.setTime(TimeUtils.getNowTimeString());
         message2.setFrom(user);
         message2.setTo(xmppConnection.getUser());
         dataList.add(message2);
         MessageDB.putMessage(message2);
         android.os.Message message3 = android.os.Message.obtain();
         message3.what=2;
         handler.sendMessage(message3);
     }
 };
```
activity_chat.xml
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:orientation="vertical" >

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="60dp"
        android:background="@color/title_layout"
        android:gravity="center"
        android:orientation="horizontal" >

        <TextView
            android:id="@+id/tv_title"
            style="@style/TitleStyle"
            android:padding="5dp"
            android:text="马化腾" />
    </LinearLayout>

    <ListView
        android:id="@+id/lv_chat"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1" >
    </ListView>

    <LinearLayout
        android:layout_width="match_parent"


        android:layout_height="50dp"
        android:gravity="center_vertical"
        android:orientation="horizontal" >

        <ImageView
            android:layout_width="30dp"
            android:layout_height="30dp"
            android:layout_marginLeft="10dp"
            android:src="@drawable/chat_emo_normal" />

        <ImageView
            android:layout_width="30dp"
            android:layout_height="30dp"
            android:layout_marginLeft="10dp"
            android:src="@drawable/chat_add_normal" />

        <EditText
            android:id="@+id/et_msg"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_gravity="center_vertical"
            android:layout_weight="1"
            android:background="#FFFFFF"
            android:padding="4dp"
            android:text="你好" />

        <Button
            android:id="@+id/btn_send"
            android:layout_width="60dp"
            android:layout_height="35dp"
            android:layout_marginBottom="3dp"
            android:layout_marginLeft="3dp"
            android:layout_marginRight="3dp"
            android:layout_marginTop="3dp"
            android:background="@color/title_layout"
            android:text="发送"
            android:textColor="#FFFFFF" />
    </LinearLayout>

</LinearLayout>
```
list_item_chat_me.xml

该布局是ListView 中“自己”发送消息时使用的布局。

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="horizontal">

    <TextView
        android:id="@+id/tv_time"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerVertical="true"
        android:layout_marginLeft="10dp"
        android:text="12:32"/>

    <TextView
        android:id="@+id/tv_me_msg"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginRight="10dp"
        android:layout_marginTop="20dp"
        android:layout_toLeftOf="@id/iv_touxiang"
        android:background="@drawable/fv_chat_content_r_normal"
        android:gravity="center_vertical"
        android:paddingRight="25dp"
        android:text="这是我发送的消息"/>

    <ImageView
        android:id="@+id/iv_touxiang"
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:layout_alignParentRight="true"
        android:layout_centerVertical="true"
        android:src="@drawable/kkj"/>

</RelativeLayout>
```
list_item_chat_you.xml
```java
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="horizontal">

    <ImageView
        android:id="@+id/iv_touxiang"
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:layout_centerVertical="true"
        android:src="@drawable/login_default_avatar"/>

    <TextView
        android:id="@+id/tv_you_msg"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginLeft="10dp"
        android:layout_marginTop="5dp"
        android:layout_toRightOf="@id/iv_touxiang"
        android:background="@drawable/fv_chat_content_l_normal"
        android:gravity="center_vertical"
        android:paddingLeft="23dp"
        android:text="这是我发送的消息"/>

    <TextView
        android:id="@+id/tv_time"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentRight="true"
        android:layout_centerVertical="true"
        android:layout_marginRight="10dp"
        android:text="12:32"/>

</RelativeLayout>
```
ChatActivity.java
```java
public class ChatActivity extends Activity implements OnClickListener {
     private ListView       lv_chat;
     private TextView       tv_title;
     private EditText       et_msg;
     private Button         btn_send;
     private XMPPConnection xmppConnection;
     private String         user;
     private ChatManager    chatManager;
     private Chat           chat;
     private MyAdapter      adapter;
     private List<com.itheima.qq.bean.Message> dataList        = new
             ArrayList<com.itheima.qq.bean.Message>();
     private MessageListener                   messageListener = new MessageListener() {



         @Override
         public void processMessage(Chat chat, Message message) {
             String body = message.getBody();
             if (TextUtils.isEmpty(body)) {
                 return ;
             }
             com.itheima.qq.bean.Message message2 =new com.itheima.qq.bean.Message();
             message2.setBody(body);
             message2.setTime(TimeUtils.getNowTimeString());
             message2.setFrom(user);
             message2.setTo(xmppConnection.getUser());
             dataList.add(message2);
             MessageDB.putMessage(message2);
             android.os.Message message3 = android.os.Message.obtain();
             message3.what=2;
             handler.sendMessage(message3);
         }
     };

     private Handler handler = new Handler(){
         public void handleMessage(android.os.Message msg) {
             switch (msg.what) {
                 case 0:
                     Toast.makeText(ChatActivity.this, "消息发送失败"+msg.obj, 0).show();
                     break;
                 case 1:
                     adapter.notifyDataSetChanged();
                     Toast.makeText(ChatActivity.this, "发送成功", 0).show();
                     break;
                 case 2:
                     //接收到新消息
                     adapter.notifyDataSetChanged();
                     break;

                 default:
                     break;
             }
         };
     };
     @Override
     protected void onCreate(Bundle savedInstanceState) {
         super.onCreate(savedInstanceState);
         setContentView(R.layout.activity_chat);
         initView();
         initData();


     }

     private void initData() {
         Intent intent = getIntent();
         user = intent.getStringExtra("user");
         if (TextUtils.isEmpty(user)) {
             Toast.makeText(this, "聊天对象为空", 0).show();
             finish();
         }
         tv_title.setText(user);
         QQApplication application = (QQApplication)getApplication();
         xmppConnection = application.getXmppConnection();
         ArrayList<com.itheima.qq.bean.Message> messages =
                 MessageDB.getMessagesIgnoreFromAndTo(xmppConnection.getUser(), user);
         if (messages!=null) {
             dataList = messages;
         }
     }

     private void initView() {
         lv_chat = (ListView) findViewById(R.id.lv_chat);
         tv_title = (TextView) findViewById(R.id.tv_title);
         et_msg = (EditText) findViewById(R.id.et_msg);
         btn_send = (Button) findViewById(R.id.btn_send);
         btn_send.setOnClickListener(this);
         adapter = new MyAdapter();
         lv_chat.setAdapter(adapter);
     }

     @Override
     public void onClick(View v) {
         if(v.getId()==R.id.btn_send){
             String msg = et_msg.getText().toString();
             sendMsg(msg);
         }
     }
     @Override
     protected void onDestroy() {
         super.onDestroy();
         if (chat!=null) {
             chat.removeMessageListener(messageListener);
         }
         startService(new Intent(this, ChatService.class));
     }


     @Override
     protected void onPause() {
         super.onPause();
         if (TextUtils.isEmpty(user)) {
             return;
         }
         Session session = new Session();
         session.setTime(TimeUtils.getNowTimeString());
         com.itheima.qq.bean.Message message = dataList.get(dataList.size()-1);
         session.setFrom(message.getFrom());
         session.setTo(message.getTo());
         session.setMsg(message.getBody());
         session.setUsr(user);
         SessionDB.updateSession(session );
     }
     @Override
     protected void onResume() {
         super.onResume();
         if (chatManager==null) {
             chatManager = xmppConnection.getChatManager();
         }
         if (chat==null) {
             chat = chatManager.createChat(user, messageListener);
         }
     }
     private void sendMsg(final String msg) {
         if (TextUtils.isEmpty(msg)) {
             Toast.makeText(this, "不能发送空消息", 0).show();
             return ;
         }
         new Thread(new Runnable() {

             @Override
             public void run() {

                 try {
                     chat.sendMessage(msg);
                     com.itheima.qq.bean.Message message = new
                             com.itheima.qq.bean.Message();
                     message.setBody(msg);
                     message.setTime(TimeUtils.getNowTimeString());
                     message.setTo(user);
                     message.setFrom(xmppConnection.getUser());
                     dataList.add(message);
                     MessageDB.putMessage(message);

                     //发送成功
                     android.os.Message message2 = android.os.Message.obtain();
                     message2.what = 1;
                     handler.sendMessage(message2);
                 } catch (XMPPException e) {
                     e.printStackTrace();
                     android.os.Message message = android.os.Message.obtain();
                     message.what = 0;
                     message.obj = e.toString();
                     handler.sendMessage(message);
                 }
             }
         }).start();
     }

     class MyAdapter extends BaseAdapter {

         @Override
         public int getCount() {
             return dataList.size();
         }

         @Override
         public Object getItem(int position) {
             return dataList.get(position);
         }

         @Override
         public long getItemId(int position) {
             return position;
         }

         @Override
         public int getViewTypeCount() {
             return 2;
         }

         @Override
         public View getView(int position, View convertView, ViewGroup parent) {
             com.itheima.qq.bean.Message message = dataList.get(position);
             if (message.getFrom().equals(xmppConnection.getUser())) {
                 //自己的消息
                 if (convertView==null) {
                     convertView = View.inflate(ChatActivity.this,
                             R.layout.list_item_chat_me, null);
                 }
                 TextView tv_me_msg = (TextView) convertView.findViewById(R.id.tv_me_msg);
                 TextView tv_time = (TextView) convertView.findViewById(R.id.tv_time);
                 tv_me_msg.setText(message.getBody());
                 tv_time.setText(message.getTime());
             }else {
                 //别人的消息
                 if (convertView==null) {
                     convertView = View.inflate(ChatActivity.this,
                             R.layout.list_item_chat_you, null);
                 }
                 TextView tv_you_msg = (TextView)
                         convertView.findViewById(R.id.tv_you_msg);
                 TextView tv_time = (TextView) convertView.findViewById(R.id.tv_time);
                 tv_you_msg.setText(message.getBody());
                 tv_time.setText(message.getTime());
             }
             return convertView;
         }
     }
 }
```
## 3.8 消息界面

SessionFragment 布局就是一个ListView 十分的简单，因此就不给出了。

SessionFragment.java
```java
public class SessionFrament extends Fragment {

    private ListView lv_session;
    private ArrayList<Session> dataList = new ArrayList<Session>();
    private MyAdapter adapter = new MyAdapter();

    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle
            savedInstanceState) {
        View view = initView();
        initData();
        return view;
    }

    private View initView() {


        View view = View.inflate(getActivity(), R.layout.fragment_session, null);
        lv_session = (ListView) view.findViewById(R.id.lv_session);
        return view;
    }

    public void initData() {
        lv_session.setAdapter(adapter);
        ArrayList<Session> sessions = SessionDB.getSessions();
        if (sessions != null && sessions.size() > 0) {
            dataList = sessions;
            adapter.notifyDataSetChanged();
        }
        lv_session.setOnItemClickListener(new OnItemClickListener() {

            @Override
            public void onItemClick(AdapterView<?> parent, View view, int position, long id)
            {
                String user = dataList.get(position).getUsr();
                Toast.makeText(getActivity(), user, 0).show();
                Intent chatIntent = new Intent(getActivity(), ChatActivity.class);
                chatIntent.putExtra("user", user);
                startActivity(chatIntent);
            }
        });
    };

    class MyAdapter extends BaseAdapter {
        @Override
        public int getCount() {
            return dataList.size();
        }

        @Override
        public Object getItem(int position) {
            return dataList.get(position);
        }

        @Override
        public long getItemId(int position) {
            return position;
        }

        @Override
        public View getView(int position, View convertView, ViewGroup parent) {


            ViewHolder viewHolder = null;
            if (convertView == null) {
                viewHolder = new ViewHolder();
                convertView = View.inflate(getActivity(), R.layout.list_item_session, null);
                viewHolder.tv_msg = (TextView) convertView.findViewById(R.id.tv_msg);
                viewHolder.tv_name = (TextView) convertView.findViewById(R.id.tv_name);
                viewHolder.tv_time = (TextView) convertView.findViewById(R.id.tv_time);
                convertView.setTag(viewHolder);
            } else {
                viewHolder = (ViewHolder) convertView.getTag();
            }
            Session session = dataList.get(position);
            viewHolder.tv_msg.setText(session.getMsg());
            viewHolder.tv_time.setText(session.getTime());
            // 到底是from 还是to
            viewHolder.tv_name.setText(session.getUsr());
            return convertView;
        }

    }
    @Override
    public void onResume() {
        super.onResume();
        if(adapter!=null){
            adapter.notifyDataSetChanged();
        }
        SessionDB.setListener(new SessionListener() {

            @Override
            public void onChange() {
                if (adapter!=null) {
                    adapter.notifyDataSetChanged();
                }
            }
        });
    }
    static class ViewHolder {
        TextView tv_name;
        TextView tv_time;
        TextView tv_msg;
    }
}
```

## 3.9 启动服务监听消息

当用户登录成功的时候在后台开启一个服务，用于监听主动发过来的消息。

ChatService.java
```java
public class ChatService extends Service {
    private static QQApplication       application;
    private static NotificationManager notificationManager;
    private        ChatManager         chatManager;
    private MessageListener     messageListener     = new MessageListener() {
        public void processMessage(Chat chat, Message message) {
            if (TextUtils.isEmpty(message.getBody())) {
                return ;
            }
            Session session = new Session();
            session.setMsg(message.getBody());
            String from = message.getFrom();
            if (from.endsWith("/Spark")) {
                from = from.substring(0, from.length()-"/Spark".length());
            }
            session.setFrom(from);
            session.setTo(message.getTo());
            session.setUsr(from);
            session.setTime(TimeUtils.getNowTimeString());
            SessionDB.updateSession(session);
            com.itheima.qq.bean.Message message2 = new com.itheima.qq.bean.Message();
            message2.setBody(message.getBody());
            message2.setFrom(from);
            message2.setTo(message.getTo());
            message2.setTime(TimeUtils.getNowTimeString());
            MessageDB.putMessage(message2);
            android.os.Message msg = android.os.Message.obtain();
            msg.obj = message;
            handler.sendMessage(msg);
        }
    };
    private ChatManagerListener chatManagerListener = new ChatManagerListener() {
        @Override
        public void chatCreated(Chat chat, boolean createdLocally) {
            if (!createdLocally) {
                chat.addMessageListener(messageListener);
            }


        }
    };
    private Handler             handler             = new Handler() {
        public void handleMessage(android.os.Message msg) {
            Notification notification = new Notification(R.drawable.kkj, "收到新消息",
                    SystemClock.uptimeMillis());
            notification.flags = Notification.FLAG_AUTO_CANCEL;
            Intent intent = new Intent(ChatService.this, ChatActivity.class);
            Message message = (Message)msg.obj;
            String from = message.getFrom();
            if (from.endsWith("/Spark")) {
                from = from.substring(0, from.length()-"/Spark".length());
            }
            intent.putExtra("user",from);
            intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
            PendingIntent pendingIntent = PendingIntent.getActivity(ChatService.this, 1,intent,
                    PendingIntent.FLAG_UPDATE_CURRENT);
            notification.setLatestEventInfo(ChatService.this, "新消息", "请注意查收",
                    pendingIntent);
            notificationManager.notify(2, notification);
        };
    };
    public IBinder onBind(Intent intent) {
        return null;
    }
    public void onCreate() {
        super.onCreate();
        application = (QQApplication) getApplication();
        notificationManager = (NotificationManager)
                ChatService.this.getSystemService(Context.NOTIFICATION_SERVICE);
    }
    public int onStartCommand(Intent intent, int flags, int startId) {
        initListener();
        return super.onStartCommand(intent, flags, startId);
    }
    private void initListener() {
        XMPPConnection xmppConnection = application.getXmppConnection();
        if (xmppConnection==null) {
            return;
        }
        chatManager = xmppConnection.getChatManager();
        Toast.makeText(this, "服务已经开启", 0).show();
        chatManager.addChatListener(chatManagerListener);
    }
}
```
