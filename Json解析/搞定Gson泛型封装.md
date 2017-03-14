在[你真的会用Gson吗?Gson使用指南（一）](http://www.jianshu.com/p/e740196225a4) 的第三节我介绍了在Gson中如何使用泛型来简化我们的类设计，但随之而来引入了一个新的问题：封装。不知道各位有没有想过这样一个问题：每次都要用 `new TypeToken<XXX>(){};` 好麻烦，有没有更好的办法?

有更好的办法么?当然有！相信也有不少人自己作了尝试，只是有人欢喜有人愁了，不过没关系，今天我们就来解决这个问题。

# 约定

1、本文涉及到的json格式

```java
// data 为 object 的情况
{"code":"0","message":"success","data":{}}
// data 为 array 的情况
{"code":"0","message":"success","data":[]}
```

2、假定第一种的对应的Java类型为 `Result<XXX>` ，第二种为 `Result<List<XXX>>`

# 1. 为何封装，如何封装

## 1.1 为何封装

- 写`new TypeToken<XXX>(){}` 麻烦，IDE格式化后还不好看
- 不同的地方每进行一次 `new TypeToken<XXX>(){}` 操作都会生成一个新的类
- 对于任意类`XXX`都只有两种情况`new TypeToken<Result<XXX>>(){}`和`new TypeToken<Result<List<XXX>>>(){}`
- 方便统一管理

## 1.2 如何封装

从上面的我们可以知道，最简单的方法就是提供两个方法分别对应`data`为Array和Object的情况并接收一个参数，即告知XXX的类型，自动将完成`new TypeToken<XXX>(){}`与`new TypeToken<Result<List<XXX>>>(){}`的过程。

**方法原型：**

```
// 处理 data 为 object 的情况
public static <T> Result<T> fromJsonObject(Reader reader, Class<T> clazz) {}
// 处理 data 为 array 的情况
public static <T> Result<List<T>> fromJsonArray(Reader reader, Class<T> clazz){}
```

# 2. 为何失败?

对于那些尝试着封装过的人可能都这么写过：

```
public static <T> Result<List<T>> fromJsonArray(Reader reader) {
    Type type = new TypeToken<Result<List<T>>>(){}.getType();
    return GSON.fromJson(reader, type);
}
```

当然上面的写法肯定是没有办法完成的，虽然代码不会报错，但运行结果肯定是不对的，因为这里的`T` 其实是一个 `TypeVariable`，他在运行时并不会变成我们想要的XXX，所以通过`TypeToken` 得到的 泛型信息只是 `"Result<List<T>>"`。

# 3. 如何解决?

既然TypeToken的作用是用于获取泛型的类，返回的类型为`Type`，真正的泛型信息就是放在这个`Type`里面，既然用TypeToken生成会有问题,那我们自己生成Type就行了嘛。

Type是Java中所有类型的父接口，在1.8以前是一个空接口，自1.8起多了个`getTypeName()`方法，下面有`ParameterizedType`、 `GenericArrayType`、 `WildcardType`、 `TypeVariable` 几个接口，以及`Class`类。这几个接口在本次封装过程中只会用到 `ParameterizedType` ，所以简单说一下：

`ParameterizedType` 简单说来就是形如“ **类型<>** ”的类型，如:`Map<String,User>`。下面就以 `Map<String,User>` 为例讲一下里面各个方法的作用。

```
public interface ParameterizedType extends Type {
     // 返回Map<String,User>里的String和User，所以这里返回[String.class,User.clas]
    Type[] getActualTypeArguments(); 
    // Map<String,User>里的Map,所以返回值是Map.class
    Type getRawType();
    // 用于这个泛型上中包含了内部类的情况,一般返回null
    Type getOwnerType(); 
}
```

所以，知道了这里需要的泛型是怎么回事，一切都好说了，下面我们来完成之前留下的空方法。

## 3.1 实现一个简易的 ParameterizedType

```
public class ParameterizedTypeImpl implements ParameterizedType {
    private final Class raw;
    private final Type[] args;
    public ParameterizedTypeImpl(Class raw, Type[] args) {
        this.raw = raw;
        this.args = args != null ? args : new Type[0];
    }
    @Override
    public Type[] getActualTypeArguments() {
        return args;
    }
    @Override
    public Type getRawType() {
        return raw;
    }
    @Override
    public Type getOwnerType() {return null;}
}
```

## 3.2 生成Gson需要的泛型

### 3.2.1 解析data是object的情况

```java
public static <T> Result<T> fromJsonObject(Reader reader, Class<T> clazz) {
    Type type = new ParameterizedTypeImpl(Result.class, new Class[]{clazz});
    return GSON.fromJson(reader, type);
}
```

### 3.2.2 解析data是array的情况

是Array的情况要比是Object的情况多那么一步。

```java
public static <T> Result<List<T>> fromJsonArray(Reader reader, Class<T> clazz) {
    // 生成List<T> 中的 List<T>
    Type listType = new ParameterizedTypeImpl(List.class, new Class[]{clazz});
    // 根据List<T>生成完整的Result<List<T>>
    Type type = new ParameterizedTypeImpl(Result.class, new Type[]{listType});
    return GSON.fromJson(reader, type);
}
```

------

**本次代码较少，不提供源码**

**虽然这篇博客是以Gson为例，但从上面的内容可以看出实际上和Gson关系不大，主要的内容还是Java的泛型基础，所以这种封装的方法同样适用于其它的框架。**

最后借这次机会给安利一个简易的泛型生成库 **TypeBuilder** ，其最初实现的目的就是让大家快速的生成泛型信息，同时也会作一些参数检查，保证正确性。

用上面的代码给大家举个例子

```java
public static <T> Result<List<T>> fromJsonArray(Reader reader, Class<T> clazz) {
    Type type = TypeBuilder
            .newInstance(Result.class)
            .beginSubType(List.class)
            .addTypeParam(clazz)
            .endSubType()
            .build();
    return GSON.fromJson(reader, type);
}

public static <T> Result<T> fromJsonObject(Reader reader, Class<T> clazz) {
    Type type = TypeBuilder
            .newInstance(Result.class)
            .addTypeParam(clazz)
            .build();
    return GSON.fromJson(reader, type);
}
```