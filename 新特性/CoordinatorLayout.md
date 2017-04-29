## CoordinatorLayout.Behavior

1. AppBarLayout.Behavior;
2. AppBarLayout.ScrollingViewBehavior;
3. FloatingActionButton.Behavior;
4. Snackbar.Behavior;
5. BottomSheetBehaviro;
6. SwipeDismissBehavior;
7. HeaderBehavior;
8. ViewOffsetBehavior;
9. HeaderScrollingViewBehavior;

## SwipeDismissBehavior

```java
public class SwipeDismissActivity extends AppCompatActivity {

    TextView tv;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_swipedismiss);
        tv = (TextView) findViewById(R.id.tv);

        CoordinatorLayout.LayoutParams params = (CoordinatorLayout.LayoutParams) tv
                .getLayoutParams();
        SwipeDismissBehavior<TextView> behavior = new SwipeDismissBehavior<>();
        params.setBehavior(behavior);

        behavior.setListener(new SwipeDismissBehavior.OnDismissListener() {
            @Override
            public void onDismiss(View view) {
                tv.setVisibility(View.GONE);
                Snackbar.make(view, "删除了一个控件", Snackbar.LENGTH_SHORT)
                        .setAction("确定", new View.OnClickListener() {
                            @Override
                            public void onClick(View view) {
                                tv.setVisibility(View.VISIBLE);
                                ViewCompat.animate(tv).alpha(1).start();
                            }
                        })
                        .show();
            }

            @Override
            public void onDragStateChanged(int state) {

            }
        });
    }
}
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextView
        android:id="@+id/tv"
        android:layout_width="match_parent"
        android:layout_height="56dp"
        android:layout_gravity="center"
        android:layout_margin="16dp"
        android:background="#f00"
        android:clickable="true"
        android:gravity="center"
        android:text="SwipeDismissBehavior"
        android:textColor="#fff"
        android:textSize="20sp"/>

</android.support.design.widget.CoordinatorLayout>
```
```java
public class SwipeDismissBehaviorActivity extends AppCompatActivity {

    private SwipeDismissBehavior swipeDismissBehavior;

    // 如果代码看不懂，本例详解博客：http://blog.csdn.net/yanzhenjie1003/article/details/51938425

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_swipe_dismiss_behavior);
        Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);
        getSupportActionBar().setDisplayHomeAsUpEnabled(true);

        swipeDismissBehavior = new SwipeDismissBehavior();
        swipeDismissBehavior.setSwipeDirection(SwipeDismissBehavior.SWIPE_DIRECTION_ANY);
        swipeDismissBehavior.setListener(onDismissListener);
        CoordinatorLayout.LayoutParams coordinatorParams = (CoordinatorLayout.LayoutParams) findViewById(R.id.textview).getLayoutParams();
        coordinatorParams.setBehavior(swipeDismissBehavior);

        swipeDismissBehavior.setDragDismissDistance(0.5F);
        swipeDismissBehavior.setStartAlphaSwipeDistance(0F);
        swipeDismissBehavior.setEndAlphaSwipeDistance(0.5F);
        swipeDismissBehavior.setSensitivity(0);
        swipeDismissBehavior.setSwipeDirection(SwipeDismissBehavior.SWIPE_DIRECTION_START_TO_END);
    }

    private SwipeDismissBehavior.OnDismissListener onDismissListener = new SwipeDismissBehavior.OnDismissListener() {
        @Override
        public void onDismiss(View view) {
        }

        @Override
        public void onDragStateChanged(int state) {
        }
    };

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        if (item.getItemId() == android.R.id.home) {
            finish();
        }
        return true;
    }
}
```

## 自定义

