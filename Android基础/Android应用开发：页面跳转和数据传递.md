# 1. Activity的跳转

## **1.1 创建第二个Activity**

* 需要在清单文件中为其配置一个activity标签
* 标签中如果带有这个子节点，则会在系统中多创建一个快捷图标

```xml
<intent-filter>
      <action android:name="android.intent.action.MAIN" />
      <category android:name="android.intent.category.LAUNCHER" />
</intent-filter>
```

* 一个应用程序可以在桌面创建多个快捷图标。
* activity的名称、图标可以和应用程序的名称、图标不相同

```xml
android:icon="@drawable/ic_launcher"
android:label="@string/app_name"
```

Activity的跳转需要创建Intent对象，通过设置intent对象的参数指定要跳转的Activity；通过设置Activity的包名和类名实现跳转，称为显式意图；通过指定动作实现跳转，称为隐式意图

## **1.2 显式意图**
* 跳转至同一项目下的另一个Activity，直接指定该Activity的字节码即可

```java
Intent intent = new Intent();
intent.setClass(this, SecondActivity.class);
startActivity(intent);
```

* 跳转至其他应用中的Activity，需要指定该应用的包名和该Activity的类名

```java
Intent intent = new Intent();
//启动系统自带的拨号器应用
intent.setClassName("com.android.dialer", "com.android.dialer.DialtactsActivity");
startActivity(intent);
```

## 1.3 隐式意图
* 隐式意图跳转至指定Activity

```java
Intent intent = new Intent();
//启动系统自带的拨号器应用
intent.setAction(Intent.ACTION_DIAL);
startActivity(intent);
```

* 要让一个Activity可以被隐式启动，需要在清单文件的activity节点中设置intent-filter子节点

```xml
<intent-filter >
    <action android:name="com.itheima.second"/>
    <data android:scheme="asd" android:mimeType="aa/bb"/>
    <category android:name="android.intent.category.DEFAULT"/>
</intent-filter>
```
* action 指定动作（可以自定义，可以使用系统自带的）
* data   指定数据（操作什么内容）
* category 类别 （默认类别，机顶盒，车载电脑）
* 隐式意图启动Activity，需要为intent设置以上三个属性，且值必须与该Activity在清单文件中对三个属性的定义匹配
* intent-filter节点及其子节点都可以同时定义多个，隐式启动时只需与任意一个匹配即可

## 1.4 获取通过setData传递的数据

```java
//获取启动此Activity的intent对象
Intent intent = getIntent();
Uri uri = intent.getData();
```

##1.5 显式意图和隐式意图的应用场景
* 显式意图用于启动同一应用中的Activity
* 隐式意图用于启动不同应用中的Activity
  * 如果系统中存在多个Activity的intent-filter同时与你的intent匹配，那么系统会显示一个对话框，列出所有匹配的Activity，由用户选择启动哪一个


# 2. Activity跳转时的数据传递
* Activity通过Intent启动时，可以通过Intent对象携带数据到目标Activity

```java
Intent intent = new Intent(this, SecondActivity.class);
intent.putExtra("maleName", maleName);
intent.putExtra("femaleName", femaleName);
startActivity(intent);
```

* 在目标Activity中取出数据

```java
Intent intent = getIntent();
String maleName = intent.getStringExtra("maleName");
String femaleName = intent.getStringExtra("femaleName");
```

# 3. Activity生命周期

- void onCreate()：Activity已经被创建完毕
- void onStart()：Activity已经显示在屏幕，但没有得到焦点
- void onResume()：Activity得到焦点，可以与用户交互
- void onPause()：Activity失去焦点，无法再与用户交互，但依然可见
- void onStop()：Activity不可见，进入后台
- void onDestroy()：Activity被销毁
- void onRestart()：Activity从不可见变成可见时会执行此方法

## 3.1 使用场景
* Activity创建时需要初始化资源，销毁时需要释放资源；或者播放器应用，在界面进入后台时需要自动暂停

## 3.2 完整生命周期（entire lifetime）
onCreate-->onStart-->onResume-->onPause-->onStop-->onDestory

## 3.3 可视生命周期（visible lifetime）
onStart-->onResume-->onPause-->onStop

## 3.4 前台生命周期（foreground lifetime）
onResume-->onPause

## 3.5 横竖屏切换的生命周期

