# 1. 加载大尺寸图片

## 压缩原因:

1. imageview大小如果是200\*300那么加载个2000*3000的图片到内存中显然是浪费可耻滴行为
2. 最重要的是图片过大时直接加载原图会造成OOM异常(out of memory内存溢出)

所以一般对于大图我们需要进行下压缩处理，权威处理方法参考 安卓开发者中心的大图片处理教程
http://developer.android.com/training/displaying-bitmaps/load-bitmap.html

看不懂英文的话木有关系，本篇会有介绍

主要处理思路是:

1. 获取图片的像素宽高(不加载图片至内存中，所以不会占用资源)
2. 计算需要压缩的比例
3. 按将图片用计算出的比例压缩，并加载至内存中使用

官网大图片加载教程(上面网址里的)对应代码就是:

```java
/**
* 获取压缩后的图片
* @param res
* @param resId
* @param reqWidth            所需图片压缩尺寸最小宽度
* @param reqHeight           所需图片压缩尺寸最小高度
* @return
*/
public static Bitmap decodeSampledBitmapFromResource(Resources res, int resId,
        int reqWidth, int reqHeight) {

    // 首先不加载图片,仅获取图片尺寸
    final BitmapFactory.Options options = new BitmapFactory.Options();
    // 当inJustDecodeBounds设为true时,不会加载图片仅获取图片尺寸信息
    options.inJustDecodeBounds = true;
    // 此时仅会将图片信息会保存至options对象内,decode方法不会返回bitmap对象
    BitmapFactory.decodeResource(res, resId, options);

    // 计算压缩比例,如inSampleSize=4时,图片会压缩成原图的1/4
    options.inSampleSize = calculateInSampleSize(options, reqWidth, reqHeight);

    // 当inJustDecodeBounds设为false时,BitmapFactory.decode...就会返回图片对象了
    options. inJustDecodeBounds = false;
    // 利用计算的比例值获取压缩后的图片对象
    return BitmapFactory.decodeResource(res, resId, options);
}
```

代码详解:

核心方法是BitmapFactory.decodeXxx(...., options)，Xxx的意思是此外还有一系列的decodeFile/decodeStream等等方法，都是利用options灵活解析获取图片，只不过解析图片的来源不同罢了，比如网络图片获取，一般就是解析字节流信息然后decode获取图片实例

Options是图片配置信息，参数详细介绍下:

- inJustDecodeBounds

是否只解析边界，设为true时去decode获取图片，只会加载像素宽高信息，设为false时decode则会完全加载图片

- inSampleSize

压缩比例，比如原图200\*300，如果值是2时会压缩成100\*150; 是4则图片压缩成50\*75，最好是2的幂数，比如2 4 8 16 .....

- outHeight 图片原高度

- outWidth 图片原宽度

其他参数自行研究，这里暂时只用到这几个

decodeSampledBitmapFromResource方法内的三段代码对应上面的三步流程，难点在于中间那步，压缩比例的计算，官网同样提供了个calculateInSampleSize方法，其中reqWidth和reqHeight是所需图片限定最小宽高值

```java
/**
* 计算压缩比例值
* @param options       解析图片的配置信息
* @param reqWidth            所需图片压缩尺寸最小宽度
* @param reqHeight           所需图片压缩尺寸最小高度
* @return
*/
public static int calculateInSampleSize(BitmapFactory.Options options,
             int reqWidth, int reqHeight) {
       // 保存图片原宽高值
       final int height = options. outHeight;
       final int width = options. outWidth;
       // 初始化压缩比例为1
       int inSampleSize = 1;

       // 当图片宽高值任何一个大于所需压缩图片宽高值时,进入循环计算系统
       if (height > reqHeight || width > reqWidth) {

             final int halfHeight = height / 2;
             final int halfWidth = width / 2;

             // 压缩比例值每次循环两倍增加,
             // 直到原图宽高值的一半除以压缩值后都~大于所需宽高值为止
             while ((halfHeight / inSampleSize) >= reqHeight
                        && (halfWidth / inSampleSize) >= reqWidth) {
                  inSampleSize *= 2;
            }
      }

       return inSampleSize;
}
```

