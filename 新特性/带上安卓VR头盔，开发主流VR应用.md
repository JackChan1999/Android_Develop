# 1. VR开发概述
时下关于“谷歌、Android与VR”的各种言论纷飞。VR群里有人在争论Android VR是不是一体机，是不是类似Android Wear、为VR打造的全新平台，是不是改良后的Android N。

随着vr设备的流行开来，各大招聘平台上也发布了不少关于andorid vr开发相关的Android工程师岗位，从这点来说掌握vr
在项目中的实际应用要点，有助于大家为自己的开发经验上增加前沿技术的积累。

经过研发市面上的主流vr app 的功能，抽取并整合项目中的vr开发知识点，希望大家掌握后，在企业相关vr app游刃有余。

## 1.1 下载google vr sdk 并搭建开发环境

1. 带大家去github上搜索并下载google vr sdk
2. 介绍sdk的组成部分与应用范围
3. 搭建一个基本android vr app的开发环境

## 1.2 CardBorad应用核心功能
1. 带大家查找本地vr 全景图片资源
2. 介绍vr全景图与普通图片的不同点
3. 使用rv列表进行展示
4. 使用VrPanoramaView控件进行本地全景图片的展示

## 1.3 UtoVR应用核心功能
1. 带大家通过网络请求获取vr视频的json数据
2. 使用 Gson解析得到javaBean数据
3. 使用VrVideoView控件进行网络全景视图的展示

以上这些功能是现流行的在线vr视频，vr图片相关app的核心功能。例如.vr管家应用，3d播播，discovery VR ，看房 vr等等热门应用。

## 1.4 知识点
1. http网络请求技术
2. Gson解析技术与gsonformat插件
3. RecyclerView与cardView
4. Glider流行图片加载框架
5. VrPanoramaView
6. VrVideoView

## 5. 效果图

<img src="http://open.itcast.cn/upload/image/20170105/1483580280119770.jpg" /> 
<img src="http://open.itcast.cn/upload/image/20170105/1483580281100694.jpg">