默认情况下 ，横竖屏切换， 销毁当前的activity，重新创建一个新的activity。在一些特殊的应用程序常见下，比如游戏，不希望横竖屏切换activity被销毁重新创建。模拟器切换横竖屏快捷键ctrl+F11

需求：禁用掉横竖屏切换的生命周期

1、横竖屏写死

```xml
android:screenOrientation="landscape"
android:screenOrientation="portrait"
```
2、让系统的环境 不再去敏感横竖屏的切换。
```xml
android:configChanges="orientation|screenSize|keyboardHidden"
```
# 5. Activity的四种启动模式

每个应用会有一个Activity任务栈，存放已启动的Activity。Activity的启动模式，修改任务栈的排列情况

* standard 标准启动模式
* singleTop 单一顶部模式
  * 如果任务栈的栈顶存在这个要开启的activity，不会重新的创建activity，而是复用已经存在的activity。保证栈顶如果存在，不会重复创建。
  * 应用场景：浏览器的书签
* singeTask 单一任务栈，在当前任务栈里面只能有一个实例存在
  * 当开启activity的时候，就去检查在任务栈里面是否有实例已经存在，如果有实例存在就复用这个已经存在的activity，并且把这个activity上面的所有的别的activity都清空，复用这个已经存在的activity。保证整个任务栈里面只有一个实例存在
  * 应用场景：浏览器的activity
  * 如果一个activity的创建需要占用大量的系统资源（cpu，内存）一般配置这个activity为singletask的启动模式。webkit内核 c代码
* singleInstance启动模式非常特殊， activity会运行在自己的任务栈里面，并且这个任务栈里面只有一个实例存在
  * 如果你要保证一个activity在整个手机操作系统里面只有一个实例存在，使用singleInstance
  * 应用场景： 电话拨打界面

# 6. 掌握开启activity获取返回值

## 6.1 从A界面打开B界面， B界面关闭的时候，返回一个数据给A界面

步骤：

1、开启activity并且获取返回值

```java
startActivityForResult(intent, 0);
```

2、在新开启的界面里面实现设置数据的逻辑

```java
Intent data = new Intent();
data.putExtra("phone", phone);
//设置一个结果数据，数据会返回给调用者
setResult(0, data);
finish();//关闭掉当前的activity，才会返回数据
```

3、在开启者activity里面实现方法

```java
onActivityResult(int requestCode, int resultCode, Intent data) //通过data获取返回的数据
```

4、根据请求码和结果码确定业务逻辑

# 7. Android四大组件
* Activity
* BroadCastReceiver
* Service
* ContentProvider

# 8. 创建第二个activity
* 新创建的activity，必须在清单文件中做配置，否则系统找不到，在显示时会直接报错

```xml
<activity android:name="com.itheima.createactivity.SecondActivity"></activity>
```

* 只要有以下代码，那么就是入口activity，就会生成快捷图标

```xml
<intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
</intent-filter>
```

* 如果Activity所在的包跟应用包名同名，那么可以省略不写

1、 创建class类继承Activity
2、 创建布局文件，作为Activity的显示内容
3、 在清单文件中注册Activity

# 9. Activity的跳转

## 隐式跳转
* 一个Activity如果需要隐式跳转，那么在清单文件中必须添加以下子节点

```xml
<intent-filter >
    <action android:name="com.itheima.sa"/>
    <category android:name="android.intent.category.DEFAULT"/>
</intent-filter>
```

* action节点的name是自己定义的，定义好之后，这个name的值就会成为这个activity动作，在隐式启动Activity时，意图中设置的action必须跟"com.itheima.sa"是完全匹配的

## 应用场景
* 显示意图：启动同一个应用中的Activity
* 隐式意图：启动不同应用中的Activity
* 再启动效率上，隐式远远低于显式
* 如果系统中有多个Activity与意图设置的Action匹配，那么在启动Activity时，会弹出一个对话框，里面包含所有匹配的Activity


# 10. Activity生命周期
* oncreate:Activity对象创建完毕，但此时不可见
* onstart:Activity在屏幕可见，但是此时没有焦点
* onResume：Activity在屏幕可见，并且获得焦点
* onPause：Activity此时在屏幕依然可见，但是已经没有焦点
* onStop：Activity已经不可见了，但此时Activity的对象还在内存中
* onDestroy：Activity对象被销毁

