本文比较长，希望大家耐心读完。

[Android](http://lib.csdn.net/base/android)中的Veiw从内存中到呈现在UI界面上需要依次经历三个阶段：量算 -> 布局 -> 绘图，关于View的量算、布局、绘图的总体机制可参见博文[《 Android中View的布局及绘图机制》](http://blog.csdn.net/iispring/article/details/49203945)。如果想了解layout布局的细节，可参见博文[《源码解析Android中View的layout布局过程》](http://blog.csdn.net/iispring/article/details/50366021)。量算是布局和绘图的基础，所以量算是很重要的一个环节。本文将从源码角度解析View的量算过程，这其中会涉及某些关键类以及关键方法。

**对View进行量算的目的是让View的父控件知道View想要多大的尺寸。**

## 量算过程概述

如果要进行量算的View是ViewGroup类型，那么ViewGroup会在onMeasure方法内会遍历子View依次进行量算，本文重点说明非ViewGroup的View的量算过程，因为我们一旦了解了非ViewGroup的View的量算过程，ViewGroup的量算理解起来就要简单许多，主要是ViewGroup在其内部对子View再依次执行量算。

- 整个应用量算的起点是ViewRootImpl类，从它开始依次对子View进行量算，如果子View是一个ViewGroup，那么又会遍历该ViewGroup的子View依次进行量算。也就是说，量算会从View树的根结点，纵向递归进行，从而实现自上而下对View树进行量算，直至完成对叶子节点View的量算。
- 那么到底如何对一个View进行量算呢？Android通过调用View的measure()方法对View进行量算，让该View的父控件知道该View想要多大的尺寸空间。
- 具体来说，View的父控件ViewGroup会调用View的measure方法，ViewGroup会将一些宽度和高度的限制条件传递给View的measure方法。
- 在View的measure方法会首先从成员变量中读取以前缓存过的量算结果，如果能找到该缓存值，那么就基本完事了，如果没有找到缓存值，那么measure方法会执行onMeasure回调方法，measure方法会将上述的宽度和高度的限制条件依次传递给onMeasure方法。onMeasure方法会完成具体的量算工作，并将量算的结果通过调用View的setMeasuredDimension方法保存到View的成员变量mMeasuredWidth 和mMeasuredHeight中。
- 量算完成之后，View的父控件就可以通过调用getMeasuredWidth、getMeasuredState、getMeasuredWidthAndState这三个方法获取View的量算结果。

以上就是非ViewGroup类型的View量算的总体过程。

## MeasureSpec简介

上面我们提到ViewGroup在调用View的measure方法时，会传入ViewGroup对View的宽度及高度的限制条件，这是合理的，例如ViewGroup的空间有限，它需要告诉子View要量算的尺寸的上限。

上面提到的尺寸的限制条件就是MeasureSpec，它可以通过一个Int类型的值来表示的，该Int值会同时包含两种信息：mode和size，即模式和尺寸。我们知道[Java](http://lib.csdn.net/base/javase)中Int类型的值是4个字节的，Android会用第一个高位字节存储mode，然后用剩余的三个字节存储size。

View有一个静态内部类MeasureSpec，该类有几个静态方法以及静态常量，我们可以用这些方法将mode和size打包成一个Int值或者是从一个Int值中解析出mode和size。

假设我们已有了一个包含MeasureSpec信息的Int值measureSpec，那么

- 通过调用MeasureSpec.getSize(int measureSpec)即可从measureSpec解析出三个字节所包含的尺寸size信息，该方法返回Int类型，也就是说我们得到的size实际上就是对原有的measureSpec的高位字节的8个二进制位都设置为0，该方法的返回值size虽然也是4个字节的Int值，但是已经完全不包含mode信息。
- 通过调用MeasureSpec.getMode(int measureSpec)即可从measureSpec解析出高位字节所包含的模式mode信息，该方法返回Int类型，也就是说我们得到的mode实际上对原有的measureSpec的低位的三个字节的24个二进制码都设置为0，该方法的返回值mode虽然也是4个字节的Int值，但是已经完全不包含size信息。

对于尺寸size，我们很好理解，比如表示某个宽度值或者表示某个高度值。那么mode是什么呢？

mode的取值有三种，分别是：

- MeasureSpec.AT_MOST，即0x80000000，该值表示View最大可以取其父ViewGroup给其指定的尺寸，例如现在有个Int值widthMeasureSpec，ViewGroup将其传递给了View的measure方法，如果widthMeasureSpec中的mode值是AT_MOST，size是200，那么表示View能取的最大的宽度是200。
- MeasureSpec.EXACTLY，即0x40000000，该值表示View必须使用其父ViewGroup指定的尺寸，还是以widthMeasureSpec为例，如果其mode值是EXACTLY，size是200，那么表示View的宽度必须是200，不多不少才行。
- MeasureSpec.UNSPECIFIED，即0x00000000，该值表示View的父ViewGroup没有给View在尺寸上设置限制条件，这种情况下View可以忽略measureSpec中的size，View可以取自己想要的值作为量算的尺寸。

更多信息可参考API文档 [android/view/View.MeasureSpec](http://blog.csdn.net/iispring/article/details/developer.android.com/reference/android/view/View.MeasureSpec.html)。

## measure方法

measure()的方法签名是`public final void measure(int widthMeasureSpec, int heightMeasureSpec)`。

当View的父控件ViewGroup对View进行量算时，会调用View的measure方法，ViewGroup会传入widthMeasureSpec和heightMeasureSpec，分别表示父控件对View的宽度和高度的一些限制条件。

measure方法的源码如下所示：

```java
public final void measure(int widthMeasureSpec, int heightMeasureSpec) {
    //首先判断当前View的layoutMode是不是特例LAYOUT_MODE_OPTICAL_BOUNDS
    boolean optical = isLayoutModeOptical(this);
    if (optical != isLayoutModeOptical(mParent)) {
        //LAYOUT_MODE_OPTICAL_BOUNDS是特例情况，比较少见
        Insets insets = getOpticalInsets();
        int oWidth  = insets.left + insets.right;
        int oHeight = insets.top  + insets.bottom;
        widthMeasureSpec  = MeasureSpec.adjust(widthMeasureSpec,  optical ? -oWidth  : oWidth);
        heightMeasureSpec = MeasureSpec.adjust(heightMeasureSpec, optical ? -oHeight : oHeight);
    }

    //根据widthMeasureSpec和heightMeasureSpec计算key值，我们在下面用key值作为键，缓存我们量算的结果
    long key = (long) widthMeasureSpec << 32 | (long) heightMeasureSpec & 0xffffffffL;

    //mMeasureCache是LongSparseLongArray类型的成员变量，
    //其缓存着View在不同widthMeasureSpec、heightMeasureSpec下量算过的结果
    //如果mMeasureCache为空，我们就新new一个对象赋值给mMeasureCache
    if (mMeasureCache == null) mMeasureCache = new LongSparseLongArray(2);

    //mOldWidthMeasureSpec和mOldHeightMeasureSpec分别表示上次对View进行量算时的widthMeasureSpec和heightMeasureSpec
    //执行View的measure方法时，View总是先检查一下是不是真的有必要费很大力气去做真正的量算工作
    //mPrivateFlags是一个Int类型的值，其记录了View的各种状态位
    //如果(mPrivateFlags & PFLAG_FORCE_LAYOUT) == PFLAG_FORCE_LAYOUT，
    //那么表示当前View需要强制进行layout（比如执行了View的forceLayout方法），所以这种情况下要尝试进行量算
    //如果新传入的widthMeasureSpec/heightMeasureSpec与上次量算时的mOldWidthMeasureSpec/mOldHeightMeasureSpec不等，
    //那么也就是说该View的父ViewGroup对该View的尺寸的限制情况有变化，这种情况下要尝试进行量算
    if ((mPrivateFlags & PFLAG_FORCE_LAYOUT) == PFLAG_FORCE_LAYOUT ||
            widthMeasureSpec != mOldWidthMeasureSpec ||
            heightMeasureSpec != mOldHeightMeasureSpec) {

        //通过按位操作，重置View的状态mPrivateFlags，将其标记为未量算状态
        mPrivateFlags &= ~PFLAG_MEASURED_DIMENSION_SET;

        //对阿拉伯语、希伯来语等从右到左书写、布局的语言进行特殊处理
        resolveRtlPropertiesIfNeeded();

        //在View真正进行量算之前，View还想进一步确认能不能从已有的缓存mMeasureCache中读取缓存过的量算结果
        //如果是强制layout导致的量算，那么将cacheIndex设置为-1，即不从缓存中读取量算结果
        //如果不是强制layout导致的量算，那么我们就用上面根据measureSpec计算出来的key值作为缓存索引cacheIndex。
        int cacheIndex = (mPrivateFlags & PFLAG_FORCE_LAYOUT) == PFLAG_FORCE_LAYOUT ? -1 :
                mMeasureCache.indexOfKey(key);

        //sIgnoreMeasureCache是一个boolean类型的成员变量，其值是在View的构造函数中计算的，而且只计算一次
        //一些老的App希望在一次layou过程中，onMeasure方法总是被调用，
        //具体来说其值是通过如下计算的: sIgnoreMeasureCache = targetSdkVersion < KITKAT;
        //也就是说如果targetSdkVersion的API版本低于KITKAT，即API level小于19，那么sIgnoreMeasureCache为true

        if (cacheIndex < 0 || sIgnoreMeasureCache) {
            //如果运行到此处，表示我们没有从缓存中找到量算过的尺寸或者是sIgnoreMeasureCache为true导致我们要忽略缓存结果
            //此处调用onMeasure方法，并把尺寸限制条件widthMeasureSpec和heightMeasureSpec传入进去
            //onMeasure方法中将会进行实际的量算工作，并把量算的结果保存到成员变量中
            onMeasure(widthMeasureSpec, heightMeasureSpec);
            //onMeasure执行完后，通过位操作，重置View的状态mPrivateFlags，将其标记为在layout之前不必再进行量算的状态
            mPrivateFlags3 &= ~PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
        } else {
            //如果运行到此处，那么表示当前的条件允许View从缓存成员变量mMeasureCache中读取量算过的结果
            //用上面得到的cacheIndex从缓存mMeasureCache中取出值，不必在调用onMeasure方法进行量算了
            long value = mMeasureCache.valueAt(cacheIndex);
            //一旦我们从缓存中读到值，我们就可以调用setMeasuredDimensionRaw方法将当前量算的结果到成员变量中
            setMeasuredDimensionRaw((int) (value >> 32), (int) value);
            mPrivateFlags3 |= PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
        }

        //如果我们自定义的View重写了onMeasure方法，但是没有调用setMeasuredDimension()方法，
        //那么此处就会抛出异常，提醒开发者在onMeasure方法中调用setMeasuredDimension()方法
        //Android是如何知道我们有没有在onMeasure方法中调用setMeasuredDimension()方法的呢？
        //方法很简单，还是通过解析状态位mPrivateFlags。
        //setMeasuredDimension()方法中会将mPrivateFlags设置为PFLAG_MEASURED_DIMENSION_SET状态，即已量算状态，
        //此处就检查mPrivateFlags是否含有PFLAG_MEASURED_DIMENSION_SET状态即可判断setMeasuredDimension是否被调用
        if ((mPrivateFlags & PFLAG_MEASURED_DIMENSION_SET) != PFLAG_MEASURED_DIMENSION_SET) {
            throw new IllegalStateException("View with id " + getId() + ": "
                    + getClass().getName() + "#onMeasure() did not set the"
                    + " measured dimension by calling"
                    + " setMeasuredDimension()");
        }

        mPrivateFlags |= PFLAG_LAYOUT_REQUIRED;
    }

    //mOldWidthMeasureSpec和mOldHeightMeasureSpec保存着最近一次量算时的MeasureSpec，
    //在量算完成后将这次新传入的MeasureSpec赋值给它们
    mOldWidthMeasureSpec = widthMeasureSpec;
    mOldHeightMeasureSpec = heightMeasureSpec;

    //最后用上面计算出的key作为键，量算结果作为值，将该键值对放入成员变量mMeasureCache中，
    //这样就实现了对本次量算结果的缓存，以便在下次measure方法执行的时候，有可能将其从中直接读出，
    //从而省去实际量算的步骤
    mMeasureCache.put(key, ((long) mMeasuredWidth) << 32 |
            (long) mMeasuredHeight & 0xffffffffL);
}
```

上面的注释对每行代码都进行了详细的说明，如果大家仔细读了的话，相信能一目了然，这里根据上面的注释简单总结一下measure方法都干了什么事：

- 首先，我们要知道并不是只要View的measure方法执行的时候View就一定要傻傻的真的去做量算工作，View也喜欢偷懒，如果View发现没有必要去量算的话，那它就不会真的去做量算的工作。
- 具体来说，View先查看是不是要强制量算以及这次measure中传入的MeasureSpec与上次量算的MeasureSpec是否相同，如果不是强制量算或者MeasureSpec与上次的量算的MeasureSpec相同，那么View就不必真的去量算了。
- 如果不满足上述条件，View就考虑去做量算工作。但是在量算之前，View还想偷懒，它会以MeasureSpec计算出的key值作为键，去成员变量mMeasureCache中查找是否缓存过对应key的量算结果，如果能找到，那么就简单调用一下setMeasuredDimensionRaw方法，将从缓存中读到的量算结果保存到成员变量mMeasuredWidth和mMeasuredHeight中。
- 如果不能从mMeasureCache中读到缓存过的量算结果，那么这次View就真的不能再偷懒了，只能乖乖地调用onMeasure方法去完成实际的量算工作，并且将尺寸限制条件widthMeasureSpec和heightMeasureSpec传递给onMeasure方法。关于onMeasure方法，我们会在下面详细介绍。
- 不论上面代码走了哪个判断的分支，最终View都会得到量算的结果，并且将结果缓存到成员变量mMeasureCache中，以便下次执行measure方法时能够从其中读取缓存值。
- 需要说明的是，View有一个成员变量mPrivateFlags，用以保存View的各种状态位，在量算开始前，会将其设置为未量算状态，在量算完成后会将其设置为已量算状态。

## onMeasure方法

我们在上面提到，当View在measure方法中发现不得不进行实际的量算工作时，将会调用onMeasure方法，并且将尺寸限制条件widthMeasureSpec和heightMeasureSpec作为参数传递给onMeasure方法。View的onMeasure方法不是空方法，它提供了一个默认的具体实现。 
onMeasure方法的代码如下：

```java
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    //onMeasure调用了setMeasuredDimension方法，
    //setMeasuredDimension又需要调用getDefaultSize方法，
    //getDefaultSize又需要调用getSuggestedMinimumWidth和getSuggestedMinimumHeight方法
    setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
            getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
}
```

我们发现onMeasure方法中会调用setMeasuredDimension方法，setMeasuredDimension又需要调用getDefaultSize方法，getDefaultSize又需要调用getSuggestedMinimumWidth和getSuggestedMinimumHeight方法，即 
setMeasuredDimension -> getDefaultSize -> getSuggestedMinimumWidth/Height

那我们就先研究getSuggestedMinimumWidth/Height，然后再依次研究getDefaultSize和setMeasuredDimension，这样就能把onMeasure方法搞明白了。其实getSuggestedMinimumWidth和getSuggestedMinimumHeight的实现逻辑基本一样，我们此处只研究getSuggestedMinimumWidth方法即可。

## getSuggestedMinimumWidth方法

getSuggestedMinimumWidth用于返回View推荐的最小宽度，其代码如下所示：

```java
protected int getSuggestedMinimumWidth() {
    //如果没有给View设置背景，那么就返回View本身的最小宽度mMinWidth
    //如果给View设置了背景，那么就取View本身最小宽度mMinWidth和背景的最小宽度的最大值
    return (mBackground == null) ? mMinWidth : max(mMinWidth, mBackground.getMinimumWidth());
}
```

- 如果没有给View设置背景，那么就返回View本身的最小宽度mMinWidth
- 如果给View设置了背景，那么就取View本身最小宽度mMinWidth和背景的最小宽度的最大值

那你可能有疑问，View中保存的最小宽度mMinWidth的值是从哪来的呢？实际上有两种办法给View设置最小宽度。

- 第一种情况是，mMinWidth是在View的构造函数中被赋值的，View通过读取XML中定义的minWidth的值来设置View的最小宽度mMinWidth，以下代码片段是View构造函数中解析minWidth的部分：

  ```java
  //遍历到XML中定义的minWith属性
  case R.styleable.View_minWidth:
  //读取XML中定义的属性值作为mMinWidth，如果XML中未定义，则设置为0
  mMinWidth = a.getDimensionPixelSize(attr, 0);
  break;
  ```

- 第二种情况是调用View的setMinimumWidth方法给View的最小宽度mMinWidth赋值，setMinimumWidth方法的代码如下所示：

  ```java
  public void setMinimumWidth(int minWidth) {
      mMinWidth = minWidth;
      requestLayout();
  }
  ```

这样我们就搞明白了getSuggestedMinimumWidth方法是怎么执行的了，getSuggestedMinimumHeight方法与其逻辑完全一致，只不过是把宽度换成了高度，在此就不再赘述了。

## getDefaultSize

我们在onMeasure方法中发现，onMeasure会执行以下两行代码：getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec) 
getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec)

我们已经研究了getSuggestedMinimumWidth/Height，知道其会返回View的最小宽度和高度，现在我们开始研究getDefaultSize方法。

Android会将View想要的尺寸以及其父控件对其尺寸限制信息measureSpec传递给getDefaultSize方法，该方法要根据这些综合信息计算最终的量算的尺寸。

其源码如下所示：

```java
public static int getDefaultSize(int size, int measureSpec) {
    //size表示的是View想要的尺寸信息，比如最小宽度或最小高度
    int result = size;
    //从measureSpec中解析出specMode信息
    int specMode = MeasureSpec.getMode(measureSpec);
    //从measureSpec中解析出specSize信息，不要将specSize与上面的size变量搞混
    int specSize = MeasureSpec.getSize(measureSpec);

    switch (specMode) {
    //如果mode是UNSPECIFIED，表示View的父ViewGroup没有给View在尺寸上设置限制条件
    case MeasureSpec.UNSPECIFIED:
        //此处当mode是UNSPECIFIED时，View就直接用自己想要的尺寸size作为量算的结果
        result = size;
        break;
    //如果mode是UNSPECIFIED，那么表示View最大可以取其父ViewGroup给其指定的尺寸
    //如果mode是EXACTLY，那么表示View必须使用其父ViewGroup指定的尺寸
    case MeasureSpec.AT_MOST:
    case MeasureSpec.EXACTLY:
        //此处mode是UNSPECIFIED或EXACTLY时，View就用其父ViewGroup指定的尺寸作为量算的结果
        result = specSize;      
        break;
    }
    return result;
}
```

通过以上代码，我们就会发现View的父ViewGroup传递给View的限制条件measureSpec的作用在该方法中体现的淋漓尽致。

- 首先根据measuredSpec解析出对应的specMode和specSize
- 当mode是UNSPECIFIED时，View就直接用自己想要的尺寸size作为量算的结果
- 当mode是UNSPECIFIED或EXACTLY时，View就用其父ViewGroup指定的尺寸作为量算的结果

最终，View会根据measuredSpec限制条件，得到最终的量算的尺寸。

这样在onMeasure方法中， 
当执行getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec)时，我们就得到了最终量算到的宽度值； 
当执行getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec)时，我们就得到了最终量算到的高度值。

