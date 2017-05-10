

# **1. 内容摘要**
- ContentProvider 基本概念
- ContentProvider 的基本使用
- 操作系统短信
- 操作系统联系人
- 内容观察者ContentObserver
- 案例-短信窃听器
- 使用ContentObserver 监听自定义ContentProvider 的变化

# **2. ContentProvider 基本概念**
内容提供者ContentProvider，是Android 的四大组件之一。内容提供者是应用程序之间共享数据的接口。

Android 系统将这种机制应用到方方面面，比如：联系人（通讯录应用程序）Provider 专为不同应用程序提供联系人数据；短信（短信应用程序）Provider 专为不同应用程序提供系统短信信息。

当应用继承ContentProvider 类，并重写该类用于提供数据和存储数据的方法，就可以向其他应用共享其数据。虽然使用其他方法也可以对外共享数据，但数据访问方式会因数据存储的方式而不同，如：采用文件方式对外共享数据，需要进行文件操作读写数据；采用SharedPreferences 共享数据，需要使用SharedPreferences API 读写数据。而使用ContentProvider 共享数据的好处是统一了数据访问方式。

官方对ContentProvider 的解释如下：
>Content providers manage access to a structured set of data. They encapsulate the data, and provide mechanisms for defining data security. Content providers are the standard interface that connects data in one process with code running in another process.

内容提供者管理了对结构化数据（最常见的就是数据库中数据）的访问操作。内容提供者封装了这些数据并且提供了一种安全的访问机制。内容提供者是不同进程之间交互数据（数据库数据）的标准方式。

用一张简易图演示内容提供者的工作原理，如下图所示

