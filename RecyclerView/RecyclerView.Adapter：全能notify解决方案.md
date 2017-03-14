在之前我们用 ListView 或者 GridView 的时候，通知适配器刷新是这样的：
```java
adapter.notifyDataSetChanged();
```

但是当我们使用了更强大的 RecyclerView 之后，如果直接这样通知适配器刷新将不会显示动画效果。它会直接将所有的 item 重新绘制。

我们需要使用如下的方法来通知适配器刷新，这样 RecyclerView 才会显示对应的动画效果：

```java
adapter.notifyItemInserted();
adapter.notifyItemChanged();
adapter.notifyItemMoved();
adapter.notifyItemRemoved();
adapter.notifyItemRangeChanged();
adapter.notifyItemRangeInserted();
adapter.notifyItemRangeRemoved();
```

在这次更新的 Support Library 24.2.0 中添加了一个新的工具类，可以用来方便快捷的处理 RecyclerView.Adapter 的通知刷新。

# DiffUtil

DifUtil 就是这次引入的工具类，它会找出 Adapter 中每一个 Item 对应发生的变化，然后对每一个变化给予对应的刷新。

最重要的就是如下的两个重载方法

```java
DifUtil.calculateDiff(Callback cb, boolean detectMoves);
DifUtil.calculateDiff(Callback cb);
```

其中DifUtil.calculateDiff(Callback cb);实际上就是DifUtil.calculateDiff(callback, true);所以我们着重研究第一个方法即可。

该方法会接收两个参数，其中第二个参数是一个 boolean 值，查看源码注释我们知道这个参数有如下作用：

True if DiffUtil should try to detect moved items, false otherwise.
如果 DiffUtil 尝试检测移动的项目就设为 true，否则设为 false。
这个参数实际上是指定是否需要项目移动的检测，如果设为 false ，那么一个项目移动了会先判定为 remove，再判定为 insert。

而Callback是一个抽象类，它有四个方法需要实现：

```java
public abstract static class Callback {
    /**
     * 旧的数据源的大小
     */
    public abstract int getOldListSize();
    /**
     * 新的数据源的大小
     */
    public abstract int getNewListSize();
    /**
     * 该方法用于判断两个 Object 是否是相同的 Item，比如有唯一标识的时候应该比较唯一标识是否相等
     */
    public abstract boolean areItemsTheSame(int oldItemPosition, int newItemPosition);
    /**
     * 当 areItemsTheSame 返回 true 时调用该方法，返回显示的 Item 的内容是否一致
     */
    public abstract boolean areContentsTheSame(int oldItemPosition, int newItemPosition);
}
```

如上所述，我们四个需要实现的方法的作用都在注释中写出了。前两个方法都很好理解，需要重点说明的是后两个

areItemsTheSame：这个方法用来判断两个 Object 是否是相同的 Item，此处最好不要简单的用equals方法判断，我们可以根据 Object 的唯一标识或者自己指定一个规则来判断两个 Object 是否是展示的相同的 Item。
areContentsTheSame：该方法只有在areItemsTheSame返回true之后才会被调用，我们在重写该方法的时候，只需要判断两个 Object 显示的元素是否一致即可。如我们有两个 Object，它们可能拥有很多属性，但是其中只有两个属性需要被显示出来，那只要这两个属性一致我们这个方法就要返回true。
使用 DiffUtils 通知刷新

下面我们写一个简单的例子来学习使用 DiffUtil

首先我们来一个 Item 对应的数据类：

```java
public class Student {
    public String id; // 学号是唯一的
    public String name; // 名字可能重复
    public Student(String id, String name) {
        this.id = id;
        this.name = name;
    }
}
```

然后写一个 Adapter：

```java
class MyAdapter extends RecyclerView.Adapter<MyAdapter.ViewHolder> {
    private final List<Student> datas;
    public MyAdapter(List<Student> datas) {
        this.datas = datas;
    }
    @Override
    public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View view = LayoutInflater.from(parent.getContext())
                .inflate(R.layout.item_recycler, parent, false);
        return new ViewHolder(view);
    }
    @Override
    public void onBindViewHolder(ViewHolder holder, int position) {
        holder.setData(datas.get(position));
    }
    @Override
    public int getItemCount() {
        return datas.size();
    }
    class ViewHolder extends RecyclerView.ViewHolder {
        public ViewHolder(View itemView) {
            super(itemView);
        }
        public void setData(Student student) {
            TextView textView = (TextView) this.itemView.findViewById(R.id.text);
            textView.setText(student.name);
        }
    }
}
```

其对应的布局文件就是一个简单的 TextView：

```xml
<?xml version="1.0" encoding="utf-8"?>
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
          xmlns:tools="http://schemas.android.com/tools"
          android:id="@+id/text"
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          android:gravity="center"
          android:orientation="vertical"
          android:padding="10dp"
          tools:text="content"/>
```

然后我们在 Activity 里使用它们并显示出来：

```java
class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        // ...
        mRandom = new Random();
        datas = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            datas.add(new Student(mRandom.nextInt(3000) + "", "Students: " + i));
        }
        mRecyclerView = (RecyclerView) findViewById(R.id.recycler_view);
        mRecyclerView.setLayoutManager(new LinearLayoutManager(this));
        mAdapter = new MyAdapter(datas);
        mRecyclerView.setAdapter(mAdapter);
        // ...
    }
}
```

这样我们就获得了一个简单的展示学生数据的 RecyclerView 了。

然后我们对 Adapter 的数据源进行更改，并通知刷新：

