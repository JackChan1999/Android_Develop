# 1. pull解析XML文件

Android推荐使用pull解析XML文件，与SAX解析XML文件类似，都是事件驱动类型的解析方式。

示例：获取天气信息

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/104147di13z8l3likl12x8.png.thumb.jpg)

res\layout\activity_main.xml

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity" >

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="获取天气信息"
        android:onClick="click"/>

</RelativeLayout>
```

src/weather.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<weather>
        <city>
                <name>西安</name>
                <temp>22°</temp>
                <pm25>100</pm25>
        </city>
        <city>
                <name>上海</name>
                <temp>24°</temp>
                <pm25>120</pm25>
        </city>
        <city>
                <name>北京</name>
                <temp>30°</temp>
                <pm25>800</pm25>
        </city>
</weather>
```
src/cn.itcast.pullParser.domain/City.java

```java
package cn.itcast.pullparser.domain;

public class City {

        private String name;
        private String temp;
        private String pm25;

        public String getName() {
                return name;
        }
        public void setName(String name) {
                this.name = name;
        }
        public String getTemp() {
                return temp;
        }
        public void setTemp(String temp) {
                this.temp = temp;
        }
        public String getPm25() {
                return pm25;
        }
        public void setPm25(String pm25) {
                this.pm25 = pm25;
        }

        @Override
        public String toString() {
                return "City [name=" + name + ", temp=" + temp + ", pm25=" + pm25 + "]";
        }
}
```
src/cn.itcast.pullparser/MainActivity.java
```java
package cn.itcast.pullparser;

import java.io.InputStream;
import java.util.ArrayList;
import java.util.List;
import org.xmlpull.v1.XmlPullParser;
import android.app.Activity;
import android.os.Bundle;
import android.util.Xml;
import android.view.View;
import cn.itcast.pullparser.domain.City;

public class MainActivity extends Activity {
        List<City> cityList;

        @Override
        protected void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                setContentView(R.layout.activity_main);
        }

        public void click(View v){
                //获取xml文件
                InputStream is = getClassLoader().getResourceAsStream("weather.xml");
                //获取pull解析器
                //XmlPullParser是一个接口，使用Xml.newPullParser()获取XmlPullParser对象
                XmlPullParser xp = Xml.newPullParser();
                //初始化
                //Android系统默认编码是“utf-8”
                try {
                        xp.setInput(is,"utf-8");

                        //开始解析
                        //获取事件类型，通过对事件类型的判断，直到当前解析的是什么节点
                        int type = xp.getEventType();
                        City city = null;
                        while(type != XmlPullParser.END_DOCUMENT){
                                switch(type){
                                        case XmlPullParser.START_TAG:
                                                //获取当前节点的名字
                                                if("weather".equals(xp.getName())){
                                                        cityList = new ArrayList<City>();
                                                }else if("city".equals(xp.getName())){
                                                        city = new City();
                                                }else if("name".equals(xp.getName())){
                                                        //获取当前节点的下一个节点的文本
                                                        String name = xp.nextText();
                                                        city.setName(name);
                                                }else if("temp".equals(xp.getName())){
                                                        String temp = xp.nextText();
                                                        city.setTemp(temp);
                                                }else if("pm25".equals(xp.getName())){
                                                        String pm25 = xp.nextText();
                                                        city.setPm25(pm25);
                                                }
                                                break;
                                        case XmlPullParser.END_TAG:
                                                if("city".equals(xp.getName())){
                                                        cityList.add(city);
                                                }
                                                break;
                                }

                                //把指针移动至下一个节点，并返回该节点的事件类型
                                type = xp.next();
                        }
                } catch (Exception e) {
                        e.printStackTrace();
                }

                for(City c:cityList){
                    System.out.println(c.toString());
                }
        }
}
```
运行结果：

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/104156uu5686865gk6f668.png.thumb.jpg)

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/104152rgassfwggsfe99gz.png.thumb.jpg)

## 1.1 用debug查看pull解析流程

首先，双击代码左侧，打一个断点。

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/105538ki7rs3bcpcuswest.png.thumb.jpg)

点击debug “01_pull解析XML文件”。

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/105539fnib9fhzffrrenag.png.thumb.jpg)

等待debug初始化，跳出对话框，不要点“Force close”！等待一会，对话框就会消失。

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/105603y0xx4l404wsgi54t.png.thumb.jpg)

点击“获取天气信息”按钮。

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/105815x3ngt8ezgtcu6t32.png.thumb.jpg)

点击“yes”，程序进入debug模式。

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/105811i773evznl5v3q1nz.png.thumb.jpg)

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/110109j5pya555jz775j30.png.thumb.jpg)

通过“step over”即可以查看代码执行的每一步。

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/110115cupjroeupqopr5ey.png.thumb.jpg)

# 2. 测试概念

- 按岗位分：
    - 黑盒测试：测试业务逻辑
    - 白盒测试：测试逻辑方法

- 按测试粒度分：
    - 方法测试 function   
    - 单元测试 unit：多个方法集成一个单元，测试
    - 集成测试 intergration：多个单元集成，测试
    - 系统测试 system：服务器端、客户端端联动，测试整个系统

- 按暴力程度分：
    - 冒烟测试 smoke
    - 压力测试 pressure：针对服务器端的测试。

目前的智能手机都有测试框架，例如，Android自带的monkey，不过需要在Android命令行中使用。

示例：随机点击模拟器一千次

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/112141dpzmiibmzi0szfty.png.thumb.jpg)

进入测试状态...

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/112143ezal80dd3xlahhtl.png.thumb.jpg)

## 2.1 单元测试
![img](http://bbs.itheima.com/data/attachment/forum/201506/24/115049bw2iioi82gtgpozx.png.thumb.jpg)

创建一个类，继承AndroidTestCase

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/112626yyjmsycox3mmo41m.png.thumb.jpg)

src/cn.itcast.junit/Test.java
```java
package cn.itcast.junit;

import cn.itcast.junit.tools.Tools;
import android.test.AndroidTestCase;

public class Test extends AndroidTestCase {

        //定义在测试框架中的方法可以直接运行
        public void test(){
                int result = Tools.damage(8, 3);
                //断言，对比实际结果与预期是否一致
                assertEquals(5, result);
        }
}
```
src/cn.itcast.junit.tools/Tools.java
```java
package cn.itcast.junit.tools;

public class Tools {
        public static int damage(int i,int j){
                return i + j;
        }
}
```
使用测试框架需要在清单文件中配置指令集和使用类库。

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/114908qk0d2f0nyfkh6qad.png.thumb.jpg)

两种方式启动测试：方法一：在代码区内选择需要测试的方法-->右击Run As-->Android JUnit Test。

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/114907rl3rng32cc3rv393.png.thumb.jpg)

方法二：在Outline内选择需要测试的方法-->右击Run As-->Android JUnit Test。

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/115634w8tvdwgyvaljmvuy.png.thumb.jpg)

通过下图，可以看到，报错，结果与断言不相符。

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/114906bvfd4irliurfvbiz.png.thumb.jpg)

修改Tools.java中代码。

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/115846u3057it8hh2h8ghu.png.thumb.jpg)

通过下图，可以看到，测试运行成功，结果与断言相符。

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/114906dwwsg55esgq2rq4k.png.thumb.jpg)