## setMeasuredDimension

在前面我们研究onMeasure方法时就已经看到setMeasuredDimension会调用getDefaultSize方法，会将已经量算到的宽度值和高度值作为参数传递给setMeasuredDimension方法，我们研究一下该方法。

其源码如下所示：

```java
protected final void setMeasuredDimension(int measuredWidth, int measuredHeight) {
    boolean optical = isLayoutModeOptical(this);
    if (optical != isLayoutModeOptical(mParent)) {
        //layoutMode是LAYOUT_MODE_OPTICAL_BOUNDS的特殊情况，我们不考虑
        Insets insets = getOpticalInsets();
        int opticalWidth  = insets.left + insets.right;
        int opticalHeight = insets.top  + insets.bottom;

        measuredWidth  += optical ? opticalWidth  : -opticalWidth;
        measuredHeight += optical ? opticalHeight : -opticalHeight;
    }
    //最终调用setMeasuredDimensionRaw方法，将量算结果传入进去
    setMeasuredDimensionRaw(measuredWidth, measuredHeight);
}
```

该方法会在开始判断layoutMode是不是LAYOUT_MODE_OPTICAL_BOUNDS的特殊情况，这种特例很少见，我们直接忽略掉。

setMeasuredDimension方法最后将量算的结果传递给方法setMeasuredDimensionRaw，我们再研究一下setMeasuredDimensionRaw这方法。