```java
mFab.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                // 创建一个原来的 List 的副本
                final ArrayList<Student> oldTemp = new ArrayList<>(datas);
                // 更改原数据源
                datas.remove(mRandom.nextInt(mAdapter.getItemCount()));
                for (int i = 0; i < mRandom.nextInt(3); i++) {
                    datas.add(mRandom.nextInt(mAdapter.getItemCount() - 1),
                            new Student(mRandom.nextInt(3000) + "", "Students: " + mRandom.nextDouble()));
                }
                // 实现 Callback
                DiffUtil.Callback callback = new DiffUtil.Callback() {
                    @Override
                    public int getOldListSize() {
                        return oldTemp.size();
                    }
                    @Override
                    public int getNewListSize() {
                        return datas.size();
                    }
                    @Override
                    public boolean areItemsTheSame(int oldItemPosition, int newItemPosition) {
                        return oldTemp.get(oldItemPosition).id.equals(datas.get(newItemPosition).id);
                    }
                    @Override
                    public boolean areContentsTheSame(int oldItemPosition, int newItemPosition) {
                        return oldTemp.get(oldItemPosition).name.equals(datas.get(newItemPosition).name);
                    }
                };
                DiffUtil.DiffResult diffResult = DiffUtil.calculateDiff(callback);
                // 把结果应用到 adapter
                diffResult.dispatchUpdatesTo(mAdapter);
            }
        });
```

效果如下：

![DiffUtil](http://7xl94a.com1.z0.glb.clouddn.com/GIF_20160825_093648.gif)

DiffUtil 的使用就是这样，根据 DiffUtil.Callback 计算出 Result，然后应用更新到 Adapter。

#封装

有的人可能说了，这样其实并不好用啊，我们原来数据的改变就直接使用对应的方法就可以了，你这里每次还要写得这么麻烦。那么我们就使用 DiffUtil 和 Adapter 结合再进行一次封装吧。

我们抽取一个 BaseAdapter 出来：

```java
public abstract class BaseAdapter<T, V extends RecyclerView.ViewHolder>
        extends RecyclerView.Adapter<V>{
    protected final List<T> temp; // 用于保存修改之前的数据源的副本
    protected final List<T> datas; // 数据源
    public BaseAdapter(List<T> datas) {
        this.datas = datas;
        temp = new ArrayList<>(datas);
    }
    protected abstract boolean areItemsTheSame(T oldItem, T newItem);
    protected abstract boolean areContentsTheSame(T oldItem, T newItem);
    @Override
    public int getItemCount() {
        return datas.size();
    }
    public void notifyDiff() {
        DiffUtil.DiffResult diffResult = DiffUtil.calculateDiff(new DiffUtil.Callback() {
            @Override
            public int getOldListSize() {
                return temp.size();
            }
            @Override
            public int getNewListSize() {
                return datas.size();
            }
            // 判断是否是同一个 item
            @Override
            public boolean areItemsTheSame(int oldItemPosition, int newItemPosition) {
                return BaseAdapter.this.areItemsTheSame(temp.get(oldItemPosition), datas.get(newItemPosition));
            }
            // 如果是同一个 item 判断内容是否相同
            @Override
            public boolean areContentsTheSame(int oldItemPosition, int newItemPosition) {
                return BaseAdapter.this.areContentsTheSame(temp.get(oldItemPosition), datas.get(newItemPosition));
            }
        });
        diffResult.dispatchUpdatesTo(this);
        // 通知刷新了之后，要更新副本数据到最新
        temp.clear();
        temp.addAll(datas);
    }
}
```

然后我们只需要令 Adapter 实现 BaseAdapter即可：

```java
class MyAdapter extends BaseAdapter<Student, MyAdapter.ViewHolder> {
    public MyAdapter(List<Student> datas) {
        super(datas);
    }
    @Override
    public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View view = LayoutInflater.from(parent.getContext())
                .inflate(R.layout.item_recycler, parent, false);
        return new ViewHolder(view);
    }
    @Override
    public void onBindViewHolder(ViewHolder holder, int position) {
        holder.setData(datas.get(position));
    }
    @Override
    public boolean areItemsTheSame(Student oldItem, Student newItem) {
        return oldItem.id.equals(newItem.id);
    }
    @Override
    public boolean areContentsTheSame(Student oldItem, Student newItem) {
        return oldItem.name.equals(newItem.name);
    }
    class ViewHolder extends RecyclerView.ViewHolder {
        public ViewHolder(View itemView) {
            super(itemView);
        }
        public void setData(Student student) {
            TextView textView = (TextView) this.itemView.findViewById(R.id.text);
            textView.setText(student.name);
        }
    }
}
```

之后我们如果数据源 List 中的数据有任何改动，我们只需要调用notifyDiff()就可以了：

```java
mFab.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                datas.remove(mRandom.nextInt(mAdapter.getItemCount()));
                for (int i = 0; i < mRandom.nextInt(3); i++) {
                    datas.add(mRandom.nextInt(mAdapter.getItemCount() - 1),
                            new Student(mRandom.nextInt(3000) + "", "Students: " + mRandom.nextDouble()));
                }
                mAdapter.notifyDiff();
            }
        });
```

#总结

最新 Support 包中的 DiffUtil 类给我们带来了一个对 RecyclerView 的不同数据变化的统一处理方案，可以对所有数据变化之后的通知刷新简化，非常好用，强烈推荐使用。

#参考

Android开发学习之路-DiffUtil使用教程

[Android 7.0带来的新工具类：DiffUtil](http://www.jianshu.com/p/6f9a52365b3e)

[RecyclerView:使用DiffUtil刷新错位](http://www.jianshu.com/p/4cd2cf4139bf)

[【Android】你可能不知道的Support(一) 0步自动定向刷新:SortedList](http://blog.csdn.net/zxt0601/article/details/53495709)