```java
class MyBehavior extends CoordinatorLayout.Behavior {

    public MyBehavior() {
    }

    public MyBehavior(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    public boolean onStartNestedScroll(CoordinatorLayout coordinatorLayout, View child, View
            directTargetChild, View target, int nestedScrollAxes) {
      	// child 绑定了behavior的view， target 发出触摸事件的view 
        return true;
    }

    @Override
    public void onNestedPreScroll(CoordinatorLayout coordinatorLayout, View child, View
            target, int dx, int dy, int[] consumed) {
      	// dy > 0 向上滑动
        super.onNestedPreScroll(coordinatorLayout, child, target, dx, dy, consumed);
        if (dy < 0) {
            ViewCompat.animate(child).scaleX(1).scaleY(1).start();
        } else {
            ViewCompat.animate(child).scaleX(0).scaleY(0).start();
        }
    }
}
```
代码设置Behavior
```java
CoordinatorLayout.LayoutParams params = (CoordinatorLayout.LayoutParams) tv.getLayoutParams();
params.setBehavior(new MyBehavior());
```
布局文件设置Behavior
```
app:layout_behavior="com.github.nestscrolldemo.MyBehavior"
```
## NestedScrollingChild

将触摸事件发送出去，只有实现了NestedScrollingChild接口的View才可以将自己的触摸事件发送出去

```
class RecyclerView extends ViewGroup implements ScrollingView, NestedScrollingChild

class NestedScrollView extends FrameLayout implements NestedScrollingParent,
        NestedScrollingChild, ScrollingView
```

android5.0之前的控件，例如ListView，没有实现NestedScrollingChild接口，是无法把自己的滚动事件发送出去的

## NestedScrollingParent

处理NestedScrollingChild发出的触摸事件，只有实现了NestedScrollingParent接口的View才能够处理内部子View发出的触摸事件

```
class CoordinatorLayout extends ViewGroup implements NestedScrollingParent
```

## 布局依赖

## BottomSheetBehavior

app:layout_behavior="@string/bottom_sheet_behavior"

BottomSheetBehavior.from(View)

mBottomSheetBehavior.getState()

mBottomSheetBehavior.setState()

setBottomSheetCallback()

BottomSheetBehavior.STATE_COLLAPSED

BottomSheetBehavior.STATE_EXPANDED

app:behavior_hideable="false"说明这个BottomSheet不可以被手动滑动隐藏，设置为true则可以滑到屏幕最底部隐藏

app:behavior_peekHeight设置的是折叠状态时的高度

## BottomSheetDialog

setContentView(view)

isShowing()

dismiss()

show()

app:behavior_hideable=”true”

## FloatingActionButton.Behavior

```
<string name="scale_up_show_behavior">com.yanzhenjie.definebehavior.behavior.ScaleUpShowBehavior</string>
```

```java
public class MyFabBehavior extends CoordinatorLayout.Behavior<View> {

    private static final Interpolator INTERPOLATOR = new FastOutSlowInInterpolator();

    private float viewY;//控件距离coordinatorLayout底部距离
    private boolean isAnimate;//动画是否在进行

    public MyFabBehavior(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    //在嵌套滑动开始前回调
    @Override
    public boolean onStartNestedScroll(CoordinatorLayout coordinatorLayout, View child, View directTargetChild, View target, int nestedScrollAxes) {

        if(child.getVisibility() == View.VISIBLE&&viewY==0){
            //获取控件距离父布局（coordinatorLayout）底部距离
            viewY=coordinatorLayout.getHeight()-child.getY();
        }

        return (nestedScrollAxes & ViewCompat.SCROLL_AXIS_VERTICAL) != 0;//判断是否竖直滚动
    }

    //在嵌套滑动进行时，对象消费滚动距离前回调
    @Override
    public void onNestedPreScroll(CoordinatorLayout coordinatorLayout, View child, View target, int dx, int dy, int[] consumed) {
        //dy大于0是向上滚动 小于0是向下滚动

        if (dy >=0&&!isAnimate&&child.getVisibility()==View.VISIBLE) {
            hide(child);
        } else if (dy <0&&!isAnimate&&child.getVisibility()==View.GONE) {
            show(child);
        }
    }

    //隐藏时的动画
    private void hide(final View view) {
        ViewPropertyAnimator animator = view.animate().translationY(viewY).setInterpolator(INTERPOLATOR).setDuration(200);

        animator.setListener(new Animator.AnimatorListener() {
            @Override
            public void onAnimationStart(Animator animator) {
                isAnimate=true;
            }

            @Override
            public void onAnimationEnd(Animator animator) {
                view.setVisibility(View.GONE);
                isAnimate=false;
            }

            @Override
            public void onAnimationCancel(Animator animator) {
                show(view);
            }

            @Override
            public void onAnimationRepeat(Animator animator) {
            }
        });
        animator.start();
    }

    //显示时的动画
    private void show(final View view) {
        ViewPropertyAnimator animator = view.animate().translationY(0).setInterpolator(INTERPOLATOR).setDuration(200);
        animator.setListener(new Animator.AnimatorListener() {
            @Override
            public void onAnimationStart(Animator animator) {
                view.setVisibility(View.VISIBLE);
                isAnimate=true;
            }

            @Override
            public void onAnimationEnd(Animator animator) {
                isAnimate=false;
            }

            @Override
            public void onAnimationCancel(Animator animator) {
                hide(view);
            }

            @Override
            public void onAnimationRepeat(Animator animator) {
            }
        });
        animator.start();
    }
}
```