## setMeasuredDimensionRaw

setMeasuredDimensionRaw接收两个参数，分别是已经量算完成的宽度和高度。

其源码如下所示：

```java
private void setMeasuredDimensionRaw(int measuredWidth, int measuredHeight) {
    //将量算完成的宽度measuredWidth保存到View的成员变量mMeasuredWidth中
    mMeasuredWidth = measuredWidth;
    //将量算完成的高度measuredHeight保存到View的成员变量mMeasuredHeight中
    mMeasuredHeight = measuredHeight;
    //最后将View的状态位mPrivateFlags设置为已量算状态
    mPrivateFlags |= PFLAG_MEASURED_DIMENSION_SET;
}
```

我们发现，在该方法中做了三件事：

- 将量算完成的宽度measuredWidth保存到View的成员变量mMeasuredWidth中
- 将量算完成的高度measuredHeight保存到View的成员变量mMeasuredHeight中
- 最后将View的状态位mPrivateFlags设置为已量算状态

## 量算完成的尺寸的state

至此，View的量算过程就完成了，但是View的父ViewGroup如何读取到View量算的结果呢？

为此，View提供了三组方法，分别是： 

1. getMeasuredWidth和getMeasuredHeight方法 
2. getMeasuredWidthAndState和getMeasuredHeightAndState方法 
3. getMeasuredState方法

