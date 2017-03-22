在日常开发中，可以说和Bitmap低头不见抬头见，基本上每个应用都会直接或间接的用到，而这里面又涉及到大量的相关知识。
所以这里把Bitmap的常用知识做个梳理，限于经验和能力，不做太深入的分析。

# 1. 区别decodeResource()和decodeFile()

这里的区别不是指方法名和参数的区别，而是对于解码后图片尺寸在处理上的区别：

> decodeFile()用于读取SD卡上的图，得到的是图片的原始尺寸
> decodeResource()用于读取Res、Raw等资源，得到的是图片的原始尺寸 * 缩放系数

可以看的出来，decodeResource()比decodeFile()多了一个缩放系数，缩放系数的计算依赖于屏幕密度，当然这个参数也是可以调整的：

```java
// 通过BitmapFactory.Options的这几个参数可以调整缩放系数
public class BitmapFactory {
    public static class Options {
        public boolean inScaled;     // 默认true
        public int inDensity;        // 无dpi的文件夹下默认160
        public int inTargetDensity;  // 取决具体屏幕
    }
}
```

我们分具体情况来看，现在有一张720x720的图片:

## 1.1 inScaled属性

如果inScaled设置为false，则不进行缩放，解码后图片大小为720x720; 否则请往下看。
如果inScaled设置为true或者不设置，则根据inDensity和inTargetDensity计算缩放系数。

## 1.2 默认情况

