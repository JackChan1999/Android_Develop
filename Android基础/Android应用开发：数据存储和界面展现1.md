# 1. 相对布局RelativeLayout

特点：相对布局所有组件可以叠加在一起；各个组件的布局是独立的，互不影响；所有组件的默认位置都是在左上角（顶部、左部对齐）

| 属性                               | 功能描述       |
| :------------------------------- | :--------- |
| android:layout_toRightOf         | 在指定控件的右边   |
| android:layout_toLeftOf          | 在指定控件的左边   |
| android:layout_above             | 在指定控件的上边   |
| android:layout_below             | 在指定控件的下边   |
| android:layout_alignBaseline     | 跟指定控件水平对齐  |
| android:layout_alignLeft         | 跟指定控件左对齐   |
| android:layout_alignRight        | 跟指定控件右对齐   |
| android:layout_alignTop          | 跟指定控件顶部对齐  |
| android:layout_alignBottom       | 跟指定控件底部对齐  |
| android:layout_alignParentLeft   | 是否跟父元素左对齐  |
| android:layout_alignParentTop    | 是否跟父元素顶部对齐 |
| android:layout_alignParentRight  | 是否跟父元素右对齐  |
| android:layout_alignParentBottom | 是否跟父元素底部对齐 |
| android:layout_centerVertical    | 在父元素中垂直居中  |
| android:layout_centerHorizontal  | 在父元素中水平居中  |
| android:layout_centerInParent    | 在父元素中居中    |

示例1：res\layout\activity_main.xml
```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity" >
    <!-- android:layout_alignParentRight="true"表示与父元素右对齐（这里，父元素指的就是RelativeLayout，而RelativeLayout占满了整个屏幕） -->
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="第一个" 
        android:layout_alignParentRight="true"/>
    <!-- android:layout_alignParentBottom="true"表示与父元素底部对齐（这里，父元素指的就是RelativeLayout，而RelativeLayout占满了整个屏幕） -->
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="第二个" 
        android:layout_alignParentBottom="true"/>
    <!-- android:layout_centerVertical="true"表示在父元素中垂直居中（这里，父元素指的就是RelativeLayout，而RelativeLayout占满了整个屏幕） -->
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="第三个" 
        android:layout_centerVertical="true"
        />
    <!-- android:layout_centerHorizontal="true"表示在父元素中水平居中（这里，父元素指的就是RelativeLayout，而RelativeLayout占满了整个屏幕） -->
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="第四个" 
        android:layout_centerHorizontal="true"
        />
    <!-- android:layout_centerInParent="true"表示在父元素中水平垂直都居中（这里，父元素指的就是RelativeLayout，而RelativeLayout占满了整个屏幕） -->
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="第五个" 
        android:layout_centerInParent="true"
        />
</RelativeLayout>
```
运行结果：

