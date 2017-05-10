[Android AsyncTask 源码解析](http://blog.csdn.net/lmj623565791/article/details/38614699)

[Android AsyncTask完全解析，带你从源码的角度彻底理解](http://blog.csdn.net/guolin_blog/article/details/11711405)

# AsyncTask

AsyncTask是Thread+Handler的替代，底层是Handler和ThreadPoolExecutor的封装，android1.5引进

```java
class AsyncTask<Params, Progress, Result>{
    private Handler mHandler;
    private ThreadPoolExecutor mExecutor;
    
    public void execute(Params... params){
        //串行执行
    }
    
    public void executeOnExecutor(){ 
        //并行，串行执行
    }
    
    public void onPreExecute(){
        //在主线程执行，做耗时操作前的准备工作
    }
    
    public Result doInBackground(Params... params){
        //子线程执行，执行耗时的操作
    }
    
    public void publishProgress(Progress... values){
        
    }
    
    public void onProgressUpdate(Progress... values){
        //主线程执行，更新进度
    }
    
    public void onPostExecute(Result result){
        
    }
}
```

## AsyncTask< Params，Progress，Result >
第一个泛型：doInBackground()方法的参数
第二个泛型：onProgressUpdate()方法的进度
第三个泛型：doInBackground ()方法的返回结果和onPostExecute()方法的参数

## void  onPreExecute()
主线程执行，主要做耗时操作前的准备工作

## Result  doInBackground(Params... params)
子线程执行，做耗时的操作，结果传递给onPostExecute()方法

## void publishProgress(Progress... values)
在doInBackground()方法中调用，调用publishProgress方法都会触发onProgressUpdate()执行

## void  onProgressUpdate(Progress... values)
主线程执行，更新进度

## void  onPostExecute(Result result)
耗时方法结束后，执行该方法，主线程执行

## execute(Params... params)
参数会在doInbackground中获取，执行参数和doInbackground()参数一样

## executeOnExecutor
executeOnExecutor(AsyncTask.THREAD_POOL_EXECUTOR,null) 并行执行
executeOnExecutor(AsyncTask.SERIAL_EXECUTOR,null) 串行执行

## 源码分析

```java
private static final int CORE_POOL_SIZE = 5;  //核心线程数
private static final int MAXIMUM_POOL_SIZE = 128;   //最大线程数
private static final int KEEP_ALIVE = 1;  
//超时时间，当线程数超过核心线程数时，超过这个时间的空线程就会被销毁，直到线程数等于核心线程
```

## AsyncTask缺陷
- 同时只有5个线程去访问网络->这个是重点（核心线程数是5）
- 线程数目超过128，会抛异常->这个情况其实还好（最大线程数是128）
## AsyncTask版本差异

| 版本     | corepoolsize | maximumpoolsize | keepalivetime | 阻塞队列 |
| :----- | :----------- | :-------------- | :------------ | :--- |
| 3.0-版本 | 5            | 128             | 10            | 10   |
| 4.0+版本 | 5            | 128             | 1             | 10   |
| 5.0+版本 | cpu_count+1  | cpu_count*2+1   | 1             | 128  |
- 1.5前	串行执行的，每次执行1个任务
- 1.6-2.3并行执行的，每次执行5个任务
- 3.0后提供串行和并行,默认情况是串行
- execute()默认是串行执行
- executeOnExecutor(AsyncTask.SERIAL_EXECUTOR, null);//串行
- executeOnExecutor(AsyncTask.THREAD_POOL_EXECUTOR, null);//并行

AsyncTask的缺陷，可以分为两个部分说，在3.0以前，最大支持128个线程的并发，10个任务的等待。在3.0以后，无论有多少任务，都会在其内部单线程执行

## **AsyncTask封装**
AsyncTask函数化的封装，AsyncTask函数式的调用

```java
public AsyncTask() {
        mWorker = new WorkerRunnable<Params, Result>() {
            public Result call() throws Exception {
                mTaskInvoked.set(true);

                Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
                //noinspection unchecked  
                return postResult(doInBackground(mParams));
            }
        };
        ...
    }
```
## 实现一个简单的AsyncTask

```java
import android.os.Handler;
import android.os.HandlerThread;
import android.os.Looper;

public abstract class SimpleAsyncTask<Result> {
    // HandlerThread内部封装了自己的Handler和Thead，有单独的Looper和消息队列
    private static final HandlerThread HT = new HandlerThread(SimpleAsyncTask.class.getName(),
            android.os.Process.THREAD_PRIORITY_BACKGROUND);
    static {
        HT.start();
    }

    // 获取调用execute的线程的Looper, 构建Handler
    final Handler mUIHandler = new Handler(Looper.getMainLooper());
    // 与异步线程队列关联的Handler
    final Handler mAsyncHandler = new Handler(HT.getLooper());

    /**
     * @功能描述 : onPreExecute任务执行之前的初始化操作等
     */
    protected void onPreExecute() {

    }

    /**
     * doInBackground后台执行任务
     * 
     * @return 返回执行结果
     */
    protected abstract Result doInBackground();

    /**
     * doInBackground返回结果传递给执行在UI线程的onPostExecute
     * 
     * @param result
     */
    protected void onPostExecute(Result result) {
    }

    /**
     * execute方法，执行任务，调用doInBackground，并且将结果投递给UI线程， 使用户可以在onPostExecute处理结果
     * 
     * @return
     */
    public final SimpleAsyncTask<Result> execute() {
        onPreExecute();
        // 将任务投递到HandlerThread线程中执行
        mAsyncHandler.post(new Runnable() {
            @Override
            public void run() {
                // 后台执行任务,完成之后向UI线程post数据，用以更新UI等操作
                postResult(doInBackground());
            }
        });

        return this;
    }

    private void postResult(final Result result) {
        mUIHandler.post(new Runnable() {
            @Override
            public void run() {
                onPostExecute(result);
            }
        });
    }
}
```
使用SimpleAsyncTask的示例代码
```java
new SimpleAsyncTask<String>() {

    private void makeToast(String msg) {
        Toast.makeText(getApplicationContext(), msg, Toast.LENGTH_SHORT).show();
    }

    @Override
    protected void onPreExecute() {
        makeToast("onPreExecute");
    }

    @Override
    protected String doInBackground() {
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "Hello";
    }

    @Override
    protected void onPostExecute(String result) {
        makeToast("onPostExecute : " + result);
    }
}.execute();
```