##Android安全加密专题文章索引

1. [Android安全加密：对称加密](http://blog.csdn.net/axi295309066/article/details/52491077)
2. [Android安全加密：非对称加密](http://blog.csdn.net/axi295309066/article/details/52494640)
3. [Android安全加密：消息摘要Message Digest](http://blog.csdn.net/axi295309066/article/details/52494725)
4. [Android安全加密：数字签名和数字证书](http://blog.csdn.net/axi295309066/article/details/52494832)
5. [Android安全加密：Https编程](http://blog.csdn.net/axi295309066/article/details/52494902)

##**一、凯撒密码**
###**1. 概述**
凯撒密码作为一种最为古老的对称加密体制，在古罗马的时候都已经很流行，他的基本思想是：通过把字母移动一定的位数来实现加密和解密。明文中的所有字母都在字母表上向后（或向前）按照一个固定数目进行偏移后被替换成密文。例如，当偏移量是3 的时候，所有的字母A 将被替换成D，B 变成E，由此可见，位数就是凯撒密码加密和解密的密钥。

例如：字符串”ABC”的每个字符都右移3 位则变成”DEF”，解密的时候”DEF”的每个字符左移3 位即能还原，如下图所示：

![这里写图片描述](http://img.blog.csdn.net/20160909221215656)

###**2. 准备知识**

```java
 //字符转换成ASCII 码数值
 char charA = 'a';
 int intA = charA; //char 强转为int 即得到对应的ASCII 码值，’a’的值为97
       
//ASCII 码值转成char
int intA = 97;//97 对应的ASCII 码’a’
char charA = (char) intA; //int 值强转为char 即得到对应的ASCII 字符，即'a'
```
![这里写图片描述](http://img.blog.csdn.net/20160909221531760)

###**3. 凯撒密码的简单代码实现**

```java
    /**
     * 加密
     * @param input 数据源（需要加密的数据）
     * @param key 秘钥，即偏移量
     * @return 返回加密后的数据
     */
    public static String encrypt(String input, int key) {
        //得到字符串里的每一个字符
        char[] array = input.toCharArray();

        for (int i = 0; i < array.length; ++i) {
            //字符转换成ASCII 码值
            int ascii = array[i];
            //字符偏移，例如a->b
            ascii = ascii + key;
            //ASCII 码值转换为char
            char newChar = (char) ascii;
            //替换原有字符
            array[i] = newChar;

            //以上4 行代码可以简写为一行
            //array[i] = (char) (array[i] + key);
        }

        //字符数组转换成String
        return new String(array);
    }

    /**
     * 解密
     * @param input 数据源（被加密后的数据）
     * @param key 秘钥，即偏移量
     * @return 返回解密后的数据
     */
    public static String decrypt(String input, int key) {
        //得到字符串里的每一个字符
        char[] array = input.toCharArray();
        for (int i = 0; i < array.length; ++i) {
            //字符转换成ASCII 码值
            int ascii = array[i];
            //恢复字符偏移，例如b->a
            ascii = ascii - key;
            //ASCII 码值转换为char
            char newChar = (char) ascii;
            //替换原有字符
            array[i] = newChar;

            //以上4 行代码可以简写为一行
            //array[i] = (char) (array[i] - key);
        }

        //字符数组转换成String
        return new String(array);
    }
```
代码输出结果：
![这里写图片描述](http://img.blog.csdn.net/20160909222525690)

###**4. 破解凯撒密码：频率分析法**
凯撒密码加密强度太低，只需要用频度分析法即可破解。
在任何一种书面语言中，不同的字母或字母组合出现的频率各不相同。而且，对于以这种语言书写的任意一段文本，都具有大致相同的特征字母分布。比如，在英语中，字母E 出现的频率很高，而X 则出现得较少。

英语文本中典型的字母分布情况如下图所示：
![这里写图片描述](http://img.blog.csdn.net/20160909222744771)
###**5. 破解流程**

- 统计密文里出现次数最多的字符，例如出现次数最多的字符是是’h’。
- 计算字符’h’到’e’的偏移量，值为3，则表示原文偏移了3 个位置。
- 将密文所有字符恢复偏移3 个位置。

注意点：统计密文里出现次数最多的字符时，需多统计几个备选，因为最多的可能是空格或者其他字符，例如下图出现次数最多的字符’#’是空格加密后的字符，’h’才是’e’偏移后的值。
![这里写图片描述](http://img.blog.csdn.net/20160909222942539)

解密时要多几次尝试，因为不一定出现次数最多的字符就是我们想要的目标字符，如下图，第二次解密的结果才是正确的。

```java
/**
 * 频率分析法破解凯撒密码
 */
public class FrequencyAnalysis {
	//英文里出现次数最多的字符
	private static final char MAGIC_CHAR = 'e';
	//破解生成的最大文件数
	private static final int DE_MAX_FILE = 4;
	
	public static void main(String[] args) throws Exception {
		//测试1，统计字符个数
		//printCharCount("article1_en.txt");
		
		//加密文件
		//int key = 3;
		//encryptFile("article1.txt", "article1_en.txt", key);
		
		//读取加密后的文件
	    String artile = file2String("article1_en.txt");
	    //解密（会生成多个备选文件）
	    decryptCaesarCode(artile, "article1_de.txt");
	}
	
	public static void printCharCount(String path) throws IOException{
		String data = file2String(path);
		List<Entry<Character, Integer>> mapList = getMaxCountChar(data);
		for (Entry<Character, Integer> entry : mapList) {
			//输出前几位的统计信息
			System.out.println("字符'" + entry.getKey() + "'出现" + entry.getValue() + "次");
		}
	}
	
	public static void encryptFile(String srcFile, String destFile, int key) throws IOException {
		String artile = file2String(srcFile);
		//加密文件
		String encryptData = MyEncrypt.encrypt(artile, key);
		//保存加密后的文件
		string2File(encryptData, destFile);
	}
	
	/**
	 * 破解凯撒密码
	 * @param input 数据源
	 * @return 返回解密后的数据
	 */
	public static void decryptCaesarCode(String input, String destPath) {
		int deCount = 0;//当前解密生成的备选文件数
		//获取出现频率最高的字符信息（出现次数越多越靠前）
		List<Entry<Character, Integer>> mapList = getMaxCountChar(input);
		for (Entry<Character, Integer> entry : mapList) {
			//限制解密文件备选数
			if (deCount >= DE_MAX_FILE) {
				break;
			}
			
			//输出前几位的统计信息
			System.out.println("字符'" + entry.getKey() + "'出现" + entry.getValue() + "次");
			
			++deCount;
			//出现次数最高的字符跟MAGIC_CHAR的偏移量即为秘钥
			int key = entry.getKey() - MAGIC_CHAR;
			System.out.println("猜测key = " + key + "， 解密生成第" + deCount + "个备选文件" + "\n");
			String decrypt = MyEncrypt.decrypt(input, key);
			
			String fileName = "de_" + deCount + destPath;
			string2File(decrypt, fileName);
		}
	}
	
	//统计String里出现最多的字符
	public static List<Entry<Character, Integer>> getMaxCountChar(String data) {
		Map<Character, Integer> map = new HashMap<Character, Integer>();
		char[] array = data.toCharArray();
		for (char c : array) {
			if(!map.containsKey(c)) {
				map.put(c, 1);
			}else{
				Integer count = map.get(c);
				map.put(c, count + 1);
			}
		}
		
		//输出统计信息
		/*for (Entry<Character, Integer> entry : map.entrySet()) {
			System.out.println(entry.getKey() + "出现" + entry.getValue() +  "次");
		}*/
		
		//获取获取最大值
		int maxCount = 0;
		for (Entry<Character, Integer> entry : map.entrySet()) {
			//不统计空格
			if (/*entry.getKey() != ' ' && */entry.getValue() > maxCount) { 
				maxCount = entry.getValue();
			}
		}
		
		//map转换成list便于排序
		List<Entry<Character, Integer>> mapList = new ArrayList<Map.Entry<Character,Integer>>(map.entrySet());
		//根据字符出现次数排序
		Collections.sort(mapList, new Comparator<Entry<Character, Integer>>(){
			@Override
			public int compare(Entry<Character, Integer> o1,
					Entry<Character, Integer> o2) {
				return o2.getValue().compareTo(o1.getValue());
			}
		});
		return mapList;
	}
	
	public static String file2String(String path) throws IOException {
		FileReader reader = new FileReader(new File(path));
		char[] buffer = new char[1024];
		int len = -1;
		StringBuffer sb = new StringBuffer();
		while ((len = reader.read(buffer)) != -1) {
			sb.append(buffer, 0, len);
		}
		return sb.toString();
	}
	
	public static void string2File(String data, String path){
		FileWriter writer = null;
		try {
			writer = new FileWriter(new File(path));
			writer.write(data);
		} catch (Exception e) {
			e.printStackTrace();
		}finally {
			if (writer != null) {
				try {
					writer.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
		}

	}
}
```

![这里写图片描述](http://img.blog.csdn.net/20160909223030000)

##**二、对称加密**

###**1、概述**
加密和解密都使用同一把秘钥，这种加密方法称为对称加密，也称为单密钥加密。
简单理解为：加密解密都是同一把钥匙。
凯撒密码就属于对称加密，他的字符偏移量即为秘钥。
###**2、对称加密常用算法**
AES、DES、3DES、TDEA、Blowfish、RC2、RC4、RC5、IDEA、SKIPJACK 等。

- **DES**

全称为Data Encryption Standard，即数据加密标准，是一种使用密钥加密的块算法，1976 年被美国联邦政府的国家标准局确定为联邦资料处理标准（FIPS），随后在国际上广泛流传开来。

- **3DES**

也叫Triple DES，是三重数据加密算法（TDEA，Triple Data Encryption Algorithm）块密码的通称。
它相当于是对每个数据块应用三次DES 加密算法。由于计算机运算能力的增强，原版DES 密码的密钥长度变得容易被暴力破解；3DES 即是设计用来提供一种相对简单的方法，即通过增加DES 的密钥长度来避免类似的攻击，而不是设计一种全新的块密码算法。

- **AES**

高级加密标准（英语：Advanced Encryption Standard，缩写：AES），在密码学中又称Rijndael 加密法，是美国联邦政府采用的一种区块加密标准。这个标准用来替代原先的DES，已经被多方分析且广为全世界所使用。经过五年的甄选流程，高级加密标准由美国国家标准与技术研究院（NIST）于2001 年11 月26 日发布于FIPS PUB 197，并在2002 年5 月26 日成为有效的标准。2006 年，高级加密标准已然成为对称密钥加密中最流行的算法之一。

###**3、DES 算法简介**
DES 加密原理（对比特位进行操作，交换位置，异或等等，无需详细了解）
###**准备知识**
Bit 是计算机最小的传输单位。以0 或1 来表示比特位的值
例如数字3 对应的二进制数据为：00000011

代码示例

```java
 int i = 97;
 String bit = Integer.toBinaryString(i);
 //输出：97 对应的二进制数据为： 1100001
 System.out.println(i + "对应的二进制数据为： " + bit);
```
###**Byte 与Bit 区别**
数据存储是以“字节”（Byte）为单位，数据传输是大多是以“位”（bit，又名“比特”）为单位，一个位就代表一个0 或1（即二进制），每8 个位（bit，简写为b）组成一个字节（Byte，简写为B），是最小一级的信息单位。

Byte 的取值范围：

```java
//byte 的取值范围：-128 到127
System.out.println(Byte.MIN_VALUE + "到" + Byte.MAX_VALUE);
```
即10000000 到01111111 之间，一个字节占8 个比特位

**二进制转十进制图示：**
![这里写图片描述](http://img.blog.csdn.net/20160909223926441)

**任何字符串都可以转换为字节数组**

```java
String data = "1234abcd";
byte[] bytes = data.getBytes();//内容为：49 50 51 52 97 98 99 100
```
上面数据49 50 51 52 97 98 99 100 对应的二进制数据（即比特位为）：
00110001
00110010
00110011
00110100
01100001
01100010
01100011
01100100

将他们间距调大一点，可看做一个矩阵：
![这里写图片描述](http://img.blog.csdn.net/20160910151407479)

之后可对他们进行各种操作，例如交换位置、分割、异或运算等，常见的加密方式就是这样操作比特位的，例如下图的**IP 置换**以及**S-Box** 操作都是常见加密的一些方式：

**IP 置换：**
![IP 置换：](http://img.blog.csdn.net/20160910121722180)

**S-BOX 置换：**
![这里写图片描述](http://img.blog.csdn.net/20160910121838848)

**DES 加密过程图解（流程很复杂，只需要知道内部是操作比特位即可）：**

![这里写图片描述](http://img.blog.csdn.net/20160910121924553)

###**对称加密应用场景**

- 本地数据加密（例如加密android 里SharedPreferences 里面的某些敏感数据）
- 网络传输：登录接口post 请求参数加密{username=lisi,pwd=oJYa4i9VASRoxVLh75wPCg==}
- 加密用户登录结果信息并序列化到本地磁盘(将user 对象序列化到本地磁盘，下次登录时反序列化到内存里)
- 网页交互数据加密（即后面学到的Https）

###**DES 算法代码实现**

```java
 //1,得到cipher 对象（可翻译为密码器或密码系统）
 Cipher cipher = Cipher.getInstance("DES");
 //2，创建秘钥
 SecretKey key = KeyGenerator.getInstance("DES").generateKey();
 //3，设置操作模式（加密/解密）
 cipher.init(Cipher.ENCRYPT_MODE, key);
 //4，执行操作
 byte[] result = cipher.doFinal("黑马".getBytes());
```
###**AES 算法代码实现**
用法同上，只需把”DES”参数换成”AES”即可。

###**使用Base64 编码加密后的结果**

```java
byte[] result = cipher.doFinal("黑马".getBytes());
System.out.println(new String(result));
```
输出结果：

![这里写图片描述](http://img.blog.csdn.net/20160910122431669)

加密后的结果是字节数组，这些被加密后的字节在码表（例如UTF-8 码表）上找不到对应字符，会出现乱码，当乱码字符串再次转换为字节数组时，长度会变化，导致解密失败，所以转换后的数据是不安全的。

使用Base64 对字节数组进行编码，任何字节都能映射成对应的Base64 字符，之后能恢复到字节数组，利于加密后数据的保存于传输，所以转换是安全的。同样，字节数组转换成16 进制字符串也是安全的。

密文转换成Base64 编码后的输出结果：
![这里写图片描述](http://img.blog.csdn.net/20160910152340810)

密文转换成16 进制编码后的输出结果：
![这里写图片描述](http://img.blog.csdn.net/20160910122649470)

Java 里没有直接提供Base64 以及字节数组转16 进制的Api，开发中一般是自己手写或直接使用第三方提供的成熟稳定的工具类（例如apache 的commons-codec）。

**Base64 字符映射表**
![Base64 字符映射表](http://img.blog.csdn.net/20160910122745327)

###**对称加密的具体应用方式**

1、生成秘钥并保存到硬盘上，以后读取该秘钥进行加密解密操作，实际开发中用得比较少

```java
//生成随机秘钥
SecretKey secretKey = KeyGenerator.getInstance("AES").generateKey();
//序列化秘钥到磁盘上
FileOutputStream fos = new FileOutputStream(new File("heima.key"));

ObjectOutputStream oos = new ObjectOutputStream(fos);
oos.writeObject(secretKey);

//从磁盘里读取秘钥
FileInputStream fis = new FileInputStream(new File("heima.key"));
ObjectInputStream ois = new ObjectInputStream(fis);
Key key = (Key) ois.readObject();
```
2、使用自定义秘钥（秘钥写在代码里）

```java
//创建密钥写法1
KeySpec keySpec = new DESKeySpec(key.getBytes());
SecretKey secretKey = SecretKeyFactory.getInstance(ALGORITHM).
generateSecret(keySpec);

//创建密钥写法2
//SecretKey secretKey = new SecretKeySpec(key.getBytes(), KEY_ALGORITHM);

Cipher cipher = Cipher.getInstance(CIPHER_ALGORITHM);
cipher.init(Cipher.DECRYPT_MODE, secretKey);
//得到key 后，后续代码就是Cipher 的写法，此处省略...
```
###**注意事项**
把秘钥写在代码里有一定风险，当别人反编译代码的时候，可能会看到秘钥，android 开发里建议用JNI 把秘钥值写到C 代码里，甚至拆分成几份，最后再组合成真正的秘钥

###**算法/工作模式/填充模式**

初始化cipher 对象时，参数可以直接传算法名：例如：

```java
Cipher c = Cipher.getInstance("DES");
```

也可以指定更详细的参数，格式："algorithm/mode/padding" ，即"算法/工作模式/填充模式"

```java
Cipher c = Cipher.getInstance("DES/CBC/PKCS5Padding");
```

###**密码块工作模式**
块密码工作模式(Block cipher mode of operation)，是对于按块处理密码的加密方式的一种扩充，不仅仅适用于AES，包括DES, RSA 等加密方法同样适用。

![这里写图片描述](http://img.blog.csdn.net/20160910132118010)

###**填充模式**
填充(Padding)，是对需要按块处理的数据，当数据长度不符合块处理需求时，按照一定方法填充满块长的一种规则。

![这里写图片描述](http://img.blog.csdn.net/20160910132725165)

###具体代码：

```java
//秘钥算法
private static final String KEY_ALGORITHM = "DES";
//加密算法：algorithm/mode/padding 算法/工作模式/填充模式
private static final String CIPHER_ALGORITHM = "DES/ECB/PKCS5Padding";
//秘钥
private static final String KEY = "12345678";//DES 秘钥长度必须是8 位或以上
//private static final String KEY = "1234567890123456";//AES 秘钥长度必须是16 位

//初始化秘钥
SecretKey secretKey = new SecretKeySpec(KEY.getBytes(), KEY_ALGORITHM);
Cipher cipher = Cipher.getInstance(CIPHER_ALGORITHM);

//加密
cipher.init(Cipher.ENCRYPT_MODE, secretKey);
byte[] result = cipher.doFinal(input.getBytes());
```
###注意：AES、DES 在CBC 操作模式下需要iv 参数

```java
//AES、DES 在CBC 操作模式下需要iv 参数
IvParameterSpec iv = new IvParameterSpec(key.getBytes());

//加密
cipher.init(Cipher.ENCRYPT_MODE, secretKey, iv);
```

##三、总结
DES 安全度在现代已经不够高，后来又出现的3DES 算法强度提高了很多，但是其执行效率低下，AES算法加密强度大，执行效率高，使用简单，实际开发中建议选择AES 算法。实际android 开发中可以用对称加密（例如选择AES 算法）来解决很多问题，例如：

- 做一个管理密码的app，我们在不同的网站里使用不同账号密码，很难记住，想做个app 统一管理，但是账号密码保存在手机里，一旦丢失了容易造成安全隐患，所以需要一种加密算法，将账号密码信息加密起来保管，这时候如果使用对称加密算法，将数据进行加密，秘钥我们自己记在心里，只需要记住一个密码。需要的时候可以还原信息。
- android 里需要把一些敏感数据保存到SharedPrefrence 里的时候，也可以使用对称加密，这样可以在需要的时候还原。
- 请求网络接口的时候，我们需要上传一些敏感数据，同样也可以使用对称加密，服务端使用同样的算法就可以解密。或者服务端需要给客户端传递数据，同样也可以先加密，然后客户端使用同样算法解密。