![img](http://bbs.itheima.com/data/attachment/forum/201506/22/115728hj4jjjfn7b48jj8f.png.thumb.jpg) 

示例2：res\layout\activity_main.xml
```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity" >
    <TextView
        android:id="@+id/tv1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="第一一一一个" />
    <!-- android:layout_alignRight="true"表示与指定组件右对齐。 -->
    <!-- @+id/就是为组件添加id，@id就是通过id引用组件 -->
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="第二个" 
        android:layout_centerVertical="true"
        android:layout_alignRight="@id/tv1"
        />
    <!-- 如果同时使用android:layout_alignRight="true"，android:layout_alignLeft="true"，组件就会被拉伸 -->
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="第三个" 
        android:layout_alignParentBottom="true"
        android:layout_alignRight="@id/tv1"
        android:layout_alignLeft="@id/tv1"
        />
</RelativeLayout>  
```
运行结果：

![img](http://bbs.itheima.com/data/attachment/forum/201506/22/120547e5iokc4vpuuodb99.png.thumb.jpg) 

示例3：res\layout\activity_main.xml
```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity" >

    <TextView
        android:id="@+id/tv1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="第一个"
        android:layout_centerHorizontal="true"/>

    <!-- android:layout_below="@id/tv1"表示在对应组件的下面 -->
    <TextView
        android:id="@+id/tv2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="第二个"
        android:layout_below="@id/tv1"
        />

    <!-- android:layout_above="@id/tv2"表示在对应组件的上面 -->
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="第三个"
        android:layout_above="@id/tv2"
        />

    <!-- android:layout_toRightOf="@id/tv2"表示在对应组件的右边 -->
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="第四个"
        android:layout_toRightOf="@id/tv2"
        />
</RelativeLayout>
```
运行结果：

![img](http://bbs.itheima.com/data/attachment/forum/201506/22/121319qs2hm8miio8vmo7p.png.thumb.jpg) 

练习：实现如下效果

![img](http://bbs.itheima.com/data/attachment/forum/201506/22/125152c7wjasfdyeszfws9.png.thumb.jpg) 

代码：res\layout\activity_main.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >
    <Button 
        android:id="@+id/center"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="中间"
        android:layout_centerInParent="true"/>
    <Button 
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="上边"
        android:layout_above="@id/center"
        android:layout_alignLeft="@id/center"
        android:layout_alignRight="@id/center"/>
    <Button 
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="下边"
        android:layout_below="@id/center"
        android:layout_alignLeft="@id/center"
        android:layout_alignRight="@id/center"/>
    <Button 
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="左边"
        android:layout_toLeftOf="@id/center"
        android:layout_alignTop="@id/center"
        android:layout_alignBottom="@id/center"
        android:layout_alignParentLeft="true"/>
    <Button 
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="右边"
        android:layout_toRightOf="@id/center"
        android:layout_alignTop="@id/center"
        android:layout_alignBottom="@id/center"
        android:layout_alignParentRight="true"/>
        <Button 
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="左上"
        android:layout_toLeftOf="@id/center"
        android:layout_above="@id/center"
        android:layout_alignParentLeft="true"/>
        <Button 
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="左下"
        android:layout_toLeftOf="@id/center"
        android:layout_below="@id/center"
        android:layout_alignParentLeft="true"/>
        <Button 
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="右上"
        android:layout_toRightOf="@id/center"
        android:layout_above="@id/center"
        android:layout_alignParentRight="true"/>
        <Button 
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="右下"
        android:layout_toRightOf="@id/center"
        android:layout_below="@id/center"
        android:layout_alignParentRight="true"/>
</RelativeLayout>
```
运行结果：

![img](http://bbs.itheima.com/data/attachment/forum/201506/22/124954vltl1aa1t8l5mnf5.png.thumb.jpg) 

# 2. 帧布局FrameLayout

特点：帧布局和相对布局一样，组件可以重叠；所有组件的默认位置是在左上角（顶部、左部对齐）
示例：res\layout\activity_main.xml
```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity" >
    <!-- 帧布局中，修改组件的位置使用android:layout_gravity -->
    <TextView 
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="第一个"
        android:layout_gravity="right"
        />
    <!-- android:layout_gravity="bottom|right" ，表示组件的位置在右下角 -->
    <TextView 
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="第二个"
        android:layout_gravity="bottom|right" 
        />
    <TextView 
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="第三个"
        android:layout_gravity="center" 
        />
</FrameLayout>  
```
运行结果：

![img](http://bbs.itheima.com/data/attachment/forum/201506/22/131052y3hxnn3za0xlfa73.png.thumb.jpg) 

练习：实现如下效果

![img](http://bbs.itheima.com/data/attachment/forum/201506/22/131636h3bwzekgxeleezex.png.thumb.jpg) 

代码：res\layout\activity_main.xml
```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity" >
    <TextView 
        android:layout_width="240dp"
        android:layout_height="240dp"
        android:background="#ff0000"
        android:layout_gravity="center"
        />
    <TextView 
        android:layout_width="200dp"
        android:layout_height="200dp"
        android:background="#00ff00"
        android:layout_gravity="center"
        />
    <TextView 
        android:layout_width="160dp"
        android:layout_height="160dp"
        android:background="#0000ff"
        android:layout_gravity="center"
        />
    <TextView 
        android:layout_width="120dp"
        android:layout_height="120dp"
        android:background="#ffff00"
        android:layout_gravity="center"
        />
    <TextView 
        android:layout_width="80dp"
        android:layout_height="80dp"
        android:background="#ff00ff"
        android:layout_gravity="center"
        />
    <TextView 
        android:layout_width="40dp"
        android:layout_height="40dp"
        android:background="#ffffff"
        android:layout_gravity="center"
        />
</FrameLayout>   
```
运行结果：

![img](http://bbs.itheima.com/data/attachment/forum/201506/22/131635rzw8wecs6wcw03xw.png.thumb.jpg) 

# 3. 表格布局TableLayout

示例：res\layout\activity_main.xml
```xml
<TableLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity" >
    <!-- 表格布局宽高可以不用设置，TableRow的宽度只能是match_parent，高度只能是wrap_content。调正宽度可以通过权重解决。-->
    <TableRow>
        <TextView 
                android:text="姓名："/>
        <EditText
            android:layout_weight="1"
            />
    </TableRow>
    <TableRow>
        <TextView 
                android:text="年龄："/>
        <EditText
            android:layout_weight="1"
            />
    </TableRow>
</TableLayout>*复制代码*   运行结果：
```
![img](http://bbs.itheima.com/data/attachment/forum/201506/22/185319l1pqm4m0z0mp0x4g.png.thumb.jpg) 

# 4. 绝对布局AbsoluteLayout
绝对布局是直接使用android:layout_x，android:layout_y定位控件的坐标，做不了屏幕适配，所以不常使用。某些没有必要做屏幕适配的开发可以用绝对布局，例如：电视屏幕固定，做机顶盒开发。

示例：res\layout\activity_main.xml
```xml
<AbsoluteLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity" >
    <Button
        android:id="@+id/button1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_x="109dp"
        android:layout_y="83dp"
        android:text="Button" />
</AbsoluteLayout>*复制代码*   运行结果：
```
![img](http://bbs.itheima.com/data/attachment/forum/201506/22/190358c1zx1qqq0qqqkoux.png.thumb.jpg) 

# 4. LogCat
Console只能显示Windows下运行的平台信息，例如：模拟器的运行状态（模拟器是运行在Windows平台上的程序）。但模拟器内的程序运行状态就不能显示在Console上了，因为这些程序不是运行到Windows上，这些信息会在LogCat中显示。

LogCat分为5个等级，依次为：error（错误）、warn（情报）、info（信息）、debug（调试）、verbose（冗余）

![img](http://bbs.itheima.com/data/attachment/forum/201506/22/192030odr888e5e3h8544d.png.thumb.jpg) 

示例：为LogCat添加过滤器，便于筛选信息

src/cn.itcast.logcat/MainActivity.java 
```java
package cn.itcast.logcat;
import android.os.Bundle;
import android.app.Activity;
import android.util.Log;
import android.view.Menu;
public class MainActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        String tag = "黑马程序员";
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        System.out.println("云鹤，儿童节快乐");
        //tag表示标签，msg表示正文
        //下面这种写法是在日志需要长期保留时使用。
        Log.v(tag, "云鹤就业薪资13000");
        Log.d(tag, "云鹤就业薪资13000");
        Log.i(tag, "云鹤就业薪资13000");
        Log.w(tag, "云鹤就业薪资13000");
        Log.e(tag, "云鹤就业薪资13000");
   }
}
```
运行结果：

![img](http://bbs.itheima.com/data/attachment/forum/201506/22/193814hast70nk0nngn7gn.png.thumb.jpg) 

第一步：点击加号

![img](http://bbs.itheima.com/data/attachment/forum/201506/22/192132n398757ajebg3m8b.png.thumb.jpg) 

第二步：填入过滤器设置信息。

![img](http://bbs.itheima.com/data/attachment/forum/201506/22/192453jj8f8lemyxamo888.png.thumb.jpg)

第三步：进行过滤。

![img](http://bbs.itheima.com/data/attachment/forum/201506/22/192454x3jij7j73edxje9z.png.thumb.jpg) 

# 5. 在内部存储中读写文件

Android内存

- RAM：运行内存，相当于电脑内存。关机，数据就会丢失。
- ROM：内部存储空间，Android系统必须的存储空间，持久化保持数据，相当于电脑的硬盘。
- SD卡/USB存储器（内置在手机内）：外部存储空间，对于Android系统是可有可无的，相当于移动硬盘。
- 内部存储路径：data/data/包名文件夹/，每个包名文件夹都是一个应用的专属空间。

![img](http://bbs.itheima.com/data/attachment/forum/201506/22/204224ojwfoone0sc0klbl.png.thumb.jpg) 

示例：res\layout\activity_main.xml
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
    <EditText
        android:id="@+id/et_name"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>
    <!-- android:inputType="textPassword"表示输入的是密码，显示时用密文代替 -->
    <EditText
        android:id="@+id/et_pass"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:inputType="textPassword"/>
    <RelativeLayout 
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
            <CheckBox 
                android:id="@+id/cb"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="记住账号密码"
                android:layout_centerVertical="true"
                />
            <Button 
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="登陆"
                android:onClick="login"
                android:layout_alignParentRight="true"
                />
        </RelativeLayout>
</LinearLayout>
```
src/cn.itcast.rwinrom/MainActivity.java
```java
package cn.itcast.rwinrom;
import java.io.BufferedReader;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.InputStreamReader;
import android.app.Activity;
import android.os.Bundle;
import android.view.View;
import android.widget.CheckBox;
import android.widget.EditText;
import android.widget.Toast;
public class MainActivity extends Activity {
    private EditText et_name;
    private EditText et_pass;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        et_name = (EditText) findViewById(R.id.et_name);
        et_pass = (EditText) findViewById(R.id.et_pass);
        readAccount();
    }
    public void login(View v){
            //获取用户输入的数据
            String name = et_name.getText().toString();
            String pass = et_pass.getText().toString();
            CheckBox cb = (CheckBox) findViewById(R.id.cb);
            if(cb.isChecked()){
                    File file = new File("data/data/cn.itcast.rwinrom/info.txt");
                    try{
                            FileOutputStream fos = new FileOutputStream(file);
                            fos.write((name + "##" + pass).getBytes());
                            fos.close();
                    }catch(Exception e){
                            e.printStackTrace();
                    }
            }
            //弹出吐司对话框
            //Activity是Context的子类
            //Toast.LENGTH_LONG表示显示5秒
            //Toast.LENGTH_SHORT表示显示3秒
            Toast.makeText(this, "登陆成功", Toast.LENGTH_LONG).show();
    }
    public void readAccount(){
            File file = new File("data/data/cn.itcast.rwinrom/info.txt");
            if(file.exists()){
                    try{
                            FileInputStream fis = new FileInputStream(file);
                            //把字节流转换成字符流
                            BufferedReader br = new BufferedReader(new InputStreamReader(fis));
                            String text = br.readLine();
                            String[] s = text.split("##");
                            et_name.setText(s[0]);
                            et_pass.setText(s[1]);
                            fis.close();
                    }catch(Exception e){
                            e.printStackTrace();
                    }
            }
    }
}
```
运行结果：

1、在模拟器上运行应用程序，输入用户名、密码，勾上记住账号密码，点击登陆。可以看到弹出吐司，登陆成功。

![img](http://bbs.itheima.com/data/attachment/forum/201506/22/211019om3z32h7hx38x9xw.png.thumb.jpg) 

2、在Android设备文件目录下，可以看到成功生成info.txt文件，点击右上角的“pull a file from the device”按钮，将info.txt提取出来，保存到桌面。

![img](http://bbs.itheima.com/data/attachment/forum/201506/22/211022rkah687x86xornqi.png.thumb.jpg) 

3、打开info.txt文件，可以看到成功存入数据。

![img](http://bbs.itheima.com/data/attachment/forum/201506/22/211024uh7sbw34zvt8h7zf.png.thumb.jpg) 

4、重新运行应用程序，可以看到读取数据成功，用户名、密码成功显示。

![img](http://bbs.itheima.com/data/attachment/forum/201506/22/211026fc7m2b2tcarb23a7.png.thumb.jpg) 

1、点击返回键，会摧毁当前Activity，重新点击应用图标，会重新创建Activity。
2、点击HOME键，只会将Activity放到后台去，重新点击图标，只是重新提取出来。

所以，上面的应用程序，即使是第一次运行，输入用户名、密码，并且不勾选“记住账户密码”。如果此时点击HOME键，Activity消失。然后，再点击应用程序图标，竟然可以看到Activity显示出了用户名、密码！然而，这根本不是回显，Activity只是暂时放到后台，然后重新提取出来罢了。

3、在内部存储空间中读写不需要任何权限！

## 5.1 使用API获取内部存储空间的路径

如果将上面应用程序的MainActivity类中的代码进行如下修改，可以看到后台报警告。原因是不能访问其他应用程序的专属空间，只能访问自己的。

![img](http://bbs.itheima.com/data/attachment/forum/201506/22/212732ntve0nknq2rx8ktn.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201506/22/214844k3bllzzbt230blui.png.thumb.jpg) 

为了防止内部存储空间不小心写错，API提供了获取内部存储空间路径的方法。修改后，代码如下：

![img](http://bbs.itheima.com/data/attachment/forum/201506/22/215529m8e0ioo8gmne0asa.png.thumb.jpg)

![img](http://bbs.itheima.com/data/attachment/forum/201506/22/215530nkam4ol2jkmbbrby.png.thumb.jpg) 

运行结果，如下：

![img](http://bbs.itheima.com/data/attachment/forum/201506/22/215637cminvhmhu6v9bvx7.png.thumb.jpg) 

getFilesDir方法所在的类为上下文包装类，ContextWrapper。

![img](http://bbs.itheima.com/data/attachment/forum/201506/22/215847o6bqnkoz29asb3yh.png.thumb.jpg) 

并且，Activity就是ContextWrapper的子类。

![img](http://bbs.itheima.com/data/attachment/forum/201506/22/220008lr4ybzvv5u4orbyy.png.thumb.jpg) 

所谓的上下文就是一个应用环境全局信息的接口，也就是说，通过上下文的API可以拿到应用环境的全局信息，例如：包名、版本号、内部存储等信息。
Toast.makeText(this,"登陆成功",0).show();语句中的this就是上下文，表示吐司需要显示在哪个上下文中。
Button bt = new Button(this);语句中的this也是上下文，表示该组件创建出来之后，显示在哪个上下文中。
谷歌的API还提供了另一个方法获取内部存储空间路径：getCacheDir()，它返回的路径为data/data/cn.itcast.rwinrom/cache，得到的是缓存路径，也就是cache文件夹路径。修改后，代码如下：

![img](http://bbs.itheima.com/data/attachment/forum/201506/22/221029kq2g5nleev5ezla9.png.thumb.jpg) 

运行结果，如下：

![img](http://bbs.itheima.com/data/attachment/forum/201506/22/221030jhitynss9qthikxs.png.thumb.jpg) 

区别：

- getFilesDir方法返回的路径中的files文件夹是持久化保存数据的。
- getCacheDir方法返回的路径中的Cache文件夹是保存缓存数据的。当设备内存可用存储空间比较少时，Cache中的文件可能会被系统删除。

# 6. 在外部存储中读写文件

SD卡路径为storage/sdcard（4.3版本），之所以还存在根目录sdcard（2.3版本之前）及mnt/sdcard（2.3版本~4.3版本），是因为早期版本的sd卡路径正是在根目录，后期移到了mnt/sdcard，为了与低版本兼容，所以现在他们还存在，其实只是快捷方式而已，指向的路径依然是storage/sdcard。

![img](http://bbs.itheima.com/data/attachment/forum/201506/22/223312yci74fbazb5iabzh.png.thumb.jpg) 

往SD卡中写数据需要权限。而从SD卡中读取数据在4.0版本以前是不需要权限，后来谷歌在Android系统中添加了保护SD卡的选项（如下图）。如果勾选了，那么在真机上是有效的（也就是说，模拟器上是无效的）。不过，一般情况下，不勾选。因为，很多应用程序并没有申请读取SD卡的权限，一旦勾上，很多应用程序将无法读取SD卡。因此，为了向下兼容，一般不勾。

![img](http://bbs.itheima.com/data/attachment/forum/201506/22/224058yqvfjhlbafovvgo1.png.thumb.jpg) 

示例：将上面“在内部存储中读取文件”中示例的MainActivity类中代码做一下修改，如下。

![img](http://bbs.itheima.com/data/attachment/forum/201506/22/224349bd22ddv1q17e2e77.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201506/22/224349cs07n09nwnj0z9ng.png.thumb.jpg) 

为读写SD卡添加权限（读SD卡权限可加可不加）：

![img](http://bbs.itheima.com/data/attachment/forum/201506/22/223313qdx8i8q1zi1lmqh1.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201506/22/224721ap33rplbbubbb35v.png.thumb.jpg) 

运行结果：

![img](http://bbs.itheima.com/data/attachment/forum/201506/22/224915ry7lr7aoynywfyrg.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201506/22/224917udrj0ys96p4mm3tc.png.thumb.jpg) 

## 6.1 检测SD卡状态

很多手机厂商都修改了SD卡路径，因此，很多手机SD卡的路径不是storage/sdcard。但是，sdcard的快捷方式基本上都是存在的，并且指向真实的SD卡路径。因此，访问SD卡路径，可以通过获取SD卡快捷方式的方法。当然，也可以调用API中提供的相应方法获取SD卡路径：Environment.getExternalStorageDirectory()

外部存储空间对于手机来说并不一定是必须的。所以，在使用前，一定要做确认手机的SD卡是否存在。可以通过Environment.getExternalStorageState()确定SD卡的状态，此方法的常见返回值说明如下：

- MEDIA_REMOVED：sd卡被拔出
- MEDIA_UNMOUNTED：sd卡未挂载（4.0版本以前，手机可以通过点击settings-->Storage-->Storage settings-->Unmounted SD card，使SD卡未挂载）  
- MEDIA_- CHECKING：sd卡正在准备   
- MEDIA_MOUNTED：sd卡已挂载，当前可用  
- MEDIA_READ_ONLY：sd卡挂载可用，但是只读

示例：将上面“在内部存储中读取文件”中示例的MainActivity类中代码做一下修改，如下

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/100721itwjpymotp70pyo7.png.thumb.jpg) 

运行结果：由于在4.0版本之前才有手动使SD卡未挂载的功能，这里使用2.3.3版本测试。首先，Unmounted SD card，观察效果，可以看到提示：sd不可用。

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/101941l2mvk2fllu0aknd4.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/101842jz16f4gieuo86828.png.thumb.jpg) 

然后，mounted SD card，观察效果，成功保存用户名、密码信息。

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/102239ej34240d2zl24sdz.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/102426bkzrrgoy1mwp1ypr.png.thumb.jpg) 

当有多个模拟器运行时，想要观察某一个模拟器的文件目录，只需要在Devices选项卡中点击那个模拟器，即可在File Explorer选项卡中看到那个模拟器的文件目录。

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/102633qoj9d0q59zv5q205.png.thumb.jpg)

## 6.2 获取SD卡剩余容量
当用户做下载等操作时，SD卡的空间可能不足，所以需要提前获取SD卡剩余容量，给予用户提示信息。在Android系统中，可以查看SD卡剩余容量，点击Setting-->Storage

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/103126cqrefwdj8x8j8p0e.png.thumb.jpg) 

sdk文件夹中的sources文件夹中的源码是Android jar包的源码，Android系统源码则不同，一定要区分开。如下图。

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/110723bmsbksk466448788.png.thumb.jpg) 

其中，packages/apps路径为Android系统级应用程序所在的目录，可以看到settings应用程序。

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/110726wuouvxazuzzvoasn.png.thumb.jpg) 

# 7. 查看settings应用程序的源码

## 7.1 导入settings应用程序到eclipse中
点击File-->New-->Other....

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/111045c8ybblzkilz8145l.png.thumb.jpg) 

点击Android下的Android Project from Existing Code-->Next

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/111212sdq3jjq6dong51hn.png.thumb.jpg) 

点击Root Directory右侧的Browse...按钮，找到settings应用程序所在的目录，点击确定

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/111453injjx1u686u6n62a.png.thumb.jpg) 

下图中的tests为测试settings的应用程序，不勾

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/111454qht6cft6ktfch1dt.png.thumb.jpg)

导入成功

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/111711q8wlfc1p515w5l8m.png.thumb.jpg) 

之所以会报错，是因为普通应用开发是无法访问系统级API的（settings工程导入eclipse就成为了一个普通应用开发项目）

## 7.2 查找实现查看SD卡剩余空间的源码

通过搜索关键字“Available space”开始查找

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/115928af986skkx8g6gdjd.png.thumb.jpg) 

通过eclipse的“File Search”在整个工作空间中搜索“Available space”

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/120116q08ay0058bmb1wka.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/121234i2cpp5sf5pxia6ul.png.thumb.jpg)

查找到以后，双击

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/121414lw89vil8ivl0989r.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/121506fd6lxt45t5u35ymk.png.thumb.jpg) 

再搜索“memory_available”

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/131110fvz12paar2l1asgr.png.thumb.jpg)

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/131112y02oo619k1i99bnc.png.thumb.jpg) 

再搜索“memory_sd_avail”

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/131459jtj14fnhd1ujucdd.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/131501cuqarmouwvoaad4k.png.thumb.jpg) 

再在本文件内搜索“MEMORY_SD_AVAIL”

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/131503v7j2j84hsl478ily.png.thumb.jpg) 