有些人可能会纳闷，只要有了第一组方法不就行了吗？后面那两组方法有啥用？

此处我们要再仔细研究一下View中保存量算结果的成员变量mMeasuredWidth和mMeasuredHeight，下面的讨论我们都只讨论宽度，理解了宽度的处理方式，高度也是完全一样的。

mMeasuredWidth是一个Int类型的值，其是由4个字节组成的。

我们先假设mMeasuredWidth只存储了量算完成的宽度信息，而且View的父ViewGroup可以通过相关方法得到该值。但是存在这样一种情况：View在量算时，父ViewGroup给其传递的widthMeasureSpec中的specMode的值是AT_MOST，specSize是100，但是View的最小宽度是200，显然父ViewGroup指定的specSize不能满足View的大小，但是由于specMode的值是AT_MOST，View在getDefaultSize方法中不得不妥协，只能含泪将量算的最终宽度设置为100。然后其父ViewGroup通过某些方法获取到该View的量算宽度为100时，ViewGroup以为子View只需要100就够了，最终给了子View宽度为100的空间，这就导致了在UI界面上View特别窄，用户体验也就不好。

Android为让其View的父控件获取更多的信息，就在mMeasuredWidth上下了很大功夫，虽然是一个Int值，但是想让它存储更多信息，具体来说就是把mMeasuredWidth分成两部分：

