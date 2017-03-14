## 1. 解决Android requires .class compatibility set to 5.0. Please fix project properties

转载自：[http://gengen201314.javaeye.com/blog/714949](http://gengen201314.javaeye.com/blog/714949)

有时候在新创建的[Android](http://lib.csdn.net/base/android)项目或者导入进来的项目的时候会出现Android requires .class compatibility set to 5.0. Please fix project properties.这个错误.大概的意思是因为android的版本不一致导致的吧.因为我导进来的项目是android sdk1.5  而我的avd的版本是2.0 ,所以就出现了这个问题.

具体的解决方法如下:

1：  选择 project -> Android Tools ->Fix Project Properties.
重新 clean project
2：如果上面不管用，只能使用暴力一点的方法，复制源代码到信的目 录. 包括AndroidManifest.xml, src/, assets/, res/. 选择 File-> New-> Android Project -> Contents -> Create project from existing source -> <your new location>

## 2. location of the android adk has not been setup in the preferences 错误处理（eclipse）

此错误从字面上理解就是说你的adk没安，或者你安了eclipse没认出来？呵呵

在点击[Android](http://lib.csdn.net/base/android) sdk and avd manager时出现此错。

如果你安好了adk，就看一下window->preferences下的android，sdk location是否正确。

我7号用SDK Manager.exe升级到2.3，莫名其妙的就出现这个问题了。

## 3. 关于模拟器 Installation error: INSTALL_FAILED_MISSING_SHARED_LIBRARY的 错误问题

在测试Google map时，我直接使用原模拟器，所以报错：
Installation error: INSTALL_FAILED_MISSING_SHARED_LIBRARY
Please check logcat output for more details.
 Launch canceled!

原因是Target不对，本应用使用了Google api，要使用相应的Target。
我有重新创建了一个模拟器，并选择对应的Google api，
问题解决。

## 4. android java.io.IOException: Parent directory of file is not writable: /sdcard/...

今天遇到这个错误，第一反应就是权限问题，搜索了下，
找到和WRITE相关的permission有以下几个：
WRITE_CALENDAR    "Android.permission.WRITE_CALENDAR"  
WRITE_CONTACTS    "android.permission.WRITE_CONTACTS"  
WRITE_OWNER_DATA    "android.permission.WRITE_OWNER_DATA"  
WRITE_SETTINGS    "android.permission.WRITE_SETTINGS"  
WRITE_SMS    "android.permission.WRITE_SMS"  
WRITE_SYNC_SETTINGS    "android.permission.WRITE_SYNC_SETTINGS"

试过了都不行，
后来一看，还有个权限我没看到，那就是WRITE_EXTERNAL_STORAGE。这次好用了。
```xml
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```

## 5. Android NDK: APP variable defined to unknown applications: hello-jni

在使用Android-ndk-r4b用命令make APP=hello-jni编译hello-jni例子的时候，出现了以下错误信息：
Android NDK: APP variable defined to unknown applications: hello-jni
Android NDK: You might want to use one of the following:
build/core/main.mk:81: *** Android NDK: Aborting . Stop.
在查阅了OVERVIEW.TXT后，才发现r4b中编译的时候是以下步骤：
  1/ Place your native sources under $PROJECT/jni/...
  2/ Write $PROJECT/jni/Android.mk to describe your sources
     to the NDK build system
  3/ Optional: write $PROJECT/jni/Application.mk to describe your
     project in more details to the build system. You don't need
     one to get started though, but this allows you to target
     more than one CPU or override compiler/linker flags
     (see docs/APPLICATION-MK.TXT for all details).
  4/ Build your native code by running "$NDK/ndk-build" from your
     project directory, or any of its sub-directories.
照着做就编译成功了，如图：
Android NDK: APP variable defined to unknown applications: hello-jni - 枫 - 枫 的博客
本文转自：
http://mp870601.blog.163.com/blog/static/13675745320107824850564/
linc注：在r5b中也是一样，现在ndk的文章从网上一搜都是以前的，所以照着做会出现很多问题。希望大家还是以帮助文档为主。这样就能以不变应万变。

## 6. Android requires compiler compliance level 5.0. Please fix project properties

1. 项目 右键 ->Android tools ->Fix Project
2. 如果不可以，检查Project -> Properties->Java Compiler
    确认JDK compliance被设置为1.6,并且enable specific seetings.
3.再clean下项目就可以了。

## 7. Javah 常见错误记录-NDK与JNI除错
测试文件：hello-jni/src/com/example/hellojni/HelloJni.Java

```java
/**
 * 该文件来自 Android NDK Sample - HelloJni， 为了便于说明问题，我作了一些修改。
 */  
package com.example.hellojni;  

public class HelloJni  
{  
    public native String  stringFromJNI();  

    public native String  unimplementedStringFromJNI();  

    static {  
        System.loadLibrary("hello-jni");  
    }  
}  
```
错误一

```
david@xmomx:hellojni$ javac HelloJni.java   
david@xmomx:hellojni$ ls  
Hello.class  Hello.h  Hello.java  HelloJni.class  HelloJni.java  
david@xmomx:hellojni$ javah -jni HelloJni  
error: cannot access HelloJni  
bad class file: ./HelloJni.class  
class file contains wrong class: com.example.hellojni.HelloJni  
Please remove or make sure it appears in the correct subdirectory of the classpath.  
com.sun.tools.javac.util.Abort  
    at com.sun.tools.javac.comp.Check.completionError(Check.java:164)  
    at com.sun.tools.javadoc.DocEnv.loadClass(DocEnv.java:149)  
    at com.sun.tools.javadoc.RootDocImpl.<init>(RootDocImpl.java:77)  
    at com.sun.tools.javadoc.JavadocTool.getRootDocImpl(JavadocTool.java:159)  
    at com.sun.tools.javadoc.Start.parseAndExecute(Start.java:330)  
    at com.sun.tools.javadoc.Start.begin(Start.java:128)  
    at com.sun.tools.javadoc.Main.execute(Main.java:66)  
    at com.sun.tools.javah.Main.main(Main.java:147)  
javadoc: error - fatal error  
```
2 errors  

错误原因，没有在正确的路径下执行 javah 命令，应该在源码根目录下执行。
错误二:
```
david@xmomx:hellojni$ cd ../../../  
david@xmomx:src$ ls  
com  
david@xmomx:src$ javah -jni HelloJni  
error: cannot access HelloJni  
class file for HelloJni not found  
javadoc: error - Class HelloJni not found.  
Error: No classes were specified on the command line.  Try -help.  
```
错误原因：Classes 参数要使用完整类名，也就是说要加上包名
错误四：

```java
david@xmomx:src$ javah -jni com/example/hellojni/HelloJni  
javadoc: error - Illegal package name: "com/example/hellojni/HelloJni"  
1 error  
```
错误原因：完整类名格式错误

```java
david@xmomx:src$ javah -jni com.example.hellojni.HelloJni
```
OK，编译通过。
如果还有错误，说是类找不到还是什么的，请尝试添加 -classpath . 参数。如下：

```java
david@xmomx:src$ javah -jni -classpath . com.example.hellojni.HelloJni
```

   ​
