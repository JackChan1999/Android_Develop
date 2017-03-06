# 1. 网络图片查看器

- 确定图片的网址
- 发送http请求

```java
URL url = new URL(address);
//获取连接对象，并没有建立连接
HttpURLConnection conn = (HttpURLConnection) url.openConnection();
//设置连接和读取超时
conn.setConnectTimeout(5000);
conn.setReadTimeout(5000);
//设置请求方法，注意必须大写
conn.setRequestMethod("GET");
//建立连接，发送get请求
//conn.connect();
//建立连接，然后获取响应吗，200说明请求成功
conn.getResponseCode();
```

-  服务器的图片是以流的形式返回给浏览器的 

```java
//拿到服务器返回的输入流
InputStream is = conn.getInputStream();
//把流里的数据读取出来，并构造成图片
Bitmap bm = BitmapFactory.decodeStream(is);
```

-  把图片设置为ImageView的显示内容

```java
ImageView iv = (ImageView) findViewById(R.id.iv);
iv.setImageBitmap(bm);
```

-  添加网络权限

##1.1 主线程不能被阻塞
-  在Android中，主线程被阻塞会导致应用不能刷新ui界面，不能响应用户操作，用户体验将非常差
-  主线程阻塞时间过长，系统会抛出ANR异常
-  ANR：Application Not Response；应用无响应
-  任何耗时操作都不可以写在主线程
-  因为网络交互属于耗时操作，如果网速很慢，代码会阻塞，所以网络交互的代码不能运行在主线程

##1.2 只有主线程能刷新ui
-  刷新ui的代码只能运行在主线程，运行在子线程是没有任何效果的
-  如果需要在子线程中刷新ui，使用消息队列机制

##1.3 消息队列
-  Looper一旦发现Message Queue中有消息，就会把消息取出，然后把消息扔给Handler对象，Handler会调用自己的handleMessage方法来处理这条消息
-  handleMessage方法运行在主线程
-  主线程创建时，消息队列和轮询器对象就会被创建，但是消息处理器对象，需要使用时，自行创建

```java
//消息队列
Handler handler = new Handler(){
    //主线程中有一个消息轮询器looper，不断检测消息队列中是否有新消息，
    //如果发现有新消息，自动调用此方法，注意此方法是在主线程中运行的
    public void handleMessage(android.os.Message msg) {

    }
};
```

-  在子线程中往消息队列里发消息

```java
//创建消息对象
Message msg = new Message();
//消息的obj属性可以赋值任何对象，通过这个属性可以携带数据
msg.obj = bm;
//what属性相当于一个标签，用于区分出不同的消息，从而运行不能的代码
msg.what = 1;
//发送消息
handler.sendMessage(msg);
```

-  通过switch语句区分不同的消息
```java
public void handleMessage(android.os.Message msg) {
			switch (msg.what) {
			//如果是1，说明属于请求成功的消息
			case 1:
				ImageView iv = (ImageView) findViewById(R.id.iv);
				Bitmap bm = (Bitmap) msg.obj;
				iv.setImageBitmap(bm);
				break;
			case 2:
				Toast.makeText(MainActivity.this, "请求失败", 0).show();
				break;
			}		
		}
```

