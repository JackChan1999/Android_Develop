## UncaughtExceptionHandler

未捕获异常指的是在程序中未使用try...catch...语句而抛出的异常。

## Java语法相关的异常

### NullPointerException

- 服务器返回数据为空
- 系统内存回收导致全局变量为空

### 角标越界

- IndexOutOfBoundsException
- StringIndexOutOfBoundsException
- ArrayIndexOutOfBoundsException

### 试图调用一个空对象的方法

### 类型转换异常

ClassCastException

### 数字转换错误

NumberFormatException

###  声明数组时长度为-1

NegativeArraySizeException

### 并发修改异常

ConcurrentModificationException，使用CopyOnWriteArrayList

### ClassNotFoundException

### 使用Arrays.asList()错误异常

```java
String str = "1,2,3,4,5";
List<String> list = Arrays.asList(str.split(","));
List arrayList = new ArrayList(list);
arrayList.remove("1");
```

## Activity相关的异常

### ActivityNotFoundException

## 序列化相关的异常

### 序列化时发现找不到类：被ProGuard混淆导致的崩溃

ProGuard对于Class.forName(className)中的class是无能为力的，它会将这个class混淆的面目全非，解决方法是在ProGuard中keep这个class