把这张图片放到drawable目录下, 默认：
以720p的红米3为例子，缩放系数 = inTargetDensity(具体320 / inDensity（默认160）= 2 = density，解码后图片大小为1440x1440。
以1080p的MX4为例子，缩放系数 = inTargetDensity(具体480 / inDensity（默认160）= 3 = density, 解码后图片大小为2160x2160。

## 1.3 *dpi文件夹的影响

把图片放到drawable或者raw这样不带dpi的文件夹，会按照上面的算法计算。
如果放到xhdpi会怎样呢？ 在MX4上，放到xhdpi，解码后图片大小为1080 x 1080。
因为放到有dpi的文件夹，会影响到inDensity的默认值，放到xhdpi为160 x 2 = 320; 所以缩放系数 = 480（屏幕） / 320 （xhdpi） = 1.5; 所以得到的图片大小为1080 x 1080。

## 1.4 手动设置缩放系数

如果你不想依赖于这个系统本身的density，你可以手动设置inDensity和inTargetDensity来控制缩放系数：

```java
BitmapFactory.Options options = new BitmapFactory.Options();
options.inJustDecodeBounds = false;
options.inSampleSize = 1;
options.inDensity = 160;
options.inTargetDensity = 160;
bitmap = BitmapFactory.decodeResource(getResources(),
        R.drawable.origin, options);
// MX4上，虽然density = 3
// 但是通过设置inTargetDensity / inDensity = 160 / 160 = 1
// 解码后图片大小为720x720
System.out.println("w:" + bitmap.getWidth()
        + ", h:" + bitmap.getHeight());
```

# 2. recycle()方法

## 2.1 官方说法

首先，Android对Bitmap内存(像素数据)的分配区域在不同版本上是有区分的：

> As of Android 3.0 (API level 11), the pixel data is stored on the Dalvik heap along with the associated bitmap.

从3.0开始，Bitmap像素数据和Bitmap对象一起存放在Dalvik堆中，而在3.0之前，Bitmap像素数据存放在Native内存中。
所以，在3.0之前，Bitmap像素数据在Nativie内存的释放是不确定的，容易内存溢出而Crash，官方强烈建议调用recycle()（当然是在确定不需要的时候）；而在3.0之后，则无此要求。
参考链接：[Managing Bitmap Memory](http://developer.android.com/training/displaying-bitmaps/manage-memory.html)

## 2.2 一点讨论

3.0之后官方无recycle()建议，是不是就真的不需要recycle()了呢？
在医生的这篇文章：[Bitmap.recycle引发的血案](http://blog.csdn.net/eclipsexys/article/details/50581162) 最后指出：“在不兼容Android2.3的情况下，别在使用recycle方法来管理Bitmap了，那是GC的事！”。文章开头指出了原因在于recycle()方法的注释说明:

```java
/**
 * ... This is an advanced call, and normally need not be called,
 * since the normal GC process will free up this memory when
 * there are no more references to this bitmap.
 */
public void recycle() {}
```

事实上这个说法是不准确的，是不能作为recycle()方法不调用的依据的。
因为从commit history中看，这行注释早在08年初始化代码的就有了，但是早期的代码并没有因此不需要recycle()方法了。
[![bitmap recycle history](http://jayfeng.com/images/bitmap_recycle_history.png)](http://jayfeng.com/images/bitmap_recycle_history.png)
如果3.0之后真的完全不需要主动recycle()，最新的AOSP源码应该有相应体现，我查了SystemUI和Gallery2的代码，并没有取缔Bitmap的recycle()方法。
所以，我个人认为，如果Bitmap真的不用了，recycle一下又有何妨？
PS：至于医生说的那个bug，显然是一种优化策略，APP开发中加个两个bitmap不相等的判断条件即可。

# 3. Bitmap到底占多大内存

这个已经有一篇bugly出品的绝好文章讲的很清楚：
[Android 开发绕不过的坑：你的 Bitmap 究竟占多大内存？](http://bugly.qq.com/bbs/forum.php?mod=viewthread&tid=498)

# 4. inBitmap

BitmapFactory.Options.inBitmap是Android3.0新增的一个属性，如果设置了这个属性则会重用这个Bitmap的内存从而提升性能。
但是这个重用是有条件的，在Android4.4之前只能重用相同大小的Bitmap，Android4.4+则只要比重用Bitmap小即可。
在官方网站有详细介绍，这里列举示例代码的两个方法了解一下：

```java
private static void addInBitmapOptions(BitmapFactory.Options options,
        ImageCache cache) {
    // inBitmap only works with mutable bitmaps, so force the decoder to
    // return mutable bitmaps.
    options.inMutable = true;

    if (cache != null) {
        // Try to find a bitmap to use for inBitmap.
        Bitmap inBitmap = cache.getBitmapFromReusableSet(options);

        if (inBitmap != null) {
            // If a suitable bitmap has been found,
            // set it as the value of inBitmap.
            options.inBitmap = inBitmap;
        }
    }
}

static boolean canUseForInBitmap(
        Bitmap candidate, BitmapFactory.Options targetOptions) {

    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
        // From Android 4.4 (KitKat) onward we can re-use
        // if the byte size of the new bitmap is smaller than
        // the reusable bitmap candidate
        // allocation byte count.
        int width = targetOptions.outWidth / targetOptions.inSampleSize;
        int height =
            targetOptions.outHeight / targetOptions.inSampleSize;
        int byteCount = width * height
            * getBytesPerPixel(candidate.getConfig());
        return byteCount <= candidate.getAllocationByteCount();
    }

    // On earlier versions,
    // the dimensions must match exactly and the inSampleSize must be 1
    return candidate.getWidth() == targetOptions.outWidth
        && candidate.getHeight() == targetOptions.outHeight
        && targetOptions.inSampleSize == 1;
}
```

参考链接：
[Managing Bitmap Memory](http://developer.android.com/training/displaying-bitmaps/manage-memory.html)
[Bitmap对象的复用](http://hukai.me/android-performance-oom/)

# 5. LRU缓存算法

> LRU，Least Recently Used，Discards the least recently used items first。

在最近使用的数据中，丢弃使用最少的数据。与之相反的还有一个MRU，丢弃使用最多的数据。
这就是著名的局部性原理。

## 5.1 实现思路

> 1.新数据插入到链表头部；
> 2.每当缓存命中（即缓存数据被访问），则将数据移到链表头部；
> 3.当链表满的时候，将链表尾部的数据丢弃。

[![bitmap_lru](http://jayfeng.com/images/bitmap_lru.png)](http://jayfeng.com/images/bitmap_lru.png)

## 5.2 LruCache

在Android3.1和support v4中均提供了Lru算法的实现类LruCache。
内部使用LinkedHashMap实现。

## 5.3 DiskLruCache

LruCache的所有对象和数据都是在内存中（或者说LinkedHashMap中），而DiskLruCache是磁盘缓存，不过它的实现要稍微复杂一点。
使用DiskLruCache后就不用担心文件或者图片太多占用过多磁盘空间，它能把那些不常用的图片自动清理掉。
DiskLruCache系统中并没有正式提供，需要另外下载: [DiskLruCache](https://android.googlesource.com/platform/libcore/+/jb-mr2-release/luni/src/main/java/libcore/io/DiskLruCache.java)

# 6. 计算inSampleSize

使用Bitmap节省内存最重要的技巧就是加载合适大小的Bitmap，因为以现在相机像素，很多照片都巨无霸的大，这些大图直接加载到内存，最容易OOM。
加载合适的Bitmap需要先读取Bitmap的原始大小，按缩小了合适的倍数的大小进行加载。
那么，这个缩小的倍数的计算就是inSampleSize的计算。

```java
// 根据maxWidth, maxHeight计算最合适的inSampleSize
public static int $sampleSize(BitmapFactory.Options options,
        int maxWidth, int maxHeight) {
    // raw height and width of image
    int rawWidth = options.outWidth;
    int rawHeight = options.outHeight;

    // calculate best sample size
    int inSampleSize = 0;
    if (rawHeight > maxHeight || rawWidth > maxWidth) {
        float ratioWidth = (float) rawWidth / maxWidth;
        float ratioHeight = (float) rawHeight / maxHeight;
        inSampleSize = (int) Math.min(ratioHeight, ratioWidth);
    }
    inSampleSize = Math.max(1, inSampleSize);

    return inSampleSize;
}
```

关于inSampleSize需要注意，它只能是2的次方，否则它会取最接近2的次方的值。

# 7. 缩略图

为了节省内存，需要先设置BitmapFactory.Options的inJustDecodeBounds为true，这样的Bitmap可以借助decodeFile方法把高和宽存放到Bitmap.Options中，但是内存占用为空（不会真正的加载图片）。
有了具备高宽信息的Options，结合上面的inSampleSize算法算出缩小的倍数，我们就能加载本地大图的某个合适大小的缩略图了。

```java
/**
 * 获取缩略图
 * 支持自动旋转
 * 某些型号的手机相机图片是反的，可以根据exif信息实现自动纠正
 * @return
 */
public static Bitmap $thumbnail(String path,
        int maxWidth, int maxHeight, boolean autoRotate) {

    int angle = 0;
    if (autoRotate) {
        angle = ImageLess.$exifRotateAngle(path);
    }

    BitmapFactory.Options options = new BitmapFactory.Options();
    options.inJustDecodeBounds = true;
    // 获取这个图片的宽和高信息到options中, 此时返回bm为空
    Bitmap bitmap = BitmapFactory.decodeFile(path, options);
    options.inJustDecodeBounds = false;
    // 计算缩放比
    int sampleSize = $sampleSize(options, maxWidth, maxHeight);
    options.inSampleSize = sampleSize;
    options.inPreferredConfig = Bitmap.Config.RGB_565;
    options.inPurgeable = true;
    options.inInputShareable = true;

    if (bitmap != null && !bitmap.isRecycled()) {
        bitmap.recycle();
    }
    bitmap = BitmapFactory.decodeFile(path, options);

    if (autoRotate && angle != 0) {
        bitmap = $rotate(bitmap, angle);
    }

    return bitmap;
}
```

系统内置了一个ThumbnailUtils也能生成缩略图，细节上不一样但原理是相同的。

# 8. Matrix变形

学过线性代数或者图像处理的同学们一定深知Matrix的强大，很多常见的图像变换一个Matrix就能搞定，甚至更复杂的也是如此。

> // Matrix matrix = new Matrix();
> // 每一种变化都包括set，pre，post三种，分别为设置、矩阵先乘、矩阵后乘。
> 平移：matrix.setTranslate()
> 缩放：matrix.setScale()
> 旋转：matrix.setRotate()
> 斜切：matrix.setSkew()

下面我举两个例子说明一下。

## 8.1 旋转

借助Matrix的postRotate方法旋转一定角度。

```java
Matrix matrix = new Matrix();
// angle为旋转的角度
matrix.postRotate(angle);
Bitmap rotatedBitmap = Bitmap.createBitmap(originBitmap,
        0,
        0,
        originBitmap.getWidth(),
        originBitmap.getHeight(),
        matrix,
        true);
```

## 8.2 缩放

借助Matrix的postScale方法旋转一定角度。

```java
Matrix matrix = new Matrix();
// scaleX，scaleY分别为为水平和垂直方向上缩放的比例
matrix.postScale(scaleX, scaleY);
Bitmap scaledBitmap = Bitmap.createBitmap(originBitmap,
        0,
        0,
        originBitmap.getWidth(),
        originBitmap.getHeight(),
        matrix,
        true);
```

Bitmap本身也带了一个缩放方法，不过是把bitmap缩放到目标大小，原理也是用Matrix，我们封装一下：

```java
// 水平和宽度缩放到指定大小，注意，这种情况下图片很容易变形
Bitmap scaledBitmap = Bitmap.createScaledBitmap(originBitmap,
        dstWidth,
        dstHeight,
        true);
```

通过组合可以实现更多效果。

# 9. 裁剪

图片的裁剪的应用场景还是很多的：头像剪切，照片裁剪，圆角，圆形等等。

## 9.1 矩形

矩阵形状的裁剪比较简单，直接用createBitmap方法即可：

```java
Canvas canvas = new Canvas(originBitmap);
draw(canvas);
// 确定裁剪的位置和裁剪的大小
Bitmap clipBitmap = Bitmap.createBitmap(originBitmap,
        left, top,
        clipWidth, clipHeight);
```

## 9.2 圆角

对于圆角我们需要借助Xfermode和PorterDuffXfermode，把圆角矩阵套在原Bitmap上取交集得到圆角Bitmap。

```java
// 准备画笔
Paint paint = new Paint();
paint.setAntiAlias(true);

// 准备裁剪的矩阵
Rect rect = new Rect(0, 0,
        originBitmap.getWidth(), originBitmap.getHeight());
RectF rectF = new RectF(new Rect(0, 0,
        originBitmap.getWidth(), originBitmap.getHeight()));

Bitmap roundBitmap = Bitmap.createBitmap(originBitmap.getWidth(),
        originBitmap.getHeight(), Bitmap.Config.ARGB_8888);
Canvas canvas = new Canvas(roundBitmap);
// 圆角矩阵，radius为圆角大小
canvas.drawRoundRect(rectF, radius, radius, paint);

// 关键代码，关于Xfermode和SRC_IN请自行查阅
paint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.SRC_IN));
canvas.drawBitmap(originBitmap, rect, rect, paint);
```

## 9.3 圆形

和上面的圆角裁剪原理相同，不过画的是圆形套在上面。
为了从中间裁剪出圆形，我们需要计算绘制原始Bitmap的left和top值。

```java
int min = originBitmap.getWidth() > originBitmap.getHeight() ?
originBitmap.getHeight() : originBitmap.getWidth();
Paint paint = new Paint();
paint.setAntiAlias(true);
Bitmap circleBitmap = Bitmap.createBitmap(min, min,
    Bitmap.Config.ARGB_8888);
Canvas canvas = new Canvas(circleBitmap);
// 圆形
canvas.drawCircle(min / 2, min / 2, min / 2, paint);
paint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.SRC_IN));

// 居中显示
int left = - (originBitmap.getWidth() - min) / 2;
int top = - (originBitmap.getHeight() - min) / 2;
canvas.drawBitmap(originBitmap, left, top, paint);
```

从圆角、圆形的处理上我们应该能看的出来绘制任意多边形都是可以的。

# 10. 保存Bitmap

很多图片应用都支持裁剪功能，滤镜功能等等，最终还是需要把处理后的Bitmap保存到本地，不然就是再强大的功能也是白忙活了。

```java
public static String $save(Bitmap bitmap,
        Bitmap.CompressFormat format, int quality, File destFile) {
    try {
        FileOutputStream out = new FileOutputStream(destFile);
        if (bitmap.compress(format, quality, out)) {
            out.flush();
            out.close();
        }

        if (bitmap != null && !bitmap.isRecycled()) {
            bitmap.recycle();
        }

        return destFile.getAbsolutePath();
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    }
    return null;
}
```

如果想更稳定或者更简单的保存到SDCard的包名路径下，可以再封装一下：

```java
// 保存到本地，默认路径/mnt/sdcard/[package]/save/，用随机UUID命名文件
public static String $save(Bitmap bitmap,
        Bitmap.CompressFormat format, int quality, Context context) {
    if (!Environment.getExternalStorageState()
            .equals(Environment.MEDIA_MOUNTED)) {
        return null;
    }

    File dir = new File(Environment.getExternalStorageDirectory()
            + "/" + context.getPackageName() + "/save/");
    if (!dir.exists()) {
        dir.mkdirs();
    }
    File destFile = new File(dir, UUID.randomUUID().toString());
    return $save(bitmap, format, quality, destFile);
}
```

# 11. 巨图加载

巨图加载，当然不能使用常规方法，必OOM。
原理比较简单，系统中有一个类BitmapRegionDecoder：

```java
public static BitmapRegionDecoder newInstance(byte[] data, int offset,
        int length, boolean isShareable) throws IOException {
}
public static BitmapRegionDecoder newInstance(
        FileDescriptor fd, boolean isShareable) throws IOException {
}
public static BitmapRegionDecoder newInstance(InputStream is,
        boolean isShareable) throws IOException {
}
public static BitmapRegionDecoder newInstance(String pathName,
        boolean isShareable) throws IOException {
}
```

可以按区域加载:

```java
public Bitmap decodeRegion(Rect rect, BitmapFactory.Options options) {
}
```

微博的大图浏览也是通过这个BitmapRegionDecoder实现的，具体可自行查阅。

# 12. 颜色矩阵ColorMatrix

图像处理其实是一门很深奥的学科，所幸Android提供了颜色矩阵ColorMatrix类，可实现很多简单的特效，以灰阶效果为例子：

```java
Bitmap grayBitmap = Bitmap.createBitmap(originBitmap.getWidth(),
        originBitmap.getHeight(), Bitmap.Config.RGB_565);
Canvas canvas = new Canvas(grayBitmap);
Paint paint = new Paint();
ColorMatrix colorMatrix = new ColorMatrix();
// 设置饱和度为0，实现了灰阶效果
colorMatrix.setSaturation(0);
ColorMatrixColorFilter colorMatrixColorFilter =
        new ColorMatrixColorFilter(colorMatrix);
paint.setColorFilter(colorMatrixColorFilter);
canvas.drawBitmap(originBitmap, 0, 0, paint);
```

除了饱和度，我们还能调整对比度，色相变化等等。

# 13. ThumbnailUtils剖析

ThumbnailUtils是系统提供的一个专门生成缩略图的方法，我专门写了一篇文章分析，内容较多，请移步：[理解ThumbnailUtils](http://www.jayfeng.com/2016/03/16/%E7%90%86%E8%A7%A3ThumbnailUtils/)

# 14. 小结

既然与Bitmap经常打交道，那就把它都理清楚弄明白，这是很有必要的。
难免会有遗漏，欢迎留言，我会酌情补充。

原文链接：http://jayfeng.com/2016/03/22/Android-Bitmap%E9%9D%A2%E9%9D%A2%E8%A7%82/