## [自定义Behavior支持所有View](http://blog.csdn.net/yanzhenjie1003/article/details/52205665)

```java
public class BasicBehavior<T extends View> extends CoordinatorLayout.Behavior<T> {

    // 源码讲解：http://blog.csdn.net/yanzhenjie1003/article/details/52205665

    private ListenerAnimatorEndBuild listenerAnimatorEndBuild;

    public BasicBehavior(Context context, AttributeSet attrs) {
        super(context, attrs);
        listenerAnimatorEndBuild = new ListenerAnimatorEndBuild();
    }

    // We only support the FAB <> Snackbar shift movement on Honeycomb and above. This is
    // because we can use view translation properties which greatly simplifies the code.
    private static final boolean SNACKBAR_BEHAVIOR_ENABLED = Build.VERSION.SDK_INT >= 11;

    private ValueAnimatorCompat mFabTranslationYAnimator;
    private float mFabTranslationY;
    private Rect mTmpRect;

    @Override
    public boolean layoutDependsOn(CoordinatorLayout parent, T child, View dependency) {
        // We're dependent on all SnackbarLayouts (if enabled)
        return SNACKBAR_BEHAVIOR_ENABLED && dependency instanceof Snackbar.SnackbarLayout;
    }

    @Override
    public boolean onDependentViewChanged(CoordinatorLayout parent, T child, View dependency) {
        if (dependency instanceof Snackbar.SnackbarLayout) {
            updateFabTranslationForSnackbar(parent, child, dependency);
        } else if (dependency instanceof AppBarLayout) {
            // If we're depending on an AppBarLayout we will show/hide it automatically
            // if the FAB is anchored to the AppBarLayout
            updateFabVisibility(parent, (AppBarLayout) dependency, child);
        }
        return false;
    }

    @Override
    public void onDependentViewRemoved(CoordinatorLayout parent, T child, View dependency) {
        if (dependency instanceof Snackbar.SnackbarLayout) {
            updateFabTranslationForSnackbar(parent, child, dependency);
        }
    }

    private boolean updateFabVisibility(CoordinatorLayout parent, AppBarLayout appBarLayout, T child) {
        final CoordinatorLayout.LayoutParams lp = (CoordinatorLayout.LayoutParams) child.getLayoutParams();
        if (lp.getAnchorId() != appBarLayout.getId()) {
            // The anchor ID doesn't match the dependency, so we won't automatically
            // show/hide the FAB
            return false;
        }

        if (child.getVisibility() != View.VISIBLE) {
            // The view isn't set to be visible so skip changing it's visibility
            return false;
        }

        if (mTmpRect == null) {
            mTmpRect = new Rect();
        }

        // First, let's get the visible rect of the dependency
        final Rect rect = mTmpRect;
        ViewGroupUtils.getDescendantRect(parent, appBarLayout, rect);

        if (rect.bottom <= appBarLayout.getMinimumHeightForVisibleOverlappingContent()) {
            if (listenerAnimatorEndBuild.isFinish())
                // If the anchor's bottom is below the seam, we'll animate our FAB out
                scaleHide(child, listenerAnimatorEndBuild.build());
        } else {
            // Else, we'll animate our FAB back in
            scaleShow(child, null);
        }
        return true;
    }

    private void updateFabTranslationForSnackbar(CoordinatorLayout parent, final T fab, View snackbar) {
        final float targetTransY = getFabTranslationYForSnackBar(parent, fab);
        if (mFabTranslationY == targetTransY) {
            // We're already at (or currently animating to) the target value, return...
            return;
        }

        final float currentTransY = ViewCompat.getTranslationY(fab);

        // Make sure that any current animation is cancelled
        if (mFabTranslationYAnimator != null && mFabTranslationYAnimator.isRunning()) {
            mFabTranslationYAnimator.cancel();
        }

        if (fab.isShown()
                && Math.abs(currentTransY - targetTransY) > (fab.getHeight() * 0.667f)) {
            // If the FAB will be travelling by more than 2/3 of it's height, let's animate
            // it instead
            if (mFabTranslationYAnimator == null) {
                mFabTranslationYAnimator = ViewUtils.createAnimator();
                mFabTranslationYAnimator.setInterpolator(
                        AnimationUtils.FAST_OUT_SLOW_IN_INTERPOLATOR);
                mFabTranslationYAnimator.setUpdateListener(
                        new ValueAnimatorCompat.AnimatorUpdateListener() {
                            @Override
                            public void onAnimationUpdate(ValueAnimatorCompat animator) {
                                ViewCompat.setTranslationY(fab, animator.getAnimatedFloatValue());
                            }
                        });
            }
            mFabTranslationYAnimator.setFloatValues(currentTransY, targetTransY);
            mFabTranslationYAnimator.start();
        } else {
            // Now update the translation Y
            ViewCompat.setTranslationY(fab, targetTransY);
        }

        mFabTranslationY = targetTransY;
    }

    private float getFabTranslationYForSnackBar(CoordinatorLayout parent, T fab) {
        float minOffset = 0;
        final List<View> dependencies = parent.getDependencies(fab);
        for (int i = 0, z = dependencies.size(); i < z; i++) {
            final View view = dependencies.get(i);
            if (view instanceof Snackbar.SnackbarLayout && parent.doViewsOverlap(fab, view)) {
                minOffset = Math.min(minOffset, ViewCompat.getTranslationY(view) - view.getHeight());
            }
        }

        return minOffset;
    }

    @Override
    public boolean onLayoutChild(CoordinatorLayout parent, T child, int layoutDirection) {
        // First, lets make sure that the visibility of the FAB is consistent
        final List<View> dependencies = parent.getDependencies(child);
        for (int i = 0, count = dependencies.size(); i < count; i++) {
            final View dependency = dependencies.get(i);
            if (dependency instanceof AppBarLayout && updateFabVisibility(parent, (AppBarLayout) dependency, child)) {
                break;
            }
        }
        // Now let the CoordinatorLayout lay out the FAB
        parent.onLayoutChild(child, layoutDirection);
        return true;
    }

    public static final ViewPropertyAnimatorListener DEFAULT_OUT_ANIMATOR_LISTENER = new ViewPropertyAnimatorListener() {
        @Override
        public void onAnimationStart(View view) {
        }

        @Override
        public void onAnimationEnd(View view) {
            view.setVisibility(View.GONE);
        }

        @Override
        public void onAnimationCancel(View view) {
        }
    };

    public static class ListenerAnimatorEndBuild {

        private boolean isOutExecute = false;

        private ViewPropertyAnimatorListener outAnimatorListener;

        public ListenerAnimatorEndBuild() {
            outAnimatorListener = new ViewPropertyAnimatorListener() {
                @Override
                public void onAnimationStart(View view) {
                    isOutExecute = true;
                }

                @Override
                public void onAnimationEnd(View view) {
                    view.setVisibility(View.GONE);
                    isOutExecute = false;
                }

                @Override
                public void onAnimationCancel(View view) {
                    isOutExecute = false;
                }
            };
        }

        public boolean isFinish() {
            return !isOutExecute;
        }

        public ViewPropertyAnimatorListener build() {
            return outAnimatorListener;
        }
    }

    public static final FastOutSlowInInterpolator FASTOUTSLOWININTERPOLATOR = new FastOutSlowInInterpolator();

    public static void scaleShow(View view) {
        scaleShow(view, null);
    }

    public static void scaleShow(View view, ViewPropertyAnimatorListener viewPropertyAnimatorListener) {
        view.setVisibility(View.VISIBLE);
        ViewCompat.animate(view)
                .scaleX(1.0f)
                .scaleY(1.0f)
                .alpha(1.0f)
                .setDuration(800)
                .setInterpolator(FASTOUTSLOWININTERPOLATOR)
                .setListener(viewPropertyAnimatorListener)
                .start();
    }

    public static void scaleHide(View view) {
        scaleHide(view, DEFAULT_OUT_ANIMATOR_LISTENER);
    }

    public static void scaleHide(View view, ViewPropertyAnimatorListener viewPropertyAnimatorListener) {
        ViewCompat.animate(view)
                .scaleX(0.0f)
                .scaleY(0.0f)
                .alpha(0.0f)
                .setDuration(800)
                .setInterpolator(FASTOUTSLOWININTERPOLATOR)
                .setListener(viewPropertyAnimatorListener)
                .start();
    }
}
```

