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
```

## Handler

```
Looper.getMainLooper() == Looper.myLooper(); // 判断是否是主线程
Handler mHandler = new Handler(Looper.getMainLooper());
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