通过Console可以看到测试过程。

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/120139iduu8u5sol2qth25.png.thumb.jpg)

虽然，测试过程中，Android项目并没有启动，整个测试过程可以看到模拟器上根本没有前台界面。但是，测试需要模拟器运行测试框架。因为，测试的项目是Android代码，必须运行在Android设备上，可以是模拟器也可以是真机。没有返回值的方法测试步骤相同，只是不需要断言。

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/121432g39r9rpp04j3n4pg.png.thumb.jpg)

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/121431gj6djg067dyxkjzb.png.thumb.jpg)

测试后，报错，提示出现异常。

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/121430zwhd8ml7od1alga0.png.thumb.jpg)

修改代码，测试成功。

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/121429jvd82v25q3sq3yc1.png.thumb.jpg)

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/121428mkti676stcj3zwb4.png.thumb.jpg)

# 3. 创建数据库
Android中存储数据用的最多的还是数据库，SQLite（Android自带的轻量级数据库）。
数据库保存在内部存储空间。所以，在备份数据的时候，不会写在数据库里。一旦应用删除，数据库就没了，备份的数据也就全没了。

示例：创建数据库

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/130908hcbmp8j8pz0gmg4j.png.thumb.jpg)

创建一个类，MyOpenHelper，继承SQLiteOpenHelper。

src/cn.itcast.sqlite/MyOpenHelper.java
```java
package cn.itcast.sqlite;

import android.content.Context;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteDatabase.CursorFactory;
import android.database.sqlite.SQLiteOpenHelper;

public class MyOpenHelper extends SQLiteOpenHelper {

        public MyOpenHelper(Context context, String name, CursorFactory factory,
                        int version) {
                //第一个参数：用来打开和创建数据库的上下文
                //第二个参数：数据库文件名字
                //第三个参数：游标工厂，用来创建游标对象的工厂。如果传null，表示使用默认的游标工厂
                //第四个参数：数据库的版本，最低为1，如果数据库做升级，版本值只能升，不能降
                super(context, name, factory, version);
        }

        //数据库创建时，此方法调用
        @Override
        public void onCreate(SQLiteDatabase db) {

        }

        //数据库升级时，此方法调用
        @Override
        public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {

        }
}
```
创建一个测试类，继承AndroidTestCase。
src/cn.itcast.sqlite/MyOpenHelper.java
```java
package cn.itcast.sqlite;

import android.database.sqlite.SQLiteDatabase;
import android.test.AndroidTestCase;

public class Test extends AndroidTestCase {

        public void createDataBase(){

                //创建数据库
                //1. 创建打开帮助器
                //测试框架提供了getContext()方法，获取虚拟上下文
                MyOpenHelper oh = new MyOpenHelper(getContext(), "people.db", null, 1);

                //2. 创建并打开数据库
                //如果数据库不存在，先创建，后打开。如果数据库存在，直接打开
                //getWritableDatabase创建的数据库可读可写。
                //getReadableDatabase返回的数据库也是可读可写。但是，在某些情况下，例如，数据库满了（但是现在，手机内部存储器都比较大，基本上不会出现这种情况），getReadableDatabase就会返回一个只读的数据库。
                SQLiteDatabase db = oh.getWritableDatabase();
        }
}
```
配置清单文件：
![img](http://bbs.itheima.com/data/attachment/forum/201506/24/131208vzs0bm278ff11u0a.png.thumb.jpg)

测试运行结果：

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/131313gyig2gf1xf4270iy.png.thumb.jpg)

生成数据库成功。

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/131415n7d5gc5woe5aqqc5.png.thumb.jpg)

导出people.db，拖入SQLite Expert，可以看到自动生成了一张表，此表不能修改，不用管它。

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/132207pnfgttxvs32blvbn.png.thumb.jpg)

## 3.1 创建表
创建数据库完毕后会调用onCreate方法，数据库升级后会调用onUpgrade方法。

示例：src/cn.itcast.sqlite/MyOpenHelper.java
```java
package cn.itcast.sqlite;

import android.content.Context;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteDatabase.CursorFactory;
import android.database.sqlite.SQLiteOpenHelper;

public class MyOpenHelper extends SQLiteOpenHelper {

        public MyOpenHelper(Context context, String name, CursorFactory factory,
                        int version) {
                super(context, name, factory, version);
        }

        //数据库创建完之后，才会调用这个函数
        @Override
        public void onCreate(SQLiteDatabase db) {
                System.out.println("数据库创建");
        }

        @Override
        public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
                System.out.println("数据库升级");
        }
}
```
运行结果：
首先，删除创建好的数据库，重新创建数据库。

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/133956cttf2swx9mh1mzic.png.thumb.jpg)

可以看到onCreate方法被调用。

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/134316ju74bzqq3u4slsy7.png.thumb.jpg)   

然后，升级数据库版本。

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/134524p491z9c67rs9kdpp.png.thumb.jpg)

可以看到onUpgrade方法被调用。

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/134522ynuufffupfnjis0k.png.thumb.jpg)

示例：创建表
src/cn.itcast.sqlite/MyOpenHelper.java
```java
package cn.itcast.sqlite;

import android.content.Context;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteDatabase.CursorFactory;
import android.database.sqlite.SQLiteOpenHelper;

public class MyOpenHelper extends SQLiteOpenHelper {

        public MyOpenHelper(Context context, String name, CursorFactory factory,
                        int version) {
                super(context, name, factory, version);
        }

        @Override
        public void onCreate(SQLiteDatabase db) {
                //执行sql语句
                //Android所有自带的数据库主键都带下划线，入乡随俗
                //所有字段类型对于SQLite都是varchar类型，sql语句写数据类型是写给程序员看的
                db.execSQL("create table person(_id integer primary key autoincrement,name char(10),phone char(20),salary integer(10))");
        }

        @Override
        public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
                System.out.println("数据库升级");
        }
}
```
src/cn.itcast.sqlite/Test.java
```java
package cn.itcast.sqlite;

import android.database.sqlite.SQLiteDatabase;
import android.test.AndroidTestCase;

public class Test extends AndroidTestCase {

        public void createDataBase(){
                MyOpenHelper oh = new MyOpenHelper(getContext(), "people.db", null, 1);
                SQLiteDatabase db = oh.getWritableDatabase();
        }
}
```
测试运行结果：
首先，删除数据库，重新创建数据库，创建表。然后导出people.db，拖入SQLite Expert，刷新。可以看到，生成了新的person表。

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/135800vvlx60bb53xsbpzp.png.thumb.jpg)

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/135758m8yujrrj8trlhrth.png.thumb.jpg)

表格中的RecNo是不存在的字段，只是为了让开发人员能够方便地看到记录行数。

## 3.2 使用SQL语句插入删除

