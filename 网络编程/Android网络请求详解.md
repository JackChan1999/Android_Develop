我们知道大多数的 Android 应用程序都是通过和服务器进行交互来获取数据的。如果使用 HTTP 协议来发送和接收网络数据，就免不了使用 HttpURLConnection 和 HttpClient，而 Android 中主要提供了上述两种方式来进行 HTTP 操作。并且这两种方式都支持 HTTPS 协议、以流的形式进行上传和下载、配置超时时间、IPv6、以及连接池等功能。

但是 Googl e发布 6.0 版本的时候声明原生剔除 HttpClient，但是笔者认为 HttpClient 会提供相应的 jar 包做支持，毕竟 Google 对向下兼容这方面一直都做的很好，相信在选择网络功能的时候我们会选自己喜欢的方法。

# HttpURLConnection

接着我们来看一下如何使用 HttpURLConnection 来处理简单的网络请求。

```java
// Get方式请求  
public static void requestByGet() throws Exception {  
    String path = "10.128.7.34:3000/name=helloworld&password=android";  
    // 新建一个URL对象  
    URL url = new URL(path);  
    // 打开一个HttpURLConnection连接  
    HttpURLConnection urlConn = (HttpURLConnection) url.openConnection();  
    // 设置连接超时时间  
    urlConn.setConnectTimeout(6 * 1000);  
    // 开始连接  
    urlConn.connect();  
    // 判断请求是否成功  
    if (urlConn.getResponseCode() == HTTP_200) {  
        // 获取返回的数据  
        byte[] data = readStream(urlConn.getInputStream());  
        Log.i(TAG_GET, new String(data, "UTF-8"));  
    } else {  
        Log.i(TAG_GET, "Get方式请求失败");  
    }  
    // 关闭连接  
    urlConn.disconnect();  
}  
```
```java
// Post方式请求  
public static void requestByPost() throws Throwable {  
    String path = "http://10.128.7.34:3000/login";  
    // 请求的参数转换为byte数组  
    String params = "name" + URLEncoder.encode("helloworld", "UTF-8")  
            + "&password=" + URLEncoder.encode("android", "UTF-8");  
    byte[] postData = params.getBytes();  
    // 新建一个URL对象  
    URL url = new URL(path);  
    // 打开一个HttpURLConnection连接  
    HttpURLConnection urlConn = (HttpURLConnection) url.openConnection();  
    // 设置连接超时时间  
    urlConn.setConnectTimeout(5 * 1000);  
    // Post请求必须设置允许输出  
    urlConn.setDoOutput(true);  
    // Post请求不能使用缓存  
    urlConn.setUseCaches(false);  
    // 设置为Post请求  
    urlConn.setRequestMethod("POST");  
    urlConn.setInstanceFollowRedirects(true);  
    // 配置请求Content-Type  
    urlConn.setRequestProperty("Content-Type","application/x-www-form-urlencode");  
    // 开始连接  
    urlConn.connect();  
    // 发送请求参数  
    DataOutputStream dos = new DataOutputStream(urlConn.getOutputStream());  
    dos.write(postData);  
    dos.flush();  
    dos.close();  
    // 判断请求是否成功  
    if (urlConn.getResponseCode() == HTTP_200) {  
        // 获取返回的数据  
        byte[] data = readStream(urlConn.getInputStream());  
        Log.i(TAG_POST, new String(data, "UTF-8"));  
    } else {  
        Log.i(TAG_POST, "Post方式请求失败");  
    }  
}  
```
关于HttpURLConnection的相关开源工程

由于 Google 已经表明了推荐广大开发者使用 HttpURLConnection，那么想必是有一定原因的。

# xUtils3

这里就推荐大伙都很熟悉的开源项目 xUtils 的“弟弟“，xUtils3。xUtils 包含了 view 注入，图片处理，数据库操作以及网络请求封装，xUtils 使用的网络请求封装是基于 HttpClient 的。xUtils3 使用的网络请求是基于 HttpURLConnection，我们着重说明一下xUtils3。

