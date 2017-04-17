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

