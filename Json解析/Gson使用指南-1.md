> JSON ([官网](http://json.org/json-zh.html)) 是一种文本形式的数据交换格式，它比XML更轻量、比二进制容易阅读和编写，调式也更加方便。其重要性不言而喻。解析和生成的方式很多，Java中最常用的类库有：JSON-Java、Gson、Jackson、FastJson等。

**该系列其它文章**

- [你真的会用Gson吗?Gson使用指南（一）](http://www.jianshu.com/p/e740196225a4)
- [你真的会用Gson吗?Gson使用指南（二）](http://www.jianshu.com/p/c88260adaf5e)
- [你真的会用Gson吗?Gson使用指南（三）](http://www.jianshu.com/p/0e40a52c0063)
- [你真的会用Gson吗?Gson使用指南（四）](http://www.jianshu.com/p/3108f1e44155)

**注：此系列基于Gson 2.4。**

对Gson使用很自信的大大可以点击关闭啦。

本篇文章的主要内容：

- Gson的基本用法
- 属性重命名 `@SerializedName` 注解的使用
- Gson中使用泛型

# 1. Gson的基本用法

Gson提供了`fromJson()` 和`toJson()` 两个直接用于解析和生成的方法，前者实现反序列化，后者实现了序列化。同时每个方法都提供了重载方法，我常用的总共有5个。

**基本数据类型的解析**

```java
Gson gson = new Gson();
int i = gson.fromJson("100", int.class);              //100
double d = gson.fromJson("\"99.99\"", double.class);  //99.99
boolean b = gson.fromJson("true", boolean.class);     // true
String str = gson.fromJson("String", String.class);   // String
```

注：不知道你是否注意到了第2、3行有什么不一样没

**基本数据类型的生成**

```java
Gson gson = new Gson();
String jsonNumber = gson.toJson(100);       // 100
String jsonBoolean = gson.toJson(false);    // false
String jsonString = gson.toJson("String"); //"String"
```

**POJO类的生成与解析**

```java
public class User {
    //省略其它
    public String name;
    public int age;
    public String emailAddress;
}
```

生成JSON：

```java
Gson gson = new Gson();
User user = new User("怪盗kidou",24);
String jsonObject = gson.toJson(user); // {"name":"怪盗kidou","age":24}
```

解析JSON：

```java
Gson gson = new Gson();
String jsonString = "{\"name\":\"怪盗kidou\",\"age\":24}";
User user = gson.fromJson(jsonString, User.class);
```

# 2. 属性重命名 @SerializedName 注解的使用

从上面POJO的生成与解析可以看出json的字段和值是的名称和类型是一一对应的，但也有一定容错机制(如第一个例子第3行将字符串的99.99转成double型，你可别告诉我都是字符串啊)，但有时候也会出现一些不和谐的情况，如：
期望的json格式

```json
{"name":"怪盗kidou","age":24,"emailAddress":"ikidou@example.com"}
```

实际

```json
{"name":"怪盗kidou","age":24,"email_address":"ikidou@example.com"}
```

这对于使用PHP作为后台开发语言时很常见的情况，php和js在命名时一般采用下划线风格，而Java中一般采用的驼峰法，让后台的哥们改吧 前端和后台都不爽，但要自己使用下划线风格时我会感到不适应，怎么办?难到没有两全齐美的方法么?

我们知道Gson在序列化和反序列化时需要使用反射，说到反射就不得不想到注解,一般各类库都将注解放到`annotations`包下，打开源码在`com.google.gson`包下果然有一个`annotations`，里面有一个`SerializedName`的注解类，这应该就是我们要找的。

那么对于json中`email_address`这个属性对应POJO的属性则变成：

```java
@SerializedName("email_address")
public String emailAddress;
```

这样的话，很好的保留了前端、后台、Android/java各自的命名习惯。

你以为这样就完了么?

如果接中设计不严谨或者其它地方可以重用该类，其它字段都一样，就`emailAddress` 字段不一样，比如有下面三种情况那怎么?重新写一个?

```json
{"name":"怪盗kidou","age":24,"emailAddress":"ikidou@example.com"}
```

```json
{"name":"怪盗kidou","age":24,"email_address":"ikidou@example.com"}
```

```json
{"name":"怪盗kidou","age":24,"email":"ikidou@example.com"}
```

**为POJO字段提供备选属性名**
`SerializedName`注解提供了两个属性，上面用到了其中一个，别外还有一个属性`alternate`，接收一个String数组。
注：`alternate`需要2.4版本

```java
@SerializedName(value = "emailAddress", alternate = {"email", "email_address"})
public String emailAddress;
```

当上面的三个属性(email_address、email、emailAddress)都中出现任意一个时均可以得到正确的结果。
注：当多种情况同时出时，以最后一个出现的值为准。

```java
Gson gson = new Gson();
String json = "{\"name\":\"怪盗kidou\",\"age\":24,\"emailAddress\":\"ikidou_1@example.com\",\"email\":\"ikidou_2@example.com\",\"email_address\":\"ikidou_3@example.com\"}";
User user = gson.fromJson(json, User.class);
System.out.println(user.emailAddress); // ikidou_3@example.com
```

# 3. Gson中使用泛型

上面了解的JSON中的Number、boolean、Object和String，现在说一下Array。

例：JSON字符串数组

```jsonArray
["Android","Java","PHP"]
```

当我们要通过Gson解析这个json时，一般有两种方式：使用数组，使用List。而List对于增删都是比较方便的，所以实际使用是还是List比较多。

数组比较简单

```java
Gson gson = new Gson();
String jsonArray = "[\"Android\",\"Java\",\"PHP\"]";
String[] strings = gson.fromJson(jsonArray, String[].class);
```

但对于List将上面的代码中的 `String[].class` 直接改为 `List<String>.class` 是行不通的。对于Java来说`List<String>` 和`List<User>` 这俩个的字节码文件只一个那就是`List.class`，这是Java泛型使用时要注意的问题 **泛型擦除**。

为了解决的上面的问题，Gson为我们提供了`TypeToken`来实现对泛型的支持，所以当我们希望使用将以上的数据解析为`List<String>`时需要这样写。

```java
Gson gson = new Gson();
String jsonArray = "[\"Android\",\"Java\",\"PHP\"]";
String[] strings = gson.fromJson(jsonArray, String[].class);
List<String> stringList = gson.fromJson(jsonArray, new TypeToken<List<String>>() {}.getType());
```

注：`TypeToken`的构造方法是`protected`修饰的,所以上面才会写成`new TypeToken<List<String>>() {}.getType()` 而不是 `new TypeToken<List<String>>().getType()`

**泛型解析对接口POJO的设计影响**
泛型的引入可以减少无关的代码，如我现在所在公司接口返回的数据分为两类：

```json
{"code":"0","message":"success","data":{}}
```

```json
{"code":"0","message":"success","data":[]}
```

我们真正需要的`data`所包含的数据，而`code`只使用一次，`message`则几乎不用。如果Gson不支持泛型或不知道Gson支持泛型的同学一定会这么定义POJO。

```java
public class UserResponse {
    public int code;
    public String message;
    public User data;
}
```

当其它接口的时候又重新定义一个`XXResponse`将`data`的类型改成XX，很明显`code`，和`message`被重复定义了多次，通过泛型的话我们可以将`code`和`message`字段抽取到一个`Result`的类中，这样我们只需要编写`data`字段所对应的POJO即可，更专注于我们的业务逻辑。如：

```java
public class Result<T> {
    public int code;
    public String message;
    public T data;
}
```

那么对于`data`字段是`User`时则可以写为 `Result<User>` ,当是个列表的时候为 `Result<List<User>>`，其它同理。

PS：嫌每次 `new TypeToken<Result<XXX>` 和 `new TypeToken<Result<List<XXX>>` 太麻烦, 想进一步封装? 查看我的另一篇博客：** 《搞定Gson泛型封装》**

# 4. 结语

本文主要通过代码向各位读者讲解了Gson的基本用法，以后还会更新更多更高级的用法，如果你还不熟悉 **注解**和**泛型** 那么你要多多努力啦。

如果你有其它的想了解的内容(不限于Gson)请给我留言评论，水平有限，欢迎拍砖。

**4月6日补充**
有说看不懂Result那段怎么个简化法，下面给个两个完整的例子，User和List<User> 。

没有引入泛型之前时写法：

```java
public class UserResult {
    public int code;
    public String message;
    public User data;
}
//=========
public class UserListResult {
    public int code;
    public String message;
    public List<User> data;
}
//=========
String json = "{..........}";
Gson gson = new Gson();
UserResult userResult = gson.fromJson(json,UserResult.class);
User user = userResult.data;

UserListResult userListResult = gson.fromJson(json,UserListResult.class);
List<User> users = userListResult.data;
```

上面有两个类`UserResult`和`UserListResult`，有两个字段重复，一两个接口就算了，如果有上百个怎么办?不得累死?所以引入泛型。

```java
//不再重复定义Result类
Type userType = new TypeToken<Result<User>>(){}.getType();
Result<User> userResult = gson.fromJson(json,userType);
User user = userResult.data;

Type userListType = new TypeToken<Result<List<User>>>(){}.getType();
Result<List<User>> userListResult = gson.fromJson(json,userListType);
List<User> users = userListResult.data;
```

看出区别了么?引入了泛型之后虽然要多写一句话用于获取泛型信息，但是返回值类型很直观，也少定义了很多无关类。