再在本文件内搜索“mSdAvail”，可以看到，已经查找到了该段代码

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/131504lz8lrsoor56aapqr.png.thumb.jpg) 

## 7.3 解析源码

所有存储设备都会被划分成若干个区块，存储设备的总容量 = 每个区块大小 * 区块总数量

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/145339otp91a39gtrqmnzu.png.thumb.jpg) 

其中的formatSize函数实际上就是将得出的字节数，转换成MB、GB、TB及PB显示给用户

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/145558avhpp2o5kqhkjcvg.png.thumb.jpg) 

示例：我们自己写程序获取SD卡剩余容量

res\layout\activity_main.xml
```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context=".MainActivity" >
    <TextView
        android:id="@+id/tv"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/hello_world" />
</RelativeLayout>
```
src/cn.itcast.getsdavail/MainActivity.java
```java
package cn.itcast.getsdavail;
import java.io.File;
import android.os.Build;
import android.os.Bundle;
import android.os.Environment;
import android.os.StatFs;
import android.app.Activity;
import android.text.format.Formatter;
import android.view.Menu;
import android.widget.TextView;
public class MainActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        File path = Environment.getExternalStorageDirectory();
        StatFs stat = new StatFs(path.getPath());
        long blockSize;
        long availableBlocks;
        //下面的很多方法已经过时了，因为getBlockSize等方法的返回值都是int类型，如果设备较大，返回的blockSize数值可能超过int类型的范围，导致溢出。
        //改为使用最新的getBlockSizeLong等方法返回为long类型数据即可。由于，只有4.3以上版本的API才支持这些方法。因此，最好做一下判断。
```
运行结果：

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/151413km3g1puusg3pjhnk.png.thumb.jpg) 

# 8. 文件访问权限
Android的文件访问权限是从Linux继承来的

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/105750aaabgyvb4u6llygb.png.thumb.jpg) 

drwxrwxrwx:

第一个字母：如果是d，表示文件夹；如果是-，表示文件；如果是l，表示快捷方式。   
三组rwx（r（read），w（write），x（exeute，执行））表示三种不同用户的权限。   
在Android中，每个应用，都是一个独立的用户。也就是说，创建一个应用，就是创建了一个新的用户。删除一个应用，就是删除了一个用户。   
第一组rwx：描述文件拥有者（owner）对文件的权限。例如，上图中的data/cn.itcast.rwinrom/cache/info.txt文件的创建者就是拥有者，也就是cn.itcast.rwinrom应用程序。   
第二组rwx：描述与文件拥有者同一用户组（grouper）的用户对文件的权限。默认情况，各个应用程序彼此是独立的用户。可以手动设置同组用户，不过只有系统应用为了方便才会设置同组用户，我们不用管。   
第三组rwx：描述其他用户（other）对文件的权限。其他应用程序就可以称为某一个应用程序的其他用户。

实验：其他应用程序读取当前应用程序的文件。