示例：src/cn.itcast.sqlite/Test.java
```java
package cn.itcast.sqlite;

import android.database.sqlite.SQLiteDatabase;
import android.test.AndroidTestCase;

public class Test extends AndroidTestCase {

        private MyOpenHelper oh;
        private SQLiteDatabase db;

        //测试框架初始化完毕后，测试方法执行前，此方法调用
        @Override
        protected void setUp() throws Exception {
                super.setUp();
                oh =  new MyOpenHelper(getContext(), "people.db", null, 1);
                db = oh.getWritableDatabase();
        }

        public void createDataBase(){
                MyOpenHelper oh = new MyOpenHelper(getContext(), "people.db", null, 1);
                SQLiteDatabase db = oh.getWritableDatabase();
        }

        public void insert(){
                db.execSQL("insert into person(name,phone,salary) values(?,?,?)",new Object[]{"云鹤",138438,"13000"});
                db.execSQL("insert into person(name,phone,salary) values(?,?,?)",new Object[]{"云鹤儿子",138438,"13000"});
                db.execSQL("insert into person(name,phone,salary) values(?,?,?)",new Object[]{"云鹤孙子",138438,"13000"});
                db.execSQL("insert into person(name,phone,salary) values(?,?,?)",new Object[]{"云鹤曾孙",138438,"13000"});
                db.execSQL("insert into person(name,phone,salary) values(?,?,?)",new Object[]{"王亚聪",138438,"13000"});
        }

        public void delete(){
                db.execSQL("delete from person where name = ?",new Object[]{"王亚聪"});
        }

        //测试方法执行完毕后，执行。
        @Override
        protected void tearDown() throws Exception {
                super.tearDown();
                db.close();
        }
}
```
测试运行结果：
首先，测试insert方法，插入数据，然后导出people.db，拖入SQLite Expert，刷新。可以看到，插入数据成功。

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/143204wxkvxt1gkltege33.png.thumb.jpg)

然后，测试delete方法，删除一条数据，导出people.db，拖入SQLite Expert，刷新。可以看到，删除数据成功。

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/143306b0pm2nhmd0d0qhmp.png.thumb.jpg)

如果将代码调整，如下图，测试insert方法或delete方法，就会报空指针异常。

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/143540ur95b4r13ug5b6s4.png.thumb.jpg)

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/143539v3kc2t7koprut743.png.thumb.jpg)

原因分析：测试框架需要先被系统创建出来，然后再对它做各个参数的初始化，所有初始化做完之后，虚拟上下文才会存在。但是，private MyOpenHelper oh =  new MyOpenHelper(getContext(), "people.db", null, 1);这条语句是在测试框架构造方法被调用之前调用的，因此，虚拟上下文（getContext方法返回值）根本不存在，获取不到。所以，MyOpenHelper根本new不出来。如此，才会报空指针异常。但是，setUp方法是在测试框架初始化完毕后，测试方法执行前，此方法才被调用。因此，虚拟上下文已经存在，没有任何问题。

## 3.3 使用SQL语句修改查询

示例：src/cn.itcast.sqlite/Test.java
```java
package cn.itcast.sqlite;

import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.test.AndroidTestCase;

public class Test extends AndroidTestCase {

        private MyOpenHelper oh;
        private SQLiteDatabase db;

        @Override
        protected void setUp() throws Exception {
                super.setUp();
                oh =  new MyOpenHelper(getContext(), "people.db", null, 1);
                db = oh.getWritableDatabase();
        }

        public void update(){
                db.execSQL("update person set salary = ? where name = ?",new Object[]{10000,"云鹤"});
        }

        public void select(){
                Cursor cursor = db.rawQuery("select name,salary from person where name = ?", new String[]{"云鹤"});
                //从游标中取出数据，方法与结果集类似
                while(cursor.moveToNext()){
                        //只能传入索引值，索引值根据sql语句中查询内容所在的顺序而定，例如，上面的sql语句，name在第一位，索引即为0
                        String name = cursor.getString(0);
                        //可以通过字段名称获取列索引，再通过列索引获取内容
                        String salary = cursor.getString(cursor.getColumnIndex("salary"));
                        System.out.println(name + ":" + salary);
                }
        }

        @Override
        protected void tearDown() throws Exception {
                super.tearDown();
                db.close();
        }
}
```
测试运行结果：
测试update方法，更新一条数据，导出people.db，拖入SQLite Expert，刷新。可以看到，更新数据成功。

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/145239wvo5kmxmgiiocco8.png.thumb.jpg)

测试select方法，查询数据，打印出来。可以看到，查询成功。

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/145242u3kq9saqtuq2u8bt.png.thumb.jpg)

## 3.4 使用API完成插入
使用SQL语句不太方便，可以使用API完成增删改查操作。

示例：src/cn.itcast.sqlite/Test.java
```java
package cn.itcast.sqlite;

import android.content.ContentValues;
import android.database.sqlite.SQLiteDatabase;
import android.test.AndroidTestCase;

public class Test extends AndroidTestCase {

        private MyOpenHelper oh;
        private SQLiteDatabase db;

        @Override
        protected void setUp() throws Exception {
                super.setUp();
                oh =  new MyOpenHelper(getContext(), "people.db", null, 1);
                db = oh.getWritableDatabase();
        }

        public void insertApi(){
                ContentValues values = new ContentValues();
                values.put("name","亚聪和云鹤");
                values.put("phone","138888");
                values.put("salary","99999999");
                //第一个参数：表名
                //第二个参数：nullColumnHack，当第三个参数ContentValues对象为null，或者里面没有任何数据时，第二个参数才会有效。开发中，ContentValues都是有值的，所以直接传入null即可
                //第三个参数：通过键值对把要插入的数据封装至ContentValues对象中，键名必须是字段名
                //db.insert返回值为行id，如果为-1，表示插入失败
                long l1 = db.insert("person", null, values);

                //ContentValues下次使用的时候记得清空
                values.clear();
                values.put("name","亚聪和云鹤的儿子");
                values.put("phone","138888");
                values.put("salary","99999999");
                long l2 = db.insert("person", null, values);

                System.out.println(l1 + ":" + l2);
        }

        @Override
        protected void tearDown() throws Exception {
                super.tearDown();
                db.close();
        }
}
```
测试运行结果：
测试insertApi方法，插入一条数据，导出people.db，拖入SQLite Expert，刷新。可以看到，插入数据成功。

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/150424j5kuu3ukkfl35nk3.png.thumb.jpg)

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/150421oi2yepmi1k27siip.png.thumb.jpg)

## 3.5 使用API完成删改查

示例：src/cn.itcast.sqlite/Test.java
```java
package cn.itcast.sqlite;

import android.content.ContentValues;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.test.AndroidTestCase;

public class Test extends AndroidTestCase {

        private MyOpenHelper oh;
        private SQLiteDatabase db;

        @Override
        protected void setUp() throws Exception {
                super.setUp();
                oh =  new MyOpenHelper(getContext(), "people.db", null, 1);
                db = oh.getWritableDatabase();
        }

        public void deleteApi(){
                //返回被删除的行的数量
                int i = db.delete("person", "name = ?", new String[]{"云鹤儿子"});
                System.out.println(i);
        }

        public void updateApi(){
                ContentValues values = new ContentValues();
                values.put("salary",10500);
                int i = db.update("person", values, "name = ?", new String[]{"云鹤"});
                System.out.println(i);
        }

        public void selectApi(){
                //第二个参数如果传入null，表示查询所有字段
                Cursor cursor = db.query("person", null, null, null, null, null, null, null);
                while(cursor.moveToNext()){
                        String name = cursor.getString(1);
                        String phone = cursor.getString(2);
                        String salary = cursor.getString(3);
                        System.out.println(name + ":" + phone + ":" + salary);
                }
        }

        @Override
        protected void tearDown() throws Exception {
                super.tearDown();
                db.close();
        }
}
```
测试运行结果：

