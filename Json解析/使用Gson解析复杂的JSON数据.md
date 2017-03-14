JSON(JavaScript Object Notation) 是一种轻量级的数据交换格式，目前广泛使用。本文主要讲解android如何解析复杂格式的JSON数据，适合android初学者和初步接触JSON的人

# **1. JSON相关介绍**
首先了解一下JSON的相关内容，JSON中的数据是以键值对的形式出现的。例如”name”:”jack”；这就是一个键值对，可以理解为name=jack。JSON中很重要的两个概念就是数组(array)和对象(object)，初学者们很容易会把这两个概念搞错。区分是数组还是对象最简单的办法就是看是在{ ……}(大括号中)，还是在 …… 。Object在大括号中，数组在中括号中。所有的JSON数据都是建立在数组和对象的基础上经过不同的组合而生成的。因此，可以说理解了数组和对象，就可以解析出各种复杂的JSON数据。

# **2. Android中使用Gson进行JSON解析**
Android解析JSON数据的方法有很多，例如：原生android的解析方法就是JSONObject和JSONArray，这样的解析有些繁琐。还有就是利用第三方的包，本文主要是使用Gson对JSON数据进行解析，当然还有FastJson，这个也很不错，大家可以试一试

导入gson包

```gradle
compile 'com.google.code.gson:gson:2.7'
```
# **3. 如何使用GSON解析**

解析JSON数据最重要的一步就是根据JSON数据构建出bean文件，这里一步一步教大家如何构建出bean文件。 JSON数据如下：