res\layout\activity_main.xml
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity" 
    android:orientation="vertical">
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:onClick="click1"
        android:text="按钮一" />
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:onClick="click2"
        android:text="按钮二" />
</LinearLayout>
```
src/cn.itcast.permission/MainActivity.java
```java
package cn.itcast.permission;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import android.app.Activity;
import android.os.Bundle;
import android.view.View;
public class MainActivity extends Activity {
        @Override
        protected void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                setContentView(R.layout.activity_main);
        }
    public void click1(View v){
            //Android中提供的openFileOutput方法可以直接获取FileOutputStream，文件会被写在内部文件空间中（data/data/报名文件夹/files/）
            try {
                        //MODE_PRIVATE表示拥有者和同组用户可以读写，其他用户无法访问
                        FileOutputStream fos = openFileOutput("info1.txt", MODE_PRIVATE);
                        fos.write("haha".getBytes());
                } catch (Exception e) {
                        e.printStackTrace();
                }
    }
    public void click2(View v){
            try {
                        //MODE_WORLD_READABLE表示任何用户都对其可读，MODE_WORLD_WRITEABLE表示任何用户都对其可写
                        FileOutputStream fos = openFileOutput("info2.txt", MODE_WORLD_READABLE|MODE_WORLD_WRITEABLE);
                        fos.write("hehe,你来读我啊".getBytes());
                } catch (Exception e) {
                        e.printStackTrace();
                }        
    }
}
```
创建新的名为“其他用户”的应用程序，代码如下：
![img](http://bbs.itheima.com/data/attachment/forum/201506/23/114518y6nlnf9rvnfnm4ms.png.thumb.jpg) 

res\layout\activity_main.xml
```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context=".MainActivity" >
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="读取" 
        android:onClick="click"/>
</RelativeLayout>
```
src/cn.itcast.other/MainActivity.java
```java
package cn.itcast.other;
import java.io.BufferedReader;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;
import java.io.InputStreamReader;
import android.app.Activity;
import android.os.Bundle;
import android.view.View;
import android.widget.Toast;
public class MainActivity extends Activity {
        @Override
        protected void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                setContentView(R.layout.activity_main);
        }
        public void click(View v){
                File file = new File("data/data/cn.itcast.permission/files","info2.txt");
                try {
                        FileInputStream fis = new FileInputStream(file);
                        BufferedReader br = new BufferedReader(new InputStreamReader(fis));
                        Toast.makeText(this,br.readLine(),Toast.LENGTH_LONG).show();
                } catch (Exception e) {
                        e.printStackTrace();
                }
        }
}
```
运行结果：

点击按钮一、按钮二，生成文件info1.txt、info2.txt

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/114823xdkidork4emih7ie.png.thumb.jpg) 

可以看到info1.txt为拥有者和同一用户组具备读写权限，info2.txt则为任何用户都具备读写权限。

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/114822s5tkmam65jozmkb8.png.thumb.jpg) 

运行“其他用户”应用程序，点击按钮，可以看到读取info2.txt文件成功。

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/115102wk6qkak5cj7co66x.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/115131u4wz811gf817xcc1.png.thumb.jpg) 

# 9. SharedPreferences
在真实开发中，类似于账号及密码保存功能等持久化保存比较零散、简单数据的情况使用SharedPerferences比较方便，它是以键值对的方式将数据存储进xml文件中。

示例：修改保存用户登录名，密码的案例，用SharedPreferences保存数据的方式实现。
src/cn.itcast.sharedpreference/MainActivity.java
```java
package cn.itcast.sharedpreference;
import android.app.Activity;
import android.content.SharedPreferences;
import android.content.SharedPreferences.Editor;
import android.os.Bundle;
import android.view.View;
import android.widget.CheckBox;
import android.widget.EditText;
import android.widget.Toast;
public class MainActivity extends Activity {
    private EditText et_name;
    private EditText et_pass;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        et_name = (EditText) findViewById(R.id.et_name);
        et_pass = (EditText) findViewById(R.id.et_pass);
        readAccount();
    }
    public void login(View v){
            String name = et_name.getText().toString();
            String pass = et_pass.getText().toString();
            CheckBox cb = (CheckBox) findViewById(R.id.cb);
            if(cb.isChecked()){
                    //SharedPreferences是接口，通过getSharedPreferences方法获取对象，第一个参数为文件名，第二个参数控制访问权限。
                    //在路径为data/data/包名文件夹/shared_prefs目录下保存生成的xml文件。
                    SharedPreferences sp = getSharedPreferences("info", MODE_PRIVATE);
                    //获取编辑器
                    Editor ed = sp.edit();
                    ed.putString("name", name);
                    ed.putString("pass", pass);
                    //提交
                    ed.commit();
            }
            Toast.makeText(this, "登陆成功", Toast.LENGTH_LONG).show();
    }
    public void readAccount(){
            SharedPreferences sp = getSharedPreferences("info", MODE_PRIVATE);
            //第二个参数表示如果没有对应第一个参数的value值返回的默认值，这里设置为空字符串。
            String name = sp.getString("name", "");
            String pass = sp.getString("pass", "");
            et_name.setText(name);
            et_pass.setText(pass);
    }
}
```
运行结果：

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/153821nuwi24sz8sss70ms.png.thumb.jpg) 

可以看到，数据成功保存到xml文件中。

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/153826np1bv41vhaz42exf.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/153827ixiqkvqmyx5n5gig.png.thumb.jpg)

回显成功。

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/153828cn6fx9frgvvls55s.png.thumb.jpg) 

# 10. 生成xml文件
虽然SharedPreferences可以实现保存数据到xml中，但是，对于类似于用户保存短信等Android应用程序来说还是不行的。例如，保存短信功能，需要保存内容、发送时间、接收人号码、短信类型（发送、接收）等等多种类型数据，尤其是数据组织结构比较复杂时，使用SharedPreferences会相当的麻烦。

示例：通过xml方式保存短信。
res\layout\activity_main.xml
```java
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context=".MainActivity" >
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="生成xml文件" 
        android:onClick="click"
        />
</RelativeLayout>
src/cn.itcast.createxml/Sms.java
​```java
package cn.itcast.createxml;
public class Sms {
        private String body;
        private String address;
        private long date;
        private int type;
        public String getBody() {
                return body;
        }
        public void setBody(String body) {
                this.body = body;
        }
        public String getAddress() {
                return address;
        }
        public void setAddress(String address) {
                this.address = address;
        }
        public long getDate() {
                return date;
        }
        public void setDate(long date) {
                this.date = date;
        }
        public int getType() {
                return type;
        }
        public void setType(int type) {
                this.type = type;
        }
        public Sms(String body, String address, long date, int type) {
                super();
                this.body = body;
                this.address = address;
                this.date = date;
                this.type = type;
        }
}
```

src/cn.itcast.createxml/MainActivity.java
```java
package cn.itcast.createxml;
import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.util.ArrayList;
import java.util.List;
import android.app.Activity;
import android.os.Bundle;
import android.view.View;
public class MainActivity extends Activity {
        List<Sms> smsList;
        @Override
        protected void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                setContentView(R.layout.activity_main);
                smsList = new ArrayList<Sms>();
                for(int i = 0; i < 10; i++){
                        Sms sms = new Sms("云鹤" + i + "号","138" + i + i,System.currentTimeMillis(),1);
                        smsList.add(sms);
                }
        }
        public void click(View v){
                StringBuffer sb = new StringBuffer();
                sb.append("<?xml version=\"1.0\" encoding=\"utf-8\"?>");
                sb.append("<smss>");
                for(Sms sms : smsList){
                        sb.append("<sms>");
                        sb.append("<body>");
                        sb.append(sms.getBody());
                        sb.append("</body>");
                        sb.append("<address>");
                        sb.append(sms.getAddress());
                        sb.append("</address>");
                        sb.append("<date>");
                        sb.append(sms.getDate());
                        sb.append("</date>");
                        sb.append("<type>");
                        sb.append(sms.getType());
                        sb.append("</type>");
                        sb.append("</sms>");
                }
                //备份数据的应用一般都存在外部存储空间中。因为，一旦应用程序被删，内部存储空间内的该应用程序就会被清除，而外部存储空间不会被清掉。
                File file = new File("sdcard/sms.xml");
                try {
                        FileOutputStream fos = new FileOutputStream(file);
                        fos.write(sb.toString().getBytes());
                        fos.close();
                } catch (Exception e) {
                        e.printStackTrace();
                }
        }
}
```
运行结果：

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/163110u16kx7fnz26l2mmn.png.thumb.jpg) 

点击“生成xml文件”按钮，可以看到成功生成了sms.xml文件。

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/163319poo07t3fa3v0500f.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/163320rq6qf6re56s5glg9.png.thumb.jpg) 

# 11. xml序列化器

上面拼接字符串的方法过于麻烦，Android中有一个xml序列化器可以帮助我们解决这个问题。
示例：将上面示例中拼接字符串的方式修改为通过xml序列化器生成xml文件。

