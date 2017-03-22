![android n](http://bbs.itheima.com/data/attachment/forum/201605/26/151545zx9z9bovevvxu2tu.jpg.thumb.jpg)

2016年5月19日，谷歌在美国加州的山景城举办了 Google I/O 开发者大会中发布。2016年6月，Android N正式命名为“牛轧糖” 本届I/O开发者大会上，Google重点介绍了Android系统三个维度：分别是性能、安全和生产力。其中性能主要新增了Vulkan API与JIT编译器(前者优化图形性能、或者加快软件安装速度)；安全性包括新的数据加密方式、恶意网站识别、系统的实时更新；生产力方面加入了分屏多任务功能、程序的快速切换，所有应用都可以支持“多窗口模式”。

- 支持Vulkan API
- 多窗口模式 (Multi-Window Support)
- 通知机制 (Notifications)
- 流量节省程序(Data Saver)
- Android框架中提供部分ICU4J API支持
- 开始支持Java 8
- Direct Boot（直接启动）
- VR支持

虽说整个大会的重心偏向了人工智能(Google assistant、Allo&Duo 、Google Home)，但Android操作系统作为智能设备的“核心”（Android系统从手表、手机覆盖到电视、汽车，去年有超过600款Android智能手机上市，目前有100款汽车支持Android系统），Android N依旧是I/O大会最为重要看点之一。

# **1. Android N新功能汇总**

本届I/O开发者大会上，Google发布的新一代的Android系统：Android N(7.0)，今年的Android N重点关注了三个维度：分别是性能、安全和生产力。其中性能主要新增了Vulkan API与JIT编译器(前者优化图形性能、或者加快软件安装速度)；安全性包括新的数据加密方式、恶意网站识别、系统的实时更新；生产力方面加入了分屏多任务功能、程序的快速切换，所有应用都可以支持“多窗口模式”。当然，也可以通过修改配置，让应用不支持多窗口模式。

# **2. Android N新功能最亮点**

## **2.1 支持Vulkan API**
Vulkan是OpenGL的下一代版本，和DirectX 12一样都是基于AMD私有的Mantle API，不同的是Vulkan是开源的图形API。Vulkan API是可以直接操纵GPU的一套3D绘图API.。Vulkan与OpenGl想比占用的GPU更少。游戏开发可以使用更华丽的特效

![Vulkan](http://img.blog.csdn.net/20161024095921368)

##**2.2 分屏多任务**

随着手机的尺寸不断刷新上限，智能手机与平板电脑之间的界限正被不断打破。大尺寸屏幕带来极佳视觉体验的同时，也使人们能够操控的屏幕面积增大了不少。大尺寸屏幕也为实现分屏多任务带来了可能性。

早先只能在PC上实现的分屏多任务，如今可以在智能手机上运行。但不同Android ROM实现该功能的方式不一，对软件的兼容也大有不同，很难使全部APP完美兼容。而Android N新增分屏多任务，除了让第三方ROM开发商可以参照这个模板进行二次开发，软件开发商也能根据Android N分屏多任务功能的实现方式去进行软件的开发。大大加快开发速度，由此支持该功能的软件会更多，兼容性也会更好。

![android n](http://bbs.itheima.com/data/attachment/forum/201605/26/151704keye41xfpx5e7b4p.jpg.thumb.jpg)

在运行 Android N 的手机和平板电脑上，用户可以并排运行两个App，或者处于分屏模式时一个App位于另一个App之上。对于Android TV设备，应用程序可以将自己置身于画中画面模式，能够继续显示在用户浏览或与其他应用程序进行交互的内容。

开启分屏多任务的方法十分简单。只要进入后台，按住其中一个卡片向上拖动至顶部即可。当然这项功能支持左右与上下分屏，并且可以拖动中间的分割线来调整两个App的比例。

## **2.3 多窗口模式配置**
### **2.3.1 多窗口模式**
```xml
android:resizeableActivity=["true" | "false"]
```
在清单文件的 <activity> 或 <application> 节点中设置该属性，启用或禁用多窗口显示：

- 如果该属性设置为 true，Activity 将能以分屏和自由形状模式启动。
- 如果此属性设置为 false，Activity 将不支持多窗口模式。
- 如果该值为 false，且用户尝试在多窗口模式下启动 Activity，该 Activity 将全屏显示。
- 如果应用未对该属性指定值，则该属性的值默认设为 true。

### **2.3.2 画中画**
在清单文件的 <activity> 节点中设置该属性，指明 Activity 是否支持画中画显示。

如果 android:resizeableActivity 为 false，将忽略该属性。
```xml
android:supportsPictureInPicture=["true" | "false"]
```
### **2.3.3 layout属性**
| 属性                    | 功能描述                           |
| :-------------------- | :----------------------------- |
| &lt;activity>         | 节点新增的节点，注：处属性应用于画中画模式          |
| android:defaultWidth  | 以自由形状模式启动时 Activity 的默认宽度      |
| android:defaultHeight | 以自由形状模式启动时 Activity 的默认高度      |
| android:gravity       | 以自由形状模式启动时 Activity 的初始位置      |
| android:minimalSize   | 分屏和自由形状模式中 Activity 的最小高度和最小宽度 |
如果用户在分屏模式中移动分界线，使 Activity 尺寸低于指定的最小值，系统会将 Activity 裁剪为用户请求的尺寸。例如:

```xml
<activity android:name=".MyActivity">
    <layout
        android:defaultHeight="500dp"
        android:defaultWidth="600dp"
        android:gravity="top|end"
        android:minimalSize="450dp" />
</activity>  
```

## **2.4 通知**
在Android N中重新设计了通知，可以达到更容易、更快使用的效果。一些主要的变化包括：

**模板更新**：更新了通知模板重点内容和头像。开发者将能够利用的新模板的优势，在他们的代码中实现最低限度的调整。

**捆绑通知**：Android N的通知功能也更加人性化，现在会自动将相同应用的通知捆绑在一起，实现分组显示，并且通过两指滑动实现预览，理论上用户可以在通知界面直接阅读邮件等内容。

**直接回复**：对于实时通信应用程序，Android系统支持在线回复，使用户可以以短信或短信通知界面内快速、直接响应。

**自定义视图**：两个新的 API 让用户在通知中使用自定义视图。

![android n](http://bbs.itheima.com/data/attachment/forum/201605/26/151805whxdehynjnd5aad3.jpg.thumb.jpg)

Android N 开发者预览版的通知系统中还加入了两个全新的 API 接口：Direct Replies 和 Bundling。前者支持为第三方应用的通知加入快速回复和快捷操作，后者则允许同时发出多条通知的应用进行通知拆分。

当一款应用完美的适配了 Android N，当收到一条消息时就可以直接在下拉通知抽屉甚至是锁屏中直接呼出输入框进行回复，或是选择事先设定好的快速处理操作（标记为已读、转发等）。而当用户同时收到来自不同联系人的消息时，可以点击知卡片上的通知拆分按钮对已经合并的通知进行拆分，拆分后的通知可以像其他的独立通知一样进行回复和处理。

当然，现阶段适配了这两个特性的应用屈指可数，除了 Google 的环聊、Messenger 以及 Gmail 等应用以外，目前仅发现第三方 Telegram 客户端 Plus Messenger 支持以上功能。

面对各种应用的通知推送， Android N取以优先级为核心的通知管理方式，而在 Android N中，通知管理也变得更加简单：只需在需要在相应的通知上左右轻扫便能看见一个设置图标，点击该图标就能在通知上方呼出一个简洁的通知优先级设定界面，在这个界面可以将应用通知设定为“静默显示”、“阻拦所有通知”和“默认”三个等级。

![android n](http://bbs.itheima.com/data/attachment/forum/201605/26/151944cedewdffhgxcdbqr.jpg.thumb.jpg)

如果在"系统界面调谐器 - 其它"中打开了”Show full importance settings”功能，这三个等级又将变为”屏蔽 - 低 - 一般 - 高 - 紧急”5 个，设定的方式也由纵列选项变为左右滑动。这个看似新颖的设计实际上是对现有通知管理操作的一次简化，在 Android 6.0 中需要在两个界面来回跳转才能完成的操作，在Android 7.0只用在一个界面就可以搞定。

同时，Google 也将其对通知优先级的定义从”幕后”搬到了”台前”，在进行完整的五层次优先级设定时 Google 还会提醒不同优先级所对应的通知效果。最后，勿扰模式也在 Android N 中得到了完善，加入了自动规则并允许用户在“请勿打扰”模式下屏蔽静音通知的弹窗甚至是手机的通知指示灯。

## **2.5 发送一个通知**

```java
private int i = 0;
private NotificationManager mNotificationManager;
private static final String GROUP_NAME = "com.heima.notification_type";

//[1]获取一个NotificationManager
mNotificationManager = (NotificationManager) getSystemService(
        Context.NOTIFICATION_SERVICE);
//[2]创建一个Notification
//[2.1]需要给这个Notification对象指定icon,标题,内容,
final Notification builder = new NotificationCompat.Builder(this)
        .setSmallIcon(R.mipmap.ic_notification)
        .setContentTitle(getString(R.string.app_name))
        .setContentText("这是一个消息通知")
        .setAutoCancel(true)
        .setGroup(GROUP_NAME)
        .build();
//[3]发送一个通知 需要指定一个Notification对象和一个id,这个id必须是唯一的
mNotificationManager.notify(i++, builder);
updateNotificationSummary();
```

## **2.6 多个通知放入到一个组内**

```java
//[4]创建一个notification组
String notificationContent = "传智播客" + i;
final NotificationCompat.Builder builder = new NotificationCompat.Builder(this)
        .setSmallIcon(R.mipmap.ic_notification)
        .setStyle(new NotificationCompat.BigTextStyle()
                .setSummaryText(notificationContent))
        .setGroup(GROUP_NAME)
        .setGroupSummary(true);
final Notification notification = builder.build();
//[5]发送这个notification组
mNotificationManager.notify(101, notification);
```
## **2.7 可以回复的通知**

```java
private NotificationManager mNotificationManager;
//[1]获取一个NotificationManager
mNotificationManager = (NotificationManager) getSystemService(
        Context.NOTIFICATION_SERVICE);
//[2]创建remoteInput对象,这个对象指定了这个notification的标题和一个key
String replyLabel = getResources().getString(R.string.app_name);
RemoteInput remoteInput = new RemoteInput.Builder("heima")
        .setLabel(replyLabel)
        .build();
//[3]创建一个Action对象 可以指定用户一个友好的输入提示，可以指定跳转意图,
Intent deleteIntent = new Intent("");
Notification.Action action =
        new Notification.Action.Builder(R.mipmap.ic_launcher,
                "请输入内容", PendingIntent.getActivity(this, 10002, deleteIntent, 0))
                .addRemoteInput(remoteInput)
                .build();

//[3]创建一个Notification对象
Notification notification =
        new Notification.Builder(this)
                .setSmallIcon(R.mipmap.ic_notification)
                .setContentTitle("传智播客")
                .setContentText("消息通知")
                .addAction(action)
                .build();

//[4]发送这个notification
mNotificationManager.notify(1, notification);
```

```java
public class MainActivity extends Activity {
    private int i = 0;
    private NotificationManager mNotificationManager;
    private static final String GROUP_NAME = "com.heima.notification_type";
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    public void sendNotification(View v){
        //[1]获取一个NotificationManager
        mNotificationManager = (NotificationManager) getSystemService(
                Context.NOTIFICATION_SERVICE);
        //[2]创建一个Notification
        //[2.1]需要给这个Notification对象指定icon,标题,内容,
        final Notification builder = new NotificationCompat.Builder(this)
                .setSmallIcon(R.mipmap.ic_notification)
                .setContentTitle(getString(R.string.app_name))
                .setContentText("这是一个消息通知")
                .setAutoCancel(true)
                .setGroup(GROUP_NAME)
                .build();
        //[3]发送一个通知 需要指定一个Notification对象和一个id,这个id必须是唯一的
        mNotificationManager.notify(i++, builder);
    }

    public void sendGroup(View v){

        //[1]获取一个NotificationManager
        mNotificationManager = (NotificationManager) getSystemService(
                Context.NOTIFICATION_SERVICE);
        //[2]创建一个Notification
        //[2.1]需要给这个Notification对象指定icon,标题,内容,
             final Notification builder = new NotificationCompat.Builder(this)
                .setSmallIcon(R.mipmap.ic_notification)
                .setContentTitle(getString(R.string.app_name))
                .setContentText("这是一个消息通知")
                .setAutoCancel(true)
                .setGroup(GROUP_NAME)
                .build();
        //[3]发送一个通知 需要指定一个Notification对象和一个id,这个id必须是唯一的
        mNotificationManager.notify(i++, builder);


        //[4]创建一个notification组
        String notificationContent = "传智播客" + i;
        final NotificationCompat.Builder builder1 = new NotificationCompat.Builder(this)
                .setSmallIcon(R.mipmap.ic_notification)
                .setStyle(new NotificationCompat.BigTextStyle()
                        .setSummaryText(notificationContent))
                .setGroup(GROUP_NAME)
                .setGroupSummary(true);
        final Notification notification = builder1.build();
        //[5]发送这个notification组
        mNotificationManager.notify(101, notification);

    }

    public void send(View v){

        //[1]获取一个NotificationManager
        mNotificationManager = (NotificationManager) getSystemService(
                Context.NOTIFICATION_SERVICE);
        //[2]创建remoteInput对象,这个对象指定了这个notification的标题和一个key
        String replyLabel = getResources().getString(R.string.app_name);
        RemoteInput remoteInput = new RemoteInput.Builder("heima")
                .setLabel(replyLabel)
                .build();
        //[3]创建一个Action对象 可以指定用户一个友好的输入提示，可以指定跳转意图,
        Intent deleteIntent = new Intent(this,MainActivity.class);
        Notification.Action action =
                new Notification.Action.Builder(R.mipmap.ic_launcher,
                        "请输入内容", PendingIntent.getActivity(this, 10002, deleteIntent, 0))
                        .addRemoteInput(remoteInput)
                        .build();

        //[3]创建一个Notification对象
        Notification notification =
                new Notification.Builder(this)
                        .setSmallIcon(R.mipmap.ic_notification)
                        .setContentTitle("传智播客")
                        .setContentText("消息通知")
                        .addAction(action)
                        .build();

        //[4]发送这个notification
        mNotificationManager.notify(1, notification);

    }
}
```

# **3. Android N 新特性持续改进**

## **3.1 改进的Doze休眠机制**

在Android 6.0中，谷歌带来了全新的休眠机制Doze。据官方表示开启Doze后，手机的续航可以延长数小时。实际测试中虽然没有Google官方说的那般优秀，但依旧对续航起到了一定的改善作用。Doze休眠机制是当设备处于空闲状态时，通过推迟应用的 CPU 和网络活动以实现省电目的的系统模式。

![Android n](http://bbs.itheima.com/data/attachment/forum/201605/26/152217d5dws4idljxbl569.jpg.thumb.jpg)

在 Android N 中，Doze休眠机制又前进了一步。只要屏幕关闭了一段时间，且设备未插入电源，Doze休眠机制开启，系统会尝试通过限制网络访问以及CPU密集的服务来保存电量，这意味着用户即使将设备放入口袋里也可以省电。

具体点来说，就是屏幕关闭片刻后，设备在使用电池时，Doze休眠机制将限制网络访问，同时延迟作业和同步。在短暂的维护时间范围后，其允许应用访问网络，并执行延迟的作业/同步。打开屏幕或将设备插入电源会使设备退出Doze休眠机制。当设备再次处于静止状态时，屏幕关闭且使用电池一段时间，Doze休眠机制针对PowerManager.WakeLock,AlarmManager警报和 GPS/Wi-Fi 扫描应用完整 CPU 和网络限制。

![Android n](http://bbs.itheima.com/data/attachment/forum/201605/26/152438dzzzgdggmoj3ujfq.jpg.thumb.jpg)

值得一提的是，新的Doze Mode还与Project Svelte强强联合，Android N将会在待机方面创造奇迹。


## **3.2 Project Svelte后台优化**

Project Svelte 是AndroidKitKat时期推出的一种有效改善运存使用的算法，它也在持续改善，以最大程度减少生态系统中一系列 Android 设备中系统和应用使用的 RAM。在 Android N 中，Project Svelte 注重优化在后台中运行应用的方式。后台处理是大多数应用的一个重要部分。处理得当能实现即时、快速的体验。不得当则会毫无必要地消耗 RAM（和电池），也影响其他应用的系统性能。   

自 Android 5.0以来，JobScheduler 已成为执行后台工作的首选。应用在安排作业的同时，允许系统基于内存、电源和连接情况进行优化。JobScheduler 可实现控制和简洁性。另一个是 GCMNetworkManager（Google Play 服务的一部分），其在旧版 Android 中提供类似的作业安排和兼容性。Google在继续扩展 JobScheduler 和 GCMNetworkManager，以符合多个用例。在 Android N 中，可以基于内容提供程序中的更改安排后台工作。弃用了一些会降低系统性能的较旧模式。

在 Android N 中，删除了三个常用隐式广播： —CONNECTIVITY_ACTION、ACTION_NEW_PICTURE 和 ACTION_NEW_VIDEO 。这些广播可能会一次唤醒多个应用的后台进程，同时会耗尽内存和电池。

## **3.3 流量节省程序(Data Saver)**

在移动设备的整个生命周期，蜂窝数据计划的成本通常会超出设备本身的成本。对于许多用户而言，蜂窝数据是他们想要节省的昂贵资源。Android N中提供了一个全局的流量控制机制：Data Saver 模式。这项新的系统服务有助于减少应用使用的蜂窝数据，无论是在漫游，账单周期即将结束，还是使用少量的预付费数据包。有效防止应用程序在后台恶意偷跑移动流量。此功能默认关闭，一旦开启后除了GMS(Google Mobile Service，谷歌移动服务)之外，其他应用都默认不允许在后台使用超过前台所消耗的移动流量。

Data Saver 让用户可以控制应用使用蜂窝数据的方式，同时让开发者打开 Data Saver 时可以提供更多有效的服务。对开发者而言，在Android N系统中要主动检查用户是否开启了流量节省程序，并注意节约后台时的数据流量消耗。

用户在 Settings 中启用 Data Saver 且设备位于按流量计费的网络上时，系统屏蔽后台数据使用，同时指示应用在前台尽可能使用较少的数据。例如通过限制用于流媒体服务的比特率、降低图片质量、延迟最佳的预缓冲等方法来实现。将特定应用加入白名单以允许后台按流量的数据使用，即使在打开 Data Saver 时也是如此。

![android n](http://bbs.itheima.com/data/attachment/forum/201605/26/152620mco6o33cz3322oju.jpg.thumb.jpg)

Android N 继承了 ConnectivityManager，以便为应用检索用户的 Data Saver 首选项并监控首选项变更提供一种方式。所有应用均应检查用户是否已启用 Data Saver 并努力限制前台和后台数据的使用。

## **3.4 作用域目录访问**

在Android N 中，应用可以使用新的 API 请求访问特定的外部存储目录，包括可移动媒体上的目录，如 SD 卡。新 API 大大简化了应用访问标准外部存储目录的方式，如 Pictures 目录。应用可以使用这些 API（而不是使用 READ_EXTERNAL_STORAGE），其授予所有存储目录的访问权限或存储访问框架，从而让用户可以导航到目录。

此外，新的 API 简化了用户向应用授予外部存储访问权限的步骤。当您使用新的 API 时，系统使用一个简单的权限 UI。

## **3.5 访问外部存储目录**

使用 StorageManager 类获取适当的 StorageVolume 实例。然后，通过调用该实例的 StorageVolume.createAccessIntent() 方法创建一个 Intent。使用此 Intent 访问外部存储目录。 若要获取所有可用卷的列表，包括可移动介质卷，请使用 StorageManager.getVolumesList()。

以下代码段展示如何在主要共享存储中打开 Pictures 目录：

```java
StorageManager sm = (StorageManager)getSystemService(Context.STORAGE_SERVICE);
StorageVolume volume = sm.getPrimaryVolume();
Intent intent = volume.createAccessIntent(Environment.DIRECTORY_PICTURES);
startActivityForResult(intent, request_code);

```
## **3.6 访问可移动介质上的目录**
若要使用作用域目录访问来访问可移动介质上的目录，首先要添加一个用于侦听 MEDIA_MOUNTED 通知的 BroadcastReceiver，例如

```xml
<receiver
    android:name=".MediaMountedReceiver"
    android:enabled="true"
    android:exported="true" >
    <intent-filter>
        <action android:name="android.intent.action.MEDIA_MOUNTED" />
        <data android:scheme="file" />
    </intent-filter>
</receiver>
```

当用户装载可移动介质时，如 SD 卡，系统将发送一则 MEDIA_MOUNTED 通知。此通知在 Intent 数据中提供一个 StorageVolume 对象，您可用它访问可移动介质上的目录。 以下示例访问可移动介质上的 Pictures 目录：

```java
// BroadcastReceiver has already cached the MEDIA_MOUNTED
// notification Intent in mediaMountedIntent
StorageVolume volume = (StorageVolume)
    mediaMountedIntent.getParcelableExtra(StorageVolume.EXTRA_STORAGE_VOLUME);
volume.createAccessIntent(Environment.DIRECTORY_PICTURES);
startActivityForResult(intent, request_code);
```
## **3.7 最佳做法**
请尽可能保留外部目录访问 URI，这样即不必重复要求用户授予访问权限。 在用户授予访问权限后，使用目录访问 URI 调用 getContentResolver().takePersistableUriPermssion()。 系统将保留此 URI，后续的访问请求将返回 RESULT_OK，且不会向用户显示确认 UI。

如果用户拒绝授予外部目录访问权限，请勿立即再次请求访问权限。 一再不停地请求访问权限会导致非常差的用户体验。


## **3.8 快速设置栏API**

“快速设置”通常用于直接从通知栏显示关键设置和操作，非常简单。在 Android N 中，已扩展“快速设置”的范围，使其更加有用更方便。Google为额外的“快速设置”Tile添加了更多空间，用户可以通过向左或向右滑动跨分页的显示区域访问它们。还让用户可以控制显示哪些“快速设置”Tile以及显示的位置 — 用户可以通过拖放Tile来添加或移动Tile。

![android n](http://bbs.itheima.com/data/attachment/forum/201605/26/152814qxmk0m55go575mt0.jpg.thumb.jpg)

对于开发者，Android N 还添加了一个新的 API，从而可以定义自己的“快速设置”Tile，可以轻松访问应用中的关键控件和操作。对于急需或频繁使用的控件和操作，保留“快速设置”Tile，且不应将其用作启动应用的快捷方式。

## **3.9 Profile-guided  JIT (Just in Time)/ AOT编译**         

Android N引入了一种包含编译、解释和JIT（Just In Time）的混合运行时，以便在安装时间、内存占用、电池消耗和性能之间获得最好的折衷。

ART取代了Dalvik，但保持了字节码级的兼容， ART的主要特征之一就是安装时对应用的AOT编译。这种方式的主要优点就是优化产生的本地代码性能更好，执行起来需要更少的电量。在Lollipop和Marshmallow（Android 6.0）中，大的应用需要数分钟才能安装完。Android中N，添加了代码分析JIT编译器技术，提高了Android应用程序的性能。应用在安装时不做编译，而是解释字节码，所以可以快速启动。JIT编译器补充ART当前的时间提前（AOT）编译器，有助于提高运行时性能，节省存储空间，加快应用程序更新和系统更新。

ART中有一种新的、更快的解释器，通过一种新的JIT完成，但这种JIT的信息不是持久化的。取而代之的是代码在执行期间被分析，分析结果保存起来。当设备空转和充电的时候，ART会执行针对“热代码”进行的基于分析的编译，其他代码不做编译。为了得到更优的代码，ART采用了几种技巧包括深度内联。

这种混合使用AOT、解释、JIT的策略的全部优点是即使是大应用，安装时间也能缩短到几秒；系统升级能更快地安装，因为不再需要优化这一步；应用的内存占用更小，有些情况下可以降低50%；改善了性能；更低的电池消耗。

Profile-guided编译管理让ART管理，根据其实际使用每个应用程序的AOT / JIT编译，以及在设备上的条件。ART保持了每个应用程序的热方法配置文件，可以预编译并缓存以获得最佳性能的方法。离开应用程序的其他部分未编译，直到它们被实际使用。

除了改善应用程序的关键部位性能，Profile-guided编译有助于减少应用程序的整体内存占用，包括相关的二进制文件。此功能在低内存设备尤其重要。

## **3.10 Android框架中提供部分ICU4J API支持**

ICU4J(International Components for Unicode)是由IBM维护，基于IBM公共许可证分发的免费开源Unicode工具库，开发者可以使用ICU4J根据各地的风俗和语言习惯，实现对数字、货币、时间、日期、和消息的格式化、解析，对字符串进行大小写转换、整理、搜索和排序等功能。

但由于Android N内置了部分ICU4J API，如果Android应用只使用了这部分的API，那今后就可以不再集成庞大的高达10MB左右的ICU4J库了。开发者可以在Google Play上针对使用Android N的用户提供不含ICU4J的轻量安装包，而针对更早版本系统提供包含ICU4J的完整安装包。

![Android n](http://bbs.itheima.com/data/attachment/forum/201605/26/153401n8jpyi5fvu8ovmpo.jpg.thumb.jpg)

# **4. 开始支持Java 8**
从Android N开始，开发者可以使用Java 8来编写应用程序，目前Android N对于Java 8的支持并不全面，但这依然是一个重量级的更新。目前支持以下内容：

- 默认和静态接口方法：使开发者可以修改接口而不破坏原来实现类的结构
- Lambda表达式：不仅让代码变得更简单、更可读、最重要的是代码量也随之减少很多
- 重复注解：允许在同一申明类型(类，属性，或方法)的多次使用同一个注解，提高可读性

反射及语言相关的API

公用工具API

为了使用Java8同时还需要引入Jack编译工具链，与传统编译工具链相比的优势在于全部开源，编译速度更快。Jack编译工具链完整地包含了重打包，压缩，混淆，MultiDex工具，使用Jack编译工具链之后将不需要再依赖类似ProGuard和Jarjar之类的单独组件

Jack编译工具链向下支持到Android 2.3应用的编译。同时Jack也是一套面向未来的编译工具链，未来预计还会支持Java 9，以及Java X

Jack 最大最大的优点，你不用再操心65K方法限制的问题了！Jack在Compile的时候就已经解决了！『65k方法限制』将成为过去式中存在的名词了。

其他的一些变化：

- 速度（每次都会提升速度）
- Library File的后缀（变成了.jack

开发者也可以继续使用Java7开发针对Android N的应用程序，但是编译时依然要使用JDK8。Jack编译工具链虽然非常诱人，但是对于开发者来说依然要做好充分的准备和测试工作

# **5. Direct Boot (直接启动)**

用户在开机但是还未解锁的情况下，很多App是无法启动的，这会导致一些问题，比如...你设置的第三方闹钟可能没响，你的微信可能收不到通知...Android N下可以申请在开机未解锁情况下直接启动

# **6. VR支持**
## **6.1 图片类**
```
com.google.vr.sdk.widgets.pano.VrPanoramaView
```
布局文件

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <com.google.vr.sdk.widgets.pano.VrPanoramaView
        android:id="@+id/vr_img"
        android:layout_width="match_parent"
        android:layout_height="250dp" />
</LinearLayout>
```

实现代码
```java
public class MainActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        try {
            setContentView(R.layout.vr_layout);
            VrPanoramaView vr_img = (VrPanoramaView) findViewById(R.id.vr_img);
            InputStream in = getAssets().open("andes.jpg");

            Bitmap bm = BitmapFactory.decodeStream(in);

            VrPanoramaView.Options options = new VrPanoramaView.Options();
            options.inputType = VrPanoramaView.Options.TYPE_STEREO_OVER_UNDER;
            vr_img.loadImageFromBitmap(bm, options);

            vr_img.setEventListener(new VrPanoramaEventListener() {
                @Override
                public void onLoadSuccess() {
                    System.out.println("加载成功");
                }

                @Override
                public void onLoadError(String errorMessage) {
                    System.out.println("加载失败");
                }
            });
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

## **6.2 视频类**
```
com.google.vr.sdk.widgets.pano.VrVideoView
```
布局文件

```java
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <com.google.vr.sdk.widgets.video.VrVideoView
        android:id="@+id/vr_video"
        android:layout_width="match_parent"
        android:layout_height="250dp" />
</LinearLayout>
```

实现代码

```java
public class MainActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.vr_layout);
        try {
            final VrVideoView vr_video = (VrVideoView) findViewById(R.id.vr_video);

            VrVideoView.Options options = new VrVideoView.Options();
            options.inputType = VrVideoView.Options.TYPE_STEREO_OVER_UNDER;

            vr_video.loadVideoFromAsset("congo.mp4", options);

            vr_video.setEventListener(new VrVideoEventListener(){
                @Override
                public void onLoadSuccess() {
                    System.out.println("加载成功");
                }

                @Override
                public void onLoadError(String errorMessage) {
                    super.onLoadError(errorMessage);
                }


                @Override
                public void onCompletion() {
                    vr_video.seekTo(0);
                }

                @Override
                public void onClick() {

                }

                @Override
                public void onNewFrame() {

                }
            });

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
Android N已经发布，但考虑到国内各种深度定制化的Android系统还要对其作出硬件适配、软件的捆绑整合，相信距离Android N的大面积普及还需要一段时间。

在Android N尚未推送之际，黑马的惊喜福利先N一步从天而降！赶在Android N系统推送之前，黑马程序员的优秀讲师已经将《An droid N新特性课程》视频已经录制完毕！黑马程序员的目标是时刻与Google保持同步更新，Google发布了Android N新版本，黑马程序员第一时间投入研发新课程，并第一时间推出，目的就是让黑马的学员一直跑在行业的最前沿，引领整个行业！

# **7. 官方文档汉化**
Android N 预览版官网几乎所有文档都已汉化，别再给自己找理由了，你可以学的有：

- 行为变更
- 后台优化
- 语言和区域设置：
- API 概览  
- 多窗口支持  
- 通知  
- 直接启动  
- Data Saver
- 网络安全配置
- ICU4J Android 框架 API
- 作用域目录访问
- Android for Work 更新
- 画中画
- TV 录制  

https://developer.android.com/preview/index.html，需要翻墙

![android n](http://ww2.sinaimg.cn/mw690/8f1fe6aajw1f4xfq4infkj20uc0o1tfo.jpg)

# **8. Android Ｎ 视频+资料**
资料：http://pan.baidu.com/s/1bpC1HGN
视频：http://pan.baidu.com/s/1c2wo9uK#list/path=%2F

# **9. 参考**
http://bbs.itheima.com/forum.php?mod=viewthread&tid=304677