测试deleteApi方法，删除一条数据，导出people.db，拖入SQLite Expert，刷新。可以看到，删除数据成功。

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/160746k04zg7gix4at54uo.png.thumb.jpg)

测试updateApi方法，更新一条数据，导出people.db，拖入SQLite Expert，刷新。可以看到，更新数据成功。

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/160745kpmy2pmxg4305gpg.png.thumb.jpg)

测试selectApi方法，查询数据，打印出来。可以看到，查询成功。

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/160809jyzfglq9jg395jiy.png.thumb.jpg)

## 3.6 SQLite的事务

示例：src/cn.itcast.sqlite/Test.java
```java
package cn.itcast.sqlite;

import android.content.ContentValues;
import android.database.sqlite.SQLiteDatabase;
import android.test.AndroidTestCase;

public class Test extends AndroidTestCase {

        private MyOpenHelper oh;
        private SQLiteDatabase db;

        @Override
        protected void setUp() throws Exception {
                super.setUp();
                oh =  new MyOpenHelper(getContext(), "people.db", null, 1);
                db = oh.getWritableDatabase();
        }

        public void transaction(){
                try{
                        //开启事务
                        db.beginTransaction();
                        ContentValues values = new ContentValues();
                        values.put("salary", 11000);
                        db.update("person", values, "name = ?", new String[]{"云鹤"});
                        values.put("salary", 12500);
                        db.update("person", values, "name = ?", new String[]{"云鹤孙子"});

                        //如果下面这行代码执行，说明事务执行成功
                        db.setTransactionSuccessful();
                }catch(Exception e){
                        e.printStackTrace();
                }finally{
                        //关闭事务，这时候就提交了，不用再次提交
                        //关闭时候的时候，如果发现db.setTransactionSuccessful();语句未执行，那么就会回滚事务
                        db.endTransaction();
                }
        }

        @Override
        protected void tearDown() throws Exception {
                super.tearDown();
                db.close();
        }
}
```
测试运行结果;
测试transaction方法导出people.db，拖入SQLite Expert，刷新。可以看到，事务执行成功。

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/162101y8pp5chzzkaf305n.png.thumb.jpg)

如果对代码作如下修改，事务执行中出现异常。

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/162237kxgvmhh099t9ev0l.png.thumb.jpg)

可以看到，报异常，并且数据未发生任何变化。

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/162106hldolzpcckkk8o1g.png.thumb.jpg)