- 其高位的第一个字节为第一部分，用于标记量算完的尺寸是不是达到了View想要的宽度，我们称该信息为量算的state信息。
- 其低位的三个字节为第二部分，用于存储实际的量算到的宽度。

由此我们可以看出Android真是物尽其用，一个变量能包含两个信息，这个有点类似于measureSpec的道理，但是二者又有不同：

- measureSpec是将限制条件mode从ViewGroup传递给其子View。
- mMeasuredWidth、mMeasuredHeight是将带有量算结果的state标志位信息从View传递给其父ViewGroup。

那么你可能会问，在本文中我们没看到对mMeasuredWidth的高位字节进行特殊处理啊？我们下面看一下View中的resolveSizeAndState方法。

## resolveSizeAndState

resolveSizeAndState方法与getDefaultSize方法类似，其内部实现的逻辑是一样的，但是又有区别，getDefaultSize仅仅返回最终量算的尺寸信息，但resolveSizeAndState除了返回最终尺寸信息还会有可能返回量算的state标志位信息。

resolveSizeAndState方法的源码如下所示：

```java
public static int resolveSizeAndState(int size, int measureSpec, int childMeasuredState) {
    final int specMode = MeasureSpec.getMode(measureSpec);
    final int specSize = MeasureSpec.getSize(measureSpec);
    final int result;
    switch (specMode) {
        case MeasureSpec.AT_MOST:
            if (specSize < size) {
                //当specMode为AT_MOST，并且父控件指定的尺寸specSize小于View自己想要的尺寸时，
                //我们就会用掩码MEASURED_STATE_TOO_SMALL向量算结果加入尺寸太小的标记
                //这样其父ViewGroup就可以通过该标记其给子View的尺寸太小了，
                //然后可能分配更大一点的尺寸给子View
                result = specSize | MEASURED_STATE_TOO_SMALL;
            } else {
                result = size;
            }
            break;
        case MeasureSpec.EXACTLY:
            result = specSize;
            break;
        case MeasureSpec.UNSPECIFIED:
        default:
            result = size;
    }
    return result | (childMeasuredState & MEASURED_STATE_MASK);
}
```

