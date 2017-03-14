## 1. Volley简介

我们平时在开发[Android](http://lib.csdn.net/base/android)应用的时候不可避免地都需要用到网络技术，而多数情况下应用程序都会使用HTTP协议来发送和接收网络数据。Android系统中主要提供了两种方式来进行HTTP通信，HttpURLConnection和HttpClient，几乎在任何项目的代码中我们都能看到这两个类的身影，使用率非常高。

不过HttpURLConnection和HttpClient的用法还是稍微有些复杂的，如果不进行适当封装的话，很容易就会写出不少重复代码。于是乎，一些Android网络通信框架也就应运而生，比如说AsyncHttpClient，它把HTTP所有的通信细节全部封装在了内部，我们只需要简单调用几行代码就可以完成通信操作了。再比如Universal-Image-Loader，它使得在界面上显示网络图片的操作变得极度简单，开发者不用关心如何从网络上获取图片，也不用关心开启线程、回收图片资源等细节，Universal-Image-Loader已经把一切都做好了。

Android开发团队也是意识到了有必要将HTTP的通信操作再进行简单化，于是在2013年Google I/O大会上推出了一个新的网络通信框架——Volley。Volley可是说是把AsyncHttpClient和Universal-Image-Loader的优点集于了一身，既可以像AsyncHttpClient一样非常简单地进行HTTP通信，也可以像Universal-Image-Loader一样轻松加载网络上的图片。除了简单易用之外，Volley在性能方面也进行了大幅度的调整，它的设计目标就是非常适合去进行数据量不大，但通信频繁的网络操作，而对于[大数据](http://lib.csdn.net/base/hadoop)量的网络操作，比如说下载文件等，Volley的表现就会非常糟糕。

下图所示的这些应用都是属于数据量不大，但网络通信频繁的，因此非常适合使用Volley。

![img](http://img.blog.csdn.net/20140406140008281?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZ3VvbGluX2Jsb2c=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 2. 下载Volley

介绍了这么多理论的东西，下面我们就准备开始进行实战了，首先需要将Volley的jar包准备好，如果你的电脑上装有[Git](http://lib.csdn.net/base/git)，可以使用如下命令下载Volley的源码：
```
git clone https://android.googlesource.com/platform/frameworks/volley  
```
下载完成后将它导入到你的Eclipse工程里，然后再导出一个jar包就可以了。如果你的电脑上没有Git，那么也可以直接使用我导出好的jar包，下载地址是：[http://download.csdn.net/detail/sinyu890807/7152015](http://download.csdn.net/detail/sinyu890807/7152015) 

新建一个Android项目，将volley.jar文件复制到libs目录下，这样准备工作就算是做好了

## 3. StringRequest的用法

前面已经说过，Volley的用法非常简单，那么我们就从最基本的HTTP通信开始学习吧，即发起一条HTTP请求，然后接收HTTP响应。首先需要获取到一个RequestQueue对象，可以调用如下方法获取到：

```java
RequestQueue mQueue = Volley.newRequestQueue(context);  
```

注意这里拿到的RequestQueue是一个请求队列对象，它可以缓存所有的HTTP请求，然后按照一定的[算法](http://lib.csdn.net/base/datastructure)并发地发出这些请求。RequestQueue内部的设计就是非常合适高并发的，因此我们不必为每一次HTTP请求都创建一个RequestQueue对象，这是非常浪费资源的，基本上在每一个需要和网络交互的Activity中创建一个RequestQueue对象就足够了。

接下来为了要发出一条HTTP请求，我们还需要创建一个StringRequest对象，如下所示：

```java
StringRequest stringRequest = new StringRequest("http://www.baidu.com",  
                        new Response.Listener<String>() {  
                            @Override  
                            public void onResponse(String response) {  
                                Log.d("TAG", response);  
                            }  
                        }, new Response.ErrorListener() {  
                            @Override  
                            public void onErrorResponse(VolleyError error) {  
                                 Log.e("TAG", error.getMessage(), error);  
                             }  
                         });  
```
可以看到，这里new出了一个StringRequest对象，StringRequest的构造函数需要传入三个参数，第一个参数就是目标服务器的URL地址，第二个参数是服务器响应成功的回调，第三个参数是服务器响应失败的回调。其中，目标服务器地址我们填写的是百度的首页，然后在响应成功的回调里打印出服务器返回的内容，在响应失败的回调里打印出失败的详细信息。

最后，将这个StringRequest对象添加到RequestQueue里面就可以了，如下所示：

```java
mQueue.add(stringRequest);  
```

另外，由于Volley是要访问网络的，因此不要忘记在你的AndroidManifest.xml中添加如下权限：

```xml
<uses-permission android:name="android.permission.INTERNET" />  
```

好了，就是这么简单，如果你现在运行一下程序，并发出这样一条HTTP请求，就会看到LogCat中会打印出如下图所示的数据。

![img](http://img.blog.csdn.net/20140406145955671?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZ3VvbGluX2Jsb2c=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

没错，百度返回给我们的就是这样一长串的HTML代码，虽然我们看起来会有些吃力，但是浏览器却可以轻松地对这段HTML代码进行解析，然后将百度的首页展现出来。

这样的话，一个最基本的HTTP发送与响应的功能就完成了。你会发现根本还没写几行代码就轻易实现了这个功能，主要就是进行了以下三步操作：

1. 创建一个RequestQueue对象
2. 创建一个StringRequest对象
3. 将StringRequest对象添加到RequestQueue里面

不过大家都知道，HTTP的请求类型通常有两种，GET和POST，刚才我们使用的明显是一个GET请求，那么如果想要发出一条POST请求应该怎么做呢？StringRequest中还提供了另外一种四个参数的构造函数，其中第一个参数就是指定请求类型的，我们可以使用如下方式进行指定：


```java
StringRequest stringRequest = new StringRequest(Method.POST, url,  listener, errorListener);  
```

可是这只是指定了HTTP请求方式是POST，那么我们要提交给服务器的参数又该怎么设置呢？很遗憾，StringRequest中并没有提供设置POST参数的方法，但是当发出POST请求的时候，Volley会尝试调用StringRequest的父类——Request中的getParams()方法来获取POST参数，那么解决方法自然也就有了，我们只需要在StringRequest的匿名类中重写getParams()方法，在这里设置POST参数就可以了，代码如下所示：


```java
StringRequest stringRequest = new StringRequest(Method.POST, url,  listener, errorListener) {  
    @Override  
    protected Map<String, String> getParams() throws AuthFailureError {  
        Map<String, String> map = new HashMap<String, String>();  
        map.put("params1", "value1");  
        map.put("params2", "value2");  
        return map;  
    }  
};  
```

你可能会说，每次都这样用起来岂不是很累？连个设置POST参数的方法都没有。但是不要忘记，Volley是开源的，只要你愿意，你可以自由地在里面添加和修改任何的方法，轻松就能定制出一个属于你自己的Volley版本。

## 4. JsonRequest的用法

学完了最基本的StringRequest的用法，我们再来进阶学习一下JsonRequest的用法。类似于StringRequest，JsonRequest也是继承自Request类的，不过由于JsonRequest是一个抽象类，因此我们无法直接创建它的实例，那么只能从它的子类入手了。JsonRequest有两个直接的子类，JsonObjectRequest和JsonArrayRequest，从名字上你应该能就看出它们的区别了吧？一个是用于请求一段JSON数据的，一个是用于请求一段JSON数组的。

至于它们的用法也基本上没有什么特殊之处，先new出一个JsonObjectRequest对象，如下所示：


```java
JsonObjectRequest jsonObjectRequest = new JsonObjectRequest("http://m.weather.com.cn/data/101010100.html", null,  
        new Response.Listener<JSONObject>() {  
            @Override  
            public void onResponse(JSONObject response) {  
                Log.d("TAG", response.toString());  
            }  
        }, new Response.ErrorListener() {  
            @Override  
            public void onErrorResponse(VolleyError error) {  
                 Log.e("TAG", error.getMessage(), error);  
             }  
         });  
```
可以看到，这里我们填写的URL地址是http://m.weather.com.cn/data/101010100.html，这是中国天气网提供的一个查询天气信息的接口，响应的数据就是以JSON格式返回的，然后我们在onResponse()方法中将返回的数据打印出来。

最后再将这个JsonObjectRequest对象添加到RequestQueue里就可以了，如下所示：


```java
mQueue.add(jsonObjectRequest);  
```

这样当HTTP通信完成之后，服务器响应的天气信息就会回调到onResponse()方法中，并打印出来。现在运行一下程序，发出这样一条HTTP请求，就会看到LogCat中会打印出如下图所示的数据。

![img](http://img.blog.csdn.net/20140406163801937?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZ3VvbGluX2Jsb2c=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

由此可以看出，服务器返回给我们的数据确实是JSON格式的，并且onResponse()方法中携带的参数也正是一个JSONObject对象，之后只需要从JSONObject对象取出我们想要得到的那部分数据就可以了。

你应该发现了吧，JsonObjectRequest的用法和StringRequest的用法基本上是完全一样的，Volley的易用之处也在这里体现出来了，会了一种就可以让你举一反三，因此关于JsonArrayRequest的用法相信已经不需要我再去讲解了吧。

好了，关于Volley的基本用法就讲到这里，下篇文章中我会带领大家继续探究Volley。感兴趣的朋友请继续阅读[**Android Volley完全解析(二)，使用Volley加载网络图片**](http://blog.csdn.net/guolin_blog/article/details/17482165)。