![img](http://bbs.itheima.com/data/attachment/forum/201506/24/162306jtmt7ww1lo7moewx.png.thumb.jpg)

## 3.7 把数据库中的数据显示至屏幕

![img](http://bbs.itheima.com/data/attachment/forum/201506/25/101953hkxmm9uskwc4qwea.png.thumb.jpg)

src/cn.itcast.showdata/MyOpenHelper.java
```java
package cn.itcast.showdata;

import android.content.Context;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;

public class MyOpenHelper extends SQLiteOpenHelper {

        public MyOpenHelper(Context context) {
                super(context, "people.db", null, 1);
        }

        @Override
        public void onCreate(SQLiteDatabase db) {
                db.execSQL("create table person(_id integer primary key autoincrement,name char(10),phone char(20),salary integer(10))");
        }

        @Override
        public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
                System.out.println("数据库升级");
        }
}
```
AndroidManifest.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="cn.itcast.showdata"
    android:versionCode="1"
    android:versionName="1.0" >

    <uses-sdk
        android:minSdkVersion="8"
        android:targetSdkVersion="17" />

        <instrumentation
        android:name="android.test.InstrumentationTestRunner"
        android:targetPackage="cn.itcast.showdata"
        ></instrumentation>

    <application
        android:allowBackup="true"
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme" >
        <uses-library android:name="android.test.runner"/>
        <activity
            android:name="cn.itcast.showdata.MainActivity"
            android:label="@string/app_name" >
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```
src/cn.itcast.showdata/Test.java
```java
package cn.itcast.showdata;

import android.content.ContentValues;
import android.database.sqlite.SQLiteDatabase;
import android.test.AndroidTestCase;

public class Test extends AndroidTestCase {

        private MyOpenHelper oh;
        private SQLiteDatabase db;

        @Override
        protected void setUp() throws Exception {
                super.setUp();
                oh =  new MyOpenHelper(getContext());
                db = oh.getWritableDatabase();
        }

        public void insertApi(){

                for(int i = 0; i < 50; i++){
                        ContentValues values = new ContentValues();
                        values.put("name","张" + i);
                        values.put("phone","138" + i + i);
                        values.put("salary","200" + i);
                        db.insert("person", null, values);
                }
        }

        @Override
        protected void tearDown() throws Exception {
                super.tearDown();
                db.close();
        }
}
```
src/cn.itcast.showdata.domain/Person.java
```java
package cn.itcast.showdata.domain;

public class Person {

        private int _id;
        private String name;
        private String phone;
        private int salary;

        public Person(int _id, String name, String phone, int salary) {
                super();
                this._id = _id;
                this.name = name;
                this.phone = phone;
                this.salary = salary;
        }

        public int get_id() {
                return _id;
        }
        public void set_id(int _id) {
                this._id = _id;
        }
        public String getName() {
                return name;
        }
        public void setName(String name) {
                this.name = name;
        }
        public String getPhone() {
                return phone;
        }
        public void setPhone(String phone) {
                this.phone = phone;
        }
        public int getSalary() {
                return salary;
        }
        public void setSalary(int salary) {
                this.salary = salary;
        }

        @Override
        public String toString() {
                return "name=" + name + ", phone=" + phone+ ", salary=" + salary;
             }
     }
```
res\layout\activity_main.xml
```xml
     <ScrollView
         android:layout_width="match_parent"
             android:layout_height="match_parent"
         xmlns:android="http://schemas.android.com/apk/res/android"
             xmlns:tools="http://schemas.android.com/tools"
         >
             <LinearLayout
                 android:id="@+id/ll"
                 android:layout_width="match_parent"
                 android:layout_height="match_parent"
                 tools:context=".MainActivity"
                 android:orientation="vertical">

        </LinearLayout>
</ScrollView>
```
src/cn.itcast.showdata/MainActivity.java
```java
package cn.itcast.showdata;

import java.util.ArrayList;
import java.util.List;

import android.app.Activity;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.os.Bundle;
import android.widget.LinearLayout;
import android.widget.TextView;
import cn.itcast.sqlite.domain.Person;

public class MainActivity extends Activity {

        List<Person> personList;

        @Override
        protected void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                setContentView(R.layout.activity_main);

                personList = new ArrayList<Person>();

                //读取数据库
                MyOpenHelper oh = new MyOpenHelper(this);
                SQLiteDatabase db = oh.getWritableDatabase();

                Cursor cursor = db.query("person", null, null, null, null, null, null, null);

                while(cursor.moveToNext()){
                        int _id = cursor.getInt(0);
                        String name = cursor.getString(1);
                        String phone = cursor.getString(2);
                        int salary = cursor.getInt(3);

                        Person p = new Person(_id, name, phone, salary);
                        personList.add(p);
                }

                //获取线性布局
                LinearLayout ll = (LinearLayout) findViewById(R.id.ll);

                for(Person p : personList){
                        TextView tv = new TextView(this);
                        tv.setText(p.toString());
                        tv.setTextSize(16);

                        //把tv设置为线性布局的子节点
                        ll.addView(tv);
                }
        }
}
```
运行结果：

首先，在模拟器上运行应用程序，在data/data目录下创建应用程序文件目录。运行Test类中的insertApi方法，导出people.db文件，拖入SQLite Expert，可以看到插入数据成功。

![img](http://bbs.itheima.com/data/attachment/forum/201506/25/101803vplzyuba8rypawcw.png.thumb.jpg)

![img](http://bbs.itheima.com/data/attachment/forum/201506/25/102151u9gxmu1e1mbuuo8x.png.thumb.jpg)

然后，再次在模拟器上运行应用程序，将数据库中的数据通过TextView的形式展示在屏幕上。

![img](http://bbs.itheima.com/data/attachment/forum/201506/25/102455hqujucwq8i0w8vw0.png.thumb.jpg)

使用ScrollView包裹LinearLayout，ScrollView表示可以上下滑动的View。如果线性布局过大，超出屏幕显示范围，那么就可以上下滑动了。

# 4. 使用ListView显示数据

开发中不会按照如上方法显示数据到屏幕上的。因为，如果待显示数据太多，程序可能会崩溃。原因在于循环提取数据，如果有10000条数据，就需要创建10000个JavaBean对象和10000个TextView对象到内存中，太耗内存。解决方案之一是采用分页查询，只查询一部分数据，通过SQL语句的limit控制读取的条数。另一个Android方式的解决方案是动态创建TextView，也就是说屏幕只能显示10条数据，那么就创建10个JavaBean对象和10个TextView对象。如果用户下滑屏幕显示数据，一方面缓存（内存不足的时候会被摧毁，内存充足的时候缓存，避免下次再显示的时候重新创建，提升性能）屏幕上方已经消失的TextView，一方面创建屏幕下方要显示的TextView。这样，内容中只有10个JavaBean对象及10个TextView，避免内存溢出。Android中用ListView实现此功能。

ListView专门用于显示列表，每一行称为一个条目（Item）

MVC

- M:模型层 Javabean personList    
- V:视图层 jsp   ListView    
- C:控制层 servlet  Adapter（Adapter是适配器，用适配器控制要显示哪些内容）

ListAdapter是一个接口，需要实现的方法太多。因此，一般通过继承BaseAdapter（抽象类）的方式创建ListAdapter对象。

![img](http://bbs.itheima.com/data/attachment/forum/201506/25/105201vhflbi8mli52hshl.png.thumb.jpg)

所有在布局文件中定义的组件（包括LinearLayout），都可以在ListView中显示。

![img](http://bbs.itheima.com/data/attachment/forum/201506/25/105923e6jiuipz6r6jg5fi.png.thumb.jpg)

示例：修改上个示例中的代码如下：res\layout\activity_main.xml
```xml
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity"
    android:orientation="vertical">

    <ListView
        android:id="@+id/lv"
        android:layout_width="match_parent"
            android:layout_height="match_parent">

    </ListView>

</LinearLayout>
```
src/cn.itcast.listView/MainActivity.java
```java
package cn.itcast.listView;

import java.util.ArrayList;
import java.util.List;

import android.app.Activity;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.os.Bundle;
import android.view.View;
import android.view.ViewGroup;
import android.widget.BaseAdapter;
import android.widget.ListView;
import android.widget.TextView;
import cn.itcast.listView.domain.Person;

public class MainActivity extends Activity {

        List<Person> personList;

        @Override
        protected void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                setContentView(R.layout.activity_main);

                personList = new ArrayList<Person>();

                //读取数据库
                MyOpenHelper oh = new MyOpenHelper(this);
                SQLiteDatabase db = oh.getWritableDatabase();

                Cursor cursor = db.query("person", null, null, null, null, null, null, null);

                while(cursor.moveToNext()){
                        int _id = cursor.getInt(0);
                        String name = cursor.getString(1);
                        String phone = cursor.getString(2);
                        int salary = cursor.getInt(3);

                        Person p = new Person(_id, name, phone, salary);
                        personList.add(p);
                }

                ListView lv = (ListView)findViewById(R.id.lv);
                lv.setAdapter(new MyAdapter());
        }

        class MyAdapter extends BaseAdapter{

                //获取集合的元素数量
                @Override
                public int getCount() {
                        return personList.size();
                }

                //系统调用此方法，获取一个View对象，该View对象会作为一个条目显示至ListView中
                //position:getView返回的View对象会作为ListView的第几个条目显示，那么position的值就是多少
                @Override
                public View getView(int position, View convertView, ViewGroup parent) {
                        System.out.println("getView调用" + position);
                        Person p = personList.get(position);
                        TextView tv = new TextView(MainActivity.this);
                        tv.setText(p.toString());
                        tv.setTextSize(16);
                        return tv;
                }

                @Override
                public Object getItem(int position) {
                        return null;
                }

                @Override
                public long getItemId(int position) {
                        return 0;
                }
        }
}
```
运行结果：

![img](http://bbs.itheima.com/data/attachment/forum/201506/25/112014nk1dj8m267s5k5o8.png.thumb.jpg)

![img](http://bbs.itheima.com/data/attachment/forum/201506/25/112157g4al7aih15rvmmti.png.thumb.jpg)

如果将屏幕向下滑动，结果如下：

![img](http://bbs.itheima.com/data/attachment/forum/201506/25/112156nhzrb8shrh889djk.png.thumb.jpg)

如果将屏幕向上滑动，结果如下：

![img](http://bbs.itheima.com/data/attachment/forum/201506/25/112155a1u1u4ou41u1ujbu.png.thumb.jpg)

## 4.1 把布局文件填充成View对象

如果想要修改ListView的展示样式，那么就需要将指定的布局文件填充为View。

示例：修改上个示例中的代码，如下：res\layout\item_listview.xml
```xml
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">


    <TextView
        android:id="@+id/tv_name"
        android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="name"
            android:textSize="22sp"/>

        <LinearLayout
        android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:layout_alignParentRight="true"
            android:layout_centerVertical="true"
            >
            <TextView
                android:id="@+id/tv_phone"
                android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="phone"                
                />
            <TextView
                android:id="@+id/tv_salary"
                android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="salary"                
                />
        </LinearLayout>

</RelativeLayout>
```
src/cn.itcast.listView/MainActivity.java
```java
package cn.itcast.listView;

import java.util.ArrayList;
import java.util.List;

import android.app.Activity;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.BaseAdapter;
import android.widget.ListView;
import android.widget.TextView;
import cn.itcast.listView.domain.Person;

public class MainActivity extends Activity {

        List<Person> personList;

        @Override
        protected void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                setContentView(R.layout.activity_main);

                personList = new ArrayList<Person>();

                MyOpenHelper oh = new MyOpenHelper(this);
                SQLiteDatabase db = oh.getWritableDatabase();

                Cursor cursor = db.query("person", null, null, null, null, null, null, null);

                while(cursor.moveToNext()){
                        int _id = cursor.getInt(0);
                        String name = cursor.getString(1);
                        String phone = cursor.getString(2);
                        int salary = cursor.getInt(3);

                        Person p = new Person(_id, name, phone, salary);
                        personList.add(p);
                }