当specMode为AT_MOST，并且父控件指定的尺寸specSize小于View自己想要的尺寸时，我们就会用掩码MEASURED_STATE_TOO_SMALL向量算结果加入尺寸太小的标记，这样其父ViewGroup就可以通过该标记其给子View的尺寸太小了，然后可能分配更大一点的尺寸给子View。

getDefaultSize方法只是onMeasure方法中获取最终尺寸的默认实现，其返回的信息比resolveSizeAndState要少，那么什么时候才会调用resolveSizeAndState方法呢？ 主要有两种情况：

- Android中的许多layout类都调用了resolveSizeAndState方法，比如LinearLayout在量算过程中会调用resolveSizeAndState方法而非getDefaultSize方法。
- 我们自己在实现自定义的View或ViewGroup时，我们可以重写onMeasure方法，并在该方法内调用resolveSizeAndState方法。

## getMeasuredXXX系列方法

现在我们再回过头来看以下三组方法：

- getMeasuredWidth和getMeasuredHeight方法 
  该组方法只返回量算结果中的的尺寸信息，去掉了高位字节的state信息，以getMeasuredWidth方法为例，其源码如下：

  ```java
  public final int getMeasuredWidth() {
      //MEASURED_SIZE_MASK的值为0x00ffffff，用mMeasuredWidth与掩码MEASURED_SIZE_MASK进行按位与运算，
      //可以将返回值中的高位字节的8个bit位全置为0，从而去掉了高位字节的state信息
      return mMeasuredWidth & MEASURED_SIZE_MASK;
  }
  ```

  MEASURED_SIZE_MASK的值为0x00ffffff，用mMeasuredWidth与掩码MEASURED_SIZE_MASK进行按位与运算，可以将返回值中的高位字节的8个bit位全置为0，从而去掉了高位字节的state信息