![VR](http://open.itcast.cn/upload/image/20170105/1483580284398216.jpg)
![VR](http://open.itcast.cn/upload/image/20170105/1483580285131199.jpg) 
![VR](http://open.itcast.cn/upload/image/20170105/1483580288132602.jpg)
![VR](http://open.itcast.cn/upload/image/20170105/1483580300771990.jpg)

# 2. 全景图片显示

##1.搭建vr全景图片的开发环境

  //步骤一。搭建vr全景图片的开发环境
###1.1导入全景图相关的三个开发库common,commonwidget,panowidget

###1.2.依赖该库
```gradle
    compile project(':common')
    compile project(':commonwidget')
    compile project(':panowidget')
    //google的一套序列化数据结构开发库
    compile 'com.google.protobuf.nano:protobuf-javanano:3.0.0-alpha-7'
```

###1.3.准备全景图片测试资源 放在assets目录 例:assets/a.jpg(全景图与普通图片的不同 大,立体)
###1.4.功能清单配置android:largeHeap="true" 全景图片比较耗资源
```xml
  <application
        android:largeHeap="true"
```

##2.布局全景控件显示加载后的全景图片

```xml
   <com.google.vr.sdk.widgets.pano.VrPanoramaView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id="@+id/vr_pv" />
```

```java
public class MainActivity extends AppCompatActivity {

    private VrPanoramaView vrPanoramaView;
    private ImageTask imageTask;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
```
####2.1.布局查找控件
```java
        //步骤二。布局全景控件显示加载后的全景图片
        //2.1.布局查找控件
        vrPanoramaView = (VrPanoramaView) findViewById(R.id.vr_pv);
```
####2.2.设置初始化参数
```java
        //2.2.设置初始化参数
        vrPanoramaView.setDisplayMode(VrWidgetView.DisplayMode.FULLSCREEN_STEREO);
        //信息图标
        vrPanoramaView.setInfoButtonEnabled(false);
        //全屏按钮
        vrPanoramaView.setFullscreenButtonEnabled(false);
```
####2.3.创建异步任务加载图片 Bitmap是图片在内存中的表示对象，全景图也可加载成bitmap
```java
        //2.3.创建异步任务加载图片 Bitmap是图片在内存中的表示对象，全景图也可加载成bitmap
        imageTask = new ImageTask();
        imageTask.execute();

    }
    private class ImageTask extends AsyncTask<Void, Void, Bitmap> {
        @Override
        protected Bitmap doInBackground(Void... params) {
            //2.4.从资产目录打开一个流
            try {
                InputStream inputStream = getAssets().open("a.jpg");
                //2.5.使用BitmapFactory转换成Bitmap
                Bitmap bitmap = BitmapFactory.decodeStream(inputStream);
                return bitmap;
            } catch (IOException e) {
                e.printStackTrace();
            }
            return null;
        }

        //2.6任务执行完后,可获取Bitmap图片
        @Override
        protected void onPostExecute(Bitmap bitmap) {
            super.onPostExecute(bitmap);
            if (bitmap != null) {
                VrPanoramaView.Options options = new VrPanoramaView.Options();
                //加载立体图片，上部分显示在左眼，下部分显示在右眼
                options.inputType = VrPanoramaView.Options.TYPE_STEREO_OVER_UNDER;
                if (listener == null) {
                    listener = new VrPanoramaEventListener() {
                        @Override
                        public void onLoadError(String errorMessage) {
                            super.onLoadError(errorMessage);
                            Toast.makeText(MainActivity.this, "错误消息:" + errorMessage, Toast.LENGTH_SHORT).show();
                        }

                        @Override
                        public void onLoadSuccess() {
                            super.onLoadSuccess();
                            Toast.makeText(MainActivity.this, "进入vr:", Toast.LENGTH_SHORT).show();
                        }
                    };
                    vrPanoramaView.setEventListener(listener);
                }
                //2.7.让控件加载bitmap对象
                vrPanoramaView.loadImageFromBitmap(bitmap, options);
                //2.8.如果loadImageFromBitmap加载失败需要提示用户相关信息则需要添加事件监听器listener
            }
        }
    }
    private VrPanoramaEventListener listener;
```
##3。VrPanoramaView控件退到后台，回到屏幕，销毁处理细节
```java
    //步骤三。VrPanoramaView控件退到后台，回到屏幕，销毁处理细节
    //3.1.退到后台.暂停显示
    @Override
    protected void onPause() {
        super.onPause();
        if (vrPanoramaView != null) {
            vrPanoramaView.pauseRendering();
        }
    }
    //3.2.回到屏幕,恢复显示
    @Override
    protected void onResume() {
        super.onResume();
        if (vrPanoramaView != null) {
            vrPanoramaView.resumeRendering();
        }
    }
    //3.3.退出界面停止显示
    @Override
    protected void onDestroy() {
        if (vrPanoramaView != null) {
            vrPanoramaView.shutdown();
        }
        if (imageTask != null && !imageTask.isCancelled()) {//销毁任务
            imageTask.cancel(true);
            imageTask = null;
        }
        super.onDestroy();
    }
}
```

#全景视频显示开发

##1。vr视频环境搭建

//步骤一。vr视频环境搭建
###1.1.导入vr sdk 中的相关库common,commonwidget,videowidget
###1.2.依赖以上三个库

```gradle
    compile project(':common')
    compile project(':commonwidget')
    compile project(':videowidget')
    // 出现类未定义错误的缺少库
    compile 'com.google.android.exoplayer:exoplayer:r1.5.10'
    compile 'com.google.protobuf.nano:protobuf-javanano:3.0.0-alpha-7'
```

###1.3.打开内存设置  android:largeHeap="true"
```xml
    <application
        android:largeHeap="true"
```

###1.4.准备测试使用的全景视频 放置在assets目录 例:assets/congo_2048.mp4

##2.布局视频控件，并加载视频内容

```java
public class MainActivity extends AppCompatActivity {

    private VrVideoView vrVideoView;
    private VideoTask videoTask;
    private SeekBar seekBar;
    private TextView text;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        //步骤二。布局视频控件，并加载视频内容
        //2.1布局控件
        //2.2查找控件
        vrVideoView = (VrVideoView) findViewById(R.id.vr_vv);
        //2.3加载视频数据
        videoTask = new VideoTask();
        videoTask.execute("congo_2048.mp4");
   
    }

    private class VideoTask extends AsyncTask<String, Void, Void> {
        @Override
        protected Void doInBackground(String... params) {
            VrVideoView.Options options = new VrVideoView.Options();
            //2.4.输入模式
            //立体模式
            options.inputType = VrVideoView.Options.TYPE_STEREO_OVER_UNDER;
            //2.5.设置视频来源
            //处理视频加载的格式(sd卡或者assets)
//            options.inputFormat = VrVideoView.Options.FORMAT_DEFAULT;
            //处理视频加载的格式 流媒体直播格式
            options.inputFormat=VrVideoView.Options.FORMAT_HLS;
            try {
                //2.8 如果资源有问题，不能正常播放需要处理下界面提示
                if (listener == null) {
                    listener = new VrVideoEventListener() {
                        @Override
                        public void onLoadSuccess() {
                            super.onLoadSuccess();
                            //获取当前总时长
                            long max=vrVideoView.getDuration();
                            seekBar.setMax((int) max);
                            seekBar.setProgress(0);
                            Toast.makeText(MainActivity.this, "准备播放vr", Toast.LENGTH_SHORT).show();

                        }
                        @Override
                        public void onNewFrame() {
                            super.onNewFrame();
                            //获取当前播放位置
                            long currentPosition = vrVideoView.getCurrentPosition();
                            //设置当前进度
                            seekBar.setProgress((int) currentPosition);
                            //时间值
                            String total=String.format("%.2f",vrVideoView.getDuration()/1000f);
                            String curr=String.format("%.2f",vrVideoView.getCurrentPosition()/1000f);
                            text.setText("播放进度"+curr+":"+total);
                        }
                        private boolean isPause=true;
                        @Override
                        public void onCompletion() {
                            super.onCompletion();
                            seekBar.setProgress(0);
                            vrVideoView.seekTo(0);//重回0位置
                            vrVideoView.pauseVideo();//暂停播放
                            isPause=true;//保存暂停状态
                        }

                        @Override
                        public void onClick() {
                            super.onClick();
                            if (isPause) {//播放
                                vrVideoView.playVideo();
                                isPause=false;
                            } else {
                                vrVideoView.pauseVideo();
                                isPause=true;
                            }
                        }

                        @Override
                        public void onLoadError(String errorMessage) {
                            super.onLoadError(errorMessage);
                            Toast.makeText(MainActivity.this, "加载失败", Toast.LENGTH_SHORT).show();
                        }
                    };
                    vrVideoView.setEventListener(listener);
                }
                //2.7加载视频资源
//              vrVideoView.loadVideoFromAsset(params[0], options);
                String url="http://youkesvideo.oss-cn-hangzhou.aliyuncs.com/movie2/2016/10/11/%E6%B9%84%E5%85%AC%E6%B2%B3%E8%A1%8C%E5%8A%A8.Operation.Mekong.2016.TC720P.X264.AAC.Mandarin.CHS.Mp4Ba.mp4";
                vrVideoView.loadVideo(Uri.parse(url),options);
            } catch (IOException e) {
                e.printStackTrace();
            }
            return null;
        }
    }

    private VrVideoEventListener listener;
```

##3.处理页面退到后台，回到屏幕,页面销毁。

```java
    //步骤三。处理页面退到后台，回到屏幕,页面销毁。

    //3.1.页面退到后台暂停视频
    @Override
    protected void onPause() {
        super.onPause();
        if (vrVideoView != null) {
            vrVideoView.pauseRendering();
        }
    }

    //3.2.页面回到屏幕继续播放
    @Override
    protected void onResume() {
        super.onResume();
        if (vrVideoView != null) {
            vrVideoView.resumeRendering();
        }
    }

    //3.3.页面销毁
    @Override
    protected void onDestroy() {
        super.onDestroy();
        if (vrVideoView != null) {
            vrVideoView.shutdown();
        }
        if (videoTask != null && !videoTask.isCancelled()) {
            videoTask.cancel(false);
            videoTask = null;
        }
    }
}
```

##4。添加进度条相关事件
###4.1布局查找出控件

```xml
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">
        <SeekBar
            android:id="@+id/seek_bar"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />
        <TextView
            android:id="@+id/text"
            android:background="#AEAEAE"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:gravity="center"
            android:textColor="#FFFFFF"
            android:textSize="22sp"
            android:text="00:00" />
    </LinearLayout>
```


```java
  seekBar = (SeekBar) findViewById(R.id.seek_bar);
  text = (TextView) findViewById(R.id.text);
```
###4.2.加载成功设置最大值 在VrVideoEventListener中的onLoadSuccess处理
###4.3.在播放过程中不断更新 进度值 onNewFrame 每播放一个画面就调用该方法一次
###4.4.同步理新文本时间值 在VrVideoEventListener中的onNewFrame处理
###4.5.播放完成重新播放 在VrVideoEventListener中的onCompletion处理