src/cn.itcast.createxml/MainActivity.java
```java
package cn.itcast.createxml;
import java.io.File;
import java.io.FileOutputStream;
import java.util.ArrayList;
import java.util.List;
import org.xmlpull.v1.XmlSerializer;
import android.app.Activity;
import android.os.Bundle;
import android.util.Xml;
import android.view.View;
public class MainActivity extends Activity {
        List<Sms> smsList;
        @Override
        protected void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                setContentView(R.layout.activity_main);
                smsList = new ArrayList<Sms>();
                for(int i = 0; i < 10; i++){
                        Sms sms = new Sms("云鹤" + i + "号","138" + i + i,System.currentTimeMillis(),1);
                        smsList.add(sms);
                }
        }
        public void click(View v){
                //获取xml序列化器
                XmlSerializer xs = Xml.newSerializer();
                try{
                        //初始化，指定生成的xml文件路径和文件名
                        File file = new File("sdcard/sms2.xml");
                        FileOutputStream fos = new FileOutputStream(file);
                        //编码：xml文件使用什么编码生成
                        xs.setOutput(fos,"utf-8");
                        //开始生成xml文件
                        //生成头结点
                        //两个参数分别用来定义XML文件的头部<?xml version="1.0" encoding="utf-8" standalone="yes"?>中encoding和standalone的值
                        xs.startDocument("utf-8", true);
                        //开始节点，第一个参数为名称空间，用不到，设置为null。
                        xs.startTag(null, "smss");
                        for(Sms sms : smsList){
                                xs.startTag(null, "sms");
                                xs.startTag(null, "body");
                                xs.text(sms.getBody());
                                xs.endTag(null, "body");
                                xs.startTag(null, "address");
                                xs.text(sms.getAddress());
                                xs.endTag(null, "address");
                                xs.startTag(null, "date");
                                xs.text(sms.getDate() + "");
                                xs.endTag(null, "date");
                                xs.startTag(null, "type");
                                xs.text(sms.getType() + "");
                                xs.endTag(null, "type");
                                xs.endTag(null, "sms");
                        }
                        //结束节点
                        xs.endTag(null, "smss");
                        //告知序列化器xml文件生成完毕
                        xs.endDocument();
                }catch(Exception e){
                        e.printStackTrace();
                }
        }
}
```
运行结果：可以看到，生成xml文件成功。

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/175635ksc0hjubcu1dh6bb.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/175637gxu1cllgvd1uuxcl.png.thumb.jpg) 

PS：使用xml序列化器生成xml文件还有一个好处，那就是会自动对特殊字符进行转义。

示例：对上面示例中加上如下代码，其中“<”及“>”都是特殊字符

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/175850skiht4rhl4ikkghp.png.thumb.jpg) 

运行结果：可以看到成功生成xml文件，但是其中的特殊字符已经转义了。

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/180125tr6e88rle2hqx3en.png.thumb.jpg) 

