经过前面两篇文章的学习，我们已经掌握了Volley各种Request的使用方法，包括StringRequest、JsonRequest、ImageRequest等。其中StringRequest用于请求一条普通的文本数据，JsonRequest(JsonObjectRequest、JsonArrayRequest)用于请求一条JSON格式的数据，ImageRequest则是用于请求网络上的一张图片。

可是Volley提供给我们的Request类型就只有这么多，而我们都知道，在网络上传输的数据通常有两种格式，JSON和XML，那么如果想要请求一条XML格式的数据该怎么办呢？其实很简单，Volley提供了非常强的扩展机制，使得我们可以很轻松地定制出任意类型的Request，这也就是本篇文章的主题了。

在开始之前还是友情提醒一下，如果你还没有阅读过我前面两篇关于Volley的文章，建议先去阅读一下[**Android Volley完全解析(一)，初识Volley的基本用法**](http://blog.csdn.net/guolin_blog/article/details/17482095)和[**Android Volley完全解析(二)，使用Volley加载网络图片**](http://blog.csdn.net/guolin_blog/article/details/17482165)。

## 1. 自定义XMLRequest

下面我们准备自定义一个XMLRequest，用于请求一条XML格式的数据。那么该从哪里开始入手呢？额，好像是有些无从下手。遇到这种情况，我们应该去参考一下Volley的源码，看一看StringRequest是怎么实现的，然后就可以模仿着写出XMLRequest了。首先看下StringRequest的源码，如下所示：

```java
/**
 * A canned request for retrieving the response body at a given URL as a String.
 */  
public class StringRequest extends Request<String> {  
    private final Listener<String> mListener;  
  
    /**
     * Creates a new request with the given method.
     *
     * @param method the request {@link Method} to use
     * @param url URL to fetch the string at
     * @param listener Listener to receive the String response
     * @param errorListener Error listener, or null to ignore errors
     */  
    public StringRequest(int method, String url, Listener<String> listener,  
            ErrorListener errorListener) {  
        super(method, url, errorListener);  
        mListener = listener;  
    }  
  
    /*
     * Creates a new GET request.
     *
     * @param url URL to fetch the string at
     * @param listener Listener to receive the String response
     * @param errorListener Error listener, or null to ignore errors
     */  
    public StringRequest(String url, Listener<String> listener, ErrorListener errorListener) {  
        this(Method.GET, ul, listener, errorListener);  
    }  
  
    @Override  
    protected void deliverResponse(String response) {  
        mListener.onResponse(response);  
    }  
  
    @Override  
    protected Response<String> parseNetworkResponse(NetworkResponse response) {  
        String parsed;  
        try {  
            parsed = new String(response.data, HttpHeaderParser.parseCharset(response.headers));  
        } catch (UnsupportedEncodingException e) {  
            parsed = new String(response.data);  
        }  
        return Response.success(parsed, HttpHeaderParser.parseCacheHeaders(response));  
    }  
 }  
```
可以看到，StringRequest的源码很简练，根本就没几行代码，我们一起来分析下。首先StringRequest是继承自Request类的，Request可以指定一个泛型类，这里指定的当然就是String了，接下来StringRequest中提供了两个有参的构造函数，参数包括请求类型，请求地址，以及响应回调等，由于我们已经很熟悉StringRequest的用法了，相信这几个参数的作用都不用再解释了吧。但需要注意的是，在构造函数中一定要调用super()方法将这几个参数传给父类，因为HTTP的请求和响应都是在父类中自动处理的。

另外，由于Request类中的deliverResponse()和parseNetworkResponse()是两个抽象方法，因此StringRequest中需要对这两个方法进行实现。deliverResponse()方法中的实现很简单，仅仅是调用了mListener中的onResponse()方法，并将response内容传入即可，这样就可以将服务器响应的数据进行回调了。parseNetworkResponse()方法中则应该对服务器响应的数据进行解析，其中数据是以字节的形式存放在NetworkResponse的data变量中的，这里将数据取出然后组装成一个String，并传入Response的success()方法中即可。

了解了StringRequest的实现原理，下面我们就可以动手来尝试实现一下XMLRequest了，代码如下所示：

```java
public class XMLRequest extends Request<XmlPullParser> {  
  
    private final Listener<XmlPullParser> mListener;  
  
    public XMLRequest(int method, String url, Listener<XmlPullParser> listener,  
            ErrorListener errorListener) {  
        super(method, url, errorListener);  
        mListener = listener;  
    }  
   
     public XMLRequest(String url, Listener<XmlPullParser> listener, ErrorListener errorListener) {  
         this(Method.GET, url, listener, errorListener);  
     }  
   
     @Override  
     protected Response<XmlPullParser> parseNetworkResponse(NetworkResponse response) {  
         try {  
             String xmlString = new String(response.data,  
                     HttpHeaderParser.parseCharset(response.headers));  
             XmlPullParserFactory factory = XmlPullParserFactory.newInstance();  
             XmlPullParser xmlPullParser = factory.newPullParser();  
             xmlPullParser.setInput(new StringReader(xmlString));  
             return Response.success(xmlPullParser, HttpHeaderParser.parseCacheHeaders(response));  
         } catch (UnsupportedEncodingException e) {  
             return Response.error(new ParseError(e));  
         } catch (XmlPullParserException e) {  
             return Response.error(new ParseError(e));  
         }  
     }  
   
     @Override  
     protected void deliverResponse(XmlPullParser response) {  
         mListener.onResponse(response);  
     }  
   
}  
```
可以看到，其实并没有什么太多的逻辑，基本都是仿照StringRequest写下来的，XMLRequest也是继承自Request类的，只不过这里指定的泛型类是XmlPullParser，说明我们准备使用Pull解析的方式来解析XML。在parseNetworkResponse()方法中，先是将服务器响应的数据解析成一个字符串，然后设置到XmlPullParser对象中，在deliverResponse()方法中则是将XmlPullParser对象进行回调。

好了，就是这么简单，下面我们尝试使用这个XMLRequest来请求一段XML格式的数据。[http://flash.weather.com.cn/wmaps/xml/china.xml](http://flash.weather.com.cn/wmaps/xml/china.xml)这个接口会将中国所有的省份数据以XML格式进行返回，如下所示：

```xml
<china dn="day" slick-uniqueid="3">  
<city quName="黑龙江" pyName="heilongjiang" cityname="哈尔滨" state1="0" state2="0" stateDetailed="晴" tem1="18" tem2="6" windState="西北风3-4级转西风小于3级"/>  
<city quName="吉林" pyName="jilin" cityname="长春" state1="0" state2="0" stateDetailed="晴" tem1="19" tem2="6" windState="西北风3-4级转小于3级"/>  
<city quName="辽宁" pyName="liaoning" cityname="沈阳" state1="0" state2="0" stateDetailed="晴" tem1="21" tem2="7" windState="东北风3-4级"/>  
<city quName="海南" pyName="hainan" cityname="海口" state1="1" state2="1" stateDetailed="多云" tem1="30" tem2="24" windState="微风"/>  
<city quName="内蒙古" pyName="neimenggu" cityname="呼和浩特" state1="0" state2="0" stateDetailed="晴" tem1="19" tem2="5" windState="东风3-4级"/>  
<city quName="新疆" pyName="xinjiang" cityname="乌鲁木齐" state1="0" state2="0" stateDetailed="晴" tem1="22" tem2="10" windState="微风转东南风小于3级"/>  
<city quName="西藏" pyName="xizang" cityname="拉萨" state1="1" state2="7" stateDetailed="多云转小雨" tem1="18" tem2="4" windState="微风"/>  
<city quName="青海" pyName="qinghai" cityname="西宁" state1="0" state2="1" stateDetailed="晴转多云" tem1="18" tem2="2" windState="微风"/>  
<city quName="宁夏" pyName="ningxia" cityname="银川" state1="0" state2="0" stateDetailed="晴" tem1="19" tem2="8" windState="微风"/>  
<city quName="甘肃" pyName="gansu" cityname="兰州" state1="0" state2="0" stateDetailed="晴" tem1="21" tem2="6" windState="微风"/>  
<city quName="河北" pyName="hebei" cityname="石家庄" state1="0" state2="0" stateDetailed="晴" tem1="25" tem2="12" windState="北风小于3级"/>  
<city quName="河南" pyName="henan" cityname="郑州" state1="0" state2="0" stateDetailed="晴" tem1="24" tem2="13" windState="微风"/>  
<city quName="湖北" pyName="hubei" cityname="武汉" state1="0" state2="0" stateDetailed="晴" tem1="24" tem2="12" windState="微风"/>  
<city quName="湖南" pyName="hunan" cityname="长沙" state1="2" state2="1" stateDetailed="阴转多云" tem1="20" tem2="15" windState="北风小于3级"/>  
<city quName="山东" pyName="shandong" cityname="济南" state1="1" state2="1" stateDetailed="多云" tem1="20" tem2="10" windState="北风3-4级转小于3级"/>  
<city quName="江苏" pyName="jiangsu" cityname="南京" state1="2" state2="2" stateDetailed="阴" tem1="19" tem2="13" windState="西北风4-5级转3-4级"/>  
<city quName="安徽" pyName="anhui" cityname="合肥" state1="2" state2="1" stateDetailed="阴转多云" tem1="20" tem2="12" windState="西北风转北风3-4级"/>  
<city quName="山西" pyName="shanxi" cityname="太原" state1="0" state2="0" stateDetailed="晴" tem1="22" tem2="8" windState="微风"/>  
<city quName="陕西" pyName="sanxi" cityname="西安" state1="1" state2="0" stateDetailed="多云转晴" tem1="21" tem2="9" windState="东北风小于3级"/>  
<city quName="四川" pyName="sichuan" cityname="成都" state1="1" state2="1" stateDetailed="多云" tem1="26" tem2="15" windState="南风小于3级"/>  
<city quName="云南" pyName="yunnan" cityname="昆明" state1="7" state2="7" stateDetailed="小雨" tem1="21" tem2="13" windState="微风"/>  
<city quName="贵州" pyName="guizhou" cityname="贵阳" state1="1" state2="3" stateDetailed="多云转阵雨" tem1="21" tem2="11" windState="东风小于3级"/>  
<city quName="浙江" pyName="zhejiang" cityname="杭州" state1="3" state2="1" stateDetailed="阵雨转多云" tem1="22" tem2="14" windState="微风"/>  
<city quName="福建" pyName="fujian" cityname="福州" state1="1" state2="2" stateDetailed="多云转阴" tem1="28" tem2="18" windState="微风"/>  
<city quName="江西" pyName="jiangxi" cityname="南昌" state1="2" state2="1" stateDetailed="阴转多云" tem1="23" tem2="15" windState="北风3-4级转微风"/>  
<city quName="广东" pyName="guangdong" cityname="广州" state1="3" state2="2" stateDetailed="阵雨转阴" tem1="26" tem2="20" windState="微风"/>  
<city quName="广西" pyName="guangxi" cityname="南宁" state1="3" state2="3" stateDetailed="阵雨" tem1="23" tem2="19" windState="东北风小于3级"/>  
<city quName="北京" pyName="beijing" cityname="北京" state1="0" state2="0" stateDetailed="晴" tem1="26" tem2="10" windState="微风"/>  
<city quName="天津" pyName="tianjin" cityname="天津" state1="1" state2="0" stateDetailed="多云转晴" tem1="22" tem2="13" windState="东北风3-4级转小于3级"/>  
<city quName="上海" pyName="shanghai" cityname="上海" state1="7" state2="1" stateDetailed="小雨转多云" tem1="20" tem2="16" windState="西北风3-4级"/>  
<city quName="重庆" pyName="chongqing" cityname="重庆" state1="1" state2="3" stateDetailed="多云转阵雨" tem1="21" tem2="14" windState="微风"/>  
<city quName="香港" pyName="xianggang" cityname="香港" state1="3" state2="1" stateDetailed="阵雨转多云" tem1="26" tem2="22" windState="微风"/>  
<city quName="澳门" pyName="aomen" cityname="澳门" state1="3" state2="1" stateDetailed="阵雨转多云" tem1="27" tem2="22" windState="东北风3-4级转微风"/>  
<city quName="台湾" pyName="taiwan" cityname="台北" state1="9" state2="7" stateDetailed="大雨转小雨" tem1="28" tem2="21" windState="微风"/>  
<city quName="西沙" pyName="xisha" cityname="西沙" state1="3" state2="3" stateDetailed="阵雨" tem1="30" tem2="26" windState="东北风4-5级"/>  
<city quName="南沙" pyName="nanshadao" cityname="南沙" state1="1" state2="1" stateDetailed="多云" tem1="32" tem2="27" windState="东风4-5级"/>  
<city quName="钓鱼岛" pyName="diaoyudao" cityname="钓鱼岛" state1="7" state2="1" stateDetailed="小雨转多云" tem1="23" tem2="19" windState="西南风3-4级转北风5-6级"/>  
</china>    
```
确定了访问接口后，我们只需要在代码中按照以下的方式来使用XMLRequest即可：

```java
XMLRequest xmlRequest = new XMLRequest(  
        "http://flash.weather.com.cn/wmaps/xml/china.xml",  
        new Response.Listener<XmlPullParser>() {  
            @Override  
            public void onResponse(XmlPullParser response) {  
                try {  
                    int eventType = response.getEventType();  
                    while (eventType != XmlPullParser.END_DOCUMENT) {  
                        switch (eventType) {  
                         case XmlPullParser.START_TAG:  
                             String nodeName = response.getName();  
                             if ("city".equals(nodeName)) {  
                                 String pName = response.getAttributeValue(0);  
                                 Log.d("TAG", "pName is " + pName);  
                             }  
                             break;  
                         }  
                        eventType = response.next();  
                     }  
                 } catch (XmlPullParserException e) {  
                     e.printStackTrace();  
                 } catch (IOException e) {  
                     e.printStackTrace();  
                 }  
             }  
         }, new Response.ErrorListener() {  
             @Override  
             public void onErrorResponse(VolleyError error) {  
                 Log.e("TAG", error.getMessage(), error);  
             }  
        });  
 mQueue.add(xmlRequest);  
```
可以看到，这里XMLRequest的用法和StringRequest几乎是一模一样的，我们先创建出一个XMLRequest的实例，并把服务器接口地址传入，然后在onResponse()方法中解析响应的XML数据，并把每个省的名字打印出来，最后将这个XMLRequest添加到RequestQueue当中。

现在运行一下代码，观察控制台日志，就可以看到每个省的名字都从XML中解析出来了，如下图所示。

![img](http://img.blog.csdn.net/20140427151850296?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZ3VvbGluX2Jsb2c=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 2. 自定义GsonRequest

JsonRequest的数据解析是利用[Android](http://lib.csdn.net/base/android)本身自带的JSONObject和JSONArray来实现的，配合使用JSONObject和JSONArray就可以解析出任意格式的JSON数据。不过也许你会觉得使用JSONObject还是太麻烦了，还有很多方法可以让JSON数据解析变得更加简单，比如说GSON。遗憾的是，Volley中默认并不支持使用自家的GSON来解析数据，不过没有关系，通过上面的学习，相信你已经知道了自定义一个Request是多么的简单，那么下面我们就来举一反三一下，自定义一个GsonRequest。

首先我们需要把gson的jar包添加到项目当中，jar包的下载地址是：[https://code.google.com/p/google-gson/downloads/list](https://code.google.com/p/google-gson/downloads/list) 。

接着定义一个GsonRequest继承自Request，代码如下所示：

```java
public class GsonRequest<T> extends Request<T> {  
  
    private final Listener<T> mListener;  
  
    private Gson mGson;  
  
    private Class<T> mClass;  
  
    public GsonRequest(int method, String url, Class<T> clazz, Listener<T> listener,  
             ErrorListener errorListener) {  
         super(method, url, errorListener);  
         mGson = new Gson();  
         mClass = clazz;  
         mListener = listener;  
     }  
   
     public GsonRequest(String url, Class<T> clazz, Listener<T> listener,  
             ErrorListener errorListener) {  
         this(Method.GET, url, clazz, listener, errorListener);  
     }  
   
     @Override  
     protected Response<T> parseNetworkResponse(NetworkResponse response) { 
         try {  
             String jsonString = new String(response.data,  
                     HttpHeaderParser.parseCharset(response.headers));  
             return Response.success(mGson.fromJson(jsonString, mClass),  
                     HttpHeaderParser.parseCacheHeaders(response));  
         } catch (UnsupportedEncodingException e) {  
             return Response.error(new ParseError(e));  
         }  
     }  
   
     @Override  
     protected void deliverResponse(T response) {  
         mListener.onResponse(response);  
     }  
  
}  
```
可以看到，GsonRequest是继承自Request类的，并且同样提供了两个构造函数。在parseNetworkResponse()方法中，先是将服务器响应的数据解析出来，然后通过调用Gson的fromJson方法将数据组装成对象。在deliverResponse方法中仍然是将最终的数据进行回调。

那么下面我们就来[测试](http://lib.csdn.net/base/softwaretest)一下这个GsonRequest能不能够正常工作吧，调用[http://www.weather.com.cn/data/sk/101010100.html](http://www.weather.com.cn/data/sk/101010100.html)这个接口可以得到一段JSON格式的天气数据，如下所示：

```
{"weatherinfo":{"city":"北京","cityid":"101010100","temp":"19","WD":"南风","WS":"2级","SD":"43%","WSE":"2","time":"19:45","isRadar":"1","Radar":"JC_RADAR_AZ9010_JB"}}  
```

接下来我们使用对象的方式将这段JSON字符串表示出来。新建一个Weather类，代码如下所示：

```java
public class Weather {  
  
    private WeatherInfo weatherinfo;  
  
    public WeatherInfo getWeatherinfo() {  
        return weatherinfo;  
    }  
  
    public void setWeatherinfo(WeatherInfo weatherinfo) {  
         this.weatherinfo = weatherinfo;  
     }  
   
 }  
```
Weather类中只是引用了WeatherInfo这个类。接着新建WeatherInfo类，代码如下所示：

```java
public class WeatherInfo {  
  
    private String city;  
  
    private String temp;  
  
    private String time;  
  
    public String getCity() {  
        return city;  
    }  
   
    public void setCity(String city) {  
        this.city = city;  
    }  
   
    public String getTemp() {  
        return temp;  
    }  
   
    public void setTemp(String temp) {  
        this.temp = temp;  
    }  
   
    public String getTime() {  
        return time;  
    }  
   
    public void setTime(String time) {  
        this.time = time;  
    }  
   
}  
```
WeatherInfo类中含有city、temp、time这几个字段。下面就是如何调用GsonRequest了，其实也很简单，代码如下所示：

```java
GsonRequest<Weather> gsonRequest = new GsonRequest<Weather>(  
        "http://www.weather.com.cn/data/sk/101010100.html", Weather.class,  
        new Response.Listener<Weather>() {  
            @Override  
            public void onResponse(Weather weather) {  
                WeatherInfo weatherInfo = weather.getWeatherinfo();  
                Log.d("TAG", "city is " + weatherInfo.getCity());  
                Log.d("TAG", "temp is " + weatherInfo.getTemp());  
                Log.d("TAG", "time is " + weatherInfo.getTime());  
             }  
         }, new Response.ErrorListener() {  
             @Override  
             public void onErrorResponse(VolleyError error) {  
                 Log.e("TAG", error.getMessage(), error);  
             }  
         });  
 mQueue.add(gsonRequest);  
```
可以看到，这里onResponse()方法的回调中直接返回了一个Weather对象，我们通过它就可以得到WeatherInfo对象，接着就能从中取出JSON中的相关数据了。现在运行一下代码，观察控制台日志，打印数据如下图所示：

![img](http://img.blog.csdn.net/20140427202306328?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZ3VvbGluX2Jsb2c=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

这样的话，XMLRequest和GsonRequest的功能就基本都实现了，我们也是借助这两个例子深刻地理解了自定义Request的方法，对Volley的认识也是更加深入了。好了，本篇文章就到此结束，下篇文章中我们将对Volley进行更深层次的研究，感兴趣的朋友请继续阅读[**Android Volley完全解析(四)，带你从源码的角度理解Volley**](http://blog.csdn.net/guolin_blog/article/details/17656437)。
