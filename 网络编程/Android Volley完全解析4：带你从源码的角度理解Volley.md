经过前三篇文章的学习，Volley的用法我们已经掌握的差不多了，但是对于Volley的工作原理，恐怕有很多朋友还不是很清楚。因此，本篇文章中我们就来一起阅读一下Volley的源码，将它的工作流程整体地梳理一遍。同时，这也是Volley系列的最后一篇文章了。

其实，Volley的官方文档中本身就附有了一张Volley的工作流程图，如下图所示。

![img](http://img.blog.csdn.net/20140511114837375?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZ3VvbGluX2Jsb2c=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

多数朋友突然看到一张这样的图，应该会和我一样，感觉一头雾水吧？没错，目前我们对Volley背后的工作原理还没有一个概念性的理解，直接就来看这张图自然会有些吃力。不过没关系，下面我们就去分析一下Volley的源码，之后再重新来看这张图就会好理解多了。

说起分析源码，那么应该从哪儿开始看起呢？这就要回顾一下Volley的用法了，还记得吗，使用Volley的第一步，首先要调用Volley.newRequestQueue(context)方法来获取一个RequestQueue对象，那么我们自然要从这个方法开始看起了，代码如下所示：

```java
public static RequestQueue newRequestQueue(Context context) {  
    return newRequestQueue(context, null);  
}  
```

这个方法仅仅只有一行代码，只是调用了

newRequestQueue()的方法重载，并给第二个参数传入null。那我们看下带有两个参数的newRequestQueue()方法中的代码，如下所示：

```java
public static RequestQueue newRequestQueue(Context context, HttpStack stack) {
    File cacheDir = new File(context.getCacheDir(), DEFAULT_CACHE_DIR);
    String userAgent = "volley/0";
    try {
        String packageName = context.getPackageName();
        PackageInfo info = context.getPackageManager().getPackageInfo(packageName, 0);
        userAgent = packageName + "/" + info.versionCode;
    } catch (NameNotFoundException e) {
    }  
if (stack == null) {  
    if (Build.VERSION.SDK_INT >= 9) {  
        stack = new HurlStack();  
    } else {  
        stack = new HttpClientStack(AndroidHttpClient.newInstance(userAgent));  
    }  
}  
Network network = new BasicNetwork(stack);  
RequestQueue queue = new RequestQueue(new DiskBasedCache(cacheDir), network);  
queue.start();  
return queue;
}
```

可以看到，这里在第10行判断如果stack是等于null的，则去创建一个HttpStack对象，这里会判断如果手机系统版本号是大于9的，则创建一个HurlStack的实例，否则就创建一个HttpClientStack的实例。实际上

HurlStack的内部就是使用HttpURLConnection进行网络通讯的，而HttpClientStack的内部则是使用HttpClient进行网络通讯的，这里为什么这样选择呢？可以参考我之前翻译的一篇文章**Android访问网络，使用HttpURLConnection还是HttpClient？**

创建好了HttpStack之后，接下来又创建了一个Network对象，它是用于根据传入的HttpStack对象来处理网络请求的，紧接着new出一个RequestQueue对象，并调用它的start()方法进行启动，然后将RequestQueue返回，这样newRequestQueue()的方法就执行结束了。

那么RequestQueue的start()方法内部到底执行了什么东西呢？我们跟进去瞧一瞧：

```java
public void start() {  
    stop();  // Make sure any currently running dispatchers are stopped.  
    // Create the cache dispatcher and start it.  
    mCacheDispatcher = new CacheDispatcher(mCacheQueue, mNetworkQueue, mCache, mDelivery);  
    mCacheDispatcher.start();  
    // Create network dispatchers (and corresponding threads) up to the pool size.  
    for (int i = 0; i < mDispatchers.length; i++) {  
        NetworkDispatcher networkDispatcher = new NetworkDispatcher(mNetworkQueue, mNetwork,  
                mCache, mDelivery);  
         mDispatchers[i] = networkDispatcher;  
         networkDispatcher.start();  
     }  
 }  
```
这里先是创建了一个CacheDispatcher的实例，然后调用了它的start()方法，接着在一个for循环里去创建NetworkDispatcher的实例，并分别调用它们的start()方法。这里的CacheDispatcher和NetworkDispatcher都是继承自Thread的，而默认情况下for循环会执行四次，也就是说当调用了Volley.newRequestQueue(context)之后，就会有五个线程一直在后台运行，不断等待网络请求的到来，

其中

CacheDispatcher是缓存线程，NetworkDispatcher是网络请求线程。

得到了RequestQueue之后，我们只需要构建出相应的Request，然后调用RequestQueue的add()方法将Request传入就可以完成网络请求操作了，那么不用说，add()方法的内部肯定有着非常复杂的逻辑，我们来一起看一下：

```java
public <T> Request<T> add(Request<T> request) {
    // Tag the request as belonging to this queue and add it to the set of current requests.  
    request.setRequestQueue(this);
    synchronized (mCurrentRequests) {
        mCurrentRequests.add(request);
    }
    // Process requests in the order they are added.  
    request.setSequence(getSequenceNumber());
    request.addMarker("add-to-queue");  
 // If the request is uncacheable, skip the cache queue and go straight to the network.  
if (!request.shouldCache()) {  
    mNetworkQueue.add(request);  
    return request;  
}  
// Insert request into stage if there's already a request with the same cache key in flight.  
synchronized (mWaitingRequests) {  
    String cacheKey = request.getCacheKey();  
    if (mWaitingRequests.containsKey(cacheKey)) {  
        // There is already a request in flight. Queue up.  
        Queue<Request<?>> stagedRequests = mWaitingRequests.get(cacheKey);  
        if (stagedRequests == null) {  
            stagedRequests = new LinkedList<Request<?>>();  
        }  
        stagedRequests.add(request);  
        mWaitingRequests.put(cacheKey, stagedRequests);  
        if (VolleyLog.DEBUG) {  
            VolleyLog.v("Request for cacheKey=%s is in flight, putting on hold.", cacheKey);  
        }  
    } else {  
        // Insert 'null' queue for this cacheKey, indicating there is now a request in  
        // flight.  
        mWaitingRequests.put(cacheKey, null);  
        mCacheQueue.add(request);  
    }  
    return request;  
}
```
可以看到，在第11行的时候会判断当前的请求是否可以缓存，如果不能缓存则在第12行直接将这条请求加入网络请求队列，可以缓存的话则在第33行将这条请求加入缓存队列。在默认情况下，每条请求都是可以缓存的，当然我们也可以调用Request的setShouldCache(false)方法来改变这一默认行为。

OK，那么既然默认每条请求都是可以缓存的，自然就被添加到了缓存队列中，于是一直在后台等待的缓存线程就要开始运行起来了，我们看下CacheDispatcher中的run()方法，代码如下所示：

```java
public class CacheDispatcher extends Thread {  

……
    @Override
    public void run() {
        if (DEBUG) VolleyLog.v("start new dispatcher");
        Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
        // Make a blocking call to initialize the cache.  
    mCache.initialize();  
    while (true) {  
        try {  
            // Get a request from the cache triage queue, blocking until  
            // at least one is available.  
            final Request<?> request = mCacheQueue.take();  
            request.addMarker("cache-queue-take");  
            // If the request has been canceled, don't bother dispatching it.  
            if (request.isCanceled()) {  
                request.finish("cache-discard-canceled");  
                continue;  
            }  
            // Attempt to retrieve this item from cache.  
            Cache.Entry entry = mCache.get(request.getCacheKey());  
            if (entry == null) {  
                request.addMarker("cache-miss");  
                // Cache miss; send off to the network dispatcher.  
                mNetworkQueue.put(request);  
                continue;  
            }  
            // If it is completely expired, just send it to the network.  
            if (entry.isExpired()) {  
                request.addMarker("cache-hit-expired");  
                request.setCacheEntry(entry);  
                mNetworkQueue.put(request);  
                continue;  
            }  
            // We have a cache hit; parse its data for delivery back to the request.  
            request.addMarker("cache-hit");  
            Response<?> response = request.parseNetworkResponse(
                                              new NetworkResponse(entry.data, entry.responseHeaders));  
            request.addMarker("cache-hit-parsed");  
            if (!entry.refreshNeeded()) {  
                // Completely unexpired cache hit. Just deliver the response.  
                mDelivery.postResponse(request, response);  
            } else {  
                // Soft-expired cache hit. We can deliver the cached response,  
                // but we need to also send the request to the network for  
                // refreshing.  
                request.addMarker("cache-hit-refresh-needed");  
                request.setCacheEntry(entry);  
                // Mark the response as intermediate.  
                response.intermediate = true;  
                // Post the intermediate response back to the user and have  
                // the delivery then forward the request along to the network.  
                mDelivery.postResponse(request, response, new Runnable() {  
                    @Override  
                    public void run() {  
                        try {  
                            mNetworkQueue.put(request);  
                        } catch (InterruptedException e) {  
                            // Not much we can do about this.  
                        }  
                    }  
                });  
            }  
        } catch (InterruptedException e) {  
            // We may have been interrupted because it was time to quit.  
            if (mQuit) {  
                return;  
            }  
            continue;  
        }  
    }  
}
```
代码有点长，我们只挑重点看。首先在11行可以看到一个while(true)循环，说明缓存线程始终是在运行的，接着在第23行会尝试从缓存当中取出响应结果，如何为空的话则把这条请求加入到网络请求队列中，如果不为空的话再判断该缓存是否已过期，如果已经过期了则同样把这条请求加入到网络请求队列中，否则就认为不需要重发网络请求，直接使用缓存中的数据即可。之后会在第39行调用Request的

parseNetworkResponse()方法来对数据进行解析，再往后就是将解析出来的数据进行回调了，这部分代码我们先跳过，因为它的逻辑和NetworkDispatcher后半部分的逻辑是基本相同的，那么我们等下合并在一起看就好了，先来看一下NetworkDispatcher中是怎么处理网络请求队列的，代码如下所示：

```java
public class NetworkDispatcher extends Thread {  
    ……  
    @Override  
    public void run() {  
        Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);  
        Request<?> request;  
        while (true) {  
            try {  
                // Take a request from the queue.  
                 request = mQueue.take();  
             } catch (InterruptedException e) {  
                 // We may have been interrupted because it was time to quit.  
                 if (mQuit) {  
                     return;  
                 }  
                 continue;  
             }  
             try {  
                 request.addMarker("network-queue-take");  
                 // If the request was cancelled already, do not perform the  
                 // network request.  
                 if (request.isCanceled()) {  
                     request.finish("network-discard-cancelled");  
                     continue;  
                 }  
                 addTrafficStatsTag(request);  
                 // Perform the network request.  
                 NetworkResponse networkResponse = mNetwork.performRequest(request);  
                 request.addMarker("network-http-complete");  
                 // If the server returned 304 AND we delivered a response already,  
                 // we're done -- don't deliver a second identical response.  
                 if (networkResponse.notModified && request.hasHadResponseDelivered()) {  
                     request.finish("not-modified");  
                     continue;  
                 }  
                 // Parse the response here on the worker thread.  
                 Response<?> response = request.parseNetworkResponse(networkResponse);  
                 request.addMarker("network-parse-complete");  
                 // Write to cache if applicable.  
                 // TODO: Only update cache metadata instead of entire record for 304s.  
                 if (request.shouldCache() && response.cacheEntry != null) {  
                     mCache.put(request.getCacheKey(), response.cacheEntry);  
                     request.addMarker("network-cache-written");  
                 }  
                 // Post the response back.  
                 request.markDelivered();  
                 mDelivery.postResponse(request, response);  
             } catch (VolleyError volleyError) {  
                 parseAndDeliverNetworkError(request, volleyError);  
             } catch (Exception e) {  
                 VolleyLog.e(e, "Unhandled exception %s", e.toString());  
                 mDelivery.postError(request, new VolleyError(e));  
             }  
         }  
     }  
 }  
```
同样地，在第7行我们看到了类似的while(true)循环，说明网络请求线程也是在不断运行的。在第28行的时候会调用Network的performRequest()方法来去发送网络请求，而Network是一个接口，这里具体的实现是BasicNetwork，我们来看下它的

performRequest()方法，如下所示：

```java
public class BasicNetwork implements Network {  
    ……  
    @Override  
    public NetworkResponse performRequest(Request<?> request) throws VolleyError {  
        long requestStart = SystemClock.elapsedRealtime();  
        while (true) {  
            HttpResponse httpResponse = null;  
            byte[] responseContents = null;  
            Map<String, String> responseHeaders = new HashMap<String, String>();  
             try {  
                 // Gather headers.  
                 Map<String, String> headers = new HashMap<String, String>();  
                 addCacheHeaders(headers, request.getCacheEntry());  
                 httpResponse = mHttpStack.performRequest(request, headers);  
                 StatusLine statusLine = httpResponse.getStatusLine();  
                 int statusCode = statusLine.getStatusCode();  
                 responseHeaders = convertHeaders(httpResponse.getAllHeaders());  
                 // Handle cache validation.  
                 if (statusCode == HttpStatus.SC_NOT_MODIFIED) {  
                     return new NetworkResponse(HttpStatus.SC_NOT_MODIFIED,  
                             request.getCacheEntry() == null ? null : request.getCacheEntry().data,  
                             responseHeaders, true);  
                 }  
                 // Some responses such as 204s do not have content.  We must check.  
                 if (httpResponse.getEntity() != null) {  
                   responseContents = entityToBytes(httpResponse.getEntity());  
                 } else {  
                   // Add 0 byte response as a way of honestly representing a  
                   // no-content request.  
                   responseContents = new byte[0];  
                 }  
                 // if the request is slow, log it.  
                 long requestLifetime = SystemClock.elapsedRealtime() - requestStart;  
                 logSlowRequests(requestLifetime, request, responseContents, statusLine);  
                 if (statusCode < 200 || statusCode > 299) {  
                     throw new IOException();  
                 }  
                 return new NetworkResponse(statusCode, responseContents, responseHeaders, false);  
             } catch (Exception e) {  
                 ……  
             }  
         }  
     }  
 }  
```
这段方法中大多都是一些网络请求细节方面的东西，我们并不需要太多关心，需要注意的是在第14行调用了HttpStack的performRequest()方法，这里的HttpStack就是在一开始调用newRequestQueue()方法是创建的实例，默认情况下如果系统版本号大于9就创建的HurlStack对象，否则创建HttpClientStack对象。前面已经说过，这两个对象的内部实际就是分别使用HttpURLConnection和HttpClient来发送网络请求的，我们就不再跟进去阅读了，之后会将服务器返回的数据组装成一个NetworkResponse对象进行返回。

在NetworkDispatcher中收到了NetworkResponse这个返回值后又会调用Request的parseNetworkResponse()方法来解析NetworkResponse中的数据，以及将数据写入到缓存，这个方法的实现是交给Request的子类来完成的，因为不同种类的Request解析的方式也肯定不同。还记得我们在上一篇文章中学习的自定义Request的方式吗？其中parseNetworkResponse()这个方法就是必须要重写的。

在解析完了NetworkResponse中的数据之后，又会调用ExecutorDelivery的postResponse()方法来回调解析出的数据，代码如下所示：

```java
public void postResponse(Request<?> request, Response<?> response, Runnable runnable) {  
    request.markDelivered();  
    request.addMarker("post-response");  
    mResponsePoster.execute(new ResponseDeliveryRunnable(request, response, runnable));  
}  
```
其中，在mResponsePoster的execute()方法中传入了一个ResponseDeliveryRunnable对象，就可以保证该对象中的run()方法就是在主线程当中运行的了，我们看下run()方法中的代码是什么样的：

```java
private class ResponseDeliveryRunnable implements Runnable {  
    private final Request mRequest;  
    private final Response mResponse;  
    private final Runnable mRunnable;  

    public ResponseDeliveryRunnable(Request request, Response response, Runnable runnable) {  
        mRequest = request;  
        mResponse = response;  
        mRunnable = runnable;  
     }  

     @SuppressWarnings("unchecked")  
     @Override  
     public void run() {  
         // If this request has canceled, finish it and don't deliver.  
         if (mRequest.isCanceled()) {  
             mRequest.finish("canceled-at-delivery");  
             return;  
         }  
         // Deliver a normal response or error, depending.  
         if (mResponse.isSuccess()) {  
             mRequest.deliverResponse(mResponse.result);  
         } else {  
             mRequest.deliverError(mResponse.error);  
         }  
         // If this is an intermediate response, add a marker, otherwise we're done  
         // and the request can be finished.  
         if (mResponse.intermediate) {  
             mRequest.addMarker("intermediate-response");  
         } else {  
             mRequest.finish("done");  
         }  
         // If we have been provided a post-delivery runnable, run it.  
         if (mRunnable != null) {  
             mRunnable.run();  
         }  
    }  
 }  
```
代码虽然不多，但我们并不需要行行阅读，抓住重点看即可。其中在第22行调用了Request的deliverResponse()方法，有没有感觉很熟悉？没错，这个就是我们在自定义Request时需要重写的另外一个方法，每一条网络请求的响应都是回调到这个方法中，最后我们再在这个方法中将响应的数据回调到Response.Listener的onResponse()方法中就可以了。

好了，到这里我们就把Volley的完整执行流程全部梳理了一遍，你是不是已经感觉已经很清晰了呢？对了，还记得在文章一开始的那张流程图吗，刚才还不能理解，现在我们再来重新看下这张图：

![img](http://img.blog.csdn.net/20140511114837375?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZ3VvbGluX2Jsb2c=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

其中蓝色部分代表主线程，绿色部分代表缓存线程，橙色部分代表网络线程。我们在主线程中调用RequestQueue的add()方法来添加一条网络请求，这条请求会先被加入到缓存队列当中，如果发现可以找到相应的缓存结果就直接读取缓存并解析，然后回调给主线程。如果在缓存中没有找到结果，则将这条请求加入到网络请求队列中，然后处理发送HTTP请求，解析响应结果，写入缓存，并回调主线程。

怎么样，是不是感觉现在理解这张图已经变得轻松简单了？好了，到此为止我们就把Volley的用法和源码全部学习完了，相信你已经对Volley非常熟悉并可以将它应用到实际项目当中了，那么Volley完全解析系列的文章到此结束，感谢大家有耐心看到最后。
