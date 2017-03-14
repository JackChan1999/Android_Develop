# 1. 概述

之前写了个okhttputils的工具类，然后有很多同学询问这个工具类和`retrofit`什么区别，于是上了下官网，发现其底层对网络的访问默认也是基于`okhttp`，不过`retrofit`非常适合于`restful url`格式的请求，更多使用注解的方式提供功能。

既然这样，我们本篇博文首先研究其所提供的常用的用法：

- 一般的get、post请求
- 动态url，动态参数设置，各种注解的使用
- 上传文件（单文件，多文件上传等）
- 下载文件等（这个不推荐retrofit去做，具体看下文）

此外，由于其内部提供了`ConverterFactory`用于对返回的`requestBody`进行转化和特殊的`requestBody`的构造，所以本文也包含：

- 如何自定义`ConverterFactory`

最后呢，因为其源码并不复杂，本文将对源码进行整体的介绍，即

- Retrofit源码分析

ok，说这么多，既然需要`restful url`，我只能捡起我那个半桶水的[spring](http://lib.csdn.net/base/javaee) mvc 搭建一个服务端的小例子~~

最后本文使用版本：

```gradle
compile 'com.squareup.retrofit2:retrofit:2.0.2'11
```

主要是源码解析，自定义`Converter.Factory`等一些细节的探索。

恩，写完后，发现本文很长，中途请没事站起来走两步。

retrofit2官网地址：[https://github.com/square/retrofit/](https://github.com/square/retrofit/)

# 2. Retrofit用法示例

## 2.1 一般的get请求

`retrofit`在使用的过程中，需要定义一个接口对象，我们首先演示一个最简单的get请求，接口如下所示：

```java
public interface IUserBiz
{
    @GET("users")
    Call<List<User>> getUsers();
}
```

可以看到有一个getUsers(）方法，通过`@GET`注解标识为get请求，`@GET`中所填写的value和`baseUrl`组成完整的路径，`baseUrl`在构造retrofit对象时给出。

下面看如何通过`retrofit`完成上述的请求：

```java
Retrofit retrofit = new Retrofit.Builder()
        .baseUrl("http://192.168.31.242:8080/springmvc_users/user/")
        .addConverterFactory(GsonConverterFactory.create())
        .build();
IUserBiz userBiz = retrofit.create(IUserBiz.class);
Call<List<User>> call = userBiz.getUsers();
        call.enqueue(new Callback<List<User>>()
        {
            @Override
            public void onResponse(Call<List<User>> call, Response<List<User>> response)
            {
                Log.e(TAG, "normalGet:" + response.body() + "");
            }

            @Override
            public void onFailure(Call<List<User>> call, Throwable t)
            {

            }
        });
```

依然是构造者模式，指定了`baseUrl`和`Converter.Factory`，该对象通过名称可以看出是用于对象转化的，本例因为服务器返回的是json格式的数组，所以这里设置了`GsonConverterFactory`完成对象的转化。

ok，这里可以看到很神奇，我们通过`Retrofit.create`就可以拿到我们定义的`IUserBiz`的实例，调用其方法即可拿到一个`Call`对象，通过`call.enqueue`即可完成异步的请求。

具体retrofit怎么得到我们接口的实例的，以及对象的返回结果是如何转化的，我们后面具体分析。

这里需要指出的是：

1. 接口中的方法必须有返回值，且比如是`Call<T>`类型

2. `.addConverterFactory(GsonConverterFactory.create())`这里如果使用gson，需要额外导入：

```gradle
compile 'com.squareup.retrofit2:converter-gson:2.0.2'

```

当然除了gson以外，还提供了以下的选择：

- Gson: com.squareup.retrofit2:converter-gson
- Jackson: com.squareup.retrofit2:converter-jackson
- Moshi: com.squareup.retrofit2:converter-moshi
- Protobuf: com.squareup.retrofit2:converter-protobuf
- Wire: com.squareup.retrofit2:converter-wire
- Simple XML: com.squareup.retrofit2:converter-simplexml
- Scalars (primitives, boxed, and String): com.squareup.retrofit2:converter-scalars

当然也支持自定义，你可以选择自己写转化器完成数据的转化，这个后面将具体介绍。

3. 既然`call.enqueue`是异步的访问数据，那么同步的访问方式为`call.execute`，这一点非常类似okhttp的API，实际上默认情况下内部也是通过`okhttp3.Call`实现。

那么，通过这么一个简单的例子，应该对`retrofit`已经有了一个直观的认识，下面看更多其支持的特性。

## 2.2 动态的url访问`@PATH`

文章开头提过，`retrofit`非常适用于`restful url`的格式，那么例如下面这样的url：

```
//用于访问zhy的信息
http://192.168.1.102:8080/springmvc_users/user/zhy
//用于访问lmj的信息
http://192.168.1.102:8080/springmvc_users/user/lmj12341234
```

即通过不同的username访问不同用户的信息，返回数据为json字符串。

那么可以通过retrofit提供的`@PATH`注解非常方便的完成上述需求。

我们再定义一个方法：

```java
public interface IUserBiz
{
    @GET("{username}")
    Call<User> getUser(@Path("username") String username);
}1234512345
```

可以看到我们定义了一个getUser方法，方法接收一个username参数，并且我们的`@GET`注解中使用`{username}`声明了访问路径，这里你可以把`{username}`当做占位符，而实际运行中会通过`@PATH("username")`所标注的参数进行替换。

那么访问的代码很类似：

```java
//省略了retrofit的构建代码
Call<User> call = userBiz.getUser("zhy");
//Call<User> call = userBiz.getUser("lmj");
call.enqueue(new Callback<User>()
{

    @Override
    public void onResponse(Call<User> call, Response<User> response)
    {
        Log.e(TAG, "getUsePath:" + response.body());
    }

    @Override
    public void onFailure(Call<User> call, Throwable t)
    {

    }
});
```

## 2.3 查询参数的设置`@Query`

看下面的url

```
http://baseurl/users?sortby=username
http://baseurl/users?sortby=id1212
```

即一般的传参，我们可以通过`@Query`注解方便的完成，我们再次在接口中添加一个方法：

```java
public interface IUserBiz
{
    @GET("users")
    Call<List<User>> getUsersBySort(@Query("sortby") String sort);
}
```

访问的代码，其实没什么写的：

```java
//省略retrofit的构建代码
Call<List<User>> call = userBiz.getUsersBySort("username");
//Call<List<User>> call = userBiz.getUsersBySort("id");
//省略call执行相关代码12341234
```

ok，这样我们就完成了参数的指定，当然相同的方式也适用于POST，只需要把注解修改为`@POST`即可。

对了，我能刚才学了`@PATH`，那么会不会有这样尝试的冲动，对于刚才的需求，我们这么写：

```java
 @GET("users?sortby={sortby}")
 Call<List<User>> getUsersBySort(@Path("sortby") String sort);1212
```

乍一看别说好像有点感觉，哈，实际上运行是不支持的~估计是`@ Path`的定位就是用于url的路径而不是参数，对于参数还是选择通过`@Query`来设置。

## 2.4 POST请求体的方式向服务器传入json字符串`@Body`

大家都清楚，我们app很多时候跟服务器通信，会选择直接使用POST方式将json字符串作为请求体发送到服务器，那么我们看看这个需求使用`retrofit`该如何实现。

再次添加一个方法：

```java
public interface IUserBiz
{
 @POST("add")
 Call<List<User>> addUser(@Body User user);
}1234512345
```

提交的代码其实基本都是一致的：

```java
//省略retrofit的构建代码
 Call<List<User>> call = userBiz.addUser(new User(1001, "jj", "123,", "jj123", "jj@qq.com"));
//省略call执行相关代码123123
```

ok，可以看到其实就是使用`@Body`这个注解标识我们的参数对象即可，那么这里需要考虑一个问题，retrofit是如何将user对象转化为字符串呢？下文将详细解释~

下面对应okhttp，还有两种requestBody，一个是`FormBody`，一个是`MultipartBody`，前者以表单的方式传递简单的键值对，后者以POST表单的方式上传文件可以携带参数，`retrofit`也二者也有对应的注解，下面继续~

## 2.5 表单的方式传递键值对`@FormUrlEncoded`

这里我们模拟一个登录的方法，添加一个方法：

```java
public interface IUserBiz
{
    @POST("login")
    @FormUrlEncoded
    Call<User> login(@Field("username") String username, @Field("password") String password);
}
```

访问的代码：

```java
//省略retrofit的构建代码
Call<User> call = userBiz.login("zhy", "123");
//省略call执行相关代码123123
```

ok，看起来也很简单，通过`@POST`指明url，添加`FormUrlEncoded`，然后通过`@Field`添加参数即可。

## 2.6 单文件上传`@Multipart`

下面看一下单文件上传，依然是再次添加个方法：

```java
public interface IUserBiz
{
    @Multipart
    @POST("register")
    Call<User> registerUser(@Part MultipartBody.Part photo, @Part("username") RequestBody username, @Part("password") RequestBody password);
}
```

这里`@MultiPart`的意思就是允许多个`@Part`了，我们这里使用了3个`@Part`，第一个我们准备上传个文件，使用了`MultipartBody.Part`类型，其余两个均为简单的键值对。

使用：

```java
File file = new File(Environment.getExternalStorageDirectory(), "icon.png");
RequestBody photoRequestBody = RequestBody.create(MediaType.parse("image/png"), file);
MultipartBody.Part photo = MultipartBody.Part.createFormData("photos", "icon.png", photoRequestBody);

Call<User> call = userBiz.registerUser(photo, RequestBody.create(null, "abc"), RequestBody.create(null, "123"));1234512345
```

ok，这里感觉略为麻烦。不过还是蛮好理解~~多个`@Part`，每个Part对应一个RequestBody。

这里插个实验过程，其实我最初对于文件，也是尝试的`@Part RequestBody`，因为`@Part("key")`，然后传入一个代表文件的`RequestBody`，我觉得更加容易理解，后来发现试验无法成功，而且查了下issue，给出了一个很奇怪的解决方案，这里可以参考：[retrofit#1063](https://github.com/square/retrofit/issues/1063)。

给出了一个类似如下的方案：

```java
public interface ApiInterface {
        @Multipart
        @POST ("/api/Accounts/editaccount")
        Call<User> editUser (@Header("Authorization") String authorization, @Part("file\"; filename=\"pp.png") RequestBody file , @Part("FirstName") RequestBody fname, @Part("Id") RequestBody id);
    }
```

可以看到对于文件的那个`@Part`value竟然写了这么多奇怪的东西，而且filename竟然硬编码了~~这个不好吧，我上传的文件名竟然不能动态指定。

为了文件名不会被写死，所以给出了最上面的上传单文件的方法，ps:上面这个方案经[测试](http://lib.csdn.net/base/softwaretest)也是可以上传成功的。

恩，这个奇怪方案，为什么这么做可行，下文会给出非常详细的解释。

最后看下多文件上传~

## 2.7 多文件上传`@PartMap`

再添加一个方法~~~

```java
 public interface IUserBiz
 {
     @Multipart
     @POST("register")
      Call<User> registerUser(@PartMap Map<String, RequestBody> params,  @Part("password") RequestBody password);
}
```

这里使用了一个新的注解`@PartMap`，这个注解用于标识一个Map，Map的key为String类型，代表上传的键值对的key(与服务器接受的key对应),value即为RequestBody，有点类似`@Part`的封装版本。

执行的代码：

```java
File file = new File(Environment.getExternalStorageDirectory(), "messenger_01.png");
        RequestBody photo = RequestBody.create(MediaType.parse("image/png", file);
Map<String,RequestBody> photos = new HashMap<>();
photos.put("photos\"; filename=\"icon.png", photo);
photos.put("username",  RequestBody.create(null, "abc"));

Call<User> call = userBiz.registerUser(photos, RequestBody.create(null, "123"));
```

可以看到，可以在Map中put进一个或多个文件，键值对等，当然你也可以分开，单独的键值对也可以使用`@Part`，这里又看到设置文件的时候，相对应的key很奇怪，例如上例`"photos\"; filename=\"icon.png"`,前面的photos就是与服务器对应的key，后面filename是服务器得到的文件名，ok，参数虽然奇怪，但是也可以动态的设置文件名，不太影响使用~~

## 2.8 下载文件

这个其实我觉得直接使用okhttp就好了，使用retrofit去做这个事情真的有点瞎用的感觉~~

增加一个方法：

```java
@GET("download")
Call<ResponseBody> downloadTest();1212
```

调用：

```java
Call<ResponseBody> call = userBiz.downloadTest();
call.enqueue(new Callback<ResponseBody>()
{
    @Override
    public void onResponse(Call<ResponseBody> call, Response<ResponseBody> response)
    {
        InputStream is = response.body().byteStream();
        //save file
    }

    @Override
    public void onFailure(Call<ResponseBody> call, Throwable t)
    {

    }
});
```

可以看到可以返回`ResponseBody`，那么很多事都能干了~~

but，也看出这种方式下载感觉非常鸡肋，并且onReponse回调虽然在UI线程，但是你还是要处理io操作，也就是说你在这里还要另外开线程操作，或者你可以考虑同步的方式下载。

最后还是建议使用okhttp去下载，例如使用[okhttputils](https://github.com/hongyangAndroid/okhttp-utils).

有人可能会问，使用okhttp，和使用retrofit会不会造成新建多个`OkHttpClient`对象呢，其实是可设置的，参考下文。

ok，上面就是一些常用的方法，当然还涉及到一些没有介绍的注解，但是通过上面这么多方法的介绍，再多一二个注解的使用方式，相信大家能够解决。

# 3. 配置OkHttpClient

这个需要简单提一下，很多时候，比如你使用retrofit需要统一的log管理，给每个请求添加统一的header等，这些都应该通过okhttpclient去操作，比如`addInterceptor`

例如：

```java
OkHttpClient client = new OkHttpClient.Builder().addInterceptor(new Interceptor()//log，统一的header等
{
    @Override
    public okhttp3.Response intercept(Chain chain) throws IOException
    {
        return null;
    }
}).build();
```

或许你需要更多的配置，你可以单独写一个OkhttpClient的单例生成类，在这个里面完成你所需的所有的配置，然后将`OkhttpClient`实例通过方法公布出来，设置给retrofit。

设置方式：

```java
Retrofit retrofit = new Retrofit.Builder()
    .callFactory(OkHttpUtils.getClient())
    .build();123123
```

`callFactory`方法接受一个`okhttp3.Call.Factory`对象，`OkHttpClient`即为一个实现类。

# 4. Retrofit源码解析

ok，接下来我们队retrofit的源码做简单的分析，首先我们看retrofit如何为我们的接口实现实例；然后看整体的执行流程；最后再看详细的细节；

## 4.1 Retrofit如何为我们的接口实现实例

通过上文的学习，我们发现使用retrofit需要去定义一个接口，然后可以通过调用`retrofit.create(IUserBiz.class);`方法，得到一个接口的实例，最后通过该实例执行我们的操作，那么retrofit如何实现我们指定接口的实例呢？

其实原理是：动态代理。但是不要被动态代理这几个词吓唬到，[Java](http://lib.csdn.net/base/javase)中已经提供了非常简单的API帮助我们来实现动态代理。

看源码前先看一个例子：

```java
public interface ITest
{
    @GET("/heiheihei")
    public void add(int a, int b);

}
public static void main(String[] args)
{
    ITest iTest = (ITest) Proxy.newProxyInstance(ITest.class.getClassLoader(), new Class<?>[]{ITest.class}, new InvocationHandler()
    {
        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable
        {
            Integer a = (Integer) args[0];
            Integer b = (Integer) args[1];
            System.out.println("方法名：" + method.getName());
            System.out.println("参数：" + a + " , " + b);

            GET get = method.getAnnotation(GET.class);
            System.out.println("注解：" + get.value());
            return null;
        }
    });
    iTest.add(3, 5);
}
```

输出结果为：

```
方法名：add
参数：3 , 5
注解：/heiheihei123123
```

可以看到我们通过`Proxy.newProxyInstance`产生的代理类，当调用接口的任何方法时，都会调用`InvocationHandler#invoke`方法，在这个方法中可以拿到传入的参数，注解等。

试想，retrofit也可以通过同样的方式，在invoke方法里面，拿到所有的参数，注解信息然后就可以去构造`RequestBody`，再去构建`Request`，得到`Call`对象封装后返回。

ok，下面看`retrofit#create`的源码：

```java
public <T> T create(final Class<T> service) {
    return (T) Proxy.newProxyInstance(service.getClassLoader(), new Class<?>[] { service },
        new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object... args) throws Throwable {
       });
  }12345671234567
```

哈，和上面对应。到这里，你应该明白retrofit为我们接口生成实例对象并不神奇，仅仅是使用了`Proxy`这个类的API而已，然后在`invoke`方法里面拿到足够的信息去构建最终返回的Call而已。

哈，其实真正的动态代理一般是有具体的实现类的，只是在这个类调用某个方法的前后去执行一些别的操作，比如开事务，打log等等。当然，本博文并不需要涉及这些详细的内容，如果你希望详细去了解，可以搜索关键字：`Proxy InvocationHandler`。

## 4.2 retrofit整体实现流程

### 4.2.1 Retrofit的构建

这里依然是通过构造者模式进行构建retrofit对象，好在其内部的成员变量比较少，我们直接看build()方法。

```java
public Builder() {
    this(Platform.get());
}

public Retrofit build() {
  if (baseUrl == null) {
    throw new IllegalStateException("Base URL required.");
  }

  okhttp3.Call.Factory callFactory = this.callFactory;
  if (callFactory == null) {
    callFactory = new OkHttpClient();
  }

  Executor callbackExecutor = this.callbackExecutor;
  if (callbackExecutor == null) {
    callbackExecutor = platform.defaultCallbackExecutor();
  }

  // Make a defensive copy of the adapters and add the default Call adapter.
  List<CallAdapter.Factory> adapterFactories = new ArrayList<>(this.adapterFactories);
  adapterFactories.add(platform.defaultCallAdapterFactory(callbackExecutor));

  // Make a defensive copy of the converters.
  List<Converter.Factory> converterFactories = new ArrayList<>(this.converterFactories);

  return new Retrofit(callFactory, baseUrl, converterFactories, adapterFactories,
      callbackExecutor, validateEagerly);
}
```

- baseUrl必须指定，这个是理所当然的；
- 然后可以看到如果不着急设置callFactory，则默认直接`new OkHttpClient()`，可见如果你需要对okhttpclient进行详细的设置，需要构建`OkHttpClient`对象，然后传入；
- 接下来是callbackExecutor，这个想一想大概是用来将回调传递到UI线程了，当然这里设计的比较巧妙，利用platform对象，对平台进行判断，判断主要是利用`Class.forName("")`进行查找，具体代码已经被放到文末，如果是Android平台，会自定义一个`Executor`对象，并且利用`Looper.getMainLooper()`实例化一个handler对象，在`Executor`内部通过`handler.post(runnable)`，ok，整理凭大脑应该能构思出来，暂不贴代码了。
- 接下来是adapterFactories，这个对象主要用于对Call进行转化，基本上不需要我们自己去自定义。
- 最后是converterFactories，该对象用于转化数据，例如将返回的`responseBody`转化为对象等；当然不仅仅是针对返回的数据，还能用于一般备注解的参数的转化例如`@Body`标识的对象做一些操作，后面遇到源码详细再描述。

ok，总体就这几个对象去构造retrofit，还算比较少的~~

### 4.2.2 具体Call构建流程

我们构造完成retrofit，就可以利用retrofit.create方法去构建接口的实例了，上面我们已经分析了这个环节利用了动态代理，而且我们也分析了具体的Call的构建流程在invoke方法中，下面看代码：

```java
public <T> T create(final Class<T> service) {
    Utils.validateServiceInterface(service);
    //...
    return (T) Proxy.newProxyInstance(service.getClassLoader(), new Class<?>[] { service },
        new InvocationHandler() {
           @Override
          public Object invoke(Object proxy, Method method, Object... args){
            //...
            ServiceMethod serviceMethod = loadServiceMethod(method);
            OkHttpCall okHttpCall = new OkHttpCall<>(serviceMethod, args);
            return serviceMethod.callAdapter.adapt(okHttpCall);
          }
        });
}
```

主要也就三行代码，第一行是根据我们的method将其包装成ServiceMethod，第二行是通过ServiceMethod和方法的参数构造`retrofit2.OkHttpCall`对象，第三行是通过`serviceMethod.callAdapter.adapt()`方法，将`OkHttpCall`进行代理包装；

下面一个一个介绍：

- ServiceMethod应该是最复杂的一个类了，包含了将一个method转化为Call的所有的信息。

```java
// Retrofit class
ServiceMethod loadServiceMethod(Method method) {
    ServiceMethod result;
    synchronized (serviceMethodCache) {
      result = serviceMethodCache.get(method);
      if (result == null) {
        result = new ServiceMethod.Builder(this, method).build();
        serviceMethodCache.put(method, result);
      }
    }
    return result;
  }

// ServiceMethod
public ServiceMethod build() {
      callAdapter = createCallAdapter();
      responseType = callAdapter.responseType();
      if (responseType == Response.class || responseType == okhttp3.Response.class) {
        throw methodError("'"
            + Utils.getRawType(responseType).getName()
            + "' is not a valid response body type. Did you mean ResponseBody?");
      }
      responseConverter = createResponseConverter();

      for (Annotation annotation : methodAnnotations) {
        parseMethodAnnotation(annotation);
      }


      int parameterCount = parameterAnnotationsArray.length;
      parameterHandlers = new ParameterHandler<?>[parameterCount];
      for (int p = 0; p < parameterCount; p++) {
        Type parameterType = parameterTypes[p];
        if (Utils.hasUnresolvableType(parameterType)) {
          throw parameterError(p, "Parameter type must not include a type variable or wildcard: %s",
              parameterType);
        }

        Annotation[] parameterAnnotations = parameterAnnotationsArray[p];
        if (parameterAnnotations == null) {
          throw parameterError(p, "No Retrofit annotation found.");
        }

        parameterHandlers[p] = parseParameter(p, parameterType, parameterAnnotations);
      }

      return new ServiceMethod<>(this);
    }
```

直接看build方法，首先拿到这个callAdapter最终拿到的是我们在构建retrofit里面时adapterFactories时添加的，即为：`new ExecutorCallbackCall<>(callbackExecutor, call)`，该`ExecutorCallbackCall`唯一做的事情就是将原本call的回调转发至UI线程。

接下来通过`callAdapter.responseType()`返回的是我们方法的实际类型，例如:`Call<User>`,则返回`User`类型，然后对该类型进行判断。

接下来是`createResponseConverter`拿到responseConverter对象，其当然也是根据我们构建retrofit时,`addConverterFactory`添加的ConverterFactory对象来寻找一个合适的返回，寻找的依据主要看该converter能否处理你编写方法的返回值类型，默认实现为`BuiltInConverters`，仅仅支持返回值的实际类型为`ResponseBody`和`Void`，也就说明了默认情况下，是不支持`Call<User>`这类类型的。

接下来就是对注解进行解析了，主要是对方法上的注解进行解析，那么可以拿到httpMethod以及初步的url（包含占位符）。

后面是对方法中参数中的注解进行解析，这一步会拿到很多的`ParameterHandler`对象，该对象在`toRequest()`构造Request的时候调用其apply方法。

ok，这里我们并没有去一行一行查看代码，其实意义也不太大，只要知道ServiceMethod主要用于将我们`接口中的方法`转化为一个`Request对象`，于是根据我们的接口返回值确定了responseConverter,解析我们方法上的注解拿到初步的url,解析我们参数上的注解拿到构建RequestBody所需的各种信息，最终调用toRequest的方法完成Request的构建。

- 接下来看OkHttpCall的构建，构造函数仅仅是简单的赋值

```java
OkHttpCall(ServiceMethod<T> serviceMethod, Object[] args) {
    this.serviceMethod = serviceMethod;
    this.args = args;
  }12341234
```

- 最后一步是`serviceMethod.callAdapter.adapt(okHttpCall)`

我们已经确定这个callAdapter是`ExecutorCallAdapterFactory.get()`对应代码为：

```java
final class ExecutorCallAdapterFactory extends CallAdapter.Factory {
  final Executor callbackExecutor;

  ExecutorCallAdapterFactory(Executor callbackExecutor) {
    this.callbackExecutor = callbackExecutor;
  }

  @Override
  public CallAdapter<Call<?>> get(Type returnType, Annotation[] annotations, Retrofit retrofit) {
    if (getRawType(returnType) != Call.class) {
      return null;
    }
    final Type responseType = Utils.getCallResponseType(returnType);
    return new CallAdapter<Call<?>>() {
      @Override public Type responseType() {
        return responseType;
      }

      @Override public <R> Call<R> adapt(Call<R> call) {
        return new ExecutorCallbackCall<>(callbackExecutor, call);
      }
    };
  }
```

可以看到`adapt`返回的是`ExecutorCallbackCall`对象，继续往下看：

```java
static final class ExecutorCallbackCall<T> implements Call<T> {
    final Executor callbackExecutor;
    final Call<T> delegate;

    ExecutorCallbackCall(Executor callbackExecutor, Call<T> delegate) {
      this.callbackExecutor = callbackExecutor;
      this.delegate = delegate;
    }

    @Override public void enqueue(final Callback<T> callback) {
      if (callback == null) throw new NullPointerException("callback == null");

      delegate.enqueue(new Callback<T>() {
        @Override public void onResponse(Call<T> call, final Response<T> response) {
          callbackExecutor.execute(new Runnable() {
            @Override public void run() {
              if (delegate.isCanceled()) {
                // Emulate OkHttp's behavior of throwing/delivering an IOException on cancellation.
                callback.onFailure(ExecutorCallbackCall.this, new IOException("Canceled"));
              } else {
                callback.onResponse(ExecutorCallbackCall.this, response);
              }
            }
          });
        }

        @Override public void onFailure(Call<T> call, final Throwable t) {
          callbackExecutor.execute(new Runnable() {
            @Override public void run() {
              callback.onFailure(ExecutorCallbackCall.this, t);
            }
          });
        }
      });
    }
    @Override public Response<T> execute() throws IOException {
      return delegate.execute();
    }
  }
```

可以看出ExecutorCallbackCall仅仅是对Call对象进行封装，类似装饰者模式，只不过将其执行时的回调通过callbackExecutor进行回调到UI线程中去了。

### 4.2.3 执行Call

在4.2.2我们已经拿到了经过封装的`ExecutorCallbackCall`类型的call对象，实际上就是我们实际在写代码时拿到的call对象，那么我们一般会执行`enqueue`方法，看看源码是怎么做的

首先是`ExecutorCallbackCall.enqueue`方法，代码在4.2.2，可以看到除了将onResponse和onFailure回调到UI线程，主要的操作还是delegate完成的，这个delegate实际上就是OkHttpCall对象，我们看它的enqueue方法

```java
 @Override
public void enqueue(final Callback<T> callback)
{
    okhttp3.Call call;
    Throwable failure;

    synchronized (this)
    {
        if (executed) throw new IllegalStateException("Already executed.");
        executed = true;

        try
        {
            call = rawCall = createRawCall();
        } catch (Throwable t)
        {
            failure = creationFailure = t;
        }
    }

    if (failure != null)
    {
        callback.onFailure(this, failure);
        return;
    }

    if (canceled)
    {
        call.cancel();
    }

    call.enqueue(new okhttp3.Callback()
    {
        @Override
        public void onResponse(okhttp3.Call call, okhttp3.Response rawResponse)
                throws IOException
        {
            Response<T> response;
            try
            {
                response = parseResponse(rawResponse);
            } catch (Throwable e)
            {
                callFailure(e);
                return;
            }
            callSuccess(response);
        }

        @Override
        public void onFailure(okhttp3.Call call, IOException e)
        {
            try
            {
                callback.onFailure(OkHttpCall.this, e);
            } catch (Throwable t)
            {
                t.printStackTrace();
            }
        }

        private void callFailure(Throwable e)
        {
            try
            {
                callback.onFailure(OkHttpCall.this, e);
            } catch (Throwable t)
            {
                t.printStackTrace();
            }
        }

        private void callSuccess(Response<T> response)
        {
            try
            {
                callback.onResponse(OkHttpCall.this, response);
            } catch (Throwable t)
            {
                t.printStackTrace();
            }
        }
    });
}
```

没有任何神奇的地方，内部实际上就是okhttp的Call对象，也是调用`okhttp3.Call.enqueue`方法。

中间对于`okhttp3.Call`的创建代码为：

```java
private okhttp3.Call createRawCall() throws IOException
{
    Request request = serviceMethod.toRequest(args);
    okhttp3.Call call = serviceMethod.callFactory.newCall(request);
    if (call == null)
    {
        throw new NullPointerException("Call.Factory returned null.");
    }
    return call;
}
```

可以看到，通过`serviceMethod.toRequest`完成对request的构建，通过request去构造call对象，然后返回.

中间还涉及一个`parseResponse`方法，如果顺利的话，执行的代码如下：

```java
Response<T> parseResponse(okhttp3.Response rawResponse) throws IOException
{
    ResponseBody rawBody = rawResponse.body();
    ExceptionCatchingRequestBody catchingBody = new ExceptionCatchingRequestBody(rawBody);

    T body = serviceMethod.toResponse(catchingBody);
    return Response.success(body, rawResponse);
```

通过serviceMethod对ResponseBody进行转化，然后返回，转化实际上就是通过`responseConverter`的convert方法。

```java
// ServiceMethod
 T toResponse(ResponseBody body) throws IOException {
    return responseConverter.convert(body);
  }
```

ok，关于`responseConverter`后面还会细说，不用担心。

到这里，我们整个源码的流程分析就差不多了，目的就掌握一个大体的原理和执行流程，了解下几个核心的类。

那么总结一下：

- 首先构造retrofit，几个核心的参数呢，主要就是baseurl,callFactory(默认okhttpclient),converterFactories,adapterFactories,excallbackExecutor。
- 然后通过create方法拿到接口的实现类，这里利用Java的`Proxy`类完成动态代理的相关代理
- 在invoke方法内部，拿到我们所声明的注解以及实参等，构造ServiceMethod，ServiceMethod中解析了大量的信息，最痛可以通过`toRequest`构造出`okhttp3.Request`对象。有了`okhttp3.Request`对象就可以很自然的构建出`okhttp3.call`，最后calladapter对Call进行装饰返回。
- 拿到Call就可以执行enqueue或者execute方法了

ok,了解这么多足以。

下面呢，有几个地方需要注意，一方面是一些特殊的细节；另一方面就是`Converter`。

# 5. retrofit中的各类细节

## 5.1 上传文件中使用的奇怪value值

第一个问题涉及到文件上传，还记得我们在单文件上传那里所说的吗？有种类似于hack的写法，上传文件是这么做的？

```java
public interface ApiInterface {
        @Multipart
        @POST ("/api/Accounts/editaccount")
        Call<User> editUser (@Part("file_key\"; filename=\"pp.png"),@Part("username") String username);
    }
```

首先我们一点明确，因为这里使用了`@ Multipart`，那么我们认为`@Part`应当支持普通的key-value，以及文件。

对于普通的key-value是没问题的，只需要这样`@Part("username") String username`。

那么对于文件，为什么需要这样呢？`@Part("file_key\"; filename=\"pp.png")`

这个value设置的值不用看就会觉得特别奇怪，然而却可以正常执行，原因是什么呢？

原因是这样的：

当上传key-value的时候，实际上对应这样的代码：

```java
builder.addPart(Headers.of("Content-Disposition", "form-data; name=\"" + key + "\""),
                        RequestBody.create(null, params.get(key)));1212
```

也就是说，我们的`@Part`转化为了

```java
Headers.of("Content-Disposition", "form-data; name=\"" + key + "\"")11
```

这么一看，很随意，只要把key放进去就可以了。

但是，retrofit2并没有对文件做特殊处理，文件的对应的字符串应该是这样的

```java
 Headers.of("Content-Disposition", "form-data; name="filekey";filename="filename.png");11
```

与键值对对应的字符串相比，多了个`;filename="filename.png`，就因为retrofit没有做特殊处理，所以你现在看这些hack的做法

```java
@Part("file_key\"; filename=\"pp.png")
拼接：==>
Content-Disposition", "form-data; name=\"" + key + "\"
结果：==>
Content-Disposition", "form-data; name=file_key\"; filename=\"pp.png\"1234512345
```

ok，到这里我相信你已经理解了，为什么要这么做，而且为什么这么做可以成功！

恩，值得一提的事，因为这种方式文件名写死了，我们上文使用的的是`@Part MultipartBody.Part file`,可以满足文件名动态设置，这个方式貌似也是2.0.1的时候支持的。

上述相关的源码：

```java
// ServiceMethod
if (annotation instanceof Part) {
    if (!isMultipart) {
      throw parameterError(p, "@Part parameters can only be used with multipart encoding.");
    }
    Part part = (Part) annotation;
    gotPart = true;

    String partName = part.value();

    Headers headers =
          Headers.of("Content-Disposition", "form-data; name=\"" + partName + "\"",
              "Content-Transfer-Encoding", part.encoding());
}
```

可以看到呢，并没有对文件做特殊处理，估计下个版本说不定`@Part`会多个`isFile=true|false`属性，甚至修改对应形参，然后在这里做简单的处理。

ok，最后来到关键的`ConverterFactory`了~

# 6. 自定义Converter.Factory

## 6.1 responseBodyConverter

关于`Converter.Factory`，肯定是通过`addConverterFactory`设置的

```java
Retrofit retrofit = new Retrofit.Builder()
    .addConverterFactory(GsonConverterFactory.create())
        .build();123123
```

该方法接受的是一个`Converter.Factory factory`对象

该对象呢，是一个抽象类，内部包含3个方法：

```java
abstract class Factory {

    public Converter<ResponseBody, ?> responseBodyConverter(Type type, Annotation[] annotations,
        Retrofit retrofit) {
      return null;
    }


    public Converter<?, RequestBody> requestBodyConverter(Type type,
        Annotation[] parameterAnnotations, Annotation[] methodAnnotations, Retrofit retrofit) {
      return null;
    }


    public Converter<?, String> stringConverter(Type type, Annotation[] annotations,
        Retrofit retrofit) {
      return null;
    }
  }
```

可以看到呢，3个方法都是空方法而不是抽象的方法，也就表明了我们可以选择去实现其中的1个或多个方法，一般只需要关注`requestBodyConverter`和`responseBodyConverter`就可以了。

ok，我们先看如何自定义，最后再看`GsonConverterFactory.create`的源码。

先来个简单的，实现`responseBodyConverter`方法，看这个名字很好理解，就是将responseBody进行转化就可以了。

ok，这里呢，我们先看一下上述中我们使用的接口：

```java
package com.zhy.retrofittest.userBiz;

public interface IUserBiz
{
    @GET("users")
    Call<List<User>> getUsers();

    @POST("users")
    Call<List<User>> getUsersBySort(@Query("sort") String sort);

    @GET("{username}")
    Call<User> getUser(@Path("username") String username);

    @POST("add")
    Call<List<User>> addUser(@Body User user);

    @POST("login")
    @FormUrlEncoded
    Call<User> login(@Field("username") String username, @Field("password") String password);

    @Multipart
    @POST("register")
    Call<User> registerUser(@Part("photos") RequestBody photos, @Part("username") RequestBody username, @Part("password") RequestBody password);

    @Multipart
    @POST("register")
    Call<User> registerUser(@PartMap Map<String, RequestBody> params,  @Part("password") RequestBody password);

    @GET("download")
    Call<ResponseBody> downloadTest();

}
```

不知不觉，方法还蛮多的，假设哈，我们这里去掉retrofit构造时的`GsonConverterFactory.create`，自己实现一个`Converter.Factory`来做数据的转化工作。

首先我们解决`responseBodyConverter`，那么代码很简单，我们可以这么写：

```java
public class UserConverterFactory extends Converter.Factory
{
    @Override
    public Converter<ResponseBody, ?> responseBodyConverter(Type type, Annotation[] annotations, Retrofit retrofit)
    {
        //根据type判断是否是自己能处理的类型，不能的话，return null ,交给后面的Converter.Factory
        return new UserConverter(type);
    }

}

public class UserResponseConverter<T> implements Converter<ResponseBody, T>
{
    private Type type;
    Gson gson = new Gson();

    public UserResponseConverter(Type type)
    {
        this.type = type;
    }

    @Override
    public T convert(ResponseBody responseBody) throws IOException
    {
        String result = responseBody.string();
        T users = gson.fromJson(result, type);
        return users;
    }
}
```

使用的时候呢，可以

```java
 Retrofit retrofit = new Retrofit.Builder()
.callFactory(new OkHttpClient())               .baseUrl("http://example/springmvc_users/user/")
//.addConverterFactory(GsonConverterFactory.create())
.addConverterFactory(new UserConverterFactory())
            .build();1234512345
```

ok，这样的话，就可以完成我们的`ReponseBody`到`List<User>`或者`User`的转化了。

可以看出，我们这里用的依然是Gson，那么有些同学肯定不希望使用Gson就能实现，如果不使用Gson的话，一般需要针对具体的返回类型，比如我们针对返回`List<User>`或者`User`

你可以这么写：

```java
package com.zhy.retrofittest.converter;
/**
 * Created by zhy on 16/4/30.
 */
public class UserResponseConverter<T> implements Converter<ResponseBody, T>
{
    private Type type;
    Gson gson = new Gson();

    public UserResponseConverter(Type type)
    {
        this.type = type;
    }

    @Override
    public T convert(ResponseBody responseBody) throws IOException
    {
        String result = responseBody.string();

        if (result.startsWith("["))
        {
            return (T) parseUsers(result);
        } else
        {
            return (T) parseUser(result);
        }
    }

    private User parseUser(String result)
    {
        JSONObject jsonObject = null;
        try
        {
            jsonObject = new JSONObject(result);
            User u = new User();
            u.setUsername(jsonObject.getString("username"));
            return u;
        } catch (JSONException e)
        {
            e.printStackTrace();
        }
        return null;
    }

    private List<User> parseUsers(String result)
    {
        List<User> users = new ArrayList<>();
        try
        {
            JSONArray jsonArray = new JSONArray(result);
            User u = null;
            for (int i = 0; i < jsonArray.length(); i++)
            {
                JSONObject jsonObject = jsonArray.getJSONObject(i);
                u = new User();
                u.setUsername(jsonObject.getString("username"));
                users.add(u);
            }
        } catch (JSONException e)
        {
            e.printStackTrace();
        }
        return users;
    }
}
```

这里简单读取了一个属性，大家肯定能看懂，这样就能满足返回值是`Call<List<User>>`或者`Call<User>`.

这里郑重提醒：如果你针对特定的类型去写`Converter`，一定要在`UserConverterFactory#responseBodyConverter`中对类型进行检查，发现不能处理的类型`return null`，这样的话，可以交给后面的`Converter.Factory`处理，比如本例我们可以按照下列方式检查:

```java
public class UserConverterFactory extends Converter.Factory
{
    @Override
    public Converter<ResponseBody, ?> responseBodyConverter(Type type, Annotation[] annotations, Retrofit retrofit)
    {
        //根据type判断是否是自己能处理的类型，不能的话，return null ,交给后面的Converter.Factory
        if (type == User.class)//支持返回值是User
        {
            return new UserResponseConverter(type);
        }

        if (type instanceof ParameterizedType)//支持返回值是List<User>
        {
            Type rawType = ((ParameterizedType) type).getRawType();
            Type actualType = ((ParameterizedType) type).getActualTypeArguments()[0];
            if (rawType == List.class && actualType == User.class)
            {
                return new UserResponseConverter(type);
            }
        }
        return null;
    }

}
```

好了，到这呢`responseBodyConverter`方法告一段落了，谨记就是将reponseBody->返回值返回中的实际类型，例如`Call<User>中的User`;还有对于该converter不能处理的类型一定要返回null。

## 6.2 requestBodyConverter

ok，上面接口一大串方法呢，使用了我们的Converter之后，有个方法我们现在还是不支持的。

```java
@POST("add")
Call<List<User>> addUser(@Body User user);1212
```

ok，这个`@Body`需要用到这个方法，叫做`requestBodyConverter`，根据参数转化为`RequestBody`，下面看下我们如何提供支持。

```java
public class UserRequestBodyConverter<T> implements Converter<T, RequestBody>
{
    private Gson mGson = new Gson();
    @Override
    public RequestBody convert(T value) throws IOException
    {
        String string = mGson.toJson(value);
        return RequestBody.create(MediaType.parse("application/json; charset=UTF-8"),string);
    }
}
```

然后在UserConverterFactory中复写requestBodyConverter方法，返回即可：

```java
public class UserConverterFactory extends Converter.Factory
{

    @Override
    public Converter<?, RequestBody> requestBodyConverter(Type type, Annotation[] parameterAnnotations, Annotation[] methodAnnotations, Retrofit retrofit)
    {
        return new UserRequestBodyConverter<>();
    }
}
```

这里偷了个懒，使用Gson将对象转化为json字符串了，如果你不喜欢使用框架，你可以选择拼接字符串，或者反射写一个支持任何对象的，反正就是对象->json字符串的转化。最后构造一个RequestBody返回即可。

ok,到这里，我相信如果你看的细致，自定义`Converter.Factory`是干嘛的，但是我还是要总结下：

- responseBodyConverter 主要是对应`@Body`注解，完成ResponseBody到实际的返回类型的转化，这个类型对应`Call<XXX>`里面的泛型XXX，其实`@Part`等注解也会需要responseBodyConverter，只不过我们的参数类型都是`RequestBody`，由默认的converter处理了。
- requestBodyConverter 完成对象到RequestBody的构造。
- 一定要注意，检查type如果不是自己能处理的类型，记得return null （因为可以添加多个，你不能处理return null ,还会去遍历后面的converter）.

## 7. 值得学习的API

其实一般情况下看源码呢，可以让我们更好的去使用这个库，当然在看的过程中如果发现了一些比较好的处理方式呢，是非常值得记录的。如果每次看别人的源码都能吸取一定的精华，比你单纯的去理解会好很多，因为你的记忆力再好，源码解析你也是会忘的，而你记录下来并能够使用的优越的代码，可能用久了就成为你的代码了。

我举个例子：比如retrofit2中判断当前运行的环境代码如下，如果下次你有这样的需求，你也可以这么写，甚至源码中根据不同的运行环境还提供了不同的`Executor`都很值得记录：

```java
class Platform {
  private static final Platform PLATFORM = findPlatform();

  static Platform get() {
    return PLATFORM;
  }

  private static Platform findPlatform() {
    try {
      Class.forName("android.os.Build");
      if (Build.VERSION.SDK_INT != 0) {
        return new Android();
      }
    } catch (ClassNotFoundException ignored) {
    }
    try {
      Class.forName("java.util.Optional");
      return new Java8();
    } catch (ClassNotFoundException ignored) {
    }
    try {
      Class.forName("org.robovm.apple.foundation.NSObject");
      return new IOS();
    } catch (ClassNotFoundException ignored) {
    }
    return new Platform();
  }
```

ok，文章结束，最后打个广告，个人[微信](http://lib.csdn.net/base/wechat)公众号目前关注大概9000人左右，无奈个人博文更新频率无法做到很好的为这么多人服务，所以本公众号欢迎大家投稿，如果你希望你的文章可以被更多人看到，直接将md、doc等格式的文章发我邮箱即可（623565791@qq.com），也可以加我好友，需要注明（投稿），谢谢。

文章要求：原创+你觉得通过文章你能学到一些有价值的东西。

福利：您的文章还可以发到csdn，简书等其他平台，但不能投稿至其他微信公众号了；更多人能够发现你的文章（好文我会帮你发送至我的各大群）+您的署名+您的原文链接，如果后期有打赏收益均归投稿人所有。
