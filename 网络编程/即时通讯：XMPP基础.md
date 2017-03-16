# 即时通讯系列阅读

1. [即时通讯基础](http://blog.csdn.net/axi295309066/article/details/62427091)
2. [即时通讯：XMPP基础](http://blog.csdn.net/axi295309066/article/details/62427206)
3. [即时通讯：XMPP项目实践-微聊](http://blog.csdn.net/axi295309066/article/details/62427366)
4. [Smack类库最好的学习资料](http://blog.csdn.net/axi295309066/article/details/62435856)

# 1. XMPP 简介

XMPP（Extensible Messaging and Presence Protocol）是一种基于标准通用标记语言的子集XML 的协议，它继承了在XML 环境中灵活的发展性。因此，基于XMPP 的应用具有超强的可扩展性。经过扩展以后的XMPP 可以通过发送扩展的信息来处理用户的需求，以及在XMPP 的顶端建立如内容发布系统和基于地址的服务等应用程序。

而且，XMPP 包含了针对服务器端的软件协议，使之能与另一个进行通话，这使得开发者更容易建立客户应用程序或给一个配好系统添加功能。

XMPP（可扩展消息处理现场协议）是基于可扩展标记语言（XML）的协议，它用于即时消息（IM）以及在线现场探测。它在促进服务器之间的准即时操作。这个协议可能最终允许因特网用户向因特网上的其他任何人发送即时消息，即使其操作系统和浏览器不同。

XMPP 的前身是Jabber，一个开源形式组织产生的网络即时通信协议。XMPP 目前被IETF 国际标准组织完成了标准化工作。标准化的核心结果分为两部分；XMPP 其实就是用TCP 传的是XML 文件流。

xmpp特点

1. **开放**: XMPP协议是自由、开放、公开的，并且易于了解。 而且在客户端 、 服务器 、 组件 、 源码库等方面，都已经各自有多种实现。 
2. **标准**: 互联网工程工作小组（ IETF ）已经将Jabber的核心XML流协议以XMPP之名，正式列为认可的实时通信及Presence技术。 而XMPP的技术规格已被定义在RFC3920及RFC3921 。 任何IM供应商在遵循XMPP协议下，都可与Google Talk实现连接。 
3. **证实可用**: 第一个Jabber(现在XMPP)技术是Jeremie Miller在1998年开发的，现在已经相​​当稳定；数以百计的开发者为XMPP技术而努力。 今日的互联网上有数以万计的XMPP服务器运作着，并有数以百万计的人们使用XMPP实时传讯软件。 
4. **分散式**: XMPP网络的架构和电子邮件十分相像；XMPP核心协议通信方式是先创建一个stream，XMPP以TCP传递XML数据流，没有中央主服务器。 任何人都可以运行自己的XMPP服务器，使个人及组织能够掌控他们的实时传讯体验。 
5. **安全**: 任何XMPP协议的服务器可以独立于公众XMPP网络（例如在企业内部网络中），而使用SASL及TLS等技术的可靠安全性，已自带于核心XMPP技术规格中。 
6. **可扩展**: XML 命名空间的威力可使任何人在核心协议的基础上建造定制化的功能；为了维持通透性，常见的扩展由XMPP标准基金会 。 弹性佳 XMPP除了可用在实时通信的应用程序，还能用在网络管理、内容供稿、协同工具、文件共享、游戏、远程系统监控等。 
7. **多样性**: 用XMPP协议来建造及布署实时应用程序及服务的公司及开放源代码计划分布在各种领域；用XMPP技术开发软件，资源及支持的来源是多样的，使得使你不会陷于被“绑架”的困境。

下面给大家介绍XMPP 通信中最核心的三个XML 节（stanza）。这些节（stanza）有自己的作用和目标，通过组织不同的节（stanza），就能达到我们各种各样的通信目的。

```XML
<stream:stream>
    <iq
        id="roster1"
        type='get'>

        <query xmlns='jabber:iq:roster'/>
    </iq>

    <message
        from='test_account@jabber.org'
        to='william_duan@jabber.org'
        type='chat'>

        <body>Hello</body>
    </message>

    <presence type='unavailable'/>
</stream:stream>
```
在上面的xml 中，我们可以看到一些XMPP 节（stanza），包括&lt;iq>，&lt;message>以及&lt;presence>。接下来就对这些节（stanza）做一个大致的了解。

## 1.1节的共通属性

### from

表示节（stanza）的发送方，在发送节（stanza）时，一般来说不推荐设定，服务器会自动设定正确的值，如果设定了不正确的值，服务器将会拒收该节（stanza）信息。如果在客户端到服务器端的通信中接收的节（stanza）中没有该属性，会被默认解释为信息是由服务器发出的。如果在服务器到服务器的通信中接收的节（stanza）中没有本属性，则会被解释为一个error。

### to

表示节（stanza）的接收方。如果在客户端到服务器端的通信中没有设置本属性，服务器会默认解释为信息是发给自己的。

### type

指定节（stanza）的类型.每种节（stanza）都会有几种可能的设定值。所有的节（stanza）都会有一个error 类型，表明这个节（stanza）是一个error 回应，对这样的节（stanza）信息不需要进行回应。

### id

表示一个特定的请求。在<iq>节中，这个属性是必须要指定的，但是在其他两个节（stanza）中是一个可选属性。

## 1.2 iq节点

iq 节（stanza）主要是用于Info/Query 模式的消息请求，他和Http 协议比较相似。可以发出get 以及set 请求，就如同http 中的GET 以及POST。iq 节点需要有回应，有get，set 两种请求以及result，error 两种回应。发送查询消息示例：

```xml
<iq from="william_duan@jabber.org/study" type="get" id="roster1">
    <query xmlns="jabber:iq:roster"/>
</iq>
```
上面xml 的意思是william_duan 查询自己的联系人列表。

接收到回应示例：如果请求错误：
```xml
<iq to="william_duan@jabber.org/study" type="error" id="roster1">
    <query xmlns="jabber:iq:roster"/>
    <error type="cancel">
        <feature-not-implemented xmlns="urn:ietf:params:xml:ns:xmpp-stanzas"/>
    </error>
</iq>
```
如果请求成功：
```xml
<iq to="william_duan@jabber.org/study" type="result" id="roster1">
    <query xmlns="jabber:iq:roster"/>
    <item jid="account_one@jabber.org" name="one"/>
    <item jid="account_two@jabber.org" name="two"/>
</iq>
```
william 获取到了自己的花名册列表，该列表中有两个好友。

## 1.3 message 节点

正如名字一样，message 节（stanza）用于用户之间传递消息。这消息可以是单纯的聊天信息，也可以某种格式化的信息。message 节点信息是传递之后就被忘记的。当消息被送出之后，发送者是不管这个消息是否已经送出或者什么时候被接收到。通过扩展协议，可以改变这样一种状况。

私人会话示例：
```xml
<message from="william_duan@jabber.org" to="test_account@jabber.org" type="chat">
    <body>Come on</body>
    <thread>23sdfewtr234weasdf</thread>
</message>
```
群组会话示例：
```xml
<message from="test_account@jabber.org" to="william_duan@jabber.org" type="groupchat">
    <body>welcome</body>
</message>
```

## 1.4 presence 节点

presence 节（stanza）用来控制和表示实体的在线状态，可以展示从离线到在线甚至于离开，不能打扰等复杂状态，另外，还能被用来建立和结束在线状态的订阅。

在线状态示例：
```xml
//设定用户状态为在线
<presence/>
..............................
//设定用户状态为离线
<presence type="unavailable"/>
..............................
//用于显示用户状态的详细信息。上面的例子表明用户因为at the ball 在离开状态。
<presence>
	<show>away</show>
	<status>at the ball</status>
</presence>
```
<show>标签在presence 节点中最多出现一次，可以有以下取值：away，chat，dnd，xa

- away：离线
- chat:交谈中
- dnd:希望不被打扰
- xa:离开一段时间

<status>标签用于显示额外信息。

在线状态预定(presence subscription) ：

```xml
<presence
	from="william_duan@jabber.org"
	to="test_account@jabber.org"
	type="subscribe"/>

..............................
<presence
	from="test_account@jabber.org"
	to="william_duan@jabber.org"
 	type="subscribed"/>
```
通过上述交互，william_duan 就能看到test_account 的在线状态，并能接收到test_account 的在线状态通知了。

# 2. XMPP 服务器平台

我们在学习JavaWEB 的时候要用到web 服务器，那么我们就选择了Tomcat 作为web 服务器。同样的道理我们要学习即时通信，这整个体系是一个C/S 架构，Server 端不需要我们编写，那么我们就选择一款市场上免费开源的服务，除了Openfire 暂无他选，Openfire 是开源免费功能强大的IM 服务器。为什么选择Openfire 呢？请往下看。

Openfire 采用Java 开发，开源的实时协作（RTC）服务器基于XMPP（Jabber）协议。Openfire 安装和使用都非常简单，并利用Web 进行管理。单台服务器可支持上万并发用户。您可以使用它轻易的构建高效率的即时通信服务器。由于是采用开放的XMPP 协议，您可以使用各种支持XMPP 协议的IM 客户端软件登陆服务。

相关的下载：[asmack github](https://github.com/Flowdalic/asmack)、[asmack下载地址1](http://asmack.freakempire.de/)、[asmack下载地址2](http://code.google.com/p/asmack/downloads/list)、[openfire下载地址](http://www.igniterealtime.org/downloads/index.jsp)、[smack使用指南](http://www.igniterealtime.org/builds/smack/docs/latest/documentation/index.html)

## 2.1 案例-Openfire 的下载和安装

下载地址：http://www.igniterealtime.org/downloads/index.jsp

![](http://upload-images.jianshu.io/upload_images/3981391-efbe9ac458cb4d8b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们下载windows 平台的openfire_3_10_2.exe 文件

![](http://upload-images.jianshu.io/upload_images/3981391-3ec4581eaf6100db.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

双击开始安装

![](http://upload-images.jianshu.io/upload_images/3981391-ec14a086fd1b9dfc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

选择中文（简体），然后点击确定

![](http://upload-images.jianshu.io/upload_images/3981391-d05fefc0e1dff23d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

选择我接受协议，然后点击下一步

![](http://upload-images.jianshu.io/upload_images/3981391-7c25fa25d83510fb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

选择好安装路径，然后点击下一步

![](http://upload-images.jianshu.io/upload_images/3981391-c359644fea35c3db.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

耐心等待，大概50 秒

![](http://upload-images.jianshu.io/upload_images/3981391-182894c9d9d64ece.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击完成

![](http://upload-images.jianshu.io/upload_images/3981391-fe264ddc4cc503da.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

openfire启动失败问题，安装好后，启动，提示 [Java](http://lib.csdn.net/base/javase).io.FileNotFoundException: ..\lib\commons-el.jar ，解决办法：只需要 以管理员的身份运行openfire 即可

启动之后弹出如上界面，有异常！大概应该是日志相关的文件找不到。我们先不管，点击Launch Admin 启动
管理控制台。

![](http://upload-images.jianshu.io/upload_images/3981391-d953788c14ff89b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们注意上图中的地址栏，端口是9090，这是openfire 使用的端口，我们有时候喜欢把tomact 配置层9090，一定注意不要占用了该端口。

我们选择中文（简体），然后点击continue。

![](http://upload-images.jianshu.io/upload_images/3981391-43c8dac39b604dac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

设置域，其他设置默认即可。然后点击继续（上一页还是continue，选中过中文后可就成继续了）。

![](http://upload-images.jianshu.io/upload_images/3981391-d79122e7b75cf859.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

注意：这个时候必须先确保本机已经安装了MySQL，并创建了数据库，我创建的数据库名字就叫做openfire。
然后在该数据库中执行sql 脚本，该sql 脚本是openfire 提供的，位置位于：

![](http://upload-images.jianshu.io/upload_images/3981391-1cf962a9ca2f581b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在数据库中将该脚本内容执行一下，让其初始化数据表结构。然后点击继续，进入下一个页面。

![](http://upload-images.jianshu.io/upload_images/3981391-3d11ce5024e9bd47.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

使用默认配置，继续点击继续。进入下一个页面。

![](http://upload-images.jianshu.io/upload_images/3981391-3a6c86973b2adcb3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在上面填写邮箱和管理员密码，然后点击继续。

![](http://upload-images.jianshu.io/upload_images/3981391-27af983ba78366b4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

成功啦！点击登录到控制台，进入如下界面。

![](http://upload-images.jianshu.io/upload_images/3981391-bdbef4f03292274a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

openfire 管理员默认账号为admin，密码就是我们上一个界面设置的密码。输入账号和密码，然后点击登录。
进入下一个界面。

![](http://upload-images.jianshu.io/upload_images/3981391-d2d4cda2e01151a1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们就来浏览一下成功后的界面吧

用户列表界面：

![](http://upload-images.jianshu.io/upload_images/3981391-ac5f1da6a78c76cc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

会话界面：

![](http://upload-images.jianshu.io/upload_images/3981391-07873a381960d377.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

分组聊天界面：

![](http://upload-images.jianshu.io/upload_images/3981391-d6ac6acd76533486.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当我们用org.jivesoftware.smack.XMPPConnection.login(String username,String password)登录时，如果出现“SASLError using DIGEST-MD5: not-authorized”异常，这说明我们客户端没有开启SASL机制，在XMPP连接配置的时候，加上“SASLAuthentication.supportSASLMechanism("PLAIN",0);”这个支持SASL机制的方法就可以解决了。

# 3. XMPP 客户端平台

## 3.1 Spark 客户端的下载和安装

Spark 是一个开源，跨平台IM 客户端。它的特性支持集组聊天，电话集成和强大安全性能。如果企业内部部署IM 使用Openfire+Spark 是最佳的组合。其官网对Spark 的介绍如下：

Spark is an Open Source， cross-platform IM client optimized for businesses and organizations. It features built-in support for group chat， telephony integration， and strong security. It also offers a great end-user experience with features like in-line spell checking， group chat room bookmarks， and tabbed conversations。

Combined with the Openfire server， Spark is the easiest and best alternative to using un-secure public IM networks。

[Spark 的下载地址](http://www.igniterealtime.org/downloads/index.jsp)

![](http://upload-images.jianshu.io/upload_images/3981391-53cab1ce6b7bd51d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

下载好以后：

![](http://upload-images.jianshu.io/upload_images/3981391-156401b1bceeac49.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

双击安装。安装非常简单，一路点击下一步即可

![](http://upload-images.jianshu.io/upload_images/3981391-55da6fd9b18bbeb7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击Next，进入下一个界面：

![](http://upload-images.jianshu.io/upload_images/3981391-87c8eb66c2b6fc87.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击Next......Next 进入下一个界面：

![](http://upload-images.jianshu.io/upload_images/3981391-17d744f62fc13bbd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

看到如下界面就OK 了，如果在上一个界面点击Finish 没有起效，那么可以找到生成的桌面快捷图标，双击。

![](http://upload-images.jianshu.io/upload_images/3981391-9be0522b3fa20839.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击账号，进入下一个界面。

![](http://upload-images.jianshu.io/upload_images/3981391-a789e60870ef0601.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在上图中输入用户、密码和服务器地址。

![](http://upload-images.jianshu.io/upload_images/3981391-14cb9c170107886d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后创建账号。

![](http://upload-images.jianshu.io/upload_images/3981391-0e4c8c275e9cec7b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在上图中点击确定，进入如下图界面。

![](http://upload-images.jianshu.io/upload_images/3981391-aee7965db2119e0e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击Login 进入如下界面：

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3981391-04e1130e80dc9c6e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在openfire 中刷新界面，打开用户/组选项卡，可以看到所有注册的用户列表。

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3981391-382875ec15dfb8f1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 3.2 Spark 和Openfire 通信原理

从网上找到一个Spark 和Openfire 直接通讯架构图。看懂这张图就知道他们之间是如何通信的了。

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3981391-fb454c3723ee0794.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Smack 是对XMPP 协议的封装库，Smack 是XMPP 协议的Java 层的封装。让我们Java 程序员不用直接跟枯燥无味的XML 打交道（生成XML 和解析XML）。

随着移动互联网的快速发展，尤其是即时通信的发展，几乎90%以上的App 都有这样的功能。因此asmack也与13 年诞生，asmack 其实就是Android Smack 的简称。asmack 也是我们该门课程用到的主要API。

下面请看《即时通讯：XMPP项目实践-微聊》吧，让我们一起学习如何使用asmack 打造我们的交友神器。
