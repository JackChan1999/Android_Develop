

# **1. 项目概述**

观察如图2-4 的完整项目中的效果界面，点击标题栏的左上角会弹出侧边栏，再次点击时会关闭侧边栏，这种效果在很多手机应用中使用，因此，我们有必要学会如何自定义一个具有侧边栏效果的控件。

<img src="http://img.blog.csdn.net/20170301144455343?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXhpMjk1MzA5MDY2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" width="300" /> <img src="http://img.blog.csdn.net/20170301144521703?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXhpMjk1MzA5MDY2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" width="300" />

# **2. 布局界面UI**

在本章中，主界面为MainActivity.java，具体代码如文件所示：res/layout/activity_main.xml

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
                xmlns:tools="http://schemas.android.com/tools"
                android:layout_width="match_parent"
                android:layout_height="match_parent" >
    <com.itheima.slidmenudemo.view.SlidMenu
        android:id="@+id/slidmenu"
        android:layout_width="match_parent"
        android:layout_height="match_parent" >
        <!-- 当前为左边菜单界面索引为0 -->
        <include layout="@layout/slidmenu_left" />
        <!-- 当前为主界面索引为1 -->
        <include layout="@layout/slidmenu_main"/>
    </com.itheima.slidmenudemo.view.SlidMenu>
</RelativeLayout>
```

其中，slidmenu_left.xml 是侧边栏的布局文件，具体代码如文件【2-7】所示：【文件2-7】res/layout/slidmenu_left.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
            android:layout_width="240dp"
            android:layout_height="match_parent" >
    <LinearLayout
        android:layout_width="240dp"
        android:layout_height="match_parent"
        android:orientation="vertical"
        android:background="@drawable/menu_bg" >
        <ImageButton
            style="@style/tab_style"
            android:text="新闻"
            android:background="#33000000"
            android:drawableLeft="@drawable/tab_news" />
        <ImageButton
            style="@style/tab_style"
            android:text="订阅"
            android:drawableLeft="@drawable/tab_read" />
        <TextView
            style="@style/tab_style"
            android:text="本地"
            android:drawableLeft="@drawable/tab_local" />

        <TextView
            style="@style/tab_style"
            android:text="跟帖"
            android:drawableLeft="@drawable/tab_ties" />
        <TextView
            style="@style/tab_style"
            android:text="图片"
            android:drawableLeft="@drawable/tab_pics" />
        <TextView
            style="@style/tab_style"
            android:text="话题"
            android:drawableLeft="@drawable/tab_ugc" />
        <TextView
            style="@style/tab_style"
            android:text="投票"
            android:drawableLeft="@drawable/tab_vote" />
        <TextView
            style="@style/tab_style"
            android:text="聚合阅读"
            android:drawableLeft="@drawable/tab_focus" />
    </LinearLayout>
</ScrollView>
```

上面的代码中我们在styles.xml 文件中定义了一个样式tab_style，它是用来修饰左边侧栏中每一个条目，详细代码如下所示。

```xml
<!-- tab 菜单界面的样式-->
<style name="tab_style">
    <item name="android:padding">5dp</item>
    <item name="android:gravity">center_vertical</item>
    <item name="android:drawablePadding">5dp</item>
    <item name="android:layout_width">match_parent</item>
    <item name="android:layout_height">wrap_content</item>
    <item name="android:textSize">18sp</item>
    <item name="android:background">@drawable/tab_bg</item>
    <item name="android:textColor">#ffffff</item>
    <item name="android:onClick">clickTab</item>
    <item name="android:focusable">true</item>
    <item name="android:clickable">true</item>
</style>
```
主界面的右边slidmenu_main.xml 布局文件的代码如文件【2-8】所示。【文件2-8】res/layout/slidmenu_left.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:orientation="vertical" >
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@drawable/top_bar_bg"
        android:orientation="horizontal" >
        <ImageButton
            android:id="@+id/main_back"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:background="@null"
            android:src="@drawable/main_back" />
        <View
            android:layout_width="1dp"
            android:layout_height="match_parent"
            android:layout_margin="5dp"
            android:background="@drawable/top_bar_divider" />
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center_vertical"
            android:layout_marginLeft="10dp"
            android:text="黑马新闻"
            android:textColor="#fff"
            android:textSize="24sp" />
    </LinearLayout>
    <TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center"
        android:text="钓鱼岛是中国的！！！！\nXXX 是世界的"
        android:textColor="#000"
        android:textSize="24sp" />
