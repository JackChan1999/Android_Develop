# **一、Touch事件和绘制事件的异同之处**

Touch事件和绘制事件很类似，都是由ViewRoot派发下来的，但是不同之处在绘制事件是由应用中的某个View发起请求，一层一层上传到ViewRoot，再有ViewRoot下发绘制，传递canvas给所有子View让其绘制自身，绘制好后，再通知WMS进行画到屏幕上。而Touch事件是由硬件捕获到触摸后由系统传递给应用的ViewRoot，再由ViewRoot往下一层一层传递。

他们的处理过程都是自上而下的分发，但是绘制多了一层自下往上的请求。

事件存在消耗，事件的处理方法都会返回一个boolean值，如果该值为true，则本次事件下发将会终止。

# **二、MotionEvent**

## **1、MotionEvent对象的产生**

系统有一个线程在循环收集屏幕硬件信息，当用户触摸屏幕时，该线程会把从硬件设备收集到的信息封装成一个MotionEvent对象，然后把该对象存放到一个消息队列中。

系统的另一个线程循环的读取消息队列中的MotionEvent，然后交给WMS去派发，WMS把该事件派发给当前处于活动的Activity，即处于活动栈最顶端的Activity。

这就是一个先进先出的消费者和生产者的模板，一个线程不停的创建MotionEvent对象放入队列中，另一个线程不断的从队列中取出MotionEvent对象进行分发。

当用户的手指从接触屏幕到离开屏幕，是一个完整的触摸事件，在该事件中，系统会不断收集事件信息封装成MotionEvent对象。收集的间隔时间取决于硬件设备，例如屏幕的灵敏度以及cpu的计算能力。目前的手机一般在20毫秒左右。

MotionEventCompat.getActionMasked()

## **2、MotionEvent对象详解**

MotionEvent对象包含了触摸事件的时间、位置、面积、压力、以及本次事件的Dwon发生的时间。

MotionEvent常用的Action分为5种：Down 、Up、Move、Cancel、OutSide

MotionEvent中我们常用的方法就是获取点击的坐标，因为这是与我们操作息息相关的。获取坐标有两种方式：

- getX和getY用于获取以该View左上角为坐标原点的坐标
- getRowX和getRowY用于获取以屏幕左上角为坐标原点的坐标

## **5种Touch事件**

- Down：一次触摸事件的第一个MotionEvent对象，即手指初次接触屏幕。
- Up：通常为一次触摸事件的最后一个MotionEvent对象，即手指离开屏幕。
- Move：通常多次发生在一次触摸事件之中。表示触摸点发生了移动，我们通常把手指放到屏幕上，实际也会触发该事件，因为人手总是在轻微抖动的。
- Cancel：常用于取消某个触摸事件，一般是由程序逻辑来指定该事件，用于取消某次触摸事件。
- OutSide：当触摸点发生在响应事件的View之外时，传递的事件，通常由程序逻辑来指定。

在上面5种事件中，Down为最重要的事件，因为这是一个触摸事件的起始点，程序的很多逻辑判断，都需要根据该事件做处理，例如分发拦截。一次触摸事件必须要有Down事件，这也是MotionEvent对象中都包含了本次触摸事件的Down事件发生的时间点这个属性。其次是Move和Up，通过这3个事件的逻辑处理，就构建出来滑动，点击，长按，双击等多种效果。

## **创建一个MotionEvent对象**
```java
public static MotionEvent obtain(
        long downTime,    //当用户最初按下开始一连串的位置事件。这必须得到SystemClock.uptimeMillis()
        long eventTime,   //当这个特定的事件是生成的。这必须得到SystemClock.uptimeMillis()            
        int action,       //该次事件的Action                       
        float x,          //该次事件的x坐标        
        float y,          //该次事件的y坐标         
        float pressure,   //该次事件的压力，通常感觉标准压力，从0-1取值     
        float size,       //点击的区域大小，通常根据特定标准范围从0-1取值     
        int metaState,    //一个修饰性的状态，好像一直都是0          
        float xPrecision, //x坐标的精确度           
        float yPrecision, //y坐标的精确度                   
        int deviceId,     //触屏设备id，如果是0，说明这个事件不是来自物理设备      
        int edgeFlags     //系统默认都是返回0，程序在传递时，可以通过逻辑判断加入方向位置 
)
```
或者一个更简单的方式：

```java
public static MotionEvent obtain(
            long downTime,
            long eventTime,
            int action,
            float x,
            float y,
            int metaState)

```
也可以通过一个MotionEvent来创建一个新的

```java
public static MotionEvent obtain(MotionEvent event)
```

通过以上的方式，我们知道，我们也可以通过代码来构建一个虚假的MotionEvent，并分发下去。

```java
view.dispatchTouchEvent(
            MotionEvent.obtain(SystemClock.uptimeMillis(),
            SystemClock.uptimeMillis(),
            MotionEvent.ACTION_DOWN,100,100,0));
```
然后通过延迟以此往下派发Move和Up时间，形成一个完整的触摸操作。