- getMeasuredWidthAndState和getMeasuredHeightAndState方法 
  该组方法返回的量算结果中同时包含尺寸和state信息（如果state存在的话），以getMeasuredWidthAndState方法为例，其源码如下所示：

  ```java
  public final int getMeasuredWidthAndState() {
      //该方法直接返回成员变量mMeasuredWidth，因为mMeasuredWidth本身已经包含了尺寸以及可能的state信息
      return mMeasuredWidth;
  }
  ```

  该方法直接返回成员变量mMeasuredWidth，因为mMeasuredWidth本身已经包含了尺寸以及可能的state信息

- getMeasuredState方法 
  该方法返回的Int值中同时包含宽度量算的state以及高度量算的state，不包含任何的尺寸信息，其源码如下所示：

  ```java
  public final int getMeasuredState() {
      //将宽度量算的state存储在Int值的第一个字节中，即高位首字节
      //将高度量算的state存储在Int值的第三个字节中
      return (mMeasuredWidth&MEASURED_STATE_MASK)
              | ((mMeasuredHeight>>MEASURED_HEIGHT_STATE_SHIFT)
                      & (MEASURED_STATE_MASK>>MEASURED_HEIGHT_STATE_SHIFT));
  }
  ```

  我们简单分析一下以上代码：