</LinearLayout>
```

# **3. 主界面业务逻辑**

自定好控件之后，MainActivity,java 主界面的业务逻辑如下所示。activity_setting.xml 如文件【2-9】所示：【文件2-9】res/layout/setup_password_dialog.xml

```java
public class MainActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {

        super.onCreate(savedInstanceState);
        requestWindowFeature(Window.FEATURE_NO_TITLE);
        setContentView(R.layout.activity_main);
        ImageButton main_back = (ImageButton) findViewById(R.id.main_back);
        final SlidMenu slidmenu = (SlidMenu) findViewById(R.id.slidmenu);
        main_back.setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View v) {
                boolean currentView = slidmenu.isMenuShow();
                if(currentView){//当前菜单界面显示，就隐藏
                    slidmenu.hideMenu();
                }else{
                    slidmenu.showMenu();
                }
            }
        });
    }
    public void clickTab(View v){
        TextView tv = (TextView) v;
        Toast.makeText(getApplicationContext(), tv.getText(), 0).show();
    }
}
```

侧边栏在项目会经常用到，这里我们将自定义一个侧边栏，具体代码如文件【2-10】所示。【文件2-10】com/itheima/slidmenudemo/view/SlidMenu.java

```java
public class SlidMenu extends ViewGroup {
    private int downX;
    private final int MAIN_VIEW = 0;// 主界面
    private final int MENU_VIEW = 1;// 左边菜单界面
    private int currentView = MAIN_VIEW;// 记录当前界面，默认为主界面
    private Scroller scroller;// 用来模拟数据
    private int      touchSlop;
    public SlidMenu(Context context, AttributeSet attrs) {
        super(context, attrs);
        init();
    }
    public SlidMenu(Context context) {
        this(context, null);
    }
    @SuppressWarnings("deprecation")
    private void init() {

        scroller = new Scroller(getContext());
        touchSlop = ViewConfiguration.getTouchSlop();// 系统默认你进行了一个滑动操作的固定值
    }
    /**
     * widthMeasureSpec 屏幕的宽度heightMeasureSpec 屏幕的高度
     */
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        // TODO Auto-generated method stub
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        // 测量主界面
        View mainView = getChildAt(1);// 主界面
        // 主界面宽高：屏幕的宽度屏幕的高度
        mainView.measure(widthMeasureSpec, heightMeasureSpec);
        // 测量左边菜单界面和主界面的宽高
        View menuView = getChildAt(0);// 左边菜单界面
        menuView.measure(menuView.getLayoutParams().width, heightMeasureSpec);
    }
    // viewgroup 的排版方法
    /**
     * int l 左边0 int t 上边0 int r 右边屏幕的宽度int b 下边屏幕的高度
     */
    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        // 排版主界面左边0， 高度0， 右边屏幕宽度，下边屏幕高度
        View mainView = getChildAt(1);
        mainView.layout(l, t, r, b);
        // 排版左边菜单界面左边-左边菜单界面的宽度， 高度0 ，右边0， 下边屏幕高度
        View menuView = getChildAt(0);
        menuView.layout(-menuView.getMeasuredWidth(), t, 0, b);
    }
    // 按下的点： downX = 100
    // 移动过后的点： moveX = 150
    // 间距diffX = -50 =100 -150 = downX - moveX;
    // scrollby(diffX,0)
    // downX = moveX = 150
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                downX = (int) event.getX();
                break;
            case MotionEvent.ACTION_MOVE:

                int moveX = (int) event.getX();
                // 计算间距
                int diffX = downX - moveX;
                int scrollX = getScrollX();// 获得当前界面的左上角的X 值
                // 屏幕将要移动到的X 值
                int dff = scrollX + diffX;
                if (dff < -getChildAt(0).getMeasuredWidth()) {// 设置左边界
                    scrollTo(-getChildAt(0).getMeasuredWidth(), 0);
                } else if (dff > 0) {// 设置右边界
                    scrollTo(0, 0);
                } else {// 如果不超过边界值，就要根据屏幕左上角的点的X 值，来计算位置
                    scrollBy(diffX, 0);
                }
                downX = moveX;
                break;
            case MotionEvent.ACTION_UP:
                int center = -getChildAt(0).getMeasuredWidth() / 2;
                if (getScrollX() > center) {
                    System.out.println("显示主界面");
                    currentView = MAIN_VIEW;
                } else {
                    System.out.println("显示左边菜单界面");
                    currentView = MENU_VIEW;
                }
                switchView();
                break;
            default:
                break;
        }
        return true;// 由我们自己来处理触摸事件
    }
    // 根据当前状态值来切换切面
    private void switchView() {
        int startX = getScrollX();
        int dx = 0;
        if (currentView == MAIN_VIEW) {
            // scrollTo(0, 0);
            dx = 0 - startX;
        } else {
            // scrollTo(-getChildAt(0).getMeasuredWidth(), 0);
            dx = -getChildAt(0).getMeasuredWidth() - startX;
        }
        int duration = Math.abs(dx) * 10;
        if (duration > 1000) {

            duration = 1000;
        }
        scroller.startScroll(startX, 0, dx, 0, duration);
        invalidate();
        // scrollTo(scroller.getCurrX(), 0);
    }
    @Override
    public void computeScroll() {
        if (scroller.computeScrollOffset()) {
            int currX = scroller.getCurrX();
            scrollTo(currX, 0);
            invalidate();
        }
    }
    // 判断当前是否是菜单界面,true 就是菜单界面
    public boolean isMenuShow() {
        return currentView == MENU_VIEW;
    }
    // 隐藏菜单界面
    public void hideMenu() {
        currentView = MAIN_VIEW;
        switchView();
    }
    // 显示菜单界面
    public void showMenu() {
        currentView = MENU_VIEW;
        switchView();
    }
    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        switch (ev.getAction()) {
            case MotionEvent.ACTION_DOWN:
                downX = (int) ev.getX();
                break;
            case MotionEvent.ACTION_MOVE:
                int moveX = (int) ev.getX();
                int diff = moveX - downX;
                if (Math.abs(diff) > touchSlop) {
                    return true;
                }
                break;
            case MotionEvent.ACTION_UP:
                break;

            default:
                break;
        }
        return super.onInterceptTouchEvent(ev);
    }
}
```

自定义好侧边栏之后，运行程序，效果图如图2-5 所示。

<img src="http://img.blog.csdn.net/20170217141057409?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXhpMjk1MzA5MDY2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" width="300" /> <img src="http://img.blog.csdn.net/20170217141111461?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXhpMjk1MzA5MDY2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" width="300" /> <img src="http://img.blog.csdn.net/20170217141155067?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXhpMjk1MzA5MDY2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" width="300" />

# **4. ViewDragHelper实现SlidingMenu**

<img src="http://img.blog.csdn.net/20170301144639474?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXhpMjk1MzA5MDY2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" width="300" /> <img src="http://img.blog.csdn.net/20170301144655814?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXhpMjk1MzA5MDY2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" width="300" /> <img src="http://img.blog.csdn.net/20170301144723490?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXhpMjk1MzA5MDY2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" width="300" />

```java
package com.github.slidingmenu.viewdraghelper;