```java
public class MainActivity extends Activity {

	static ImageView iv;
	static MainActivity ma;
	static Handler handler = new Handler(){
		//此方法在主线程中调用，可以用来刷新ui
		public void handleMessage(android.os.Message msg) {
			//处理消息时，需要知道到底是成功的消息，还是失败的消息
			switch (msg.what) {
			case 1:
				//把位图对象显示至imageview
				iv.setImageBitmap((Bitmap)msg.obj);
				break;

			case 0:
				Toast.makeText(ma, "请求失败", 0).show();
				break;
			}
			
		}
	};
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		iv = (ImageView) findViewById(R.id.iv);
		ma = this;
	}

	public void click(View v){
		Thread t = new Thread(){
			@Override
			public void run() {
				//下载图片
				//1.确定网址
				String path = "http://192.168.13.13:8080/dd.jpg";
				try {
					//2.把网址封装成一个url对象
					URL url = new URL(path);
					//3.获取客户端和服务器的连接对象，此时还没有建立连接
					HttpURLConnection conn = (HttpURLConnection) url.openConnection();
					//4.对连接对象进行初始化
					//设置请求方法，注意大写
					conn.setRequestMethod("GET");
					//设置连接超时
					conn.setConnectTimeout(5000);
					//设置读取超时
					conn.setReadTimeout(5000);
					//5.发送请求，与服务器建立连接
					conn.connect();
					//如果响应码为200，说明请求成功
					if(conn.getResponseCode() == 200){
						//获取服务器响应头中的流，流里的数据就是客户端请求的数据
						InputStream is = conn.getInputStream();
						//读取出流里的数据，并构造成位图对象
						Bitmap bm = BitmapFactory.decodeStream(is);
						
//						ImageView iv = (ImageView) findViewById(R.id.iv);
//						//把位图对象显示至imageview
//						iv.setImageBitmap(bm);
						
						Message msg = new Message();
						//消息对象可以携带数据
						msg.obj = bm;
						msg.what = 1;
						//把消息发送至主线程的消息队列
						handler.sendMessage(msg);
						
					}
					else{
//						Toast.makeText(MainActivity.this, "请求失败", 0).show();
						
						Message msg = handler.obtainMessage();
						msg.what = 0;
						handler.sendMessage(msg);
					}
				} catch (Exception e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
		};
		t.start();
	}
}
```
## **1.4 案例1：网络图片查看器**

```java
public class MainActivity extends Activity {

	static ImageView iv;
	static MainActivity ma;
	static Handler handler = new Handler(){
		//此方法在主线程中调用，可以用来刷新ui
		public void handleMessage(android.os.Message msg) {
			//处理消息时，需要知道到底是成功的消息，还是失败的消息
			switch (msg.what) {
			case 1:
				//把位图对象显示至imageview
				iv.setImageBitmap((Bitmap)msg.obj);
				break;

			case 0:
				Toast.makeText(ma, "请求失败", 0).show();
				break;
			}
			
		}
	};
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		iv = (ImageView) findViewById(R.id.iv);
		ma = this;
	}

	public void click(View v){
		Thread t = new Thread(){
			@Override
			public void run() {
				//下载图片
				//1.确定网址
				String path = "http://192.168.13.13:8080/dd.jpg";
				try {
					//2.把网址封装成一个url对象
					URL url = new URL(path);
					//3.获取客户端和服务器的连接对象，此时还没有建立连接
					HttpURLConnection conn = (HttpURLConnection) url.openConnection();
					//4.对连接对象进行初始化
					//设置请求方法，注意大写
					conn.setRequestMethod("GET");
					//设置连接超时
					conn.setConnectTimeout(5000);
					//设置读取超时
					conn.setReadTimeout(5000);
					//5.发送请求，与服务器建立连接
					conn.connect();
					//如果响应码为200，说明请求成功
					if(conn.getResponseCode() == 200){
						//获取服务器响应头中的流，流里的数据就是客户端请求的数据
						InputStream is = conn.getInputStream();
						//读取出流里的数据，并构造成位图对象
						Bitmap bm = BitmapFactory.decodeStream(is);
						
//						ImageView iv = (ImageView) findViewById(R.id.iv);
//						//把位图对象显示至imageview
//						iv.setImageBitmap(bm);
						
						Message msg = new Message();
						//消息对象可以携带数据
						msg.obj = bm;
						msg.what = 1;
						//把消息发送至主线程的消息队列
						handler.sendMessage(msg);
						
					}
					else{
//						Toast.makeText(MainActivity.this, "请求失败", 0).show();
						
						Message msg = handler.obtainMessage();
						msg.what = 0;
						handler.sendMessage(msg);
					}
				} catch (Exception e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
		};
		t.start();	
	}
}
```
## 1.5 加入缓存图片的功能
-  把服务器返回的流里的数据读取出来，然后通过文件输入流写至本地文件

```java
//1.拿到服务器返回的输入流
InputStream is = conn.getInputStream();
//2.把流里的数据读取出来，并构造成图片
                    
FileOutputStream fos = new FileOutputStream(file);
byte[] b = new byte[1024];
int len = 0;
while((len = is.read(b)) != -1){
    fos.write(b, 0, len);
}
```

