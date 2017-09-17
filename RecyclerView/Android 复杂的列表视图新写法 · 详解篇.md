> 原文链接：http://gank.io/post/5823bcf6421aa90e799ec2ad

# 前言

> MultiType 这个项目，至今 v3.x 稳定多时，考虑得非常多，但也做得非常克制。原则一直是 直观、灵活、可靠、简单纯粹（其中直观和灵活是非常看重的）。

在开发我的 **[TimeMachine](https://github.com/drakeet/TimeMachine)** 时，我有一个复杂的聊天页面，于是我设计了我的类型池系统，它是完全解耦的，我能够轻松将它抽离出来分享，并给它取名为 **MultiType**.

从前，**比如我们写一个类似微博列表页面**，这样的列表是十分复杂的：有纯文本的、带转发原文的、带图片的、带视频的、带文章的等等，甚至穿插一条可以横向滑动的好友推荐条目。不同的 item 类型众多，而且随着业务发展，还会更多。如果我们使用传统的开发方式，经常要做一些繁琐的工作，代码可能都堆积在一个 `Adapter` 中：我们需要覆写 `RecyclerView.Adapter` 的 `getItemViewType` 方法，罗列一些 `type` 整型常量，并且 `ViewHolder` 转型、绑定数据也比较麻烦。一旦产品需求有变，或者产品设计说需要增加一种新的 item 类型，我们需要去代码堆里找到原来的逻辑去修改，或找到正确的位置去增加代码。这些过程都比较繁琐，侵入较强，需要小心翼翼，以免改错影响到其他地方。

现在好了，我们有了 **MultiType**，简单来说，**MultiType 就是一个多类型列表视图的中间分发框架，它能帮助你快速并且清晰地开发一些复杂的列表页面，数据驱动视图。** 它本是为聊天页面开发的，聊天页面的消息类型也是有大量不同种类，且新增频繁，而 **MultiType** 能够轻松胜任。

**MultiType** 以灵活直观为第一宗旨进行设计，它内建了 `类型` - `View` 的复用池系统，支持 `RecyclerView`，随时可拓展新的类型进入列表当中，使用简单，令代码清晰、模块化、灵活可变。

因此，我写了这篇文章，目的有几个：一是以作者的角度对 **MultiType** 进行入门和进阶详解。二是传递我开发过程中的思想、设计理念，这些偏细腻的内容，即使不使用 **MultiType**，想必也能带来很多启发。最后就是把自我觉得不错的东西分享给大家，试想如果你制造的东西很多人在用，即使没有带来任何收益，也是一件很自豪的事情。

# MultiType 的特性

- 轻盈，整个类库只有 14 个类文件，`aar` 或 `jar` 包大小只有 13 KB
- 周到，支持 data type `<-->` item view binder 之间 一对一 和 **一对多** 的关系绑定
- 灵活，几乎所有的部件(类)都可被替换、可继承定制，面向接口 / 抽象编程
- 纯粹，只负责本分工作，专注多类型的列表视图 类型分发，绝不会去影响 views 的内容或行为
- 高效，没有性能损失，内存友好，最大限度发挥 `RecyclerView` 的复用性
- 可读，代码清晰干净、设计精巧，极力避免复杂化，可读性很好，为拓展和自行解决问题提供了基础

# 总览

**MultiType** 能轻松实现如下页面，它们将在示例篇章具体提供: 

![](http://ww4.sinaimg.cn/large/86e2ff85gw1f9mqd8lwzkj21hc0f8n4k.jpg)

MultiType 的源码关系：

[![](http://ww1.sinaimg.cn/large/86e2ff85gy1fhq79brqjzj21kw12hhdt.jpg)](http://ww1.sinaimg.cn/large/86e2ff85gy1fhq79brqjzj21kw12hhdt.jpg)

# MultiType 基础用法

可能有的新手看到以上特性介绍说什么 "一对多"、抽象编程等等，都不太懂，我想说完全不要紧，不懂可以回过头来再看，我们先从基础用法入手，其实 **MultiType** 使用起来特别简单。使用 **MultiType** 一般情况下只要 maven 引入 + 三个小步骤。之后还会介绍使用插件生成代码方式，步骤将更加简化：

### 引入

在你的 `build.gradle`:

```groovy
dependencies {
    compile 'me.drakeet.multitype:multitype:3.3.0'
}
```

> 注：**MultiType** 内部引用了 `recyclerview-v7:25.3.1`，如果你不想使用这个版本，可以使用 `exclude` 将它排除掉，再自行引入你选择的版本。示例如下：

```groovy
dependencies {
    compile('me.drakeet.multitype:multitype:3.3.0', {
       exclude group: 'com.android.support'
    })
    compile 'com.android.support:recyclerview-v7:你选择的版本'
}
```

_Note: MultiType does not support RecyclerView below version 23.0.0._


## 使用

**Step 1**. 创建一个 `class`，它将是你的数据类型或 Java bean / model. 对这个类的内容没有任何限制。示例如下：

```java
public class Category {

    @NonNull public final String text;

    public Category(@NonNull String text) {
        this.text = text;
    }
}
```

**Step 2**. 创建一个 `class` 继承 `ItemViewBinder`. 

 `ItemViewBinder` 是个抽象类，其中 `onCreateViewHolder` 方法用于生产你的 item view holder, `onBindViewHolder` 用于绑定数据到 `View`s. 一般一个 `ItemViewBinder` 类在内存中只会有一个实例对象，**MultiType** 内部将复用这个 binder 对象来生产所有相关的 item views 和绑定数据。示例：

```java
public class CategoryViewBinder extends ItemViewBinder<Category, CategoryViewBinder.ViewHolder> {

    @NonNull @Override
    protected ViewHolder onCreateViewHolder(@NonNull LayoutInflater inflater, @NonNull ViewGroup parent) {
        View root = inflater.inflate(R.layout.item_category, parent, false);
        return new ViewHolder(root);
    }

    @Override
    protected void onBindViewHolder(@NonNull ViewHolder holder, @NonNull Category category) {
        holder.category.setText(category.text);
    }

    static class ViewHolder extends RecyclerView.ViewHolder {

        @NonNull private final TextView category;

        ViewHolder(@NonNull View itemView) {
            super(itemView);
            this.category = (TextView) itemView.findViewById(R.id.category);
        }
    }
}
```

**Step 3**. 在 `Activity` 中加入 `RecyclerView` 和 `List` 并注册你的类型，示例：

```java
public class MainActivity extends AppCompatActivity {

    private MultiTypeAdapter adapter;

    /* Items 等同于 ArrayList<Object> */
    private Items items;

    @Override 
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        RecyclerView recyclerView = (RecyclerView) findViewById(R.id.list);
        /* 注意：我们已经在 XML 布局中通过 app:layoutManager="LinearLayoutManager"
         * 给这个 RecyclerView 指定了 LayoutManager，因此此处无需再设置 */

        adapter = new MultiTypeAdapter();

        /* 注册类型和 View 的对应关系 */
        adapter.register(Category.class, new CategoryViewBinder());
        adapter.register(Song.class, new SongViewBinder());
        recyclerView.setAdapter(adapter);

        /* 模拟加载数据，也可以稍后再加载，然后使用
         * adapter.notifyDataSetChanged() 刷新列表 */
        items = new Items();
        for (int i = 0; i < 20; i++) {
            items.add(new Category("Songs"));
            items.add(new Song("drakeet", R.drawable.avatar_dakeet));
            items.add(new Song("许岑", R.drawable.avatar_cen));
        }
        adapter.setItems(items);
        adapter.notifyDataSetChanged();
    }
}
```

大功告成！这就是 **MultiType** 的基础用法了。其中 `onCreateViewHolder` 和 `onBindViewHolder` 方法名沿袭了使用 `RecyclerView` 的习惯，令人一目了然，减少了新人的学习成本。


# 设计思想

**MultiType** 设计伊始，我给它定了几个原则：

- 要简单，便于他人阅读代码

  因此我极力避免将它复杂化，避免加入许多不相干的内容。我想写人人可读的代码，使用简单精巧的方式，去实现复杂的需求。过多不相干、没必要的代码，将会使项目变得令人晕头转向，难以阅读，遇到需要定制、解决问题的时候，无从下手。

- 要灵活，便于拓展和适应各种需求

  很多人会得意地告诉我，他们把 **MultiType** 源码精简成三四个类，甚至一个类，以为代码越少就是越好，这我不能赞同。**MultiType** 考虑得更远，这是一个提供给大众使用的类库，过度的精简只会使得大幅失去灵活性。**它或许不是使用起来最简单的，但很可能是使用起来最灵活的，均衡性最好的。** 在我看来，"直观"、"灵活"优先级大于"简单"。因此，**MultiType** 以接口或抽象进行连接，这意味着它的角色、组件都可以被替换，或者被拓展和继承。如果你觉得它使用起来还不够简单，完全可以通过继承封装出更具体符合你使用需求的方法。它已经暴露了足够丰富、周到的接口以供拓展，我们不应该直接去修改源码，这会导致一旦后续发现你的精简版满足不了你的需求时，已经没有回头路了。

- 要直观，使用起来能令项目代码更清晰可读，一目了然

  **MultiType** 提供的 `ItemViewBinder` 沿袭了 `RecyclerView Adapter` 的接口命名，使用起来更加舒适，符合习惯。另外，MultiType 很多地方放弃使用反射而是让用户**显式**指明一些关系，如：`MultiTypeAdapter#register` 方法，需要传递一个数据模型 `class` 和 `ItemViewBinder` 对象，虽然有很多方法可以把它精简成单一参数方法，但我们认为显式声明数据模型类与对应关系，更加直观。


# 高级用法

介绍了基础用法和设计思想后，我们可以来介绍一下 **MultiType** 的高级用法。这是一些典型需求和案例，它们是基础用法的延伸，也是设计思想的体现。也许一开始并不会使用到，但如若了解，能够拓宽使用 **MultiType** 的思路，也能过了解到我们考虑问题的角度。

## 使用 MultiTypeTemplates 插件自动生成代码

在基础用法中，我们了通过 3 个步骤完成 **MultiType** 的初次接入使用，实际上这个过程可以更加简化，**MultiType** 提供了 Android Studio 插件来自动生成代码：

**MultiTypeTemplates**，源码也是开源的，[https://github.com/drakeet/MultiTypeTemplates](https://github.com/drakeet/MultiTypeTemplates)。这个插件不仅提供了一键生成 item 类文件和 `ItemViewBinder`，而且**是一个很好的利用代码模版自动生成代码的示例。**其中使用到了官方提供的代码模版 API，也用到了我自己发明的更灵活修改模版内容的方法，有兴趣做这方面插件的可以看看。

话说回来，安装和使用 **MultiTypeTemplates** 非常简单：

**Step 1.** 打开 Android Studio 的`设置` -> `Plugin` -> `Browse repositories`，搜索 `MultiTypeTemplates` 即可获得下载安装：

<img src="http://ww4.sinaimg.cn/large/86e2ff85gw1f935l0kwilj21kw0t3akm.jpg" width=640/>


**Step 2.** 安装完成后，重启 Android Studio. 右键点击你的 package，选择 `New` -> `MultiType Item`，然后输入你的 item 名字，它就会自动生成 item 模型类 和 `ItemViewBinder` 文件和代码。

比如你输入的是 "Category"，它就会自动生成 `Category.java` 和 `CategoryViewBinder.java`.

特别方便，相信你会很喜欢它。未来这个插件也将会支持自动生成布局文件，这是目前欠缺的，但不要紧，其实 AS 在这方面已经很方便了，对布局 `R.layout.item_category` 使用 `alt + enter` 快捷键即可自动生成布局文件。


## 一个类型对应多个 `ItemViewBinder`

**MultiType** 支持一个类型对应多个 `ItemViewBinder`，首创了具有良好 API 且高性能的一对多 Link 模型。使用方式也很简单直观，如下：

```java
adapter.register(Data.class).to(
    new DataType1ViewBinder(),
    new DataType2ViewBinder()
).withClassLinker(new ClassLinker<Data>() {
    @NonNull @Override
    public Class<? extends ItemViewBinder<Data, ?>> index(@NonNull Data data) {
        if (data.type == Data.TYPE_2) {
            return DataType2ViewBinder.class;
        } else {
            return DataType1ViewBinder.class;
        }
    }
});
```

或者：

```java
adapter.register(Data.class).to(
    new DataType1ViewBinder(),
    new DataType2ViewBinder()
).withLinker(new Linker<Data>() {
    @Override
    public int index(@NonNull Data data) {
        return data.type == Data.TYPE_2 ? 1 : 0;
    }
});
```
如果你使用 Lambda 表达式，以上代码可以更简洁：

<img src="https://github.com/drakeet/MultiType/raw/3.x/art/sample-one2many.png" width=640/>

解释：

如上示例代码，对于一对多，我们需要使用 `MultiType#register(class)` 方法，它会返回一个 `OneToManyFlow` 让你紧接着绑定多个 `ItemViewBinder` 实例，最后再调用 `OneToManyEndpoint#withLinker` 或 `OneToManyEndpoint#withClassLinker` 操作符方法类设置 linker. 所谓 linker，是负责动态连接这个 "一" 对应 "多" 中哪一个 binder 的角色。

这个方案具有很好的性能表现，而且可谓十分直观。另外，我使用了 `@CheckResult` 注解来让编译器督促开发者一定要完整调用方法链才不至于出错。

更详细的"一对多"示例可以参考我的 sample 源码：https://github.com/drakeet/MultiType/tree/master/sample/src/main/java/me/drakeet/multitype/sample/one2many 

## 使用 全局类型池

**MultiType** 在 3.0 版本之前一直是支持全局类型池的，你可以往一个全局类型池中 register 类型和 view binder，然后让你的各个 `MultiTypeAdapter` 都能使用它。

但在 **MultiType** 3.0 之后，我们废弃并删除了内置的全局类型池。原因在于全局类型池容易对全局产生不可见影响，比如你注册了一堆全局类型关系并在多处引用它，某一天你的伙伴不小心修改了全局类型池的某个内容，将导致所有使用的地方皆受到变化，是我们不希望发生的。一个好的模块，应该是高内聚、自包含的，如果过多下放权力到外围，很容易遭受破坏或影响。

另外，全局类型池一般都是 static 形式的，如果我们给这个 static 容器传递了 `Activity` 或 `Context` 对象，而没有在退出时释放，就容易造出内存泄漏，这对新手来说很容易触犯。

因此我们删除了内置的全局类型池，当你创建一个 `MultiTypeAdapter` 对象时，默认情况下，它内部会自动创建一个局部类型池以供你接下来注册类型。当然了，如果你实在需要它，完全可以自己创建一个 static 的 `MultiTypePool`，然后通过 `MultiTypeAdapter#registerAll(pool)` 将这个类型池传入，以此达到多个地方共同使用。


## 与 `ItemViewBinder` 通讯

`ItemViewBinder` 对象可以接受外部类型、回调函数，只要在使用之前，传递进去即可，例如：

```java
OnClickListener listener = new OnClickListener() {
    @Override
    public void onClick(View v) {
        // ...
    }
}
adapter.register(Post.class, new PostViewBinder(xxx, listener));
```

但话说回来，对于点击事件，能不依赖 `binder` 外部内容的话，最好就在 `binder` 内部完成。`binder` 内部能够拿到 Views 和 数据，大部分情况下，完全有能力不依赖外部 独立完成逻辑。这样能使代码更加模块化，实现解耦和内聚。例如下面便是一个完全自包含的例子：

```java
public class SquareViewBinder extends ItemViewBinder<Square, SquareViewBinder.ViewHolder> {

    @NonNull @Override
    protected ViewHolder onCreateViewHolder(@NonNull LayoutInflater inflater, @NonNull ViewGroup parent) {
        View root = inflater.inflate(R.layout.item_square, parent, false);
        return new ViewHolder(root);
    }

    @Override
    protected void onBindViewHolder(@NonNull ViewHolder holder, @NonNull Square square) {
        holder.square = square;
        holder.squareView.setText(valueOf(square.number));
        holder.squareView.setSelected(square.isSelected);
    }

    public class ViewHolder extends RecyclerView.ViewHolder {

        private TextView squareView;
        private Square square;

        ViewHolder(final View itemView) {
            super(itemView);
            squareView = (TextView) itemView.findViewById(R.id.square);
            itemView.setOnClickListener(new View.OnClickListener() {
                @Override 
                public void onClick(View v) {
                    itemView.setSelected(square.isSelected = !square.isSelected);
                }
            });
        }
    }
}
```

## 使用断言，比传统 Adapter 更加易于调试

**众所周知，如果一个传统的 `RecyclerView` `Adapter` 内部有异常导致崩溃，它的异常栈是不会指向到你的 `Activity`**，这给我们开发调试过程中带来了麻烦。如果我们的 `Adapter` 是复用的，就不知道是哪一个页面崩溃。而对于 `MultiTypeAdapter`，我们显然要用于多个地方，而且可能出现开发者忘记注册类型等等问题。为了便于调试，开发期快速失败，**MultiType** 提供了很方便的断言 API: `MultiTypeAsserts`，使用方式如下：

```java
import static me.drakeet.multitype.MultiTypeAsserts.assertAllRegistered;
import static me.drakeet.multitype.MultiTypeAsserts.assertHasTheSameAdapter;

public class SimpleActivity extends MenuBaseActivity {

    private Items items;
    private MultiTypeAdapter adapter;

    @Override 
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        RecyclerView recyclerView = (RecyclerView) findViewById(R.id.list);

        items = new Items();
        adapter = new MultiTypeAdapter(items);
        adapter.register(TextItem.class, new TextItemViewBinder());

        for (int i = 0; i < 20; i++) {
            items.add(new TextItem(valueOf(i)));
        }

        /* 断言所有使用的类型都已注册 */
        assertAllRegistered(adapter, items);
        recyclerView.setAdapter(adapter);
        /* 断言 recyclerView 使用的是正确的 adapter */
        assertHasTheSameAdapter(recyclerView, adapter);
    }
}
```

`assertAllRegistered` 和 `assertHasTheSameAdapter` 都是可选择性使用，`assertAllRegistered` 需要在加载或更新数据之后， `assertHasTheSameAdapter` 必须在 `recyclerView.setAdapter(adapter)` 之后。

这样做以后，`MultiTypeAdapter` 相关的异常都会报到你的 `Activity`，并且会详细注明出错的原因，而如果符合断言，断言代码不会有任何副作用或影响你的代码逻辑，这时你可以把它当作废话。关于这个类的源代码是很简单的，有兴趣可以直接看看源码：[drakeet/multitype/MultiTypeAsserts.java](https://github.com/drakeet/MultiType/blob/master/library/src/main/java/me/drakeet/multitype/MultiTypeAsserts.java)


## 支持 Google AutoValue

[AutoValue](https://github.com/google/auto/tree/master/value) 是 Google 提供的一个在 Java 实体类中自动生成代码的类库，使你更专注于处理项目的其他逻辑，它可使代码更少，更干净，以及更少的 bug. 

当我们使用传统方式创建一个 Java 模型类的时候，经常需要写一堆 `toString()`、`hashCode()`、getter、setter 等等方法，而且对于 Android 开发，大多情况下还需要实现 `Parcelable` 接口。这样的结果是，我本来想要一个只有几个属性的小模型类，但出于各种原因，这个模型类方法数变得十分繁复，阅读起来很不清爽，并且难免会写错内容。AutoValue 的出现解决了这个问题，我们只需定义一些抽象类交给 AutoValue，AutoValue 会**自动**生成该抽象类的具体实现子类，并携带各种样板代码。

更详细的介绍内容和使用教程，我会在文章末尾会给出 AutoValue 的相关链接，不熟悉 AutoValue 可以借此机会看一下，在这里就不做过多介绍了。新手暂时看不懂也不必纠结，了解之后都是十分容易的。

**MultiType** 支持了 Google AutoValue，支持自动映射某个已经注册的类型的**子类**到同一 `ItemViewBinder`，规则是：如果子类**有**注册，就用注册的映射关系；如果子类**没**注册，则该子类对象使用注册过的父类映射关系。

## FlatTypeAdapter(已废弃)

MultiType 3.0 之前提供了一个 `FlatTypeAdapter` 类，3.0 之后，这个类已经被删除了，你可以完全不必关心它。如果你使用过它，现在它已经被一对多方案替代了，请转成使用一对多功能实现。

## MultiType 与下拉刷新、加载更多、HeaderView、FooterView、Diff

**MultiType** 设计从始至终，都极力避免往复杂化方向发展，一开始我的设计宗旨就是它应该是一个非常纯粹的、专一的项目，而非各种乱七八糟的功能都要囊括进来的多合一大型库，因此它很克制，期间有许多人给我发过一些无关特性的 Pull Request，表示感谢，但全被拒绝了。

对于很多人关心的 下拉刷新、加载更多、HeaderView、FooterView、Diff 这些功能特性，其实都不应该是 **MultiType** 的范畴，**MultiType 的分内之事是做类型、事件与 View 的分发、连接工作**，其余无关的需求，都是可以在 **MultiType** 外部完成，或者通过继承 进行自行封装和拓展，而作为一个基础、公共类库，我想它是不应该包含这些内容。

但很多新手可能并不习惯代码分工、模块化，因此在此我有必要对这几个点简单示范下如何在 **MultiType** 之外去实现：

- **下拉刷新：**

  对于下拉刷新，`Android` 官方提供了 `support.v4` `SwipeRefreshLayout`，在 `Activity` 层面，可以拿到 `SwipeRefreshLayout` 调用 `setOnRefreshListener` 设置监听器即可.

  或者参考我的 rebase-android 项目编写的 [SwipeRefreshDelegate.java](https://github.com/drakeet/rebase-android/blob/master/app/src/main/java/com/drakeet/rebase/tool/SwipeRefreshDelegate.java).

- **加载更多：**

  `RecyclerView` 提供了 `addOnScrollListener` 滚动位置变化监听，要实现加载更多，只要监听并检测列表是否滚动到底部即可，有多种方式，鉴于 `LayoutManager` 本应该只做布局相关的事务，因此我们推荐直接在 `OnScrollListener` 层面进行判断。提供一个简单版 `OnScrollListener` 继承类：

  ```java
  public abstract class OnLoadMoreListener extends RecyclerView.OnScrollListener {

      private LinearLayoutManager layoutManager;
      private int itemCount, lastPosition, lastItemCount;

      public abstract void onLoadMore();

      @Override
      public void onScrolled(RecyclerView recyclerView, int dx, int dy) {
          if (recyclerView.getLayoutManager() instanceof LinearLayoutManager) {
              layoutManager = (LinearLayoutManager) recyclerView.getLayoutManager();

              itemCount = layoutManager.getItemCount();
              lastPosition = layoutManager.findLastCompletelyVisibleItemPosition();
          } else {
              Log.e("OnLoadMoreListener", "The OnLoadMoreListener only support LinearLayoutManager");
              return;
          }

          if (lastItemCount != itemCount && lastPosition == itemCount - 1) {
              lastItemCount = itemCount;
              this.onLoadMore();
          }
      }
  }
  ```

  或者参考我的 rebase-android 项目编写的 [LoadMoreDelegate.java](https://github.com/drakeet/rebase-android/blob/master/app/src/main/java/com/drakeet/rebase/tool/LoadMoreDelegate.java).

- **获取数据后做 Diff 更新：**

  **MultiType** 支持 onBindViewHolder with `payloads`，详情见 `ItemViewBinder` 类文档。对于 Diff，可以在 `Activity` 中进行 Diff，或者继承 `MultiTypeAdapter` 提供接收数据方法，在方法中进行 Diff. **MultiType** 不提供内置 Diff 方案，不然需要依赖 v4 包，并且这也不应该属于它的范畴。

  示例代码：https://github.com/drakeet/MultiType/issues/56

- **HeaderView、FooterView**

  **MultiType** 其实本身就支持 `HeaderView`、`FooterView`，只要创建一个 `Header.class` - `HeaderViewBinder` 和 `Footer.class` - `FooterViewBinder` 即可，然后把 `new Header()` 添加到 `items` 第一个位置，把 `new Footer()` 添加到 `items` 最后一个位置。需要注意的是，如果使用了 Footer View，在底部插入数据的时候，需要添加到 `最后位置 - 1`，即倒二个位置，或者把 `Footer` remove 掉，再添加数据，最后再插入一个新的 `Footer`.

## 实现 RecyclerView 嵌套横向 RecyclerView

**MultiType** 天生就适合实现类似 Google Play 或 iOS App Store 那样复杂的首页列表，这种页面通常会在垂直列表中嵌套横向列表，其实横向列表我们完全可以把它视为一种 `Item` 类型，这个 item 持有一个列表数据和当前横向列表滑动到的位置，类似这样：

```java
public class PostList {

    public final List<Post> posts;
    public int currentPosition;

    public PostList(@NonNull List<Post> posts) { this.posts = posts; }
}
```

对应的 `HorizontalItemViewBinder` 类似这样：

```java
public class HorizontalItemViewBinder extends ItemViewBinder<PostList, HorizontalItemViewBinder.ViewHolder> {

    @NonNull @Override
    protected ViewHolder onCreateViewHolder(@NonNull LayoutInflater inflater, @NonNull ViewGroup parent) {
        /* item_horizontal_list 就是一个只有 RecyclerView 的布局 */
        View view = inflater.inflate(R.layout.item_horizontal_list, parent, false);
        return new ViewHolder(view);
    }

    @Override
    protected void onBindViewHolder(@NonNull ViewHolder holder, @NonNull PostList postList) {
        holder.setPosts(postList.posts);
    }

    static class ViewHolder extends RecyclerView.ViewHolder {

        private RecyclerView recyclerView;
        private PostsAdapter adapter;

        private ViewHolder(@NonNull View itemView) {
            super(itemView);
            recyclerView = (RecyclerView) itemView.findViewById(R.id.post_list);
            LinearLayoutManager layoutManager = new LinearLayoutManager(itemView.getContext());
            layoutManager.setOrientation(LinearLayoutManager.HORIZONTAL);
            recyclerView.setLayoutManager(layoutManager);
            /* adapter 只负责灌输、适配数据，布局交给 LayoutManager，可复用 */
            adapter = new PostsAdapter();    // 或者直接使用 MultiTypeAdapter 更加方便
            recyclerView.setAdapter(adapter);
            /* 在此设置横向滑动监听器，用于记录和恢复当前滑动到的位置，略 */
            ...
        }

        private void setPosts(List<Post> posts) {
            adapter.setPosts(posts);
            adapter.notifyDataSetChanged();
        }
    }
}
```

## 实现线性布局和网格布局混排列表

这个课题其实也不属于 **MultiType** 的范畴，**MultiType** 的职责是做数据类型分发，而不是布局，但鉴于很多复杂页面都会需要线性布局和网格布局混排，我就简单讲一讲，关键在于 `RecyclerView` 的 `LayoutManager`. 虽然是线性和网格混合，但实现起来其实只要一个网格布局 `GridLayoutManager`，如果你查看 `GridLayoutManager` 的官方源码，你会发现它其实继承自 `LinearLayoutManager`. 以下是示例和解释：

```java
public class MultiGridActivity extends MenuBaseActivity {

    private final static int SPAN_COUNT = 5;
    private MultiTypeAdapter adapter;
    private Items items;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_multi_grid);
        items = new Items();
        RecyclerView recyclerView = (RecyclerView) findViewById(R.id.list);
        
        final GridLayoutManager layoutManager = new GridLayoutManager(this, SPAN_COUNT);
        
        /* 关键内容：通过 setSpanSizeLookup 来告诉布局，你的 item 占几个横向单位，
         * 如果你横向有 5 个单位，而你返回当前 item 占用 5 个单位，那么它就会看起来单独占用一行 */
        layoutManager.setSpanSizeLookup(new GridLayoutManager.SpanSizeLookup() {
            @Override
            public int getSpanSize(int position) {
                return (items.get(position) instanceof Category) ? SPAN_COUNT : 1;
            }
        });
        recyclerView.setLayoutManager(layoutManager);
        
        adapter = new MultiTypeAdapter(items);
        adapter.applyGlobalMultiTypePool();
        adapter.register(Square.class, new SquareViewBinder());

        assertAllRegistered(adapter, items);
        recyclerView.setAdapter(adapter);
        loadData();
    }

    private void loadData() {
        // ...
    }
}
```

## 数据扁平化处理

在一个**垂直** `RecyclerView` 中，item 们都是同级的，没有任何嵌套关系，但我们的数据结构往往存在嵌套关系，比如 `Post` 内部包含了 `Comment`s 数据，或换句话说 `Post` 嵌套了 `Comment`，就像微信朋友圈一样，"动态" 伴随着 "评论"。那么如何把 非扁平化 的数据排布在 扁平 的列表中呢？必然需要一个_数据扁平化处理_的过程，就像 `ListView` 的数据需要一个 `Adapter` 来适配，`Adapter` 就像一个油漏斗，把油引入瓶子中。我们在面对嵌套数据结构的时候，可以采用如下的扁平化处理，关于扁平化这个词，不必太纠结，简单说，就是把嵌套数据都拉出来，摊平，让 `Comment` 和 `Post` 同级，最后把它们都 add 进同一个 `Items` 容器，交给 `MultiTypeAdapter`. 示例：

假设：你的 `Post` 是这样的：

```java
public class Post {

    public String content;
    public List<Comment> comments; 
}
```

假设：你的 `Comment` 是这样的：

```java
public class Comment {

    public String content;
}
```

假设：你服务端返回的 JSON 数据是这样的：

```json
[
    {
        "content":"I have released the MultiType v2.2.2", 
        "comments":[
            {"content":"great"},
            {"content":"I love your post!"}
        ]
    }
]
```

那么你的 JSON 转成 Java Bean 之后，你拿到手应该是个 `List<Post> posts` 对象，现在我们写一个扁平化处理的方法：

```java
private List<Object> flattenData(List<Post> posts) {
    final List<Object> items = new ArrayList<>();
    for (Post post : posts) {
        /* 将 post 加进 items，Binder 内部拿到它的时候，
         * 我们无视它的 comments 内容即可 */
        items.add(post);
        /* 紧接着将 comments 拿出来插入进 items，
         * 评论就能正好处于该条 post 下面 */
        items.addAll(post.comments);
    }
    return items;
}
```

最后我们所有的 `posts` 在加入全局 MultiType `Items` 之前，都需要经过扁平化处理：

```java
items.addAll(flattenData(posts));
adapter.notifyDataSetChanged();
```

整个过程其实并不困难，相信大家都已经理解了。

# 更多示例

**MultiType** 的开源项目提供了许多的 samples (示例) 程序，这些示例秉承了一贯的代码清晰、干净的风格，十分易于阅读：

- [仿造**微博**的数据结构和二级 ItemViewBinder](https://github.com/drakeet/MultiType/tree/master/sample/src/main/java/me/drakeet/multitype/sample/weibo)

  这是一个类似微博数据结构的示例，数据两层结构，Item 也是两层结构：一层框架（包含头像用户名等），一层 content view(微博内容)，内容嵌套于框架中。微博的每一条微博 item 都包含了这样两层嵌套关系，这样做的好处是，你不必每个 item 都去重复制造一遍外层框架。

  或者换一个比喻，就像聊天消息，一条聊天消息也是两层的，一层头像、用户名、聊天气泡框，一层你的文字、图片等。另外，每一种消息都有左边和右边的样式，分别对应别人发来的消息和你发出的消息。如果左边算一种，右边又算一种，就是比较不好的设计了，会导致布局内容重复、冗余，修改操作都要做两遍。最好的方案是让他们视被为同一种类型，然后在 item 框层次进行左右边判断和框架相关数据绑定。

  我提供的这个二级 `ItemViewBinder` 示例便是这样的两层结构。它能够让你每次新增加一个类型，只要实现内容即可，框不应该重复实现。

  如果再不明白，或许你可以看看我的这个示例中 微博 Item 框的布局：

  ![](http://ww2.sinaimg.cn/large/86e2ff85gw1f9brj5gw1jj21e211ane6.jpg)

  从我这个 `frame` 布局可以看出来，它内部有一个 `FrameLayout` 作为 `container` 将用于容纳不同的微博内容，而这一层框架则是共同的。

  这个例子算高级中的高级，但实际上也是很简单，展示了 **MultiType** 优秀的可拓展能力。完整运行结果展示如下：

  <img src="http://ww3.sinaimg.cn/large/86e2ff85jw1f9a7tek74lj21401z414s.jpg" width=216/> <img src="http://ww1.sinaimg.cn/mw1024/86e2ff85jw1f9a7z4yqlkj21401z4n8r.jpg" width=216/>

  > 注：以上我们并没有提到服务端 JSON 数据转为我们定义的 Weibo 对象过程，实际上对于完整链路，这个过程是需要做数据转换，我们需要在 `WeiboContent` 层加一个 `type` 或 `describe` 字段用于描述微博内容类型，然后再将微博内容的 JSON 文本转为具体微博内容对象交给 Weibo. 这个内容建议直接阅读这个 sample 的 `WeiboContentDeserializer` 源码，我利用了一种很简单又巧妙的方式，在 JSON 解析底层便进行抽象数据具体化，使得客户端和服务端都能够轻松适应这种微博和微博内容嵌套关系。

- [drakeet/about-page](https://github.com/drakeet/about-page)

  一个 Material Design 的关于页面，核心基于 MultiType，包含了多种 items，美观，容易使用。

  ![](http://ww2.sinaimg.cn/large/86e2ff85gw1f93gq2tevbj21700pcjyp.jpg)

- [线性和网格布局混排](https://github.com/drakeet/MultiType/tree/master/sample/src/main/java/me/drakeet/multitype/sample/grid)

  使用 `MultiType` 和 `GridLayoutManager` 实现网格和线性混合布局，实现一个选集页面。

  <img src="http://ww1.sinaimg.cn/large/86e2ff85gw1f96yfddvksj21401z479t.jpg" width=216/>

- [drakeet/TimeMachine](https://github.com/drakeet/TimeMachine)

  TimeMachine 使用了 **MultiType** 来创建一个复杂的聊天页面，页面和需求虽然复杂，但使用 **MultiType** 显得轻松简单。

  <img src="http://ww1.sinaimg.cn/large/86e2ff85gw1f96yg01b5nj20k00zkgng.jpg" width=216/><img src="http://ww3.sinaimg.cn/large/86e2ff85gw1f94i6ysea2j20k00zkmzn.jpg" width=216/>

- [类似 Bilibili iOS 端首页](https://github.com/drakeet/MultiType/tree/master/sample/src/main/java/me/drakeet/multitype/sample/bilibili)

  使用 `MultiType` 实现类似 Bilibili iOS 端首页复杂的多类型列表视图，包括嵌套横向 `RecyclerView`.

  <img src="http://ww3.sinaimg.cn/large/86e2ff85gw1f96ygmroy0j21401z4nl9.jpg" width=216/>

  ​
# Q & A

- **Q: 觉得 MultiType 不够精简，应该怎么做？**

  A: 在前面 "设计思想" 中我们谈到：_MultiType 或许不是使用起来最简单的，但很可能是使用起来最灵活的。_其中的缘由是它高度可定制、可拓展，而不是把一些路封死。作为一个基础类库，简单和灵活需要一个均衡点，过度精简便要以失去灵活性为代价。如果觉得 **MultiType** 不够精简，想将它修改得更加容易使用，我推荐的方式是去继承 `MultiTypeAdapter` 或 `ItemViewBinder`，甚至你可以重新实现一个 `TypePool` 再设置给 `MultiTypeAdapter`. 我们不应该直接到底层去修改、破坏它们。总之，利用开放接口或继承的做法不管对于 **MultiType** 还是其它开源库，都应该是定制的首选。

- **Q: 在 `ItemViewBinder` 中如何拿到 `Context` 对象？**

  A: 有人问我说，他在 `ItemViewBinder` 里使用 [Glide](https://github.com/bumptech/glide) 来加载图片需要获取到 Activity `Context` 对象，要怎么才能拿到 `Context` 对象？这是一个特别简单的问题，但我想既然有人问，应该比较典型，我就详细解答下：首先，在 Android 开发中，任何 `View` 对象都能通过 `view.getContext()` 拿到 `Context` 对象，如果你需要通过 View 拿到 Activity 对象。

  总而言之，拿到 `Context` 对象非常简单，只要你能拿到一个 `View` 对象，调用 `view.getContext()` 即可。另外，也可以参考 _[与 binder 通讯](#与-viewbinder-通讯)_ 章节，我们可以很方便地给 `binder` 传递任何对象进去，包括 `Context` 对象。

- **Q：如何在 `ItemViewBinder` 中获取到 item position？**

  A: 从 v2.3.5 版本开始，只需要在你的 `ItemViewBinder` 子类里调用 `getPosition(holder)` 方法即可。另外，`ItemViewBinder` 还提供了 `getAdapter()` 或许也是很多人想要的，比如调用 adapter 进行 notify 刷新视图等。

# 感谢

在 **MultiType** 开发维护过程中，很多朋友给了我很多反馈，我也非常乐意于与大家交流，有问必答，因为这是一个难得不错的项目，它比较接近我心中对于一个完美项目的要求：设计精巧，代码干净漂亮。

我向来是不太在意项目的 star 数目的，但热衷于把我的好东西分享给更多人使用，因此在[我的 GitHub](https://github.com/drakeet) 首页我不会把我一些高 star 项目摆出来，而是放一些我觉得代码相对比较好的项目。这是我的动力，我想写一份完美的代码，就像王垠的 40 行一样，达到自觉得天衣无缝、犹如天神衣袖般的优雅，嗯，要是哪天我做到了，我就停止开源，哈哈。

话说回来，这个项目，特别感谢大家的帮忙、反馈，感谢一些朋友的 PR、贡献和推荐，是你们让我觉得开源是一件除了完善自我之外 还充满了意义的一件事情 -- 能够与更多人协同，能够面向更宽广的世界，谢谢大家！以下是感谢名单：

[70kg](https://github.com/70kg)、[zubinxiong](https://github.com/zubinxiong)、[WanLiLi](https://github.com/WanLiLi)、[代码家](https://github.com/daimajia)、[CaMnter](https://github.com/CaMnter)、[android-xiaowei](https://github.com/android-xiaowei)、[burgessjp](https://github.com/burgessjp)、[lixi0912](https://github.com/lixi0912)、[simidaxu](https://github.com/simidaxu)、[咕咚](https://github.com/maoruibin)、[LuckyJayce](https://github.com/LuckyJayce)、[BelongsH](https://github.com/BelongsH)、[tmexcept](https://github.com/tmexcept)、[TellH](https://github.com/TellH)、[Ray Pan](https://github.com/Panl)、[Zack](https://github.com/DearZack)、[Chris](https://github.com/ChrisZou)

# 引用文献

- 《Android 内存泄漏案例和解析》https://drakeet.me/android-leaks
- 《Android 复杂的多类型列表视图新写法：MultiType》https://drakeet.me/multitype
- 《使用 Google AutoValue 自动生成代码》http://tedyin.me/2016/04/11/auto-value/
- 我的个人博客 http://drakeet.me
- **MultiType** 源代码 https://github.com/drakeet/MultiType