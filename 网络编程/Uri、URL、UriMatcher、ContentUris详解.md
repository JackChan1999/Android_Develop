# **1. URI：统一资源标识符**

URI是统一资源标识符，是一个用于标识某一互联网资源名称的字符串。 该种标识允许用户对任何（包括本地和互联网）的资源通过特定的协议进行交互操作。URI由包括确定语法和相关协议的方案所定义。由是三个组成部分：访问资源的命名机制、存放资源的主机名、资源自身的名称，由路径表示。

## **1.1 URI的使用方法**
![url](http://img.blog.csdn.net/20161019100615837)

- **schema**：用来说明一个ContentProvider 控制哪些数据。"content://"

- **authority**：主机名或授权，它定义了是哪个ContentProvider 提供这些数据

- **path**：路径，URI 下的某一个Item。

- **ID**：通常定义Uri 时使用”#”号占位符代替, 使用时替换成对应的数字
  content://com.itheima.provider/person/#：#表示数据id（#代表任意数字）
  content://com.itheima.provider/person/*：*来匹配任意文本。

为系统的每一个资源给其一个名字，比方说通话记录。

1、每一个ContentProvider都拥有一个公共的URI，这个URI用于表示这个ContentProvider所提供的数据。 

2、Android所提供的ContentProvider都存放在android.provider包中。 将其分为A，B，C，D 4个部分：

- A：**标准前缀**，用来说明一个Content Provider控制这些数据，无法改变的；"content://"

- B：**URI 的标识**，它定义了是哪个Content Provider提供这些数据。对于第三方应用程序，为了保证URI标识的唯一性，它必须是一个完整的、小写的 类名。这个标识在 元素的 authorities属性中说明：一般是定义该ContentProvider的包.类的名称;"content://hx.android.text.myprovider"

- C：**路径**，不知道是不是路径，通俗的讲就是你要操作的数据库中表的名字，或者你也可以自己定义，记得在使用的时候保持一致就ok了；"content://hx.android.text.myprovider/tablename"

- D：如果URI中包含表示需要获取的记录的ID；则就返回该id对应的数据，如果没有ID，就表示返回全部； "content://hx.android.text.myprovider/tablename/#" #表示数据id

通用资源标志符（Universal Resource Identifier, 简称"URI"）。
Uri代表要操作的数据，Android上可用的每种资源 - 图像、视频片段等都可以用Uri来表示。

URI一般由三部分组成：

- 访问资源的命名机制。 
- 存放资源的主机名。 
- 资源自身的名称，由路径表示。

Android的Uri由以下三部分组成： "content://"、数据的路径、标示ID(可选)举些例子，如： 

- 所有联系人的Uri： content://contacts/people
- 某个联系人的Uri: content://contacts/people/5
- 所有图片Uri: content://media/external
- 某个图片的Uri：content://media/external/images/media/4

## 1.2 URI常用方法
| 方法                   | 说明              |
| -------------------- | --------------- |
| withAppendedPath()   | 追加路径            |
| encode()             |                 |
| fromFile()           | 从一个文件新建一个uri    |
| parse()              | 解析uri           |
| getScheme()          | 获取uri中的schema部分 |
| getLastPathSegment() |                 |
<br>
我们很经常需要解析Uri，并从Uri中获取数据。
Android系统提供了两个用于操作Uri的工具类，分别为UriMatcher 和ContentUris 。

虽然这两类不是非常重要，但是掌握它们的使用，会便于我们的开发工作。下面就一起看一下这两个类的作用。


# **2. UriMatcher**
UriMatcher 类主要用于匹配Uri。使用方法如下
首先第一步，初始化：

```java
UriMatcher matcher = new UriMatcher(UriMatcher.NO_MATCH);  
```

第二步注册需要的Uri:

```java
matcher.addURI("com.yfz.Lesson", "people", PEOPLE);  
matcher.addURI("com.yfz.Lesson", "person/#", PEOPLE_ID);  
```

第三部，与已经注册的Uri进行匹配:

```java
Uri uri = Uri.parse("content://" + "com.yfz.Lesson" + "/people");  
int match = matcher.match(uri);  
       switch (match)  
       {  
           case PEOPLE:  
               return "vnd.android.cursor.dir/people";  
           case PEOPLE_ID:  
               return "vnd.android.cursor.item/people";  
           default:  
               return null;  
       }  
Uri uri = Uri.parse("content://" + "com.yfz.Lesson" + "/people");
int match = matcher.match(uri);
       switch (match)
       {
           case PEOPLE:
               return "vnd.android.cursor.dir/people";
           case PEOPLE_ID:
               return "vnd.android.cursor.item/people";
           default:
               return null;
       } 
```

match方法匹配后会返回一个匹配码Code，即在使用注册方法addURI时传入的第三个参数。上述方法会返回"vnd.android.cursor.dir/person".

总结: 

- 常量 UriMatcher.NO_MATCH 表示不匹配任何路径的返回码
- \# 号为通配符
- \* 号为任意字符

另外说一下，官方SDK说明中关于Uri的注册是这样写的：

```java
private static final UriMatcher sURIMatcher = new UriMatcher();  
static  
{  
    sURIMatcher.addURI("contacts", "/people", PEOPLE);  
    sURIMatcher.addURI("contacts", "/people/#", PEOPLE_ID);  
    sURIMatcher.addURI("contacts", "/people/#/phones", PEOPLE_PHONES);  
    sURIMatcher.addURI("contacts", "/people/#/phones/#", PEOPLE_PHONES_ID);  
    sURIMatcher.addURI("contacts", "/people/#/contact_methods", PEOPLE_CONTACTMETHODS);  
    sURIMatcher.addURI("contacts", "/people/#/contact_methods/#", PEOPLE_CONTACTMETHODS_ID);  
    sURIMatcher.addURI("contacts", "/deleted_people", DELETED_PEOPLE);  
    sURIMatcher.addURI("contacts", "/phones", PHONES);  
    sURIMatcher.addURI("contacts", "/phones/filter/*", PHONES_FILTER);  
    sURIMatcher.addURI("contacts", "/phones/#", PHONES_ID);  
    sURIMatcher.addURI("contacts", "/contact_methods", CONTACTMETHODS);  
    sURIMatcher.addURI("contacts", "/contact_methods/#", CONTACTMETHODS_ID);  
    sURIMatcher.addURI("call_log", "/calls", CALLS);  
    sURIMatcher.addURI("call_log", "/calls/filter/*", CALLS_FILTER);  
    sURIMatcher.addURI("call_log", "/calls/#", CALLS_ID);  
}  
```

这个说明估计已经是Google官方没有更新，首先是初始化方法，没有传参，那么现在初始化时，实际是必须传参的。 可以看一下Android2.2的源码，无参数的构造方法已经是private的了。

另外就是addURI这个方法，第二个参数开始时不需要"/"， 否则是无法匹配成功的。

| 方法                                       | 说明                |
| ---------------------------------------- | ----------------- |
| addURI(String authority, String path, int code) | code：表示匹配成功以后的返回值 |
| match()                                  | 匹配uri             |

# **3. ContentUris**
ContentUris 类用于获取Uri路径后面的ID部分

1、为路径加上ID: withAppendedId(uri, id)
比如有这样一个Uri

```java
Uri uri = Uri.parse("content://com.yfz.Lesson/people")  
```

通过withAppendedId方法，为该Uri加上ID

```java
Uri resultUri = ContentUris.withAppendedId(uri, 10);  
```

最后resultUri为: content://com.yfz.Lesson/people/10

2、从路径中获取ID: parseId(uri)

```java
Uri uri = Uri.parse("content://com.yfz.Lesson/people/10")  
long personid = ContentUris.parseId(uri);  
```

最后personid 为 :10

附上实验的代码： 

```java
public class Lesson_14 extends Activity {        
    private static final String AUTHORITY = "com.yfz.Lesson";  
    private static final int PEOPLE = 1;  
    private static final int PEOPLE_ID = 2;        
    //NO_MATCH表示不匹配任何路径的返回码   
    private static final UriMatcher sURIMatcher = new UriMatcher(UriMatcher.NO_MATCH);  
    static  
    {  
        sURIMatcher.addURI(AUTHORITY, "people", PEOPLE);            
        //这里的#代表匹配任意数字，另外还可以用*来匹配任意文本   
        sURIMatcher.addURI(AUTHORITY, "people/#", PEOPLE_ID);  
    }        
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        Logger.d("------ Start Activity !!! ------");            
        Uri uri1 = Uri.parse("content://" + AUTHORITY + "/people");  
        Logger.e("Uri:" + uri1);  
        Logger.d("Match 1" + getType(uri1));            
        Uri uri2 = Uri.parse("content://" + AUTHORITY + "/people" + "/2");   
        Logger.e("Uri:" + uri2);  
        Logger.d("Match 2" + getType(uri2));  
        //拼接Uri   
        Uri cUri = ContentUris.withAppendedId(uri1, 15);  
        Logger.e("Uri:" + cUri);  
        //获取ID   
        long id = ContentUris.parseId(cUri);  
        Logger.d("Uri ID: " + id);  
    }        
    private String getType(Uri uri) {  
        int match = sURIMatcher.match(uri);  
        switch (match)  
        {  
            case PEOPLE:  
                return "vnd.android.cursor.dir/person";  
            case PEOPLE_ID:  
                return "vnd.android.cursor.item/person";  
            default:  
                return null;  
        }  
    }  
}  
```

ContentUris 是解析Uri 的一个工具类，可以将Uri 中的id 给解析出来

| 方法                      | 说明                               |
| ----------------------- | -------------------------------- |
| parseId(uri)            | 获取path中的数据                       |
| withAppendedId(uri, id) | 使用ContentUris工具类将id追加到uri中，返回给客户 |

# **4. URL：统一资源定位符**
统一资源定位符也就是说根据URL能够定位到网络上的某个资源，它是指向互联网“资源”的指针。是对可以从互联网上得到的资源的位置和访问方法的一种简洁的表示，是互联网上标准资源的地址。互联网上的每个文件都有一个唯一的URL，它包含的信息指出文件的位置以及浏览器应该怎么处理它。

每个URL都是URI，但不一定每个URI都是URL。这是因为URI还包括一个子类，即统一资源名称（URN），它命名资源但不指定如何定位资源。

比如百度URL即是http://www.baidu.com

![url](http://img.blog.csdn.net/20161105201213875)

# **5. URI、URL 和 URN** 
URI 是统一资源标识符，而 URL 是统一资源定位符。因此，笼统地说，每个 URL 都是 URI，但不一定每个 URI 都是 URL。这是因为 URI 还包括一个子类，即统一资源名称 (URN)，它命名资源但不指定如何定位资源。