-  创建bitmap对象的代码改成
```java
Bitmap bm = BitmapFactory.decodeFile(file.getAbsolutePath());
```

-  每次发送请求前检测一下在缓存中是否存在同名图片，如果存在，则读取缓存

```java
public class MainActivity extends Activity {

	static ImageView iv;
	static MainActivity ma;
	static Handler handler = new Handler(){
		//此方法在主线程中调用，可以用来刷新ui
		public void handleMessage(android.os.Message msg) {
			//处理消息时，需要知道到底是成功的消息，还是失败的消息
			switch (msg.what) {
			case 1:
				//把位图对象显示至imageview
				iv.setImageBitmap((Bitmap)msg.obj);
				break;

			case 0:
				Toast.makeText(ma, "请求失败", 0).show();
				break;
			}
			
		}
	};
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		iv = (ImageView) findViewById(R.id.iv);
		ma = this;
	}

	public void click(View v){
		//下载图片
		//1.确定网址
		final String path = "http://192.168.13.13:8080/dd.jpg";
		final File file = new File(getCacheDir(), getFileName(path));
		//判断，缓存中是否存在该文件
		if(file.exists()){
			//如果缓存存在，从缓存读取图片
			System.out.println("从缓存读取的");
			Bitmap bm = BitmapFactory.decodeFile(file.getAbsolutePath());
			iv.setImageBitmap(bm);
		}
		else{
			//如果缓存不存在，从网络下载
			System.out.println("从网上下载的");
			Thread t = new Thread(){
				@Override
				public void run() {
					
					try {
						//2.把网址封装成一个url对象
						URL url = new URL(path);
						//3.获取客户端和服务器的连接对象，此时还没有建立连接
						HttpURLConnection conn = (HttpURLConnection) url.openConnection();
						//4.对连接对象进行初始化
						//设置请求方法，注意大写
						conn.setRequestMethod("GET");
						//设置连接超时
						conn.setConnectTimeout(5000);
						//设置读取超时
						conn.setReadTimeout(5000);
						//5.发送请求，与服务器建立连接
						conn.connect();
						//如果响应码为200，说明请求成功
						if(conn.getResponseCode() == 200){
							//获取服务器响应头中的流，流里的数据就是客户端请求的数据
							InputStream is = conn.getInputStream();
							
							//读取服务器返回的流里的数据，把数据写到本地文件，缓存起来
							
							FileOutputStream fos = new FileOutputStream(file);
							byte[] b = new byte[1024];
							int len = 0;
							while((len = is.read(b)) != -1){
								fos.write(b, 0, len);
							}
							fos.close();
							
							//读取出流里的数据，并构造成位图对象
							//流里已经没有数据了
//							Bitmap bm = BitmapFactory.decodeStream(is);
							Bitmap bm = BitmapFactory.decodeFile(file.getAbsolutePath());
							
							
							Message msg = new Message();
							//消息对象可以携带数据
							msg.obj = bm;
							msg.what = 1;
							//把消息发送至主线程的消息队列
							handler.sendMessage(msg);
							
						}
						else{
//							Toast.makeText(MainActivity.this, "请求失败", 0).show();
							
							Message msg = handler.obtainMessage();
							msg.what = 0;
							handler.sendMessage(msg);
						}
					} catch (Exception e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
				}
			};
			t.start();
		}
		
		
	}
	
	public String getFileName(String path){
		int index = path.lastIndexOf("/");
		return path.substring(index + 1);
	}

}
```
## 1.6 **案例2：带缓存的网络图片查看器**

