> 文 /孙有军（网易资深移动应用开发工程师）
>
> 本文主要介绍了关于 Android 界面适配的相关内容，适合 1-3 年的 Android 开发工程师阅读。

## 1. 为什么要适配？

我们先来看一组统计数据和图表。

【2015 设备分布图 】

![img](https://mmbiz.qpic.cn/mmbiz_png/4MXV7svuTWIkh40peIzfaGq7p8GrSMJPVtxEa3dicC7kZcajtibGN3bSwfSHIX4hRQh40vdRdic6sZsMziamuEFpow/640?tp=webp&wxfrom=5&wx_lazy=1)

【2016 设备分布图 】

![img](https://mmbiz.qpic.cn/mmbiz_png/4MXV7svuTWIkh40peIzfaGq7p8GrSMJPAuMNsgaegibicgjqKniaQ0RDS2ga3OicjUC7bjTdwmeCkdIF5gO3oO1ibXw/640?tp=webp&wxfrom=5&wx_lazy=1)

【设备品牌分布 】

![img](https://mmbiz.qpic.cn/mmbiz_png/4MXV7svuTWIkh40peIzfaGq7p8GrSMJP5eqjqQ0rnDPcG7D3R0V6rKJibkfXDo88JrCv8QomDPPUvmKcPTk3yag/640?tp=webp&wxfrom=5&wx_lazy=1)

【屏幕尺寸分布图 】

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

【系统分布图 】

![img](https://mmbiz.qpic.cn/mmbiz_png/4MXV7svuTWIkh40peIzfaGq7p8GrSMJPuKetViaH25RiaDicAmWZePSaS6CQDjmb1EichRSfs643xlu8WR08KraicZA/0?tp=webp&wxfrom=5&wx_lazy=1)

【与 iOS 的对比 】

![img](https://mmbiz.qpic.cn/mmbiz_png/4MXV7svuTWIkh40peIzfaGq7p8GrSMJP91D9ymR9ETqNh5jA1Q7Vnn6HCaCGqicU0ibKxcFiaq78Rp2b4OzEWBAEA/0?tp=webp&wxfrom=5&wx_lazy=1)

从上面几个图就可以看出 android 设备多，品牌多，屏幕尺寸多，还有系统版本分布比较大，碎片化比较严重。这也就是 android 之所以要进行适配的原因。此外，android 的适配包括了系统版本的适配，屏幕尺寸的适配等等。

## 2. 关于适配各种各样的概念

**单位**

px （pixel）：像素，屏幕上的点，最小的独立显示单位，px 均为整数，没有小数。一般都是正方行像素参考链接 

in：表示英寸，每英寸相当于 2.54 厘米。

**概念**

screen size（屏幕尺寸）：屏幕的物理尺寸，表示的是对角线长，如手机屏幕 3.5 寸，就表示对角线长度为 3.5 寸，大概 8.89 厘米。

屏幕分辨率：指屏幕在横边和纵边上的像素点数，单位是 px，比如1920*1080 3：

屏幕像素密度：dpi（dots per inch），每英寸像素点数，比如 120dpi，160dpi，它与屏幕尺寸与屏幕分辨率有关。

**Android 单位与换算**

dp 或者 dip，设备独立像素，即密度无关像素，注意与 dpi 不同，以 160dpi 为基准，1dip=1px 屏幕密码，density = dpi / 160，因此如个屏幕密度为1则1dp = 1px， 如果为2则1dp = 2px 3：sp（scale-independent pixels），字体的推荐单位，可以根据文字首选项进行大小缩放，官方建议最小使用值为12sp，其次尽量使用偶数值。

**dpi**

mdpi → [120dpi ~ 160dpi] 

hdpi → [160dpi ~ 240dpi] 

xdpi → [240dpi ~ 320dpi] 

xxdpi → [320dpi ~ 480dpi] 

xxxdpi → [480dpi ~ 640dpi]

**案例**

比如一个手机屏幕分辨率 480*800 , 屏幕尺寸 3.7in，它的 dpi 是多少，在布局中宽设置 320dp，该宽度为多少像素？ 

理论计算值：先计算出对角线的像素点数 480*480 + 800*800 = 933933，再计算出每英寸的像素点数 933/3.7 = 252dpi，最终计算出的 dpi 为 252。 计算 320dp 对应的像素值：首先计算出屏幕密度，屏幕密度值则相当于 1dp 对应的像素值: 320*（252/160） = 504 px, (手机屏幕才480px，算出的宽度居然比屏幕还宽，如果成立，则说明如果在上诉屏幕上设置为320dp，则有一部分处于屏幕外)

注意：手机上面计算出的 DPI 为理论值，实际上只有 120(low)、160(medium)、240(high)、320(xhigh)等这几种, 因此实际的计算公式为： 320*（240 /160）=480px，与屏幕宽度相同，说明在上诉屏幕设置为320dp，刚好占据整个屏幕。

3

怎么适配？

上面仅仅是知道了为什么适配，和各种各样的概念，那怎么适配？这里主要看手机，不涉及平板适配。

**切图规则**

从上面的概念我们知道，160dpi 的时候 1dp=1px，因此在设计图标时，（mdpi， hdpi，xhpi，xxhpi，xxhpi）的比例值为 2：3：4：6：8。比如系统 icon，mdpi 为 4848，则 xdpi 为 7272，比例值为1.5。 从上图res结构看到有一类 mipmap- *文件夹，这个系统新加为了放置系统图标的文件夹。

**各种图标的尺寸**

以下是官方建议的图标尺寸

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

**宽度值设置**

我们先来看看我们在一个界面中设置一个320dp宽度的一个view。 

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

从图上可以看到不同的屏幕上展示了不同的效果（忽略平板），有的手机上占满了整个屏幕宽度，而有的手机上确只占据屏幕宽度的一部分。因此在视觉出图的时候，比如已800 * 480的尺寸出图，标注占满整个屏幕宽度240dp，则真实展现的效果则会在各个手机上不一致。

因此在开发中，可以采用match_parent来设置占满整个屏幕，如果是其他尺寸，可以采用自适应或者weight来设置view所占用的宽高。

**限定符**

我们从上面看到有 *dpi 作为了限定符，同时还有其他的比如 large限定符，sw 限定符等等限定符，就不一一展开了。

**.9图片**

.9图为系统在图片周围加一个像素的透明边，图片必须要以.9作为描述符。 比如以一个图片来作为背景，如果不是.9图片，则如果内容区域大于图标，则图片会被拉伸。下图四个角都被拉伸了。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

下图设置了图片的拉伸区域，则可以看到图片的四个角都未被拉伸，这样最终呈现的视觉效果就好很多。 

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

*标注位置：相邻两边进行标注，不能只标注一边，也不能只标注相对的两边，如果只标注两边，则拉伸区域为相交部分，文本区域为右边与下边控制，如果不标注则沾满整个宽度。*

同时需要注意的是：图片可以分段标注，但图片最终拉伸的效果与标注的像素点有关，比如同一边上下均被标注且上下像素点比为3：1，则最终拉伸比例也是3：1。

4

更多Tips

- hdpi ，xhdpi 等中的相同图片大小要成比例，这样才能在相同屏幕不同分辨率下展现一致。注意上面的 2：3：4：6：8


- 同一类型，图片大小要一致，可能多人做多个模块，导致切除的同一类型图片相差1，2个像素（可以建立一个资源库，反查已有图标尺寸）。


- 相同图片问题，不同人做不同模块，很多图标都是相同的，由于开发不同，会导致一个包中有相同图片，这样会导致包大小增长。


- 能使用纯色的图片，就让开发尽量使用颜色值，不用切图。


- jpg 与 png 图片相比较，jpg 大小会小很多，如果有大图且没有模糊渐变等要求，尽量采用 jpg 格式。


- 很多简单图片都能用代码实现，比如圆，矩形等，可以让开发用代码实现，减少包的大小。


- 如果包的大小太大时，尽量保证更高尺寸的图片存在，这样低屏幕密度的手机也能展现很清晰的图片，但是如果只有小图，就会放大拉伸，会导致图片变形或者不清晰。


- .9 注意标注拉伸区域与内容区域，与图片外边距的 padding，可以在图上直接标注。（如果内容区域上下距离不相等，再填充多行文字时会造成文字不居中，这时可以直接在图片上空出 padding）


- .9 图片只能拉伸不能压缩，压缩会导致图片变形，因此在作图过程中要确定一下图片的最小尺寸，（比如，给出一个确定高度的矩形区域，里面放置一个初始高度大于矩形的 .9 图片，会导致图片压缩）。


- .9 图片一般只做小尺寸就可以，除非边框有渐变等元素，才做多个尺寸。


- 关于图片标注，美术要转换一下单位，px 转换到相应的 dp 上，开发可以直接使用该数值。


- 关于字体，字体大小 sp，但是如果字体呈现在一个固定高度的矩形框中时，再能调整字体大小的手机上时，可能会展示不全（展现字体的外部图片，背景等尽量不要写死高度）。


- 标注图片时，如果一个 icon 占满整个宽度，则可以不用标注icon宽度尺寸，只需要标注距离边框的尺寸，开发会采用自适应，如果确定宽度，在有的屏幕上只能占据一半宽度，

![img](https://mmbiz.qpic.cn/mmbiz_jpg/4MXV7svuTWIkh40peIzfaGq7p8GrSMJPbV6tF9VicpQn4sicQeZkiack4fRLXoqRLLhZb84FiaKjVHqbz567ClFiciag/0?tp=webp&wxfrom=5&wx_lazy=1)

看到这里，不知道你对自己的Android开发经验是不是有了更多的感悟