```java
RequestParams params = new RequestParams("http://10.128.7.34:3000/upload");  
        // 加到url里的参数, http://xxxx/s?wd=xUtils
        params.addQueryStringParameter("wd", "xUtils");
        // 添加到请求 body 体的参数, 只有 POST, PUT, PATCH, DELETE 请求支持.
        // params.addBodyParameter("wd", "xUtils");

        // 使用 multipart 表单上传文件
        params.setMultipart(true);
        params.addBodyParameter(
                "file",
                new File("/sdcard/test.jpg"),
                null); // 如果文件没有扩展名, 最好设置contentType 参数.
        params.addBodyParameter(
                "file2",
                new FileInputStream(new File("/sdcard/test2.jpg")),
                "image/jpeg",
                // 测试中文文件名
                "你+& \" 好.jpg"); // InputStream 参数获取不到文件名, 最好设置, 除非服务端不关心这个参数.
        x.http().post(params, new Callback.CommonCallback<String>() {
            @Override
            public void onSuccess(String result) {
                Toast.makeText(x.app(), result, Toast.LENGTH_LONG).show();
            }

            @Override
            public void onError(Throwable ex, boolean isOnCallback) {
                Toast.makeText(x.app(), ex.getMessage(), Toast.LENGTH_LONG).show();
            }

            @Override
            public void onCancelled(CancelledException cex) {
                Toast.makeText(x.app(), "cancelled", Toast.LENGTH_LONG).show();
            }

            @Override
            public void onFinished() {

            }
        });
```
具有 cache 功能
```java
RequestParams params = new RequestParams("http://10.128.7.34:3000/upload");  
        params.wd = "xUtils";
        // 默认缓存存活时间, 单位:毫秒.(如果服务没有返回有效的max-age 或 Expires)
        params.setCacheMaxAge(1000 * 60);
        Callback.Cancelable cancelable
                // 使用 CacheCallback, xUtils 将为该请求缓存数据.
                = x.http().get(params, new Callback.CacheCallback<String>() {

            private boolean hasError = false;
            private String result = null;

            @Override
            public boolean onCache(String result) {
                // 得到缓存数据, 缓存过期后不会进入这个方法.
                // 如果服务端没有返回过期时间, 参考params.setCacheMaxAge(maxAge)方法.
                //
                // * 客户端会根据服务端返回的 header 中 max-age 或 expires 来确定本地缓存是否给 onCache 方法.
                //   如果服务端没有返回 max-age 或 expires, 那么缓存将一直保存, 除非这里自己定义了返回false的
                //   逻辑, 那么xUtils将请求新数据, 来覆盖它.
                //
                // * 如果信任该缓存返回 true, 将不再请求网络;
                //   返回 false 继续请求网络, 但会在请求头中加上ETag, Last-Modified等信息,
                //   如果服务端返回 304, 则表示数据没有更新, 不继续加载数据.
                //
                this.result = result;
                return false; // true: 信任缓存数据, 不在发起网络请求; false 不信任缓存数据.
            }

            @Override
            public void onSuccess(String result) {
                // 注意: 如果服务返回 304 或 onCache 选择了信任缓存, 这里将不会被调用,
                // 但是 onFinished 总会被调用.
                this.result = result;
            }

            @Override
            public void onError(Throwable ex, boolean isOnCallback) {
                hasError = true;
                Toast.makeText(x.app(), ex.getMessage(), Toast.LENGTH_LONG).show();
                if (ex instanceof HttpException) { // 网络错误
                    HttpException httpEx = (HttpException) ex;
                    int responseCode = httpEx.getCode();
                    String responseMsg = httpEx.getMessage();
                    String errorResult = httpEx.getResult();
                    // ...
                } else { // 其他错误
                    // ...
                }
            }

            @Override
            public void onCancelled(CancelledException cex) {
                Toast.makeText(x.app(), "cancelled", Toast.LENGTH_LONG).show();
            }

            @Override
            public void onFinished() {
                if (!hasError && result != null) {
                    // 成功获取数据
                    Toast.makeText(x.app(), result, Toast.LENGTH_LONG).show();
                }
            }
        });
```
post请求直接更改 post 方式，以及需要提交的参数即可，篇幅问题这里就不在一一列举了。通过以上代码可以看到，我们在使用 xUtils 来请求网络的时候会非常的方便，只用在回调函数里面做对应的代码逻辑处理即可，不用关心线程问题，极大的解放了我们的生产力。
Android Stuido Gradle使用方法如下：
```Gradle
compile 'org.xutils:xutils:3.1.+'
```
# Volley

