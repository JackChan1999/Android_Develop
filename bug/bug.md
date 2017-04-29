## IndexOutOfBoundsException

java.lang.IndexOutOfBoundsException: Invalid index 0, size is 0

## 构造方法和成员变量初始化顺序

- 执行父类静态代码 执行子类静态代码
- 初始化父类成员变量（我们常说的赋值语句）
- 初始化父类构造函数
- 初始化子类成员变量
- 初始化子类构造函数

```java
public abstract class BaseMenuDetailPager {

	public Activity mActivity;
	public View mRootView;

	public BaseMenuDetailPager(Activity activity) {
		mActivity = activity;
		mRootView = initView();
		// initData();
	}

	protected abstract View initView();

	public void initData() {
	}

}
```

```java
public abstract class BaseTabDetailPager extends BaseMenuDetailPager{
    
  	protected MultiItemTypeAdapter<NewsData> mMultiItemTypeAdapter;
    protected TopNewsAdapter  mTopNewsAdapter;

  	// 在initView()方法调用后才初始化
    protected List<NewsData>     mNewsData = new ArrayList<>();
    protected List<NewsData.Ads> mAdsData = new ArrayList<>();
  
    @Override
    protected View initView() {
      ...
      // mNewsData，mAdsData空指针异常
      mMultiItemTypeAdapter = new MultiItemTypeAdapter<>(mActivity, mNewsData);
      mTopNewsAdapter = new TopNewsAdapter(mAdsData);
    }
}
```

先初始化父类BaseMenuDetailPager的构造方法，在父类的构造方法中调用的initView()方法，而此时子类BaseTabDetailPager的成员变量还没有初始化，即mNewsData和mAdsData还没有初始化，因此导致了空指针异常

## 权限问题

没有申请权限，例如：网络权限，读写SDcard权限，读取联系人权限

## RecyclerView

没有设置setLayoutManager()

## Application没有注册

```xml
<application
        android:name=".base.BaseApplication"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme">
```
## method.invoke()
```
警告: 最后一个参数使用了不准确的变量类型的 varargs 方法的非 varargs 调用;
对于 varargs 调用, 应使用 Object
对于非 varargs 调用, 应使用 Object[], 这样也可以抑制此警告
```
```
method.invoke(obj, null);
method.invoke(obj, new Object[]{});
```