利用此方法获取到所需压缩比例值，最终获取到压缩后的图片

以上代码能够看懂的话，下面这段扯淡可以跳过

逻辑是将原图宽高一半一半的缩减，一直减到宽高都小于自己设定的限定宽高时为止，测试的时候问题来了，原图400\*300，我限定值200\*150，if满足进入，while循环第一次，400/2/1=200不满足>的条件~结束循环，最终返回了个inSampleSize=1给我

马丹我限定值正好是原图的一半啊，你应该返回给我2啊~你特么最后返回个1给我，那压缩处理后的图还是400\*300!!!

当我将限定值稍微改一下变成195\*145稍微降低一点点时~if满足进入，while循环第一次，400/2/1>195满足~
然后压缩比例1\*2变成了2，在下一次while循环时不满足条件结束，最后返回比例值2~ 满足压缩预期

官网的这个方法是: 将图片一半一半的压缩，直到压缩成成大于所需宽高数的那个最低值
大于~不是大于等于，所以就会出现我上面那种情况，我觉得方法不是太好= = 能满足压缩的需求，但是压缩的比例不够准确~
所以最好改成大于等于，如下(个人意见，仅供参考，在实际压缩中很少遇到恰巧等于的这个情况，所以>和>=差别也不大额~看我这扯扯淡就当对计算比例的逻辑加深个理解吧)

```java
while ((halfHeight / inSampleSize) >= reqHeight
          && (halfWidth / inSampleSize) >= reqWidth) {
     inSampleSize *= 2;
}
```

优化:

还是上面例子，如果限定了200\*150，而原图是390\*290会是个啥情况?
还是第一次while循环，390/2/1结果是195不满足>200的情况，结束循环，比例值为1，最后图片压缩成400\*300
虽然压缩一次以后没有满足大于所需宽高，但是和所需宽高很接近啊!!!
能不能做一个获取压缩成最接近所需宽高数的比例值呢?
我也不知道= = 回头可以慢慢研究， 这个"接近"的定义比较模糊，不好掌握~
找了几个有名的图片加载开源框架发现也都没有这种处理- -不知道是这样设计是不需要呢，还是没啥用呢

以上，图片的像素大小已经做了缩放，但是图片的大小除了和像素有关，还和色彩样式有关
不同的样式决定了图片单个像素占的字节数
比如，图片默认的色彩样式为ARGB_8888，每个像素占4byte(字节)大小
参考资料:http://developer.android.com/reference/android/graphics/Bitmap.Config.html

可以看到一共有四种色彩样式

- ALPHA_8：每个像素只要1字节~可惜只能代表透明度，没有颜色属性
- ARGB_4444：每个像素要2字节~带透明度的颜色~可惜官方不推荐使用了
- ARGB_8888：每个像素要4字节~带透明度的颜色， 默认色样
- RGB_565：每个像素要2字节~不带透明度的颜色

默认为ARGB_8888，如果想丧心病狂的继续减少图片所占大小~不需要透明度参数的话，
那就可以把色彩样式设为RGB_565

设置方法是在BitmapFactory.decode..获取图片事例时
修改配置参数的inPreferredConfig 参数
opts.inPreferredConfig = Bitmap.Config. RGB_565 ;


想亲自撸一撸试一试压缩图片了吧?
要注意点问题，如果用res包下图片测试的话，你会发现有图片尺寸有点混乱
那是因为在drawable-\*dpi文件夹中的图片会根据对应对应的屏幕密度值不同自动进行一定的缩放，
比如放在drawable-hdpi里的图片，直接不经过压缩BitmapFactor.decode..出来，会发现bitmap的宽高值是原图的2/3，
测试的时候图片记得放在drawable包下(没有的话自己res下新建一个)，否则你会被奇怪的宽高值弄凌乱的
具体变化原因参考源代码处理，或者网上搜搜看

还有就是BitmapFactory.decodeStream方法会偶尔解析图片失败(好像是安卓低版本的一个bug)
此时推荐做法是将流转换为字节流处理，然后利用decodeByteArray方法获取图片