在 Android 2.3 及以上版本，使用的是 HttpURLConnection，而在Android 2.2 及以下版本，使用的是 HttpClient。鉴于现在的手机行业发展速度，我们已经不考虑 Android2.2 了。

简单提供一些 Volley 的实例：
```java
//简单的 GET 请求
mQueue = Volley.newRequestQueue(getApplicationContext());  
mQueue.add(new JsonObjectRequest(Method.GET, url, null,  
            new Listener() {  
                @Override  
                public void onResponse(JSONObject response) {  
                    Log.d(TAG, "response : " + response.toString());  
                }  
            }, null));  
mQueue.start();  
//对 ImageView 的操作
ImageListener listener = ImageLoader.getImageListener(imageView, android.R.drawable.ic_launcher, android.R.drawable.ic_launcher);  
mImageLoader.get(url, listener);  

//对 ImageView 网络加载的处理
mImageView.setImageUrl(url, imageLoader);
当然我们也可以定制自己的 request

mRequestQueue.add( new GsonRequest(url, ListResponse.class, null,  
    new Listener() {  
        public void onResponse(ListResponse response) {  
            appendItemsToList(response.item);  
            notifyDataSetChanged();  
        }  
    }  
}
```  
# HttpClient

同样，我们来看一下 HttpClient 的简单请求。
```java
// HttpGet 方式请求  
public static void requestByHttpGet() throws Exception {  
    String path = "http://10.128.7.34:3000/name=helloworld&password=android";  
    // 新建 HttpGet 对象  
    HttpGet httpGet = new HttpGet(path);  
    // 获取 HttpClient 对象  
    HttpClient httpClient = new DefaultHttpClient();  
    // 获取 HttpResponse 实例  
    HttpResponse httpResp = httpClient.execute(httpGet);  
    // 判断是够请求成功  
    if (httpResp.getStatusLine().getStatusCode() == HTTP_200) {  
        // 获取返回的数据  
        String result = EntityUtils.toString(httpResp.getEntity(), "UTF-8");  
        Log.i(TAG_HTTPGET, "HttpGet 方式请求成功，返回数据如下：");  
        Log.i(TAG_HTTPGET, result);  
    } else {  
        Log.i(TAG_HTTPGET, "HttpGet 方式请求失败");  
    }  
}  
// HttpPost 方式请求  
public static void requestByHttpPost() throws Exception {  
    String path = "https://reg.163.com/login";  
    // 新建 HttpPost 对象  
    HttpPost httpPost = new HttpPost(path);  
    // Post 参数  
    List<NameValuePair> params = new ArrayList<NameValuePair>();  
    params.add(new BasicNameValuePair("name", "helloworld"));  
    params.add(new BasicNameValuePair("password", "android"));  
    // 设置字符集  
    HttpEntity entity = new UrlEncodedFormEntity(params, HTTP.UTF_8);  
    // 设置参数实体  
    httpPost.setEntity(entity);  
    // 获取 HttpClient 对象  
    HttpClient httpClient = new DefaultHttpClient();  
    // 获取 HttpResponse 实例  
    HttpResponse httpResp = httpClient.execute(httpPost);  
    // 判断是够请求成功  
    if (httpResp.getStatusLine().getStatusCode() == HTTP_200) {  
        // 获取返回的数据  
        String result = EntityUtils.toString(httpResp.getEntity(), "UTF-8");  
        Log.i(TAG_HTTPGET, "HttpPost 方式请求成功，返回数据如下：");  
        Log.i(TAG_HTTPGET, result);  
    } else {  
        Log.i(TAG_HTTPGET, "HttpPost 方式请求失败");  
    }  
}  
```
关于 HttpClinet 的相关开源工程