```json
{
    "showapi_res_body": {
        "pagebean": {
            "allNum": 577,
            "allPages": 29,
            "contentlist": [
                {
                    "channelId": "5572a108b3cdc86cf39001cd",
                    "channelName": "国内焦点",
                    "desc": " 　　中国警察网北京4月22日电中国共产党的优秀党员，公安部原部长、党委书记陶驷驹同志，因病医治无效，于2016年4月18日0时36分在北京逝世， 享年81岁。22日上午，陶驷驹同志遗体送别仪式在北京举行。　　习近平、李克强、刘云山、张高丽、刘延东、李源潮、孟建柱、赵乐际、胡锦....",
                    "imageurls": [
                        {
                            "height": 480,
                            "url": "http://n.sinaimg.cn/news/transform/20160423/P7Jb-fxrqhar9853560.jpg",
                            "width": 400
                        }
                    ],
                    "link": "http://news.sina.com.cn/c/nd/2016-04-23/doc-ifxrpvea1126476.shtml",
                    "pubDate": "2016-04-23 10:16:54",
                    "source": "新浪",
                    "title": "公安部原部长陶驷驹逝世 习近平胡锦涛等送花圈"
                },
                {
                    "channelId": "5572a108b3cdc86cf39001cd",
                    "channelName": "国内焦点",
                    "desc": " 　　原标题：山东3县市购房补钱！继邹城、寿光后，即墨也加入资料图　　日前，即墨市制定《关于促进房地产市场持续健康平稳发展的实施意见》，即墨市财政 将对购房者予以补贴。从《意见》出台之日起到今年年底前，在即墨首次购新建商品房的市民，每平米可领取补贴50到200元。对购....",
                    "imageurls": [
                        {
                            "height": 327,
                            "url": "http://ww4.sinaimg.cn/mw690/77de9208jw1f36aamk03mj20go093406.jpg",
                            "width": 600
                        }
                    ],
                    "link": "http://news.sina.com.cn/c/nd/2016-04-23/doc-ifxrpvea1122744.shtml",
                    "pubDate": "2016-04-23 07:52:13",
                    "source": "新浪",
                    "title": "山东3县市对购房者予以财政补贴"
                },
                {
                    "channelId": "5572a108b3cdc86cf39001cd",
                    "channelName": "国内焦点",
                    "desc": " 　　原标题：首家商业火箭公司成立　　京华时报讯（记者潘珊菊）昨天下午，记者从航天科工集团获悉，在成功发射首颗卫星“东方红一号”46年后，中国航天 技术步入“商用时代”，我国首家商业模式开展研发和应用的专业化火箭公司已于今年2月16日在武汉注册成立。　　据介绍，该公....",
                    "imageurls": [],
                    "link": "http://news.sina.com.cn/o/2016-04-23/doc-ifxrpvqz6479220.shtml",
                    "pubDate": "2016-04-23 03:19:35",
                    "source": "新浪",
                    "title": "中国首家商业火箭公司成立 注册时曾引官方争议"
                }
            ],
            "currentPage": 1,
            "maxResult": 20
        },
        "ret_code": 0
    },
    "showapi_res_code": 0,
    "showapi_res_error": ""
}
```
![json](http://img.blog.csdn.net/20161125143531323)

一步一步来

![json](http://itfish.net/Home/Modules/Images/itfish_58577_1.jpg)

这是把所有的括号都收起来的样子

## 3.1 展开大括号
这是第一层，我们给一个标记为A

![json](http://itfish.net/Home/Modules/Images/itfish_58577_2.jpg)

## 3.2 展开showapi_res_body后面的大括号

这是第二层，我们给一个标记为B

![json](http://itfish.net/Home/Modules/Images/itfish_58577_3.jpg)
## 3.3 展开pagebean后面的大括号

这是第三层，我们给一个标记为C

![json](http://itfish.net/Home/Modules/Images/itfish_58577_4.jpg)
## 3.4 展开contentlist后面的中括号，这是一个数组

这是第四层，我们给一个标记为D

![json](http://itfish.net/Home/Modules/Images/itfish_58577_5.jpg)
## 3.5 展开contentlist里面的object中的大括号
这是第五层，我们给一个标记为E

![json](http://itfish.net/Home/Modules/Images/itfish_58577_6.jpg)

## 3.6 展开imageurls的中括号

这是第六层，我们给一个标记为F

![json](http://itfish.net/Home/Modules/Images/itfish_58577_7.jpg)
## 3.7 展开imageurls里面object的大括号 

这是第七层，我们给一个标记为G

![json](http://itfish.net/Home/Modules/Images/itfish_58577_8.jpg)

到此，所有的括号全部展开，所有的结构也非常清晰。contentlist和imageurls是两个JSON数组而且数组的元素是JSON对象。

下面就开始构建bean文件了。 首先，建一个包 com.example.bean ，包内放的就是bean文件。 如图A所示，第一层构建一个类：


1．我们构建一个java类HomeNewsBean

```java
package com.example.bean.homenews;

public class HomeNewsBean {
    private String showapi_res_code;
    private String showapi_res_error;
    private HomeNewsBeanBody showapi_res_body;
    public String getShowapi_res_code() {
        return showapi_res_code;
    }
    public void setShowapi_res_code(String showapi_res_code) {
        this.showapi_res_code = showapi_res_code;
    }
    public String getShowapi_res_error() {
        return showapi_res_error;
    }
    public void setShowapi_res_error(String showapi_res_error) {
        this.showapi_res_error = showapi_res_error;
    }
    public HomeNewsBeanBody getShowapi_res_body() {
        return showapi_res_body;
    }
    public void setShowapi_res_body(HomeNewsBeanBody showapi_res_body) {
        this.showapi_res_body = showapi_res_body;
    }
    @Override
    public String toString() {
        return "HomeNewsBean [showapi_res_code=" + showapi_res_code
                + ", showapi_res_error=" + showapi_res_error
                + ", showapi_res_body=" + showapi_res_body + "]";
    }
}
```

2．类中HomeNewsBeanBody是第二层B中的类。

```java
package com.example.bean.homenews;

public class HomeNewsBeanBody {
    private HomeNewsPageBean pagebean;
    private String ret_code;
    public HomeNewsPageBean getPagebean() {
        return pagebean;
    }
    public void setPagebean(HomeNewsPageBean pagebean) {
        this.pagebean = pagebean;
    }
    public String getRet_code() {
        return ret_code;
    }
    public void setRet_code(String ret_code) {
        this.ret_code = ret_code;
    }
    @Override
    public String toString() {
        return "HomeNewsBeanBody [pagebean=" + pagebean + ", ret_code="
                + ret_code + "]";
    }
}
```

3．类中HomeNewsPageBean是第三层C中的类

```java
package com.example.bean.homenews;

import java.util.List;

public class HomeNewsPageBean {
    private String allNum;
    private String allPages;
    private String currentPage;
    private String maxResult;
    private List<HomeNewsContentList> contentlist;
    public String getAllNum() {
        return allNum;
    }
    public void setAllNum(String allNum) {
        this.allNum = allNum;
    }
    public String getAllPages() {
        return allPages;
    }
    public void setAllPages(String allPages) {
        this.allPages = allPages;
    }
    public String getCurrentPage() {
        return currentPage;
    }
    public void setCurrentPage(String currentPage) {
        this.currentPage = currentPage;
    }
    public String getMaxResult() {
        return maxResult;
    }
    public void setMaxResult(String maxResult) {
        this.maxResult = maxResult;
    }
    public List<HomeNewsContentList> getContentlist() {
        return contentlist;
    }
    public void setContentlist(List<HomeNewsContentList> contentlist) {
        this.contentlist = contentlist;
    }
    @Override
    public String toString() {
        return "HomeNewsPageBean [allNum=" + allNum + ", allPages=" + allPages
                + ", currentPage=" + currentPage + ", maxResult=" + maxResult
                + ", contentlist=" + contentlist + "]";
    }
}
```

4．类中HomeNewsContentList是第和第四层D和第五层E中的类，这里注意：JSON数据中，这是数组，因此要使用List来存放。

```java
package com.example.bean.homenews;

import java.util.List;

public class HomeNewsContentList {
    private String channelId;
    private String channelName;
    private String desc;
    private List<HomeNewsImages> imageurls;
    private String link;
    private String pubDate;
    private String source;
    private String title;
    public String getChannelId() {
        return channelId;
    }
    public void setChannelId(String channelId) {
        this.channelId = channelId;
    }
    public String getChannelName() {
        return channelName;
    }
    public void setChannelName(String channelName) {
        this.channelName = channelName;
    }
    public String getDesc() {
        return desc;
    }
    public void setDesc(String desc) {
        this.desc = desc;
    }
    public List<HomeNewsImages> getImageurls() {
        return imageurls;
    }
    public void setImageurls(List<HomeNewsImages> imageurls) {
        this.imageurls = imageurls;
    }
    public String getLink() {
        return link;
    }
    public void setLink(String link) {
        this.link = link;
    }
    public String getPubDate() {
        return pubDate;
    }
    public void setPubDate(String pubDate) {
        this.pubDate = pubDate;
    }
    public String getSource() {
        return source;
    }
    public void setSource(String source) {
        this.source = source;
    }
    public String getTitle() {
        return title;
    }
    public void setTitle(String title) {
        this.title = title;
    }
    @Override
    public String toString() {
        return "HomeNewsContentList [channelId=" + channelId + ", channelName="
                + channelName + ", desc=" + desc + ", imageurls=" + imageurls
                + ", link=" + link + ", pubDate=" + pubDate + ", source="
                + source + ", title=" + title + "]";
    }
}
```

5．同理类中HomeNewsImages是第六层F和第七层G中的类，也是List存放。

```java
package com.example.bean.homenews;

public class HomeNewsImages {
    private String height;
    private String url;
    private String width;
    public String getHeight() {
        return height;
    }
    public void setHeight(String height) {
        this.height = height;
    }
    public String getUrl() {
        return url;
    }
    public void setUrl(String url) {
        this.url = url;
    }
    public String getWidth() {
        return width;
    }
    public void setWidth(String width) {
        this.width = width;
    }
    @Override
    public String toString() {
        return "HomeNewsImages [height=" + height + ", url=" + url + ", width="
                + width + "]";
    }
}
```

到此为止，所有的bean文件全部构建完毕。

注意：所有的JSON数据的key：value键值对中的key要和bean文件中的对象一一对应，名字要完全一样！不然无法解析！！！
例如，这里面的showapi_res_body要和JSON数据中的 一模一样。

接下来就是解析的过程了，最难的地方已经过去，接下来就是几行代码的事情了。

首先：传入URL,发送http请求，从服务器获取JSON数据。

```java
public static String netLink(String URL) {
        HttpClient httpClient = new DefaultHttpClient();
        //访问指定的服务器
        HttpGet httpGet = new HttpGet(URL);
        HttpResponse httpResponse = null;
        String response = null;
        try {
            httpResponse = httpClient.execute(httpGet);
        } catch (Exception e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        if (httpResponse.getStatusLine().getStatusCode() == 200) {
            //200说明请求和响应都是成功的
            HttpEntity entity = httpResponse.getEntity();
            try {
                response = EntityUtils.toString(entity,"utf-8");
            } catch (Exception e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }
        return response;
    }
```

返回的JSON数据在response中。然后，对返回的JSON进行解析。

```java
Gson gson = new Gson();
HomeNewsBean homeNewsBean = gson.fromJson(response, HomeNewsBean.class);
```

两句话完成解析。
我把解析过程写成了一个函数

```java
public static List<HomeNewsContentList> parseJsonWithGson2(List<HomeNewsContentList> newsLists, String jsonData) {
        Gson gson = new Gson();
        HomeNewsBean homeNewsBean = gson.fromJson(jsonData, HomeNewsBean.class);
        for (int i = 0; i < homeNewsBean.getShowapi_res_body().getPagebean().getContentlist().size(); i++) {
            newsLists.add(homeNewsBean.getShowapi_res_body().getPagebean().getContentlist().get(i));
        }
        return newsLists;
    }
```

这样JSON数据就存在了List中。到此，解析结束。想要拿出数据只需调用newsLists.get()就可以了。

该文的JSON实例应该算比较复杂的JSON数据了，如果能掌握文中的复杂JSON解析，那么其他的复杂JSON数据解析应该没有问题
