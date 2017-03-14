# 1. 概述

之前写了篇[Android OkHttp完全解析 是时候来了解OkHttp了](http://blog.csdn.net/lmj623565791/article/details/47911083)，其实主要是作为okhttp的普及文章，当然里面也简单封装了工具类，没想到关注和使用的人还挺多的，由于这股热情，该工具类中的方法也是剧增，各种重载方法，以致于使用起来极不方便，实在惭愧。

于是，在这个周末，抽点时间对该工具类，进行了重新的拆解与编写，顺便完善下功能，尽可能的提升其使用起来的方便性和易扩展性。

标题的改善，也是指的是对于我之前的代码进行改善。

如果你对okhttp不了解，可以通过[Android OkHttp完全解析 是时候来了解OkHttp了](http://blog.csdn.net/lmj623565791/article/details/47911083)进行了解。

ok，那么目前，该封装库志支持：

- 一般的get请求
- 一般的post请求
- 基于Http的文件上传
- 文件下载
- 上传下载的进度回调
- 加载图片
- 支持请求回调，直接返回对象、对象集合
- 支持session的保持
- 支持自签名网站https的访问，提供方法设置下证书就行
- 支持取消某个请求

源码地址：[https://github.com/hongyangAndroid/okhttp-utils](https://github.com/hongyangAndroid/okhttp-utils)

引入：

- [Android](http://lib.csdn.net/base/android) Studio

使用前，对于Android Studio的用户，可以选择添加:

```gradle
compile project(':okhttputils')11
```

或者

```gradle
compile 'com.zhy:okhttputils:2.0.0'11
```

- Eclipse 自行copy源码。

# 2. 基本用法

目前基本的用法格式为：

```java
OkHttpUtils
    .get()
    .url(url)
    .addParams("username", "hyman")
    .addParams("password", "123")
    .build()
    .execute(callback);12345671234567
```

通过链式去根据自己的需要添加各种参数，最后调用execute(callback)进行执行，传入callback则代表是异步。如果单纯的execute()则代表同步的方法调用。

可以看到，取消了之前一堆的get重载方法，参数也可以进行灵活的选择了。

下面简单看一下，全部的用法：

## 2.1 GET请求

```java
String url = "http://www.csdn.net/";
OkHttpUtils
    .get()
    .url(url)
    .addParams("username", "hyman")
    .addParams("password", "123")
    .build()
    .execute(new StringCallback()
            {
                @Override
                public void onError(Request request, Exception e)
                {

                }

                @Override
                public void onResponse(String response)
                {

                }
            });123456789101112131415161718192021123456789101112131415161718192021
```

## 2.2 POST请求

```java
 OkHttpUtils
    .post()
    .url(url)
    .addParams("username", "hyman")
    .addParams("password", "123")
    .build()
    .execute(callback);
1234567812345678
```

## 2.3 Post String

```java
OkHttpUtils
    .postString()
    .url(url)
    .content(new Gson().toJson(new User("zhy", "123")))
    .build()
    .execute(new MyStringCallback());   123456123456
```

将string作为请求体传入到服务端，例如json字符串。

## 2.4 Post File

```java
OkHttpUtils
    .postFile()
    .url(url)
    .file(file)
    .build()
    .execute(new MyStringCallback());123456123456
```

将file作为请求体传入到服务端.

## 2.5 基于POST的文件上传（类似web上的表单）

```java
OkHttpUtils.post()//
    .addFile("mFile", "messenger_01.png", file)//
    .addFile("mFile", "test1.txt", file2)//
    .url(url)
    .params(params)//
    .headers(headers)//
    .build()//
    .execute(new MyStringCallback());1234567812345678
```

## 2.6 下载文件

```java
OkHttpUtils//
    .get()//
    .url(url)//
    .build()//
    .execute(new FileCallBack(Environment.getExternalStorageDirectory().getAbsolutePath(), "gson-2.2.1.jar")//
    {
        @Override
        public void inProgress(float progress)
        {
            mProgressBar.setProgress((int) (100 * progress));
        }

        @Override
        public void onError(Request request, Exception e)
        {
            Log.e(TAG, "onError :" + e.getMessage());
        }

        @Override
        public void onResponse(File file)
        {
            Log.e(TAG, "onResponse :" + file.getAbsolutePath());
        }
    });123456789101112131415161718192021222324123456789101112131415161718192021222324
```

## 2.7 显示图片

```java
OkHttpUtils
    .get()//
    .url(url)//
    .build()//
    .execute(new BitmapCallback()
    {
        @Override
        public void onError(Request request, Exception e)
        {
            mTv.setText("onError:" + e.getMessage());
        }

        @Override
        public void onResponse(Bitmap bitmap)
        {
            mImageView.setImageBitmap(bitmap);
        }
    });
```

哈，目前来看，清晰多了。

# 3. 对于上传下载的回调

```java
new Callback<?>()
{
    //...
    @Override
    public void inProgress(float progress)
    {
       //use progress: 0 ~ 1
    }
}
```

对于传入的callback有个inProgress方法，需要拿到进度直接复写该方法即可。

# 4. 对于自动解析为实体类

目前去除了Gson的依赖，提供了自定义Callback的方式，让用户自己去解析返回的数据，目前提供了`StringCallback`，`FileCallback`,`BitmapCallback` 分别用于返回string，文件下载，加载图片。

当然如果你希望解析为对象，你可以：

```java
public abstract class UserCallback extends Callback<User>
{
    //非UI线程，支持任何耗时操作
    @Override
    public User parseNetworkResponse(Response response) throws IOException
    {
        String string = response.body().string();
        User user = new Gson().fromJson(string, User.class);
        return user;
    }
}
```

自己使用自己喜欢的Json解析库完成即可。

解析成`List<User>`，则如下：

```java
public abstract class ListUserCallback extends Callback<List<User>>
{
    @Override
    public List<User> parseNetworkResponse(Response response) throws IOException
    {
        String string = response.body().string();
        List<User> user = new Gson().fromJson(string, List.class);
        return user;
    }


}
```

# 5. 对于https单向认证

非常简单，拿到xxx.cert的证书。然后调用

```java
OkHttpUtils.getInstance().setCertificates(inputstream);123123
```

建议使用方式，例如我的证书放在assets目录：

```java
/**
 * Created by zhy on 15/8/25.
 */
public class MyApplication extends Application
{
    @Override
    public void onCreate()
    {
        super.onCreate();

        try
        {    
        OkHttpUtils
         .getInstance()
         .setCertificates(getAssets().open("aaa.cer"),
 getAssets().open("server.cer"));
        } catch (IOException e)
        {
            e.printStackTrace();
        }
    }
}
```

即可。别忘了注册Application。

注意：如果https网站为权威机构颁发的证书，不需要以上设置。自签名的证书才需要。

# 6. 配置

## 6.1 全局配置

可以在Application中，通过：

```java
OkHttpClient client =
OkHttpUtils.getInstance().getOkHttpClient();1212
```

然后调用client的各种set方法。例如：

```java
client.setConnectTimeout(100000, TimeUnit.MILLISECONDS);11
```

## 6.2 为单个请求设置超时

比如涉及到文件的需要设置读写等待时间多一点。

```java
 OkHttpUtils
    .get()//
    .url(url)//
    .tag(this)//
    .build()//
    .connTimeOut(20000)
    .readTimeOut(20000)
    .writeTimeOut(20000)
    .execute()123456789123456789
```

调用build()之后，可以随即设置各种timeOut.

## 6.3 取消单个请求

```java
 RequestCall call = OkHttpUtils.get().url(url).build();
 call.cancel();
```

## 6.4 根据tag取消请求

目前对于支持的方法都添加了最后一个参数`Object tag`，取消则通过`OkHttpUtils.cancelTag(tag)`执行。

例如：在Activity中，当Activity销毁取消请求：

```java
OkHttpUtils
    .get()//
    .url(url)//
    .tag(this)//
    .build()//

@Override
protected void onDestroy()
{
    super.onDestroy();
    //可以取消同一个tag的
    OkHttpUtils.cancelTag(this);//取消以Activity.this作为tag的请求
}
```

比如，当前Activity页面所有的请求以Activity对象作为tag，可以在onDestory里面统一取消。

# 7. 浅谈封装

其实整个封装的过程比较简单，这里简单描述下，对于okhttp一个请求的流程大致是这样的：

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

其中主要的差异，其实就是request的构建过程。我对Request抽象了一个类：`OkHttpRequest`

```java
public abstract class OkHttpRequest
{
    protected RequestBody requestBody;
    protected Request request;

    protected String url;
    protected String tag;
    protected Map<String, String> params;
    protected Map<String, String> headers;

    protected OkHttpRequest(String url, String tag,
                            Map<String, String> params, Map<String, String> headers)
    {
        this.url = url;
        this.tag = tag;
        this.params = params;
        this.headers = headers;
    }

    protected abstract Request buildRequest();
    protected abstract RequestBody buildRequestBody();

    protected void prepareInvoked(ResultCallback callback)
    {
        requestBody = buildRequestBody();
        requestBody = wrapRequestBody(requestBody, callback);
        request = buildRequest();
    }

    protected RequestBody wrapRequestBody(RequestBody requestBody, final ResultCallback callback)
    {
        return requestBody;
    }


    public void invokeAsyn(ResultCallback callback)
    {
        prepareInvoked(callback);
        mOkHttpClientManager.execute(request, callback);
    }


     // other common methods
 }   
```

一个request的构建呢，我分三个步骤：`buildRequestBody` , `wrapRequestBody` ,`buildRequest`这样的次序，当以上三个方法没有问题时，我们就拿到了request，然后执行即可。

但是对于不同的请求，requestBody以及request的构建过程是不同的，所以大家可以看到`buildRequestBody` ,`buildRequest`为抽象的方法，也就是不同的请求类，比如`OkHttpGetRequest`、`OkHttpPostRequest`等需要自己去构建自己的request。

对于`wrapRequestBody`方法呢，可以看到它默认基本属于空实现，主要是因为并非所有的请求类都需要复写它，只有上传的时候呢，需要回调进度，需要对requestBody进行包装，所以这个方法类似于一个钩子。

其实这个过程有点类似模板方法模式，有兴趣可以看看一个短篇介绍[设计模式 模版方法模式 展现程序员的一天](http://blog.csdn.net/lmj623565791/article/details/26276093) .

对于更加详细的用法，可以查看github上面的readme，以及demo，目前demo包含：

![img](http://img.blog.csdn.net/20151109094247142)

对于上传文件的两个按钮，需要自己搭建服务器，其他的按钮可以直接[测试](http://lib.csdn.net/base/softwaretest)。

最后，由于本人水平有限，以及时间比较仓促~~发现问题，欢迎提issue，我会抽时间解决