![ContentProvider](http://img.blog.csdn.net/20161021114145924)

该图中假设手机助手App 要获取系统的短信，系统的短信是存储在数据库中的，当然该数据库只能由系统短信App 内部代码直接访问。手机助手App 直接访问短信数据库是行不通的，这时候就可以借助内容提供者，系统短信App 已经写好了内容提供者，该内容提供者对外提供了短信数据。因此手机助手App直接去访问系统短信的内容提供者即可间接实现对短信数据库的访问

# **3. ContentProvider 基本使用**
为了演示ContentProvider 的基本用法，在该节中我们将创建两个Android 工程：01-内容提供者A（简称AppA）和内容访问者B（简称AppB）。在AppA 中创建数据库user.db，该db 中有两张表，t_woman和t_man。AppA 通过内容提供者对外提供数据库中的数据。AppB 作为一个内容访问者，通过内容解析者（ContentResolver）实现对AppA 中的数据进行增删改查功能。

## **3.1 在AppA 中创建MySQLiteOpenHelper 类**
在该类中实现数据库和数据表的创建业务逻辑

```java
package com.example.contentProviderA;

    import android.content.Context;
    import android.database.sqlite.SQLiteDatabase;
    import android.database.sqlite.SQLiteDatabase.CursorFactory;
    import android.database.sqlite.SQLiteOpenHelper;
    //创建数据库user.db 同时初始化表t_woman 和t_man
    public class MySQLiteOpenHelper extends SQLiteOpenHelper {

        //数据库名称
        private static final String DB_NAME = "user.db";
        //数据库版本号
        private static final int VERSION = 1;

        private MySQLiteOpenHelper(Context context, String name, CursorFactory factory,
                                   int version) {
            super(context, name, factory, version);
        }

        public MySQLiteOpenHelper(Context context){
            this(context, DB_NAME, null, VERSION);
        }

        @Override
        public void onCreate(SQLiteDatabase db) {
            String sql_woman="create table t_woman(id integer primary key,
            c_name varchar(10),c_phone varchar(20))";
            String sql_man = "create table t_man(id integer primary key,
            c_name varchar(10),c_phone varchar(20))";
            db.execSQL(sql_woman);
            db.execSQL(sql_man);
        }

        @Override
        public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {

        }

    }
```
## **3.2 使用测试类初始化user.db**
为了让MySQLiteOpenHelper.java 中onCreate 方法被调用，我们可以采用一个测试类来实现，当然使用其他方法也是可行的。这里使用测试类初始化AppA 的数据库也是为了复习Android Junit Test的使用

```java
package com.example.contentProviderA;

    import android.database.sqlite.SQLiteDatabase;
    import android.test.AndroidTestCase;

    //定义测试类，用于初始化user.db
    public class MyTest extends AndroidTestCase {

        public void test(){
            /**
             * 在AndroidTestCase 类中可以直接使用getContext()方法获取到Context 对象
             */
            MySQLiteOpenHelper sqLiteOpenHelper = new MySQLiteOpenHelper(getContext());
            SQLiteDatabase writableDatabase = sqLiteOpenHelper.getWritableDatabase();
            writableDatabase.close();
        }
    }
```
在AndroidManifest.xml 中添加测试指令集和测试库。在application 节点外添加指令集

```xml
<instrumentation
    android:name="android.test.InstrumentationTestRunner"
    android:targetPackage="com.example.contentProviderA" />
```
在application 节点中添加测试库

```xml
<uses-library android:name="android.test.runner" />
```
万事俱备后一定记得运行一下该测试方法，看见如下绿条和数据库文件才行哦
![ContentProvider](http://img.blog.csdn.net/20161021114941420)

数据库初始化完成

![ContentProvider](http://img.blog.csdn.net/20161021115029794)

## **3.3 在AppA 中创建内容提供者**
我们已经将user.db 创建好了，接下来我们创建一个内容提供者将user.db 中的数据提供出去

```java
public class MyContentProvider extends ContentProvider {

        private SQLiteDatabase database;
        /**
         * 创建一个Uri 匹配器，用于区分用户使用的uri 种类，比如区分用户到底想操作哪张表，
         * 以及哪个表中的哪条数据
         */
        private static final UriMatcher sURIMatcher =
                new UriMatcher(UriMatcher.NO_MATCH);

        private static final int T_WOMAN = 1;
        private static final int T_MAN = 2;
        private static final int T_WOMAN_ID = 3;
        /**
         * 添加匹配项
         */
        static{
            String authority = "com.itheima.provider";
            /**
             * 参数1：内容提供者的主机名，类似于我们隐式意图中intent-filter 中action，
             * 用于区分内容提供者，
             * 毕竟对于一个Android 系统来讲，可能内容提供者不止一个
             * 参数2：表名或者表名/#，其中#用于指定表中数据的id
             * 参数3：如果匹配成功了的返回值
             */
            sURIMatcher.addURI(authority, "t_woman", 1);
            sURIMatcher.addURI(authority, "t_man", 2);
            sURIMatcher.addURI(authority, "t_woman/#", 3);
        }


        /**
         * 执行删除方法
         */
        @Override
        public int delete(Uri uri, String selection, String[] selectionArgs) {
            /**
             * 获取用户传递进来的uri 的匹配码，如果匹配不成功在，则返回UriMatcher.NO_MATCH
             */
            int match = sURIMatcher.match(uri);
            switch (match) {
                case T_WOMAN:
                    Log.d("tag", "匹配t_woman 表");
                    return database.delete("t_woman", selection, selectionArgs);
                case T_MAN:
                    Log.d("tag", "匹配t_man 表");
                    return database.delete("t_man", selection, selectionArgs);
                case UriMatcher.NO_MATCH:
                    Log.d("tag", "没有匹配项");
                    return 0;
                default:

                    break;
            }
            return 0;
        }

        @Override
        public String getType(Uri uri) {
            return null;
        }
        /**
         * 用于处理ContentResolver 发送的insert 请求，一般在该方法中执行数据库的插入操作
         * @param uri
         * @param values
         */
        @Override
        public Uri insert(Uri uri, ContentValues values) {
            int match = sURIMatcher.match(uri);
            switch (match) {
                case T_WOMAN:
                    database.insert("t_woman", null, values);
                    break;
                case T_MAN:
                    database.insert("t_man", null, values);
                    break;
                case UriMatcher.NO_MATCH:
                    Log.d("tag", "匹配不正确");
                    break;
                default:
                    break;
            }
            return uri;
        }
        /**
         * 在该内容提供者注册以后，当该应用第一次运行起来的时候该方法会被回调
         * 用来做一些初始化工作，比如初始化数据库对象
         * 如果初始化成功则返回true，否则返回false。
         */
        @Override
        public boolean onCreate() {
            /**
             * 在ContentProvider 中可以直接通过getContext()方法获取Context 对象
             */
            MySQLiteOpenHelper sqLiteOpenHelper =
                    new MySQLiteOpenHelper(getContext());
            //获取一个SQLiteDatabase 对象，并声明为成员变量，方便其他方法中使用
            database = sqLiteOpenHelper.getWritableDatabase();

            return true;
        }
        /**
         * 执行查询方法
         */
        @Override
        public Cursor query(Uri uri, String[] projection, String selection,
                            String[] selectionArgs, String sortOrder) {
            int match = sURIMatcher.match(uri);
            switch (match) {
                case T_MAN:
                    return database.query("t_man", projection, selection,
                            selectionArgs, null, null, null);
                case T_WOMAN:
                    return database.query("t_woman", projection,
                            selection, selectionArgs, null, null, null);
                case T_WOMAN_ID:
                    //ContentUris 是解析Uri 的一个工具类，可以将Uri 中的id 给解析出来
                    long id = ContentUris.parseId(uri);
                    return database.query("t_woman", null, "id=?",
                            new String[]{id+""}, null, null, null);
                case UriMatcher.NO_MATCH:
                    Log.d("tag", "匹配失败");
                    break;
                default:
                    break;
            }
            return null;
        }
        /**
         * 执行修改方法
         */
        @Override
        public int update(Uri uri, ContentValues values, String selection,
                          String[] selectionArgs) {
            int match = sURIMatcher.match(uri);
            switch (match) {
                case T_MAN:
                    return database.update("t_man", values, selection, selectionArgs);
                case T_WOMAN:
                    return database.update("t_woman", values, selection, selectionArgs);
                case UriMatcher.NO_MATCH:
                    Log.d("tag", "匹配失败");
                    break;
                default:
                    break;
            }

            return 0;
        }
    }
```
## 3.4 在AndroidManifest.xml 中注册ContentProvider

```xml
<provider 
    android:name="com.example.contentProviderA.MyContentProvider"
    android:exported="true"
    android:authorities="com.itheima.provider"/>
```
注意：exported 属性必须设置为true 才可以被其他应用访问到该内容提供者

## **3.5 创建工程02-内容访问者B**
在AppB 中直接使用MainActivity 类，布局文件如下

![ContentProvider](http://img.blog.csdn.net/20161021120644176)
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:onClick="insert1"
        android:text="给woman 表插入数据"/>
    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:onClick="insert2"
        android:text="给man 表插入数据"/>
    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:onClick="delete1"
        android:text="删除woman 表的数据"/>
    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:onClick="update1"
        android:text="修改woman 表的数据"/>
    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:onClick="query1"
        android:text="查询woman 表的所有数据"/>
    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:onClick="query2"
        android:text="查询woman 表中的单条数据"/>
</LinearLayout>
```
MainActivity.java 代码实现

```java
/**
     * 点击界面不同的按钮实现对AppA 中不同的增删改查操作
     */
    public class MainActivity extends Activity {

        private ContentResolver contentResolver;

        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            //获取内容解析者（ContentResolver）
            contentResolver = getContentResolver();
        }

        /**
         * 往t_woman 表中插入一条数据
         */
        public void insert1(View view) {
            /**
             * 访问内容提供者的uri 的标准写法：
             * 1.必须以content：//开头
             * 2. 主机名必须跟内容提供者的authorities 属性一致
             * 3. 主机名后面跟要操作的表名
             */
            Uri url = Uri.parse("content://com.itheima.provider/t_woman");
            ContentValues values = new ContentValues();
            values.put("c_name", "lili");
            values.put("c_phone", "5554");
            //插入一条数据
            contentResolver.insert(url, values);
            //再插入一条新数据
            values.clear();
            values.put("c_name", "lucy");
            values.put("c_phone", "5556");
            contentResolver.insert(url, values);
            //再插入一条数据
            values.clear();
            values.put("c_name", "fengjie");
            values.put("c_phone", "5558");
            contentResolver.insert(url, values);

        }

        /**
         * 往t_man 表中插入数据
         */
        public void insert2(View view) {
            Uri url = Uri.parse("content://com.itheima.provider/t_woman");
            ContentValues values = new ContentValues();
            values.put("c_name", "zhangsan");
            values.put("c_phone", "6500");
            contentResolver.insert(url, values);
            //再插入一条新数据
            values.clear();
            values.put("c_name", "lisi");
            values.put("c_phone", "6600");
            contentResolver.insert(url, values);
        }

        /**
         * 删除t_woman 表中姓名为lili 的数据
         */
        public void delete1(View view) {
            Uri url = Uri.parse("content://com.itheima.provider/t_woman");
            String where = "c_name=?";
            String[] selectionArgs = new String[]{"lili"};
            contentResolver.delete(url, where, selectionArgs);
        }

        /**
         * 删除t_man 表中姓名为zhangsan 的数据
         */
        public void delete2(View view) {
            Uri url = Uri.parse("content://com.itheima.provider/t_man");
            String where = "c_name=?";
            String[] selectionArgs = new String[]{"zhangsan"};
            contentResolver.delete(url, where, selectionArgs);
        }

        /**
         * 修改t_woman 表中姓名为lucy 的电话为110
         */
        public void update1(View view) {
            Uri url = Uri.parse("content://com.itheima.provider/t_woman");
            String where = "c_name=?";
            String[] selectionArgs = new String[]{"lucy"};
            ContentValues values = new ContentValues();
            values.put("c_phone", "110");
            contentResolver.update(url, values, where, selectionArgs);

        }

        /**
         * 查询t_woman 表中的所有数据
         */
        public void query1(View view) {

            Uri uri = Uri.parse("content://com.itheima.provider/t_woman");

            /**
             * 参数2：要查询的字段
             * 参数3：where 表达式，null 代表查询所有
             * 参数4：用于替换where 表达式中？号的真实值
             * 参数5：排序规则
             */
            Cursor cursor = contentResolver.query(uri,
                    new String[]{"id", "c_name", "c_phone"}, null, null, null);
            while (cursor.moveToNext()) {
                int id = cursor.getInt(0);
                String name = cursor.getString(1);
                String phone = cursor.getString(2);
                Log.d("tag", "id=" + id + ",name=" + name + ",phone=" + phone);
            }
            cursor.close();

        }

        /**
         * 查询t_woman 表中id 为2 的数据
         */
        public void query2(View view) {
            /**
             * 该方法的需求的是查询id 为2 的数据，因此只需要在uri 的最后面添加上/id 即可。
             */
            Uri uri = Uri.parse("content://com.itheima.provider/t_woman/2");
            Cursor cursor = contentResolver.query(uri,
                    new String[]{"id", "c_name", "c_phone"}, null, null, null);
            while (cursor.moveToNext()) {
                int id = cursor.getInt(0);
                String name = cursor.getString(1);
                String phone = cursor.getString(2);
                Log.d("tag", "id=" + id + ",name=" + name + ",phone=" + phone);
            }
            cursor.close();
        }
    }
```
## **3.6 内容提供者Uri 的书写规范**
![ContentProvider](http://img.blog.csdn.net/20161021121139574)

- schema，用来说明一个ContentProvider 控制这些数据。"content://"
- Authority主机名或授权：它定义了是哪个ContentProvider 提供这些数据。
- path：路径，URI 下的某一个Item。
- ID：通常定义Uri 时使用”#”号占位符代替, 使用时替换成对应的数字
- content://com.itheima.provider/person/#：#表示数据id（#代表任意数字）
- content://com.itheima.provider/person/\*：*来匹配任意文本。

# **4. 操作系统短信**
使用内容提供者操作系统短信和操作系统联系人是我们企业开发中“经常”遇到的需求，而自定义内容提供者对外提供数据反而使用的场景并不多，除非我们开发的短信或者联系人应用
## 4.1 准备知识
打开Android 系统源码，查看packages\providers\路径下的工程，这些就是Android 系统中的内容提供者，其中TelephonyProvider 就是短信的内容提供者文件
![ContentProvider](http://img.blog.csdn.net/20161021121808634)

打开TelephonyProvider 下的src 文件，查看java 文件，其中的SmsProvider.java 即短信息内容提供者逻辑代码。UriMatcher 一般在静态代码块中进行初始化操作，查找静态代码块，找到的逻辑代码如下

```java
private static final UriMatcher sURLMatcher = new UriMatcher(UriMatcher.NO_MATCH);

    static {
        URLMatcher.addURI("sms", null, SMS_ALL);
        URLMatcher.addURI("sms", "#", SMS_ALL_ID);
        URLMatcher.addURI("sms", "inbox", SMS_INBOX);
        URLMatcher.addURI("sms", "inbox/#", SMS_INBOX_ID);
        URLMatcher.addURI("sms", "sent", SMS_SENT);
        sURLMatcher.addURI("sms", "sent/#", SMS_SENT_ID);
        sURLMatcher.addURI("sms", "draft", SMS_DRAFT);
        sURLMatcher.addURI("sms", "draft/#", SMS_DRAFT_ID);
        sURLMatcher.addURI("sms", "outbox", SMS_OUTBOX);
        sURLMatcher.addURI("sms", "outbox/#", SMS_OUTBOX_ID);
        sURLMatcher.addURI("sms", "undelivered", SMS_UNDELIVERED);
        sURLMatcher.addURI("sms", "failed", SMS_FAILED);
        sURLMatcher.addURI("sms", "failed/#", SMS_FAILED_ID);
        sURLMatcher.addURI("sms", "queued", SMS_QUEUED);
        sURLMatcher.addURI("sms", "conversations", SMS_CONVERSATIONS);
        sURLMatcher.addURI("sms", "conversations/*", SMS_CONVERSATIONS_ID);
        sURLMatcher.addURI("sms", "raw", SMS_RAW_MESSAGE);
        sURLMatcher.addURI("sms", "attachments", SMS_ATTACHMENT);
        sURLMatcher.addURI("sms", "attachments/#", SMS_ATTACHMENT_ID);
        sURLMatcher.addURI("sms", "threadID", SMS_NEW_THREAD_ID);
        sURLMatcher.addURI("sms", "threadID/*", SMS_QUERY_THREAD_ID);
        sURLMatcher.addURI("sms", "status/#", SMS_STATUS_ID);
        sURLMatcher.addURI("sms", "sr_pending", SMS_STATUS_PENDING);
        sURLMatcher.addURI("sms", "icc", SMS_ALL_ICC);
        sURLMatcher.addURI("sms", "icc/#", SMS_ICC);
        //we keep these for not breaking old applications
        sURLMatcher.addURI("sms", "sim", SMS_ALL_ICC);
        sURLMatcher.addURI("sms", "sim/#", SMS_ICC);
    }
```
通过查找系统源码，可以确定短信息内容提供者的Uri 应该为:”content://sms”。查看Android 模拟器下的/data/data/com.android.providers.telephony/databases/目录，查看其mmssms.db文件。

![ContentProvider](http://img.blog.csdn.net/20161021122102336)

打开数据库，其中sms 表存储的就是短信的数据，其存储格式如下
![ContentProvider](http://img.blog.csdn.net/20161021122146760)

其中，address 存储的是联系人号码，date 是发送日期，type 对应短信的类型（发送是1/接收是2）,body是短信的主体内容

由于读取和插入系统短信数据库都涉及到可能侵犯用户隐私，因此创建的工程必须添加如下权限
```xml
<uses-permission android:name="android.permission.READ_SMS"/>
<uses-permission android:name="android.permission.WRITE_SMS"/>
```
## 4.2 查询系统短信
创建Android 工程03-操作系统短信。布局文件十分简单，只有两个Button，点击不同的Button 分别触发短信的读取和短信的插入功能。因此布局文件不再给出。

```java
//读取系统短信
    public void readSms(View view){
        Uri uri=Uri.parse("content://sms");
        //contentResolver 在MainActivity 的onCreate 方法中获取
        Cursor cursor = contentResolver.query(uri,
                new String[]{"address","date","body","type"}, null, null, null);

        while(cursor.moveToNext()){
            String address = cursor.getString(0);
            String date = cursor.getString(1);
            String body = cursor.getString(2);
            String type = cursor.getString(3);

            Log.d("tag", "address="+address+",body="+body+",type="+type+",
                    date="+new Date(Long.valueOf(date)));
        }
    }
```
![内容提供者](http://img.blog.csdn.net/20161021122448590)

# 5. 内容提供者
* 应用的数据库是不允许其他应用访问的
* 内容提供者的作用就是让别的应用访问到你的数据库
* 自定义内容提供者，继承ContentProvider类，重写增删改查方法，在方法中写增删改查数据库的代码，举例增方法

```
@Override
		public Uri insert(Uri uri, ContentValues values) {
			db.insert("person", null, values);
			return uri;
		}
```

* 在清单文件中定义内容提供者的标签，注意必须要有authorities属性，这是内容提供者的主机名，功能类似地址
```
<provider android:name="com.itheima.contentprovider.PersonProvider"
android:authorities="com.itheima.person"
android:exported="true">
</provider>
```

* 创建一个其他应用，访问自定义的内容提供者，实现对数据库的插入操作

```
public void click(View v){
  //得到内容分解器对象
  ContentResolver cr = getContentResolver();
  ContentValues cv = new ContentValues();
  cv.put("name", "小方");
  cv.put("phone", 138856);
  cv.put("money", 3000);
  //url:内容提供者的主机名
  cr.insert(Uri.parse("content://com.itheima.person"), cv);
}
```

# **6. UriMatcher**
* 用于判断一条uri跟指定的多条uri中的哪条匹配
* 添加匹配规则

```
//指定多条uri
um.addURI("com.itheima.person", "person", PERSON_CODE);
um.addURI("com.itheima.person", "company", COMPANY_CODE);
//#号可以代表任意数字
um.addURI("com.itheima.person", "person/#", QUERY_ONE_PERSON_CODE);
```

* 通过Uri匹配器可以实现操作不同的表

```java
@Override
public Uri insert(Uri uri, ContentValues values) {
  if(um.match(uri) == PERSON_CODE){
    db.insert("person", null, values);
  }
  else if(um.match(uri) == COMPANY_CODE){
    db.insert("company", null, values);
  }
  else{
    throw new IllegalArgumentException();
  }
  return uri;
}
```

* 如果路径中带有数字，把数字提取出来的api

```
int id = (int) ContentUris.parseId(uri);
```

# 7. 短信数据库
* 只需要关注sms表
* 只需要关注4个字段
  * body：短信内容
  * address:短信的发件人或收件人号码（跟你聊天那哥们的号码）
  * date：短信时间
  * type：1为收到，2为发送
* 读取系统短信，首先查询源码获得短信数据库内容提供者的主机名和路径，然后

```java
ContentResolver cr = getContentResolver();
Cursor c = cr.query(Uri.parse("content://sms"), new String[]{"body", "date", "address", "type"}, null, null, null);
while(c.moveToNext()){
  String body = c.getString(0);
  String date = c.getString(1);
  String address = c.getString(2);
  String type = c.getString(3);
  System.out.println(body+";" + date + ";" + address + ";" + type);
}
```

* 插入系统短信

```java
ContentResolver cr = getContentResolver();
ContentValues cv = new ContentValues();
cv.put("body", "您尾号为XXXX的招行储蓄卡收到转账1,000,000人民币");
cv.put("address", 95555);
cv.put("type", 1);
cv.put("date", System.currentTimeMillis());
cr.insert(Uri.parse("content://sms"), cv);
```

* 插入查询系统短信需要注册权限

# 8. 联系人数据库

* raw\_contacts表：
  * contact_id：联系人id
* data表：联系人的具体信息，一个信息占一行
  * data1：信息的具体内容
  * raw\_contact_id：联系人id，描述信息属于哪个联系人
  * mimetype_id：描述信息是属于什么类型
* mimetypes表：通过mimetype_id到该表查看具体类型

## 8.1 读取联系人
* 先查询raw\_contacts表拿到联系人id

```java
Cursor cursor = cr.query(Uri.parse("content://com.android.contacts/raw_contacts"), new String[]{"contact_id"}, null, null, null);
```

* 然后拿着联系人id去data表查询属于该联系人的信息

```java
Cursor c = cr.query(Uri.parse("content://com.android.contacts/data"), new String[]{"data1", "mimetype"}, "raw_contact_id = ?", new String[]{contactId}, null);
```

* 得到data1字段的值，就是联系人的信息，通过mimetype判断是什么类型的信息

```java
while(c.moveToNext()){
			String data1 = c.getString(0);
			String mimetype = c.getString(1);
			if("vnd.android.cursor.item/email_v2".equals(mimetype)){
				contact.setEmail(data1);
			}
			else if("vnd.android.cursor.item/name".equals(mimetype)){
				contact.setName(data1);
			}
			else if("vnd.android.cursor.item/phone_v2".equals(mimetype)){
				contact.setPhone(data1);
			}
		}
```

## 8.2 插入联系人
-  先查询raw\_contacts表，确定新的联系人的id应该是多少
* 把确定的联系人id插入raw\_contacts表

```java
cv.put("contact_id", _id);	cr.insert(Uri.parse("content://com.android.contacts/raw_contacts"), cv);
```

- 在data表插入数据
  - 插3个字段：data1、mimetype、raw\_contact_id
```java
cv = new ContentValues();
cv.put("data1", "赵六");
cv.put("mimetype", "vnd.android.cursor.item/name");
cv.put("raw_contact_id", _id);
cr.insert(Uri.parse("content://com.android.contacts/data"), cv);

cv = new ContentValues();
cv.put("data1", "1596874");
cv.put("mimetype", "vnd.android.cursor.item/phone_v2");
cv.put("raw_contact_id", _id);
cr.insert(Uri.parse("content://com.android.contacts/data"), cv);
```

# 9. 内容观察者
- 当数据库数据改变时，内容提供者会发出通知，在内容提供者的uri上注册一个内容观察者，就可以收到数据改变的通知

```java
cr.registerContentObserver(Uri.parse("content://sms"), true, new MyObserver(new Handler()));
		
		class MyObserver extends ContentObserver{

			public MyObserver(Handler handler) {
				super(handler);
				// TODO Auto-generated constructor stub
			}
	
			//内容观察者收到数据库发生改变的通知时，会调用此方法
			@Override
			public void onChange(boolean selfChange) {

			}
		
		}
```

- 在内容提供者中发通知的代码

```java
ContentResolver cr = getContext().getContentResolver();
//发出通知，所有注册在这个uri上的内容观察者都可以收到通知
cr.notifyChange(uri, null);
```

# 10. ContentProvider
- 四大组件之一
- 内容提供者的作用：把私有数据暴露给其他应用，通常，是把私有数据库的数据暴露给其他应用

## 10.1 短信数据库
- sms表
  - body：短信内容
  - date：短信时间
  - address：对方号码
  - type：发送还是接收

## 10.2 联系人数据库
- raw_contacts表
  - contact_id：联系人id
- data表：存放联系人的详细的信息，每行数据是单独的一条联系人信息
  - data1：联系人的具体的信息
  - raw_contact_id：该行信息属于哪个联系人
  - mimetype_id：该行信息属于什么类型
- mimetypes表：mimetype_id对应的类型的字符串

# 11. 内容提供者案例

## **案例1：自定义内容提供者**

```java
public class PersonProvider extends ContentProvider {

	private MyOpenHelper oh;
	SQLiteDatabase db;

	//创建uri匹配器对象
	static UriMatcher um = new UriMatcher(UriMatcher.NO_MATCH);
	//检测其他用户传入的uri与匹配器定义好的uri中，哪条匹配
	static {
		um.addURI("com.itheima.people", "person", 1);//content://com.itheima.people/person
		um.addURI("com.itheima.people", "teacher", 2);//content://com.itheima.people/teacher
		um.addURI("com.itheima.people", "person/#", 3);//content://com.itheima.people/person/4
	}
	
	//内容提供者创建时调用
	@Override
	public boolean onCreate() {
		oh = new MyOpenHelper(getContext());
		db = oh.getWritableDatabase();
		return false;
	}

	@Override
	public Cursor query(Uri uri, String[] projection, String selection,
			String[] selectionArgs, String sortOrder) {
		Cursor cursor = null;
		if(um.match(uri) == 1){
			cursor = db.query("person", projection, selection, selectionArgs, null, null, sortOrder, null);
		}
		else if(um.match(uri) == 2){
			cursor = db.query("teacher", projection, selection, selectionArgs, null, null, sortOrder, null);
		}
		else if(um.match(uri) == 3){
			//把uri末尾携带的数字取出来
			long id = ContentUris.parseId(uri);
			cursor = db.query("person", projection, "_id = ?", new String[]{id + ""}, null, null, sortOrder, null);
		}
		else{
			throw new IllegalArgumentException("uri又有问题哟亲么么哒");
		}
		return cursor;
	}

	@Override
	public String getType(Uri uri) {
		if(um.match(uri) == 1){
			return "vnd.android.cursor.dir/person";
		}
		else if(um.match(uri) == 3){
			return "vnd.android.cursor.item/person";
		}
		return null;
	}

	//此方法供其他应用调用，用于往people数据库里插数据
	//values：由其他应用传入，用于封装要插入的数据
	//uri:内容提供者的主机名，也就是地址
	@Override
	public Uri insert(Uri uri, ContentValues values) {
		//使用uri匹配器匹配传入的uri
		if(um.match(uri) == 1){
			db.insert("person", null, values);
			
			//发送数据改变的通知
			//uri:通知发送到哪一个uri上，所有注册在这个uri上的内容观察者都可以收到这个通知
			getContext().getContentResolver().notifyChange(uri, null);
		}
		else if(um.match(uri) == 2){
			db.insert("teacher", null, values);
			
			getContext().getContentResolver().notifyChange(uri, null);
		}
		else{
			throw new IllegalArgumentException("uri有问题哟亲么么哒");
		}
		return uri;
	}

	@Override
	public int delete(Uri uri, String selection, String[] selectionArgs) {
		int i = db.delete("person", selection, selectionArgs);
		return i;
	}

	@Override
	public int update(Uri uri, ContentValues values, String selection,
			String[] selectionArgs) {
		int i = db.update("person", values, selection, selectionArgs);
		return i;
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


    public void insert(View v){
    	//通过内容提供者把数据插入people数据库
    	//拿到contentResolver
    	ContentResolver cr = getContentResolver();
    	ContentValues values = new ContentValues();
    	values.put("name", "赵帅哥");
//    	values.put("money", "13000");
    	//url:内容提供者的主机名
    	//values:要插入的数据
    	cr.insert(Uri.parse("content://com.itheima.people/teacher"), values);
    	
    }
    
    public void delete(View v){
    	ContentResolver cr = getContentResolver();
    	int i = cr.delete(Uri.parse("content://com.itheima.people"), "name = ?", new String[]{"小志"});
    	System.out.println(i);
    }
    public void update(View v){
    	ContentResolver cr = getContentResolver();
    	ContentValues values = new ContentValues();
    	values.put("name", "sb志");
    	int i = cr.update(Uri.parse("content://com.itheima.people"), values, "name = ?", new String[]{"大志"});
    	System.out.println(i);
    }
    
    public void select(View v){
    	ContentResolver cr = getContentResolver();
    	Cursor cursor = cr.query(Uri.parse("content://com.itheima.people/person/4"), null, null, null, null);
    	while(cursor.moveToNext()){
    		String name = cursor.getString(1);
    		String money = cursor.getString(2);
    		System.out.println(name + ";" + money);
    	}
    }
    
}
```
## **案例2：获取系统短信**

```java
public class MainActivity extends Activity {

	List<Message> smsList;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        smsList = new ArrayList<Message>();
    }


    public void click(View v){
    	//访问内容提供者获取短信
    	ContentResolver cr = getContentResolver();
    	//						短信内容提供者的主机名
    	Cursor cursor = cr.query(Uri.parse("content://sms"), new String[]{"address", "date", "body", "type"}, 
    			null, null, null);
    	while(cursor.moveToNext()){
    		String address = cursor.getString(0);
    		long date = cursor.getLong(1);
    		String body = cursor.getString(2);
    		String type = cursor.getString(3);
    		Message sms = new Message(body, type, address, date);
    		smsList.add(sms);
    	}
    }
    
    public void click2(View v){
    	XmlSerializer xs = Xml.newSerializer();
    	File file = new File("sdcard/sms.xml");
    	FileOutputStream fos;
		try {
			fos = new FileOutputStream(file);
			xs.setOutput(fos, "utf-8");
			
			xs.startDocument("utf-8", true);
			xs.startTag(null, "message");
			
			for (Message sms : smsList) {
				xs.startTag(null, "sms");
				
				xs.startTag(null, "body");
				xs.text(sms.getBody());
				xs.endTag(null, "body");
				
				xs.startTag(null, "date");
				xs.text(sms.getDate() + "");
				xs.endTag(null, "date");
				
				xs.startTag(null, "type");
				xs.text(sms.getType());
				xs.endTag(null, "type");
				
				xs.startTag(null, "address");
				xs.text(sms.getAddress());
				xs.endTag(null, "address");
				
				xs.endTag(null, "sms");
			}
			
			xs.endTag(null, "message");
			xs.endDocument();
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
    }
    
}
```
## **案例3：插入系统短信**

```java
public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }


    public void click(View v){
    	Thread t = new Thread(){
    		@Override
    		public void run() {
    			try {
					sleep(7500);
				} catch (InterruptedException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
    			ContentResolver cr = getContentResolver();
    	    	ContentValues values = new ContentValues();
    	    	values.put("address", 95555);
    	    	values.put("type", 1);
    	    	values.put("date", System.currentTimeMillis());
    	    	values.put("body", "您尾号为XXXX的信用卡收到1,000,000RMB转账，请注意查收");
    	    	cr.insert(Uri.parse("content://sms"), values);
    		}
    	};
    	t.start();
    }
    
}
```
## **案例4：查询联系人** 

```java
public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }


    public void click(View v){
    	//通过内容提供者访问联系人数据库
    	ContentResolver cr = getContentResolver();
    	Cursor cursorContactId = cr.query(Uri.parse("content://com.android.contacts/raw_contacts"), new String[]{"contact_id"}, null, null, null);
    	while(cursorContactId.moveToNext()){
    		//获取联系人id
    		String contactId = cursorContactId.getString(0);
    		Cursor cursorData =  cr.query(Uri.parse("content://com.android.contacts/data"), new String[]{"data1", "mimetype"}, 
    				"raw_contact_id = ?", new String[]{contactId}, null);
    		//获取所有字段的名字
//    		String[] names = cursorData.getColumnNames();
//    		for (String string : names) {
//				System.out.println(string);
//			}
    		Contact con = new Contact();
    		while(cursorData.moveToNext()){
    			String data1 = cursorData.getString(0);
    			String mimetype = cursorData.getString(1);
    			//通过mimetype的判断，把data1存入对应的属性
    			if("vnd.android.cursor.item/email_v2".equals(mimetype)){
    				con.setEmail(data1);
    			}
    			else if("vnd.android.cursor.item/phone_v2".equals(mimetype)){
    				con.setPhone(data1);
    			}
    			else if("vnd.android.cursor.item/name".equals(mimetype)){
    				con.setName(data1);
    			}
    		}
    		System.out.println(con.toString());
    	}
    }
    
}
```
## **案例5：插入联系人** 

```java
public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }


    public void click(View v){
    	ContentResolver cr = getContentResolver();
    	//先查询raw_contacts表，获取最新联系人的主键，然后主键+1，就是要插入的联系人的id
    	Cursor cursorContactId = cr.query(Uri.parse("content://com.android.contacts/raw_contacts"), new String[]{"_id"}, null, null, null);
    	//默认联系人id就是1
    	int contact_id = 1;
    	if(cursorContactId.moveToLast()){
    		//拿到主键
    		int _id = cursorContactId.getInt(0);
    		//主键+1，就是要插入的联系人id
    		contact_id = ++_id;
    	}
    	
    	ContentValues values = new ContentValues();
    	values.put("contact_id", contact_id);
    	//把联系人id插入raw_contacts数据库
    	cr.insert(Uri.parse("content://com.android.contacts/raw_contacts"), values);
    	
    	values.clear();
    	values.put("data1", "二bi");
    	values.put("mimetype", "vnd.android.cursor.item/name");
    	values.put("raw_contact_id", contact_id);
    	cr.insert(Uri.parse("content://com.android.contacts/data"), values);
    	
    	values.clear();
    	values.put("data1", "1344567");
    	values.put("mimetype", "vnd.android.cursor.item/phone_v2");
    	values.put("raw_contact_id", contact_id);
    	cr.insert(Uri.parse("content://com.android.contacts/data"), values);
    }
    
}
```
## **案例6：内容观察者**

```java
public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        //注册一个内容观察者，监听短信数据库内容的改变
        ContentResolver cr = getContentResolver();
        //uri:监听哪个uri上的内容提供者的通知
        //notifyForDescendents:如果是true，那么只要以content://sms开头的uri的数据改变，都能收到通知，比如content://sms/inbox
        cr.registerContentObserver(Uri.parse("content://sms"), true, new MyObserver(new Handler()));
    }
    
    class MyObserver extends ContentObserver{

		public MyObserver(Handler handler) {
			super(handler);
			// TODO Auto-generated constructor stub
		}
    	
		//收到数据改变的通知，此方法调用
		@Override
		public void onChange(boolean selfChange) {
			// TODO Auto-generated method stub
			super.onChange(selfChange);
			System.out.println("短信数据库改变");
		}
		
    } 
}
```
## **案例7：监听数据库的改变**

```java
public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        getContentResolver().registerContentObserver(Uri.parse("content://com.itheima.people"), true, 
        		new ContentObserver(new Handler()) {
        				@Override
        				public void onChange(boolean selfChange) {
        					// TODO Auto-generated method stub
        					super.onChange(selfChange);
        					System.out.println("01项目的数据库改变了");
        				}
				});
    } 
}
```