- 掩码MEASURED_STATE_MASK的值为常量0xff000000，其高位字节的8个bit位全为1，剩余低位字节的三个字节的24个bit位全为0

- MEASURED_HEIGHT_STATE_SHIFT的值为常量16

- 当执行(mMeasuredWidth&MEASURED_STATE_MASK)时，将mMeasuredWidth与MEASURED_STATE_MASK进行按位与操作，该表达式的值高位字节保留了量算后宽度的state，过滤掉了其低位三个字节所存储的宽度size

- 由于我们已经用高位首字节存储了量算后宽度的state，所以高度的state就不能存储在高位首字节了。Android打算把它存储在第三个字节中。(mMeasuredHeight>>MEASURED_HEIGHT_STATE_SHIFT)表示将mMeasuredHeight向右移16位，这样高度的state字节就从原来的第一个字节右移动到了第三个字节，由于高度的state向右移动了，所以其对应的掩码也有相应移动。(MEASURED_STATE_MASK>>MEASURED_HEIGHT_STATE_SHIFT)表示state的掩码也从第一个字节右移16位到了第三个字节，即掩码从0xff000000变成了0x0000ff00。然后用右移后的state与右移后的掩码执行按位与操作，这样就在第三个字节保留了高度的state信息，并且过滤掉了第1、2、4字节中的信息，即将这三个字节中的24个bit位置为0。

- 最后，将我们得到的宽度的state与高度的state进行按位或操作，这样就将宽度和高度的state都保存在一个Int值中：第一个字节存储宽度的state，第三个字节存储高度的state。

## 总结

至此，View中量算的关键类以及方法我们基本都涉及到了，我们发现View的measure方法还是比较聪明的，知道如何偷懒利用以前量算过的数据，如果情况有变，那么就调用onMeasure方法进行实际的量算工作，在onMeasure中，View要根据父ViewGroup给其传递进来的widthMeasureSpec和heightMeasureSpec，并结合View自身想要的尺寸，综合考虑，计算出最终的量算的宽度和高度，并存储到相应的成员变量中，这才标志着该View量算有效的完成了，如果没有将值存入到成员变量中，View会抛出异常。在该成员变量中有可能也存储了量算过程中的state信息。由于View的measure已经实现了很多逻辑判断，所以我们在自定义View或ViewGroup时，都不应该重写measure方法，而应该重写onMeasure方法，在其中实现我们自己的量算逻辑。

啰啰嗦嗦写了将近一天，终于写完了，希望本文对大家深入理解Android中的量算过程有所帮助，感谢大家耐心读完本文。

相关阅读： 
[Android相关博文整理汇总](http://blog.csdn.net/iispring/article/details/47690011) 
[Android中View的布局及绘图机制](http://blog.csdn.net/iispring/article/details/49203945) 
[源码解析Android中View的layout布局过程](http://blog.csdn.net/iispring/article/details/50366021)