```java
public class MainActivity extends Activity {

	static ImageView iv;
	static MainActivity ma;
	static Handler handler = new Handler(){
		//此方法在主线程中调用，可以用来刷新ui
		public void handleMessage(android.os.Message msg) {
			//处理消息时，需要知道到底是成功的消息，还是失败的消息
			switch (msg.what) {
			case 1:
				//把位图对象显示至imageview
				iv.setImageBitmap((Bitmap)msg.obj);
				break;

			case 0:
				Toast.makeText(ma, "请求失败", 0).show();
				break;
			}
			
		}
	};
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		iv = (ImageView) findViewById(R.id.iv);
		ma = this;
	}

	public void click(View v){
		//下载图片
		//1.确定网址
		final String path = "http://192.168.13.13:8080/dd.jpg";
		final File file = new File(getCacheDir(), getFileName(path));
		//判断，缓存中是否存在该文件
		if(file.exists()){
			//如果缓存存在，从缓存读取图片
			System.out.println("从缓存读取的");
			Bitmap bm = BitmapFactory.decodeFile(file.getAbsolutePath());
			iv.setImageBitmap(bm);
		}
		else{
			//如果缓存不存在，从网络下载
			System.out.println("从网上下载的");
			Thread t = new Thread(){
				@Override
				public void run() {
					
					try {
						//2.把网址封装成一个url对象
						URL url = new URL(path);
						//3.获取客户端和服务器的连接对象，此时还没有建立连接
						HttpURLConnection conn = (HttpURLConnection) url.openConnection();
						//4.对连接对象进行初始化
						//设置请求方法，注意大写
						conn.setRequestMethod("GET");
						//设置连接超时
						conn.setConnectTimeout(5000);
						//设置读取超时
						conn.setReadTimeout(5000);
						//5.发送请求，与服务器建立连接
						conn.connect();
						//如果响应码为200，说明请求成功
						if(conn.getResponseCode() == 200){
							//获取服务器响应头中的流，流里的数据就是客户端请求的数据
							InputStream is = conn.getInputStream();
							
							//读取服务器返回的流里的数据，把数据写到本地文件，缓存起来
							
							FileOutputStream fos = new FileOutputStream(file);
							byte[] b = new byte[1024];
							int len = 0;
							while((len = is.read(b)) != -1){
								fos.write(b, 0, len);
							}
							fos.close();
							
							//读取出流里的数据，并构造成位图对象
							//流里已经没有数据了
//							Bitmap bm = BitmapFactory.decodeStream(is);
							Bitmap bm = BitmapFactory.decodeFile(file.getAbsolutePath());
							
							
							Message msg = new Message();
							//消息对象可以携带数据
							msg.obj = bm;
							msg.what = 1;
							//把消息发送至主线程的消息队列
							handler.sendMessage(msg);
							
						}
						else{
//							Toast.makeText(MainActivity.this, "请求失败", 0).show();
							
							Message msg = handler.obtainMessage();
							msg.what = 0;
							handler.sendMessage(msg);
						}
					} catch (Exception e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
				}
			};
			t.start();
		}
		
		
	}
	
	public String getFileName(String path){
		int index = path.lastIndexOf("/");
		return path.substring(index + 1);
	}

}
```
## 1.7 **案例3：开源图片查看器**

```java
public class MainActivity extends Activity {

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
	}

	public void click(View v){
		//下载图片
		//1.确定网址
		String path = "http://192.168.13.13:8080/dd.jpg";
		//2.找到智能图片查看器对象
		SmartImageView siv = (SmartImageView) findViewById(R.id.iv);
		//3.下载并显示图片
		siv.setImageUrl(path);
	}

}
```
# 2. 获取开源代码的网站
-  code.google.com
-  github.com
-  在github搜索smart-image-view
-  下载开源项目smart-image-view
-  使用自定义组件时，标签名字要写包名

```xml
<com.loopj.android.image.SmartImageView/>
```

-  SmartImageView的使用

```
SmartImageView siv = (SmartImageView) findViewById(R.id.siv);
siv.setImageUrl("http://192.168.1.102:8080/dd.jpg");
```

# 3. 新闻客户端

