

# **1. 什么是View**
在Android的官方文档中是这样描述的：表示了用户界面的基本构建模块。一个View占用了屏幕上的一个矩形区域并且负责界面绘制和事件处理。

手机屏幕上所有看得见摸得着的都是View。这一点对所有图形系统来说都一样，例如ios的UIView。

# **2. View和Activity的区别**

我们之前学习过android的四大组件，Activity是四大组件中唯一一个用来和用户进行交互的组件。可以说Activity就是android的视图层。

如果再细化，Activity相当于视图层中的控制层，是用来控制和管理View的，真正用来显示和处理事件的实际上是View。

每个Activity内部都有一个Window对象， Window对象包含了一个DecorView(实际上就是FrameLayout)，我们通过setContentView给Activity设置显示的View实际上都是加到了DecorView中。

# **3. View种类**

android提供了种类丰富的View来应对各种需求，例如提供文字显示的TextView，提供点击事件的Button，提供图片显示的ImageView，还有各种布局文件，例如Relativilayout，Linearlayout等等。他们都是继承自View。

![view](http://img.blog.csdn.net/20170302162501877?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXhpMjk1MzA5MDY2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

# **4. ViewGroup**

ViewGroup继承自View，并实现了两个接口ViewParent和ViewManager。

ViewManager提供了三个抽象方法addView，removeView，updateViewLayout。用来添加、删除、更新布局。

ViewParent主要提供了一系列操作子View的方法例如焦点的切换，显示区域的控制等等。

# **5. 为什么要有ViewGroup？**

实际上所有的事情View都能做，包括显示复杂的界面，我们只需要设计一个复杂的View即可。例如短信通知的icon，一个可以显示图片又可以显示文字的View，我们后期学习了View的draw方法后，可以轻松的设计一个View来达到这个效果，但是这样不仅复杂，而且重用性较差，还会因为一点小改动而重复的创造轮子，这显然不符合程序员偷懒的原则，所以我们可以完全把ImageView和TextView组合到一起就可以了，这个时候我们就需要一个容器，ViewGroup，来装这两个View。

ViewGroup和View最大的不同是可以组合多个View，那么多个View在一起，该如何摆放，这就是ViewGroup需要解决的问题。

# **6. View树**

我们看到的界面，都是以一个ViewGroup作为根View，通过往ViewGroup中添加子View（可以是View，也可以是ViewGroup），来组合出各具特色的界面。

这种从根到叶的组合方式，我们可以看做成一个View树。（类似于XML），而View的显示和事件处理，都是依赖于这个View树。

绘制和事件处理的起始点，都是从根View开始一级一级的往下传递。我们从任意一层发起绘制，都将反馈到根View，然后再从上往下传递。

之前我们说过根View就是Window中的DecorView，也就是一个FrameLayout。

## **6.1 View树示意图**

![view](http://img.blog.csdn.net/20170302163002515?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXhpMjk1MzA5MDY2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

对SystemUI，也就是我们常说的StatusBar显示在哪儿呢，其实SystemUI是一个单独的App，随着系统启动而启动，将会启动一个系统级服务，接收我们提交的通知，该应用也会有一个window，并且级别比我们普通应用的window要高，所以会显示在我们的应用的外面，只不过该window的高度比较小。

# **7. View的测量、布局、绘制过程**

整个android系统 CS架构，view被展示到界面上需要经过3个步骤

- 需要花多大：measure --> onMeasure --> setMeasuredDimension
- 画在什么地方：layout --> setFrame --> onLayout
- 怎么画：draw --> > onDraw --> dispatchDraw

## **7.1 显示一个View需要经过哪些步骤**

- Measure测量一个View的大小
- Layout摆放一个View的位置
- Draw画出View的显示内容

其中measure和layout方法都是final的，无法重写，虽然draw不是final的，但是也不建议重写该方法。这三个方法都已经写好了View的逻辑，如果我们想实现自身的逻辑，而又不破坏View的工作流程，可以重写onMeasure、onLayout、onDraw方法。

## **7.2 如何发起一个View树的测量/布局/绘制流程**

通过调用requestLayout/requestFocus都将发起一个View树的测量。测量完毕后会进行布局，布局完毕后就会绘制。

如果View的大小没有发生改变，布局也没有变化，只是显示的内容发生了变化，则可以通过invalidate来请求绘制，此时将不会测量和布局，直接从绘制开始。

## **7.3 View内部的mPrivateFlags变量**

View中有一个私有int变量mPrivateFlags，用于保存View的状态，int型32位，通过0/1可以保存32个状态的true或者false，采用这种方式可以有效的减少内存占用，提高运算效率。

当某一个View发起了测量请求时，将会把mPrivateFlags中的某一位从0变为1，同时请求父View，父View也会把自身的该值从0变为1，同时也将会把其他子View的值从0变为1。这样一层一层传递，最终传到到DecorView，DecorView的父View是ViewRoot，所以最终都将由ViewRoot来进行处理。

ViewRoot收到请求后，将会从上至下开始遍历，检查标记，只要有相对应的标记就执行测量/布局/绘制

当Activity被创建时，会相应的创建一个Window对象，Window对象创建时会获取应用的WindowManager（注意，这是应用的窗口管理者，不是系统的）。

Activity被创建后，会调用Activity的onCreate方法。我们通过设置setContentView就会调用到Window中的setContextView，从而初始化DecorView。

所以我们需要隐藏标题栏什么的，都需要在DecorView初始化之前进行设置。

DecorView初始化之后将会被添加到WindowManager中，同时WindowManager中会为新添加的DecorView创建一个对应的ViewRoot，并把DecorView设置给ViewRoot。

所以根View就是DecorView，因为DecorView的父亲是ViewRoot，实现自ViewParent接口，但是没有继承自View，所以根本不是一个View。

从系统的命名来看，WindowManger继承自ViewManager，而添加到WindowManager中的是DecorView，不是Window，都说明了其实真正意义上的window就是View。

在ViewRoot的构造方法中会通过getWindowSession来获取WindowManagerService系统服务的远程对象(这才是系统级的)。

当ViewRoot的setView方法中将会调用requestLayout进行第一次视图测量请求。同时sWindowSession.add自身内部的一个W对象，以此达到和WindowManagerService的关联。

W是一个Binder对象。可以实现跨进程的通信了，并且是一个双方都掌握着主动调用的跨进程通信方式。

## **7.4 常用的标记位**

- FORCE_LAYOUT 请求绘制，将从measure开始，，并增加LAYOUT_REQUIRED标记
- 持有LAYOUT_REQUIRED标记的View将会被执行layout，完毕后会去掉LAYOUT_REQUIRED和FORCE_LAYOUT 
- DRAWN带有该标签的将不会被draw，注意，这和上面两个不一致，当draw完毕后会加上该标签，当没有该标签才会被draw。

还有一些其他的标记位，大家可以自行阅读源码。

## **7.5 测量/布局/绘制流程**

![view](http://img.blog.csdn.net/20170302164157729?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXhpMjk1MzA5MDY2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

测量事件最终传递到decorView的父亲ViewRoot那里，由它的函数performTraversals来执行，听名字就知道是执行遍历了。

首先它会检测之前设置的标记为来确定是否需要测量大小，是，就会直接执行decorView的measture方法，该方法内部会测量完自身后，将会继续遍历所有子View，直到每一个设置有标记的子View都测量完。

然后它会检测是否需要布局，是，将会执行decorView的layout方法进行，该方法内部也会遍历所有设置有标记位子View。

# **8. measure 测量**
## **8.1 测量流程**

![measure](http://img.blog.csdn.net/20170226131046026?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXhpMjk1MzA5MDY2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

测量View是在measure()方法中，而measure()方法是final修饰的，不允许重写，但是在measure()方法中回调了onMeasure()方法，所以我们自定义View的时候需要重写onMeasure()方法，在该方法中实现测量的逻辑

- 如果是普通View，则直接通过setMeasureDimension()方法设置大小即可
- 如果是ViewGroup，则需要循环遍历所有子View，调用子View的measure()方法，测量每个子View的大小，等所有的子View都测量完毕，最后通过setMeasureDimension()设置ViewGroup自身的大小

## 8.2 LayoutParams

每个View都包含一个ViewGroup.LayoutParams类或者其派生类，LayoutParams中包含了View和它的父View之间的关系，而View大小正是View和它的父View共同决定的。

我们设置View的大小，有match_parent、wrap_content和具体的dip值。

match_parent对应值为-1、wrap_conten对应值为-2，具体dip对应其设定的值。在测量时，View的父类从Layout中读出宽高值，根据不同的值设置不同的计算模式。

布局文件中所有layout_开头的在代码中都是需要通过LayoutParams来设置。

当我们通过addView添加一个子View时，如果它没有LayoutParams或者是LayoutParams的类型不匹配，那么将会创建一个默认的LayoutParams。

通过布局文件进行layout_width,layout_height进行设定。通过代码设置，需要一个LayoutParams来描述。

```java
View view = new View(this);
LayoutParams lp = new LayoutParams(LayoutParams.MATCH_PARENT,LayoutParams.WRAP_CONTENT);
view.setLayoutParams(lp);
```

## **8.3 measure**

```java
/**
  * This is called to find out how big a view should be. The parent
  * supplies constraint information in the width and height parameters.
  * The actual measurement work of a view is performed in
  * {@link #onMeasure(int, int)}, called by this method. Therefore, only
  * {@link #onMeasure(int, int)} can and must be overridden by subclasses.
  * @param widthMeasureSpec Horizontal space requirements as imposed by the
  *        parent
  * @param heightMeasureSpec Vertical space requirements as imposed by the
  *        parent
  *
  * @see #onMeasure(int, int)
  */
 public final void measure(int widthMeasureSpec, int heightMeasureSpec) {
    ...
    onMeasure(widthMeasureSpec, heightMeasureSpec);
    ...
 }
```
measure是final修饰的方法，不可被重写。在外部调用时，直接调用view.measure(int wSpec, int hSpec)。measure中调用了onMeasure。自定义view时，重写onMeasure即可

## **8.4 onMeasure**
measure是一个final方法，用来测量View自身的大小，View类该方法体逻辑比较简单，只是根据判断条件决定是否需要调用onMeasure。方法接受两个参数，分别就是通过MeasureSpec类合成测量模式和大小的宽与高。

实际上View的大小是无限大的，measure测量出来的大小只是为了layout时父View分配给它的显示区，也就是后来draw时画布的剪裁大小，和touch事件分发时计算落点是否在它上面。

onMeasure通过父View传递过来的大小和模式，以及自身的背景图片的大小得出自身最终的大小，通过setMeasuredDimension()方法设置给mMeasuredWidth和mMeasuredHeight。

普通View的onMeasure逻辑大同小异，基本都是测量自身内容和背景，然后根据父View传递过来的MeasureSpec进行最终的大小判定，例如TextView会根据文字的长度，文字的大小，文字行高，文字的行宽，显示方式，背景图片，以及父View传递过来的模式和大小最终确定自身的大小。

```java
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
                getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
    }
```

## 8.4 ViewGroup的onMeasure

ViewGroup是个抽象类，本身没有实现onMeasure，但是他的子类都有各自的实现，通常他们都是通过measureChildWithMargins函数或者其他类似于measureChild的函数来遍历测量子View，被GONE的子View将不参与测量，当所有的子View都测量完毕后，才根据父View传递过来的模式和大小来最终决定自身的大小。

在测量子View时，会先获取子View的LayoutParams，从中取出宽高，如果是大于0，将会以精确的模式加上其值组合成MeasureSpec传递子View，如果是小于0，将会把自身的大小或者剩余的大小传递给子View，其模式判定在前面已经讲过。

ViewGroup一般都在测量完所有子View后才会调用setMeasuredDimension()设置自身大小。

如果是一个View，重写onMeasure时要注意：如果在使用自定义view时，用了wrap_content。那么在onMeasure中就要调用setMeasuredDimension，来指定view的宽高。如果使用的fill_parent或者一个具体的dp值。那么直接使用super.onMeasure即可。

如果是一个ViewGroup，重写onMeasure时要注意：首先，结合上面两条，来测量自身的宽高。然后，需要测量子View的宽高。测量子view的方式有：
​    
getChildAt(int index)，可以拿到index上的子view。通过getChildCount得到子view的数目，再循环遍历出子view。接着，subView.measure(int wSpec, int hSpec)，使用子view自身的测量方法

或者调用viewGroup的测量子view的方法：

```java
//某一个子view，多宽，多高, 内部减去了viewGroup的padding值
measureChild(subView, int wSpec, int hSpec); 

//所有子view 都是 多宽，多高, 内部调用了measureChild方法
measureChildren(int wSpec, int hSpec);

//某一个子view，多宽，多高, 内部减去了viewGroup的padding值、margin值和传入的宽高wUsed、hUsed  
measureChildWithMargins(subView, intwSpec, int wUsed, int hSpec, int hUsed); 
```

Tips：自定义ViewGroup的时候，通常继承FrameLayout，这样就不必实现onMeasure()方法，让FrameLayout帮我们实现测量的工作，我们实现onlayout()即可

- onFinishInflate()
  当布局加载完成的时候的回调，自定义View的时候我们可以在该方法中获取View的宽高

- onSizeChange()
  当view的大小发生变化的时候的回调

  - requestLayout()
    重新布局，包括测量measure和布局onlayout

- resolveSize(int size, int measureSpec)
  算出来的size和测量出来的spec那个合适用那个

```java
public static int resolveSize(int size, int measureSpec) {
        return resolveSizeAndState(size, measureSpec, 0) & MEASURED_SIZE_MASK;
    }

 public static int resolveSizeAndState(int size, int measureSpec, int childMeasuredState) {
        final int specMode = MeasureSpec.getMode(measureSpec);
        final int specSize = MeasureSpec.getSize(measureSpec);
        final int result;
        switch (specMode) {
            case MeasureSpec.AT_MOST:
                if (specSize < size) {
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

## **8.5 setMeasuredDimension**

## **8.6 MeasureSpec** 

一个MeasureSpec封装了从父容器传递给子容器的布局要求，更精确的说法应该这个MeasureSpec是由父View的MeasureSpec和子View的LayoutParams通过简单的计算得出一个针对子View的测量要求，这个测量要求就是MeasureSpec

这是一个含mode和size的结合体，不需要我们来具体的关心。当在测量时，可以调用MeasureSpec.getSize|getMode 得到相应的size和mode。然后使用MeasureSpec.makeMeasureSpec(size,mode); 来创建MeasureSpec对象。那么mode是怎么来的呢？是根据使用该自定义view时的layoutWith|height参数决定的，所以不能自己随便new一个。而size可以自己指定，也可以直接使用 measureSpec.getSize。

MeasureSpe描述了父View对子View大小的期望。里面包含了测量模式和大小。

MeasureSpe类把测量模式和大小组合到一个int型的数值中，其中高2位表示模式，低30位表示大小。

我们可以通过以下方式从MeasureSpec中提取模式和大小，该方法内部是采用位移计算。

```java
int specMode = MeasureSpec.getMode(measureSpec);
int specSize = MeasureSpec.getSize(measureSpec);
```

也可以通过MeasureSpec的静态方法把大小和模式合成，该方法内部只是简单的相加。

```java
MeasureSpec.makeMeasureSpec(specSize,specMode);
```
采用这种方式，是为了提升效率，因为onMeasure在绘制过程中会被大量递归调用。

MeasureSpec中的测量模式有以下三种

- EXACTLY（-1）：精确的，表达了父View期望子View的大小就是父View通过MeasureSpec传递过来的大小。
- AT_MOST（-2）：最多的，表达了父View期望子View通过测量自身的大小来决定自己的大小，但是最多不要超过MeasureSpec传递过来的大小。
- UNSPECIFIED（0）：未指明，通常这时候MeasureSpec传递过来的大小也是0，说明父View不对子View的大小做任何期望，随子View自己决定。

通常情况下，我们应该遵守这种规则，当然如果也特殊需求也可以不遵守。但是不遵守该方式，在后面的layout中父View给你的视图大小仍然是它给的期望值。

### **8.6.1 常用方法**

-	MeasureSpec.getSize(widthMeasureSpec) 获取view的宽
   -MeasureSpec.getMode(int measurespec) 获取测量模式
   -MeasureSpec.makeMeasureSpec(size,mode) 组装32位的测量策略，高2位：mode，低30位：size

### **8.6.2 测量策略**
-  MeasureSpec.AT_MOST
表示子布局被限制在一个最大值内，一般当childView设置其宽、高为wrap_content时，ViewGroup会将其设置为AT_MOST

- MeasureSpec.EXACTLY
表示设置了精确的值，一般当childView设置其宽、高为精确值、match_parent时，ViewGroup会将其设置为EXACTLY

- MeasureSpec.UNSPECIFIED
表示子布局想要多大就多大，一般出现在AadapterView的item的heightMode中、ScrollView的childView的heightMode中；此种模式比较少见

###  **8.6.3 获取View的宽高**
getMeasuredHeight()，测量后的高度，实际高度。获取测量完的高度，只要在onMeasure方法执行完，就可以用它获取到宽高，在自定义控件内部多使用这个使用view.measure(0,0)方法可以主动通知系统去测量，然后就可以直接使用它获取宽高

getHeight()，显示的高度。必须在onLayout方法执行完后，才能获得宽高

### **8.6.4 measure(0,0)**
view.measure(0,0)主动通知系统去测量
```java
View view = new View(context);
view.measure(0,0);//等价于下面的代码

// MeasureSpec.UNSPECIFIED = 0
int widthMeasureSpec = MeasureSpec.makeMeasureSpec(0,MeasureSpec.UNSPECIFIED);
int heightMeasureSpec = MeasureSpec.makeMeasureSpec(0,MeasureSpec.UNSPECIFIED);
view.measure(widthMeasureSpec,heightMeasureSpec);

int measureWidth = view.getMeasuredWidth();
```
```java
view.getViewTreeObserver().addOnGlobalLayoutListener(new ViewTreeObserver.OnGlobalLayoutListener() {
    @Override
    public void onGlobalLayout() {
        int width = view.getWidth();
    }
});

```
### **8.6.5 getMeasuredWidth()与getWidth()的区别**
首先getMeasureWidth()方法在measure()过程结束后就可以获取到了，而getWidth()方法要在layout()过程结束后才能获取到。

getMeasureWidth()方法中的值是通过setMeasuredDimension()方法来进行设置的，而getWidth()方法中的值则是通过layout(left,top,right,bottom)方法设置的。

getWidth()：只有调用了onLayout()方法，getWidth()才赋值，显示的宽度

getMeasureWidth()：获取测量完的宽度，只要在onMeasure()方法执行完，就可以用它获取到高度，实际的宽度

### 8.6.6 getHeight()和getMeasuredHeight()的区别

getMeasuredHeight():获取测量完的高度，只要在onMeasure方法执行完，就可以用它获取到宽高，在自定义控件内部多使用这个。使用view.measure(0,0)方法可以主动通知系统去测量，然后就可以直接使用它获取宽高

getHeight()：必须在onLayout方法执行完后，才能获得宽高
```java	
view.getViewTreeObserver().addOnGlobalLayoutListener(new OnGlobalLayoutListener() {
      @Override
      public void onGlobalLayout() {              
            headerView.getViewTreeObserver().removeGlobalOnLayoutListener(this);
            int headerViewHeight = headerView.getHeight(); //直接可以获取宽高
    }
});		
```

### **8.6.7 getChildMeasureSpec**

getChildMeasureSpec( )的总体思路就是通过其父视图提供的MeasureSpec参数得到specMode和specSize，并根据计算出来的specMode以及子视图的childDimension（layout_width和layout_height中定义的）来计算自身的measureSpec，如果其本身包含子视图，则计算出来的measureSpec将作为调用其子视图measure函数的参数，同时也作为自身调用setMeasuredDimension的参数，如果其不包含子视图则默认情况下最终会调用onMeasure的默认实现，并最终调用到setMeasuredDimension，而该函数的参数正是这里计算出来的

![getChildMeasureSpec](http://img.blog.csdn.net/20170226131106666?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXhpMjk1MzA5MDY2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

```java
/**
     * Does the hard part of measureChildren: figuring out the MeasureSpec to
     * pass to a particular child. This method figures out the right MeasureSpec
     * for one dimension (height or width) of one child view.
     *
     * The goal is to combine information from our MeasureSpec with the
     * LayoutParams of the child to get the best possible results. For example,
     * if the this view knows its size (because its MeasureSpec has a mode of
     * EXACTLY), and the child has indicated in its LayoutParams that it wants
     * to be the same size as the parent, the parent should ask the child to
     * layout given an exact size.
     *
     * @param spec The requirements for this view
     * @param padding The padding of this view for the current dimension and
     *        margins, if applicable
     * @param childDimension How big the child wants to be in the current
     *        dimension
     * @return a MeasureSpec integer for the child
     */
    public static int getChildMeasureSpec(int spec, int padding, int childDimension) {
        int specMode = MeasureSpec.getMode(spec);
        int specSize = MeasureSpec.getSize(spec);

        int size = Math.max(0, specSize - padding);

        int resultSize = 0;
        int resultMode = 0;

        switch (specMode) {
        // Parent has imposed an exact size on us
        case MeasureSpec.EXACTLY:
            if (childDimension >= 0) {
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size. So be it.
                resultSize = size;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size. It can't be
                // bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            }
            break;

        // Parent has imposed a maximum size on us
        case MeasureSpec.AT_MOST:
            if (childDimension >= 0) {
                // Child wants a specific size... so be it
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size, but our size is not fixed.
                // Constrain child to not be bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size. It can't be
                // bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            }
            break;

        // Parent asked to see how big we want to be
        case MeasureSpec.UNSPECIFIED:
            if (childDimension >= 0) {
                // Child wants a specific size... let him have it
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size... find out how big it should
                // be
                resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                resultMode = MeasureSpec.UNSPECIFIED;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size.... find out how
                // big it should be
                resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                resultMode = MeasureSpec.UNSPECIFIED;
            }
            break;
        }
        //noinspection ResourceType
        return MeasureSpec.makeMeasureSpec(resultSize, resultMode);
    }
```

## **8.7 measureChild**
```java
/**
     * Ask one of the children of this view to measure itself, taking into
     * account both the MeasureSpec requirements for this view and its padding.
     * The heavy lifting is done in getChildMeasureSpec.
     *
     * @param child The child to measure
     * @param parentWidthMeasureSpec The width requirements for this view
     * @param parentHeightMeasureSpec The height requirements for this view
     */
    protected void measureChild(View child, int parentWidthMeasureSpec,
            int parentHeightMeasureSpec) {
        final LayoutParams lp = child.getLayoutParams();

        final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                mPaddingLeft + mPaddingRight, lp.width);
        final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
                mPaddingTop + mPaddingBottom, lp.height);

        child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    }
```
## **8.8 measureChildren**
```java
/**
     * Ask all of the children of this view to measure themselves, taking into
     * account both the MeasureSpec requirements for this view and its padding.
     * We skip children that are in the GONE state The heavy lifting is done in
     * getChildMeasureSpec.
     *
     * @param widthMeasureSpec The width requirements for this view
     * @param heightMeasureSpec The height requirements for this view
     */
    protected void measureChildren(int widthMeasureSpec, int heightMeasureSpec) {
        final int size = mChildrenCount;
        final View[] children = mChildren;
        for (int i = 0; i < size; ++i) {
            final View child = children[i];
            if ((child.mViewFlags & VISIBILITY_MASK) != GONE) {
                measureChild(child, widthMeasureSpec, heightMeasureSpec);
            }
        }
    }
```
## **8.9 measureChildWithMargins**
```java
 /**
     * Ask one of the children of this view to measure itself, taking into
     * account both the MeasureSpec requirements for this view and its padding
     * and margins. The child must have MarginLayoutParams The heavy lifting is
     * done in getChildMeasureSpec.
     *
     * @param child The child to measure
     * @param parentWidthMeasureSpec The width requirements for this view
     * @param widthUsed Extra space that has been used up by the parent
     *        horizontally (possibly by other children of the parent)
     * @param parentHeightMeasureSpec The height requirements for this view
     * @param heightUsed Extra space that has been used up by the parent
     *        vertically (possibly by other children of the parent)
     */
    protected void measureChildWithMargins(View child,
            int parentWidthMeasureSpec, int widthUsed,
            int parentHeightMeasureSpec, int heightUsed) {
        final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();

        final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin
                        + widthUsed, lp.width);
        final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
                mPaddingTop + mPaddingBottom + lp.topMargin + lp.bottomMargin
                        + heightUsed, lp.height);

        child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    }
```

# **9. layout 布局**

![layout](http://img.blog.csdn.net/20170302165052092?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXhpMjk1MzA5MDY2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

Layout方法中接受四个参数，是由父View提供，指定了子View在父View中的左、上、右、下的位置。父View在指定子View的位置时通常会根据子View在measure中测量的大小来决定。

子View的位置通常还受有其他属性左右，例如父View的orientation，gravity，自身的margin等等，特别是RelativeLayout，影响布局的因素非常多。

## 9.1 setFrame

setFrame方法是一个隐藏方法，所以作为应用层程序员来说，无法重写该方法。该方法体内部通过比对本次的l、t、r、b四个值与上次是否相同来判断自身的位置和大小是否发生了改变。

如果发生了改变，将会调用invalidate请求重绘。

记录本次的l、t、r、b，用于下次比对。

如果大小发生了变化，onSizeChanged方法，该方法在大多数View中都是空实现，程序员可以重写该方法用于监听View大小发生变化的事件，在可以滚动的视图中重载了该方法，用于重新根据大小计算出需要滚动的值，以便显示之前显示的区域。

## 9.2 View的layout()

```java
public final void layout(int l, int t, int r, int b) {
    .....
    //设置View位于父视图的坐标轴
    boolean changed = setFrame(l, t, r, b);
    //判断View的位置是否发生过变化，看有必要进行重新layout吗
    if (changed || (mPrivateFlags & LAYOUT_REQUIRED) == LAYOUT_REQUIRED) {
        if (ViewDebug.TRACE_HIERARCHY) {
            ViewDebug.trace(this, ViewDebug.HierarchyTraceType.ON_LAYOUT);
        }
        //调用onLayout(changed, l, t, r, b); 函数
        onLayout(changed, l, t, r, b);
        mPrivateFlags &= ~LAYOUT_REQUIRED;
    }
    mPrivateFlags &= ~FORCE_LAYOUT;
    .....
}
```
## 9.3 onLayout()
setFrame(l, t, r, b) 设置View位于父视图的坐标轴

onLayout是ViewGroup用来决定子View摆放位置的，各种布局的差异都在该方法中得到了体现。

onLayout比layout多一个参数，changed，该参数是在setFrame通过比对上次的位置得出是否发生了变化，通常该参数没有被使用的意义，因为父View位置和大小不变，并不能代表子View的位置和大小没有发生改变。

```java 
int childCount = getChildCount() ;
for(int i=0 ;i<childCount ;i++){
    View child = getChildAt(i) ;
    //整个layout()过程就是个递归过程
    child.layout(l, t, r, b) ;
}

public final int getMeasuredWidth() {
    return mMeasuredWidth & MEASURED_SIZE_MASK;
}
public final int getWidth() {
    return mRight - mLeft;
}
```

View中：

```java
public void layout(int l,int t,int r,int b) {
     ...
     onLayout
     ...
}
//changed 表示是否有新的位置或尺寸
protected void onLayout(boolean changed,int left,int top,int right,int bottom) {
     //空实现
}
```

ViewGroup中：

```java
public final void layout(int l,int t,int r,int b) {
     ...
     super.layout(l, t, r, b);
     ...
}
//changed 表示是否有新的位置或尺寸
protected abstract void onLayout(boolean changed, int l,int t, int r,int b);
```
说明：

- 自定义一个view时，建议重写onLayout，以设定它的位置。 
在外部调用时，调用layout()，触发设定位置。

- 自定义一个viewGroup时，必须且只能重写onLayout。
需要在设定子view的位置:调用subview.layout(); 触发

# **10. draw 绘制**

![ondraw](http://img.blog.csdn.net/20170302165255827?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXhpMjk1MzA5MDY2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

draw同样是由ViewRoot的performTraversals方法发起，它将调用DecorView的draw方法，并把成员变量canvas传给给draw方法。而在后面draw遍历中，传递的都是同一个canvas。所以android的绘制是同一个window中的所有View都绘制在同一个画布上。等绘制完成，将会通知WMS把canvas上的内容绘制到屏幕上。

## **10.1 draw的流程**
1. 绘制背景
2. 绘制渐变效果（通常不绘制）
3. 调用onDraw
4. 调用dispatchDraw
5. 调用onDrawScrollBars

绘制流程

- Step 1, draw the background, if needed 绘制背景
- Step 2, save the canvas' layers
- Step 3, draw the content 绘制内容
- Step 4, draw the children 绘制子view
- Step 5, draw the fade effect and restore layers
- Step 6, draw decorations (scrollbars) 对View的滚动条进行绘制

## 10.2 onDraw()
绘制视图自身，View用来绘制自身的实现方法，如果我们想要自定义View，通常需要重载该方法。TextView中在该方法中绘制文字、光标和CompoundDrawable，ImageView中相对简单，只是绘制了图片

## 10.3 dispatchDraw(canvas) 

- 先根据自身的padding剪裁画布，所有的子View都将在画布剪裁后的区域绘制。
- 遍历所有子View，调用子View的computeScroll对子View的滚动值进行计算。
- 根据滚动值和子View在父View中的坐标进行画布原点坐标的移动，根据子在父View中的坐标计算出子View的视图大小，然后对画布进行剪裁，请看下面的示意图。
- dispatchDraw的逻辑其实比较复杂，但是幸运的是对子View流程都采用该方式，而ViewGroup已经处理好了，我们不必要重载该方法对子View进行绘制事件的派遣分发。

用来绘制子View的，遍历子View然后drawChild(),drawChild()方法实际调用的是子View.draw()方法,ViewGroup类已经为我们实现绘制子View的默认过程，这个实现基本能满足大部分需求，所以ViewGroup类的子类（LinearLayout,FrameLayout）也基本没有去重写dispatchDraw方法

无论是View还是ViewGroup对它们俩的调用顺序都是onDraw()->dispatchDraw() 

但在ViewGroup中，当它有背景的时候就会调用onDraw()方法，否则就会跳过onDraw()直接调用dispatchDraw()；所以如果要在ViewGroup中绘图时，往往是重写dispatchDraw()方法。dispatchDraw()方法内部遍历子view，调用子view的绘制方法来完成绘制工作

在View中，onDraw()和dispatchDraw()都会被调用的，所以我们无论把绘图代码放在onDraw()或者dispatchDraw()中都是可以得到效果的，但是由于dispatchDraw()的含义是绘制子控件，所以原则来上讲，在绘制View控件时，我们是重新onDraw()函数 

在绘制View控件时，需要重写onDraw()函数，在绘制ViewGroup时，需要重写dispatchDraw()函数。

在自定义控件public class CircleProgressView extends LinearLayout的时候，如果不设置背景的话setBackground()的话，是不会走onDraw()方法的
dispatchDraw()绘制具体的内容（图片和文本）

View中：

```java
public void draw(Canvas canvas) {
/*
1. Draw the background   绘制背景
2. If necessary, save the canvas' layers to prepare for fading  如有必要，颜色渐变淡之前保存画布层(即锁定原有的画布内容)
3. Draw view's content  绘制view的内容
4. Draw children    绘制子view
5. If necessary, draw the fading edges and restore layers   如有必要，绘制颜色渐变淡的边框，并恢复画布(即画布改变的内容附加到原有内容上)
6. Draw decorations (scrollbars for instance)   绘制装饰，比如滚动条
*/
   ...
   if (!dirtyOpaque) {
       drawBackground(canvas); //背景绘制
   }
   // skip step 2 & 5 if possible (common case) 通常情况跳过第2和第5步
   ...
   if (!dirtyOpaque) onDraw(canvas); //调用onDraw
   dispatchDraw(canvas);   //绘制子view
   onDrawScrollBars(canvas); //绘制滚动条
   ...
}
protected void dispatchDraw(Canvas canvas) { //空实现 }
protected void onDraw(Canvas canvas) { //空实现 }
```

ViewGroup中：

```java
protected void dispatchDraw(Canvas canvas) {
    ...
    drawChild(...); //绘制子view
    ...
}

protected boolean drawChild(Canvas canvas, View child, long drawingTime) {
        return child.draw(canvas, this, drawingTime);
}
```
说明：

- 自定义一个view时，重写onDraw。
调用view.invalidate(),会触发onDraw和computeScroll()。前提是该view被附加在当前窗口上view.postInvalidate(); //是在非UI线程上调用的

- 自定义一个ViewGroup，重写onDraw。
onDraw可能不会被调用，原因是需要先设置一个背景(颜色或图)。表示这个group有东西需要绘制了，才会触发draw，之后是onDraw。因此，一般直接重写dispatchDraw来绘制viewGroup

- 自定义一个ViewGroup，dispatchDraw会调用drawChild。

# **11. 画布的移动和剪裁**

## 11.1 画布的移动和剪裁1

下面是一个ViewGroup视图，绿色原点代表其原点，根据padding剪裁画布后，黄色区域代表其剪裁后的画布区域，画布的原点将会移到黄色原点处

![view](http://img.blog.csdn.net/20170302165759773?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXhpMjk1MzA5MDY2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 11.2 画布的移动和剪裁2

在ViewGroup中，我们放一个TextView，Viewgroup完全满足TextView的测量大小，给了它合适的显示区域，也就是layout中设置的位置和它的大小一致。画布的原点会移动到粉色原点处。此时画布剪裁为粉色区域这么大。

![view](http://img.blog.csdn.net/20170302165903884?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXhpMjk1MzA5MDY2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 11.3 画布的移动和剪裁3

如果TextView的内容足够多，onMeasure的时候我们不理会父View给的参数，直接根据自身的内容来设置大小，但是父View在onLayout的时候分配的位置还是它期望的大小，也就是黑色的边框，这个时候粉色区域是TextView的大小，但是画布仍旧是黑色边框，画布原点仍旧是粉色原点

![view](http://img.blog.csdn.net/20170302170029699?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXhpMjk1MzA5MDY2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 11.4 画布的移动和剪裁4

我们为了看到其他的区域文字，对TextView进行了scroll的滚动，这个时候画布的剪裁大小任然是黑色边框，但是原点由透明原点根据TextView的滚动值进行移动到了TextView的原点，绘制会从textView的原点进行绘制，但是因为他们超出了画布的剪裁区域，将不会把数据绘制到画布上。

![view](http://img.blog.csdn.net/20170302170131769?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXhpMjk1MzA5MDY2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

# **12. 动画的绘制**

- 动画就是让画面“动”起来，其原理就是不断的绘制，但是每次绘制都有区别。
- 在ViewGroup的drawChild方法中会判断child是否包含动画，如果包含，则根据动画类计算出动画执行的区域矩形，判断动画是否启动了，启动了就获取动画当前的值，例如位移值等等。然后根据值对画布进行剪裁调整，执行子View的draw进行绘制。
- 判断动画是否结束，如果没有，则调用invalidate再次请求绘制。

## 12.1 动画绘制1
在ViewGroup中有一个红色的子View，将执行一个位移动画，位移动画将执行到A的位置，那么将会先根据动画参数计算出A位置的矩形大小。

![animation](http://img.blog.csdn.net/20170302170443345?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXhpMjk1MzA5MDY2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 12.2 动画绘制2
剪裁画布区域A的位置，把该画布交给子View，让其执行draw方法，那么View的内容都将会被绘制到A区域，而V所处的位置并没有发生变化。View继续移动到B位置。

![animation](http://img.blog.csdn.net/20170302170500069?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXhpMjk1MzA5MDY2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
## 12.3 动画绘制3
此时剪裁画布区域B的位置，把该画布交给子View，让其执行draw方法，那么View的内容都将会被绘制到B区域，但是B区域有一部分已经超过了ViewGroup画布区域。超出的地方虽然被绘制了，但是不会添加到画布上，也就不会显示出来

![animation](http://img.blog.csdn.net/20170302170510158?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXhpMjk1MzA5MDY2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

# 参考链接

- [ Android onMeasure、Measure、measureChild、measureChildren 一些简要说明](http://blog.csdn.net/jjwwmlp456/article/details/43964785)
- [Android layout、onLayout 一些简要说明](http://blog.csdn.net/jjwwmlp456/article/details/43983265)
- [Android draw、onDraw、dispatchDraw、invalidate、computeScroll 一些简要说明](http://blog.csdn.net/jjwwmlp456/article/details/43986141)