                ListView lv = (ListView)findViewById(R.id.lv);
                lv.setAdapter(new MyAdapter());
        }

        class MyAdapter extends BaseAdapter{

                @Override
                public int getCount() {
                        return personList.size();
                }

                @Override
                public View getView(int position, View convertView, ViewGroup parent) {
                        System.out.println("getView调用" + position);
                        Person p = personList.get(position);

                        //把指定布局文件填充成View对象
                        //第二个参数指定把哪个布局文件转换成View对象，变成ListView条目显示在屏幕上
                        //第三个参数给布局文件指定父节点，不需要，设置null
                        View v = View.inflate(MainActivity.this, R.layout.item_listview, null);

                        //第二种方式，不常用
                        //LayoutInflater inflater = LayoutInflater.from(MainActivity.this);
                        //View v = inflater.inflate(R.layout.item_listview, null);

                        //第三种方式，不常用
                        //LayoutInflater inflater = (LayoutInflater) getSystemService(LAYOUT_INFLATER_SERVICE);
                        //View v = inflater.inflate(R.layout.item_listview, null);

                        //修改View对象中各个组件的内容
                        //findViewById(R.id.tv_name);通过id寻找tv_name默认是在activity_mian.xml布局文件中查找（由于本类中onCreate方法有一条语句setContentView(R.layout.activity_main);）。
                        //v.findViewById(R.id.tv_name);就会在被填充的item_listview布局文件中查找tv_name。
                        TextView tv_name = (TextView)v.findViewById(R.id.tv_name);
                        tv_name.setText(p.getName());

                        TextView tv_phone = (TextView)v.findViewById(R.id.tv_phone);
                        tv_phone.setText(p.getPhone());

                        TextView tv_salary = (TextView)v.findViewById(R.id.tv_salary);
                        //参数一定要为字符串，如果是整数，那么就会把它作为一个id，去寻找相应的资源
                        tv_salary.setText(p.getSalary() + "");

                        return v;
                }

                //下面两个方法，系统不会调用，程序员可以调用
                //返回条目，一般返回某个元素，例如：personList.get(position);
                @Override
                public Object getItem(int position) {
                        return null;
                }

                //返回条目的id，一般返回元素的主键
                @Override
                public long getItemId(int position) {
                        return 0;
                }
        }
}
```
运行结果：

![img](http://bbs.itheima.com/data/attachment/forum/201506/25/121514v22uhqp5piqi8qpg.png.thumb.jpg)

## 4.2 ListView的优化
上面的代码，只要用户一直上下滑屏，就会不断填充新的View对象（View v = View.inflate(MainActivity.this, R.layout.item_listview, null);），一方面耗费内存（内存可能会溢出）、CPU，另一方面导致加载时间变长。

![img](http://bbs.itheima.com/data/attachment/forum/201506/25/162718ghotmcsiccuyotgh.png.thumb.jpg)

![img](http://bbs.itheima.com/data/attachment/forum/201506/25/162717e1qz3jq5uu2uvffg.png.thumb.jpg)

可以使用listView的缓存机制解决这个问题。任何一个条目如果被滑出屏幕，那么就缓存起来，下次再显示，不用重新创建，直接从缓存中提取就好。ListView对条目的缓存。在显示新的条目时，只要内存中有缓存，就会拿来用，不管是哪个条目的缓存。

示例：修改上面的示例代码，如下：src/cn.itcast.listView/MainActivity.java
```java
package cn.itcast.listView;

import java.util.ArrayList;
import java.util.List;

import android.app.Activity;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.os.Bundle;
import android.view.View;
import android.view.ViewGroup;
import android.widget.BaseAdapter;
import android.widget.ListView;
import android.widget.TextView;
import cn.itcast.listView.domain.Person;

public class MainActivity extends Activity {

        List<Person> personList;

        @Override
        protected void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                setContentView(R.layout.activity_main);

                personList = new ArrayList<Person>();

                MyOpenHelper oh = new MyOpenHelper(this);
                SQLiteDatabase db = oh.getWritableDatabase();

                Cursor cursor = db.query("person", null, null, null, null, null, null, null);

                while(cursor.moveToNext()){
                        int _id = cursor.getInt(0);
                        String name = cursor.getString(1);
                        String phone = cursor.getString(2);
                        int salary = cursor.getInt(3);

                        Person p = new Person(_id, name, phone, salary);
                        personList.add(p);
                }

                ListView lv = (ListView)findViewById(R.id.lv);
                lv.setAdapter(new MyAdapter());
        }

        class MyAdapter extends BaseAdapter{

                @Override
                public int getCount() {
                        return personList.size();
                }

                //convertView就是被缓存起来的View对象，系统每次调用getView方法，都会把convertView传递进来
                @Override
                public View getView(int position, View convertView, ViewGroup parent) {
                        System.out.println("getView调用" + position);
                        Person p = personList.get(position);

                        View v = null;

                            System.out.println(convertView);

                        if(convertView == null){
                                //如果没有缓存，填充新的View对象
                                v = View.inflate(MainActivity.this, R.layout.item_listview, null);
                        }else{
                                //如果有缓存，复用缓存
                                v = convertView;
                        }

                        //缓存并不是指给每个单独的条目缓存，系统是只要系统中有对象被缓存就会拿来使用。
                        //所以缓存的是View对象，并不是内容。也就是说系统中缓存的对象个数等于同屏能够显示的条目+1个。
                        TextView tv_name = (TextView)v.findViewById(R.id.tv_name);
                        tv_name.setText(p.getName());

                        TextView tv_phone = (TextView)v.findViewById(R.id.tv_phone);
                        tv_phone.setText(p.getPhone());

                        TextView tv_salary = (TextView)v.findViewById(R.id.tv_salary);
                        tv_salary.setText(p.getSalary() + "");

                        return v;
                }

                @Override
                public Object getItem(int position) {
                        return null;
                }

                @Override
                public long getItemId(int position) {
                        return 0;
                }
        }
}
```
运行结果：

![img](http://bbs.itheima.com/data/attachment/forum/201506/25/162715l1iz5wamp1ywiwwi.png.thumb.jpg)

![img](http://bbs.itheima.com/data/attachment/forum/201506/25/162716b2bhkbxhxszvxsbk.png.thumb.jpg)

可以看到，缓存了11个View对象之后，后来不会再创建新的的View对象了，而都是通过缓存获取的。

## 4.3 ArrayAdapter和SimpleAdapter
ListAdapter，我们用的最多的是BaseAdapter，因为它的四个方法是我们自己实现的，自由度比较大，做复杂列表的时候比较容易。但列表不是那么复杂的时候，ListAdapter有一些封装度更高的Adapter提供给我们使用，用起来会更方便，其中的getCount、getView都不需要我们自己写代码。

示例1：res\layout\activity_main.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal" >

    <ImageView
        android:id="@+id/iv"
        android:layout_width="40dp"
        android:layout_height="40dp"
        android:src="@drawable/ic_launcher"
        />

    <TextView
        android:id="@+id/tv"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="name"
        android:textSize="25sp"
        android:layout_gravity="center_vertical"
        />

</LinearLayout>
```
src/cn.itcast.simplearray/MainActivity.java
```java
package cn.itcast.simplearray;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import android.app.Activity;
import android.os.Bundle;
import android.widget.ArrayAdapter;
import android.widget.ListView;
import android.widget.SimpleAdapter;

public class MainActivity extends Activity {

        @Override
        protected void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                setContentView(R.layout.activity_main);

                String[] objects = new String[]{"白吃","沙比","亚聪"};

                ListView lv = (ListView)findViewById(R.id.lv);
                //使用ArrayAdapter只能处理文本数据，图片数据无法个性化单独设置，所以如果只是简单的文本显示，就是用ArrayAdapter。
                //第二个参数为资源文件，填充ListView条目的布局文件
                //第三个参数指定文本显示至哪一个textView中
                lv.setAdapter(new ArrayAdapter<String>(this, R.layout.item_listview, R.id.tv, objects));
        }
}
```
运行结果：