```java
public class MainActivity extends Activity {

	List<News> newsList;
	Handler handler = new Handler(){
		public void handleMessage(android.os.Message msg) {
			ListView lv = (ListView) findViewById(R.id.lv);
			lv.setAdapter(new MyAdapter());
		}
	};
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		
		getNewsInfo();
//		ListView lv = (ListView) findViewById(R.id.lv);
//		//要保证在设置适配器时，新闻xml文件已经解析完毕了
//		lv.setAdapter(new MyAdapter());
	}

	class MyAdapter extends BaseAdapter{

		//得到模型层中元素的数量，用来确定listview需要有多少个条目
		@Override
		public int getCount() {
			// TODO Auto-generated method stub
			return newsList.size();
		}
		@Override
		//返回一个View对象，作为listview的条目显示至界面
		public View getView(int position, View convertView, ViewGroup parent) {

			News news = newsList.get(position);
			View v = null;
			ViewHolder mHolder;
			if(convertView == null){
				v = View.inflate(MainActivity.this, R.layout.item_listview, null);
				mHolder = new ViewHolder();
				//把布局文件中所有组件的对象封装至ViewHolder对象中
				mHolder.tv_title = (TextView) v.findViewById(R.id.tv_title);
				mHolder.tv_detail = (TextView) v.findViewById(R.id.tv_detail);
				mHolder.tv_comment = (TextView) v.findViewById(R.id.tv_comment);
				mHolder.siv = (SmartImageView) v.findViewById(R.id.iv);
				//把ViewHolder对象封装至View对象中
				v.setTag(mHolder);
			}
			else{
				v = convertView;
				mHolder = (ViewHolder) v.getTag();
			}
			//给三个文本框设置内容
			mHolder.tv_title.setText(news.getTitle());
			
			mHolder.tv_detail.setText(news.getDetail());
			
			mHolder.tv_comment.setText(news.getComment() + "条评论");
			
			//给新闻图片imageview设置内容
			mHolder.siv.setImageUrl(news.getImageUrl());
			return v;
		}
		
		class ViewHolder{
			//条目的布局文件中有什么组件，这里就定义什么属性
			TextView tv_title;
			TextView tv_detail;
			TextView tv_comment;
			SmartImageView siv;
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
	private void getNewsInfo() {
		Thread t = new Thread(){
			@Override
			public void run() {
				String path = "http://192.168.13.13:8080/news.xml";
				try {
					URL url = new URL(path);
					HttpURLConnection conn = (HttpURLConnection) url.openConnection();
					conn.setRequestMethod("GET");
					conn.setConnectTimeout(5000);
					conn.setReadTimeout(5000);
					//发送http GET请求，获取相应码
					if(conn.getResponseCode() == 200){
						InputStream is = conn.getInputStream();
						//使用pull解析器，解析这个流
						parseNewsXml(is);
					}
				} catch (Exception e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
		};
		t.start();
	}
	
	private void parseNewsXml(InputStream is) {
		XmlPullParser xp = Xml.newPullParser();
		try {
			xp.setInput(is, "utf-8");
			//对节点的事件类型进行判断，就可以知道当前节点是什么节点
			int type = xp.getEventType();
			News news = null;
			while(type != XmlPullParser.END_DOCUMENT){
				switch (type) {
				case XmlPullParser.START_TAG:
					if("newslist".equals(xp.getName())){
						newsList = new ArrayList<News>();
					}
					else if("news".equals(xp.getName())){
						news = new News();
					}
					else if("title".equals(xp.getName())){
						String title = xp.nextText();
						news.setTitle(title);
					}
					else if("detail".equals(xp.getName())){
						String detail = xp.nextText();
						news.setDetail(detail);
					}
					else if("comment".equals(xp.getName())){
						String comment = xp.nextText();
						news.setComment(comment);
					}
					else if("image".equals(xp.getName())){
						String image = xp.nextText();
						news.setImageUrl(image);
					}
					break;
				case XmlPullParser.END_TAG:
					if("news".equals(xp.getName())){
						newsList.add(news);
					}
					break;

				}
				//解析完当前节点后，把指针移动至下一个节点，并返回它的事件类型
				type = xp.next();
			}
			
			//发消息，让主线程设置listview的适配器，如果消息不需要携带数据，可以发送空消息
			handler.sendEmptyMessage(1);
			
//			for (News n : newsList) {
//				System.out.println(n.toString());
//			}
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}

	}

}
```
## 3.1 **案例5：新闻客户端**

