

原文出处：郭霖，http://blog.csdn.net/sinyu890807/article/details/47028975?locationNum=1&fps=1

Context相信所有的Android开发人员基本上每天都在接触，因为它太常见了。但是这并不代表Context没有什么东西好讲的，实际上Context有太多小的细节并不被大家所关注，那么今天我们就来学习一下那些你所不知道的细节。Context细节精华全掌握，从此用起Context再不迷糊。这个知识体系接下来将分为三部曲给大家详细深入说一下，第一篇主要聊聊Context的类型与数量。第二篇会深扒Application Context的设计。第三篇来说说使用Application的问题

# **1. Context的类型与数量**
## **1.1 Context类型**
我们知道，Android应用都是使用Java语言来编写的，那么大家可以思考一下，一个Android程序和一个Java程序，他们最大的区别在哪里？划分界限又是什么呢？其实简单点分析，Android程序不像Java程序一样，随便创建一个类，写个main()方法就能跑了，而是要有一个完整的Android工程环境，在这个环境下，我们有像Activity、Service、BroadcastReceiver等系统组件，而这些组件并不是像一个普通的Java对象new一下就能创建实例的了，而是要有它们各自的上下文环境，也就是我们这里讨论的Context。可以这样讲，Context是维持Android程序中各组件能够正常工作的一个核心功能类。

下面我们来看一下Context的继承结构：