import android.content.Context;
import android.graphics.Color;
import android.support.v4.view.ViewCompat;
import android.support.v4.widget.ViewDragHelper;
import android.support.v4.widget.ViewDragHelper.Callback;
import android.util.AttributeSet;
import android.util.Log;
import android.view.MotionEvent;
import android.view.View;
import android.widget.FrameLayout;

public class SlideMenu2 extends FrameLayout{
	private String TAG = SlideMenu2.class.getSimpleName();

	public SlideMenu2(Context context, AttributeSet attrs) {
		super(context, attrs);
		init();
	}

	private ViewDragHelper viewDragHelper;
	private void init(){
		viewDragHelper = ViewDragHelper.create(this, callback);
	}
	
	private View menuView,mainView;
	private int menuWidth,menuHeight,mainWidth;
	private int dragRange;

	@Override
	protected void onFinishInflate() {
		super.onFinishInflate();
		if(getChildCount()<2){
			throw new IllegalStateException("Your layout must has 2 children or more!");
		}
		menuView = getChildAt(0);
		mainView = getChildAt(1);
		setBackgroundColor(Color.BLACK);
	}
	
//	@Override
//	protected void onLayout(boolean changed, int left, int top, int right,
//			int bottom) {
//		super.onLayout(changed, left, top, right, bottom);
//		menuView.layout(left, top, right, bottom);
//		mainView.layout(left, top, right, bottom);
//	}
	
	
	@Override
	protected void onSizeChanged(int w, int h, int oldw, int oldh) {
		super.onSizeChanged(w, h, oldw, oldh);
		menuWidth = menuView.getMeasuredWidth();
		menuHeight = menuView.getMeasuredHeight();
		mainWidth = mainView.getMeasuredWidth();
		dragRange = (int) (menuWidth * 0.6);
//		ViewHelper.setScaleX(menuView, 0.5f);
//		ViewHelper.setScaleY(menuView, 0.5f);
//		ViewHelper.setTranslationX(menuView, -dragRange/2);
	}
	
