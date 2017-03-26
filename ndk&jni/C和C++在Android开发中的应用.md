# 1. 前言

## 1.1 Android SDK介绍

Android是基于Linux内核的一个手机操作系统，谷歌提供了开发包（Android SDK），程序员可以通过开发包开发Android App(应用程序)。Android SDK提供Java语言接口，因此Android应用是使用Java开发的。

## 1.2 使用纯Java开发App的缺点

在某些场合下，使用纯Java开发Android应用程序不完美，比如：

- 有高性能算法，Java语言无法满足
- 有跨平台需求，希望将App移植到iOS
- 已有代码的重用

## 1.3 引入NDK

早在Android 1.6(2009年)时，google就提供了NDK（native development kit），NDK包括了一套Android的交叉编译环境和开发库，利用它可以编写C/C++程序，并编译成Android环境下使用的动态库，Java代码通过Jni规范，调用C/C++写的动态库。

目前最新的Android Studio 2.2中，集成了C/C++开发环境，开发人员在使用C/C++更加简单了。

# 2. 课程内容

## [NDK中文官方开发技术文档地址](https://developer.android.google.cn/ndk/index.html)

![](img/jni14.png)

## 下载配置NDK

[NDK下载地址](https://developer.android.google.cn/ndk/downloads/index.html)

![](img/jni13.png)

配置NDK

![](img/jni4.png)

![](img/jni5.png)

如果不配置NDK路径，会报NDK没有配置错误

![](img/jni3.png)

## JNI开发HelloWorld

把 Include C++ support的勾打上

![](img/jni1.png)

选择C++11和Toolchain Default均可，C++11有更多的新特性和功能

![](img/jni2.png)

点击Finish后，进入工程目录，如图所示，除了java文件夹外多了一个cpp文件夹，cpp就是存放c和c++代码的文件夹

![](img/jni8.png)

## 配置NDK开发环境中遇到的坑

Failed to find CMake

![](img/jni6.png)

什么，CMake是什么鬼，原来，在Android Studio 2.2 后，NDK开发更加人性化了，使用了[**CMake**](https://cmake.org/)，一款外部构建工具，可与 Gradle 搭配使用来构建原生库。如果您只计划使用 ndk-build，则不需要此组件。还有[**LLDB**](http://lldb.llvm.org/)，一种调试程序，Android Studio 使用它来[调试原生代码](https://developer.android.google.cn/studio/debug/index.html)。

点击Install CMake and sync project，提示如下错误

![](img/jni7.png)

```
Gradle sync failed: Failed to find CMake.
Install from Android Studio under File/Settings/Appearance & Behavior/System Settings/Android SDK/SDK Tools/CMake.
Expected CMake executable at D:\android-sdk\cmake\bin\cmake.exe.
Consult IDE log for more details (Help | Show Log)
```

原来是我使用了代理，因为之前Google的链接需要翻墙才能够使用，所以配置了某代理，但是该代理不管用，在设置中把代理去掉即可。在Google在中国开了发布会后，Google的链接可以使用了，Android开发官网也可以上了，而且翻译了大量的技术文档，方便了英语不太好的同学

![](img/jni11.png)

打开 SDK Manager，安装上CMake和LLDB

![](img/jni15.png)

![](img/jni9.png)

![](img/jni10.png)

更多更详细的NDK开发文档，请看Android官方中文文档[向您的项目添加 C 和 C++ 代码](https://developer.android.google.cn/studio/projects/add-native-code.html)

## 2.3 Android Java代码调用C++代码

Java部分代码

```java
public class Jni {
    static  {
        System.loadLibrary("bc-lib"); // libbc-lib.so
    }
 
    private static Jni obj = new Jni();
    private Jni(){}
 
    public static Jni instance(){
        return obj;
    }
 
    // native接口
    public native boolean Login(String username, String password, String type);
    public native boolean Reg(String username, String password, String mobile, String email, String id);
    public native boolean LocationChange(double lng, double lat);
    public native boolean StartOrder(double lng1, double lat1, double lng2, double lat2);
}
```

C++部分代码

```c++
JNIEXPORT jboolean JNICALL Java_cn_xueguoliang_hc_Jni_Login
        (JNIEnv *env, jobject /* Jni object */, jstring jUsername, jstring jPassword, jstring type)
{
    return (jboolean)User::instance()->Login(j2c(env, jUsername), j2c(env, jPassword),
    j2c(env, type));
}
 
JNIEXPORT jboolean JNICALL Java_cn_xueguoliang_hc_Jni_Reg
        (JNIEnv *env, jobject /* Jni object */,
         jstring jUsername, jstring jPassword, jstring mobile, jstring email, jstring id)
{
    return (jboolean)User::instance()->Reg(
            j2c(env, jUsername),
            j2c(env, jPassword),
            j2c(env, mobile),
            j2c(env, email),
            j2c(env, id));
}
 
JNIEXPORT jboolean JNICALL Java_cn_xueguoliang_hc_Jni_LocationChange
        (JNIEnv *, jobject, jdouble lng, jdouble lat)
{
    User::instance()->LocationChange(lng, lat);
    return (jboolean)true;
}
 
 
JNIEXPORT jboolean JNICALL Java_cn_xueguoliang_hc_Jni_StartOrder
        (JNIEnv *, jobject, jdouble lng1, jdouble lat1, jdouble lng2, jdouble lat2)
{
    return (jboolean)Order::instance()->start(lng1, lat1, lng2, lat2);
}
```

## 2.4 C++代码调用Java代码

Java代码

```java
public class Jni {
    static {
        System.loadLibrary("native-lib");
    }
 
    private static Jni obj = new Jni();
    public static Jni instance()
    {
        return obj;
    }
 
    public native void HelloWorld();
 
    void callByCpp()
    {
        Log.e("JniCallback", "hello java");
    }
}
```

C++代码

```c++
extern "C"
void
Java_com_example_xueguoliang_test_Jni_HelloWorld(
        JNIEnv* env,
        jobject  This ) {
    std::string hello = "Hello from C++";
 
    jclass jniClass = env->FindClass("com/example/xueguoliang/test/Jni");
    jmethodID jmethodID1 = env->GetMethodID(jniClass, "callByCpp", "()V");
    env->CallVoidMethod(This, jmethodID1);
 
    return;
}
```

## 2.5 Java和C++字符串转换

```c++
jstring c2j(JNIEnv* env, string cstr)
{
    return env->NewStringUTF(cstr.c_str());
}
 
string j2c(JNIEnv* env, jstring jstr)
{
    string ret;
    jclass stringClass = env->FindClass("java/lang/String");
    jmethodID getBytes = env->GetMethodID(stringClass, "getBytes", "(Ljava/lang/String;)[B");
 
    // 把参数用到的字符串转化成java的字符
    jstring arg = c2j(env, "utf-8");
 
    jbyteArray jbytes = (jbyteArray)env->CallObjectMethod(jstr, getBytes, arg);
 
    // 从jbytes中，提取UTF8格式的内容
    jsize byteLen = env->GetArrayLength(jbytes);
    jbyte* JBuffer = env->GetByteArrayElements(jbytes, JNI_FALSE);
 
    // 将内容拷贝到C++内存中
    if(byteLen > 0)
    {
        char* buf = (char*)JBuffer;
        std::copy(buf, buf+byteLen, back_inserter(ret));
    }
 
    // 释放
    env->ReleaseByteArrayElements(jbytes, JBuffer, 0);
    return ret;
}
```

## 2.6 javah和javap

javah用于生成native接口定义，比如

```
javah -d ../cpp/ com.example.xueguoliang.test.Jni
```

javap用于生成java函数的签名，比如

```
javap -s Jni
```