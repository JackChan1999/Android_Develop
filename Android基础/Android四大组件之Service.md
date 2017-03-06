本篇博客讲解的是Android四大组件之Service，下面是内容摘要：

- 了解Service
- Android 中的进程
- 电话窃听器
- Service 的生命周期
- AIDL的使用

# **1. Service 的基本概念**

## **1.1 Android 官方文档对Service 的解释如下**

> A Service is an application component that can perform long-running operations in the background and does not provide a user interface. Another application component can start a service and it will continue to run in the background even if the user switches to another application. Additionally, a component can bind to a service to interact with it and even perform interprocess communication (IPC). For example, a service might handle network transactions, play music, perform file I/O, or interact with a content provider, all from the background.

**翻译**
Service 是应用组件之一，用来执行需要长时间运行的操作，Service 运行在后台并且跟用户没有交互界面。第三方应用的组件（指四大组件的任意一种）可以启动一个Service，一旦启动起来，该Service 就一直在后台运行，就算用户已经将该应用置为后台进程。另外，组件也可以通过绑定的形式跟一个Service进行交互，甚至完成进程间通信。比如：Service 可以在后台处理网络传输、播放音乐、进行I/O 流的读写或者跟内容提供者进行交互。

## **1.2 Service 的特点如下**

- Service 在Android 中是一种长生命周期的组件，它不实现任何用户界面，是一个没有界面的Activity
- Service 长期在后台运行，执行不关乎界面的一些操作比如：网易新闻服务，每隔1 分钟去服务查看是否有最新新闻
- Service 和Thread 有点相似，但是使用Thread 不安全，不严谨
- Service 和其他组件一样，都是运行在主线程中，因此不能用它来做耗时的操作

# **2. Android 中的进程**

在Android 中进程优先级由高到低，依次分为Foreground process、Visible process、Service process、Background process、Empty process。

Android 官方文档的解释如下：

**1、Foreground process 前台进程**
A process that is required for what the user is currently doing. A process is consideredto be in the foreground if any of the following conditions are true:

- It hosts an Activity that the user is interacting with (the Activity's onResume() method has been called).
- It hosts a Service that's bound to the activity that the user is interacting with.
- It hosts a Service that's running "in the foreground"—the service has called startForeground().
- It hosts a Service that's executing one of its lifecycle callbacks (onCreate(),onStart(), or onDestroy()).
- It hosts a BroadcastReceiver that's executing its onReceive() method.Generally, only a few foreground processes exist at any given time. They are killed only as a last resort—if memory is so low that they cannot all continue to run.Generally, at that point, the device has reached a memory paging state, so killing some foreground processes is required to keep the user interface responsive.

**翻译：**
进程是用户执行当前任务所需要的。下面任何一个条件满足了，那么当前进程就视为前台进程：

- 拥有一个正在和用户交互的Activity（也就是说Activity 的onResume（）方法被执行了）。
- 拥有一个被用户的正在交互的Activity 绑定的Service。
- 拥有一个以“前台模式”运行的Service--该Service 已经被调用了startForeground()方法
- 拥有一个正在执行生命周期方法的Service（onCreate()、onStart()、onDestory()）
- 拥有一个BroadCastReceiver 并且正在执行onReceive()方法
  通常情况下，在任何时候系统只存在一小部分前台进程。这些进程只会作为最后的手段才会被杀死-当内存不足以继续运行他们的时候。
  通常情况下，在这个时刻，设备已经达到内存分页状态（当系统达到内存分页状态时只能通过虚拟地址访问内存，是不是很不好理解？那大家就理解为达到这个状态时系统已经无法继续分配新的内存空间即可），因此杀死一些前台进程（释放内存空间）以保证应用能够继续响应用户的交互是必要的手段。

**2、Visible process 可视进程，可以看见，但不可以交互**
## **2.1 进程的回收机制**

Android 系统有一套内存回收机制,会根据优先级进行回收。Android 系统会尽可能的维持程序的进程,但是终究还是需要回收一些旧的进程节省内存提供给新的或者重要的进程使用。

进程的回收顺序是：从低到高

- 当系统内存不够用时, 会把空进程一个一个回收掉
- 当系统回收所有的完空进程不够用时, 继续向上回收后台进程, 依次类推
- 当系统回收所有的完空进程不够用时, 继续向上回收后台进程, 依次类推
- 但是当回收服务, 可视, 前台这三种进程时, 系统非必要情况下不会轻易回收, 如果需要回收掉这三种进程, 那么在系统内存够用时, 会再给重新启动进程;但是服务进程如果用户手动的关闭服务, 这时服
  务不会再重启了。
## **2.2 为什么用服务而不是线程**

服务是运行在后台的进程，跟我们熟悉的Thread 很像，Android 官方文档给出Service 和Thread 的区别如下：

Service 是Android 中的（四大）组件之一，即使用户跟你的应用（不可见）没交互也依然运行在系统后台。因此，如果你真的需要这样的服务时才需要创建一个Service。

如果你不需要在主线程中做一些工作，而是仅仅当应用跟用户交互的时候做一些工作，那么你就可以创建一个子线程而不是开启一个服务。比如，如果你想播放音乐，但是仅仅想当你的Activity 处于运行状态时播放，那么你就可以在onCreate()方法中创建个线程对象，然后在onStart()方法中启动该线程，最后在onStop()方法中停止（终止该线程）播放。当然也可以考虑使用AsyncTask 和HandlerThread 来替代传统的Thread。可以查看Processes and Threading 文档获取更多关于线程的内容。

在使用Service 时一定要注意，Service 默认运行在应用的主线程中，因此如果你需要在Service 中执行大量的或者阻塞（比如网络访问、IO 等）的任务，那么依然需要在Service 中开启一个Thread 来完成这些
工作。

其实使用Service 并不影响我们使用Thread，而且很多情形下，我们都是在Service 中开启子Thread的混合使用方式。
# **3. 案例-电话窃听器**

# **4. Service 的生命周期**

![service](https://developer.android.google.cn/images/service_lifecycle.png)

# **5. Android 中服务的调用**

## **5.1管理绑定服务的生命周期**

![service](https://developer.android.google.cn/images/fundamentals/service_binding_tree_lifecycle.png)

## **5.2 绑定服务**

### **5.2.1 扩展 Binder 类**

LocalService.java
```java
public class LocalService extends Service {
    private NotificationManager mNM;

    // Unique Identification Number for the Notification.
    // We use it on Notification start, and to cancel it.
    private int NOTIFICATION = R.string.local_service_started;

    /**
     * Class for clients to access.  Because we know this service always
     * runs in the same process as its clients, we don't need to deal with
     * IPC.
     */
    public class LocalBinder extends Binder {
        LocalService getService() {
            return LocalService.this;
        }
    }
    
    @Override
    public void onCreate() {
        mNM = (NotificationManager)getSystemService(NOTIFICATION_SERVICE);

        // Display a notification about us starting.  We put an icon in the status bar.
        showNotification();
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        Log.i("LocalService", "Received start id " + startId + ": " + intent);
        return START_NOT_STICKY;
    }

    @Override
    public void onDestroy() {
        // Cancel the persistent notification.
        mNM.cancel(NOTIFICATION);

        // Tell the user we stopped.
        Toast.makeText(this, R.string.local_service_stopped, Toast.LENGTH_SHORT).show();
    }

    @Override
    public IBinder onBind(Intent intent) {
        return mBinder;
    }

    // This is the object that receives interactions from clients.  See
    // RemoteService for a more complete example.
    private final IBinder mBinder = new LocalBinder();

    /**
     * Show a notification while this service is running.
     */
    private void showNotification() {
        // In this sample, we'll use the same text for the ticker and the expanded notification
        CharSequence text = getText(R.string.local_service_started);

        // The PendingIntent to launch our activity if the user selects this notification
        PendingIntent contentIntent = PendingIntent.getActivity(this, 0,
                new Intent(this, LocalServiceActivities.Controller.class), 0);

        // Set the info for the views that show in the notification panel.
        Notification notification = new Notification.Builder(this)
                .setSmallIcon(R.drawable.stat_sample)  // the status icon
                .setTicker(text)  // the status text
                .setWhen(System.currentTimeMillis())  // the time stamp
                .setContentTitle(getText(R.string.local_service_label))  // the label of the entry
                .setContentText(text)  // the contents of the entry
                .setContentIntent(contentIntent)  // The intent to send when the entry is clicked
                .build();

        // Send the notification.
        mNM.notify(NOTIFICATION, notification);
    }
}
```
LocalServiceActivities.java
```java
public class LocalServiceActivities {
    /**
     * <p>Example of explicitly starting and stopping the local service.
     * This demonstrates the implementation of a service that runs in the same
     * process as the rest of the application, which is explicitly started and stopped
     * as desired.</p>
     * 
     * <p>Note that this is implemented as an inner class only keep the sample
     * all together; typically this code would appear in some separate class.
     */
    public static class Controller extends Activity {
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);

            setContentView(R.layout.local_service_controller);

            // Watch for button clicks.
            Button button = (Button)findViewById(R.id.start);
            button.setOnClickListener(mStartListener);
            button = (Button)findViewById(R.id.stop);
            button.setOnClickListener(mStopListener);
        }

        private OnClickListener mStartListener = new OnClickListener() {
            public void onClick(View v) {
                // Make sure the service is started.  It will continue running
                // until someone calls stopService().  The Intent we use to find
                // the service explicitly specifies our service component, because
                // we want it running in our own process and don't want other
                // applications to replace it.
                startService(new Intent(Controller.this,
                        LocalService.class));
            }
        };

        private OnClickListener mStopListener = new OnClickListener() {
            public void onClick(View v) {
                // Cancel a previous call to startService().  Note that the
                // service will not actually stop at this point if there are
                // still bound clients.
                stopService(new Intent(Controller.this,
                        LocalService.class));
            }
        };
    }

    // ----------------------------------------------------------------------

    /**
     * Example of binding and unbinding to the local service.
     * This demonstrates the implementation of a service which the client will
     * bind to, receiving an object through which it can communicate with the service.</p>
     * 
     * <p>Note that this is implemented as an inner class only keep the sample
     * all together; typically this code would appear in some separate class.
     */
    public static class Binding extends Activity {
        private boolean mIsBound;


        private LocalService mBoundService;
        
        private ServiceConnection mConnection = new ServiceConnection() {
            public void onServiceConnected(ComponentName className, IBinder service) {
                // This is called when the connection with the service has been
                // established, giving us the service object we can use to
                // interact with the service.  Because we have bound to a explicit
                // service that we know is running in our own process, we can
                // cast its IBinder to a concrete class and directly access it.
                mBoundService = ((LocalService.LocalBinder)service).getService();
                
                // Tell the user about this for our demo.
                Toast.makeText(Binding.this, R.string.local_service_connected,
                        Toast.LENGTH_SHORT).show();
            }

            public void onServiceDisconnected(ComponentName className) {
                // This is called when the connection with the service has been
                // unexpectedly disconnected -- that is, its process crashed.
                // Because it is running in our same process, we should never
                // see this happen.
                mBoundService = null;
                Toast.makeText(Binding.this, R.string.local_service_disconnected,
                        Toast.LENGTH_SHORT).show();
            }
        };
        
        void doBindService() {
            // Establish a connection with the service.  We use an explicit
            // class name because we want a specific service implementation that
            // we know will be running in our own process (and thus won't be
            // supporting component replacement by other applications).
            bindService(new Intent(Binding.this, 
                    LocalService.class), mConnection, Context.BIND_AUTO_CREATE);
            mIsBound = true;
        }
        
        void doUnbindService() {
            if (mIsBound) {
                // Detach our existing connection.
                unbindService(mConnection);
                mIsBound = false;
            }
        }
        
        @Override
        protected void onDestroy() {
            super.onDestroy();
            doUnbindService();
        }


        private OnClickListener mBindListener = new OnClickListener() {
            public void onClick(View v) {
                doBindService();
            }
        };

        private OnClickListener mUnbindListener = new OnClickListener() {
            public void onClick(View v) {
                doUnbindService();
            }
        };
        
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);

            setContentView(R.layout.local_service_binding);

            // Watch for button clicks.
            Button button = (Button)findViewById(R.id.bind);
            button.setOnClickListener(mBindListener);
            button = (Button)findViewById(R.id.unbind);
            button.setOnClickListener(mUnbindListener);
        }
    }
}
```
### **5.2.2 使用 Messenger**
```java
public class MessengerService extends Service {
    /** For showing and hiding our notification. */
    NotificationManager mNM;
    /** Keeps track of all current registered clients. */
    ArrayList<Messenger> mClients = new ArrayList<Messenger>();
    /** Holds last value set by a client. */
    int mValue = 0;
    
    /**
     * Command to the service to register a client, receiving callbacks
     * from the service.  The Message's replyTo field must be a Messenger of
     * the client where callbacks should be sent.
     */
    static final int MSG_REGISTER_CLIENT = 1;
    
    /**
     * Command to the service to unregister a client, ot stop receiving callbacks
     * from the service.  The Message's replyTo field must be a Messenger of
     * the client as previously given with MSG_REGISTER_CLIENT.
     */
    static final int MSG_UNREGISTER_CLIENT = 2;
    
    /**
     * Command to service to set a new value.  This can be sent to the
     * service to supply a new value, and will be sent by the service to
     * any registered clients with the new value.
     */
    static final int MSG_SET_VALUE = 3;
    
    /**
     * Handler of incoming messages from clients.
     */
    class IncomingHandler extends Handler {
        @Override
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case MSG_REGISTER_CLIENT:
                    mClients.add(msg.replyTo);
                    break;
                case MSG_UNREGISTER_CLIENT:
                    mClients.remove(msg.replyTo);
                    break;
                case MSG_SET_VALUE:
                    mValue = msg.arg1;
                    for (int i=mClients.size()-1; i>=0; i--) {
                        try {
                            mClients.get(i).send(Message.obtain(null,
                                    MSG_SET_VALUE, mValue, 0));
                        } catch (RemoteException e) {
                            // The client is dead.  Remove it from the list;
                            // we are going through the list from back to front
                            // so this is safe to do inside the loop.
                            mClients.remove(i);
                        }
                    }
                    break;
                default:
                    super.handleMessage(msg);
            }
        }
    }
    
    /**
     * Target we publish for clients to send messages to IncomingHandler.
     */
    final Messenger mMessenger = new Messenger(new IncomingHandler());
    
    @Override
    public void onCreate() {
        mNM = (NotificationManager)getSystemService(NOTIFICATION_SERVICE);

        // Display a notification about us starting.
        showNotification();
    }

    @Override
    public void onDestroy() {
        // Cancel the persistent notification.
        mNM.cancel(R.string.remote_service_started);

        // Tell the user we stopped.
        Toast.makeText(this, R.string.remote_service_stopped, Toast.LENGTH_SHORT).show();
    }
    
    /**
     * When binding to the service, we return an interface to our messenger
     * for sending messages to the service.
     */
    @Override
    public IBinder onBind(Intent intent) {
        return mMessenger.getBinder();
    }

    /**
     * Show a notification while this service is running.
     */
    private void showNotification() {
        // In this sample, we'll use the same text for the ticker and the expanded notification
        CharSequence text = getText(R.string.remote_service_started);

        // The PendingIntent to launch our activity if the user selects this notification
        PendingIntent contentIntent = PendingIntent.getActivity(this, 0,
                new Intent(this, Controller.class), 0);

        // Set the info for the views that show in the notification panel.
        Notification notification = new Notification.Builder(this)
                .setSmallIcon(R.drawable.stat_sample)  // the status icon
                .setTicker(text)  // the status text
                .setWhen(System.currentTimeMillis())  // the time stamp
                .setContentTitle(getText(R.string.local_service_label))  // the label of the entry
                .setContentText(text)  // the contents of the entry
                .setContentIntent(contentIntent)  // The intent to send when the entry is clicked
                .build();

        // Send the notification.
        // We use a string id because it is a unique number.  We use it later to cancel.
        mNM.notify(R.string.remote_service_started, notification);
    }
}
```
MessengerServiceActivities.java
```java
public class MessengerServiceActivities {
    /**
     * Example of binding and unbinding to the remote service.
     * This demonstrates the implementation of a service which the client will
     * bind to, interacting with it through an aidl interface.</p>
     * 
     * <p>Note that this is implemented as an inner class only keep the sample
     * all together; typically this code would appear in some separate class.
     */
    public static class Binding extends Activity {

        /** Messenger for communicating with service. */
        Messenger mService = null;
        /** Flag indicating whether we have called bind on the service. */
        boolean mIsBound;
        /** Some text view we are using to show state information. */
        TextView mCallbackText;
        
        /**
         * Handler of incoming messages from service.
         */
        class IncomingHandler extends Handler {
            @Override
            public void handleMessage(Message msg) {
                switch (msg.what) {
                    case MessengerService.MSG_SET_VALUE:
                        mCallbackText.setText("Received from service: " + msg.arg1);
                        break;
                    default:
                        super.handleMessage(msg);
                }
            }
        }
        
        /**
         * Target we publish for clients to send messages to IncomingHandler.
         */
        final Messenger mMessenger = new Messenger(new IncomingHandler());
        
        /**
         * Class for interacting with the main interface of the service.
         */
        private ServiceConnection mConnection = new ServiceConnection() {
            public void onServiceConnected(ComponentName className,
                    IBinder service) {
                // This is called when the connection with the service has been
                // established, giving us the service object we can use to
                // interact with the service.  We are communicating with our
                // service through an IDL interface, so get a client-side
                // representation of that from the raw service object.
                mService = new Messenger(service);
                mCallbackText.setText("Attached.");

                // We want to monitor the service for as long as we are
                // connected to it.
                try {
                    Message msg = Message.obtain(null,
                            MessengerService.MSG_REGISTER_CLIENT);
                    msg.replyTo = mMessenger;
                    mService.send(msg);
                    
                    // Give it some value as an example.
                    msg = Message.obtain(null,
                            MessengerService.MSG_SET_VALUE, this.hashCode(), 0);
                    mService.send(msg);
                } catch (RemoteException e) {
                    // In this case the service has crashed before we could even
                    // do anything with it; we can count on soon being
                    // disconnected (and then reconnected if it can be restarted)
                    // so there is no need to do anything here.
                }
                
                // As part of the sample, tell the user what happened.
                Toast.makeText(Binding.this, R.string.remote_service_connected,
                        Toast.LENGTH_SHORT).show();
            }

            public void onServiceDisconnected(ComponentName className) {
                // This is called when the connection with the service has been
                // unexpectedly disconnected -- that is, its process crashed.
                mService = null;
                mCallbackText.setText("Disconnected.");

                // As part of the sample, tell the user what happened.
                Toast.makeText(Binding.this, R.string.remote_service_disconnected,
                        Toast.LENGTH_SHORT).show();
            }
        };
        
        void doBindService() {
            // Establish a connection with the service.  We use an explicit
            // class name because there is no reason to be able to let other
            // applications replace our component.
            bindService(new Intent(Binding.this, 
                    MessengerService.class), mConnection, Context.BIND_AUTO_CREATE);
            mIsBound = true;
            mCallbackText.setText("Binding.");
        }
        
        void doUnbindService() {
            if (mIsBound) {
                // If we have received the service, and hence registered with
                // it, then now is the time to unregister.
                if (mService != null) {
                    try {
                        Message msg = Message.obtain(null,
                                MessengerService.MSG_UNREGISTER_CLIENT);
                        msg.replyTo = mMessenger;
                        mService.send(msg);
                    } catch (RemoteException e) {
                        // There is nothing special we need to do if the service
                        // has crashed.
                    }
                }
                
                // Detach our existing connection.
                unbindService(mConnection);
                mIsBound = false;
                mCallbackText.setText("Unbinding.");
            }
        }

        
        /**
         * Standard initialization of this activity.  Set up the UI, then wait
         * for the user to poke it before doing anything.
         */
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);

            setContentView(R.layout.messenger_service_binding);

            // Watch for button clicks.
            Button button = (Button)findViewById(R.id.bind);
            button.setOnClickListener(mBindListener);
            button = (Button)findViewById(R.id.unbind);
            button.setOnClickListener(mUnbindListener);
            
            mCallbackText = (TextView)findViewById(R.id.callback);
            mCallbackText.setText("Not attached.");
        }

        private OnClickListener mBindListener = new OnClickListener() {
            public void onClick(View v) {
                doBindService();
            }
        };

        private OnClickListener mUnbindListener = new OnClickListener() {
            public void onClick(View v) {
                doUnbindService();
            }
        };
    }
}
```
### **5.2.3 使用 AIDL**

IRemoteService.aidl
```java
/**
 * Example of defining an interface for calling on to a remote service
 * (running in another process).
 */
interface IRemoteService {
    /**
     * Often you want to allow a service to call back to its clients.
     * This shows how to do so, by registering a callback interface with
     * the service.
     */
    void registerCallback(IRemoteServiceCallback cb);
    
    /**
     * Remove a previously registered callback interface.
     */
    void unregisterCallback(IRemoteServiceCallback cb);
}
```
ISecondary.aidl
```
/**
 * Example of a secondary interface associated with a service.  (Note that
 * the interface itself doesn't impact, it is just a matter of how you
 * retrieve it from the service.)
 */
interface ISecondary {
    /**
     * Request the PID of this service, to do evil things with it.
     */
    int getPid();
    
    /**
     * This demonstrates the basic types that you can use as parameters
     * and return values in AIDL.
     */
    void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat,
            double aDouble, String aString);
}
```
IRemoteServiceCallback.aidl
```java
/**
 * Example of a callback interface used by IRemoteService to send
 * synchronous notifications back to its clients.  Note that this is a
 * one-way interface so the server does not block waiting for the client.
 */
oneway interface IRemoteServiceCallback {
    /**
     * Called when the service has a new value for you.
     */
    void valueChanged(int value);
}
```

RemoteService.java
```java
public class RemoteService extends Service {
    /**
     * This is a list of callbacks that have been registered with the
     * service.  Note that this is package scoped (instead of private) so
     * that it can be accessed more efficiently from inner classes.
     */
    final RemoteCallbackList<IRemoteServiceCallback> mCallbacks
            = new RemoteCallbackList<IRemoteServiceCallback>();
    
    int mValue = 0;
    NotificationManager mNM;
    
    @Override
    public void onCreate() {
        mNM = (NotificationManager)getSystemService(NOTIFICATION_SERVICE);

        // Display a notification about us starting.
        showNotification();
        
        // While this service is running, it will continually increment a
        // number.  Send the first message that is used to perform the
        // increment.
        mHandler.sendEmptyMessage(REPORT_MSG);
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        Log.i("LocalService", "Received start id " + startId + ": " + intent);
        return START_NOT_STICKY;
    }

    @Override
    public void onDestroy() {
        // Cancel the persistent notification.
        mNM.cancel(R.string.remote_service_started);

        // Tell the user we stopped.
        Toast.makeText(this, R.string.remote_service_stopped, Toast.LENGTH_SHORT).show();
        
        // Unregister all callbacks.
        mCallbacks.kill();
        
        // Remove the next pending message to increment the counter, stopping
        // the increment loop.
        mHandler.removeMessages(REPORT_MSG);
    }
    

    @Override
    public IBinder onBind(Intent intent) {
        // Select the interface to return.  If your service only implements
        // a single interface, you can just return it here without checking
        // the Intent.
        if (IRemoteService.class.getName().equals(intent.getAction())) {
            return mBinder;
        }
        if (ISecondary.class.getName().equals(intent.getAction())) {
            return mSecondaryBinder;
        }
        return null;
    }

    /**
     * The IRemoteInterface is defined through IDL
     */
    private final IRemoteService.Stub mBinder = new IRemoteService.Stub() {
        public void registerCallback(IRemoteServiceCallback cb) {
            if (cb != null) mCallbacks.register(cb);
        }
        public void unregisterCallback(IRemoteServiceCallback cb) {
            if (cb != null) mCallbacks.unregister(cb);
        }
    };

    /**
     * A secondary interface to the service.
     */
    private final ISecondary.Stub mSecondaryBinder = new ISecondary.Stub() {
        public int getPid() {
            return Process.myPid();
        }
        public void basicTypes(int anInt, long aLong, boolean aBoolean,
                float aFloat, double aDouble, String aString) {
        }
    };

    
    @Override
    public void onTaskRemoved(Intent rootIntent) {
        Toast.makeText(this, "Task removed: " + rootIntent, Toast.LENGTH_LONG).show();
    }
    
    private static final int REPORT_MSG = 1;

    /**
     * Our Handler used to execute operations on the main thread.  This is used
     * to schedule increments of our value.
     */
    private final Handler mHandler = new Handler() {
        @Override public void handleMessage(Message msg) {
            switch (msg.what) {
                
                // It is time to bump the value!
                case REPORT_MSG: {
                    // Up it goes.
                    int value = ++mValue;
                    
                    // Broadcast to all clients the new value.
                    final int N = mCallbacks.beginBroadcast();
                    for (int i=0; i<N; i++) {
                        try {
                            mCallbacks.getBroadcastItem(i).valueChanged(value);
                        } catch (RemoteException e) {
                            // The RemoteCallbackList will take care of removing
                            // the dead object for us.
                        }
                    }
                    mCallbacks.finishBroadcast();
                    
                    // Repeat every 1 second.
                    sendMessageDelayed(obtainMessage(REPORT_MSG), 1*1000);
                } break;
                default:
                    super.handleMessage(msg);
            }
        }
    };

    /**
     * Show a notification while this service is running.
     */
    private void showNotification() {
        // In this sample, we'll use the same text for the ticker and the expanded notification
        CharSequence text = getText(R.string.remote_service_started);

        // The PendingIntent to launch our activity if the user selects this notification
        PendingIntent contentIntent = PendingIntent.getActivity(this, 0,
                new Intent(this, Controller.class), 0);

        // Set the info for the views that show in the notification panel.
        Notification notification = new Notification.Builder(this)
                .setSmallIcon(R.drawable.stat_sample)  // the status icon
                .setTicker(text)  // the status text
                .setWhen(System.currentTimeMillis())  // the time stamp
                .setContentTitle(getText(R.string.remote_service_label))  // the label of the entry
                .setContentText(text)  // the contents of the entry
                .setContentIntent(contentIntent)  // The intent to send when the entry is clicked
                .build();

        // Send the notification.
        // We use a string id because it is a unique number.  We use it later to cancel.
        mNM.notify(R.string.remote_service_started, notification);
    }
    
    // ----------------------------------------------------------------------
    
    /**
     * <p>Example of explicitly starting and stopping the remove service.
     * This demonstrates the implementation of a service that runs in a different
     * process than the rest of the application, which is explicitly started and stopped
     * as desired.</p>
     * 
     * <p>Note that this is implemented as an inner class only keep the sample
     * all together; typically this code would appear in some separate class.
     */
    public static class Controller extends Activity {
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);

            setContentView(R.layout.remote_service_controller);

            // Watch for button clicks.
            Button button = (Button)findViewById(R.id.start);
            button.setOnClickListener(mStartListener);
            button = (Button)findViewById(R.id.stop);
            button.setOnClickListener(mStopListener);
        }

        private OnClickListener mStartListener = new OnClickListener() {
            public void onClick(View v) {
                // Make sure the service is started.  It will continue running
                // until someone calls stopService().
                // We use an action code here, instead of explictly supplying
                // the component name, so that other packages can replace
                // the service.
                startService(new Intent(Controller.this, RemoteService.class));
            }
        };

        private OnClickListener mStopListener = new OnClickListener() {
            public void onClick(View v) {
                // Cancel a previous call to startService().  Note that the
                // service will not actually stop at this point if there are
                // still bound clients.
                stopService(new Intent(Controller.this, RemoteService.class));
            }
        };
    }
    
    // ----------------------------------------------------------------------
    
    /**
     * Example of binding and unbinding to the remote service.
     * This demonstrates the implementation of a service which the client will
     * bind to, interacting with it through an aidl interface.</p>
     * 
     * <p>Note that this is implemented as an inner class only keep the sample
     * all together; typically this code would appear in some separate class.
     */

    public static class Binding extends Activity {
        /** The primary interface we will be calling on the service. */
        IRemoteService mService = null;
        /** Another interface we use on the service. */
        ISecondary mSecondaryService = null;
        
        Button mKillButton;
        TextView mCallbackText;

        private boolean mIsBound;

        /**
         * Standard initialization of this activity.  Set up the UI, then wait
         * for the user to poke it before doing anything.
         */
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);

            setContentView(R.layout.remote_service_binding);

            // Watch for button clicks.
            Button button = (Button)findViewById(R.id.bind);
            button.setOnClickListener(mBindListener);
            button = (Button)findViewById(R.id.unbind);
            button.setOnClickListener(mUnbindListener);
            mKillButton = (Button)findViewById(R.id.kill);
            mKillButton.setOnClickListener(mKillListener);
            mKillButton.setEnabled(false);
            
            mCallbackText = (TextView)findViewById(R.id.callback);
            mCallbackText.setText("Not attached.");
        }

        /**
         * Class for interacting with the main interface of the service.
         */
        private ServiceConnection mConnection = new ServiceConnection() {
            public void onServiceConnected(ComponentName className,
                    IBinder service) {
                // This is called when the connection with the service has been
                // established, giving us the service object we can use to
                // interact with the service.  We are communicating with our
                // service through an IDL interface, so get a client-side
                // representation of that from the raw service object.
                mService = IRemoteService.Stub.asInterface(service);
                mKillButton.setEnabled(true);
                mCallbackText.setText("Attached.");

                // We want to monitor the service for as long as we are
                // connected to it.
                try {
                    mService.registerCallback(mCallback);
                } catch (RemoteException e) {
                    // In this case the service has crashed before we could even
                    // do anything with it; we can count on soon being
                    // disconnected (and then reconnected if it can be restarted)
                    // so there is no need to do anything here.
                }
                
                // As part of the sample, tell the user what happened.
                Toast.makeText(Binding.this, R.string.remote_service_connected,
                        Toast.LENGTH_SHORT).show();
            }

            public void onServiceDisconnected(ComponentName className) {
                // This is called when the connection with the service has been
                // unexpectedly disconnected -- that is, its process crashed.
                mService = null;
                mKillButton.setEnabled(false);
                mCallbackText.setText("Disconnected.");

                // As part of the sample, tell the user what happened.
                Toast.makeText(Binding.this, R.string.remote_service_disconnected,
                        Toast.LENGTH_SHORT).show();
            }
        };

        /**
         * Class for interacting with the secondary interface of the service.
         */
        private ServiceConnection mSecondaryConnection = new ServiceConnection() {
            public void onServiceConnected(ComponentName className,
                    IBinder service) {
                // Connecting to a secondary interface is the same as any
                // other interface.
                mSecondaryService = ISecondary.Stub.asInterface(service);
                mKillButton.setEnabled(true);
            }

            public void onServiceDisconnected(ComponentName className) {
                mSecondaryService = null;
                mKillButton.setEnabled(false);
            }
        };

        private OnClickListener mBindListener = new OnClickListener() {
            public void onClick(View v) {
                // Establish a couple connections with the service, binding
                // by interface names.  This allows other applications to be
                // installed that replace the remote service by implementing
                // the same interface.
                Intent intent = new Intent(Binding.this, RemoteService.class);
                intent.setAction(IRemoteService.class.getName());
                bindService(intent, mConnection, Context.BIND_AUTO_CREATE);
                intent.setAction(ISecondary.class.getName());
                bindService(intent, mSecondaryConnection, Context.BIND_AUTO_CREATE);
                mIsBound = true;
                mCallbackText.setText("Binding.");
            }
        };

        private OnClickListener mUnbindListener = new OnClickListener() {
            public void onClick(View v) {
                if (mIsBound) {
                    // If we have received the service, and hence registered with
                    // it, then now is the time to unregister.
                    if (mService != null) {
                        try {
                            mService.unregisterCallback(mCallback);
                        } catch (RemoteException e) {
                            // There is nothing special we need to do if the service
                            // has crashed.
                        }
                    }
                    
                    // Detach our existing connection.
                    unbindService(mConnection);
                    unbindService(mSecondaryConnection);
                    mKillButton.setEnabled(false);
                    mIsBound = false;
                    mCallbackText.setText("Unbinding.");
                }
            }
        };

        private OnClickListener mKillListener = new OnClickListener() {
            public void onClick(View v) {
                // To kill the process hosting our service, we need to know its
                // PID.  Conveniently our service has a call that will return
                // to us that information.
                if (mSecondaryService != null) {
                    try {
                        int pid = mSecondaryService.getPid();
                        // Note that, though this API allows us to request to
                        // kill any process based on its PID, the kernel will
                        // still impose standard restrictions on which PIDs you
                        // are actually able to kill.  Typically this means only
                        // the process running your application and any additional
                        // processes created by that app as shown here; packages
                        // sharing a common UID will also be able to kill each
                        // other's processes.
                        Process.killProcess(pid);
                        mCallbackText.setText("Killed service process.");
                    } catch (RemoteException ex) {
                        // Recover gracefully from the process hosting the
                        // server dying.
                        // Just for purposes of the sample, put up a notification.
                        Toast.makeText(Binding.this,
                                R.string.remote_call_failed,
                                Toast.LENGTH_SHORT).show();
                    }
                }
            }
        };
        
        // ----------------------------------------------------------------------
        // Code showing how to deal with callbacks.
        // ----------------------------------------------------------------------
        
        /**
         * This implementation is used to receive callbacks from the remote
         * service.
         */
        private IRemoteServiceCallback mCallback = new IRemoteServiceCallback.Stub() {
            /**
             * This is called by the remote service regularly to tell us about
             * new values.  Note that IPC calls are dispatched through a thread
             * pool running in each process, so the code executing here will
             * NOT be running in our main thread like most other things -- so,
             * to update the UI, we need to use a Handler to hop over there.
             */
            public void valueChanged(int value) {
                mHandler.sendMessage(mHandler.obtainMessage(BUMP_MSG, value, 0));
            }
        };
        
        private static final int BUMP_MSG = 1;
        
        private Handler mHandler = new Handler() {
            @Override public void handleMessage(Message msg) {
                switch (msg.what) {
                    case BUMP_MSG:
                        mCallbackText.setText("Received from service: " + msg.arg1);
                        break;
                    default:
                        super.handleMessage(msg);
                }
            }
            
        };
    }


    // ----------------------------------------------------------------------

    /**
     * Examples of behavior of different bind flags.</p>
     */

    public static class BindingOptions extends Activity {
        ServiceConnection mCurConnection;
        TextView mCallbackText;
        Intent mBindIntent;

        class MyServiceConnection implements ServiceConnection {
            final boolean mUnbindOnDisconnect;

            public MyServiceConnection() {
                mUnbindOnDisconnect = false;
            }

            public MyServiceConnection(boolean unbindOnDisconnect) {
                mUnbindOnDisconnect = unbindOnDisconnect;
            }

            public void onServiceConnected(ComponentName className,
                    IBinder service) {
                if (mCurConnection != this) {
                    return;
                }
                mCallbackText.setText("Attached.");
                Toast.makeText(BindingOptions.this, R.string.remote_service_connected,
                        Toast.LENGTH_SHORT).show();
            }

            public void onServiceDisconnected(ComponentName className) {
                if (mCurConnection != this) {
                    return;
                }
                mCallbackText.setText("Disconnected.");
                Toast.makeText(BindingOptions.this, R.string.remote_service_disconnected,
                        Toast.LENGTH_SHORT).show();
                if (mUnbindOnDisconnect) {
                    unbindService(this);
                    mCurConnection = null;
                    Toast.makeText(BindingOptions.this, R.string.remote_service_unbind_disconn,
                            Toast.LENGTH_SHORT).show();
                }
            }
        }

        /**
         * Standard initialization of this activity.  Set up the UI, then wait
         * for the user to poke it before doing anything.
         */
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);

            setContentView(R.layout.remote_binding_options);

            // Watch for button clicks.
            Button button = (Button)findViewById(R.id.bind_normal);
            button.setOnClickListener(mBindNormalListener);
            button = (Button)findViewById(R.id.bind_not_foreground);
            button.setOnClickListener(mBindNotForegroundListener);
            button = (Button)findViewById(R.id.bind_above_client);
            button.setOnClickListener(mBindAboveClientListener);
            button = (Button)findViewById(R.id.bind_allow_oom);
            button.setOnClickListener(mBindAllowOomListener);
            button = (Button)findViewById(R.id.bind_waive_priority);
            button.setOnClickListener(mBindWaivePriorityListener);
            button = (Button)findViewById(R.id.bind_important);
            button.setOnClickListener(mBindImportantListener);
            button = (Button)findViewById(R.id.bind_with_activity);
            button.setOnClickListener(mBindWithActivityListener);
            button = (Button)findViewById(R.id.unbind);
            button.setOnClickListener(mUnbindListener);

            mCallbackText = (TextView)findViewById(R.id.callback);
            mCallbackText.setText("Not attached.");

            mBindIntent = new Intent(this, RemoteService.class);
            mBindIntent.setAction(IRemoteService.class.getName());
        }

        private OnClickListener mBindNormalListener = new OnClickListener() {
            public void onClick(View v) {
                if (mCurConnection != null) {
                    unbindService(mCurConnection);
                    mCurConnection = null;
                }
                ServiceConnection conn = new MyServiceConnection();
                if (bindService(mBindIntent, conn, Context.BIND_AUTO_CREATE)) {
                    mCurConnection = conn;
                }
            }
        };

        private OnClickListener mBindNotForegroundListener = new OnClickListener() {
            public void onClick(View v) {
                if (mCurConnection != null) {
                    unbindService(mCurConnection);
                    mCurConnection = null;
                }
                ServiceConnection conn = new MyServiceConnection();
                if (bindService(mBindIntent, conn,
                        Context.BIND_AUTO_CREATE | Context.BIND_NOT_FOREGROUND)) {
                    mCurConnection = conn;
                }
            }
        };

        private OnClickListener mBindAboveClientListener = new OnClickListener() {
            public void onClick(View v) {
                if (mCurConnection != null) {
                    unbindService(mCurConnection);
                    mCurConnection = null;
                }
                ServiceConnection conn = new MyServiceConnection();
                if (bindService(new Intent(IRemoteService.class.getName()),
                        conn, Context.BIND_AUTO_CREATE | Context.BIND_ABOVE_CLIENT)) {
                    mCurConnection = conn;
                }
            }
        };

        private OnClickListener mBindAllowOomListener = new OnClickListener() {
            public void onClick(View v) {
                if (mCurConnection != null) {
                    unbindService(mCurConnection);
                    mCurConnection = null;
                }
                ServiceConnection conn = new MyServiceConnection();
                if (bindService(mBindIntent, conn,
                        Context.BIND_AUTO_CREATE | Context.BIND_ALLOW_OOM_MANAGEMENT)) {
                    mCurConnection = conn;
                }
            }
        };

        private OnClickListener mBindWaivePriorityListener = new OnClickListener() {
            public void onClick(View v) {
                if (mCurConnection != null) {
                    unbindService(mCurConnection);
                    mCurConnection = null;
                }
                ServiceConnection conn = new MyServiceConnection(true);
                if (bindService(mBindIntent, conn,
                        Context.BIND_AUTO_CREATE | Context.BIND_WAIVE_PRIORITY)) {
                    mCurConnection = conn;
                }
            }
        };

        private OnClickListener mBindImportantListener = new OnClickListener() {
            public void onClick(View v) {
                if (mCurConnection != null) {
                    unbindService(mCurConnection);
                    mCurConnection = null;
                }
                ServiceConnection conn = new MyServiceConnection();
                if (bindService(mBindIntent, conn,
                        Context.BIND_AUTO_CREATE | Context.BIND_IMPORTANT)) {
                    mCurConnection = conn;
                }
            }
        };

        private OnClickListener mBindWithActivityListener = new OnClickListener() {
            public void onClick(View v) {
                if (mCurConnection != null) {
                    unbindService(mCurConnection);
                    mCurConnection = null;
                }
                ServiceConnection conn = new MyServiceConnection();
                if (bindService(mBindIntent, conn,
                        Context.BIND_AUTO_CREATE | Context.BIND_ADJUST_WITH_ACTIVITY
                        | Context.BIND_WAIVE_PRIORITY)) {
                    mCurConnection = conn;
                }
            }
        };

        private OnClickListener mUnbindListener = new OnClickListener() {
            public void onClick(View v) {
                if (mCurConnection != null) {
                    unbindService(mCurConnection);
                    mCurConnection = null;
                }
            }
        };
    }
}
```

## **5.3 服务两种启动方式**

* startService：服务被启动之后，跟启动它的组件没有一毛钱关系
* bindService：跟启动它的组件同生共死
* 绑定服务和解绑服务的生命周期方法：onCreate->onBind->onUnbind->onDestroy
## **5.4 案例1：服务的开启方式**

```
public class MainActivity extends Activity {

    private Intent intent;
	private MyServiceConn conn;

	@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        intent = new Intent(this, MyService.class);
        conn = new MyServiceConn();
    }


    public void start(View v){
    	startService(intent);
    }
    
    public void stop(View v){
    	stopService(intent);
    }
    
    public void bind(View v){
    	//绑定服务
    	bindService(intent, conn, BIND_AUTO_CREATE);
    }
    
    public void unbind(View v){
    	//解绑服务
    	unbindService(conn);
    }
    
    class MyServiceConn implements ServiceConnection{

    	//服务连接成功时，此方法调用
		@Override
		public void onServiceConnected(ComponentName name, IBinder service) {
			// TODO Auto-generated method stub
			
		}

		//服务失去连接时，此方法调用
		@Override
		public void onServiceDisconnected(ComponentName name) {
			// TODO Auto-generated method stub
		}
    }
}
```

```java
public class MyService extends Service {

	//绑定时调用
	@Override
	public IBinder onBind(Intent intent) {
		// TODO Auto-generated method stub
		System.out.println("bind方法");
		return null;
	}

	//解绑时调用
	@Override
	public boolean onUnbind(Intent intent) {
		// TODO Auto-generated method stub
		System.out.println("unbind方法");
		return super.onUnbind(intent);
	}
	
	@Override
	public void onCreate() {
		// TODO Auto-generated method stub
		super.onCreate();
		System.out.println("create方法");
	}
	
	@Override
	public int onStartCommand(Intent intent, int flags, int startId) {
		// TODO Auto-generated method stub
		System.out.println("start方法");
		return super.onStartCommand(intent, flags, startId);
	}
	
	@Override
	public void onDestroy() {
		// TODO Auto-generated method stub
		System.out.println("destroy方法");
		super.onDestroy();
	}
	
}
```
## **5.5 找领导办证**

* 把服务看成一个领导，服务中有一个banZheng方法，如何才能访问？
* 绑定服务时，会触发服务的onBind方法，此方法会返回一个Ibinder的对象给MainActivity，通过这个对象访问服务中的方法
* 绑定服务

```java
Intent intent = new Intent(this, BanZhengService.class);
bindService(intent, conn, BIND_AUTO_CREATE);
```

* 绑定服务时要求传入一个ServiceConnection实现类的对象
* 定义这个实现类

```java
class MyServiceconn implements ServiceConnection{
		@Override
		public void onServiceConnected(ComponentName name, IBinder service) {
			zjr = (PublicBusiness) service;
		}
		@Override
		public void onServiceDisconnected(ComponentName name) {	
		}
    	
    }
```

* 创建实现类对象

```java
conn = new MyServiceconn();
```

* 在服务中定义一个类实现Ibinder接口，以在onBind方法中返回

```java
class ZhongJianRen extends Binder implements PublicBusiness{
		public void QianXian(){
			//访问服务中的banZheng方法
			BanZheng();
		}	
		public void daMaJiang(){
			
		}
	}
```

* 把QianXian方法抽取到接口PublicBusiness中定义

## **5.6 案例2：找领导办证**
```
public class LeaderService extends Service {

	@Override
	public IBinder onBind(Intent intent) {
		// 返回一个Binder对象，这个对象就是中间人对象
		return new ZhouMi();
	}

	class ZhouMi extends Binder implements PublicBusiness{
		public void QianXian(){
			banZheng();
		}
		
		public  void daMaJiang(){
			System.out.println("陪李处打麻将");
		}
	}
	
	public void banZheng(){
		System.out.println("李处帮你来办证");
	}
}
```

```
public class MainActivity extends Activity {

    private Intent intent;
	private MyServiceConn conn;
	PublicBusiness pb;
	
	@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        intent = new Intent(this, LeaderService.class);
        conn = new MyServiceConn();
        //绑定领导服务
        bindService(intent, conn, BIND_AUTO_CREATE);
    }
    
    public void click(View v){
    	//调用服务的办证方法
    	pb.QianXian();
    }

    class MyServiceConn implements ServiceConnection{

    	//连接服务成功，此方法调用
		@Override
		public void onServiceConnected(ComponentName name, IBinder service) {
			// TODO Auto-generated method stub
			pb = (PublicBusiness) service;
		}

		@Override
		public void onServiceDisconnected(ComponentName name) {
			// TODO Auto-generated method stub	
		}
    }
}
```
```
public interface PublicBusiness {
	void QianXian();
}
```
## **5.7 两种启动方法混合使用**

* 用服务实现音乐播放时，因为音乐播放必须运行在服务进程中，可是音乐服务中的方法，需要被前台Activity所调用，所以需要混合启动音乐服务
* 先start，再bind，销毁时先unbind，在stop

## **5.8 使用服务注册广播接收者**

* Android四大组件都要在清单文件中注册
* 广播接收者比较特殊，既可以在清单文件中注册，也可以直接使用代码注册
* 有的广播接收者，必须代码注册
  * 电量改变
  * 屏幕锁屏和解锁
* 注册广播接收者

```java
//创建广播接收者对象
receiver = new ScreenOnOffReceiver();
//通过IntentFilter对象指定广播接收者接收什么类型的广播
IntentFilter filter = new IntentFilter();
filter.addAction(Intent.ACTION_SCREEN_OFF);
filter.addAction(Intent.ACTION_SCREEN_ON);

//注册广播接收者
registerReceiver(receiver, filter);
```

* 解除注册广播接收者

```java
unregisterReceiver(receiver);
```
* 解除注册之后，广播接收者将失去作用

## **5.9 案例4：使用服务注册广播接收者**

```java
public class MainActivity extends Activity {

    private Intent intent;
	@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        intent = new Intent(this, RegisterService.class);
    }

    public void start(View v){
    	startService(intent);
    }
    public void stop(View v){
    	stopService(intent);
    }    
}
```

```
public class RegisterService extends Service {

	private ScreenReceiver receiver;
	@Override
	public IBinder onBind(Intent intent) {
		// TODO Auto-generated method stub
		return null;
	}

	@Override
	public void onCreate() {
		super.onCreate();
		//1.创建广播接收者对象
		receiver = new ScreenReceiver();
		//2.创建intent-filter对象
		IntentFilter filter = new IntentFilter();
		filter.addAction(Intent.ACTION_SCREEN_OFF);
		filter.addAction(Intent.ACTION_SCREEN_ON);
		
		//3.注册广播接收者
		registerReceiver(receiver, filter);
		
	}
	@Override
	public void onDestroy() {
		super.onDestroy();
		//解除注册
		unregisterReceiver(receiver);
	}
}
```

```
public class ScreenReceiver extends BroadcastReceiver {

	@Override
	public void onReceive(Context context, Intent intent) {
		// TODO Auto-generated method stub
		String action = intent.getAction();
		if(Intent.ACTION_SCREEN_OFF.equals(action)){
			System.out.println("屏幕关闭");
		}
		else if(Intent.ACTION_SCREEN_ON.equals(action)){
			System.out.println("屏幕打开");
		}
	}
}
```
## **5.10本地服务：服务和启动它的组件在同一个进程**

## **5.11 远程服务：服务和启动它的组件不在同一个进程**

* 远程服务只能隐式启动，类似隐式启动Activity，在清单文件中配置Service标签时，必须配置intent-filter子节点，并指定action子节点

# **6. AIDL**

* Android interface definition language
* 安卓接口定义语言
* 作用：跨进程通信
* 应用场景：远程服务中的中间人对象，其他应用是拿不到的，那么在通过绑定服务获取中间人对象时，就无法强制转换，使用aidl，就可以在其他应用中拿到中间人类所实现的接口

## **6.1 支付宝远程服务**

1. 定义支付宝的服务，在服务中定义pay方法
2. 定义中间人对象，把pay方法抽取成接口
3. 把抽取出来的接口后缀名改成aidl
4. 中间人对象直接继承Stub对象
5. 注册这个支付宝服务，定义它的intent-Filter

## **6.2 需要支付的应用**

1. 把刚才定义好的aidl文件拷贝过来，注意aidl文件所在的包名必须跟原包名一致
2. 远程绑定支付宝的服务，通过onServiceConnected方法我们可以拿到中间人对象
3. 把中间人对象通过Stub.asInterface方法强转成定义了pay方法的接口
4. 调用中间人的pay方法

## **6.3 五种前台进程**

1. activity执行了onresume方法，获得焦点
2. 拥有一个跟正在与用户交互的activity绑定的服务
3. 拥有一个服务执行了startForeground()方法
4. 拥有一个正在执行onCreate()、onStart()或者onDestroy()方法中的任意一个的服务
5. 拥有一个正在执行onReceive方法的广播接收者

## **6.4 两种可见进程**

1. activity执行了onPause方法，失去焦点，但是可见
2. 拥有一个跟可见或前台activity绑定的服务

# **7. 服务**

## **7.1 开启方式**

* startService
  * 该方法启动的服务所在的进程属于服务进程
  * Activity一旦启动服务，服务就跟Activity一毛钱关系也没有了
* bindService
  * 该方法启动的服务所在进程不属于服务进程
  * Activity与服务建立连接，Activity一旦死亡，服务也会死亡

* 服务的混合调用
  * 先开始、再绑定，先解绑、再停止

## **7.2 使用代码配置广播接收者**

* 可以使用清单文件注册
  * 广播一旦发出，系统就会去所有清单文件中寻找，哪个广播接收者的action和广播的action是匹配的，如果找到了，就把该广播接收者的进程启动起来
* 可以使用代码注册
  * 需要使用广播接收者时，执行注册的代码，不需要时，执行解除注册的代码

## **7.3 特殊的广播接收者**

* 安卓中有一些广播接收者，必须使用代码注册，清单文件注册是无效的
1. 屏幕锁屏和解锁
2. 电量改变

## **7.4 服务的分类**

* 本地服务：指的是服务和启动服务的activity在同一个进程中
* 远程服务：指的是服务和启动服务的activity不在同一个进程中

## **7.5 AIDL**

* Android interface definition language
* 进程间通信
1. 把远程服务的方法抽取成一个单独的接口java文件
2. 把接口java文件的后缀名改成aidl
3. 在自动生成的PublicBusiness.java文件中，有一个静态抽象类Stub，它已经继承了binder类，实现了publicBusiness接口，这个类就是新的中间人
4. 把aidl文件复制粘贴到06项目，粘贴的时候注意，aidl文件所在的包名必须跟05项目中aidl所在的包名一致
5. 在06项目中，强转中间人对象时，直接使用Stub.asInterface（）

## **7.6 进程优先级**

* 前台进程
  * 拥有一个正在与用户交互的activity（onResume调用）的进程
  * 拥有一个与正在和用户交互的activity绑定的服务的进程
  * 拥有一个正在“运行于前台”的服务——服务的startForeground方法调用
  * 拥有一个正在执行以下三个生命周期方法中任意一个的服务(onCreate(), onStart(), or onDestroy())
  * 拥有一个正在执行onReceive方法的广播接收者的进程
* 可见进程
  * 拥有一个不在前台，但是对用户依然可见的activity（onPause方法调用）的进程
  * 拥有一个与可见（或前台）activity绑定的服务的进程
# **8. 案例** 

## **案例3：音乐播放器**

```
public class MainActivity extends Activity {

	MusicInterface mi;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Intent intent = new Intent(this, MusicService.class);
        //混合调用
        //为了把服务所在进程变成服务进程
        startService(intent);
        //为了拿到中间人对象
        bindService(intent, new MusicServiceConn(), BIND_AUTO_CREATE);
    }

    class MusicServiceConn implements ServiceConnection{

		@Override
		public void onServiceConnected(ComponentName name, IBinder service) {
			// TODO Auto-generated method stub
			mi = (MusicInterface) service;
		}

		@Override
		public void onServiceDisconnected(ComponentName name) {
			// TODO Auto-generated method stub
			
		}
    	
    }

    //开始播放按钮
    public void play(View v){
    	mi.play();
    }
    //暂停播放按钮
    public void pause(View v){
    	mi.pause();
    }
}
```

```
public interface MusicInterface {

	void play();
	void pause();
}
```

```
public class MusicService extends Service{
	@Override
	public IBinder onBind(Intent intent) {
		// TODO Auto-generated method stub
		return new MusicController();
	}
	
	
	//必须继承binder，才能作为中间人对象返回
	class MusicController extends Binder implements MusicInterface{
		public void play(){
			MusicService.this.play();
		}
		public void pause(){
			MusicService.this.pause();
		}
	}
	
	public void play(){
		System.out.println("播放音乐");
	}
	
	public void pause(){
		System.out.println("暂停播放");
	}
}
```

## **案例5：远程服务**

```
public class MainActivity extends Activity {

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
	}

	@Override
	public boolean onCreateOptionsMenu(Menu menu) {
		// Inflate the menu; this adds items to the action bar if it is present.
		getMenuInflater().inflate(R.menu.main, menu);
		return true;
	}
}
```

```
public class RemoteService extends Service {

	@Override
	public IBinder onBind(Intent intent) {
		// TODO Auto-generated method stub
		System.out.println("bind方法");
		return new furong();
	}

	class furong extends Stub{

		@Override
		public void qianXian() {
			// TODO Auto-generated method stub
			banzheng();
		}
	}
```

```
interface PublicBusiness {
	void qianXian();
}
```

## **案例6：启动远程服务**

```
public class RemoteService extends Service {

	@Override
	public IBinder onBind(Intent intent) {
		// TODO Auto-generated method stub
		System.out.println("bind方法");
		return new furong();
	}

	class furong extends Stub{

		@Override
		public void qianXian() {
			// TODO Auto-generated method stub
			banzheng();
		}

	}
```
## **案例7：支付宝**

```
public class MainActivity extends Activity {

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
	}

	@Override
	public boolean onCreateOptionsMenu(Menu menu) {
		// Inflate the menu; this adds items to the action bar if it is present.
		getMenuInflater().inflate(R.menu.main, menu);
		return true;
	}
}
```

```
public class PayService extends Service {

	@Override
	public IBinder onBind(Intent intent) {
		// TODO Auto-generated method stub
		return new PayPangZhi();
	}

	//中间人对象
	class PayPangZhi extends Stub{

		@Override
		public void pay() throws RemoteException {
			//  调用服务的pay方法
			PayService.this.pay();
		}
	}
	
	public void pay(){
		System.out.println("检测运行环境");
		System.out.println("加密用户名密码");
		System.out.println("建立连接");
		System.out.println("上传数据");
		System.out.println("完成支付");
	}
}
```

```
interface PayInterface {
	void pay();
}
```
## **案例8：游戏支付**

```
public class MainActivity extends Activity {

	PayInterface pi;
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		
		Intent intent = new Intent();
		intent.setAction("com.itheima.pangzhi");
		bindService(intent, new ServiceConnection() {
			
			@Override
			public void onServiceDisconnected(ComponentName name) {
				// TODO Auto-generated method stub
			}
			
			@Override
			public void onServiceConnected(ComponentName name, IBinder service) {
				// 使用aidl中自动生成的方法来强转
				pi = Stub.asInterface(service);
			}
		}, BIND_AUTO_CREATE);
	}

	public void click(View v){
		//调用远程服务的支付方法
		try {
			pi.pay();
		} catch (RemoteException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
}
```

```
interface PayInterface {
	void pay();
}
```

# **9. Service Intent must be explicit**

有些时候我们使用Service的时需要采用隐私启动的方式，但是Android 5.0一出来后，其中有个特性就是Service Intent  must be explitict，也就是说从Lollipop开始，service服务必须采用显示方式启动。而android源码是这样写的（源码位置：sdk/sources/android-21/android/app/ContextImpl.java）：

```java
private void validateServiceIntent(Intent service) {
    if (service.getComponent() == null && service.getPackage() == null) {
        if (getApplicationInfo().targetSdkVersion >= Build.VERSION_CODES.LOLLIPOP) {
            IllegalArgumentException ex = new IllegalArgumentException(
                    "Service Intent must be explicit: " + service);
            throw ex;
        } else {
            Log.w(TAG, "Implicit intents with startService are not safe: " + service
                    + " " + Debug.getCallers(2, 3));
        }
    }
}
```

既然，源码里是这样写的，那么这里有两种解决方法：

## [9.1 设置Action和packageName](https://developer.android.google.cn/guide/components/services.html#CreatingStartedService)

为了确保应用的安全性，请始终使用显式 Intent 启动或绑定 Service，且不要为服务声明 Intent 过滤器。 启动哪个服务存在一定的不确定性，而如果对这种不确定性的考量非常有必要，则可为服务提供 Intent 过滤器并从 Intent 中排除相应的组件名称，但随后必须使用 setPackage() 方法设置 Intent 的软件包，这样可以充分消除目标服务的不确定性。

此外，还可以通过添加 android:exported 属性并将其设置为 "false"，确保服务仅适用于您的应用。这可以有效阻止其他应用启动您的服务，即便在使用显式 Intent 时也如此。

参考代码如下：

```java
Intent mIntent = new Intent();
mIntent.setAction("XXX.XXX.XXX");//你定义的service的action
mIntent.setPackage("packagename");//这里你需要设置你应用的包名
context.startService(mIntent);
```

[此方式是google官方推荐使用的解决方法](http://developer.android.com/google/play/billing/billing_integrate.html#billing-service)


##[9.2 将隐式启动转换为显示启动](http://stackoverflow.com/a/26318757/1446466)

```java
public static Intent getExplicitIntent(Context context, Intent implicitIntent) {
        // Retrieve all services that can match the given intent
        PackageManager pm = context.getPackageManager();
        List<ResolveInfo> resolveInfo = pm.queryIntentServices(implicitIntent, 0);
        // Make sure only one match was found
        if (resolveInfo == null || resolveInfo.size() != 1) {
            return null;
        }
        // Get component info and create ComponentName
        ResolveInfo serviceInfo = resolveInfo.get(0);
        String packageName = serviceInfo.serviceInfo.packageName;
        String className = serviceInfo.serviceInfo.name;
        ComponentName component = new ComponentName(packageName, className);
        // Create a new intent. Use the old one for extras and such reuse
        Intent explicitIntent = new Intent(implicitIntent);
        // Set the component to be explicit
        explicitIntent.setComponent(component);
        return explicitIntent;
    }
```

调用方式如下：

```java
Intent mIntent = new Intent();
mIntent.setAction("XXX.XXX.XXX");
Intent eintent = new Intent(getExplicitIntent(mContext,mIntent));
context.startService(eintent);
```
https://developer.android.google.cn/guide/components/services.html