# **三、dispatchTouchEvent触摸事件分发**

![](http://img.blog.csdn.net/20170302184158181?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXhpMjk1MzA5MDY2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

之前我们知道触摸事件是被包装成MotionEvent进行传递的，而该对象是继承了Parcelable接口，正因为如此，才可以从系统中传递到我们的应用中。系统通过跨进程通知ViewRoot，ViewRoot会调用DecorView的dispatchTouchEvent下发。

这里有一个和其他事件传递不同的地方，DecorView会优先传递给Activity，而不是它的子View。而Activity如果不处理又会回传给DecorView，DecorView才会再将事件传给子View。

dispatchTouchEvent就是触摸事件传递的对外接口，无论是DecorView传给Activity，还是ViewGroup传递给子View，都是直接调用对方的dispatchTouchEvent方法，并传递MotionEvent参数。

我们首先来看看Activity中的dispatchTouchEvent逻辑：

```java
public boolean dispatchTouchEvent(MotionEvent ev) {
    if (ev.getAction() == MotionEvent.ACTION_DOWN) {
        onUserInteraction();
        //这是一个空实现的方法，以便子类实现，该方法在Key事件和touch事件的dispatch方法中都被调用，
        // 就是方便用户在事件被传递之前做一下自己的处理。
    }
    //这才是事件真正的分发
    if (getWindow().superDispatchTouchEvent(ev)) {
        //superDispatchTouchEvent是一个抽象方法，但是getWindow()获取的对象实际是FrameWork层的
        // PhoneWindow，该对象实现了这个方法，内部是直接调用DecorView的superDispatchTouchEvent
        // 是直接调用dispatchTouchEvent，这样就传递到子View中了   
        return true;
    }
    //如果上面事件没有被消费掉，那么就调用Activity的onTouchEvent事件。
    return onTouchEvent(ev);
}
//PhoneWindow的superDispatchTouchEvent方法直接调用了mDecor的superDispatchTouchEvent
public boolean superDispatchTouchEvent(MotionEvent event) {
    return mDecor.superDispatchTouchEvent(event);
}
//mDecor即为Activity真正的根View，我们通过setContentView所添加的内容就是添加在该View上，
// 它实际上就是一个FrameLayout
public boolean superDispatchTouchEvent(MotionEvent event) {
    return super.dispatchTouchEvent(event);//FrameLayout.dispatchTouchEvent
}
```

至此我们已经至少明白了以下几点：

1、我们可以重载Activity的onUserInteraction方法，在Down事件触发传递前，实现我们的一些需求，实际上源码中有很多这样的方法，再某个方法体的第一行提供一个空实现的回调方法，在某个方法的最后一行提供一个空实现的回调方法，以便子类去实现自己的逻辑，例如AsyncTask就有类似的方式。这些技巧都能很好的提高我们代码的扩展性。

2、Activity会间接的调用根View的dispatchTouchEvent，并通过if判断返回值，如果为true，即向上层返回true，也就是调用Activity的dispatchTouchEvent的WMS，即操作系统。

3、如果if判断为false，即根View和根View下的所有子View均为消费掉该事件，那么下面的代码就有执行机会，即Activity的onTouchEvent，并把该方法的返回值作为结果返回给上层。

## **3.1、View的dispatchTouchEvent**

![](http://img.blog.csdn.net/20170302184634662?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXhpMjk1MzA5MDY2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

View中的处理相当简单明了，因为不涉及到子View，所以只在自身内部进行分发。首先判断是否设置了触摸监听，并且可以响应事件，就交由监听的onTouch处理。如果上述条件不成立，或者监听的onTouch事件没有消费掉该事件，则交由onTouchEvent进行处理，并把返回结果交给上层。

```java
public boolean dispatchTouchEvent(MotionEvent event) {
    if (mOnTouchListener != null && (mViewFlags & ENABLED_MASK) == ENABLED &&
            mOnTouchListener.onTouch(this, event)) {
        //判断mOnTouchListener是否存在，并且控件可点的情况下，执行onTouch，如果onTouch返回true，就消耗该事件
        return true;
    }
    //如果以上条件都不成立，则把事件交给onTouchEvent来处理
    return onTouchEvent(event);
}
```

## **3.2、ViewGroup的dispatchTouchEvent**

![](http://img.blog.csdn.net/20170302183800394?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXhpMjk1MzA5MDY2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### Down事件：

- 通过onInterceptTouchEvent方法判断是否要拦截事件，默认fasle
- 根据scroll换算后的坐标找出所接受的子View。有动画的子View将不接受触摸事件。
- 找到能接受的子View后把event中的坐标转换成子View的坐标
- 调用子View的dispatchTouchEvent把事件传递给子View。
- 如果子View消费了该事件，则把target记录为子View，方便后面的Move和Up事件的传递。
- 如果子View没有消费，则继续寻找下一个子View。
- 如果没找到，或者找到的子View都不消费，就会调用View的dispatchTouchEvent的逻辑，也就是判断是否有触摸监听，有的话交给监听的onTouch处理，没有的话交给自己的onTouchEvent处理

接下来我们来研究ViewGroup的dispatchTouchEvent，这是稍微复杂的分发逻辑。

```java
public boolean dispatchTouchEvent(MotionEvent ev) {
    final int action = ev.getAction();//获取事件
    final float xf = ev.getX();//获取触摸坐标
    final float yf = ev.getY();
    final float scrolledXFloat = xf + mScrollX;//获取当前需要偏移的偏移量量
    final float scrolledYFloat = yf + mScrollY;
    final Rect frame = mTempRect;    //当前ViewGroup的视图矩阵

    boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;//是否禁止拦截

    if (action == MotionEvent.ACTION_DOWN) {//如果事件是按下事件
        if (mMotionTarget != null) {    //判断接受事件的target是否为空
            //不为空肯定是不正常的，因为一个事件是由DOWN开始的，而DOWN还没有被消费，所以目标也不是不可能被确定，
            //造成这个的原因可能是在上一次up事件或者cancel事件的时候，没有把目标赋值为空
            mMotionTarget = null;    //在此处挽救
        }
        //不允许拦截，或者onInterceptTouchEvent返回false，也就是不拦截。注意，这个判断都是在DOWN事件中判断
        if (disallowIntercept || !onInterceptTouchEvent(ev)) {
            //从新设置一下事件为DOWN事件，其实没有必要，这只是一种保护错误，防止被篡改了
            ev.setAction(MotionEvent.ACTION_DOWN);
            //开始寻找能响应该事件的子View
            final int scrolledXInt = (int) scrolledXFloat;
            final int scrolledYInt = (int) scrolledYFloat;
            final View[] children = mChildren;
            final int count = mChildrenCount;
            for (int i = count - 1; i >= 0; i--) {
                final View child = children[i];
                if ((child.mViewFlags & VISIBILITY_MASK) == VISIBLE
                        || child.getAnimation() != null) {//如果child可见，或者有动画，获取该child的矩阵
                    child.getHitRect(frame);
                    if (frame.contains(scrolledXInt, scrolledYInt)) {
                        // 设置系统坐标
                        final float xc = scrolledXFloat - child.mLeft;
                        final float yc = scrolledYFloat - child.mTop;
                        ev.setLocation(xc, yc);
                        if (child.dispatchTouchEvent(ev))  {//调用child的dispatchTouchEvent
                            //如果消费了，目标就确定了，以便接下来的事件都传递给child
                            mMotionTarget = child;
                            return true;    //事件消费了，返回true
                        }
                    }
                }
            }
            //能到这里来，证明所有的子View都没消费掉Down事件，那么留给下面的逻辑进行处理
        }
    }
    //判断是不是up或者cancel事件
    boolean isUpOrCancel = (action == MotionEvent.ACTION_UP) ||
            (action == MotionEvent.ACTION_CANCEL);

    if (isUpOrCancel) {
        //如果是取消，把禁止拦截这个标志位给取消
        mGroupFlags &= ~FLAG_DISALLOW_INTERCEPT;
    }


    final View target = mMotionTarget;
    if (target == null) {
        //判断该值是否为空，如果为空，则没找到能响应的子View，那么直接调用super的dispatchTouchEvent，也就是View的dispatchTouchEvent
        ev.setLocation(xf, yf);
        return super.dispatchTouchEvent(ev);
    }

    //能走到这里来，说明已经有target，那也说明，这里不是DOWN事件，因为DOWN事件如果有target，已经在前面返回了，执行不到这里
    if (!disallowIntercept && onInterceptTouchEvent(ev)) {//如果有目标，又非要拦截，则给目标发送一个cancel事件
        final float xc = scrolledXFloat - (float) target.mLeft;
        final float yc = scrolledYFloat - (float) target.mTop;
        ev.setAction(MotionEvent.ACTION_CANCEL);//该为cancel
        ev.setLocation(xc, yc);
        if (!target.dispatchTouchEvent(ev)) {
            //调用子View的dispatchTouchEvent，就算它没有消费这个cancel事件，我们也无能为力了。
        }
        //清除目标
        mMotionTarget = null;
        //有目标，又拦截，自身也享受不了了，因为一个事件应该由一个View去完成
        return true;//直接返回true，以完成这次事件，好让系统开始派发下一次
    }

    if (isUpOrCancel) {//取消或者UP的话，把目标赋值为空，以便下一次DOWN能重新找，此处就算不赋值，下一次DOWN也会先把它赋值为空
        mMotionTarget = null;
    }
    //又不拦截，又有目标，那么就直接调用目标的dispatchTouchEvent
    final float xc = scrolledXFloat - (float) target.mLeft;
    final float yc = scrolledYFloat - (float) target.mTop;
    ev.setLocation(xc, yc);

    return target.dispatchTouchEvent(ev);
    //也就是说，如果是DOWN事件，拦截了，那么每次一次MOVE或者UP都不会再判断是否拦截，直接调用super的dispatchTouchEvent
    //如果DOWN没拦截，就是有其他View处理了DOWN事件，那么接下来的MOVE或者UP事件拦截了，那么给目标View发送一个cancel事件，告诉它touch被取消了，并且自身也不会处理，直接返回true
    //这是为了不违背一个Touch事件只能由一个View处理的原则。
}
```

### Move和Up事件

判断事件是否被取消或者事件是否要拦截住，是的话，给Down事件找到的target发送一个取消事件。如果不取消，也不拦截，并且Down已经找到了target，则直接交给target处理，不再遍历子View寻找合适的View了。这种处理事件是正确的，我们用手机经常可以体会到，当我手指按在一个拖动条上之后，在拖动的时候手指就算移出了拖动条，依然会把事件分发给拖动条控制它的拖动。

# **四、onInterceptTouchEvent**
ViewGroup的方法，事件拦截，return true表示拦截触摸事件，事件就不往下传递

子View可以调用getParent().requestDisallowInterceptTouchEvent( true ) 请求父控件不拦截touch事件

# **五、View的onTouchEvent**

从View的dispatchTouchEvent可以看出，事件最终的处理无非是交给TouchListener的onTouch方法或者是交由onTouchEvent处理，由于onTouch默认是空实现，由程序员来编写逻辑，那么我们来看看onTouchEvent事件。View只能响应click和longclick，不具备滑动等特性。

Down时，设置按压状态，发送一个延迟500毫秒的长按事件。
Move时，判断是否移出了View，移出后移除按压状态，长按事件。
Up时，取消按压，并判断它是否可以通过触摸获取焦点，是的话设置焦点，判断长按事件是否执行了，如果还没执行，就删除，并执行点击事件。

```java
public boolean onTouchEvent(MotionEvent event) {
    final int viewFlags = mViewFlags;
    //先判断标示位是否为disable，也就是无法处理事件。
    if ((viewFlags & ENABLED_MASK) == DISABLED) {
        if (event.getAction() == MotionEvent.ACTION_UP && (mPrivateFlags & PFLAG_PRESSED) != 0) {
            setPressed(false);
        }//如果是UP事件，并且状态为按压，取消按压。
        //系统源码解释：虽然是disable，但是还是可以消费掉触摸事件，只是不触发任何click或者longclick事件。
        //根据是否可点击,可长按来决定是否消费点击事件。
        return (((viewFlags & CLICKABLE) == CLICKABLE ||
                (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE));
    }
    if (mTouchDelegate != null) {
        //先检查触摸的代理对象是否存在，如果存在，就交由代理对象处理。
        // 触摸代理对象是可以进行设置的，一般用于当我们手指在某个View上，而让另外一个View响应事件，另外一个View就是该View的事件代理对象。
        if (mTouchDelegate.onTouchEvent(event)) {//如果代理对象消费了，则返回true消费该事件
            return true;
        }
    }
    if (((viewFlags & CLICKABLE) == CLICKABLE ||
            (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)) {
        //如果是可点击或者长按的标识位执行下面的逻辑，这些标志位可以设置，也可以设置了对应的listener后自动添加
        //因为作为一个View，它只能单纯的接受处理点击事件，像滑动之类的复杂事件普通View是不具备的。
        switch (event.getAction()) {
            case MotionEvent.ACTION_UP://处理Up事件
                boolean prepressed = (mPrivateFlags & PFLAG_PREPRESSED) != 0;//是否包含临时按压状态
                if ((mPrivateFlags & PFLAG_PRESSED) != 0 || prepressed) {//如果本身处于被按压状态或者临时按压状态
                    //临时按压状态会在下面的Move事件中说明
                    boolean focusTaken = false;
                    if (isFocusable() && isFocusableInTouchMode() && !isFocused()) {
                        //如果它可以获取焦点，并且可以通过触摸来获取焦点，并且现在不是焦点，则请求获取焦点，因为一个被按压的View理论上应该获取焦点
                        focusTaken = requestFocus();
                    }
                    if (prepressed) {
                        //如果是临时按压，则设置为按压状态，PFLAG_PREPRESSED是一个非常短暂的状态，用于在某些时候短时间内表示Pressed状态，但不需要绘制
                        setPressed(true);//设置为按压状态，是因为临时按压不会绘制，这个时候强制绘制一次，确保用户能够看见按压状态
                    }
                    if (!mHasPerformedLongPress) {
                        //是否执行了长按事件，还没有的话，这个时候可以移除长按的回调了，因为UP都已经触发，说明从按下到UP的时间不足以触发longPress
                        //至于longPress，会在Down事件中说明
                        removeLongPressCallback();
                        if (!focusTaken) {//如果是焦点状态，就不会触摸click，这是为什么呢？因为焦点状态一般是交给按键处理的，
                            //pressed状态才是交给触摸处理，如果它是焦点，那么它的click事件应该由按键来触发
                            if (mPerformClick == null) {    //封装一个Runnable对象，这个对象中实际就调用了performClick();
                                mPerformClick = new PerformClick();
                            }
                            if (!post(mPerformClick)) {//向消息队列发生该runnabel，如果发送不成功，则直接执行该方法。
                                performClick();//这个方法内部会调用clickListner
                            }
                            //为什么不直接执行呢？如果这个时候直接执行，UP事件还没执行完，发送post，可以保障在这个代码块执行完毕之后才执行
                        }
                    }
                    if (mUnsetPressedState == null) {//仍旧是创建一个Runnabel对象，执行setPressed(false)
                        mUnsetPressedState = new UnsetPressedState();
                    }
                    if (prepressed) {
                        //如果是临时按压状态，之前的Down和move都还未触发按压状态，只在up时设置了，这个状态才刚刚绘制，为了保证用户能看到，发生一个64秒的延迟消息，来取消按压状态。                        postDelayed(mUnsetPressedState,
                        ViewConfiguration.getPressedStateDuration());
                        //这是一个64毫秒的短暂时间，这是为了让这个按压状态持续一小段时间，以便手指离开时候，还能看见View的按压状态
                    } else if (!post(mUnsetPressedState)) {//如果不是临时按压，则直接发送，发送失败，则直接执行
                        mUnsetPressedState.run();
                    }
                    removeTapCallback();
                    //移除这个callBack，这个callBack内部就是把临时按压状态设置成按压状态，因为这个已经没必要了，手指已经up了
                }
                break;
            case MotionEvent.ACTION_DOWN:
                mHasPerformedLongPress = false;
                //按下事件把长按事件执行的变量设置为false，代表还没执行长按，因为才按下，表示新的一个长按事件可以开始计算了
                if (performButtonActionOnTouchDown(event)) {
                    //先把这个事件交由该方法，该方法内部会判断是否为上下文的菜单按钮，或者是否为鼠标右键，如果是就弹出上下文菜单。
                    //现在有些手机的上下文菜单按钮也是在屏幕触屏上的
                    break;
                }
                //这个方法会一直往上找父View，判断自身是否在一个可以滚动的容器中
                boolean isInScrollingContainer = isInScrollingContainer();
                //如果是在一个滚动的容器中，那么按压事件将会被推迟一段时间，如果这段时间内，发生了Move，那么按压状态讲不会被显示，直接滚动父视图
                if (isInScrollingContainer) {
                    mPrivateFlags |= PFLAG_PREPRESSED; //先添加临时的按压状态，该状态表示按压，但不会绘制
                    if (mPendingCheckForTap == null) {
                        mPendingCheckForTap = new CheckForTap();
                        //创建一个runnable对象，这个runnable内部会取消临时按压状态，设置为按压状态，并启动长按的延迟事件
                    }
                    postDelayed(mPendingCheckForTap, ViewConfiguration.getTapTimeout());
                    //向消息机制发生一个64毫秒的延迟时间，该事件会取消临时按压状态，设置为直接按压，并启动长按时间的计时
                } else {
                    //如果不在一个滚动的容器中，则直接设置按压状态，并启动长按计时
                    setPressed(true);
                    checkForLongClick(0);
                    //长按事件就是向消息机制发送一个runnable对象，封装的就是我们在lisner中的代码，延迟500毫秒执行，也就是说长按事件在我们按下的时候发送，在up的时候检查一下执行了吗？如果没执行，就取消，并执行click
                }
                break;
            case MotionEvent.ACTION_CANCEL: //如果是取消事件，那就好办了，把我们之前发送的几个延迟runnable对象给取消掉
                setPressed(false);      //设置为非按压状态
                removeTapCallback();    //取消mPendingCheckForTap，也就是不用再把临时按压设置为按压了
                removeLongPressCallback();    //取消长按事件的延迟回调
                break;
            case MotionEvent.ACTION_MOVE:    //move事件
                final int x = (int) event.getX();    //取触摸点坐标
                final int y = (int) event.getY();
                // 用于判断是否在View中，为什么还要判断呢？
                //这是因为父View是在Down事件中判断是否在该View中的，如果在，以后的Move和up都会传递过来，不再进行范围判断
                if (!pointInView(x, y, mTouchSlop)) {
                    //mTouchSlop是一个常量，数值为8,也就是说，就算你的落点超出了View的8像素位置，也算在View中。
                    //是因为人的手指触摸点比较大，有可能你感觉点在某个控件的边缘，但是实际落点已经超出这个View，所以这里给了8像素的范围
                    removeTapCallback();//如果在范围外，就移除这些runnable回调
                    if ((mPrivateFlags & PFLAG_PRESSED) != 0) {
                        //如果是按压状态，就取消长按，设置为非按压状态，为什么这个时候取消呢，因为在Down的时候，我们可以知道，只有是按压状态，才会设置长按
                        removeLongPressCallback();
                        setPressed(false);
                    }
                }
                break;
        }
        return true;    //至此，可以返回true，消费该事件
    }
    return false;    //如果不可点击，也不可长按，则返回false，因为View只具备消费点击事件
}
```

从上面的代码我们总结一下View对触摸事件的处理：

1、是否为diabale，如果是，直接根据是否设置了click和longclick来返回。
2、是否设置了触摸代理对象，如果有，把事件传递给触摸代理对象，交由其处理，如果消费了，直接返回
3、是否为click或者longclick的，如果是，返回true，不是返回false。

而View对click和longclick的处理如下：

Down：

- 判断是否可以触摸上下文菜单。
- 是否在可以滑动的容器中，如果是先设置临时按压，再发送一个延迟消息把临时按压改为按压，并发送一个延迟500毫秒的事件去执行长按代码
- 如果不在滚动容器中，直接设置按压状态，并发送一个延迟500毫秒的事件去执行长按代码。

Move：

- 取触摸点坐标判断是否在View中（额外增加了8像素的范围）
- 如果在，不用做任何事。
- 如果不在，取消临时按压到按压回调，取消长按延迟回调，设置为非按压状态

Up

- 判断是否为按压或者临时按压状态
- 如果不是，不做任何处理
- 如果是先判断其是否可以获取焦点，然后请求焦点。
- 如果是临时按压状态，设置临时按压状态为按压状态。保证界面被绘制成按压状态，让用户可以看见。
- 如果长按回调还未触发，取消长按回调，如果不是焦点状态，触发click事件。
- 如果是临时按压状态，发送一个延迟取消按压状态的，保证按压状态持续一段时间，让用户可见。
- 如果不是临时按压状态，直接发送消息取消按压状态。发送失败，直接取消按压状态。
- 取消把临时按压设置按压的回调。

从中我们知道View的onTouchEvent主要处理了click和longclick事件，当按下时，向消息机制发送一个延迟500毫秒的长按回调事件，当移动时候判断是否移出了View的范围，超出则取消事件。当离开时，判断长按事件是否触发了，如果没触发且不是焦点，就触发click事件。

在这里最绕的就是临时按压和按压状态，临时按压是为了处理滑动容器的，让处于滑动容器中，按下时，我们先设置的是临时按压，持续64毫秒，是为了判断接下来的时间内是否发生了move事件，如果发生了，将不会再出发按压状态，这样不会让用户看到listView滚动时，item还处于按压状态。在离开时，我们再次判断是否处于临时按压，如果是在64毫秒内触发了down和up，说明按压状态还没来得急绘制，则强制设置为按压状态，保证用户能看到，并在取消回调的方法上加上64毫秒的延迟

# **六、onTouch与onClick**

```java
ImageView iv_image = (ImageView) findViewById(R.id.iv_image);
iv_image.setOnTouchListener(new OnTouchListener() {

    @Override
    public boolean onTouch(View v, MotionEvent event) {
        System.out.println("iv_image---onTouch--" + event.getAction());
        return false;
    }
});
```
点击ImageView的时候只会打印一次，因为onTouch()返回false，只传递down事件，不会传递up事件
```
System.out: iv_image---onTouch--0
```

```java
// ImageView天生不能被点击，没有点击事件
ImageView iv_image = (ImageView) findViewById(R.id.iv_image);
iv_image.setOnTouchListener(new OnTouchListener() {

    @Override
    public boolean onTouch(View v, MotionEvent event) {
        System.out.println("iv_image---onTouch--" + event.getAction());
        return true; // 把返回值改为true 
    }
});
```
把onTouch()方法返回值改为true，点击ImageView会打印两次（down and up）

```
System.out: iv_image---onTouch--0
System.out: iv_image---onTouch--1
```

```java
ImageView iv_image = (ImageView) findViewById(R.id.iv_image);
iv_image.setOnTouchListener(new OnTouchListener() {
    @Override
    public boolean onTouch(View v, MotionEvent event) {
        System.out.println("iv_image---onTouch--" + event.getAction());
        return true;
    }
});
//添加click事件
iv_image.setOnClickListener(new OnClickListener() {
    @Override
    public void onClick(View v) {
        System.out.println("iv_image---onClick");
    }
});
```
还是打印两次，onTouch()返回true，click事件并不会得到执行
```java
ImageView iv_image = (ImageView) findViewById(R.id.iv_image);
iv_image.setOnTouchListener(new OnTouchListener() {
    @Override
    public boolean onTouch(View v, MotionEvent event) {
        System.out.println("iv_image---onTouch--" + event.getAction());
        return false;
    }
});

iv_image.setOnClickListener(new OnClickListener() {
    @Override
    public void onClick(View v) {
        System.out.println("iv_image---onClick");
    }
});
```
打印三次，两次touch事件（down and up）和一次click事件
```java
Button button = (Button) findViewById(R.id.button);
button.setOnTouchListener(new OnTouchListener() {
    @Override
    public boolean onTouch(View v, MotionEvent event) {
        System.out.println("button---onTouch--" + event.getAction());
        return false;
    }
});
```
点击Button会打印两次

```java

Button button = (Button) findViewById(R.id.button);
button.setOnTouchListener(new OnTouchListener() {
    @Override
    public boolean onTouch(View v, MotionEvent event) {
        System.out.println("button---onTouch--" + event.getAction());
        return true;
    }
});

button.setOnClickListener(new OnClickListener() {
    @Override
    public void onClick(View v) {
        System.out.println("button---onClick");
    }
});
```
打印两次，因为onTouch()返回true，不会执行onTouchEvent()，而click事件是在onTouchEvent()中执行，所以也不会执行click事件
```java

Button button = (Button) findViewById(R.id.button);
button.setOnTouchListener(new OnTouchListener() {
    @Override
    public boolean onTouch(View v, MotionEvent event) {
        System.out.println("button---onTouch--" + event.getAction());
        return false;
    }
});

button.setOnClickListener(new OnClickListener() {
    @Override
    public void onClick(View v) {
        System.out.println("button---onClick");
    }
});
```
打印三次
```java
public boolean dispatchTouchEvent(MotionEvent event) {
    if (!onFilterTouchEventForSecurity(event)) {
        return false;
    }

    if (mOnTouchListener != null && (mViewFlags & ENABLED_MASK) == ENABLED &&
            mOnTouchListener.onTouch(this, event)) {
        return true;
    }
    return onTouchEvent(event);
}

```
a 判断mOnTouchListener是否为null
b 判断当前的控件是否可用
c 判断view的onTouch。
d 如果以上一个返回为false。那么就会调用onTouchEvent

首先判断mOnTouchListener不为null，并且view是enable的状态，然后 mOnTouchListener.onTouch(this, event)返回true，这三个条件如果都满足，直接return true ; 也就是下面的onTouchEvent(event）不会被执行了。如果我们设置了setOnTouchListener，并且return true，那么View自己的onTouchEvent就不会被执行了

onTouch是优先于onClick执行, onClick的调用在onTouchEvent(event)方法中

view的事件分发

1. 返回true，说明可以响应down事件和up事件
2. 返回false，只会响应down事件。不会响应up事件。在down事件如果能消费(处理)当前事件。那么在up的时候也会把事件传递给当前的view，在down事件处理不了当前事件。那么在up的时候。也不会把事件传递给当前的view

[Android的事件分发实例分析](http://blog.csdn.net/axi295309066/article/details/60139074)

# **七、ScrollView的onTouchEvent**

普通的ViewGroup并没有对onTouchEvent事件做处理，只有可以滚动的才有，我们可以分析一下ScrollView

- Down时，判断落点是否在子View中，不再就不处理，因为ScrollView只有一个子View。

- Move时，通过对比本次手指的位置和上一次的位置的距离，计算出Y方向的差值，然后用scorllBy进行滚动视图

- Up时，通过速度进行fling，这里利用了两个帮助类，一个是计算速度的帮助类VelocityTracker，一个是滚动的帮助类Scroller

```java
public boolean onTouchEvent(MotionEvent ev) {
    if (ev.getAction() == MotionEvent.ACTION_DOWN && ev.getEdgeFlags() != 0) {
        //如果是down事件，并且触摸到边缘，就不处理EdgeFlags代表是否为边缘，其值是1/2/4/8。代表上下左右
        return false;
    }
    if (mVelocityTracker == null) {
        //这是一个追踪触摸事件，并计算速度的帮助类，实现原理就是用三个数组分别记录每次触摸的x/y和时间
        mVelocityTracker = VelocityTracker.obtain();
    }
    mVelocityTracker.addMovement(ev);
    final int action = ev.getAction();
    switch (action & MotionEvent.ACTION_MASK) {//与上ff，去掉高位有关多点的信息
        case MotionEvent.ACTION_DOWN: {//如果是down
            final float y = ev.getY();//获取y坐标
            if (!(mIsBeingDragged = inChild((int) ev.getX(), (int) y))) {//判断是否开始拖动
                //原理就是判断落点是否在child中，ScrollView只能由一个child，如果在，返回true，反之false
                //也就是说落点在child中，就是准备开始拖动，不在，就直接返回，这可能是因为设置了padding之类的缘故造成的
                return false;
            }
            if (!mScroller.isFinished()) {//判断滚动是否完成
                mScroller.abortAnimation();//如果没完成，停止滚动
                //对应上一次用户手指离开时候处理fling状态，这次按下手指，直接停止滚动
            }
            //记录y坐标，以便下次事件来对比
            mLastMotionY = y;
            mActivePointerId = ev.getPointerId(0);//记住多点的id，下次取值时只取该点的
            break;
        }
        case MotionEvent.ACTION_MOVE:
            if (mIsBeingDragged) {//可以看出，如果down的时候落点在child外，则以后就算滑进了child也不处理
                //根据上次记录的多点id，找到对应的点，取y值
                final int activePointerIndex = ev.findPointerIndex(mActivePointerId);
                final float y = ev.getY(activePointerIndex);
                final int deltaY = (int) (mLastMotionY - y);//计算位移
                mLastMotionY = y;//重新记录y值
                scrollBy(0, deltaY);//滚动指定的距离，这也说明了ScrollView只具备纵向滑动
            }
            break;
        case MotionEvent.ACTION_UP:
            if (mIsBeingDragged) {//如果是离开事件
                final VelocityTracker velocityTracker = mVelocityTracker;
                velocityTracker.computeCurrentVelocity(1000, mMaximumVelocity);//计算最后1秒钟内的速度，并给定一个最大速度进行限制
                //这个最大速度是根据屏幕密度不同而不同的，所以大家也没事别使劲滑动屏幕，因为有这个最大速度限制
                //获取y方向的速度
                int initialVelocity = (int) velocityTracker.getYVelocity(mActivePointerId);
                if (getChildCount() > 0 && Math.abs(initialVelocity) > mMinimumVelocity) {
                    //如果有子View，并且计算出来的y的速度比最小速度要大，执行fling状态
                    //手指滑动的方向和屏幕移动的方向是相反的，所以这里加-
                    fling(-initialVelocity);
                }
                mActivePointerId = INVALID_POINTER;//给mActivePointerId重新赋值为-1，防止下次事件找到了错误的点
                mIsBeingDragged = false;//恢复默认值
                if (mVelocityTracker != null) {//清空速度计算帮助类
                    mVelocityTracker.recycle();
                    mVelocityTracker = null;
                }
            }
            break;
        case MotionEvent.ACTION_CANCEL:
            if (mIsBeingDragged && getChildCount() > 0) {//判断条件，只有这2个条件成立，才会发生滚动事件，下面的值才会被改变，才需要恢复默认
                mActivePointerId = INVALID_POINTER;
                mIsBeingDragged = false;
                if (mVelocityTracker != null) {
                    mVelocityTracker.recycle();
                    mVelocityTracker = null;
                }
            }
            break;
        case MotionEvent.ACTION_POINTER_UP://多点触摸时，不是最后一个点离开
            onSecondaryPointerUp(ev);
            break;
    }
    return true;
}
//用于应对先按下1点，然后按下2点，1点离开后，2点仍能继续滑动的逻辑
private void onSecondaryPointerUp(MotionEvent ev) {
    final int pointerIndex = (ev.getAction() & MotionEvent.ACTION_POINTER_INDEX_MASK) >>
            MotionEvent.ACTION_POINTER_INDEX_SHIFT;//首先对高位进行与操作，然后右移8位，获取其高位代表index的值
    final int pointerId = ev.getPointerId(pointerIndex);//取出该点的id
    if (pointerId == mActivePointerId) {//如果这个id对应的就是第一个按下的点
        //理论上pointerIndex应该是0，所以用第二个按下的点，即1index的点代替
        final int newPointerIndex = pointerIndex == 0 ? 1 : 0;
        mLastMotionY = ev.getY(newPointerIndex);//取出新点的y坐标
        mActivePointerId = ev.getPointerId(newPointerIndex);//记录新点的id
        if (mVelocityTracker != null) {//清空之前存入的MotionEvent，也就是说最后的速度只计算该点产生的
            mVelocityTracker.clear();
        }
    }
}
```
通过以上分析，我们得出以下知识：

- 在down事件的时候先判断触摸是否处于边缘，如果是，则不处理
- 在down事件中判断落点是否在子View中，如果不在，不处理
- 在down事件中判断是否仍在滑动，如果是，先停止
- 记录第一个按下点的索引值
- 每次事件都记录住当前的y值
- 在move事件中通过记录的索引值找到对应的点，获取y坐标
- 与上一次y坐标进行比对，scrollBy两次的差值
- 在up事件的时候计算最后一秒钟的速度，并且有最大速度进行限制，当计算的速度大于系统默认的最小速度时，只想fling
- up和cancel事件还原变量为默认值
- 如果为多点离开，进行多点离开的处理
- 该处理方式时：如果离开的是第一个按下的点，那么由第二个按下的点代替其进行y值偏移计算的基点，并清空速度计算的帮助类，重新记录MotionEvnet

# **八、Layout和Scroll的区别**

- Layout中设置的是自身在父View中的显示区域
- Scroll是调整自己的显示区域
- 当父View滚动或者layout变化后，自身在屏幕上的位置会发生变化。
  当自身Scroll滚动后，在屏幕上的显示位置是不变的，变的只是自身的显示内容。
- Scroll滚动不会影响Layout，只是在draw的时候影响画布偏移和触摸时的坐标计算。

![](http://img.blog.csdn.net/20170302181345556?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXhpMjk1MzA5MDY2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![](http://img.blog.csdn.net/20170302181357798?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXhpMjk1MzA5MDY2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![](http://img.blog.csdn.net/20170302181409595?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXhpMjk1MzA5MDY2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)