```java
public class DefineBehavior extends BasicBehavior<View> {

    // 源码讲解：http://blog.csdn.net/yanzhenjie1003/article/details/52205665

    private ListenerAnimatorEndBuild listenerAnimatorEndBuild;

    public DefineBehavior(Context context, AttributeSet attrs) {
        super(context, attrs);
        listenerAnimatorEndBuild = new ListenerAnimatorEndBuild();
    }

    @Override
    public boolean onStartNestedScroll(CoordinatorLayout coordinatorLayout, View child, View directTargetChild, View target, int nestedScrollAxes) {
        return nestedScrollAxes == ViewCompat.SCROLL_AXIS_VERTICAL;
    }

    @Override
    public void onNestedScroll(CoordinatorLayout coordinatorLayout, View child, View target, int dxConsumed, int dyConsumed, int dxUnconsumed, int dyUnconsumed) {
//        if (dyConsumed > 0 && dyUnconsumed == 0) {
//            System.out.println("上滑中。。。");
//        }
//        if (dyConsumed == 0 && dyUnconsumed > 0) {
//            System.out.println("到边界了还在上滑。。。");
//        }
//        if (dyConsumed < 0 && dyUnconsumed == 0) {
//            System.out.println("下滑中。。。");
//        }
//        if (dyConsumed == 0 && dyUnconsumed < 0) {
//            System.out.println("到边界了，还在下滑。。。");
//        }

        // 这里可以写你的其他逻辑动画，这里只是举例子写了个缩放动画。
        if ((dyConsumed > 0 || dyUnconsumed > 0) && listenerAnimatorEndBuild.isFinish() && child.getVisibility() == View.VISIBLE) {//往下滑
            scaleHide(child, listenerAnimatorEndBuild.build());
        } else if ((dyConsumed < 0 || dyUnconsumed < 0) && child.getVisibility() != View.VISIBLE) {
            scaleShow(child, null);
        }
    }
}
```