	private int lastX,lastY;
	@Override
	public boolean onInterceptTouchEvent(MotionEvent ev) {
		int x = (int) ev.getX();
		int y = (int) ev.getY();
		if(mStatus==Status.Open && viewDragHelper.isViewUnder(mainView, x, y)){
			return true;
		}
//		Log.e(TAG, "onInterceptTouchEvent : "+);
		if(viewDragHelper.isViewUnder(mainView, x, y)){
			switch (ev.getAction()) {
			case MotionEvent.ACTION_MOVE:
				int deltaX = x - lastX;
				int deltaY = y - lastY;
				if(Math.abs(deltaX)>Math.abs(deltaY)*2) {
//					Log.e(TAG, "移动斜角太大，拦截事件");
					viewDragHelper.cancel();
					return true;
				}
				break;
			case MotionEvent.ACTION_UP:
				lastX = 0;
				lastY = 0;
				break;
			}
			lastX = x;
			lastY = y;
		}
		return viewDragHelper.shouldInterceptTouchEvent(ev);
	}
	@Override
	public boolean onTouchEvent(MotionEvent event) {
		int x = (int) event.getX();
		int y = (int) event.getY();
		if(mStatus==Status.Open && viewDragHelper.isViewUnder(mainView, x, y)){
			if(event.getAction()==MotionEvent.ACTION_UP){
				Log.e(TAG, "抬起");
				close();
			}
		}
		viewDragHelper.processTouchEvent(event);
		return true;
	}
	