HttpClient 也拥有这大量优秀的开源工程，afinal、xUtils 以及AsyncHttpClient，也可以为广大开发者提供已经造好的轮子。由于xUtils 是基于 afinal 重写的，现在 xUtils3 也有替代 xUtils 的趋势，所以我们在这就简单介绍一下 AsyncHttpClient。

# AsyncHttpClient

见名知意，AsyncHttpClient 对处理异步 Http 请求相当擅长，并通过匿名内部类处理回调结果，Http 异步请求均位于非 UI 线程，不会阻塞 UI 操作,通过线程池处理并发请求处理文件上传、下载、响应结果自动打包 JSON 格式。使用起来会很方便。
```java
//GET请求
AsyncHttpClient client = new AsyncHttpClient();  
//当然这里也可以换成 new JsonHttpResponseHandler(),我们就能直接获得 JSON 数据了。
client.get("http://www.google.com", new AsyncHttpResponseHandler() {

    @Override
    public void onStart() {
        // called before request is started
    }

    @Override
    public void onSuccess(int statusCode, Header[] headers, byte[] response) {
        // called when response HTTP status is "200 OK"
    }

    @Override
    public void onFailure(int statusCode, Header[] headers, byte[] errorResponse, Throwable e) {
        // called when response HTTP status is "4XX" (eg. 401, 403, 404)
    }

    @Override
    public void onRetry(int retryNo) {
        // called when request is retried
    }
});
//POST 请求
AsyncHttpClient client = new AsyncHttpClient();  
RequestParams params = new RequestParams();  
params.put("key", "value");  
params.put("more", "data");  
//同上，这里一样可以改成处理 JSON 数据的方法
client.get("http://www.google.com", params, new  
    TextHttpResponseHandler() {
        @Override
        public void onSuccess(int statusCode, Header[] headers, String response） {
            System.out.println(response);
        }

        @Override
        public void onFailure(int statusCode, Header[] headers, String responseBody, Throwable error） {
            Log.d("ERROR", error);
        }    
    }
);
```
经过上面的代码发现，AsyncHttpClient 使用起来也是异常简洁，主要靠回调方法来处理成功或失败之后的逻辑。仔细想想，xUtils 的处理方式和这个处理方式很类似，看来好用设计还是很受人青睐的。

# OkHttp

如果两种网络请求都想使用怎么办？那么 OkHttp 是一个最佳解决方案了。

OkHttp 在网络请求方面的口碑很好，就连 Google 自己也有使用OkHttp。虽然 Google6.0 中剔除了 HttpClient 的 Api，但是由于OkHttp 的影响力以及其强大的功能，使用 OkHttp 无需重写您程序中的网络代码。同时最重要的一点 OkHttp 实现了几乎和java.net.HttpURLConnection 一样的 API。如果您用了 Apache HttpClient，则 OkHttp 也提供了一个对应的 okhttp-apache 模块。足以说明 OkHttp 的强大，下面是一些例子。

- 一般的 get 请求
- 一般的 post 请求
- 基于 Http 的文件上传
- 文件下载
- 加载图片
- 支持请求回调，直接返回对象、对象集合
- 支持 session 的保持

简单代码实例
```java
//GET 请求
OkHttpClient client = new OkHttpClient();

String run(String url) throws IOException {  
  Request request = new Request.Builder()
      .url(url)
      .build();

  Response response = client.newCall(request).execute();
  return response.body().string();
}
//POST 请求
public static final MediaType JSON  
    = MediaType.parse("application/json; charset=utf-8");

OkHttpClient client = new OkHttpClient();

String post(String url, String json) throws IOException {  
  RequestBody body = RequestBody.create(JSON, json);
  Request request = new Request.Builder()
      .url(url)
      .post(body)
      .build();
  Response response = client.newCall(request).execute();
  return response.body().string();
}
```
Android Studio Gradle 使用方式:
```Gradle
compile 'com.squareup.okhttp:okhttp:3.7.0'
```
# 总结

Android 开发应用少不了使用网络，移动互联时代，抢占终端入口变得异常重要，那么我们在开发过程中，不管使用哪一种网络请求，HttpURLConnection 或者是 HttpClient，都可以满足我们和服务器的沟通。