## 内存不足
* 内存不足时，系统会优先杀死后台Activity所在的进程，都杀光了，如果内存还是不足，那么就会杀死暂停状态的Activity所在的进程，如果还是不够，有可能杀死前台进程
* 如果有多个后台进程，在选择杀死的目标时，采用最近最少使用算法（LRU）

## Activity任务栈
* 应用运行过程中，内存中可能会打开多个Activity，那么所有打开的Activity都会被保存在Activity任务栈
* 栈：后进先出，最先进栈，就会最后出栈

## Activity的启动模式
* 标准模式：默认就是这个模式
* singleTop：如果目标Activity不在栈顶，那么就会启动一个新的Activity，如果已经在栈顶了，那么就不会再启动了
* singleTask：如果栈中没有该Activity，那么启动时就会创建一个该Activity，如果栈中已经有该Activity的实例存在了，那么在启动时，就会杀死在栈中处于该Activity上方的所有Activity全部杀死，那么此时该Activity就成为了栈顶Activity。
  * singleTask的作用：保证整个栈中只有一个该Activity的实例
* singleInstance：设为此模式的Activity会有一个自己独立的任务栈，该Activity的实例只会创建一个，保存在独立的任务栈中
  * singleInstance的作用：保证整个系统的内存中只有一个该Activity的实例

# 11. 横竖屏的切换
* Activity在横竖屏切换时会销毁重建，目的就是为了读取新的布局文件
* 写死方向，不允许切换

```xml
android:screenOrientation="portrait"
android:screenOrientation="landscape"
```

* 配置Activity时添加以下属性，横竖屏切换时就不会销毁重建

```xml
android:configChanges="orientation|keyboardHidden|screenSize"
```

# 12. Activity返回时传递数据
* 请求码：用来区分数据来自于哪一个Activity
* 结果码：用来区分，返回的数据时属于什么类型

# 案例1：Activity的跳转

```java

public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }


    /**
     * 跳转至打电话activity
     * 跳转至其他应用的activity
     * 隐式跳转：通过指定action和data
     * @param v
     */
    public void click1(View v){
    	Intent intent = new Intent();
    	//隐式意图
    	intent.setAction(Intent.ACTION_CALL);
    	intent.setData(Uri.parse("tel:110"));
    	//跳转
    	startActivity(intent);
    }

    /**
     * 跳转至secondActivity
     * 在本应用中跳转
     * 显示跳转：直接指定目标Activity的包名和类名
     * @param v
     */
    public void click2(View v){
    	Intent intent = new Intent();
    	//cls：直接指定目标Activity的类名
    	//显示意图
    	intent.setClass(this, SecondActivity.class);
    	startActivity(intent);
    }

    /**
     * 显示跳转至拨号器
     */
    public void click3(View v){
    	Intent intent = new Intent();
    	//指定目标Activity的包名和类名
    	intent.setClassName("com.android.dialer", "com.android.dialer.DialtactsActivity");
    	startActivity(intent);
    }
    /**
     * 隐式跳转至拨号器
     */
    public void click4(View v){
    	Intent intent = new Intent();
    	//隐式设置拨号器的动作
    	intent.setAction(Intent.ACTION_DIAL);
    	startActivity(intent);
    }

    /**
     * 隐式跳转至secondActivity
     * @param v
     */
    public void click5(View v){
    	Intent intent = new Intent();
    	intent.setAction("com.itheima.sa2");
//    	intent.setData(Uri.parse("heima2:qwe"));
//    	intent.setType("text/username");
//    	intent.setData(Uri.parse("heima2:qwe123"));

    	intent.setDataAndType(Uri.parse("heima2:qwe123"), "text/username");
    	//系统会自动添加默认的category
    	intent.addCategory(Intent.CATEGORY_DEFAULT);
    	startActivity(intent);
    }

    /**
     * 显式跳转至浏览器
     */
    public void click6(View v){
    	Intent intent = new Intent();
    	intent.setClassName("com.android.browser", "com.android.browser.BrowserActivity");
    	startActivity(intent);
    }
    /**
     * 隐式跳转至浏览器
     * @param v
     */
    public void click7(View v){
    	Intent intent = new Intent();
    	intent.setAction(Intent.ACTION_VIEW);
    	intent.setData(Uri.parse("http://www.baidu.com"));
    	startActivity(intent);
    }
}

```
# 案例2：Activity在跳转时携带数据 ##

