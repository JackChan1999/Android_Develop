抓取某新闻客户端的数据

## 注意事项

- 在抓取数据之前先清除缓存，缓存会影响抓包

## 过滤get请求

![](http://upload-images.jianshu.io/upload_images/3981391-53b9a44d8a6d2519.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过ping命令获取新闻数据的目标IP地址，封包信息中网址带有163可能就是新闻客户端的url

![](http://upload-images.jianshu.io/upload_images/3981391-f8ddb3b4df6262c5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

拿到目标IP地址，就可以通过目标IP地址过滤数据

![](http://upload-images.jianshu.io/upload_images/3981391-874c2b17823c9686.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

用浏览器打开抓到的url，即可得到新闻的数据

http://c.m.163.com/nc/article/list/T1467284926140/0-20.html

![](http://upload-images.jianshu.io/upload_images/3981391-6a3da6d3563f9f43.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/3981391-0e1a7a10b0f3103c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 新闻分类id

url组织形式http://c.m.163.com/nc/article/list/id/startindex-count.html

- 体育T1348649079062
- 头条T1467284926140
- 娱乐T1348648517839
- 要闻T1348647909107

## 新闻Tab标签

http://c.m.163.com/nc/topicset/android/subscribe/manage/listspecial.html

![](http://upload-images.jianshu.io/upload_images/3981391-e7b266d455078420.jpg)

![](http://upload-images.jianshu.io/upload_images/3981391-827228e461396936.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 新闻详情页

http://c.m.163.com/nc/article/docid/full.html

例如：http://c.m.163.com/nc/article/CG4A99320001899O/full.html

其中CG4A99320001899O是docid，如图所示

![docid](http://upload-images.jianshu.io/upload_images/3981391-a06e438905993f1b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



![](http://upload-images.jianshu.io/upload_images/3981391-77301aa2c9cf9e4f.png)

![](http://upload-images.jianshu.io/upload_images/3981391-3fdd29f53d2e4e35.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/3981391-2740befd744c6cec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
