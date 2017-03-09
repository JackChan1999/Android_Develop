

# 测试
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
# 单元测试junit
* 定义一个类继承AndroidTestCase，在类中定义方法，即可测试该方法


* 在指定指令集时，targetPackage指定你要测试的应用的包名

```xml
<instrumentation 
android:name="android.test.InstrumentationTestRunner"
android:targetPackage="com.itheima.junit"
></instrumentation>
```

* 定义使用的类库

```xml
<uses-library android:name="android.test.runner"></uses-library>
```

* 断言的作用，检测运行结果和预期是否一致
* 如果应用出现异常，会抛给测试框架

-----
#SQLite数据库
* 轻量级关系型数据库
* 创建数据库需要使用的api：SQLiteOpenHelper
  * 必须定义一个构造方法：

```java
//arg1:数据库文件的名字
//arg2:游标工厂
//arg3:数据库版本
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
	// TODO Auto-generated method stub
	db.execSQL("create table person (_id integer primary key autoincrement, name char(10), phone char(20), money integer(20))");
}
```
---
#数据库的增删改查
###SQL语句
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
```

* 测试方法执行前会调用此方法

```java
protected void setUp() throws Exception {
	super.setUp();
	//					获取虚拟上下文对象
	oh = new MyOpenHelper(getContext(), "people.db", null, 1);
}
```

## 使用api实现增删改查
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

  	//返回值是删除的行数
  	int i = db.delete("person", "_id = ? and name = ?", new String[]{"1", "张三"});
* 修改

  	ContentValues cv = new ContentValues();
  	cv.put("money", 25000);
  	int i = db.update("person", cv, "name = ?", new String[]{"赵四"});
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
try {
	//开启事务
	db.beginTransaction();
	...........
	//设置事务执行成功
	db.setTransactionSuccessful();
} finally{
	//关闭事务
	//如果此时已经设置事务执行成功，则sql语句生效，否则不生效
	db.endTransaction();
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
###BaseAdapter
* 必须实现的两个方法

  * 第一个

    	//系统调用此方法，用来获知模型层有多少条数据
    	@Override
    	public int getCount() {
    		return people.size();
    	}

  * 第二个

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
* 屏幕上能显示多少个条目，getView方法就会被调用多少次，屏幕向下滑动时，getView会继续被调用，创建更多的View对象显示至屏幕
###条目的缓存
* 当条目划出屏幕时，系统会把该条目缓存至内存，当该条目再次进入屏幕，系统在重新调用getView时会把缓存的条目作为convertView参数传入，但是传入的条目不一定是之前被缓存的该条目，即系统有可能在调用getView方法获取第一个条目时，传入任意一个条目的缓存

---
#对话框
###确定取消对话框
* 创建对话框构建器对象，类似工厂模式
* 
  	AlertDialog.Builder builder = new Builder(this);
* 设置标题和正文
* 
    	builder.setTitle("警告");
    	builder.setMessage("若练此功，必先自宫");
* 设置确定和取消按钮

    	builder.setPositiveButton("现在自宫", new OnClickListener() {
  ​		
  		@Override
  		public void onClick(DialogInterface dialog, int which) {
  			// TODO Auto-generated method stub
  			Toast.makeText(MainActivity.this, "恭喜你自宫成功，现在程序退出", 0).show();
  		}
  	});

    	builder.setNegativeButton("下次再说", new OnClickListener() {
  ​		
  		@Override
  		public void onClick(DialogInterface dialog, int which) {
  			// TODO Auto-generated method stub
  			Toast.makeText(MainActivity.this, "若不自宫，一定不成功", 0).show();
  		}
  	});
* 使用构建器创建出对话框对象

    	AlertDialog ad = builder.create();
    	ad.show();

###单选对话框

		AlertDialog.Builder builder = new Builder(this);
		builder.setTitle("选择你的性别");
* 定义单选选项
* 
    	final String[] items = new String[]{
    			"男", "女", "其他"
    	};
  	//-1表示没有默认选择
  	//点击侦听的导包要注意别导错
    	builder.setSingleChoiceItems(items, -1, new OnClickListener() {
  ​		
  		//which表示点击的是哪一个选项
  		@Override
  		public void onClick(DialogInterface dialog, int which) {
  			Toast.makeText(MainActivity.this, "您选择了" + items[which], 0).show();
  			//对话框消失
  			dialog.dismiss();
  		}
    	});
    	
    	builder.show();

###多选对话框

		AlertDialog.Builder builder = new Builder(this);
		builder.setTitle("请选择你认为最帅的人");
* 定义多选的选项，因为可以多选，所以需要一个boolean数组来记录哪些选项被选了
* 
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
  ​		
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


----------
## 随堂笔记 ##
#测试
###按岗位划分
* 黑盒测试：测试逻辑业务
* 白盒测试：测试逻辑方法
###按测试粒度分
* 方法测试：function test
* 单元测试：unit test
* 集成测试：integration test
* 系统测试：system test
###按测试的暴力程度分
* 冒烟测试：smoke test
* 压力测试：pressure test

---
#单元测试
* junit
* 在清单文件中指定指令集

  	<instrumentation 
  	  android:name="android.test.InstrumentationTestRunner"
  	//指定该测试框架要测试哪一个项目
  	  android:targetPackage="com.itheima.junit"
  	  ></instrumentation>
* 定义使用的类库

  	<uses-library android:name="android.test.runner"/>

---
#SQLite
###事务
> 保证所有sql语句要么一起成功，要么一起失败

---
#ListView
###MVC架构
* M：模型层 ---- javaBean ---- personList
* V：视图层 ---- jsp      ---- ListView
* C：控制层 ---- servlet  ---- Adapter
###Adapter
* ListView的每个条目都是一个View对象
## 案例1：SQLite数据库 ##

```
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

```
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
## 案例2：把数据显示至屏幕

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