![img](http://bbs.itheima.com/data/attachment/forum/201506/25/162713x70ggoxxp9l6e73a.png.thumb.jpg)

示例2：src/cn.itcast.simplearray/MainActivity.java

```java
package cn.itcast.simplearray;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import android.app.Activity;
import android.os.Bundle;
import android.widget.ArrayAdapter;
import android.widget.ListView;
import android.widget.SimpleAdapter;

public class MainActivity extends Activity {

        @Override
        protected void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                setContentView(R.layout.activity_main);
    
                ListView lv = (ListView)findViewById(R.id.lv);
    
                //集合中的每一个元素都是一个条目要显示的数据，因为有多种数据类型，
                // 所以List不能使用单一泛型，先把所有类型的数据都封装至map，然后把map存入List。
                List<Map<String,Object>> data = new ArrayList<Map<String,Object>>();
    
                Map<String,Object> map1= new HashMap<String,Object>();
                map1.put("image", R.drawable.ic_launcher);
                map1.put("name", "白吃");
                data.add(map1);
    
                Map<String,Object> map2= new HashMap<String,Object>();
                map2.put("image", R.drawable.photo1);
                map2.put("name", "沙比");
                data.add(map2);
    
                Map<String,Object> map3= new HashMap<String,Object>();
                map3.put("image", R.drawable.photo2);
                map3.put("name", "亚聪");
                data.add(map3);
    
                lv.setAdapter(new SimpleAdapter(this, data, R.layout.item_listview,
                              new String[]{"image","name"}, new int[]{R.id.iv,R.id.tv}));
        }
}   
```
运行结果：

![img](http://bbs.itheima.com/data/attachment/forum/201506/25/162710n68g2gd2g8z2t2z5.png.thumb.jpg)

# 案例1：SQLite数据库

```java
public class MyOpenHelper extends SQLiteOpenHelper {

	public MyOpenHelper(Context context, String name, CursorFactory factory,
			int version) {
		super(context, name, factory, version);
		// TODO Auto-generated constructor stub
	}

	//数据库创建时，此方法会调用
	@Override
	public void onCreate(SQLiteDatabase db) {
		db.execSQL("create table person(_id integer primary key autoincrement, name char(10), salary char(20), phone integer(20))");

	}

	//数据库升级时，此方法会调用
	@Override
	public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
		System.out.println("数据库升级了");
	}
}
```

```java
public class TestCase extends AndroidTestCase {

	//此时测试框架还没有初始化完毕，没有虚拟上下文对象
//	private MyOpenHelper oh = new MyOpenHelper(getContext(), "people.db", null, 1);
	private MyOpenHelper oh;
	private SQLiteDatabase db;
	public void test(){
		//getContext():获取一个虚拟的上下文
		MyOpenHelper oh = new MyOpenHelper(getContext(), "people.db", null, 1);
		//如果数据库不存在，先创建数据库，再获取可读可写的数据库对象，如果数据库存在，就直接打开
		SQLiteDatabase db = oh.getWritableDatabase();
		//如果存储空间满了，那么返回只读数据库对象
//		SQLiteDatabase db = oh.getReadableDatabase();
	}

	//测试框架初始化完毕之后，在测试方法执行之前，此方法调用
	@Override
	protected void setUp() throws Exception {
		super.setUp();

		oh = new MyOpenHelper(getContext(), "people.db", null, 1);
		db = oh.getWritableDatabase();
	}

	//测试方法执行完毕之后，此方法调用
	@Override
	protected void tearDown() throws Exception {
		// TODO Auto-generated method stub
		super.tearDown();
		db.close();
	}

	public void insert(){

//		db.execSQL("insert into person (name, salary, phone)values(?, ?, ?)", new Object[]{"小志的老婆[1]", "13000", 138438});
//		db.execSQL("insert into person (name, salary, phone)values(?, ?, ?)", new Object[]{"小志的儿子", 14000, "13888"});
		db.execSQL("insert into person (name, salary, phone)values(?, ?, ?)", new Object[]{"小志", 14000, "13888"});
	}

	public void delete(){
		db.execSQL("delete from person where name = ?", new Object[]{"小志"});
	}

	public void update(){
		db.execSQL("update person set phone = ? where name = ?", new Object[]{186666, "小志的儿子"});
	}

	public void select(){
		Cursor cursor = db.rawQuery("select name, salary from person", null);

		while(cursor.moveToNext()){
			//通过列索引获取列的值
			String name = cursor.getString(cursor.getColumnIndex("name"));
			String salary = cursor.getString(1);
			System.out.println(name + ";" + salary);
		}
	}

	public void insertApi(){
		//把要插入的数据全部封装至ContentValues对象
		ContentValues values = new ContentValues();
		values.put("name", "游天龙");
		values.put("phone", "15999");
		values.put("salary", 16000);
		db.insert("person", null, values);
	}

	public void deleteApi(){
		int i = db.delete("person", "name = ? and _id = ?", new String[]{"小志的儿子", "3"});
		System.out.println(i);
	}

	public void updateApi(){
		ContentValues values = new ContentValues();
		values.put("salary", 26000);
		int i = db.update("person", values, "name = ?", new String[]{"游天龙"});
		System.out.println(i);
	}

	public void selectApi(){
		Cursor cursor = db.query("person", null, null, null, null, null, null, null);
		while(cursor.moveToNext()){
			String name = cursor.getString(cursor.getColumnIndex("name"));
			String phone = cursor.getString(cursor.getColumnIndex("phone"));
			String salary = cursor.getString(cursor.getColumnIndex("salary"));
			System.out.println(name + ";" + phone + ";" + salary);
		}
	}

	public void transaction(){
		try{
			//开启事务
			db.beginTransaction();
			ContentValues values = new ContentValues();
			values.put("salary", 12000);
			db.update("person", values, "name = ?", new String[]{"小志"});

			values.clear();
			values.put("salary", 16000);
			db.update("person", values, "name = ?", new String[]{"小志的儿子"});

			int i = 3/0;
			//设置  事务执行成功
			db.setTransactionSuccessful();
		}
		finally{
			//关闭事务，同时提交，如果已经设置事务执行成功，那么sql语句就生效了，反之，sql语句回滚
			db.endTransaction();
		}
	}
}
```
# 案例2：把数据显示至屏幕

```java
public class MainActivity extends Activity {

	List<Person> personList;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        personList = new ArrayList<Person>();
        //把数据库的数据查询出来
        MyOpenHelper oh = new MyOpenHelper(this);
        SQLiteDatabase db =  oh.getWritableDatabase();
        Cursor cursor = db.query("person", null, null, null, null, null, null, null);
        while(cursor.moveToNext()){
        	String _id = cursor.getString(0);
        	String name = cursor.getString(1);
        	String salary = cursor.getString(2);
        	String phone = cursor.getString(3);

        	Person p = new Person(_id, name, phone, salary);
        	personList.add(p);
        }

        LinearLayout ll = (LinearLayout) findViewById(R.id.ll);
        //把数据显示至屏幕
        for (Person p : personList) {
        	//1.集合中每有一条元素，就new一个textView
			TextView tv = new TextView(this);
			//2.把人物的信息设置为文本框的内容
			tv.setText(p.toString());
			tv.setTextSize(18);
			//3.把textView设置为线性布局的子节点
			ll.addView(tv);
		}
    }

}

```
# 案例3：使用ListView把数据显示至屏幕

```java
public class MainActivity extends Activity {

	List<Person> personList;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        personList = new ArrayList<Person>();
        //把数据库的数据查询出来
        MyOpenHelper oh = new MyOpenHelper(this);
        SQLiteDatabase db =  oh.getWritableDatabase();
        Cursor cursor = db.query("person", null, null, null, null, null, null, null);
        while(cursor.moveToNext()){
        	String _id = cursor.getString(0);
        	String name = cursor.getString(1);
        	String salary = cursor.getString(2);
        	String phone = cursor.getString(3);

        	Person p = new Person(_id, name, phone, salary);
        	personList.add(p);
        }

        ListView lv = (ListView) findViewById(R.id.lv);
        lv.setAdapter(new MyAdapter());
    }

    class MyAdapter extends BaseAdapter{

    	//系统调用，用来获知集合中有多少条元素
		@Override
		public int getCount() {
			return personList.size();
		}

		//由系统调用，获取一个View对象，作为ListView的条目
		//position:本次getView方法调用所返回的View对象，在listView中是处于第几个条目，那么position的值就是多少
		@Override
		public View getView(int position, View convertView, ViewGroup parent) {
			Person p = personList.get(position);

//			TextView tv = new TextView(MainActivity.this);
			System.out.println("getView调用：" + position + ";" + convertView);
//			tv.setText(p.toString());
//			tv.setTextSize(18);

			View v = null;
			//判断条目是否有缓存
			if(convertView == null){
				//把布局文件填充成一个View对象
				v = View.inflate(MainActivity.this, R.layout.item_listview, null);
			}
			else{
				v = convertView;
			}


			//获取布局填充器对象
//			LayoutInflater inflater = LayoutInflater.from(MainActivity.this);
//			使用布局填充器填充布局文件
//			View v2 = inflater.inflate(R.layout.item_listview, null);

//			LayoutInflater inflater2 = (LayoutInflater) getSystemService(LAYOUT_INFLATER_SERVICE);
//			View v3 = inflater2.inflate(R.layout.item_listview, null);

			//通过资源id查找组件,注意调用的是View对象的findViewById
			TextView tv_name = (TextView) v.findViewById(R.id.tv_name);
			tv_name.setText(p.getName());
			TextView tv_phone = (TextView) v.findViewById(R.id.tv_phone);
			tv_phone.setText(p.getPhone());
			TextView tv_salary = (TextView) v.findViewById(R.id.tv_salary);
			tv_salary.setText(p.getSalary());
			return v;
		}

		@Override
		public Object getItem(int position) {
			return null;
		}

		@Override
		public long getItemId(int position) {
			return 0;
		}
  }
}

```

# 案例4：ArrayAdapter&SimpleAdapter

```java
public class MainActivity extends Activity {

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);

		String[] objects = new String[]{
				"小志",
				"小志的儿子",
				"萌萌"
		};

		ListView lv = (ListView) findViewById(R.id.lv);
//		lv.setAdapter(new ArrayAdapter<String>(this, R.layout.item_listview, R.id.tv_name, objects));

		//集合中每个元素都包含ListView条目需要的所有数据,该案例中每个条目需要一个字符串和一个整型,所以使用一个map来封装这两种数据
		List<Map<String, Object>> data = new ArrayList<Map<String,Object>>();

		Map<String, Object> map1 = new HashMap<String, Object>();
		map1.put("photo", R.drawable.photo1);
		map1.put("name", "小志的儿子");
		data.add(map1);

		Map<String, Object> map2 = new HashMap<String, Object>();
		map2.put("photo", R.drawable.photo2);
		map2.put("name", "小志");
		data.add(map2);

		Map<String, Object> map3 = new HashMap<String, Object>();
		map3.put("photo", R.drawable.photo3);
		map3.put("name", "赵帅哥");
		data.add(map3);

		lv.setAdapter(new SimpleAdapter(this, data, R.layout.item_listview,
				new String[]{"photo", "name"}, new int[]{R.id.iv_photo, R.id.tv_name}));
	}

}
```

# 案例5：对话框

```java
public class MainActivity extends Activity {

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
	}


	public void click1(View v){
		AlertDialog.Builder builder = new Builder(this);
		//设置图标
		builder.setIcon(android.R.drawable.alert_dark_frame);
		//设置标题
		builder.setTitle("欲练此功必先自宫");
		//设置文本
		builder.setMessage("李志平，想清楚哦");

		//设置确定按钮
		builder.setPositiveButton("确定", new OnClickListener() {

			@Override
			public void onClick(DialogInterface dialog, int which) {
				Toast.makeText(MainActivity.this, "感谢使用本软件,再见", 0).show();
			}
		});

		//设置取消按钮
		builder.setNegativeButton("取消", new OnClickListener() {

			@Override
			public void onClick(DialogInterface dialog, int which) {
				Toast.makeText(MainActivity.this, "若不自宫,一定不成功", 0).show();
			}
		});
		//使用创建器,生成一个对话框对象
		AlertDialog ad = builder.create();
		ad.show();
	}

	public void click2(View v){
		AlertDialog.Builder builder = new Builder(this);
		builder.setTitle("请选择性别");
		final String[] items = new String[]{
				"男",
				"女"
		};

		builder.setSingleChoiceItems(items, -1, new OnClickListener() {

			//which:用户所选的条目的下标
			//dialog:触发这个方法的对话框
			@Override
			public void onClick(DialogInterface dialog, int which) {
				Toast.makeText(MainActivity.this, "您选择的是:" + items[which], 0).show();
				//关闭对话框
				dialog.dismiss();
			}
		});
		builder.show();
	}

	public void click3(View v){
		AlertDialog.Builder builder = new Builder(this);
		builder.setTitle("请选择您觉得帅的人");
		final String[] items = new String[]{
				"侃哥",
				"赵帅哥",
				"赵老师",
				"赵师兄"
		};
		final boolean[] checkedItems = new boolean[]{
				true,
				true,
				false,
				false

		};

		builder.setMultiChoiceItems(items, checkedItems, new OnMultiChoiceClickListener() {

			//which:用户点击的条目的下标
			//isChecked:用户是选中该条目还是取消该条目
			@Override
			public void onClick(DialogInterface dialog, int which, boolean isChecked) {
				checkedItems[which] = isChecked;

			}
		});

		//设置一个确定按钮
		builder.setPositiveButton("确定", new OnClickListener() {

			@Override
			public void onClick(DialogInterface dialog, int which) {
				String text = "";
				for(int i = 0; i < 4; i++){
					text += checkedItems[i]? items[i] + "," : "";
				}
				Toast.makeText(MainActivity.this, text, 0).show();
				dialog.dismiss();
			}
		});
		builder.show();
	}
}
```