```java
public class ScaleDownShowBehavior extends FloatingActionButton.Behavior {

    private BasicBehavior.ListenerAnimatorEndBuild listenerAnimatorEndBuild;

    private OnStateChangedListener mOnStateChangedListener;

    public ScaleDownShowBehavior(Context context, AttributeSet attrs) {
        super();
        listenerAnimatorEndBuild = new BasicBehavior.ListenerAnimatorEndBuild();
    }

    @Override
    public boolean onStartNestedScroll(CoordinatorLayout coordinatorLayout, FloatingActionButton child, View directTargetChild, View target, int nestedScrollAxes) {
        return nestedScrollAxes == ViewCompat.SCROLL_AXIS_VERTICAL;
    }

    @Override
    public void onNestedScroll(CoordinatorLayout coordinatorLayout, FloatingActionButton child, View target, int dxConsumed, int dyConsumed, int dxUnconsumed, int dyUnconsumed) {
        if ((dyConsumed > 0 || dyUnconsumed > 0) && listenerAnimatorEndBuild.isFinish() && child.getVisibility() == View.VISIBLE) {//往下滑
            BasicBehavior.scaleHide(child, listenerAnimatorEndBuild.build());
            if (mOnStateChangedListener != null) {
                mOnStateChangedListener.onChanged(false);
            }
        } else if ((dyConsumed < 0 || dyUnconsumed < 0) && child.getVisibility() != View.VISIBLE) {
            BasicBehavior.scaleShow(child, null);
            if (mOnStateChangedListener != null) {
                mOnStateChangedListener.onChanged(true);
            }
        }
    }

    public void setOnStateChangedListener(OnStateChangedListener mOnStateChangedListener) {
        this.mOnStateChangedListener = mOnStateChangedListener;
    }

    // 外部监听显示和隐藏。
    public interface OnStateChangedListener {
        void onChanged(boolean isShow);
    }

    public static <V extends View> ScaleDownShowBehavior from(V view) {
        ViewGroup.LayoutParams params = view.getLayoutParams();
        if (!(params instanceof CoordinatorLayout.LayoutParams)) {
            throw new IllegalArgumentException("The view is not a child of CoordinatorLayout");
        }
        CoordinatorLayout.Behavior behavior = ((CoordinatorLayout.LayoutParams) params).getBehavior();
        if (!(behavior instanceof ScaleDownShowBehavior)) {
            throw new IllegalArgumentException("The view is not associated with ScaleDownShowBehavior");
        }
        return (ScaleDownShowBehavior) behavior;
    }
}
```