![img](http://bbs.itheima.com/data/attachment/forum/201506/23/180125ojwalaioo0fo9btw.png.thumb.jpg) 

---

# 常见布局
## 相对布局
### RelativeLayout
* 组件默认左对齐、顶部对齐
* 设置组件在指定组件的右边

```xml
android:layout_toRightOf="@id/tv1"
```

* 设置在指定组件的下边
```xml
android:layout_below="@id/tv1"
```
* 设置右对齐父元素
```xml
android:layout_alignParentRight="true"
```
* 设置与指定组件右对齐
```xml
 android:layout_alignRight="@id/tv1"
```

## 线性布局
### LinearLayout
* 指定各个节点的排列方向
```xml
android:orientation="horizontal"
```
* 设置右对齐
```xml
android:layout_gravity="right"
```
* 当竖直布局时，只能左右对齐和水平居中，顶部底部对齐竖直居中无效
* 当水平布局时，只能顶部底部对齐和竖直居中
* 使用match_parent时注意不要把其他组件顶出去
* 线性布局非常重要的一个属性：权重
```xml
android:layout_weight="1"
```
* 权重设置的是按比例分配剩余的空间

## 帧布局
### FrameLayout
* 默认组件都是左对齐和顶部对齐，每个组件相当于一个div
* 可以更改对齐方式
```xml
android:layout_gravity="bottom"
```
* 不能相对于其他组件布局

## 表格布局
### TableLayout
* 每个<TableRow/>节点是一行，它的每个子节点是一列
* 表格布局中的节点可以不设置宽高，因为设置了也无效
  * 根节点<TableLayout/>的子节点宽为匹配父元素，高为包裹内容
  * <TableRow/>节点的子节点宽为包裹内容，高为包裹内容
  * 以上默认属性无法修改

* 根节点中可以设置以下属性，表示让第1列拉伸填满屏幕宽度的剩余空间
```xml
android:stretchColumns="1"
```
## 绝对布局
### AbsoluteLayout
* 直接指定组件的x、y坐标
```xml
android:layout_x="144dp"
android:layout_y="154dp"
```

# logcat
* 日志信息总共分为5个等级
  * verbose 冗余，最低等级
  * debug 调试信息
  * info 正常信息
  * warn 警告
  * error 错误
* 定义过滤器方便查看
* System.out.print输出的日志级别是info，tag是System.out
* Android提供的日志输出api，打印出来的是error级别的信息

```java
Log.v(TAG, "加油吧，童鞋们");
Log.d(TAG, "加油吧，童鞋们");
Log.i(TAG, "加油吧，童鞋们");
Log.w(TAG, "加油吧，童鞋们");
Log.e(TAG, "加油吧，童鞋们");
```

# 文件读写操作
* Ram内存：运行内存，相当于电脑的内存
* Rom内存：内部存储空间，相当于电脑的硬盘
* sd卡：外部存储空间，相当于电脑的移动硬盘
## 在内部存储空间中读写文件

小案例：用户输入账号密码，勾选“记住账号密码”，点击登录按钮，登录的同时持久化保存账号和密码

## 1. 定义布局

## 2. 完成按钮的点击事件
* 弹土司提示用户登录成功

```java
Toast.makeText(this, "登录成功", Toast.LENGTH_SHORT).show();
```

## 3. 拿到用户输入的数据
* 判断用户是否勾选保存账号密码

```java
CheckBox cb = (CheckBox) findViewById(R.id.cb);
if(cb.isChecked()){

}
```

## 4. 开启io流把文件写入内部存储
* 直接开启文件输出流写数据

```java
//持久化保存数据
File file = new File("data/data/com.itheima.rwinrom/info.txt");
FileOutputStream fos = new FileOutputStream(file);
fos.write((name + "##" + pass).getBytes());
fos.close();
```

* 读取数据前先检测文件是否存在

```java
if(file.exists())
```

* 读取保存的数据，也是直接开文件输入流读取

```java
File file = new File("data/data/com.itheima.rwinrom/info.txt");
FileInputStream fis = new FileInputStream(file);
//把字节流转换成字符流
BufferedReader br = new BufferedReader(new InputStreamReader(fis));
String text = br.readLine();
String[] s = text.split("##");
```

* 读取到数据之后，回显至输入框

```java
et_name.setText(s[0]);
et_pass.setText(s[1]);
```

* 应用只能在自己的包名目录下创建文件，不能到别人家去创建

###直接复制项目
* 需要改动的地方：
  * 项目名字
  * 应用包名
  * R文件重新导包

###使用路径api读写文件
* getFilesDir()得到的file对象的路径是data/data/com.itheima.rwinrom2/files
  * 存放在这个路径下的文件，只要你不删，它就一直在
* getCacheDir()得到的file对象的路径是data/data/com.itheima.rwinrom2/cache
  * 存放在这个路径下的文件，当内存不足时，有可能被删除

* 系统管理应用界面的清除缓存，会清除cache文件夹下的东西，清除数据，会清除整个包名目录下的东西
* getCacheDir()：获取缓存文件夹


# 在外部存储读写数据
## sd卡的路径
* sdcard：2.3之前的sd卡路径
* mnt/sdcard：4.3之前的sd卡路径
* storage/sdcard：4.3之后的sd卡路径

* 最简单的打开sd卡的方式

```java
File file = new File("sdcard/info.txt");
```

* 写sd卡需要权限

```xml
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```
* 读sd卡，在4.0之前不需要权限，4.0之后可以设置为需要

```xml
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
```

* 使用api获得sd卡的真实路径，部分手机品牌会更改sd卡的路径

```java
Environment.getExternalStorageDirectory()
```

* 判断sd卡是否准备就绪

```java
if(Environment.getExternalStorageState().equals(Environment.MEDIA_MOUNTED))
//MEDIA_UNKNOWN:不能识别sd卡
//MEDIA_REMOVED:没有sd卡
//MEDIA_UNMOUNTED:sd卡存在但是没有挂载
//MEDIA_CHECKING:sd卡正在准备
//MEDIA_MOUNTED：sd卡已经挂载，可用
```


#查看源代码查找获取sd卡剩余容量的代码
* 导入Settings项目
* 查找“可用空间”得到

```xml
<string name="memory_available" msgid="418542433817289474">"可用空间"</string>
```

* 查找"memory_available"，得到

```xml
<Preference android:key="memory_sd_avail"
    style="?android:attr/preferenceInformationStyle"
    android:title="@string/memory_available"
    android:summary="00"/>
```

* 查找"memory_sd_avail"，得到

```java
//这个字符串就是sd卡剩余容量
formatSize(availableBlocks * blockSize) + readOnly
//这两个参数相乘，得到sd卡以字节为单位的剩余容量
availableBlocks * blockSize
```

* 存储设备会被分为若干个区块，每个区块有固定的大小
* 区块大小 * 区块数量 等于 存储设备的总大小


# Linux文件的访问权限
* 在Android中，每一个应用是一个独立的用户
* drwxrwxrwx
* 第1位：d表示文件夹，-表示文件
* 第2-4位：rwx，表示这个文件的拥有者用户（owner）对该文件的权限
  * r：读
  * w：写
  * x：执行
* 第5-7位：rwx，表示跟文件拥有者用户同组的用户（grouper）对该文件的权限
* 第8-10位：rwx，表示其他用户组的用户（other）对该文件的权限

----
#openFileOutput的四种模式
* MODE_PRIVATE：-rw-rw----
* MODE_APPEND:-rw-rw----
* MODE_WORLD_WRITEABLE:-rw-rw--w-
* MODE_WORLD_READABLE:-rw-rw-r--

```
FileOutputStream fos = openFileOutput(name,mode);
```
----
#SharedPreference
>用SharedPreference存储账号密码

* 往SharedPreference里写数据

```java
//拿到一个SharedPreference对象
SharedPreferences sp = getSharedPreferences("config", MODE_PRIVATE);
//拿到编辑器
Editor ed = sp.edit();
//写数据
ed.putBoolean("name", name);
ed.commit();
```

* 从SharedPreference里取数据

```java
SharedPreferences sp = getSharedPreferences("config", MODE_PRIVATE);
//从SharedPreference里取数据
String name = sp.getBoolean("name", "");
```

---
#生成XML文件备份短信

* 创建几个虚拟的短信对象，存在list中
* 备份数据通常都是备份至sd卡
###使用StringBuffer拼接字符串
* 把整个xml文件所有节点append到sb对象里

```java
sb.append("<?xml version='1.0' encoding='utf-8' standalone='yes' ?>");
//添加smss的开始节点
sb.append("<smss>");
```

* 把sb写到输出流中

```java
fos.write(sb.toString().getBytes());
```

###使用XMl序列化器生成xml文件
* 得到xml序列化器对象

```java
XmlSerializer xs = Xml.newSerializer();
```

* 给序列化器设置输出流

```java
File file = new File(Environment.getExternalStorageDirectory(), "backupsms.xml");
FileOutputStream fos = new FileOutputStream(file);
//给序列化器指定好输出流
xs.setOutput(fos, "utf-8");
```

* 开始生成xml文件

```java
xs.startDocument("utf-8", true);
xs.startTag(null, "smss");
```

```java
package com.itheima.createxml.domain;

public class Message {

	private String body;
	private String date;
	private String address;
	private String type;
	public String getBody() {
		return body;
	}
	public void setBody(String body) {
		this.body = body;
	}
	public String getDate() {
		return date;
	}
	public void setDate(String date) {
		this.date = date;
	}
	public String getAddress() {
		return address;
	}
	public void setAddress(String address) {
		this.address = address;
	}
	public String getType() {
		return type;
	}
	public void setType(String type) {
		this.type = type;
	}
	public Message(String body, String date, String address, String type) {
		super();
		this.body = body;
		this.date = date;
		this.address = address;
		this.type = type;
	}


}

```

```java
package com.itheima.xmlserializer;

import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

import org.xmlpull.v1.XmlSerializer;

import com.itheima.createxml.domain.Message;

import android.os.Bundle;
import android.app.Activity;
import android.util.Xml;
import android.view.Menu;
import android.view.View;

public class MainActivity extends Activity {

	List<Message> smsList;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		// 虚拟10条短信
		smsList = new ArrayList<Message>();
		for (int i = 0; i < 10; i++) {
			Message sms = new Message("小志好棒" + i, System.currentTimeMillis()
					+ "", "138" + i + i, "1");
			smsList.add(sms);
		}
	}

	public void click(View v){
		//使用xml序列化器生成xml文件
		//1.拿到序列化器对象
		XmlSerializer xs = Xml.newSerializer();
		//2.初始化
		File file = new File("sdcard/sms2.xml");
		try {
			FileOutputStream fos = new FileOutputStream(file);
			//enconding:指定用什么编码生成xml文件
			xs.setOutput(fos, "utf-8");

			//3.开始生成xml文件
			//enconding:指定头结点中的enconding属性的值
			xs.startDocument("utf-8", true);

			xs.startTag(null, "message");

			for (Message sms : smsList) {
				xs.startTag(null, "sms");

				xs.startTag(null, "body");
				xs.text(sms.getBody() + "<body>");
				xs.endTag(null, "body");

				xs.startTag(null, "date");
				xs.text(sms.getDate());
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

			//告诉序列化器，文件生成完毕
			xs.endDocument();
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}

}

```
---
#pull解析xml文件
* 先自己写一个xml文件，存一些天气信息
## 拿到xml文件

```java
InputStream is = getClassLoader().getResourceAsStream("weather.xml");
```
## 拿到pull解析器

```java
XmlPullParser xp = Xml.newPullParser();
```

## 开始解析
* 拿到指针所在当前节点的事件类型

int type = xp.getEventType();

* 事件类型主要有五种
  * START_DOCUMENT：xml头的事件类型
  * END_DOCUMENT：xml尾的事件类型
  * START_TAG：开始节点的事件类型
  * END_TAG：结束节点的事件类型
  * TEXT：文本节点的事件类型

* 如果获取到的事件类型不是END_DOCUMENT，就说明解析还没有完成，如果是，解析完成，while循环结束

  	while(type != XmlPullParser.END_DOCUMENT)
* 当我们解析到不同节点时，需要进行不同的操作，所以判断一下当前节点的name
  * 当解析到weather的开始节点时，new出list
  * 当解析到city的开始节点时，创建city对象，创建对象是为了更方便的保存即将解析到的文本
  * 当解析到name开始节点时，获取下一个节点的文本内容，temp、pm也是一样

```java
case XmlPullParser.START_TAG:
//获取当前节点的名字
	if("weather".equals(xp.getName())){
		citys = new ArrayList<City>();
	}
	else if("city".equals(xp.getName())){
		city = new City();
	}
	else if("name".equals(xp.getName())){
		//获取当前节点的下一个节点的文本
		String name = xp.nextText();
		city.setName(name);
	}
	else if("temp".equals(xp.getName())){
		String temp = xp.nextText();
		city.setTemp(temp);
	}
	else if("pm".equals(xp.getName())){
		String pm = xp.nextText();
		city.setPm(pm);
	}
	break;
```

* 当解析到city的结束节点时，说明city的三个子节点已经全部解析完了，把city对象添加至list

```java
case XmlPullParser.END_TAG:
	if("city".equals(xp.getName())){
			citys.add(city);
	}
```

```java
package com.itheima.pullparser;

import java.io.InputStream;
import java.util.ArrayList;
import java.util.List;

import org.xmlpull.v1.XmlPullParser;
import org.xmlpull.v1.XmlPullParserException;

import com.itheima.pullparser.domain.City;

import android.os.Bundle;
import android.app.Activity;
import android.util.Xml;
import android.view.Menu;
import android.view.View;

public class MainActivity extends Activity {

	List<City> cityList;
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
	}

	public void click(View v){
		//获取到src文件夹下的资源文件
		InputStream is = getClassLoader().getResourceAsStream("weather.xml");

		//拿到pull解析器对象
		XmlPullParser xp = Xml.newPullParser();
		//初始化
		try {
			xp.setInput(is, "gbk");

			//获取当前节点的事件类型，通过事件类型的判断，我们可以知道当前节点是什么节点，从而确定我们应该做什么操作
			int type = xp.getEventType();
			City city = null;
			while(type != XmlPullParser.END_DOCUMENT){
				//根据节点的类型，要做不同的操作
				switch (type) {
				case XmlPullParser.START_TAG:
					//					获取当前节点的名字
					if("weather".equals(xp.getName())){
						//创建city集合对象，用于存放city的javabean
						cityList = new ArrayList<City>();
					}
					else if("city".equals(xp.getName())){
						//创建city的javabean对象
						city = new City();
					}
					else if("name".equals(xp.getName())){
						//				获取当前节点的下一个节点的文本
						String name = xp.nextText();
						city.setName(name);
					}
					else if("temp".equals(xp.getName())){
						//				获取当前节点的下一个节点的文本
						String temp = xp.nextText();
						city.setTemp(temp);
					}
					else if("pm".equals(xp.getName())){
						//				获取当前节点的下一个节点的文本
						String pm = xp.nextText();
						city.setPm(pm);
					}
					break;
				case XmlPullParser.END_TAG:
					if("city".equals(xp.getName())){
						//把city的javabean放入集合中
						cityList.add(city);
					}
					break;

				}

				//把指针移动到下一个节点，并返回该节点的事件类型
				type = xp.next();
			}

			for (City c : cityList) {
				System.out.println(c.toString());
			}
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}

}

```

#常见布局
###线性布局
* 有一个布局方向，水平或者竖直
* 在竖直布局下，左对齐、右对齐，水平居中生效
* 在水平布局下，顶部对齐、底部对齐、竖直居中生效
* 权重：按比例分配屏幕的剩余宽度或者高度

###相对布局
* 组件默认位置都是左上角，组件之间可以重叠
* 可以相对于父元素上下左右对齐，相对于父元素，水平居中、竖直居中、水平竖直同时居中
* 可以相对于其他组件上下左右对齐
* 可以布局于其他组件的上方、下方、左边、右边

###帧布局
* 组件默认位置都是左上角，组件之间可以重叠
* 可以设置上下左右对齐，水平竖直居中，设置方式与线性布局一样

###表格布局
* 每有一个TableRow子节点表示一行，该子节点的每一个子节点都表示一列
* TableLayout的一级子节点默认宽都是匹配父元素
* TableRow的子节点默认宽高都是包裹内容

---
#Logcat
###等级
* verbose：冗余，最低等级
* debug：调试
* info：正常等级的信息
* warn：警告
* error：错误

---
#Android的存储
###内部存储空间
* RAM内存：运行内存，相当于电脑的内存
* ROM内存：存储内存，相当于电脑的硬盘
###外部存储空间
* SD卡：相当于电脑的移动硬盘
  * 2.2之前，sd卡路径：sdcard
  * 4.3之前，sd卡路径：mnt/sdcard
  * 4.3开始，sd卡路径：storage/sdcard

* 所有存储设备，都会被划分成若干个区块，每个区块有固定的大小
* 存储设备的总大小 = 区块大小 * 区块数量

---
#文件访问权限
* 指的是谁能访问这个文件
* 在Android中，每一个应用，都是一个独立的用户
* 使用10个字母表示
* drwxrwxrwx
* 第一个字母：
  * d：表示文件夹
  * -：表示文件
* 第一组rwx：表示的是文件拥有者（owner）对文件的权限
  * r：read，读
  * w：write
  * x：execute
* 第二组rwx：表示的是跟文件拥有者属于同一用户组的用户（grouper）对文件的权限
* 第三组rwx：表示的其他用户（other）对文件的权限

---
#SharedPreference
* 非常适合用来保存零散的简单的数据
#测试
* 黑盒测试
  * 测试逻辑业务
* 白盒测试
  * 测试逻辑方法

* 根据测试粒度
  * 方法测试：function test
  * 单元测试：unit test
  * 集成测试：integration test
  * 系统测试：system test

* 根据测试暴力程度
  * 冒烟测试：smoke test
  * 压力测试：pressure test

------
#单元测试junit
* 定义一个类继承AndroidTestCase，在类中定义方法，即可测试该方法
* 在指定指令集时，targetPackage指定你要测试的应用的包名

```xml
<instrumentation
	  android:name="android.test.InstrumentationTestRunner"
	    //指定测试框架要测试哪一个项目
	    android:targetPackage="com.itheima.junit"
	    ></instrumentation>
```
* 定义使用的类库

```xml
<uses-library android:name="android.test.runner"></uses-library>
```

* 断言的作用，assertEquals(期待值，实际值)，检测运行结果和预期是否一致
* 如果应用出现异常，会抛给测试框架

-----
#SQLite数据库
* 轻量级关系型数据库
* 创建数据库需要使用的api：SQLiteOpenHelper
  * 必须定义一个构造方法：

```java
//name：数据库文件的名字
//factory：游标工厂
//version：数据库版本
public MyOpenHelper(Context context, String name, CursorFactory factory, int version){}
```

	* 数据库被创建时会调用：onCreate方法
	* 数据库升级时会调用：onUpgrade方法

## 创建数据库

```java
//创建OpenHelper对象
MyOpenHelper oh = new MyOpenHelper(getContext(), "person.db", null, 1);
//获得数据库对象,如果数据库不存在，先创建数据库，后获得，如果存在，则直接获得
SQLiteDatabase db = oh.getWritableDatabase();
```


* getWritableDatabase()：打开可读写的数据库
* getReadableDatabase()：在磁盘空间不足时打开只读数据库，否则打开可读写数据库
* 在创建数据库时创建表
```java
public void onCreate(SQLiteDatabase db) {
		db.execSQL("create table person (_id integer primary key autoincrement, name char(10), phone char(20), money integer(20))");
	}
```
---
# 数据库的增删改查
## SQL语句
* insert into person (name, phone, money) values ('张三', '159874611', 2000);
* delete from person where name = '李四' and _id = 4;
* update person set money = 6000 where name = '李四';
* select name, phone from person where name = '张三';

###执行SQL语句实现增删改查

```java
//插入
db.execSQL("insert into person (name, phone, money) values (?, ?, ?);", new Object[]{"张三", 15987461, 75000});
//查找
Cursor cs = db.rawQuery("select _id, name, money from person where name = ?;", new String[]{"张三"});
* 测试方法执行前会调用此方法
```
```java
protected void setUp() throws Exception {
	super.setUp();
	//					获取虚拟上下文对象
	oh = new MyOpenHelper(getContext(), "people.db", null, 1);
}
```
```java
//测试方法执行完毕之后，此方法调用
@Override
protected void tearDown() throws Exception {
	// TODO Auto-generated method stub
	super.tearDown();
	db.close();
}
```
查询
```java
public void select(){
	Cursor cursor = db.rawQuery("select name, salary from person", null);
	while(cursor.moveToNext()){
	//通过列索引获取列的值
	String name = cursor.getString(cursor.getColumnIndex("name"));
			String salary = cursor.getString(1);
			System.out.println(name + ";" + salary);
	}
}
```
###使用api实现增删改查
* 插入

```java
//以键值对的形式保存要存入数据库的数据
ContentValues cv = new ContentValues();
cv.put("name", "刘能");
cv.put("phone", 1651646);
cv.put("money", 3500);
//返回值是改行的主键，如果出错返回-1
long i = db.insert("person", null, cv);
```

* 删除

```java
//返回值是删除的行数
int i = db.delete("person", "_id = ? and name = ?", new String[]{"1", "张三"});
```

* 修改

```java
ContentValues cv = new ContentValues();
cv.put("money", 25000);
int i = db.update("person", cv, "name = ?", new String[]{"赵四"});
```

* 查询

```java
//arg1:要查询的字段
//arg2：查询条件
//arg3:填充查询条件的占位符
Cursor cs = db.query("person", new String[]{"name", "money"}, "name = ?", new String[]{"张三"}, null, null, null);
while(cs.moveToNext()){
	//							获取指定列的索引值
	String name = cs.getString(cs.getColumnIndex("name"));
	String money = cs.getString(cs.getColumnIndex("money"));
	System.out.println(name + ";" + money);
}
```

###事务
* 保证多条SQL语句要么同时成功，要么同时失败
* 最常见案例：银行转账
* 事务api

```java
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
```
---
#把数据库的数据显示至屏幕
1. 任意插入一些数据
*  定义业务bean：Person.java
*  读取数据库的所有数据

```java
Cursor cs = db.query("person", null, null, null, null, null, null);
while(cs.moveToNext()){
	String name = cs.getString(cs.getColumnIndex("name"));
	String phone = cs.getString(cs.getColumnIndex("phone"));
	String money = cs.getString(cs.getColumnIndex("money"));
	//把读到的数据封装至Person对象
	Person p = new Person(name, phone, money);
	//把person对象保存至集合中
	people.add(p);
}
```

* 把集合中的数据显示至屏幕

```java
LinearLayout ll = (LinearLayout) findViewById(R.id.ll);
for(Person p : people){
	 //创建TextView，每条数据用一个文本框显示
	 TextView tv = new TextView(this);
	 tv.setText(p.toString());
	 //把文本框设置为ll的子节点
	 ll.addView(tv);
}
```

* 分页查询

Cursor cs = db.query("person", null, null, null, null, null, null, "0, 10");

---
#ListView
* 就是用来显示一行一行的条目的
* MVC结构
  * M：model模型层，要显示的数据           ————people集合
  * V：view视图层，用户看到的界面          ————ListView
  * c：control控制层，操作数据如何显示     ————adapter对象
* 每一个条目都是一个View对象
* 点击监听：lv.setOnItemClickListener();
###BaseAdapter
* 必须实现的两个方法

  * 第一个

    	//系统调用此方法，用来获知模型层有多少条数据
    	@Override
    	public int getCount() {
    		return people.size();
    	}

  * 第二个

```java
//系统调用此方法，获取要显示至ListView的View对象
//position:是return的View对象所对应的数据在集合中的位置
@Override
public View getView(int position, View convertView, ViewGroup parent) {
	System.out.println("getView方法调用" + position);
	TextView tv = new TextView(MainActivity.this);
	//拿到集合中的元素
	Person p = people.get(position);
	tv.setText(p.toString());

	//把TextView的对象返回出去，它会变成ListView的条目
	return tv;
}
```

* 屏幕上能显示多少个条目，getView方法就会被调用多少次，屏幕向下滑动时，getView会继续被调用，创建更多的View对象显示至屏幕
###条目的缓存
* 当条目划出屏幕时，系统会把该条目缓存至内存，当该条目再次进入屏幕，系统在重新调用getView时会把缓存的条目作为convertView参数传入，但是传入的条目不一定是之前被缓存的该条目，即系统有可能在调用getView方法获取第一个条目时，传入任意一个条目的缓存
###布局填充
* View.inflate(context , resourceid , viewgroup)
* LayoutInflater.from()布局填充器
* LayoutInflater inflater2 = (LayoutInflater) getSystemService(LAYOUT_INFLATER_SERVICE);
* ViewGroup
###ListView的优化
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
			//判断条目是否有缓存，ListView优化
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
## ArrayAdapter、SimpleAdapter

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
* 图片：R.drawable.name
* 控件：R.id.name
* 布局：R.layout.name

---
#对话框
![](http://img.blog.csdn.net/20151021212943123)
###确定取消对话框
* 创建对话框构建器对象，类似工厂模式

AlertDialog.Builder builder = new Builder(this);

* 设置标题和正文

```java
builder.setTitle("警告");
builder.setMessage("若练此功，必先自宫");
```

* 设置确定和取消按钮

```java
builder.setPositiveButton("现在自宫", new OnClickListener() {

	@Override
	public void onClick(DialogInterface dialog, int which) {
		// TODO Auto-generated method stub
		Toast.makeText(MainActivity.this, "恭喜你自宫成功，现在程序退出", 0).show();
	}
});

builder.setNegativeButton("下次再说", new OnClickListener() {

	@Override
	public void onClick(DialogInterface dialog, int which) {
		// TODO Auto-generated method stub
		Toast.makeText(MainActivity.this, "若不自宫，一定不成功", 0).show();
	}
});
```

* 使用构建器创建出对话框对象

```java
AlertDialog ad = builder.create();
ad.show();
```

## 单选对话框
![](http://img.blog.csdn.net/20151021214655698)

```java
AlertDialog.Builder builder = new Builder(this);
builder.setTitle("选择你的性别");
// 定义单选选项
final String[] items = new String[]{
		"男", "女", "其他"
};
//-1表示没有默认选择
//点击侦听的导包要注意别导错
builder.setSingleChoiceItems(items, -1, new OnClickListener() {

	//which表示点击的是哪一个选项
	@Override
	public void onClick(DialogInterface dialog, int which) {
		Toast.makeText(MainActivity.this, "您选择了" + items[which], 0).show();
		//对话框消失
		dialog.dismiss();
	}
});

builder.show();
```

###多选对话框
![这里写图片描述](http://img.blog.csdn.net/20151022074118392)

```java
AlertDialog.Builder builder = new Builder(this);
    	builder.setTitle("请选择你认为最帅的人");
// 定义多选的选项，因为可以多选，所以需要一个boolean数组来记录哪些选项被选了

final String[] items = new String[]{
		"赵帅哥",
		"赵师哥",
		"赵老师",
		"侃哥"
};
//true表示对应位置的选项被选了
final boolean[] checkedItems = new boolean[]{
		true,
		false,
		false,
		false,
};
builder.setMultiChoiceItems(items, checkedItems, new OnMultiChoiceClickListener() {

	//点击某个选项，如果该选项之前没被选择，那么此时isChecked的值为true
	@Override
	public void onClick(DialogInterface dialog, int which, boolean isChecked) {
		checkedItems[which] = isChecked;
	}
});

builder.setPositiveButton("确定", new OnClickListener() {

	@Override
	public void onClick(DialogInterface dialog, int which) {
		StringBuffer sb = new StringBuffer();
		for(int i = 0;i < items.length; i++){
			sb.append(checkedItems[i] ? items[i] + " " : "");
		}
		Toast.makeText(MainActivity.this, sb.toString(), 0).show();
	}
});
builder.show();
```

#常见布局

## 线性布局
* 有一个布局方向，水平或者竖直
* 在竖直布局下，左对齐、右对齐，水平居中生效
* 在水平布局下，顶部对齐、底部对齐、竖直居中生效
* 权重：按比例分配屏幕的剩余宽度或者高度

## 相对布局
* 组件默认位置都是左上角，组件之间可以重叠
* 可以相对于父元素上下左右对齐，相对于父元素，水平居中、竖直居中、水平竖直同时居中
* 可以相对于其他组件上下左右对齐
* 可以布局于其他组件的上方、下方、左边、右边

## 帧布局
* 组件默认位置都是左上角，组件之间可以重叠
* 可以设置上下左右对齐，水平竖直居中，设置方式与线性布局一样

## 表格布局
* 每有一个TableRow子节点表示一行，该子节点的每一个子节点都表示一列
* TableLayout的一级子节点默认宽都是匹配父元素
* TableRow的子节点默认宽高都是包裹内容

---
#Logcat
###等级
* verbose：冗余，最低等级
* debug：调试
* info：正常等级的信息
* warn：警告
* error：错误

---
#Android的存储
###内部存储空间
* RAM内存：运行内存，相当于电脑的内存
* ROM内存：存储内存，相当于电脑的硬盘
###外部存储空间
* SD卡：相当于电脑的移动硬盘
  * 2.2之前，sd卡路径：sdcard
  * 4.3之前，sd卡路径：mnt/sdcard
  * 4.3开始，sd卡路径：storage/sdcard

* 所有存储设备，都会被划分成若干个区块，每个区块有固定的大小
* 存储设备的总大小 = 区块大小 * 区块数量

---
#文件访问权限
* 指的是谁能访问这个文件
* 在Android中，每一个应用，都是一个独立的用户
* 使用10个字母表示
* drwxrwxrwx
* 第一个字母：
  * d：表示文件夹
  * -：表示文件
* 第一组rwx：表示的是文件拥有者（owner）对文件的权限
  * r：read，读
  * w：write
  * x：execute
* 第二组rwx：表示的是跟文件拥有者属于同一用户组的用户（grouper）对文件的权限
* 第三组rwx：表示的其他用户（other）对文件的权限

---
# SharedPreference
* 非常适合用来保存零散的简单的数据

# 案例1：SharedPreference

```java
public class MainActivity extends Activity {

    private EditText et_name;
	private EditText et_pass;

	@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        et_name = (EditText) findViewById(R.id.et_name);
    	et_pass = (EditText) findViewById(R.id.et_pass);

        readAccount();

    }

    public void readAccount(){
	    SharedPreferences sp = getSharedPreferences("info", MODE_PRIVATE);
	    String name = sp.getString("name", "");
	    String pass = sp.getString("pass", "");

		et_name.setText(name);
		et_pass.setText(pass);
    }

    public void login(View v){

    	String name = et_name.getText().toString();
    	String pass = et_pass.getText().toString();

    	CheckBox cb = (CheckBox) findViewById(R.id.cb);
    	//判断选框是否被勾选
    	if(cb.isChecked()){
    		//使用sharedPreference来保存用户名和密码
    		//路径在data/data/com.itheima.sharedpreference/share_
    		SharedPreferences sp = getSharedPreferences("info", MODE_PRIVATE);
    		//拿到sp的编辑器
    		Editor ed = sp.edit();
    		ed.putString("name", name);
    		ed.putString("pass", pass);
    		//提交
    		ed.commit();
    	}

    	//创建并显示吐司对话框
    	Toast.makeText(this, "登录成功", 0).show();
    }

}

```
# 案例2：生成XML文件

```java
public class MainActivity extends Activity {

	List<Message> smsList;
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);

		//虚拟10条短信
		smsList = new ArrayList<Message>();
		for(int i = 0; i < 10; i++){
			Message sms = new Message("小志好棒" + i, System.currentTimeMillis() + "", "138"+i+i, "1");
			smsList.add(sms);
		}
	}

	public void click(View v){
		//在内存中把xml备份短信的格式拼接出来
		StringBuffer sb = new StringBuffer();
		sb.append("<?xml version='1.0' encoding='utf-8' standalone='yes' ?>");
		sb.append("<messages>");
		for (Message sms : smsList) {
			sb.append("<sms>");

			sb.append("<body>");
			sb.append(sms.getBody());
			sb.append("</body>");

			sb.append("<date>");
			sb.append(sms.getDate());
			sb.append("</date>");

			sb.append("<type>");
			sb.append(sms.getType());
			sb.append("</type>");

			sb.append("<address>");
			sb.append(sms.getAddress());
			sb.append("</address>");

			sb.append("</sms>");
		}
		sb.append("</messages>");

		File file = new File("sdcard/sms.xml");
		try {
			FileOutputStream fos = new FileOutputStream(file);
			fos.write(sb.toString().getBytes());
			fos.close();
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}

}

```
# 案例3：XML序列化器

```java
public class MainActivity extends Activity {

	List<Message> smsList;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		// 虚拟10条短信
		smsList = new ArrayList<Message>();
		for (int i = 0; i < 10; i++) {
			Message sms = new Message("小志好棒" + i, System.currentTimeMillis()
					+ "", "138" + i + i, "1");
			smsList.add(sms);
		}
	}

	public void click(View v){
		//使用xml序列化器生成xml文件
		//1.拿到序列化器对象
		XmlSerializer xs = Xml.newSerializer();
		//2.初始化
		File file = new File("sdcard/sms2.xml");
		try {
			FileOutputStream fos = new FileOutputStream(file);
			//enconding:指定用什么编码生成xml文件
			xs.setOutput(fos, "utf-8");

			//3.开始生成xml文件
			//enconding:指定头结点中的enconding属性的值
			xs.startDocument("utf-8", true);

			xs.startTag(null, "message");

			for (Message sms : smsList) {
				xs.startTag(null, "sms");

				xs.startTag(null, "body");
				xs.text(sms.getBody() + "<body>");
				xs.endTag(null, "body");

				xs.startTag(null, "date");
				xs.text(sms.getDate());
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

			//告诉序列化器，文件生成完毕
			xs.endDocument();
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}

}
```
# 案例4：PULL解析

```java
public class MainActivity extends Activity {

	List<Message> smsList;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		// 虚拟10条短信
		smsList = new ArrayList<Message>();
		for (int i = 0; i < 10; i++) {
			Message sms = new Message("小志好棒" + i, System.currentTimeMillis()
					+ "", "138" + i + i, "1");
			smsList.add(sms);
		}
	}

	public void click(View v){
		//使用xml序列化器生成xml文件
		//1.拿到序列化器对象
		XmlSerializer xs = Xml.newSerializer();
		//2.初始化
		File file = new File("sdcard/sms2.xml");
		try {
			FileOutputStream fos = new FileOutputStream(file);
			//enconding:指定用什么编码生成xml文件
			xs.setOutput(fos, "utf-8");

			//3.开始生成xml文件
			//enconding:指定头结点中的enconding属性的值
			xs.startDocument("utf-8", true);

			xs.startTag(null, "message");

			for (Message sms : smsList) {
				xs.startTag(null, "sms");

				xs.startTag(null, "body");
				xs.text(sms.getBody() + "<body>");
				xs.endTag(null, "body");

				xs.startTag(null, "date");
				xs.text(sms.getDate());
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

			//告诉序列化器，文件生成完毕
			xs.endDocument();
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}

}
```

