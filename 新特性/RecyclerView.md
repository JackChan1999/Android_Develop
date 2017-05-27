## RecyclerView介绍

该控件用于在有限的窗口中展示大量数据集，可以用于替代ListView、GridView，通过设置RecyclerView的LayoutManager可以轻松的通过这个控件实现ListView,GirdView，瀑布流等效果。

### RecyclerView.LayoutManager:

- LinearLayoutManager 线性管理器，支持横向、纵向。- 实现ListView的布局
- GridLayoutManager 网格布局管理器  - 实现GridView的布局
- StaggeredGridLayoutManager 瀑布流式布局管理器

## RecyclerView.Adapter介绍

### 普通的ListView的Adapter的抽取

普通的ListView的Adapter的定义步骤：

1. 继承BaseAdapter

2. 必须重写getCount和getView方法

3. 在重写getView方法的时候，为了优化ListView，需要：
- 复用convertView
- 使用ViewHolder

### NormalAdapter

```java
/**
 * 普通的ListView的Adapter
 */
public class NormalAdapter extends BaseAdapter {

    /** ListView要展示的数据 */
    ArrayList<Object> mDatas;

    /**  上下文环境*/
    Context context;

    public NormalAdapter(Context context, ArrayList<Object> datas) {
        this.context = context;
        mDatas = datas;
    }

    @Override
    public int getCount() {
        return mDatas.size();
    }

    @Override
    public Object getItem(int position) {
        return mDatas.get(position);
    }

    @Override
    public long getItemId(int position) {
        return position;
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        ViewHolder holder;
        if (convertView == null) {
            convertView = View.inflate(context, R.layout.list_item, null);
            holder = new ViewHolder();
            holder.icon = (ImageView) convertView.findViewById(R.id.icon);
            holder.name = (TextView) convertView.findViewById(R.id.name);
            holder.subname = (TextView) convertView.findViewById(R.id.subname);
            convertView.setTag(holder);
        } else {
            holder = (ViewHolder) convertView.getTag();
        }

        // 设置数据...
        Object data = mDatas.get(position);
        holder.name.setText(data.toString());

        return convertView;
    }

    class ViewHolder {
        ImageView icon;
        TextView name;
        TextView subname;
    }
}
```

既然所有的getView方法都需要构建ViewHolder，进行findViewById操作，并为ViewHolder中的控件赋值，那么我们可以把构建ViewHolder和赋值的操作抽取成方法

### UpgradeNormalAdapter

```java
/**
 * 升级版ListView的Adapter
 */
public class UpgradeNormalAdapter extends BaseAdapter {

    /** ListView要展示的数据 */
    ArrayList<Object> mDatas;

    /**  上下文环境*/
    Context context;

    public UpgradeNormalAdapter(Context context, ArrayList<Object> datas) {
        this.context = context;
        mDatas = datas;
    }

    @Override
    public int getCount() {
        return mDatas.size();
    }

    @Override
    public Object getItem(int position) {
        return mDatas.get(position);
    }

    @Override
    public long getItemId(int position) {
        return position;
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        ViewHolder holder;
        if (convertView == null) {
            holder = onCreateViewHolder(parent);
        } else {
            holder = (ViewHolder) convertView.getTag();
        }
        // 设置数据...
        onBindViewHolder(holder, position);
        return holder.rootView;
    }

    /**
     *  构建ViewHolder
     */
    public ViewHolder onCreateViewHolder(ViewGroup parent) {
        View view = View.inflate(context, R.layout.list_item, null);
        return new ViewHolder(view);
    }

    /**
     *  设置要显示的数据
     */
    public void onBindViewHolder(ViewHolder holder, int position) {
        Object data = mDatas.get(position);
        holder.name.setText(data.toString());
    }

    public class ViewHolder {
        ImageView icon;
        TextView  name;
        TextView  subname;
        View      rootView;

        public ViewHolder(View convertView) {
            rootView = convertView;
            icon = (ImageView) convertView.findViewById(R.id.icon);
            name = (TextView) convertView.findViewById(R.id.name);
            subname = (TextView) convertView.findViewById(R.id.subname);
            convertView.setTag(this);
        }
    }
}
```

每个ListView的Adapter的getView方法中的ViewHolder和设置值的内容都是不一样的，那么我们可以抽取一个父类出来：

- 既然不知道ViewHolder是谁，由子类决定，那么就用一个泛型T来代表ViewHolder；
- 既然不知道创建ViewHolder的具体方法和设置值的具体方法，那么就将这两个方法做成抽象的，由子类去实现

### BaseUpgradeNormalAdapter

```java
/**
 * 升级版ListView的Adapter -- 抽取父类
 */
public abstract  class BaseUpgradeNormalAdapter<T extends BaseUpgradeNormalAdapter.ViewHolder> extends BaseAdapter {

    /** ListView要展示的数据 */
    ArrayList<Object> mDatas;

    /**  上下文环境*/
    Context context;

    public BaseUpgradeNormalAdapter(Context context, ArrayList<Object> datas) {
        this.context = context;
        mDatas = datas;
    }

    @Override
    public int getCount() {
        return mDatas.size();
    }

    @Override
    public Object getItem(int position) {
        return mDatas.get(position);
    }

    @Override
    public long getItemId(int position) {
        return position;
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        T holder;
        if (convertView == null) {
            holder = onCreateViewHolder(parent);
        } else {
            holder = (T) convertView.getTag();
        }
        // 设置数据...
        onBindViewHolder(holder, position);
        return holder.rootView;
    }

    public abstract T onCreateViewHolder(ViewGroup parent);

    public abstract void onBindViewHolder(T holder, int position);

    public class ViewHolder {
        View rootView;

        public ViewHolder(View convertView) {
            rootView = convertView;
            convertView.setTag(this);
        }
    }
}
```

