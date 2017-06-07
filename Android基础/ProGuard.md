## 压缩Shrink

## 混淆Obfuscate 

DexGuard，代码混淆库

- 保留内嵌类
- 处理反射

## 优化Optimize

## 预检Preverify

## 通用版本混淆1

```
# 避免混淆泛型
-keepattributes Signature
 
# 不混淆注解
-keepattributes *Annatation*

# 代码混淆压缩比，在0和7之间，默认为5，一般不需要改
-optimizationpasses 5

# 混淆时不使用大小写混合，混淆后的类名为小写
-dontusemixedcaseclassnames

# 指定不去忽略非公共的库的类
-dontskipnonpubliclibraryclasses

# 指定不去忽略非公共的库的类的成员
-dontskipnonpubliclibraryclassmembers

# 不做预校验，preverify是proguard的4个步骤之一
# Android不需要preverify，去掉这一步可加快混淆速度
-dontpreverify

# 有了verbose这句话，混淆后就会生成映射文件
# 包含有类名->混淆后类名的映射关系
# 然后使用printmapping指定映射文件的名称
-verbose
-printmapping proguardMapping.txt

# 指定混淆时采用的算法，后面的参数是一个过滤器
# 这个过滤器是谷歌推荐的算法，一般不改变
-optimizations !code/simplification/arithmetic,!field/*,!class/merging/*

# 保护代码中的Annotation不被混淆，这在JSON实体映射时非常重要，比如fastJson
-keepattributes *Annotation*

# 避免混淆泛型，这在JSON实体映射时非常重要，比如fastJson
-keepattributes Signature

# 抛出异常时保留代码行号，在第6章异常分析中我们提到过
-keepattributes SourceFile,LineNumberTable

# 保留所有的本地native方法不被混淆
-keepclasseswithmembernames class * {
    native <methods>;
}

# 保留了继承自Activity、Application这些类的子类
# 因为这些子类，都有可能被外部调用
# 比如说，第一行就保证了所有Activity的子类不要被混淆
-keep public class * extends android.app.Activity

-keep public class * extends android.app.Application

-keep public class * extends android.app.Service

-keep public class * extends android.content.BroadcastReceiver

-keep public class * extends android.content.ContentProvider

-keep public class * extends android.app.backup.BackupAgentHelper

-keep public class * extends android.preference.Preference

-keep public class * extends android.view.View

-keep public class com.android.vending.licensing.ILicensingService

# 如果有引用android-support-v4.jar包，可以添加下面这行
-keep public class com.youngheart.app.ui.fragment.** {*;}

# 保留在Activity中的方法参数是view的方法，
# 从而我们在layout里面编写onClick就不会被影响
-keepclassmembers class * extends android.app.Activity {
    public void *(android.view.View);
}

# 枚举类不能被混淆
-keepclassmembers enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}

# 保留自定义控件（继承自View）不被混淆
-keep public class * extends android.view.View {

    *** get*();

    void set*(***);

    public <init>(android.content.Context);

    public <init>(android.content.Context, android.util.AttributeSet);

    public <init>(android.content.Context, android.util.AttributeSet, int);
}

# 保留Parcelable序列化的类不被混淆
-keep class * implements android.os.Parcelable {
    public static final android.os.Parcelable$Creator *;
}

# 保留Serializable序列化的类不被混淆
-keepclassmembers class * implements java.io.Serializable {

    static final long serialVersionUID;

    private static final java.io.ObjectStreamField[] serialPersistentFields;

    private void writeObject(java.io.ObjectOutputStream);

    private void readObject(java.io.ObjectInputStream);

    java.lang.Object writeReplace();

    java.lang.Object readResolve();
}

 

# 对于R（资源）下的所有类及其方法，都不能被混淆
-keep class **.R$* {
    *;
}

# 对于带有回调函数onXXEvent的，不能被混淆
-keepclassmembers class * {
    void *(**On*Event);
}

# 保留实体类和成员不被混淆
-keep public class com.youndheart.entity.** {

    public void set*(***);

    public *** get*();

    public *** is*();
}

# 保留内嵌类不被混淆
-keep class com.example.youngheart.MainActivity$* { *; }

# 对WebView的处理
-keepclassmembers class * extends android.webkit.webViewClient {

    public void *(android.webkit.WebView, java.lang.String, android.graphics.Bitmap);

    public boolean *(android.webkit.WebView, java.lang.String);
}

-keepclassmembers class * extends android.webkit.webViewClient {
    public void *(android.webkit.webView, java.lang.String);
}

# 保留JS方法不被混淆
-keepclassmembers class com.example.youngheart.MainActivity$JSInterface1 {
    <methods>;
}

# 针对android-support-v4.jar的解决方案
-libraryjars libs/android-support-v4.jar

-dontwarn android.support.v4.**

-keep class android.support.v4.**  { *; }

-keep interface android.support.v4.app.** { *; }

-keep public class * extends android.support.v4.**

-keep public class * extends android.app.Fragment

# 对alipay的混淆处理
-libraryjars libs/alipaysdk.jar

-dontwarn com.alipay.android.app.**

-keep public class com.alipay.**  { *; }
```

## 通用版本混淆2

```
#指定压缩级别
-optimizationpasses 5

#不跳过非公共的库的类成员
-dontskipnonpubliclibraryclassmembers

#混淆时采用的算法
-optimizations !code/simplification/arithmetic,!field/*,!class/merging/*

#把混淆类中的方法名也混淆了
-useuniqueclassmembernames

#优化时允许访问并修改有修饰符的类和类的成员 
-allowaccessmodification

#将文件来源重命名为“SourceFile”字符串
-renamesourcefileattribute SourceFile
#保留行号
-keepattributes SourceFile,LineNumberTable
#保持泛型
-keepattributes Signature

#保持所有实现 Serializable 接口的类成员
-keepclassmembers class * implements java.io.Serializable {
    static final long serialVersionUID;
    private static final java.io.ObjectStreamField[] serialPersistentFields;
    private void writeObject(java.io.ObjectOutputStream);
    private void readObject(java.io.ObjectInputStream);
    java.lang.Object writeReplace();
    java.lang.Object readResolve();
}

#Fragment不需要在AndroidManifest.xml中注册，需要额外保护下
-keep public class * extends android.support.v4.app.Fragment
-keep public class * extends android.app.Fragment

# 保持测试相关的代码
-dontnote junit.framework.**
-dontnote junit.runner.**
-dontwarn android.test.**
-dontwarn android.support.test.**
-dontwarn org.junit.**
```