```java

public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }


    public void click(View v){
    	Intent intent = new Intent(this, SecondActivity.class);
    	//把数据封装至intent对象中
//    	intent.putExtra("malename", "李志");
//    	intent.putExtra("femalename", "芙蓉姐姐");

    	//把数据封装至bundle对象中
    	Bundle bundle = new Bundle();
    	bundle.putString("malename", "李志");
    	bundle.putString("femalename", "芙蓉姐姐");

    	//把bundle对象封装至intent对象中
    	intent.putExtras(bundle);
    	startActivity(intent);
    }

}

```

```java
public class SecondActivity extends Activity {

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		// TODO Auto-generated method stub
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_second);

		Intent intent = getIntent();
		//从intent对象中把封装好的数据取出来
//		String maleName = intent.getStringExtra("malename");
//		String feMaleName = intent.getStringExtra("femalename");

		Bundle bundle = intent.getExtras();
		String maleName = bundle.getString("malename");
		String feMaleName = bundle.getString("femalename");

		Random rd = new Random();
		int yinyuan = rd.nextInt(100);

		TextView tv = (TextView) findViewById(R.id.tv);
		tv.setText(maleName + "和" + feMaleName + "的姻缘值为" + yinyuan);
	}
}

```
# 案例3：Activity获取返回值

```java
public class MainActivity extends Activity {

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);

	}

	public void click(View c){
		//跳转至选择联系人Activity
		Intent intent = new Intent(this, ContactActivity.class);
//		startActivity(intent);
		//用这个api启动的Activity，在销毁时，系统会回调onActivityResult
		startActivityForResult(intent, 10);
	}


	public void click2(View v){
		//跳转至选择快捷回复的Activity
		Intent intent = new Intent(this, CallbackActivity.class);
		startActivityForResult(intent, 20);
	}
	//如果有Activity在销毁时返回了数据，那么就会调用此方法来接收数据
	//requestCode:用来区分数据来自于哪一个Activity
	//resultCode:用来区分返回的数据是什么类型的
	@Override
	protected void onActivityResult(int requestCode, int resultCode, Intent data) {
		// TODO Auto-generated method stub
		super.onActivityResult(requestCode, resultCode, data);

		String name = data.getStringExtra("name");
		if(requestCode == 10){
			EditText et = (EditText)findViewById(R.id.et);
			et.setText(name);
		}
		else if(requestCode == 20){
			EditText et_content = (EditText)findViewById(R.id.et_content);
			et_content.setText(name);
		}
	}
}
```

```java
public class ContactActivity extends Activity {

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		// TODO Auto-generated method stub
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_contact);

		ListView lv = (ListView) findViewById(R.id.lv);

		final String[] objects = new String[]{
				"小志",
				"逼哥",
				"世界级XXX",
				"国服第一"
		};

		lv.setAdapter(new ArrayAdapter<String>(this, R.layout.item_listview, R.id.tv, objects));

		//给listview设置条目的点击侦听
		lv.setOnItemClickListener(new OnItemClickListener() {

			//当某个条目被点击时，此方法调用
			@Override
			public void onItemClick(AdapterView<?> parent, View view,
					int position, long id) {

				//Activity返回时传递数据，也是通过意图对象
				Intent data = new Intent();
				//把要传递的数据封装至意图对象中
				data.putExtra("name", objects[position]);

				//当前Activity销毁时，data这个意图就会传递给启动当前Activity的那个Activity
				setResult(1, data);

				//销毁当前Activity
				finish();
			}
		});
	}

	@Override
	public void onBackPressed() {
		// TODO Auto-generated method stub
		super.onBackPressed();
	}
}
```

```java
public class CallbackActivity extends Activity {

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		// TODO Auto-generated method stub
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_contact);

		ListView lv = (ListView) findViewById(R.id.lv);

		final String[] objects = new String[]{
				"免谈，没戏，滚犊子",
				"媳妇我错了，求原谅",
				"老子才是一家之主"
		};
		lv.setAdapter(new ArrayAdapter<String>(this, R.layout.item_listview, R.id.tv, objects));

		lv.setOnItemClickListener(new OnItemClickListener() {

			@Override
			public void onItemClick(AdapterView<?> parent, View view,int position, long id) {
				Intent data = new Intent();
				data.putExtra("name", objects[position]);
				setResult(2, data);
				finish();
			}
		});
	}
}
```