```
public class MainActivity extends Activity {

	List<News> newsList;
	Handler handler = new Handler(){
		public void handleMessage(android.os.Message msg) {
			ListView lv = (ListView) findViewById(R.id.lv);
			lv.setAdapter(new MyAdapter());
		}
	};
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		
		getNewsInfo();
//		ListView lv = (ListView) findViewById(R.id.lv);
//		//要保证在设置适配器时，新闻xml文件已经解析完毕了
//		lv.setAdapter(new MyAdapter());
	}

	class MyAdapter extends BaseAdapter{

		//得到模型层中元素的数量，用来确定listview需要有多少个条目
		@Override
		public int getCount() {
			// TODO Auto-generated method stub
			return newsList.size();
		}
		@Override
		//返回一个View对象，作为listview的条目显示至界面
		public View getView(int position, View convertView, ViewGroup parent) {

			News news = newsList.get(position);
			View v = null;
			ViewHolder mHolder;
			if(convertView == null){
				v = View.inflate(MainActivity.this, R.layout.item_listview, null);
				mHolder = new ViewHolder();
				//把布局文件中所有组件的对象封装至ViewHolder对象中
				mHolder.tv_title = (TextView) v.findViewById(R.id.tv_title);
				mHolder.tv_detail = (TextView) v.findViewById(R.id.tv_detail);
				mHolder.tv_comment = (TextView) v.findViewById(R.id.tv_comment);
				mHolder.siv = (SmartImageView) v.findViewById(R.id.iv);
				//把ViewHolder对象封装至View对象中
				v.setTag(mHolder);
			}
			else{
				v = convertView;
				mHolder = (ViewHolder) v.getTag();
			}
			//给三个文本框设置内容
			mHolder.tv_title.setText(news.getTitle());
			
			mHolder.tv_detail.setText(news.getDetail());
			
			mHolder.tv_comment.setText(news.getComment() + "条评论");
			
			//给新闻图片imageview设置内容
			mHolder.siv.setImageUrl(news.getImageUrl());
			return v;
		}
		
		class ViewHolder{
			//条目的布局文件中有什么组件，这里就定义什么属性
			TextView tv_title;
			TextView tv_detail;
			TextView tv_comment;
			SmartImageView siv;
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
	private void getNewsInfo() {
		Thread t = new Thread(){
			@Override
			public void run() {
				String path = "http://192.168.13.13:8080/news.xml";
				try {
					URL url = new URL(path);
					HttpURLConnection conn = (HttpURLConnection) url.openConnection();
					conn.setRequestMethod("GET");
					conn.setConnectTimeout(5000);
					conn.setReadTimeout(5000);
					//发送http GET请求，获取相应码
					if(conn.getResponseCode() == 200){
						InputStream is = conn.getInputStream();
						//使用pull解析器，解析这个流
						parseNewsXml(is);
					}
				} catch (Exception e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
		};
		t.start();
	}
	
	private void parseNewsXml(InputStream is) {
		XmlPullParser xp = Xml.newPullParser();
		try {
			xp.setInput(is, "utf-8");
			//对节点的事件类型进行判断，就可以知道当前节点是什么节点
			int type = xp.getEventType();
			News news = null;
			while(type != XmlPullParser.END_DOCUMENT){
				switch (type) {
				case XmlPullParser.START_TAG:
					if("newslist".equals(xp.getName())){
						newsList = new ArrayList<News>();
					}
					else if("news".equals(xp.getName())){
						news = new News();
					}
					else if("title".equals(xp.getName())){
						String title = xp.nextText();
						news.setTitle(title);
					}
					else if("detail".equals(xp.getName())){
						String detail = xp.nextText();
						news.setDetail(detail);
					}
					else if("comment".equals(xp.getName())){
						String comment = xp.nextText();
						news.setComment(comment);
					}
					else if("image".equals(xp.getName())){
						String image = xp.nextText();
						news.setImageUrl(image);
					}
					break;
				case XmlPullParser.END_TAG:
					if("news".equals(xp.getName())){
						newsList.add(news);
					}
					break;

				}
				//解析完当前节点后，把指针移动至下一个节点，并返回它的事件类型
				type = xp.next();
			}
			
			//发消息，让主线程设置listview的适配器，如果消息不需要携带数据，可以发送空消息
			handler.sendEmptyMessage(1);
			
//			for (News n : newsList) {
//				System.out.println(n.toString());
//			}
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}

	}

}
```
# 4. Html源文件查看器
-  发送GET请求

```java
URL url = new URL(path);
//获取连接对象
HttpURLConnection conn = (HttpURLConnection) url.openConnection();
//设置连接属性
conn.setRequestMethod("GET");
conn.setConnectTimeout(5000);
conn.setReadTimeout(5000);
//建立连接，获取响应吗
if(conn.getResponseCode() == 200){
        
}
```

