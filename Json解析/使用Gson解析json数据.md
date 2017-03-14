# Gson解析系列阅读
- [Gson使用指南](http://blog.csdn.net/axi295309066/article/details/53339918)

- [使用Gson解析复杂的JSON数据](http://blog.csdn.net/axi295309066/article/details/53334086)

- [使用Gson解析json数据](http://blog.csdn.net/axi295309066/article/details/53335014)

- [Gson全解析](http://www.jianshu.com/p/fc5c9cdf3aab)

- [搞定Gson泛型封装](http://www.jianshu.com/p/d62c2be60617)

- [简单新闻客户端](http://blog.csdn.net/axi295309066/article/details/53349708)

# **1. json**
Json 全称 JavaScript Object Natation ，用来描述数据结构，它是基于纯文本的数据格式，是一种轻量级的数据交换格式。广泛应用于服务端与客户端的数据交互。

# **2. 格式**
- Json 以key－value的形式存储数据
- Key的取值为String类型
- Value的取值为String，boolean，Number，数组，Object，null
- Json串以{ 开始，以 } 结尾
- Json 串中数组是以 [ 开始，以 ] 结尾
- Json串中Object是以 { 开始，以 } 结尾



# **3. 原生解析**
```java
JSONObject root = new JSONObject(response);
JSONArray jsonArray = root.getJSONArray("appinfo");
JSONObject jsonObject = jsonArray.getJSONObject(0);
String des = jsonObject.getString("des");
jsonObject.optString()
```

```json
[
    {
        "infos": [
            {
                "name1": "休闲",
                "name2": "棋牌",
                "name3": "益智",
                "url1": "image/category_game_0.jpg",
                "url2": "image/category_game_1.jpg",
                "url3": "image/category_game_2.jpg"
            },
            {
                "name1": "射击",
                "name2": "体育",
                "name3": "儿童",
                "url1": "image/category_game_3.jpg",
                "url2": "image/category_game_4.jpg",
                "url3": "image/category_game_5.jpg"
            },
            {
                "name1": "网游",
                "name2": "角色",
                "name3": "策略",
                "url1": "image/category_game_6.jpg",
                "url2": "image/category_game_7.jpg",
                "url3": "image/category_game_8.jpg"
            },
            {
                "name1": "经营",
                "name2": "竞速",
                "name3": "",
                "url1": "image/category_game_9.jpg",
                "url2": "image/category_game_10.jpg",
                "url3": ""
            }
        ],
        "title": "游戏"
    },
    {
        "infos": [
            {
                "name1": "浏览器",
                "name2": "输入法",
                "name3": "健康",
                "url1": "image/category_app_0.jpg",
                "url2": "image/category_app_1.jpg",
                "url3": "image/category_app_2.jpg"
            },
            {
                "name1": "效率",
                "name2": "教育",
                "name3": "理财",
                "url1": "image/category_app_3.jpg",
                "url2": "image/category_app_4.jpg",
                "url3": "image/category_app_5.jpg"
            },
            {
                "name1": "阅读",
                "name2": "个性化",
                "name3": "购物",
                "url1": "image/category_app_6.jpg",
                "url2": "image/category_app_7.jpg",
                "url3": "image/category_app_8.jpg"
            },
            {
                "name1": "资讯",
                "name2": "生活",
                "name3": "工具",
                "url1": "image/category_app_9.jpg",
                "url2": "image/category_app_10.jpg",
                "url3": "image/category_app_11.jpg"
            },
            {
                "name1": "出行",
                "name2": "通讯",
                "name3": "拍照",
                "url1": "image/category_app_12.jpg",
                "url2": "image/category_app_13.jpg",
                "url3": "image/category_app_14.jpg"
            },
            {
                "name1": "社交",
                "name2": "影音",
                "name3": "安全",
                "url1": "image/category_app_15.jpg",
                "url2": "image/category_app_16.jpg",
                "url3": "image/category_app_17.jpg"
            }
        ],
        "title": "应用"
    }
]
```
![json](http://img.blog.csdn.net/20161125165710548)

```java
public class CategoryInfoBean {
    public String  title;    // title
    public String  name1;    // 休闲
    public String  name2;    // 棋牌
    public String  name3;    // 益智
    public String  url1;     // image/category_game_0.jpg
    public String  url2;     // image/category_game_1.jpg
    public String  url3;     // image/category_game_2.jpg
    public boolean isTitle;   // 自己添加的一个属性,确定是否是title
}
```
```java
List<CategoryInfoBean> categoryInfoBeans = new ArrayList<CategoryInfoBean>();
try {
    JSONArray rootJsonArray = new JSONArray(jsonString);
    // 遍历jsonArray
    for (int i = 0; i < rootJsonArray.length(); i++) {
        JSONObject itemJsonObject = rootJsonArray.getJSONObject(i);
        String title = itemJsonObject.getString("title");
        CategoryInfoBean titleBean = new CategoryInfoBean();
        titleBean.title = title;
        titleBean.isTitle = true;
        categoryInfoBeans.add(titleBean);
        JSONArray infosJsonArray = itemJsonObject.getJSONArray("infos");
        // 遍历infosJsonArray
        for (int j = 0; j < infosJsonArray.length(); j++) {
            JSONObject infoJsonObject = infosJsonArray.getJSONObject(j);
            String name1 = infoJsonObject.getString("name1");
            String name2 = infoJsonObject.getString("name2");
            String name3 = infoJsonObject.getString("name3");
            String url1 = infoJsonObject.getString("url1");
            String url2 = infoJsonObject.getString("url2");
            String url3 = infoJsonObject.getString("url3");

            CategoryInfoBean infoBean = new CategoryInfoBean();
            infoBean.name1 = name1;
            infoBean.name2 = name2;
            infoBean.name3 = name3;
            infoBean.url1 = url1;
            infoBean.url2 = url2;
            infoBean.url3 = url3;
            categoryInfoBeans.add(infoBean);
        }
    }

```

```java
public List<CategoryInfo> parseData(String result) {
        if (!TextUtils.isEmpty(result)) {
            try {
                categoryInfoList = new ArrayList<CategoryInfo>();
                JSONArray jsonArray=new JSONArray(result);

                for (int i = 0; i < jsonArray.length(); i++) {
                    JSONObject jsonObject=jsonArray.getJSONObject(i);
                    CategoryInfo categoryInfo=new CategoryInfo();
                    if (jsonObject.has("title")) {
                        categoryInfo.setTitle(jsonObject.getString("title"));
                        categoryInfo.setTitle(true);
                        categoryInfoList.add(categoryInfo);
                    }
                    if (jsonObject.has("info")) {
                        categoryInfo.setName1(jsonObject.getString("name1"));
                        categoryInfo.setName2(jsonObject.getString("name2"));
                        categoryInfo.setName3(jsonObject.getString("name3"));
                        categoryInfo.setUrl1(jsonObject.getString("url1"));
                        categoryInfo.setUrl1(jsonObject.getString("url2"));
                        categoryInfo.setUrl1(jsonObject.getString("url3"));
                        categoryInfo.setTitle(false);
                        categoryInfoList.add(categoryInfo);
                    }
                }
                return categoryInfoList;
            } catch (JSONException e) {
                e.printStackTrace();
            }
        }
        return null;
    }
```
![json](http://img.blog.csdn.net/20161125165817637)

# **4. Gson格式化**
将 java 对象 格式化为 Json 字符串.

实现步骤 :

获得需要的对象:

```java
Student stu = new Student();
stu.setName("张三");
stu.setAge(18);
stu.setSex(true);
```

格式化

```java
Gson gson = new Gson();
//将 对象 转化成 json 字符串
String json = gson.toJson(stu)
```

# **5. Gson解析**
遇到[ ]就解析成集合，遇到{ }就解析成对象，遇到""就解析成一个对应的基本数据类型

```json
{
    "data": [
        {
            "children": [
                {
                    "id": 10007,
                    "title": "北京",
                    "type": 1,
                    "url": "/10007/list_1.json"
                },
                {
                    "id": 10006,
                    "title": "中国",
                    "type": 1,
                    "url": "/10006/list_1.json"
                },
                {
                    "id": 10008,
                    "title": "国际",
                    "type": 1,
                    "url": "/10008/list_1.json"
                },
                {
                    "id": 10010,
                    "title": "体育",
                    "type": 1,
                    "url": "/10010/list_1.json"
                },
                {
                    "id": 10091,
                    "title": "生活",
                    "type": 1,
                    "url": "/10091/list_1.json"
                },
                {
                    "id": 10012,
                    "title": "旅游",
                    "type": 1,
                    "url": "/10012/list_1.json"
                },
                {
                    "id": 10095,
                    "title": "科技",
                    "type": 1,
                    "url": "/10095/list_1.json"
                },
                {
                    "id": 10009,
                    "title": "军事",
                    "type": 1,
                    "url": "/10009/list_1.json"
                },
                {
                    "id": 10093,
                    "title": "时尚",
                    "type": 1,
                    "url": "/10093/list_1.json"
                },
                {
                    "id": 10011,
                    "title": "财经",
                    "type": 1,
                    "url": "/10011/list_1.json"
                },
                {
                    "id": 10094,
                    "title": "育儿",
                    "type": 1,
                    "url": "/10094/list_1.json"
                },
                {
                    "id": 10105,
                    "title": "汽车",
                    "type": 1,
                    "url": "/10105/list_1.json"
                }
            ],
            "id": 10000,
            "title": "新闻",
            "type": 1
        },
        {
            "id": 10002,
            "title": "专题",
            "type": 10,
            "url": "/10006/list_1.json",
            "url1": "/10007/list1_1.json"
        },
        {
            "id": 10003,
            "title": "组图",
            "type": 2,
            "url": "/10008/list_1.json"
        },
        {
            "dayurl": "",
            "excurl": "",
            "id": 10004,
            "title": "互动",
            "type": 3,
            "weekurl": ""
        }
    ],
    "extend": [
        10007,
        10006,
        10008,
        10014,
        10012,
        10091,
        10009,
        10010,
        10095
    ],
    "retcode": 200
}
```
![json](http://img.blog.csdn.net/20161125191206197)

```java
/**
 * 网络分类信息的封装
 * 
 * 字段名字必须和服务器返回的字段名一致, 方便gson解析
 * 
 */
public class NewsData {

	public int retcode;
	public ArrayList<NewsMenuData> data;

	// 侧边栏数据对象
	public class NewsMenuData {
		public String id;
		public String title;
		public int type;
		public String url;

		public ArrayList<NewsTabData> children;

		@Override
		public String toString() {
			return "NewsMenuData [title=" + title + ", children=" + children
					+ "]";
		}
	}

	// 新闻页面下11个子页签的数据对象
	public class NewsTabData {
		public String id;
		public String title;
		public int type;
		public String url;

		@Override
		public String toString() {
			return "NewsTabData [title=" + title + "]";
		}
	}

	@Override
	public String toString() {
		return "NewsData [data=" + data + "]";
	}

}
```

```java
NewsData mNewsData Gson gson = new Gson();
mNewsData = gson.fromJson(result, NewsData.class);
```
解析结果

![gson](http://img.blog.csdn.net/20161125230839752)
## **5.1 标准解析**
```java
public class GsonTools {
	public GsonTools() {
	}

	public static String createGsonString(Object object) {
		Gson gson = new Gson();
		String gsonString = gson.toJson(object);
		return gsonString;
	}

	public static <T> T changeGsonToBean(String gsonString, Class<T> cls) {
		Gson gson = new Gson();
		T t = gson.fromJson(gsonString, cls);
		return t;
	}

	public static <T> T changeGsonToBean(String gsonString, Type type) {
		Gson gson = new Gson();
		T t = gson.fromJson(gsonString,type);
		return t;
	}

	public static <T> List<T> changeGsonToList(String gsonString, Class<T> cls) {
		Gson gson = new Gson();
		List<T> list = gson.fromJson(gsonString, new TypeToken<List<T>>() {
		}.getType());
		return list;
	}

	public static <T> List<Map<String, T>> changeGsonToListMaps(String gsonString) {
		List<Map<String, T>> list = null;
		Gson gson = new Gson();
		list = gson.fromJson(gsonString, new TypeToken<List<Map<String, T>>>(){}.getType());
		return list;
	}

	public static <T> Map<String, T> changeGsonToMaps(String gsonString) {
		Map<String, T> map = null;
		Gson gson = new Gson();
		map = gson.fromJson(gsonString, new TypeToken<Map<String, T>>(){}.getType());
		return map;
	}
}
```

Json的解析成 java 对象

```java
Gson gson = new Gson();

// 将json 转化成 java 对象
Student stu = gson.fromJson(json, Student.class);

// 将 json 转化 成 List泛型
List<Student> stus = gson.fromJson(json, new TypeToken<List<Student>>() {}.getType());

// 将 json 转化 成 Map泛型 
Map<String,String> map = gson.fromJson(json, new TypeToken<Map<String,String>>() {}.getType());
```
## **5.2 泛型解析**

```json
[
    {
        "des": "2005-2014 你的校园一直在这儿。中国最大的实名制SNS网络平台，大学生",
        "downloadUrl": "app/com.renren.mobile.android/com.renren.mobile.android.apk",
        "iconUrl": "app/com.renren.mobile.android/icon.jpg",
        "id": 1580615,
        "name": "人人",
        "packageName": "com.renren.mobile.android",
        "size": 21803987,
        "stars": 2
    },
    {
        "des": "中国电信掌上营业厅是中国电信集团【官方】唯一指定服务全国电信用户的自助服务客户端",
        "downloadUrl": "app/com.ct.client/com.ct.client.apk",
        "iconUrl": "app/com.ct.client/icon.jpg",
        "id": 1540629,
        "name": "掌上营业厅",
        "packageName": "com.ct.client",
        "size": 4794202,
        "stars": 2
    }
]
```
![json](http://img.blog.csdn.net/20161125163226125)
```java
public class AppInfoBean {
	public String	des;			// 应用的描述
	public String	downloadUrl;	// 应用的下载地址
	public String	iconUrl;		// 应用的图标地址
	public long		id;			    // 应用的id
	public String	name;			// 应用的名字
	public String	packageName;	// 应用的包名
	public long		size;			// 应用的长度
	public float	stars;			// 应用的评分
}
```

```java
public List<AppInfoBean> parseJson(String jsonString) {
		Gson gson = new Gson();

		/*=============== 泛型解析 ===============*/
		return gson.fromJson(jsonString, new TypeToken<List<AppInfoBean>>() {
		}.getType());

	}
```

```java
Type type = new TypeToken<ArrayList<String>>() {}.getType();  
```

```java
Gson gson = new Gson();
List<HomeBean> list = gson.fromJson(result,new TypeToken<List< HomeBean>>(){}.getType());
```

```java
Map<String,String> map = gson.fromJson(response,new TypeToken<Map<String,String>>(){}.getType());
```

```java
public abstract class BaseProtocol<T> {
    protected abstract String getInterfaceKey();
    protected T parseJson(String result){
        ParameterizedType genericSuperclass = (ParameterizedType) getClass().getGenericSuperclass();
        Type[] types = genericSuperclass.getActualTypeArguments();
        Type type = types[0];
        return new Gson().fromJson(result,type);
    }
}
```
## **5.3 节点解析**
适合属性比较多的情况，要啥取啥

![jsonelement](http://img.blog.csdn.net/20161125160139129)

![json](http://img.blog.csdn.net/20161125164107021)

Gson的节点对象
| 节点对象          | 说明                                    |
| :------------ | :------------------------------------ |
| JsonElement   | 所有的节点都是JsonElement对象                  |
| JsonPrimitive | 基本的数据类型的节点对象， JsonElement的子类          |
| JsonNull      | 代表空节点对象，即有key，value为空，JsonElement 的子类 |
| JsonObject    | 对象 数据类型的节点对象, JsonElement的子类          |
| JsonArray     | 数组 数据类型的节点对象, JsonElement的子类          |

## **5.4 JsonElement的取值**

JsonPrimitive : value 的 取值对应 java

int，double，float，long，short，boolean，char，byte，String，BigDecimal，BigInteger，Number

JsonObject : value 的 取值对应 java 的 Object 对象

JsonArray : value 的 取值对应 java 的 List 及其子类对象

```java
//1、获得 解析者

JsonParser parser = new JsonParser();

//2、获得 根节点元素

JsonElement root = parser.parse(json);

//3、根据 文档判断根节点属于 什么类型的 Gson节点对象

//假如文档 显示 根节点 为对象类型
// 获得 根节点 的实际 节点类型
JsonObject element = root.getAsJsonObject();

//4、取得 节点 下 的某个节点的 value

// 获得 name 节点的值，name 节点为基本数据节点
JsonPrimitive nameJson = element.getAsJsonPrimitive("name");
String name = nameJson.getAsString();

// 获得 students 节点的值, students 节点为 数组数据节点
JsonArray arrayJson = element.getAsJsonArray("students");
// 获得数据 的长度
int size = arrayJson.size();
for(int i = 0; i < size; i++) {
    //获得每一个json 元素
    JsonElement e = arrayJson.get(i);
    // 通过元素 得到需要的节点值 TODO:
}
```


```java
public List<AppInfoBean> parseJson(String jsonString) {
		Gson gson = new Gson();
		/*=============== 泛型解析 ===============*/
		// return gson.fromJson(jsonString, new TypeToken<List<AppInfoBean>>() {
		// }.getType());
		/*=============== 结点解析 ===============*/

		List<AppInfoBean> appInfoBeans = new ArrayList<AppInfoBean>();
		// 获得json的解析器
		JsonParser parser = new JsonParser();

		JsonElement rootJsonElement = parser.parse(jsonString);

		// JsonElement-->转换成jsonArray
		JsonArray rootJsonArray = rootJsonElement.getAsJsonArray();

		// 遍历jsonArray
		for (JsonElement itemJsonElement : rootJsonArray) {
			// jsonElement-->转换成JsonObject
			JsonObject itemJsonObject = itemJsonElement.getAsJsonObject();

			// 得到具体的jsonPrimitivie
			JsonPrimitive desPrimitivie = itemJsonObject.getAsJsonPrimitive("des");
			// jsonPrimitivie-->转换成具体的类型
			String des = desPrimitivie.getAsString();

			// 得到具体的jsonPrimitivie
			JsonPrimitive downloadUrlPrimitivie = itemJsonObject.getAsJsonPrimitive("downloadUrl");
			// jsonPrimitivie-->转换成具体的类型
			String downloadUrl = downloadUrlPrimitivie.getAsString();

			String iconUrl = itemJsonObject.get("iconUrl").getAsString();
			long id = itemJsonObject.get("id").getAsLong();
			String name = itemJsonObject.get("name").getAsString();
			String packageName = itemJsonObject.get("packageName").getAsString();
			long size = itemJsonObject.get("size").getAsLong();
			float stars = itemJsonObject.get("stars").getAsFloat();

			AppInfoBean info = new AppInfoBean();
			info.des = des;
			info.downloadUrl = downloadUrl;
			info.iconUrl = iconUrl;
			info.id = id;
			info.name = name;
			info.packageName = packageName;
			info.size = size;
			info.stars = stars;
			// 添加到集合
			appInfoBeans.add(info);

		}

		// 返回结果
		return appInfoBeans;
	}
```

com.google.gson.JsonSyntaxException: com.google.gson.stream.MalformedJsonException: Use JsonReader.setLenient(true) to accept malformed JSON

```java
Gson gson = new Gson();
JsonReader reader = new JsonReader(new StringReader(result1));
reader.setLenient(true);
Userinfo userinfo1 = gson.fromJson(reader, Userinfo.class);
```

```java
String trimmed = result1.trim();
gson.fromJson(trimmed, T);
```

com.google.gson.internal.LinkedTreeMap cannot be cast to com.google.smartcit

```java
    public static <T> List<T> changeGsonToList(String gsonString, Class<T> cls) {
        Gson gson = new Gson();
        List<T> list = gson.fromJson(gsonString, new TypeToken<List<T>>() {
        }.getType());
        return list;
    }
```

```java
public static <T> List<T> getObjectList(String jsonString,Class<T> cls){  
        List<T> list = new ArrayList<T>();  
        try {  
            Gson gson = new Gson();  
            JsonArray arry = new JsonParser().parse(jsonString).getAsJsonArray();  
            for (JsonElement jsonElement : arry) {  
                list.add(gson.fromJson(jsonElement, cls));  
            }  
        } catch (Exception e) {  
            e.printStackTrace();  
        }  
        return list;  
    }  

```
## 5.5 处理json格式特殊字符

```java
public static String changeStr(String json){
		json = json.replaceAll(",", "，");
		json = json.replaceAll(":", "：");
		json = json.replaceAll("\\[", "【"); 
		json = json.replaceAll("\\]", "】"); 
		json = json.replaceAll("\\{", "<"); 
		json = json.replaceAll("\\}", ">"); 
		json = json.replaceAll("\"", "”"); 
		
		return json.toString();
	}
```