	private Callback callback = new Callback() {
		@Override
		public boolean tryCaptureView(View child, int pointerId) {
			return child==menuView || child==mainView;
		}
		@Override
		public void onViewDragStateChanged(int state) {
			super.onViewDragStateChanged(state);
		}
		@Override
		public void onViewPositionChanged(View changedView, int left, int top,
				int dx, int dy) {
			super.onViewPositionChanged(changedView, left, top, dx, dy);
//			Log.e(TAG, "onViewPositionChanged   dx: "+dx);
			if(changedView==menuView){
				menuView.layout(0, 0, menuWidth, menuHeight);
				if(mainView.getLeft()>dragRange){
					mainView.layout(dragRange, 0, dragRange+mainWidth, mainView.getBottom());
				}else {
					mainView.layout(mainView.getLeft()+dx, 0, mainView.getRight()+dx, mainView.getBottom());
				}
			}
			
			float percent = mainView.getLeft()/(float)dragRange;
			excuteAnimation(percent);
			
			
			if(mainView.getLeft()==0 && mStatus != Status.Close){
				mStatus = Status.Close;
				if(onDragStatusChangeListener!=null ){
					onDragStatusChangeListener.onClose();
				}
			}else if (mainView.getLeft()==dragRange && mStatus != Status.Open) {
				mStatus = Status.Open;
				if(onDragStatusChangeListener!=null ){
					onDragStatusChangeListener.onOpen();
				}
			}else {
				if(onDragStatusChangeListener!=null){
					onDragStatusChangeListener.onDragging(percent);
				}
			}
			
		}

		private void excuteAnimation(float percent) {
			menuView.setScaleX(0.5f + 0.5f * percent);
			menuView.setScaleY(0.5f + 0.5f * percent);

			mainView.setScaleX(1 - percent * 0.2f);
			mainView.setScaleY( 1 - percent * 0.2f);

			menuView.setTranslationX( -dragRange / 2 + dragRange / 2 * percent);

			menuView.setAlpha(percent);

			getBackground().setAlpha((int) ((1 - percent) * 255));
		}
		@Override
		public void onViewCaptured(View capturedChild, int activePointerId) {
			super.onViewCaptured(capturedChild, activePointerId);
		}
		@Override
		public void onViewReleased(View releasedChild, float xvel, float yvel) {
			super.onViewReleased(releasedChild, xvel, yvel);
//			Log.e(TAG, "onViewReleased ："+(releasedChild==mainView));
			if(mainView.getLeft()>dragRange/2){
				open();
			}else {
				close();
			}
		}
		@Override
		public int getViewHorizontalDragRange(View child) {
			return menuWidth;
		}
		@Override
		public int clampViewPositionHorizontal(View child, int left, int dx) {
			if(child==mainView){
				if(left<0)return 0;
				if(left>dragRange) return dragRange;
			}
			if(child==menuView){
				mainView.layout(mainView.getLeft()+dx, 0, mainView.getRight()+dx, mainView.getBottom());
				menuView.layout(0, 0, menuWidth, menuHeight);
				return 0;
			}
			return left;
		}
	};
	
	public void open(){
		viewDragHelper.smoothSlideViewTo(mainView, dragRange, 0);
		ViewCompat.postInvalidateOnAnimation(SlideMenu2.this);
	}
	
	public void close(){
		viewDragHelper.smoothSlideViewTo(mainView, 0, 0);
		ViewCompat.postInvalidateOnAnimation(SlideMenu2.this);
	}
	
	public void computeScroll() {
		if(viewDragHelper.continueSettling(true)){
			ViewCompat.postInvalidateOnAnimation(this);
		}
	}
	
	public OnDragStatusChangeListener getOnDragStatusChangeListener() {
		return onDragStatusChangeListener;
	}

	public void setOnDragStatusChangeListener(OnDragStatusChangeListener onDragStatusChangeListener) {
		this.onDragStatusChangeListener = onDragStatusChangeListener;
	}

	public Status mStatus = Status.Close;;
	public enum Status{
		Open,Close
	}
	
	private OnDragStatusChangeListener onDragStatusChangeListener;
	
	public interface OnDragStatusChangeListener{
		void onOpen();
		void onClose();
		void onDragging(float dragProgress);
	}
	
}
```

代码：https://github.com/JackChen1999/SlidingMenu