```java
public class ScaleUpShowBehavior extends FloatingActionButton.Behavior {

    private BasicBehavior.ListenerAnimatorEndBuild listenerAnimatorEndBuild;

    public ScaleUpShowBehavior(Context context, AttributeSet attrs) {
        super();
        listenerAnimatorEndBuild = new BasicBehavior.ListenerAnimatorEndBuild();
    }

    @Override
    public boolean onStartNestedScroll(CoordinatorLayout coordinatorLayout, FloatingActionButton child, View directTargetChild, View target, int nestedScrollAxes) {
        return nestedScrollAxes == ViewCompat.SCROLL_AXIS_VERTICAL;
    }

    @Override
    public void onNestedScroll(CoordinatorLayout coordinatorLayout, FloatingActionButton child, View target, int dxConsumed, int dyConsumed, int dxUnconsumed, int dyUnconsumed) {
//        System.out.println("onNestedScroll- dxConsumed:" + dxConsumed + "; dyConsumed: " + dyConsumed + "; dxUnconsumed: " + dxUnconsumed + "; dyUnconsumed: " + dyUnconsumed);

//        if (dyConsumed > 0 && dyUnconsumed == 0) {
//            System.out.println("上滑中。。。");
//        }
//        if (dyConsumed == 0 && dyUnconsumed > 0) {
//            System.out.println("到边界了还在上滑。。。");
//        }
//        if (dyConsumed < 0 && dyUnconsumed == 0) {
//            System.out.println("下滑中。。。");
//        }
//        if (dyConsumed == 0 && dyUnconsumed < 0) {
//            System.out.println("到边界了，还在下滑。。。");
//        }

        if (((dyConsumed > 0 && dyUnconsumed == 0) || (dyConsumed == 0 && dyUnconsumed > 0)) && child.getVisibility() != View.VISIBLE) {// 显示
            BasicBehavior.scaleShow(child, null);
        } else if (((dyConsumed < 0 && dyUnconsumed == 0) || (dyConsumed == 0 && dyUnconsumed < 0)) && child.getVisibility() != View.GONE && listenerAnimatorEndBuild.isFinish()) {
            BasicBehavior.scaleHide(child, listenerAnimatorEndBuild.build());
        }
    }
}
```