那么，实现了这个父类的子类只需关注自己需要重写的方法即可

### SubUpgradeNormalAdapter

```java
/**
 * 升级版ListView的Adapter -- 子类
 */
public  class SubUpgradeNormalAdapter extends BaseUpgradeNormalAdapter<SubUpgradeNormalAdapter.ListViewHolder> {

    public SubUpgradeNormalAdapter(Context context, ArrayList datas) {
        super(context, datas);
    }

    @Override
    public ListViewHolder onCreateViewHolder(ViewGroup parent) {
        View view = View.inflate(context, R.layout.list_item, null);
        return new ListViewHolder(view);
    }

    @Override
    public void onBindViewHolder(ListViewHolder holder, int position) {
        Object data = mDatas.get(position);
        holder.name.setText(data.toString());
    }

    public class ListViewHolder extends BaseUpgradeNormalAdapter.ViewHolder {
        ImageView icon;
        TextView  name;
        TextView  subname;

        public ListViewHolder(View convertView) {
            super(convertView);
            icon = (ImageView) convertView.findViewById(R.id.icon);
            name = (TextView) convertView.findViewById(R.id.name);
            subname = (TextView) convertView.findViewById(R.id.subname);
        }
    }
}
```

### RecyclerView的Adapter

RecyclerView的Adapter其实也是遵循这个思想去做的抽取，它的使用步骤：

1. 继承自RecyclerView.Adapter，指定ViewHolder的类型
2. 重写getItemCount方法，返回要展示的数据量
3. 自定义一个ViewHolder，继承自RecyclerView.ViewHolder
4. 重写onCreateViewHolder方法，返回被创建的ViewHolder对象
5. 重写onBindViewHolder方法，绑定要展示的数据

```java
/**
 * RecyclerView的Adapter
 */
public class ListRecyclerAdapter extends RecyclerView.Adapter<ListRecyclerAdapter.MyRecyclerViewHolder>{

    private ArrayList<String> mDatas;

    private Context context;

    public ListRecyclerAdapter(Context context, ArrayList<String> mDatas) {
        this.mDatas = mDatas;
        this.context = context;
    }

    @Override
    public MyRecyclerViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View view = View.inflate(context, R.layout.list_item, null);
        return new MyRecyclerViewHolder(view);
    }

    @Override
    public int getItemCount() {
        return mDatas == null ? 0 : mDatas.size();
    }

    @Override
    public void onBindViewHolder(MyRecyclerViewHolder holder, int position) {
        String data = mDatas.get(position);
        holder.mTvName.setText(data);
    }

    public  class MyRecyclerViewHolder extends RecyclerView.ViewHolder {
        TextView mTvName;
        public MyRecyclerViewHolder(View itemView) {
            super(itemView);
            mTvName = (TextView) itemView.findViewById(R.id.name);
        }
    }
}
```

## RecyclerView的使用

### 添加RecyclerView的依赖库：

```gradle
compile 'com.android.support:recyclerview-v7:23.4.0'
```

### 在布局中使用RecyclerView
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
  android:layout_width="match_parent"
  android:layout_height="match_parent"
  android:orientation="vertical">

  <android.support.v7.widget.RecyclerView
  android:id="@+id/recyclerView"
  android:layout_width="match_parent"
  android:layout_height="match_parent"/>

</LinearLayout>
```
### 定义RecyclerView的Adapter
```java
   /**
	*  RecyclerView的Adapter
    */
   public class ListRecyclerAdapter extends RecyclerView.Adapter<ListRecyclerAdapter.MyRecyclerViewHolder>{

    private ArrayList<String> mDatas;
    private Context context;

    public ListRecyclerAdapter(Context context, ArrayList<String> mDatas) {
        this.mDatas = mDatas;
        this.context = context;
    }

    @Override
    public MyRecyclerViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View view = View.inflate(context, R.layout.list_item, null);
        return new MyRecyclerViewHolder(view);
    }

    @Override
    public int getItemCount() {
        return mDatas == null ? 0 : mDatas.size();
    }

    @Override
    public void onBindViewHolder(MyRecyclerViewHolder holder, int position) {
        String data = mDatas.get(position);
        holder.mTvName.setText(data);
    }

    public  class MyRecyclerViewHolder extends RecyclerView.ViewHolder {
        TextView mTvName;
        public MyRecyclerViewHolder(View itemView) {
            super(itemView);
            mTvName = (TextView) itemView.findViewById(R.id.name);
        }
    }
   }
```

### 给RecyclerView设置LayoutManager
```java
// 设置RecyclerView的LayoutManager
LinearLayoutManager layoutManager = new LinearLayoutManager(this, mCurrentOrientation, false);
mRecyclerView.setLayoutManager(layoutManager);
```
### 给RecyclerView设置Adapter
```java
// 设置Adapter
if (mAdapter == null) {
  mAdapter = new ListRecyclerAdapter(this, mDatas);
  mRecyclerView.setAdapter(mAdapter);
} else {
  mAdapter.notifyDataSetChanged();
}
```