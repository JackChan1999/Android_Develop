在上一篇文章中，我们了解了Volley到底是什么，以及它的基本用法。本篇文章中我们即将学习关于Volley更加高级的用法，如何你还没有看过我的上一篇文章的话，建议先去阅读[**Android Volley完全解析(一)，初识Volley的基本用法**](http://blog.csdn.net/guolin_blog/article/details/17482095)。

在上篇文章中有提到过，Volley是将AsyncHttpClient和Universal-Image-Loader的优点集成于一身的一个框架。我们都知道，Universal-Image-Loader具备非常强大的加载网络图片的功能，而使用Volley，我们也可以实现基本类似的效果，并且在性能上也豪不逊色于Universal-Image-Loader，下面我们就来具体学习一下吧。

## 1. ImageRequest的用法

前面我们已经学习过了StringRequest和JsonRequest的用法，并且总结出了它们的用法都是非常类似的，基本就是进行以下三步操作即可：

1. 创建一个RequestQueue对象。

2. 创建一个Request对象。

3. 将Request对象添加到RequestQueue里面。

其中，StringRequest和JsonRequest都是继承自Request的，所以它们的用法才会如此类似。那么不用多说，今天我们要学习的ImageRequest，相信你从名字上就已经猜出来了，它也是继承自Request的，因此它的用法也是基本相同的，首先需要获取到一个RequestQueue对象，可以调用如下方法获取到：

```java
RequestQueue mQueue = Volley.newRequestQueue(context);  
```

接下来自然要去new出一个ImageRequest对象了，代码如下所示：

```java
ImageRequest imageRequest = new ImageRequest(  
        "http://developer.android.com/images/home/aw_dac.png",  
        new Response.Listener<Bitmap>() {  
            @Override  
            public void onResponse(Bitmap response) {  
                imageView.setImageBitmap(response);  
            }  
        }, 0, 0, Config.RGB_565, new Response.ErrorListener() {  
            @Override  
             public void onErrorResponse(VolleyError error) {  
                 imageView.setImageResource(R.drawable.default_image);  
             }  
         });  
```
可以看到，ImageRequest的构造函数接收六个参数，第一个参数就是图片的URL地址，这个没什么需要解释的。第二个参数是图片请求成功的回调，这里我们把返回的Bitmap参数设置到ImageView中。第三第四个参数分别用于指定允许图片最大的宽度和高度，如果指定的网络图片的宽度或高度大于这里的最大值，则会对图片进行压缩，指定成0的话就表示不管图片有多大，都不会进行压缩。第五个参数用于指定图片的颜色属性，Bitmap.Config下的几个常量都可以在这里使用，其中ARGB_8888可以展示最好的颜色属性，每个图片像素占据4个字节的大小，而RGB_565则表示每个图片像素占据2个字节大小。第六个参数是图片请求失败的回调，这里我们当请求失败时在ImageView中显示一张默认图片。

最后将这个ImageRequest对象添加到RequestQueue里就可以了，如下所示：

```java
mQueue.add(imageRequest);  
```

现在如果运行一下程序，并尝试发出这样一条网络请求，很快就能看到网络上的图片在ImageView中显示出来了，如下图所示：

![img](http://img.blog.csdn.net/20140413205455484?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZ3VvbGluX2Jsb2c=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 2. ImageLoader的用法

如果你觉得ImageRequest已经非常好用了，那我只能说你太容易满足了 ^\_^。实际上，Volley在请求网络图片方面可以做到的还远远不止这些，而ImageLoader就是一个很好的例子。ImageLoader也可以用于加载网络上的图片，并且它的内部也是使用ImageRequest来实现的，不过ImageLoader明显要比ImageRequest更加高效，因为它不仅可以帮我们对图片进行缓存，还可以过滤掉重复的链接，避免重复发送请求。

由于ImageLoader已经不是继承自Request的了，所以它的用法也和我们之前学到的内容有所不同，总结起来大致可以分为以下四步：

1. 创建一个RequestQueue对象。

2. 创建一个ImageLoader对象。

3. 获取一个ImageListener对象。

4. 调用ImageLoader的get()方法加载网络上的图片。

下面我们就来按照这个步骤，学习一下ImageLoader的用法吧。首先第一步的创建RequestQueue对象我们已经写过很多遍了，相信已经不用再重复介绍了，那么就从第二步开始学习吧，新建一个ImageLoader对象，代码如下所示：

```java
ImageLoader imageLoader = new ImageLoader(mQueue, new ImageCache() {  
    @Override  
    public void putBitmap(String url, Bitmap bitmap) {  
    }  

    @Override  
    public Bitmap getBitmap(String url) {  
        return null;  
    }  
 });  
```

可以看到，ImageLoader的构造函数接收两个参数，第一个参数就是RequestQueue对象，第二个参数是一个ImageCache对象，这里我们先new出一个空的ImageCache的实现即可。

接下来需要获取一个ImageListener对象，代码如下所示：

```java
ImageListener listener = ImageLoader.getImageListener(imageView,  
        R.drawable.default_image, R.drawable.failed_image);  
```
我们通过调用ImageLoader的getImageListener()方法能够获取到一个ImageListener对象，getImageListener()方法接收三个参数，第一个参数指定用于显示图片的ImageView控件，第二个参数指定加载图片的过程中显示的图片，第三个参数指定加载图片失败的情况下显示的图片。

最后，调用ImageLoader的get()方法来加载图片，代码如下所示：

```java
imageLoader.get("http://img.my.csdn.net/uploads/201404/13/1397393290_5765.jpeg", listener);  
```

get()方法接收两个参数，第一个参数就是图片的URL地址，第二个参数则是刚刚获取到的ImageListener对象。当然，如果你想对图片的大小进行限制，也可以使用get()方法的重载，指定图片允许的最大宽度和高度，如下所示：

```java
imageLoader.get("http://img.my.csdn.net/uploads/201404/13/1397393290_5765.jpeg",  
                listener, 200, 200);  
```

现在运行一下程序并开始加载图片，你将看到ImageView中会先显示一张默认的图片，等到网络上的图片加载完成后，ImageView则会自动显示该图，效果如下图所示。

![img](http://img.blog.csdn.net/20140413210233046)

虽然现在我们已经掌握了ImageLoader的用法，但是刚才介绍的ImageLoader的优点却还没有使用到。为什么呢？因为这里创建的ImageCache对象是一个空的实现，完全没能起到图片缓存的作用。其实写一个ImageCache也非常简单，但是如果想要写一个性能非常好的ImageCache，最好就要借助[Android](http://lib.csdn.net/base/android)提供的LruCache功能了，如果你对LruCache还不了解，可以参考我之前的一篇博客[**Android高效加载大图、多图解决方案，有效避免程序OOM**](http://blog.csdn.net/guolin_blog/article/details/9316683)。

这里我们新建一个BitmapCache并实现了ImageCache接口，如下所示：

```java
public class BitmapCache implements ImageCache {  

    private LruCache<String, Bitmap> mCache;  

    public BitmapCache() {  
        int maxSize = 10 * 1024 * 1024;  
        mCache = new LruCache<String, Bitmap>(maxSize) {  
            @Override  
            protected int sizeOf(String key, Bitmap bitmap) {  
                 return bitmap.getRowBytes() * bitmap.getHeight();  
             }  
         };  
     }  

     @Override  
     public Bitmap getBitmap(String url) {  
         return mCache.get(url);  
     }  

     @Override  
     public void putBitmap(String url, Bitmap bitmap) {  
         mCache.put(url, bitmap);  
     }  

 }  
```
可以看到，这里我们将缓存图片的大小设置为10M。接着修改创建ImageLoader实例的代码，第二个参数传入BitmapCache的实例，如下所示：

```java
ImageLoader imageLoader = new ImageLoader(mQueue, new BitmapCache());  
```
这样我们就把ImageLoader的功能优势充分利用起来了。

## 3. NetworkImageView的用法

除了以上两种方式之外，Volley还提供了第三种方式来加载网络图片，即使用NetworkImageView。不同于以上两种方式，NetworkImageView是一个自定义控制，它是继承自ImageView的，具备ImageView控件的所有功能，并且在原生的基础之上加入了加载网络图片的功能。NetworkImageView控件的用法要比前两种方式更加简单，大致可以分为以下五步：

1. 创建一个RequestQueue对象。

2. 创建一个ImageLoader对象。

3. 在布局文件中添加一个NetworkImageView控件。

4. 在代码中获取该控件的实例。

5. 设置要加载的图片地址。

其中，第一第二步和ImageLoader的用法是完全一样的，因此这里我们就从第三步开始学习了。首先修改布局文件中的代码，在里面加入NetworkImageView控件，如下所示：

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  
    android:layout_width="fill_parent"  
    android:layout_height="fill_parent"  
    android:orientation="vertical" >  

    <Button  
        android:id="@+id/button"  
        android:layout_width="wrap_content"  
        android:layout_height="wrap_content"  
         android:text="Send Request" />  
       
     <com.android.volley.toolbox.NetworkImageView   
         android:id="@+id/network_image_view"  
         android:layout_width="200dp"  
         android:layout_height="200dp"  
         android:layout_gravity="center_horizontal"  
         />  

 </LinearLayout>  
```

接着在Activity获取到这个控件的实例，这就非常简单了，代码如下所示：

```java
networkImageView = (NetworkImageView) findViewById(R.id.network_image_view);  
```

得到了NetworkImageView控件的实例之后，我们可以调用它的setDefaultImageResId()方法、setErrorImageResId()方法和setImageUrl()方法来分别设置加载中显示的图片，加载失败时显示的图片，以及目标图片的URL地址，如下所示：

```java
networkImageView.setDefaultImageResId(R.drawable.default_image);  
networkImageView.setErrorImageResId(R.drawable.failed_image);  
networkImageView.setImageUrl("http://img.my.csdn.net/uploads/201404/13/1397393290_5765.jpeg", imageLoader);  
```

其中，setImageUrl()方法接收两个参数，第一个参数用于指定图片的URL地址，第二个参数则是前面创建好的ImageLoader对象。

好了，就是这么简单，现在重新运行一下程序，你将看到和使用ImageLoader来加载图片一模一样的效果，这里我就不再截图了。

这时有的朋友可能就会问了，使用ImageRequest和ImageLoader这两种方式来加载网络图片，都可以传入一个最大宽度和高度的参数来对图片进行压缩，而NetworkImageView中则完全没有提供设置最大宽度和高度的方法，那么是不是使用NetworkImageView来加载的图片都不会进行压缩呢？

其实并不是这样的，NetworkImageView并不需要提供任何设置最大宽高的方法也能够对加载的图片进行压缩。这是由于NetworkImageView是一个控件，在加载图片的时候它会自动获取自身的宽高，然后对比网络图片的宽度，再决定是否需要对图片进行压缩。也就是说，压缩过程是在内部完全自动化的，并不需要我们关心，NetworkImageView会始终呈现给我们一张大小刚刚好的网络图片，不会多占用任何一点内存，这也是NetworkImageView最简单好用的一点吧。

当然了，如果你不想对图片进行压缩的话，其实也很简单，只需要在布局文件中把NetworkImageView的layout_width和layout_height都设置成wrap_content就可以了，这样NetworkImageView就会将该图片的原始大小展示出来，不会进行任何压缩。

这样我们就把使用Volley来加载网络图片的用法都学习完了，今天的讲解也就到此为止，下一篇文章中我会带大家继续探究Volley的更多功能。感兴趣的朋友请继续阅读[**Android Volley完全解析(三)，定制自己的Request**](http://blog.csdn.net/guolin_blog/article/details/17612763)。
