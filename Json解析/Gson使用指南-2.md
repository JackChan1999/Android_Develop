**该系列其它文章**

- [你真的会用Gson吗?Gson使用指南（一）](http://www.jianshu.com/p/e740196225a4)
- [你真的会用Gson吗?Gson使用指南（二）](http://www.jianshu.com/p/c88260adaf5e)
- [你真的会用Gson吗?Gson使用指南（三）](http://www.jianshu.com/p/0e40a52c0063)
- [你真的会用Gson吗?Gson使用指南（四）](http://www.jianshu.com/p/3108f1e44155)

**注：此系列基于Gson 2.4。**

上一篇文章 [你真的会用Gson吗?Gson使用指南（一）](http://www.jianshu.com/p/e740196225a4) 我们了解了Gson的基础用法，这次我们继续深入了解Gson的使用方法。

**本次的主要内容：**

- Gson的流式反序列化
- Gson的流式序列化
- 使用GsonBuilder导出null值、格式化输出、日期时间及其它小功能

# 1. Gson的流式反序列化

**自动方式**

> Gson提供了`fromJson()`和`toJson()` 两个直接用于解析和生成的方法，前者实现反序列化，后者实现了序列化。同时每个方法都提供了重载方法，我常用的总共有5个。

这是我在上一篇文章开头说的，但我到最后也一直没有是哪5个，这次我给列出来之后，你就知道这次讲的是哪个了。

```java
Gson.toJson(Object);
Gson.fromJson(Reader,Class);
Gson.fromJson(String,Class);
Gson.fromJson(Reader,Type);
Gson.fromJson(String,Type);
```

好了，本节结束！

看第2、4行,Reader懂了吧

**手动方式**
手动的方式就是使用`stream`包下的`JsonReader`类来手动实现反序列化，和Android中使用pull解析XML是比较类似的。

```java
String json = "{\"name\":\"怪盗kidou\",\"age\":\"24\"}";
User user = new User();
JsonReader reader = new JsonReader(new StringReader(json));
reader.beginObject(); // throws IOException
while (reader.hasNext()) {
    String s = reader.nextName();
    switch (s) {
        case "name":
            user.name = reader.nextString();
            break;
        case "age":
            user.age = reader.nextInt(); //自动转换
            break;
        case "email":
            user.email = reader.nextString();
            break;
    }
}
reader.endObject(); // throws IOException
System.out.println(user.name);  // 怪盗kidou
System.out.println(user.age);   // 24
System.out.println(user.email); // ikidou@example.com
```

其实自动方式最终都是通过`JsonReader`来实现的，如果第一个参数是`String`类型，那么Gson会创建一个`StringReader`转换成流操作。

Gson流式解析

# 2. Gson的流式序列化

**自动方式**

Gson.toJson方法列表

所以啊，学会利用IDE的自动完成是多么重要这下知道了吧！
可以看出用红框选中的部分就是我们要找的东西。

提示：`PrintStream`(System.out) 、`StringBuilder`、`StringBuffer`和`*Writer`都实现了`Appendable`接口。

```java
Gson gson = new Gson();
User user = new User("怪盗kidou",24,"ikidou@example.com");
gson.toJson(user,System.out); // 写到控制台
```

**手动方式**

```java
JsonWriter writer = new JsonWriter(new OutputStreamWriter(System.out));
writer.beginObject() // throws IOException
        .name("name").value("怪盗kidou")
        .name("age").value(24)
        .name("email").nullValue() //演示null
        .endObject(); // throws IOException
writer.flush(); // throws IOException
//{"name":"怪盗kidou","age":24,"email":null}
```

提示：除了`beginObject`、`endObject`还有`beginArray`和`endArray`，两者可以相互嵌套，注意配对即可。`beginArray`后不可以调用`name`方法，同样`beginObject`后在调用`value`之前必须要调用`name`方法。

# 3. 使用GsonBuilder导出null值、格式化输出、日期时间

一般情况下`Gson`类提供的 API已经能满足大部分的使用场景，但我们需要更多更特殊、更强大的功能时，这时候就引入一个新的类 `GsonBuilder`。

`GsonBuilder`从名上也能知道是用于构建Gson实例的一个类，要想改变Gson默认的设置必须使用该类配置Gson。

**GsonBuilder用法**

```java
Gson gson = new GsonBuilder()
               //各种配置
               .create(); //生成配置好的Gson
```

Gson在默认情况下是不动导出值`null`的键的，如：

```java
public class User {
    //省略其它
    public String name;
    public int age;
    public String email;
}
```

```java
Gson gson = new Gson();
User user = new User("怪盗kidou",24);
System.out.println(gson.toJson(user)); //{"name":"怪盗kidou","age":24}
```

可以看出，`email`字段是没有在json中出现的，当我们在调试是、需要导出完整的json串时或API接中要求没有值必须用Null时，就会比较有用。

使用方法：

```java
Gson gson = new GsonBuilder()
        .serializeNulls()
        .create();
User user = new User("怪盗kidou", 24);
System.out.println(gson.toJson(user)); //{"name":"怪盗kidou","age":24,"email":null}
```

**格式化输出、日期时间及其它**：

这些都比较简单就不一一分开写了。

```java
Gson gson = new GsonBuilder()
        //序列化null
        .serializeNulls()
        // 设置日期时间格式，另有2个重载方法
        // 在序列化和反序化时均生效
        .setDateFormat("yyyy-MM-dd")
        // 禁此序列化内部类
        .disableInnerClassSerialization()
        // 生成不可执行的Json（多了 )]}' 这4个字符）
        .generateNonExecutableJson()
        // 禁止转义html标签
        .disableHtmlEscaping()
        // 格式化输出
        .setPrettyPrinting()
        .create();
```

注意：内部类(Inner Class)和嵌套类(Nested Class)的区别

这次文章就到这里，欢迎提问互动，如有不对的地方请指正。

# 4. 下篇文章内容提要

- 字段过滤的几种方法
  - 基于`@Expose`注解
  - 基于访问修饰符
  - 基于版本
  - 自定义规则
- POJO与JSON的字段映射规则