-  获取服务器返回的流，从流中把html源码读取出来

```java
byte[] b = new byte[1024];
int len = 0;
ByteArrayOutputStream bos = new ByteArrayOutputStream();
while((len = is.read(b)) != -1){
    //把读到的字节先写入字节数组输出流中存起来
    bos.write(b, 0, len);
}
//把字节数组输出流中的内容转换成字符串
//默认使用utf-8
text = new String(bos.toByteArray());
```

## 4.1 乱码的处理

-  乱码的出现是因为服务器和客户端码表不一致导致
```
//手动指定码表
text = new String(bos.toByteArray(), "gb2312");
```
## 4.2 **案例4：HTML源文件查看器**

```java
public class MainActivity extends Activity {

	Handler handler = new Handler(){
		public void handleMessage(android.os.Message msg) {
			TextView tv = (TextView) findViewById(R.id.tv);
			tv.setText((String)msg.obj);
		}
	};
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
	}

	public void click(View v){
		Thread t = new Thread(){
			@Override
			public void run() {
				String path = "http://192.168.13.13:8080/baidu.html";
				try {
					URL url = new URL(path);
					//获取连接对象，此时还未建立连接
					HttpURLConnection conn = (HttpURLConnection) url.openConnection();
					conn.setRequestMethod("GET");
					conn.setConnectTimeout(5000);
					conn.setReadTimeout(5000);
					//先建立连接，然后获取响应码
					if(conn.getResponseCode() == 200){
						//拿到服务器返回的输入流，流里的数据就是html的源文件
						InputStream is = conn.getInputStream();
						//从流里把文本数据取出来
						String text = Utils.getTextFromStream(is);
						
						//发送消息，让主线程刷新ui，显示源文件
						Message msg = handler.obtainMessage();
						msg.obj = text;
						handler.sendMessage(msg);
					}
				} catch (Exception e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
		};
		t.start();
	}

}
```
#5. 提交数据
##5.1 GET方式提交数据
-  get方式提交的数据是直接拼接在url的末尾

```
final String path = "http://192.168.1.104/Web/servlet/CheckLogin?name=" + name + "&pass=" + pass;
```

-  发送get请求，代码和之前一样

```java
URL url = new URL(path);
HttpURLConnection conn = (HttpURLConnection) url.openConnection();
conn.setRequestMethod("GET");
conn.setReadTimeout(5000);
conn.setConnectTimeout(5000);
if(conn.getResponseCode() == 200){

}
```

-  浏览器在发送请求携带数据时会对数据进行URL编码，我们写代码时也需要为中文进行URL编码

```
String path = "http://192.168.1.104/Web/servlet/CheckLogin?name=" + URLEncoder.encode(name) + "&pass=" + pass;
```

## **5.2 案例6：使用get方式提交数据**

```java
public class MainActivity extends Activity {

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
	}

	Handler handler = new Handler(){
		public void handleMessage(android.os.Message msg) {
			Toast.makeText(MainActivity.this, (String)msg.obj, 0).show();
		}
	};
	
	public void click(View v){
		EditText et_name = (EditText) findViewById(R.id.et_name);
		EditText et_pass = (EditText) findViewById(R.id.et_pass);
		
		final String name = et_name.getText().toString();
		final String pass = et_pass.getText().toString();
		
		Thread t = new Thread(){
			@Override
			public void run() {
				//提交的数据需要经过url编码，英文和数字编码后不变
				@SuppressWarnings("deprecation")
				String path = "http://192.168.13.13/Web2/servlet/LoginServlet?name=" + URLEncoder.encode(name) + "&pass=" + pass;
				
				try {
					URL url = new URL(path);
					HttpURLConnection conn = (HttpURLConnection) url.openConnection();
					conn.setRequestMethod("GET");
					conn.setConnectTimeout(5000);
					conn.setReadTimeout(5000);
					
					if(conn.getResponseCode() == 200){
						InputStream is =conn.getInputStream();
						String text = Utils.getTextFromStream(is);
						
						Message msg = handler.obtainMessage();
						msg.obj = text;
						handler.sendMessage(msg);
					}
				} catch (Exception e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
		};
		t.start();
	}

}
```

