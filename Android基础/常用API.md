## Runtime

```
Runtime.getRuntime().availableProcessors(); // 获取CPU核心数
Runtime.getRuntime().maxMemory();
```

## Process

```
Process.THREAD_PRIORITY_BACKGROUND;
Process.killProcess();
Process.myTid();
Process.setThreadPriority();
System.exit(1);
```

```java
public static String getProcessName(Context cxt, int pid) {
        ActivityManager am = (ActivityManager) cxt
                .getSystemService(Context.ACTIVITY_SERVICE);
        List<RunningAppProcessInfo> runningApps = am.getRunningAppProcesses();
        if (runningApps == null) {
            return null;
        }
        for (RunningAppProcessInfo procInfo : runningApps) {
            if (procInfo.pid == pid) {
                return procInfo.processName;
            }
        }
        return null;
    }
```

## Handler

```
Looper.getMainLooper() == Looper.myLooper(); // 判断是否是主线程
Handler mHandler = new Handler(Looper.getMainLooper());
mhandler.post(Runnable);// handler在那个线程创建的，任务就在那个线程执行
```

## 多线程

- AtomicInteger incrementAndGet()
- BlockingQueue，PriorityBlockingQueue
- AtomicInteger incrementAndGet()

## ViewCompat

ViewCompat.SCROLL_AXIS_VERTICAL

ViewCompat.offsetTopAndBottom

## Activity

public void onWindowFocusChanged(boolean hasFocus)

package保护的类，新建一个一样的包，把类移到新建的包下即可

## TextUtils

- isEmpty()
- join()

## TaskStackBuilder

任务栈Builder

## UncaughtExceptionHandler

```
public class CrashHandler implements Thread.UncaughtExceptionHandler {
    @Override
    public void uncaughtException(Thread thread, Throwable throwable) {
        
    }
}
```
子线程弹土司
```
new Thread(new Runnable() {
    @Override
    public void run() {
        Looper.prepare();
        Toast.makeText(context,"很抱歉，程序出现异常，即将退出。",Toast.LENGTH_SHORT).show();
        Looper.loop();
    }
}).start();
```
## ViewFlipper

实现左右滑动的效果。

通常情况下都是用ViewPager 来实现的
viewPager可以兼容低版本,而ViewFlipper是android4.0才引入的新控件
viewPager是一页一页的,可以带动画效果
而ViewFlipper 是一层一层的,当然也可以实现切换的动画效果,但是比viewpager复杂些

## FragmentTabHost

## Activity

    recreate();
    setTheme();
## GZIP压缩解压缩

GZIPOutputStream

GZIPInputStream

## 网络请求CallBack封装

```
public abstract class BaseCallBack<T> {
    
    Type type;
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

    public BaseCallBack()
    {
        type = getSuperclassTypeParameter(getClass());
    }

    public abstract void onFailure(Call call, IOException e);

    public abstract void onSuccess(Call call, T t);
}
```