![context](http://bbs.itheima.com/data/attachment/forum/201606/02/100918g558pqtm68z2sssd.jpg.thumb.jpg)

Context的继承结构还是稍微有点复杂的，可以看到，直系子类有两个，一个是ContextWrapper，一个是ContextImpl。那么从名字上就可以看出，ContextWrapper是上下文功能的封装类，而ContextImpl则是上下文功能的实现类。而ContextWrapper又有三个直接的子类， ContextThemeWrapper、Service和Application。其中，ContextThemeWrapper是一个带主题的封装类，而它有一个直接子类就是Activity。

那么在这里我们至少看到了几个所比较熟悉的面孔，Activity、Service、还有Application。由此，其实我们就已经可以得出结论了，Context一共有三种类型，分别是Application、Activity和Service。这三个类虽然分别各种承担着不同的作用，但它们都属于Context的一种，而它们具体Context的功能则是由ContextImpl类去实现的。

那么Context到底可以实现哪些功能呢？这个就实在是太多了，弹出Toast、启动Activity、启动Service、发送广播、操作数据库等都需要用到Context。由于Context的具体能力是由ContextImpl类去实现的，因此在绝大多数场景下，Activity、Service和Application这三种类型的Context都是可以通用的。不过有几种场景比较特殊，比如启动Activity，还有弹出Dialog。出于安全原因的考虑，Android是不允许Activity或Dialog凭空出现的，一个Activity的启动必须要建立在另一个Activity的基础之上，也就是以此形成的返回栈。而Dialog则必须在一个Activity上面弹出（除非是System Alert类型的Dialog），因此在这种场景下，我们只能使用Activity类型的Context，否则将会出错。

## **1.2 Context数量**
那么一个应用程序中到底有多少个Context呢？其实根据上面的Context类型我们就已经可以得出答案了。Context一共有Application、Activity和Service三种类型，因此一个应用程序中Context数量的计算公式就可以这样写：
```
Context数量 = Activity数量 + Service数量 + 1  
```
上面的1代表着Application的数量，因为一个应用程序中可以有多个Activity和多个Service，但是只能有一个Application。

## **1.3 Application Context的设计**
基本上每一个应用程序都会有一个自己的Application，并让它继承自系统的Application类，然后在自己的Application类中去封装一些通用的操作。其实这并不是Google所推荐的一种做法，因为这样我们只是把Application当成了一个通用工具类来使用的，而实际上使用一个简单的单例类也可以实现同样的功能。但是根据观察，有太多的项目都是这样使用Application的。当然这种做法也并没有什么副作用，只是说明还是有不少人对于Application理解的还有些欠缺。那么这里我们先来对Application的设计进行分析，讲一些大家所不知道的细节，然后再看一下平时使用Application的问题。

首先新建一个MyApplication并让它继承自Application，然后在AndroidManifest.xml文件中对MyApplication进行指定，如下所示：

```xml

<application  
    android:name=".MyApplication"  
    android:allowBackup="true"  
    android:icon="@drawable/ic_launcher"  
    android:label="@string/app_name"  
    android:theme="@style/AppTheme" >  
    ......  
</application>
```
指定完成后，当我们的程序启动时Android系统就会创建一个MyApplication的实例，如果这里不指定的话就会默认创建一个Application的实例。 

前面提到过，现在很多的Application都是被当作通用工具类来使用的，那么既然作为一个通用工具类，我们要怎样才能获取到它的实例呢？如下所示：

```java
public class MainActivity extends Activity { 
        @Override 
        protected void onCreate(Bundle savedInstanceState) { 
                   super.onCreate(savedInstanceState); 
                   setContentView(R.layout.activity_main); 
                    MyApplication myApp = (MyApplication) getApplication(); 
                   Log.d("TAG", "getApplication is " + myApp); 
            } 
} 
```
可以看到，代码很简单，只需要调用getApplication()方法就能拿到我们自定义的Application的实例了，打印结果如下所示：

 ![context](http://img.blog.csdn.net/20151101215354919)

那么除了getApplication()方法，其实还有一个getApplicationContext()方法，这两个方法看上去好像有点关联，那么它们的区别是什么呢？我们将代码修改一下：

```java
public class MainActivity extends Activity {  
      
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);  
        MyApplication myApp = (MyApplication) getApplication();  
        Log.d("TAG", "getApplication is " + myApp);  
        Context appContext = getApplicationContext();  
        Log.d("TAG", "getApplicationContext is " + appContext);  
    }  
} 
```
同样，我们把getApplicationContext()的结果打印了出来，现在重新运行代码，结果如下图所示：

![context](http://img.blog.csdn.net/20151101215827074)

咦？好像打印出的结果是一样的呀，连后面的内存地址都是相同的，看来它们是同一个对象。其实这个结果也很好理解，因为前面已经说过了，Application本身就是一个Context，所以这里获取getApplicationContext()得到的结果就是MyApplication本身的实例。

那么，既然这两个方法得到的结果都是相同的，那么Android为什么要提供两个功能重复的方法呢？实际上这两个方法在作用域上有比较大的区别。getApplication()方法的语义性非常强，一看就知道是用来获取Application实例的，但是这个方法只有在Activity和Service中才能调用的到。那么也许在绝大多数情况下我们都是在Activity或者Service中使用Application的，但是如果在一些其它的场景，比如BroadcastReceiver中也想获得Application的实例，这时就可以借助getApplicationContext()方法了，如下所示：

```java
public class MyReceiver extends BroadcastReceiver {  
  
    @Override  
    public void onReceive(Context context, Intent intent) {  
        MyApplication myApp = (MyApplication) context.getApplicationContext();  
        Log.d("TAG", "myApp is " + myApp);  
    }  
}  
```
也就是说，getApplicationContext()方法的作用域会更广一些，任何一个Context的实例，只要调用getApplicationContext()方法都可以拿到我们的Application对象。

那么更加细心的朋友会发现，除了这两个方法之外，其实还有一个getBaseContext()方法，这个baseContext又是什么东西呢？我们还是通过打印的方式来验证一下：

![Context](http://img.blog.csdn.net/20151102215834048)

这次得到的是不同的对象了，getBaseContext()方法得到的是一个ContextImpl对象。这个ContextImpl是不是感觉有点似曾相识？回去看一下Context的继承结构图吧，ContextImpl正是上下文功能的实现类。也就是说像Application、Activity这样的类其实并不会去具体实现Context的功能，而仅仅是做了一层接口封装而已，Context的具体功能都是由ContextImpl类去完成的。那么这样的设计到底是怎么实现的呢？我们还是来看一下源码吧。因为Application、Activity、Service都是直接或间接继承自ContextWrapper的，我们就直接看ContextWrapper的源码，如下所示：

```java
/** 
* Proxying implementation of Context that simply delegates all of its calls to 
* another Context.  Can be subclassed to modify behavior without changing 
* the original Context. 
*/  
public class ContextWrapper extends Context {  
    Context mBase;  
      
    /** 
     * Set the base context for this ContextWrapper.  All calls will then be 
     * delegated to the base context.  Throws 
     * IllegalStateException if a base context has already been set. 
     *  
     * @param base The new base context for this wrapper. 
     */  
    protected void attachBaseContext(Context base) {  
        if (mBase != null) {  
            throw new IllegalStateException("Base context already set");  
        }  
        mBase = base;  
    }  
  
    /** 
     * @return the base context as set by the constructor or setBaseContext 
     */  
    public Context getBaseContext() {  
        return mBase;  
    }  
  
    @Override  
    public AssetManager getAssets() {  
        return mBase.getAssets();  
    }  
  
    @Override  
    public Resources getResources() {  
        return mBase.getResources();  
    }  
  
    @Override  
    public ContentResolver getContentResolver() {  
        return mBase.getContentResolver();  
    }  
  
    @Override  
    public Looper getMainLooper() {  
        return mBase.getMainLooper();  
    }  
      
    @Override  
    public Context getApplicationContext() {  
        return mBase.getApplicationContext();  
    }  
  
    @Override  
    public String getPackageName() {  
        return mBase.getPackageName();  
    }  
  
    @Override  
    public void startActivity(Intent intent) {  
        mBase.startActivity(intent);  
    }  
      
    @Override  
    public void sendBroadcast(Intent intent) {  
        mBase.sendBroadcast(intent);  
    }  
  
    @Override  
    public Intent registerReceiver(  
        BroadcastReceiver receiver, IntentFilter filter) {  
        return mBase.registerReceiver(receiver, filter);  
    }  
  
    @Override  
    public void unregisterReceiver(BroadcastReceiver receiver) {  
        mBase.unregisterReceiver(receiver);  
    }  
  
    @Override  
    public ComponentName startService(Intent service) {  
        return mBase.startService(service);  
    }  
  
    @Override  
    public boolean stopService(Intent name) {  
        return mBase.stopService(name);  
    }  
  
    @Override  
    public boolean bindService(Intent service, ServiceConnection conn,  
            int flags) {  
        return mBase.bindService(service, conn, flags);  
    }  
  
    @Override  
    public void unbindService(ServiceConnection conn) {  
        mBase.unbindService(conn);  
    }  
  
    @Override  
    public Object getSystemService(String name) {  
        return mBase.getSystemService(name);  
    }  
  
    ......  
}  
```
由于ContextWrapper中的方法还是非常多的，我就进行了一些筛选，只贴出来了部分方法。那么上面的这些方法相信大家都是非常熟悉的，getResources()、getPackageName()、getSystemService()等等都是我们经常要用到的方法。那么所有这些方法的实现又是什么样的呢？其实所有ContextWrapper中方法的实现都非常统一，就是调用了mBase对象中对应当前方法名的方法。

那么这个mBase对象又是什么呢？来看第16行的attachBaseContext()方法，这个方法中传入了一个base参数，并把这个参数赋值给了mBase对象。而attachBaseContext()方法其实是由系统来调用的，它会把ContextImpl对象作为参数传递到attachBaseContext()方法当中，从而赋值给mBase对象，之后ContextWrapper中的所有方法其实都是通过这种委托的机制交由ContextImpl去具体实现的，所以说ContextImpl是上下文功能的实现类是非常准确的。

那么另外再看一下我们刚刚打印的getBaseContext()方法，在第26行。这个方法只有一行代码，就是返回了mBase对象而已，而mBase对象其实就是ContextImpl对象，因此刚才的打印结果也得到了印证。

# **4. Application**
自定义Application

```java
public class BaseApplication extends Application {

    private static BaseApplication mApplication;
    private static Context         mContext;
    private static Handler         mHandler;
    private static long            mMainTreadId;

    @Override
    public void onCreate() {
        super.onCreate();

        mApplication = this;
        mHandler = new Handler();
        mMainTreadId = android.os.Process.myTid();
        mContext = getApplicationContext();

        ToastMabager.builder.init(mContext);
        Thread.setDefaultUncaughtExceptionHandler(new ExceptionHandler());
    }

    public static Context getApplication() {
        return mApplication;
    }

    public static Context getContext() {
        return mContext;
    }

    public static Handler getHandler() {
        return mHandler;
    }

    public static long getMainTreadId() {
        return mMainTreadId;
    }

    public enum ToastMabager {
        builder;

        private View     view;
        private TextView tv;
        private Toast    toast;

        /**
         * 初始化toast
         */
        public void init(Context context) {
            view = LayoutInflater.from(context).inflate(R.layout.toast_view, null);
            tv = (TextView) view.findViewById(R.id.tv_toast);
            toast = new Toast(context);
            toast.setView(view);
        }

        /**
         * 显示toast
         * @param content  显示的内容
         * @param duration 持续时间
         */
        public void display(CharSequence content, int duration) {
            if (content.length() != 0) {
                tv.setText(content);
                toast.setDuration(duration);
                toast.setGravity(Gravity.CENTER, 0, 0);
                toast.show();
            }
        }
    }

    class ExceptionHandler implements Thread.UncaughtExceptionHandler {

        @Override
        public void uncaughtException(Thread thread, Throwable ex) {
            ex.printStackTrace();
            try {
                PrintWriter printWriter = new PrintWriter(
                        Environment.getExternalStorageDirectory() + "/googleplay.log");
                ex.printStackTrace(printWriter);
                printWriter.close();
            } catch (FileNotFoundException e) {
                e.printStackTrace();
            }
            Process.killProcess(Process.myPid());
        }
    }
}
```
在&lt;application>标签下配置Application

```xml
<application
    android:name="com.google.base.BaseApplication"
    android:icon="@mipmap/ic_launcher"
    android:label="@string/app_name"
    android:theme="@style/AppTheme">
</application>
```

获取AndroidManifest.xml文件下的Application标签中的meta-data的value值

```java
public String getMetaData() {
    try {
        PackageInfo info = getPackageManager().getPackageInfo(getPackageName(),
                PackageManager.GET_META_DATA);
        String value = info.applicationInfo.metaData.getString("metaname");
        System.out.println("value=" + value);
        return value;
    } catch (PackageManager.NameNotFoundException e) {
        e.printStackTrace();
    }
    return null;
}

```

# **5. 相关阅读**
[Android Context完全解析，你所不知道的Context的各种细节](http://blog.csdn.net/sinyu890807/article/details/47028975?locationNum=1&fps=1)

[Android Context 上下文 你必须知道的一切](http://blog.csdn.net/lmj623565791/article/details/40481055)

Android中的Context详解

http://www.race604.com/android-context-intro/

https://github.com/xesam/tech-translate/blob/master/android/Context-What_Context.md