```java
public class Utils {

	public static String getTextFromStream(InputStream is){
		
		byte[] b = new byte[1024];
		int len = 0;
		//创建字节数组输出流，读取输入流的文本数据时，同步把数据写入数组输出流
		ByteArrayOutputStream bos = new ByteArrayOutputStream();
		try {
			while((len = is.read(b)) != -1){
				bos.write(b, 0, len);
			}
			//把字节数组输出流里的数据转换成字节数组
			String text = new String(bos.toByteArray());
			return text;
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		return null;
	}
}
```
## 5.3 POST方式提交数据

-  post提交数据是用流写给服务器的
-  协议头中多了两个属性
   -  Content-Type: application/x-www-form-urlencoded，描述提交的数据的mimetype
   -  Content-Length: 32，描述提交的数据的长度

```java
//给请求头添加post多出来的两个属性
String data = "name=" + URLEncoder.encode(name) + "&pass=" + pass;
conn.setRequestProperty("Content-Type", "application/x-www-form-urlencoded");
conn.setRequestProperty("Content-Length", data.length() + "");
```

-  设置允许打开post请求的流

```java
conn.setDoOutput(true);
```

-  获取连接对象的输出流，往流里写要提交给服务器的数据

```java
OutputStream os = conn.getOutputStream();
os.write(data.getBytes());
```

## 5.4 **案例7：使用post方式提交数据**

```java
public class MainActivity extends Activity {

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
	}

	Handler handler = new Handler(){
		public void handleMessage(android.os.Message msg) {
			Toast.makeText(MainActivity.this, (String)msg.obj, 0).show();
		}
	};
	
	public void click(View v){
		EditText et_name = (EditText) findViewById(R.id.et_name);
		EditText et_pass = (EditText) findViewById(R.id.et_pass);
		
		final String name = et_name.getText().toString();
		final String pass = et_pass.getText().toString();
		
		Thread t = new Thread(){
			@Override
			public void run() {
				//提交的数据需要经过url编码，英文和数字编码后不变
				@SuppressWarnings("deprecation")
				String path = "http://192.168.13.13/Web2/servlet/LoginServlet";
				
				try {
					URL url = new URL(path);
					HttpURLConnection conn = (HttpURLConnection) url.openConnection();
					conn.setRequestMethod("POST");
					conn.setConnectTimeout(5000);
					conn.setReadTimeout(5000);
					
					//拼接出要提交的数据的字符串
					String data = "name=" + URLEncoder.encode(name) + "&pass=" + pass;
					//添加post请求的两行属性
					conn.setRequestProperty("Content-Type", "application/x-www-form-urlencoded");
					conn.setRequestProperty("Content-Length", data.length() + "");
					
					//设置打开输出流
					conn.setDoOutput(true);
					//拿到输出流
					OutputStream os = conn.getOutputStream();
					//使用输出流往服务器提交数据
					os.write(data.getBytes());
					if(conn.getResponseCode() == 200){
						InputStream is = conn.getInputStream();
						String text = Utils.getTextFromStream(is);
						
						Message msg = handler.obtainMessage();
						msg.obj = text;
						handler.sendMessage(msg);
					}
				} catch (Exception e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
		};
		t.start();
	}
}
```
# 6. 网络请求

## 6.1 主线程阻塞

-  UI停止刷新，应用无法响应用户操作
-  耗时操作不应该在主线程进行
-  ANR
   -  application not responding
   -  应用无响应异常
   -  主线程阻塞时间过长，就会抛出ANR

-  主线程又称UI线程，因为只有在主线程中，才能刷新UI

## 6.2 消息队列机制

-  主线程创建时，系统会同时创建消息队列对象（MessageQueue）和消息轮询器对象（Looper）
-  轮询器的作用，就是不停的检测消息队列中是否有消息（Message）
-  消息队列一旦有消息，轮询器会把消息对象传给消息处理器（Handler），处理器会调用handleMessage方法来处理这条消息，handleMessage方法运行在主线程中，所以可以刷新ui
-  总结：只要消息队列有消息，handleMessage方法就会调用
-  子线程如果需要刷新ui，只需要往消息队列中发一条消息，触发handleMessage方法即可
-  子线程使用处理器对象的sendMessage方法发送消息