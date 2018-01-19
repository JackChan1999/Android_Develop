> 英文链接：http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2016/1125/6806.html
> 译文原文：http://niorgai.github.io/2016/11/17/Writing-Better-Adapters/ －Jianqiu's blog

这是一篇关于如何更好的编写 RecyclerView 的 Adapter 文章， 原文链接为 [Writing-Better-Adapters](https://medium.com/@dpreussler/writing-better-adapters-1b09758407d2#.oz9a26kb2) 。

原文中的示例代码是用 [Kotlin](https://kotlinlang.org/) 编写的， 这里我会变成 Java 版本， 同时也会结合自己的理解做些改变。所以建议大家还是看一遍原文。

对于 Android 开发者来说， 实现 Adapter 是最频繁的工作之一。Adapter 是所有列表的基本， 而列表也是很多 App 的基本组成。

编写一个列表控件的方法大多数时间都是一样的: 用一个绑定了 Adapter 的 View 来展示数据。然而一直这样会让我们对自己编写的代码变得盲目， 尽管那是辣鸡代码。或者说， 我们一直在重复创造辣鸡代码。

让我们深入地看看 Adapter 的代码。

# RecyclerView 的基础

RecyclerView(同样适用于 ListView) 的基础操作:

- 创建 View 和储存 View 信息的 ViewHolder
- 根据 ViewHolder 储存的信息来绑定 ViewHolder ， 大部分是 List 中的 model

这些步骤都比较简单， 一般不会出错。

# 包含多种类型的 RecyclerView

当 View 需要展示多重不同的类型时， 事情就变得麻烦了。比如下面这个例子。

```java
interface Animal {}
class Mouse implements Animal {}
class Duck implements Animal {}
class Dog implements Animal {}
class Car
```

这个例子中，我们需要处理不同类型的动物， 而另一个对象”车”又与动物无关。这意味着你需要创建不同的 ViewHolder 并对每一个 ViewHolder 初始化不同的布局。API 对每个类型定义了不同的 int 对象， 这就是辣鸡代码的开始。

让我们看看一些常见的代码， 当你需要多个类型时， 需要重写这个方法:

```java
@Override
public int getItemViewType(int position) {
	return 0;
}
```

默认实现返回0， 但多种类型时需要根据类型返回不同的值。

下一步，创建 ViewHolder， 即实现这个方法:

```java
@Override
public ItemViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {

}
```

根据上一步 getItemViewType() 返回的不同 viewType 值来创建不同的 ViewHolder， 方法可能是 switch 语句或 if-else语句， 不过这个关系不大。

同样的， 在绑定这个创建了的(或者回收了的) ViewHolder 时， 也需要处理不同类型。

```java
@Override
public void onBindViewHolder(ItemViewHolder holder, int position) {

}
```

因为这里没有 type 参数， 所以会根据 ViewHolder 的 Instance-of 判断不同类型， 或者也可以在这些 ViewHolder 的基类中处理 onBind 。

# 不好的地方

所以上面这种实现有什么问题呢? 看起来不是很直观吗?

让我们再看一下 getItemViewType() 方法: 系统需要知道每个位置的类型， 所以你需要将你的数据列表中的每一项都转化成视图类型。那就可能产生这种代码:

```java
if (things.get(position) == Duck) {
	return TYPE_DUCK;
} else if (things.get(position) == Mouse) {
	return TYPE_MOUSE;
}
```

你现在觉得这个代码糟糕吗? 如果不觉得， 那我们接下来看看 onBindViewHolder() 的实现。

```java
@Override
public void onBindViewHolder(ItemViewHolder holder, int position) {
	Thing thing = things.get(position);
	if (thing == Animal) {
		((AnimalViewHolder) thing).bind((Animal) thing);
	} else if (thing == Car) {
		((CarViewHolder) thing).bind((Car) thing);
	}
}
```

这段代码看起来就很乱了， Instance-of 的检查和强制类型转化使这段代码非常违背设计模式。

许多年前我的显示器上就贴着一段引用自 Scott Meyers 所写的 [Effective C++](https://books.google.de/books/about/Effective_C++.html?id=eQq9AQAAQBAJ&source=kp_cover&redir_esc=y) 中的一段话:

> Anytime you find yourself writing code of the form “if the object is of type T1, then do something, but if it’s of type T2, then do something else,” slap yourself.
>
> 每次你发现自己写的代码是像”如果这个对象是 T1 类型， 就做这个， 或者如果这个对象是 T2 类型， 就做那个”的样式的话， 就扇自己一巴掌

所以当然回头看 Adapter 的实现时， 就需要扇自己很多下。

- 我们有很多类型检查和强制类型转化。
- 这是明显的适合使用面向对象却没有使用的代码。
- 实现方式违背了 [SOLID](https://en.wikipedia.org/wiki/SOLID_%28object-oriented_design%29) 原则中的 [开闭原则](https://en.wikipedia.org/wiki/Open/closed_principle) ， 即拓展时不需要修改内部实现.

# 尝试解决

一种替代方式就是在过程中间加上转化步骤， 比如一种很简单的方式就是把所有类的类型都放在一个 map 里面， 通过一次调用获取即可。

```java
@Override
public int getItemViewType(int position) {
    return types.get(thing.class);
}
```

这样做会好点吗?

不幸的是， 这不完全解决问题， 这种方式只是隐藏了 Instance-of 的检查。因为接下来实现 onBindViewHolder() 时还是会产生 如果这个对象是 T1 类型， 就做这个， 或者如果这个对象是 T2 类型， 就做那个 这样的代码。

我们的目标应该是 添加新的类型时不需要修改 Adapter 的代码 。

因此， 一开始不需要在 Adapter 中创建视图和数据的对应关系。另外 Google 也推荐使用布局 id 来区分不同类型， 这样你就不用自己定义每种类型了。

从另一个角度想， 如果我们想创建视图和数据的对应关系， 不一定非要从数据集入手， 可以对每个数据添加一个方法:

```java
public int getType() {
	return R.layout.item_duck;
}
```

那 Adapter 中获取类型的方法就可以变成

```java
@Override
public int getItemViewType(int position) {
    return things[position].getType();
}
```

这样就符合了开闭原则， 当我们想要添加新类型时不需要再改变 Adapter 中的代码。

但这种做法把架构中的每个层打乱了。实体类型知道它的表现形式， 这种关系指向是错误的。对我们来说不可接受。另一方面， 在数据中添加方法来表示它的类型， 这种做法并不是面向对象的， 我们只是再一次隐藏了 Instance-of 的检查。

# ViewModel 视图模型

更进一步的处理方法， 便是使用独立的视图模型而不是直接使用模型。归根结底， 我们的数据是不想交的， 他们没有一个相同的基类: 车不是动物。这对数据层来说是对的， 但对表现层来说， 他们都是展现在同一个视图中， 所以它们可以有一个共同的基类。

```java
abstract class ViewModel {
	abstract int type();
}
 
class DuckViewModel extends ViewModel {

    @Override
    int type() {
        return R.layout.duck;
    }
}

class CarViewModel extends ViewModel {

    @Override
    int type() {
        return R.layout.car;
    }
}
```

这样就简单封装了数据。当添加新的视图类型时， 不需要改动 Adapter 和原来的视图类型的代码。比如 RecyclerView 的其他类型: 分割线， 段落头部， 广告等。

这只是一个比较接近的解决方法， 但并不唯一。

# The Visitor 访问者模式

如果你有很多模型类 (Model class) ， 可能你不会向对每一个再创建对应的视图模型类 (ViewModel class)。那我们再来看看如何只使用数据模型 (model) 。

一开始的时候， 当我们把 type() 方法加到每个模型中， 这种方法就太耦合了。我们应该把方法抽象出来， 比如添加一个接口:

```java
interface Visitable {
  int type(TypeFactory typeFactory);
}
```

那么每个模型就会变成这样:

```java
interface Animal extends Visitable {
 
}
 
class Mouse implements Visitable {
  
  @Override
  int type(TypeFactory typeFactory) {
    return typeFactory.type(this);
  }
}
 
class Car implements Visitable {
  
  @Override
  int type(TypeFactory typeFactory) {
    return typeFactory.type(this);
  }
}
```

而工厂类也应该是个抽象类， 它包含了所需的类型。

```java
interface TypeFactory {
  int type(Duck duck);
  int type(Mouse mouse);
  int type(Dog dog);
  int type(Car car);
}
```

这样就完全是类型安全的实现， 不需要 Instance-of 的判断， 也不需要强制类型转化。对于每个具体工厂的实现也是清晰的， 实现对应的类型即可。

```java
class TypeFactoryForList extends TypeFactory {
 
    @Override
    public int type(Duck duck) {
        return R.layout.duck;
    }

    @Override
    public int type(Mouse mouse) {
        return R.layout.mouse;
    }

    @Override
    public int type(Dog dog) {
        return R.layout.dog;
    }

    @Override
    public int type(Car car) {
        return R.layout.car;
    }
}
```

当我们还想添加新的类型时， 这是我们需要添加代码的地方。这就非常符合 SOLID 原则。你可能需要别的方法给新的类型， 但不需要修改任何已经存在的方法: 对拓展打开， 对修改关闭。

现在你可能会问: 为什么不直接在 Adapter 使用工厂类， 而是使用抽象工厂呢?

因为只有这样才能保证类型安全， 避免类型转化或类型检查。花点时间认真想想这里， 这里没有用到任何一个转化。这就是访问者模式带来的间接性。

根据以上这些步骤能保证 Adapter 非常通用。

# 结论

- 尝试保持让你的实现代码简洁
- Instance-of 检查应该是一个红色的警告标志
- 注意向下转化， 这是不好的代码的味道
- 尝试让上面那个点转化为正确的面向对象的用法， 想想接口和继承
- 尝试用通用的点来阻止转化
- 使用 ViewModel 
- 注意访问者模式的使用

我很乐于去学习更多让 Adapter 简洁的方法。

其实这篇文章着重介绍的是优化的思路， 代码实现也只是给了一些小模块， 并不是完整的 Adapter 实现。

我沿着这个思路， 写了一个 Java 版本的 Demo :

[BetterAdapter](https://github.com/niorgai/BetterAdapter)。欢迎各位指导并提出宝贵的修改意见(我也认为这个实现还是有改进空间)。

在实现的过程中， 发现了一些需要注意的地方。

- 由于每个 Adapter 中可能 model 和 type 的对应不同， 比如同一个 model M ， 在 Adapter a1 中对应 type1， 在 Adapter a2 中对应 type2， 所以 onCreateViewHolder 的实现也需要放在 TypeFactory 中解耦。

- 为了解耦(同时也是因为想不到好一点的写法)， 我抽出了一个 BetterHolder 作为 ViewHolder 的基类。然后为每个类型创建对应的 holder 来实现不同的 onCreateViewHolder 和 onBindViewHolder。

```java
public abstract class BetterViewHolder extends RecyclerView.ViewHolder {
	
    public BetterViewHolder(View itemView) {
        super(itemView);
    }
 
    public abstract BetterViewHolder onCreateViewHolder(ViewGroup parent);
 
    public abstract void onBindViewHolder(BetterViewHolder holder);
}
```

- 所有的数据必须以平级的方式添加进去， 并且需要按照顺序添加。如果后台 json 数据返回的格式如下:

```
{
  key_a : A
  key_b : {
      A,
      A
  }
  key_c : C
}
```

那么在添加的时候:

```java
mAdapter.add(A);
//把 key_b 数组遍历
mAdapter.add(A);
mAdapter.add(A);
mAdapter.add(C);
```

# 一点题外话

- [drakeet](https://github.com/drakeet) 所写的 [MultiType](https://github.com/drakeet/MultiType) 也是一种很好的方法， 同时他也写了一篇博客 [Android 复杂的多类型列表视图新写法：MultiType](https://drakeet.me/multitype)来介绍这个库。只是我个人不喜欢在这些很基础的地方使用自己没有理解透的第三方库， 所以才没有用到项目中。不过我很看好这个库。
- 无论是上面提到的 [MultiType](https://github.com/drakeet/MultiType) ， 还是原文中提到的 BetterAdapter ， 最终想要的都是解耦。如果解决了 model 和 holder 之间的一一对应， 理论上来说一个 Application 只要维护一个 Adapter 就行了。当然这是我的想法， 哈哈。