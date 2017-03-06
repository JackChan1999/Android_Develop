# 网络编程
- [Java基础：网络编程](http://blog.csdn.net/axi295309066/article/details/52854772)
- [Uri、URL、UriMatcher、ContentUris详解](http://blog.csdn.net/axi295309066/article/details/60129690)
- [Android应用开发：网络编程1](http://blog.csdn.net/axi295309066/article/details/50315017)
- [Android应用开发：网络编程2](http://blog.csdn.net/axi295309066/article/details/50330375)

# **1. HttpClient**
## **1.1 发送get请求**
* 创建一个客户端对象

```java
HttpClient client = new DefaultHttpClient();

```

* 创建一个get请求对象

```java
HttpGet hg = new HttpGet(path);
```

* 发送get请求，建立连接，返回响应头对象

```java
HttpResponse hr = hc.execute(hg);
```

* 获取状态行对象，获取状态码，如果为200则说明请求成功

```java
if(hr.getStatusLine().getStatusCode() == 200){
    //拿到服务器返回的输入流
    InputStream is = hr.getEntity().getContent();
    String text = Utils.getTextFromStream(is);
}
```

## **1.2 发送post请求**

```java
//创建一个客户端对象
HttpClient client = new DefaultHttpClient();
//创建一个post请求对象
HttpPost hp = new HttpPost(path);
```

* 往post对象里放入要提交给服务器的数据

```java
//要提交的数据以键值对的形式存在BasicNameValuePair对象中
List<NameValuePair> parameters = new ArrayList<NameValuePair>();
BasicNameValuePair bnvp = new BasicNameValuePair("name", name);
BasicNameValuePair bnvp2 = new BasicNameValuePair("pass", pass);
parameters.add(bnvp);
parameters.add(bnvp2);
//创建实体对象，指定进行URL编码的码表
UrlEncodedFormEntity entity = new UrlEncodedFormEntity(parameters, "utf-8");
//为post请求设置实体
hp.setEntity(entity);
```

## **1.3 案例1：HttpClient框架提交数据**

```java
public class MainActivity extends Activity {

	Handler handler = new Handler(){
		@Override
		public void handleMessage(android.os.Message msg) {
			Toast.makeText(MainActivity.this, (String)msg.obj, 0).show();
		}
	};
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }


    public void get(View v){
    	EditText et_name = (EditText) findViewById(R.id.et_name);
    	EditText et_pass = (EditText) findViewById(R.id.et_pass);

    	final String name = et_name.getText().toString();
    	final String pass = et_pass.getText().toString();


    	Thread t = new Thread(){
    		@Override
    		public void run() {
    			String path = "http://192.168.13.13/Web/servlet/CheckLogin?name=" + URLEncoder.encode(name) + "&pass=" + pass;
    	    	//使用httpClient框架做get方式提交
    	    	//1.创建HttpClient对象
    	    	HttpClient hc = new DefaultHttpClient();

    	    	//2.创建httpGet对象，构造方法的参数就是网址
    	    	HttpGet hg = new HttpGet(path);

    	    	//3.使用客户端对象，把get请求对象发送出去
    	    	try {
    				HttpResponse hr = hc.execute(hg);
    				//拿到响应头中的状态行
    				StatusLine sl = hr.getStatusLine();
    				if(sl.getStatusCode() == 200){
    					//拿到响应头的实体
    					HttpEntity he = hr.getEntity();
    					//拿到实体中的内容，其实就是服务器返回的输入流
    					InputStream is = he.getContent();
    					String text = Utils.getTextFromStream(is);

    					//发送消息，让主线程刷新ui显示text
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

    public void post(View v){
    	EditText et_name = (EditText) findViewById(R.id.et_name);
    	EditText et_pass = (EditText) findViewById(R.id.et_pass);

    	final String name = et_name.getText().toString();
    	final String pass = et_pass.getText().toString();

    	Thread t = new Thread(){
    		@Override
    		public void run() {
    			String path = "http://192.168.13.13/Web/servlet/CheckLogin";
    	    	//1.创建客户端对象
    	    	HttpClient hc = new DefaultHttpClient();
    	    	//2.创建post请求对象
    	    	HttpPost hp = new HttpPost(path);

    	    	//封装form表单提交的数据
    	    	BasicNameValuePair bnvp = new BasicNameValuePair("name", name);
    	    	BasicNameValuePair bnvp2 = new BasicNameValuePair("pass", pass);
    	    	List<BasicNameValuePair> parameters = new ArrayList<BasicNameValuePair>();
    	    	//把BasicNameValuePair放入集合中
    	    	parameters.add(bnvp);
    	    	parameters.add(bnvp2);

    	    	try {
    	    		//要提交的数据都已经在集合中了，把集合传给实体对象
    		    	UrlEncodedFormEntity entity = new UrlEncodedFormEntity(parameters, "utf-8");
    		    	//设置post请求对象的实体，其实就是把要提交的数据封装至post请求的输出流中
    		    	hp.setEntity(entity);
    		    	//3.使用客户端发送post请求
    				HttpResponse hr = hc.execute(hp);
    				if(hr.getStatusLine().getStatusCode() == 200){
    					InputStream is = hr.getEntity().getContent();
    					String text = Utils.getTextFromStream(is);

    					//发送消息，让主线程刷新ui显示text
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
if (code == 200) {// 数据获取成功
    // 获取读取的字节流
    InputStream is = conn.getInputStream();
    // 把字节流转换成字符流
    bfr = new BufferedReader(new InputStreamReader(is));
    // 读取一行信息
    String line = bfr.readLine();
    // json字符串数据的封装
    StringBuilder json = new StringBuilder();
    while (line != null) {
        json.append(line);
        line = bfr.readLine();
    }
    parseJson = parseJson(json);//返回数据封装信息

}
```

```java
public class Utils {

	public static String getTextFromStream(InputStream is){

		byte[] b = new byte[1024];
		int len = 0;
		ByteArrayOutputStream bos = new ByteArrayOutputStream();
		try {
			while((len = is.read(b)) != -1){
				bos.write(b, 0, len);
			}
			String text = new String(bos.toByteArray());
			bos.close();
			return text;
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		return null;
	}
}
```
StreamUtils.java
```java
public class StreamUtils {

	/**
	 * 将输入流读取成String后返回
	 *
	 * @param in
	 * @return
	 * @throws IOException
	 */
	public static String readFromStream(InputStream in) throws IOException {
		ByteArrayOutputStream out = new ByteArrayOutputStream();
		int len = 0;
		byte[] buffer = new byte[1024];

		while ((len = in.read(buffer)) != -1) {
			out.write(buffer, 0, len);
		}

		String result = out.toString();
		in.close();
		out.close();
		return result;
	}
}
```
# **2. 异步HttpClient框架**

## **2.1 发送get请求**

```java
//创建异步的httpclient对象
AsyncHttpClient ahc = new AsyncHttpClient();
//发送get请求
ahc.get(path, new MyHandler());
```
* 注意AsyncHttpResponseHandler两个方法的调用时机

```java
class MyHandler extends AsyncHttpResponseHandler{

    //http请求成功，返回码为200，系统回调此方法
    @Override
    public void onSuccess(int statusCode, Header[] headers,
            //responseBody的内容就是服务器返回的数据
            byte[] responseBody) {
        Toast.makeText(MainActivity.this, new String(responseBody), 0).show();

    }

    //http请求失败，返回码不为200，系统回调此方法
    @Override
    public void onFailure(int statusCode, Header[] headers,
            byte[] responseBody, Throwable error) {
        Toast.makeText(MainActivity.this, "返回码不为200", 0).show();   
    }
}
```

## **2.2 发送post请求**

* 使用RequestParams对象封装要携带的数据

```java
//创建异步httpclient对象
AsyncHttpClient ahc = new AsyncHttpClient();
//创建RequestParams封装要携带的数据
RequestParams rp = new RequestParams();
rp.add("name", name);
rp.add("pass", pass);
//发送post请求
ahc.post(path, rp, new MyHandler());
```

## **2.3 案例2：异步HttpClient框架**

```java
public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }


    public void get(View v){
    	EditText et_name = (EditText) findViewById(R.id.et_name);
    	EditText et_pass = (EditText) findViewById(R.id.et_pass);

    	final String name = et_name.getText().toString();
    	final String pass = et_pass.getText().toString();
    	String url = "http://192.168.13.13/Web/servlet/CheckLogin?name=" + URLEncoder.encode(name) + "&pass=" + pass;
    	//创建异步httpclient
    	AsyncHttpClient ahc = new AsyncHttpClient();

    	//发送get请求提交数据
    	ahc.get(url, new MyResponseHandler());
    }

    public void post(View v){
    	EditText et_name = (EditText) findViewById(R.id.et_name);
    	EditText et_pass = (EditText) findViewById(R.id.et_pass);

    	final String name = et_name.getText().toString();
    	final String pass = et_pass.getText().toString();
    	String url = "http://192.168.13.13/Web/servlet/CheckLogin";

    	//创建异步httpclient
    	AsyncHttpClient ahc = new AsyncHttpClient();

    	//发送post请求提交数据
    	//把要提交的数据封装至RequestParams对象
    	RequestParams params = new RequestParams();
    	params.add("name", name);
    	params.add("pass", pass);
    	ahc.post(url, params, new MyResponseHandler());
    }

    class MyResponseHandler extends AsyncHttpResponseHandler{

    	//请求服务器成功时，此方法调用
		@Override
		public void onSuccess(int statusCode, Header[] headers,
				byte[] responseBody) {
			Toast.makeText(MainActivity.this, new String(responseBody), 0).show();

		}

		//请求失败此方法调用
		@Override
		public void onFailure(int statusCode, Header[] headers,
				byte[] responseBody, Throwable error) {
			Toast.makeText(MainActivity.this, "请求失败", 0).show();

		}

    }

}
```
# **3. 多线程下载**

原理：服务器CPU分配给每条线程的时间片相同，服务器带宽平均分配给每条线程，所以客户端开启的线程越多，就能抢占到更多的服务器资源

## **3.1 确定每条线程下载多少数据**
* 发送http请求至下载地址

```java
String path = "http://192.168.1.102:8080/editplus.exe";     
URL url = new URL(path);
HttpURLConnection conn = (HttpURLConnection) url.openConnection();
conn.setReadTimeout(5000);
conn.setConnectTimeout(5000);
conn.setRequestMethod("GET");  	
```
* 获取文件总长度，然后创建长度一致的临时文件
```java
if(conn.getResponseCode() == 200){
//获得服务器流中数据的长度
int length = conn.getContentLength();
//创建一个临时文件存储下载的数据
RandomAccessFile raf = new RandomAccessFile(getFileName(path), "rwd");
//设置临时文件的大小
raf.setLength(length);
raf.close();
```
* 确定线程下载多少数据
```java
//计算每个线程下载多少数据
int blockSize = length / THREAD_COUNT;
```

## **3.2 计算每条线程下载数据的开始位置和结束位置**

```java
for(int id = 1; id <= 3; id++){
    //计算每个线程下载数据的开始位置和结束位置
    int startIndex = (id - 1) * blockSize;
    int endIndex = id * blockSize - 1;
    if(id == THREAD_COUNT){
        endIndex = length;
    }

    //开启线程，按照计算出来的开始结束位置开始下载数据
    new DownLoadThread(startIndex, endIndex, id).start();
}
```

## **3.3 再次发送请求至下载地址，请求开始位置至结束位置的数据**

```java
String path = "http://192.168.1.102:8080/editplus.exe";

URL url = new URL(path);
HttpURLConnection conn = (HttpURLConnection) url.openConnection();
conn.setReadTimeout(5000);
conn.setConnectTimeout(5000);
conn.setRequestMethod("GET");

//向服务器请求部分数据
conn.setRequestProperty("Range", "bytes=" + startIndex + "-" + endIndex);
conn.connect();
```
* 下载请求到的数据，存放至临时文件中
```java
if(conn.getResponseCode() == 206){
    InputStream is = conn.getInputStream();
    RandomAccessFile raf = new RandomAccessFile(getFileName(path), "rwd");
    //指定从哪个位置开始存放数据
    raf.seek(startIndex);
    byte[] b = new byte[1024];
    int len;
    while((len = is.read(b)) != -1){
        raf.write(b, 0, len);
    }
    raf.close();
}
```

## **3.4 案例3：多线程下载**

```java
public class MultiDownload {

	static int ThreadCount = 3;
	static int finishedThread = 0;
	//确定下载地址
	static String path = "http://192.168.13.13:8080/QQPlayer.exe";
	public static void main(String[] args) {

		//发送get请求，请求这个地址的资源
		try {
			URL url = new URL(path);
			HttpURLConnection conn = (HttpURLConnection) url.openConnection();
			conn.setRequestMethod("GET");
			conn.setConnectTimeout(5000);
			conn.setReadTimeout(5000);

			if(conn.getResponseCode() == 200){
				//拿到所请求资源文件的长度
				int length = conn.getContentLength();

				File file = new File("QQPlayer.exe");
				//生成临时文件
				RandomAccessFile raf = new RandomAccessFile(file, "rwd");
				//设置临时文件的大小
				raf.setLength(length);
				raf.close();
				//计算出每个线程应该下载多少字节
				int size = length / ThreadCount;

				for (int i = 0; i < ThreadCount; i++) {
					//计算线程下载的开始位置和结束位置
					int startIndex = i * size;
					int endIndex = (i + 1) * size - 1;
					//如果是最后一个线程，那么结束位置写死
					if(i == ThreadCount - 1){
						endIndex = length - 1;
					}
//					System.out.println("线程" + i + "的下载区间是：" + startIndex + "---" + endIndex);
					new DownLoadThread(startIndex, endIndex, i).start();
				}
			}
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}


}
class DownLoadThread extends Thread{
	int startIndex;
	int endIndex;
	int threadId;

	public DownLoadThread(int startIndex, int endIndex, int threadId) {
		super();
		this.startIndex = startIndex;
		this.endIndex = endIndex;
		this.threadId = threadId;
	}

	@Override
	public void run() {
		//再次发送http请求，下载原文件
		try {
			File progressFile = new File(threadId + ".txt");
			//判断进度临时文件是否存在
			if(progressFile.exists()){
				FileInputStream fis = new FileInputStream(progressFile);
				BufferedReader br = new BufferedReader(new InputStreamReader(fis));
				//从进度临时文件中读取出上一次下载的总进度，然后与原本的开始位置相加，得到新的开始位置
				startIndex += Integer.parseInt(br.readLine());
				fis.close();
			}
			System.out.println("线程" + threadId + "的下载区间是：" + startIndex + "---" + endIndex);
			HttpURLConnection conn;
			URL url = new URL(MultiDownload.path);
			conn = (HttpURLConnection) url.openConnection();
			conn.setRequestMethod("GET");
			conn.setConnectTimeout(5000);
			conn.setReadTimeout(5000);
			//设置本次http请求所请求的数据的区间
			conn.setRequestProperty("Range", "bytes=" + startIndex + "-" + endIndex);

			//请求部分数据，相应码是206
			if(conn.getResponseCode() == 206){
				//流里此时只有1/3原文件的数据
				InputStream is = conn.getInputStream();
				byte[] b = new byte[1024];
				int len = 0;
				int total = 0;
				//拿到临时文件的输出流
				File file = new File("QQPlayer.exe");
				RandomAccessFile raf = new RandomAccessFile(file, "rwd");
				//把文件的写入位置移动至startIndex
				raf.seek(startIndex);
				while((len = is.read(b)) != -1){
					//每次读取流里数据之后，同步把数据写入临时文件
					raf.write(b, 0, len);
					total += len;
//					System.out.println("线程" + threadId + "下载了" + total);

					//生成一个专门用来记录下载进度的临时文件
					RandomAccessFile progressRaf = new RandomAccessFile(progressFile, "rwd");
					//每次读取流里数据之后，同步把当前线程下载的总进度写入进度临时文件中
					progressRaf.write((total + "").getBytes());
					progressRaf.close();
				}
				System.out.println("线程" + threadId + "下载完毕-------------------小志参上！");
				raf.close();

				MultiDownload.finishedThread++;
				synchronized (MultiDownload.path) {
					if(MultiDownload.finishedThread == MultiDownload.ThreadCount){
						for (int i = 0; i < MultiDownload.ThreadCount; i++) {
							File f = new File(i + ".txt");
							f.delete();
						}
						MultiDownload.finishedThread = 0;
					}
				}

			}
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
}
```
# **4. 带断点续传的多线程下载**

* 定义一个int变量记录每条线程下载的数据总长度，然后加上该线程的下载开始位置，得到的结果就是下次下载时，该线程的开始位置，把得到的结果存入缓存文件

```java
//用来记录当前线程总的下载长度
int total = 0;
while((len = is.read(b)) != -1){
    raf.write(b, 0, len);
    total += len;
    //每次下载都把新的下载位置写入缓存文本文件
    RandomAccessFile raf2 = new RandomAccessFile(threadId + ".txt", "rwd");
    raf2.write((startIndex + total + "").getBytes());
    raf2.close();
}
```

* 下次下载开始时，先读取缓存文件中的值，得到的值就是该线程新的开始位置

```java
FileInputStream fis = new FileInputStream(file);
BufferedReader br = new BufferedReader(new InputStreamReader(fis));
String text = br.readLine();
int newStartIndex = Integer.parseInt(text);
//把读到的值作为新的开始位置
startIndex = newStartIndex;
fis.close();
```
* 三条线程都下载完毕之后，删除缓存文件

```
RUNNING_THREAD--;
if(RUNNING_THREAD == 0){
    for(int i = 0; i <= 3; i++){
        File f = new File(i + ".txt");
        f.delete();
    }
}
```

# **5. 手机版的断点续传多线程下载器**
* 把刚才的代码直接粘贴过来就能用，记得在访问文件时的路径要改成Android的目录，添加访问网络和外部存储的路径
## **5.1 用进度条显示下载进度**
* 拿到下载文件总长度时，设置进度条的最大值

```java
pb.setMax(length);//设置进度条的最大值
```

* 进度条需要显示三条线程的整体下载进度，所以三条线程每下载一次，就要把新下载的长度加入进度条
  * 定义一个int全局变量，记录三条线程的总下载长度
```
int progress;
```

* 刷新进度条
```java
while((len = is.read(b)) != -1){
raf.write(b, 0, len);

//把当前线程本次下载的长度加到进度条里
progress += len;
pb.setProgress(progress);
```

* 每次断点下载时，从新的开始位置开始下载，进度条也要从新的位置开始显示，在读取缓存文件获取新的下载开始位置时，也要处理进度条进度

```java
FileInputStream fis = new FileInputStream(file);
BufferedReader br = new BufferedReader(new InputStreamReader(fis));
String text = br.readLine();
int newStartIndex = Integer.parseInt(text);

//新开始位置减去原本的开始位置，得到已经下载的数据长度
int alreadyDownload = newStartIndex - startIndex;
//把已经下载的长度设置入进度条
progress += alreadyDownload;
```

## **5.2 添加文本框显示百分比进度**

```java
tv.setText(progress * 100 / pb.getMax() + "%");
```

## **5.3 案例4：断点续传**

```java
public class MainActivity extends Activity {

	static int ThreadCount = 3;
	static int finishedThread = 0;

	int currentProgress;
	String fileName = "QQPlayer.exe";
	//确定下载地址
	String path = "http://192.168.13.13:8080/" + fileName;
	private ProgressBar pb;
	TextView tv;

	Handler handler = new Handler(){
		public void handleMessage(android.os.Message msg) {
			//把变量改成long，在long下运算
			tv.setText((long)pb.getProgress() * 100 / pb.getMax() + "%");
		}
	};
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		pb = (ProgressBar) findViewById(R.id.pb);
		tv = (TextView) findViewById(R.id.tv);
	}

	public void click(View v){

		Thread t = new Thread(){
			@Override
			public void run() {
				//发送get请求，请求这个地址的资源
				try {
					URL url = new URL(path);
					HttpURLConnection conn = (HttpURLConnection) url.openConnection();
					conn.setRequestMethod("GET");
					conn.setConnectTimeout(5000);
					conn.setReadTimeout(5000);

					if(conn.getResponseCode() == 200){
						//拿到所请求资源文件的长度
						int length = conn.getContentLength();

						//设置进度条的最大值就是原文件的总长度
						pb.setMax(length);

						File file = new File(Environment.getExternalStorageDirectory(), fileName);
						//生成临时文件
						RandomAccessFile raf = new RandomAccessFile(file, "rwd");
						//设置临时文件的大小
						raf.setLength(length);
						raf.close();
						//计算出每个线程应该下载多少字节
						int size = length / ThreadCount;

						for (int i = 0; i < ThreadCount; i++) {
							//计算线程下载的开始位置和结束位置
							int startIndex = i * size;
							int endIndex = (i + 1) * size - 1;
							//如果是最后一个线程，那么结束位置写死
							if(i == ThreadCount - 1){
								endIndex = length - 1;
							}
//							System.out.println("线程" + i + "的下载区间是：" + startIndex + "---" + endIndex);
							new DownLoadThread(startIndex, endIndex, i).start();
						}
					}
				} catch (Exception e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
		};
		t.start();
	}

	class DownLoadThread extends Thread{
		int startIndex;
		int endIndex;
		int threadId;

		public DownLoadThread(int startIndex, int endIndex, int threadId) {
			super();
			this.startIndex = startIndex;
			this.endIndex = endIndex;
			this.threadId = threadId;
		}

		@Override
		public void run() {
			//再次发送http请求，下载原文件
			try {
				File progressFile = new File(Environment.getExternalStorageDirectory(), threadId + ".txt");
				//判断进度临时文件是否存在
				if(progressFile.exists()){
					FileInputStream fis = new FileInputStream(progressFile);
					BufferedReader br = new BufferedReader(new InputStreamReader(fis));
					//从进度临时文件中读取出上一次下载的总进度，然后与原本的开始位置相加，得到新的开始位置
					int lastProgress = Integer.parseInt(br.readLine());
					startIndex += lastProgress;

					//把上次下载的进度显示至进度条
					currentProgress += lastProgress;
					pb.setProgress(currentProgress);

					//发送消息，让主线程刷新文本进度
					handler.sendEmptyMessage(1);
					fis.close();
				}
				System.out.println("线程" + threadId + "的下载区间是：" + startIndex + "---" + endIndex);
				HttpURLConnection conn;
				URL url = new URL(path);
				conn = (HttpURLConnection) url.openConnection();
				conn.setRequestMethod("GET");
				conn.setConnectTimeout(5000);
				conn.setReadTimeout(5000);
				//设置本次http请求所请求的数据的区间
				conn.setRequestProperty("Range", "bytes=" + startIndex + "-" + endIndex);

				//请求部分数据，相应码是206
				if(conn.getResponseCode() == 206){
					//流里此时只有1/3原文件的数据
					InputStream is = conn.getInputStream();
					byte[] b = new byte[1024];
					int len = 0;
					int total = 0;
					//拿到临时文件的输出流
					File file = new File(Environment.getExternalStorageDirectory(), fileName);
					RandomAccessFile raf = new RandomAccessFile(file, "rwd");
					//把文件的写入位置移动至startIndex
					raf.seek(startIndex);
					while((len = is.read(b)) != -1){
						//每次读取流里数据之后，同步把数据写入临时文件
						raf.write(b, 0, len);
						total += len;
						System.out.println("线程" + threadId + "下载了" + total);

						//每次读取流里数据之后，把本次读取的数据的长度显示至进度条
						currentProgress += len;
						pb.setProgress(currentProgress);
						//发送消息，让主线程刷新文本进度
						handler.sendEmptyMessage(1);

						//生成一个专门用来记录下载进度的临时文件
						RandomAccessFile progressRaf = new RandomAccessFile(progressFile, "rwd");
						//每次读取流里数据之后，同步把当前线程下载的总进度写入进度临时文件中
						progressRaf.write((total + "").getBytes());
						progressRaf.close();
					}
					System.out.println("线程" + threadId + "下载完毕-------------------小志参上！");
					raf.close();

					finishedThread++;
					synchronized (path) {
						if(finishedThread == ThreadCount){
							for (int i = 0; i < ThreadCount; i++) {
								File f = new File(Environment.getExternalStorageDirectory(), i + ".txt");
								f.delete();
							}
							finishedThread = 0;
						}
					}

				}
			} catch (Exception e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}

}
```
# **6. HttpUtils的使用**

>HttpUtils本身就支持多线程断点续传，使用起来非常的方便

* 创建HttpUtils对象

  	HttpUtils http = new HttpUtils();
* 下载文件
  ​
```
http.download(url, //下载请求的网址
				target, //下载的数据保存路径和文件名
				true, //是否开启断点续传
				true, //如果服务器响应头中包含了文件名，那么下载完毕后自动重命名
				new RequestCallBack<File>() {//侦听下载状态

			//下载成功此方法调用
			@Override
			public void onSuccess(ResponseInfo<File> arg0) {
				tv.setText("下载成功" + arg0.result.getPath());
			}

			//下载失败此方法调用，比如文件已经下载、没有网络权限、文件访问不到，方法传入一个字符串参数告知失败原因
			@Override
			public void onFailure(HttpException arg0, String arg1) {
				tv.setText("下载失败" + arg1);
			}

			//在下载过程中不断的调用，用于刷新进度条
			@Override
			public void onLoading(long total, long current, boolean isUploading) {
				super.onLoading(total, current, isUploading);
				//设置进度条总长度
				pb.setMax((int) total);
				//设置进度条当前进度
				pb.setProgress((int) current);
				tv_progress.setText(current * 100 / total + "%");
			}
		});
```
## **6.1 案例5：开源框架xUtils的使用**

```java
public class MainActivity extends Activity {

	private TextView tv_failure;
	private TextView tv_progress;
	private ProgressBar pb;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		tv_failure = (TextView) findViewById(R.id.tv_failure);
		tv_progress = (TextView) findViewById(R.id.tv_progress);
		pb = (ProgressBar) findViewById(R.id.pb);
	}

	public void click(View v){
		HttpUtils utils = new HttpUtils();
		String fileName = "QQPlayer.exe";
		//确定下载地址
		String path = "http://192.168.13.13:8080/" + fileName;
		utils.download(path, //下载地址
				"sdcard/QQPlayer.exe", //文件保存路径
				true,//是否支持断点续传
				true, new RequestCallBack<File>() {

					//下载成功后调用
					@Override
					public void onSuccess(ResponseInfo<File> arg0) {
						Toast.makeText(MainActivity.this, arg0.result.getPath(), 0).show();

					}

					//下载失败调用
					@Override
					public void onFailure(HttpException arg0, String arg1) {
						// TODO Auto-generated method stub
						tv_failure.setText(arg1);
					}

					@Override
					public void onLoading(long total, long current,
							boolean isUploading) {
						// TODO Auto-generated method stub
						super.onLoading(total, current, isUploading);
						pb.setMax((int)total);
						pb.setProgress((int)current);
						tv_progress.setText(current * 100 / total + "%");
					}
				});
	}
}
```
