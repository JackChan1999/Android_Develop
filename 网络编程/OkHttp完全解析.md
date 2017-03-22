# 1. 概述

最近在群里听到各种讨论okhttp的话题，可见okhttp的口碑相当好了。再加上Google貌似在6.0版本里面删除了HttpClient相关API，对于这个行为不做评价。为了更好的在应对网络访问，学习下okhttp还是蛮必要的，本篇博客首先介绍okhttp的简单使用，主要包含：

- 一般的get请求
- 一般的post请求
- 基于Http的文件上传
- 文件下载
- 加载图片
- 支持请求回调，直接返回对象、对象集合
- 支持session的保持

最后会对上述几个功能进行封装，完整的封装类的地址见：[https://github.com/hongyangAndroid/okhttp-utils](https://github.com/hongyangAndroid/okhttp-utils)

使用前，对于[Android](http://lib.csdn.net/base/android) Studio的用户，可以选择添加:

```gradle
compile 'com.squareup.okhttp:okhttp:2.4.0'11
```

或者Eclipse的用户，可以下载最新的jar [okhttp he latest JAR ](https://search.maven.org/remote_content?g=com.squareup.okhttp&a=okhttp&v=LATEST)，添加依赖就可以用了。

注意:okhttp内部依赖okio，别忘了同时导入okio：

gradle: `compile 'com.squareup.okio:okio:1.5.0'`

最新的jar地址：[okio the latest JAR](https://search.maven.org/remote_content?g=com.squareup.okio&a=okio&v=LATEST)

# 2. 使用教程

## 2.1 Http Get

对了网络加载库，那么最常见的肯定就是http get请求了，比如获取一个网页的内容。

```java
//创建okHttpClient对象
OkHttpClient mOkHttpClient = new OkHttpClient();
//创建一个Request
final Request request = new Request.Builder()
                .url("https://github.com/hongyangAndroid")
                .build();
//new call
Call call = mOkHttpClient.newCall(request);
//请求加入调度
call.enqueue(new Callback()
        {
            @Override
            public void onFailure(Request request, IOException e)
            {
            }

            @Override
            public void onResponse(final Response response) throws IOException
            {
                    //String htmlStr =  response.body().string();
            }
        });             
```

1. 以上就是发送一个get请求的步骤，首先构造一个Request对象，参数最起码有个url，当然你可以通过Request.Builder设置更多的参数比如：`header`、`method`等。
2. 然后通过request的对象去构造得到一个Call对象，类似于将你的请求封装成了任务，既然是任务，就会有`execute()`和`cancel()`等方法。
3. 最后，我们希望以异步的方式去执行请求，所以我们调用的是call.enqueue，将call加入调度队列，然后等待任务执行完成，我们在Callback中即可得到结果。

看到这，你会发现，整体的写法还是比较长的，所以封装肯定是要做的，不然每个请求这么写，得累死。

ok，需要注意几点：

- onResponse回调的参数是response，一般情况下，比如我们希望获得返回的字符串，可以通过`response.body().string()`获取；如果希望获得返回的二进制字节数组，则调用`response.body().bytes()`；如果你想拿到返回的inputStream，则调用`response.body().byteStream()`

看到这，你可能会奇怪，竟然还能拿到返回的inputStream，看到这个最起码能意识到一点，这里支持大文件下载，有inputStream我们就可以通过IO的方式写文件。不过也说明一个问题，这个`onResponse`执行的线程并不是UI线程。的确是的，如果你希望操作控件，还是需要使用handler等，例如：

```java
@Override
public void onResponse(final Response response) throws IOException
{
      final String res = response.body().string();
      runOnUiThread(new Runnable()
      {
          @Override
          public void run()
          {
            mTv.setText(res);
          }

      });
}
```

- 我们这里是异步的方式去执行，当然也支持阻塞的方式，上面我们也说了Call有一个`execute()`方法，你也可以直接调用`call.execute()`通过返回一个`Response`。

## 2.2 Http Post 携带参数

看来上面的简单的get请求，基本上整个的用法也就掌握了，比如post携带参数，也仅仅是Request的构造的不同。

```java
Request request = buildMultipartFormRequest(url, new File[]{file}, new String[]{fileKey}, null);
FormEncodingBuilder builder = new FormEncodingBuilder();   
builder.add("username","张鸿洋");

Request request = new Request.Builder()
                .url(url)
                .post(builder.build())
                .build();
 mOkHttpClient.newCall(request).enqueue(new Callback(){});1234567891012345678910
```

大家都清楚，post的时候，参数是包含在请求体中的；所以我们通过FormEncodingBuilder。添加多个String键值对，然后去构造RequestBody，最后完成我们Request的构造。

后面的就和上面一样了。

## 2.3 基于Http的文件上传

接下来我们在介绍一个可以构造RequestBody的Builder，叫做`MultipartBuilder`。当我们需要做类似于表单上传的时候，就可以使用它来构造我们的requestBody。

```java
File file = new File(Environment.getExternalStorageDirectory(), "balabala.mp4");

RequestBody fileBody = RequestBody.create(MediaType.parse("application/octet-stream"), file);

RequestBody requestBody = new MultipartBuilder()
     .type(MultipartBuilder.FORM)
     .addPart(Headers.of("Content-Disposition", "form-data; name=\"username\""),
          RequestBody.create(null, "张鸿洋"))
     .addPart(Headers.of(
         "Content-Disposition",
         "form-data; name=\"mFile\";
         filename=\"wjd.mp4\""), fileBody)
     .build();

Request request = new Request.Builder()
    .url("http://192.168.1.103:8080/okHttpServer/fileUpload")
    .post(requestBody)
    .build();

Call call = mOkHttpClient.newCall(request);
call.enqueue(new Callback()
{
    //...
});
```

上述代码向服务器传递了一个键值对`username:张鸿洋`和一个文件。我们通过MultipartBuilder的addPart方法可以添加键值对或者文件。

其实类似于我们拼接模拟浏览器行为的方式，如果你对这块不了解，可以参考：[从原理角度解析Android （Java） http 文件上传](http://blog.csdn.net/lmj623565791/article/details/23781773)

ok，对于我们最开始的目录还剩下图片下载，文件下载；这两个一个是通过回调的Response拿到byte[]然后decode成图片；文件下载，就是拿到inputStream做写文件操作，我们这里就不赘述了。

关于用法，也可以参考[泡网OkHttp使用教程](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0106/2275.html)

接下来我们主要看如何封装上述的代码。

# 3. 封装

由于按照上述的代码，写多个请求肯定包含大量的重复代码，所以我希望封装后的代码调用是这样的：

## 3.1 使用

1. 一般的get请求

```java
OkHttpClientManager.getAsyn("https://www.baidu.com", new OkHttpClientManager.ResultCallback<String>()
       {
           @Override
           public void onError(Request request, Exception e)
           {
               e.printStackTrace();
           }

           @Override
           public void onResponse(String u)
           {
               mTv.setText(u);//注意这里是UI线程
           }
       });12345678910111213141234567891011121314
```

对于一般的请求，我们希望给个url，然后CallBack里面直接操作控件。

2. 文件上传且携带参数

我们希望提供一个方法，传入url,params,file,callback即可。

```java
OkHttpClientManager.postAsyn("http://192.168.1.103:8080/okHttpServer/fileUpload",//
  new OkHttpClientManager.ResultCallback<String>()
  {
      @Override
      public void onError(Request request, IOException e)
      {
          e.printStackTrace();
      }

      @Override
      public void onResponse(String result)
      {

      }
  },//
  file,//
  "mFile",//
  new OkHttpClientManager.Param[]{
          new OkHttpClientManager.Param("username", "zhy"),
          new OkHttpClientManager.Param("password", "123")}
      );
```
键值对没什么说的，参数3为file，参数4为file对应的name，这个name不是文件的名字； 对应于http中的`<input type="file" name="mFile" >`对应的是name后面的值，即mFile.

3. 文件下载
对于文件下载，提供url，目标dir，callback即可。

```java
OkHttpClientManager.downloadAsyn(
    "http://192.168.1.103:8080/okHttpServer/files/messenger_01.png",    
    Environment.getExternalStorageDirectory().getAbsolutePath(),
new OkHttpClientManager.ResultCallback<String>()
    {
        @Override
        public void onError(Request request, IOException e)
        {

        }

        @Override
        public void onResponse(String response)
        {
            //文件下载成功，这里回调的reponse为文件的absolutePath
        }
});
```

4. 展示图片
展示图片，我们希望提供一个url和一个imageview，如果下载成功，直接帮我们设置上即可。

```
OkHttpClientManager.displayImage(mImageView,"http://images.csdn.net/20150817/1.jpg");1212
```
内部会自动根据imageview的大小自动对图片进行合适的压缩。虽然，这里可能不适合一次性加载大量图片的场景，但是对于app中偶尔有几个图片的加载，还是可用的。

# 4. 整合Gson

很多人提出项目中使用时，服务端返回的是Json字符串，希望客户端回调可以直接拿到对象，于是整合进入了Gson，完善该功能。

## 4.1 直接回调对象

例如现在有个User实体类：

```java
package com.zhy.utils.http.okhttp;

public class User {

    public String username ;
    public String password  ;

    public User() {
        // TODO Auto-generated constructor stub
    }

    public User(String username, String password) {
        this.username = username;
        this.password = password;
    }

    @Override
    public String toString()
    {
        return "User{" +
                "username='" + username + '\'' +
                ", password='" + password + '\'' +
                '}';
    }
}
```

服务端返回：

```
{"username":"zhy","password":"123"}11
```

客户端可以如下方式调用：

```java
OkHttpClientManager.getAsyn("http://192.168.56.1:8080/okHttpServer/user!getUser",
new OkHttpClientManager.ResultCallback<User>()
{
    @Override
    public void onError(Request request, Exception e)
    {
        e.printStackTrace();
    }

    @Override
    public void onResponse(User user)
    {
        mTv.setText(u.toString());//UI线程
    }
});
```

我们传入泛型User，在onResponse里面直接回调User对象。
这里特别要注意的事，如果在`json字符串->实体对象`过程中发生错误，程序不会崩溃，`onError`方法会被回调。

注意：这里做了少许的更新，接口命名从`StringCallback`修改为`ResultCallback`。接口中的`onFailure`方法修改为`onError`。

## 4.2 回调对象集合

依然是上述的User类，服务端返回

```
[{"username":"zhy","password":"123"},{"username":"lmj","password":"12345"}]
```

则客户端可以如下调用：

```java
OkHttpClientManager.getAsyn("http://192.168.56.1:8080/okHttpServer/user!getUsers",
new OkHttpClientManager.ResultCallback<List<User>>()
{
    @Override
    public void onError(Request request, Exception e)
    {
        e.printStackTrace();
    }
    @Override
    public void onResponse(List<User> us)
    {
        Log.e("TAG", us.size() + "");
        mTv.setText(us.get(1).toString());
    }
});
```

唯一的区别，就是泛型变为`List<User>` ，ok ， 如果发现bug或者有任何意见欢迎留言。

# 5. 源码

ok，基本介绍完了，对于封装的代码其实也很简单，我就直接贴出来了，因为也没什么好介绍的，如果你看完上面的用法，肯定可以看懂：

```java
package com.zhy.utils.http.okhttp;

import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.os.Handler;
import android.os.Looper;
import android.widget.ImageView;

import com.google.gson.Gson;
import com.google.gson.internal.$Gson$Types;
import com.squareup.okhttp.Call;
import com.squareup.okhttp.Callback;
import com.squareup.okhttp.FormEncodingBuilder;
import com.squareup.okhttp.Headers;
import com.squareup.okhttp.MediaType;
import com.squareup.okhttp.MultipartBuilder;
import com.squareup.okhttp.OkHttpClient;
import com.squareup.okhttp.Request;
import com.squareup.okhttp.RequestBody;
import com.squareup.okhttp.Response;

import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;
import java.net.CookieManager;
import java.net.CookiePolicy;
import java.net.FileNameMap;
import java.net.URLConnection;
import java.util.HashMap;
import java.util.Map;
import java.util.Set;

/**
 * Created by zhy on 15/8/17.
 */
public class OkHttpClientManager
{
    private static OkHttpClientManager mInstance;
    private OkHttpClient mOkHttpClient;
    private Handler mDelivery;
    private Gson mGson;


    private static final String TAG = "OkHttpClientManager";

    private OkHttpClientManager()
    {
        mOkHttpClient = new OkHttpClient();
        //cookie enabled
        mOkHttpClient.setCookieHandler(new CookieManager(null, CookiePolicy.ACCEPT_ORIGINAL_SERVER));
        mDelivery = new Handler(Looper.getMainLooper());
        mGson = new Gson();
    }

    public static OkHttpClientManager getInstance()
    {
        if (mInstance == null)
        {
            synchronized (OkHttpClientManager.class)
            {
                if (mInstance == null)
                {
                    mInstance = new OkHttpClientManager();
                }
            }
        }
        return mInstance;
    }

    /**
     * 同步的Get请求
     *
     * @param url
     * @return Response
     */
    private Response _getAsyn(String url) throws IOException
    {
        final Request request = new Request.Builder()
                .url(url)
                .build();
        Call call = mOkHttpClient.newCall(request);
        Response execute = call.execute();
        return execute;
    }

    /**
     * 同步的Get请求
     *
     * @param url
     * @return 字符串
     */
    private String _getAsString(String url) throws IOException
    {
        Response execute = _getAsyn(url);
        return execute.body().string();
    }


    /**
     * 异步的get请求
     *
     * @param url
     * @param callback
     */
    private void _getAsyn(String url, final ResultCallback callback)
    {
        final Request request = new Request.Builder()
                .url(url)
                .build();
        deliveryResult(callback, request);
    }


    /**
     * 同步的Post请求
     *
     * @param url
     * @param params post的参数
     * @return
     */
    private Response _post(String url, Param... params) throws IOException
    {
        Request request = buildPostRequest(url, params);
        Response response = mOkHttpClient.newCall(request).execute();
        return response;
    }


    /**
     * 同步的Post请求
     *
     * @param url
     * @param params post的参数
     * @return 字符串
     */
    private String _postAsString(String url, Param... params) throws IOException
    {
        Response response = _post(url, params);
        return response.body().string();
    }

    /**
     * 异步的post请求
     *
     * @param url
     * @param callback
     * @param params
     */
    private void _postAsyn(String url, final ResultCallback callback, Param... params)
    {
        Request request = buildPostRequest(url, params);
        deliveryResult(callback, request);
    }

    /**
     * 异步的post请求
     *
     * @param url
     * @param callback
     * @param params
     */
    private void _postAsyn(String url, final ResultCallback callback, Map<String, String> params)
    {
        Param[] paramsArr = map2Params(params);
        Request request = buildPostRequest(url, paramsArr);
        deliveryResult(callback, request);
    }

    /**
     * 同步基于post的文件上传
     *
     * @param params
     * @return
     */
    private Response _post(String url, File[] files, String[] fileKeys, Param... params) throws IOException
    {
        Request request = buildMultipartFormRequest(url, files, fileKeys, params);
        return mOkHttpClient.newCall(request).execute();
    }

    private Response _post(String url, File file, String fileKey) throws IOException
    {
        Request request = buildMultipartFormRequest(url, new File[]{file}, new String[]{fileKey}, null);
        return mOkHttpClient.newCall(request).execute();
    }

    private Response _post(String url, File file, String fileKey, Param... params) throws IOException
    {
        Request request = buildMultipartFormRequest(url, new File[]{file}, new String[]{fileKey}, params);
        return mOkHttpClient.newCall(request).execute();
    }

    /**
     * 异步基于post的文件上传
     *
     * @param url
     * @param callback
     * @param files
     * @param fileKeys
     * @throws IOException
     */
    private void _postAsyn(String url, ResultCallback callback, File[] files, String[] fileKeys, Param... params) throws IOException
    {
        Request request = buildMultipartFormRequest(url, files, fileKeys, params);
        deliveryResult(callback, request);
    }

    /**
     * 异步基于post的文件上传，单文件不带参数上传
     *
     * @param url
     * @param callback
     * @param file
     * @param fileKey
     * @throws IOException
     */
    private void _postAsyn(String url, ResultCallback callback, File file, String fileKey) throws IOException
    {
        Request request = buildMultipartFormRequest(url, new File[]{file}, new String[]{fileKey}, null);
        deliveryResult(callback, request);
    }

    /**
     * 异步基于post的文件上传，单文件且携带其他form参数上传
     *
     * @param url
     * @param callback
     * @param file
     * @param fileKey
     * @param params
     * @throws IOException
     */
    private void _postAsyn(String url, ResultCallback callback, File file, String fileKey, Param... params) throws IOException
    {
        Request request = buildMultipartFormRequest(url, new File[]{file}, new String[]{fileKey}, params);
        deliveryResult(callback, request);
    }

    /**
     * 异步下载文件
     *
     * @param url
     * @param destFileDir 本地文件存储的文件夹
     * @param callback
     */
    private void _downloadAsyn(final String url, final String destFileDir, final ResultCallback callback)
    {
        final Request request = new Request.Builder()
                .url(url)
                .build();
        final Call call = mOkHttpClient.newCall(request);
        call.enqueue(new Callback()
        {
            @Override
            public void onFailure(final Request request, final IOException e)
            {
                sendFailedStringCallback(request, e, callback);
            }

            @Override
            public void onResponse(Response response)
            {
                InputStream is = null;
                byte[] buf = new byte[2048];
                int len = 0;
                FileOutputStream fos = null;
                try
                {
                    is = response.body().byteStream();
                    File file = new File(destFileDir, getFileName(url));
                    fos = new FileOutputStream(file);
                    while ((len = is.read(buf)) != -1)
                    {
                        fos.write(buf, 0, len);
                    }
                    fos.flush();
                    //如果下载文件成功，第一个参数为文件的绝对路径
                    sendSuccessResultCallback(file.getAbsolutePath(), callback);
                } catch (IOException e)
                {
                    sendFailedStringCallback(response.request(), e, callback);
                } finally
                {
                    try
                    {
                        if (is != null) is.close();
                    } catch (IOException e)
                    {
                    }
                    try
                    {
                        if (fos != null) fos.close();
                    } catch (IOException e)
                    {
                    }
                }

            }
        });
    }

    private String getFileName(String path)
    {
        int separatorIndex = path.lastIndexOf("/");
        return (separatorIndex < 0) ? path : path.substring(separatorIndex + 1, path.length());
    }

    /**
     * 加载图片
     *
     * @param view
     * @param url
     * @throws IOException
     */
    private void _displayImage(final ImageView view, final String url, final int errorResId)
    {
        final Request request = new Request.Builder()
                .url(url)
                .build();
        Call call = mOkHttpClient.newCall(request);
        call.enqueue(new Callback()
        {
            @Override
            public void onFailure(Request request, IOException e)
            {
                setErrorResId(view, errorResId);
            }

            @Override
            public void onResponse(Response response)
            {
                InputStream is = null;
                try
                {
                    is = response.body().byteStream();
                    ImageUtils.ImageSize actualImageSize = ImageUtils.getImageSize(is);
                    ImageUtils.ImageSize imageViewSize = ImageUtils.getImageViewSize(view);
                    int inSampleSize = ImageUtils.calculateInSampleSize(actualImageSize, imageViewSize);
                    try
                    {
                        is.reset();
                    } catch (IOException e)
                    {
                        response = _getAsyn(url);
                        is = response.body().byteStream();
                    }

                    BitmapFactory.Options ops = new BitmapFactory.Options();
                    ops.inJustDecodeBounds = false;
                    ops.inSampleSize = inSampleSize;
                    final Bitmap bm = BitmapFactory.decodeStream(is, null, ops);
                    mDelivery.post(new Runnable()
                    {
                        @Override
                        public void run()
                        {
                            view.setImageBitmap(bm);
                        }
                    });
                } catch (Exception e)
                {
                    setErrorResId(view, errorResId);

                } finally
                {
                    if (is != null) try
                    {
                        is.close();
                    } catch (IOException e)
                    {
                        e.printStackTrace();
                    }
                }
            }
        });


    }

    private void setErrorResId(final ImageView view, final int errorResId)
    {
        mDelivery.post(new Runnable()
        {
            @Override
            public void run()
            {
                view.setImageResource(errorResId);
            }
        });
    }


    //*************对外公布的方法************


    public static Response getAsyn(String url) throws IOException
    {
        return getInstance()._getAsyn(url);
    }


    public static String getAsString(String url) throws IOException
    {
        return getInstance()._getAsString(url);
    }

    public static void getAsyn(String url, ResultCallback callback)
    {
        getInstance()._getAsyn(url, callback);
    }

    public static Response post(String url, Param... params) throws IOException
    {
        return getInstance()._post(url, params);
    }

    public static String postAsString(String url, Param... params) throws IOException
    {
        return getInstance()._postAsString(url, params);
    }

    public static void postAsyn(String url, final ResultCallback callback, Param... params)
    {
        getInstance()._postAsyn(url, callback, params);
    }


    public static void postAsyn(String url, final ResultCallback callback, Map<String, String> params)
    {
        getInstance()._postAsyn(url, callback, params);
    }


    public static Response post(String url, File[] files, String[] fileKeys, Param... params) throws IOException
    {
        return getInstance()._post(url, files, fileKeys, params);
    }

    public static Response post(String url, File file, String fileKey) throws IOException
    {
        return getInstance()._post(url, file, fileKey);
    }

    public static Response post(String url, File file, String fileKey, Param... params) throws IOException
    {
        return getInstance()._post(url, file, fileKey, params);
    }

    public static void postAsyn(String url, ResultCallback callback, File[] files, String[] fileKeys, Param... params) throws IOException
    {
        getInstance()._postAsyn(url, callback, files, fileKeys, params);
    }


    public static void postAsyn(String url, ResultCallback callback, File file, String fileKey) throws IOException
    {
        getInstance()._postAsyn(url, callback, file, fileKey);
    }


    public static void postAsyn(String url, ResultCallback callback, File file, String fileKey, Param... params) throws IOException
    {
        getInstance()._postAsyn(url, callback, file, fileKey, params);
    }

    public static void displayImage(final ImageView view, String url, int errorResId) throws IOException
    {
        getInstance()._displayImage(view, url, errorResId);
    }


    public static void displayImage(final ImageView view, String url)
    {
        getInstance()._displayImage(view, url, -1);
    }

    public static void downloadAsyn(String url, String destDir, ResultCallback callback)
    {
        getInstance()._downloadAsyn(url, destDir, callback);
    }

    //****************************


    private Request buildMultipartFormRequest(String url, File[] files,
                                              String[] fileKeys, Param[] params)
    {
        params = validateParam(params);

        MultipartBuilder builder = new MultipartBuilder()
                .type(MultipartBuilder.FORM);

        for (Param param : params)
        {
            builder.addPart(Headers.of("Content-Disposition", "form-data; name=\"" + param.key + "\""),
                    RequestBody.create(null, param.value));
        }
        if (files != null)
        {
            RequestBody fileBody = null;
            for (int i = 0; i < files.length; i++)
            {
                File file = files[i];
                String fileName = file.getName();
                fileBody = RequestBody.create(MediaType.parse(guessMimeType(fileName)), file);
                //TODO 根据文件名设置contentType
                builder.addPart(Headers.of("Content-Disposition",
                                "form-data; name=\"" + fileKeys[i] + "\"; filename=\"" + fileName + "\""),
                        fileBody);
            }
        }

        RequestBody requestBody = builder.build();
        return new Request.Builder()
                .url(url)
                .post(requestBody)
                .build();
    }

    private String guessMimeType(String path)
    {
        FileNameMap fileNameMap = URLConnection.getFileNameMap();
        String contentTypeFor = fileNameMap.getContentTypeFor(path);
        if (contentTypeFor == null)
        {
            contentTypeFor = "application/octet-stream";
        }
        return contentTypeFor;
    }


    private Param[] validateParam(Param[] params)
    {
        if (params == null)
            return new Param[0];
        else return params;
    }

    private Param[] map2Params(Map<String, String> params)
    {
        if (params == null) return new Param[0];
        int size = params.size();
        Param[] res = new Param[size];
        Set<Map.Entry<String, String>> entries = params.entrySet();
        int i = 0;
        for (Map.Entry<String, String> entry : entries)
        {
            res[i++] = new Param(entry.getKey(), entry.getValue());
        }
        return res;
    }

    private static final String SESSION_KEY = "Set-Cookie";
    private static final String mSessionKey = "JSESSIONID";

    private Map<String, String> mSessions = new HashMap<String, String>();

    private void deliveryResult(final ResultCallback callback, Request request)
    {
        mOkHttpClient.newCall(request).enqueue(new Callback()
        {
            @Override
            public void onFailure(final Request request, final IOException e)
            {
                sendFailedStringCallback(request, e, callback);
            }

            @Override
            public void onResponse(final Response response)
            {
                try
                {
                    final String string = response.body().string();
                    if (callback.mType == String.class)
                    {
                        sendSuccessResultCallback(string, callback);
                    } else
                    {
                        Object o = mGson.fromJson(string, callback.mType);
                        sendSuccessResultCallback(o, callback);
                    }


                } catch (IOException e)
                {
                    sendFailedStringCallback(response.request(), e, callback);
                } catch (com.google.gson.JsonParseException e)//Json解析的错误
                {
                    sendFailedStringCallback(response.request(), e, callback);
                }

            }
        });
    }

    private void sendFailedStringCallback(final Request request, final Exception e, final ResultCallback callback)
    {
        mDelivery.post(new Runnable()
        {
            @Override
            public void run()
            {
                if (callback != null)
                    callback.onError(request, e);
            }
        });
    }

    private void sendSuccessResultCallback(final Object object, final ResultCallback callback)
    {
        mDelivery.post(new Runnable()
        {
            @Override
            public void run()
            {
                if (callback != null)
                {
                    callback.onResponse(object);
                }
            }
        });
    }

    private Request buildPostRequest(String url, Param[] params)
    {
        if (params == null)
        {
            params = new Param[0];
        }
        FormEncodingBuilder builder = new FormEncodingBuilder();
        for (Param param : params)
        {
            builder.add(param.key, param.value);
        }
        RequestBody requestBody = builder.build();
        return new Request.Builder()
                .url(url)
                .post(requestBody)
                .build();
    }


    public static abstract class ResultCallback<T>
    {
        Type mType;

        public ResultCallback()
        {
            mType = getSuperclassTypeParameter(getClass());
        }

        static Type getSuperclassTypeParameter(Class<?> subclass)
        {
            Type superclass = subclass.getGenericSuperclass();
            if (superclass instanceof Class)
            {
                throw new RuntimeException("Missing type parameter.");
            }
            ParameterizedType parameterized = (ParameterizedType) superclass;
            return $Gson$Types.canonicalize(parameterized.getActualTypeArguments()[0]);
        }

        public abstract void onError(Request request, Exception e);

        public abstract void onResponse(T response);
    }

    public static class Param
    {
        public Param()
        {
        }

        public Param(String key, String value)
        {
            this.key = key;
            this.value = value;
        }

        String key;
        String value;
    }

}
```

源码地址[okhttp-utils](https://github.com/hongyangAndroid/okhttp-utils)，大家可以自己下载查看。
