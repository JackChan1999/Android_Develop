##Android安全加密专题文章索引
 
1. [Android安全加密：对称加密](http://blog.csdn.net/axi295309066/article/details/52491077)
2. [Android安全加密：非对称加密](http://blog.csdn.net/axi295309066/article/details/52494640)
3. [Android安全加密：消息摘要Message Digest](http://blog.csdn.net/axi295309066/article/details/52494725)
4. [Android安全加密：数字签名和数字证书](http://blog.csdn.net/axi295309066/article/details/52494832)
5. [Android安全加密：Https编程](http://blog.csdn.net/axi295309066/article/details/52494902)

##**1. 介绍**
与对称加密算法不同，非对称加密算法需要两个密钥：公钥（publickey）和私钥（privatekey）。公钥与私钥是一对，如果用公钥对数据进行加密，只有用对应的私钥才能解密；如果用私钥对数据进行加密，那么只有用对应的公钥才能解密。因为加密和解密使用的是两个不同的密钥，所以这种算法叫作非对称加密算法。

简单理解为：加密和解密是不同的钥匙

![这里写图片描述](http://img.blog.csdn.net/20160910133438466)

##**2. 常见算法**
RSA、Elgamal、背包算法、Rabin、D-H、ECC（椭圆曲线加密算法）等，其中支付宝使用的就是RSA算法

##**3. RSA 算法原理**
质因数、欧拉函数、模反元素
原理很复杂，只需要知道内部是基于分解质因数和取模操作即可

##**4. 使用步骤**

```java
//1，获取cipher 对象
Cipher cipher = Cipher.getInstance("RSA");
//2，通过秘钥对生成器KeyPairGenerator 生成公钥和私钥
KeyPair keyPair = KeyPairGenerator.getInstance("RSA").generateKeyPair();
//使用公钥进行加密，私钥进行解密（也可以反过来使用）
PublicKey publicKey = keyPair.getPublic();
PrivateKey privateKey = keyPair.getPrivate();
//3,使用公钥初始化密码器
cipher.init(Cipher.ENCRYPT_MODE, publicKey);
//4，执行加密操作
byte[] result = cipher.doFinal(content.getBytes());
//使用私钥初始化密码器
cipher.init(Cipher.DECRYPT_MODE, privateKey);
//执行解密操作
byte[] deResult = cipher.doFinal(result);
```
##**5. 注意点**

```java
//一次性加密数据的长度不能大于117 字节
private static final int ENCRYPT_BLOCK_MAX = 117;
//一次性解密的数据长度不能大于128 字节
private static final int DECRYPT_BLOCK_MAX = 128;
```
##**6. 分批操作**

```java
/**
* 分批操作
*
* @param content 需要处理的数据
* @param cipher 密码器（根据cipher 的不同，操作可能是加密或解密）
* @param blockSize 每次操作的块大小，单位为字节
* @return 返回处理完成后的结果
* @throws Exception
*/
 public static byte[] doFinalWithBatch(byte[] content, Cipher cipher, int blockSize) throwseption {
 int offset = 0;//操作的起始偏移位置
 int len = content.length;//数据总长度
 byte[] tmp;//临时保存操作结果
 ByteArrayOutputStream baos = new ByteArrayOutputStream();
 //如果剩下数据
 while (len - offset > 0) {
	 if (len - offset >= blockSize) {
		//剩下数据还大于等于一个blockSize
		tmp = cipher.doFinal(content, offset, blockSize);
	 }else {
		 //剩下数据不足一个blockSize
		 tmp = cipher.doFinal(content, offset, len - offset);
		 }
	 //将临时结果保存到内存缓冲区里
	 baos.write(tmp);
	 offset = offset + blockSize;
	 }
	 baos.close();
	 return baos.toByteArray();
 }
```
##**7. 非对称加密用途**
###**身份认证**
一条加密信息若能用A 的公钥能解开，则该信息一定是用A 的私钥加密的，该能确定该用户是A。

![这里写图片描述](http://img.blog.csdn.net/20160910134305813)

###**陌生人通信**
A 和B 两个人互不认识，A 把自己的公钥发给B，B 也把自己的公钥发给A，则双方可以通过对方的公钥加密信息通信。C 虽然也能得到A、B 的公钥，但是他解不开密文。

![这里写图片描述](http://img.blog.csdn.net/20160910134435438)

###**秘钥交换**
A 先得到B 的公钥，然后A 生成一个随机秘钥，例如13245768，之后A 用B 的公钥加密该秘钥，得到加密后的秘钥，例如dxs#fd@dk，之后将该密文发给B，B 用自己的私钥解密得到123456，之后双方使用13245768 作为对称加密的秘钥通信。C 就算截获加密后的秘钥dxs#fd@dk，自己也解不开，这样A、B 二人能通过对称加密进行通信。

![这里写图片描述](http://img.blog.csdn.net/20160910134559638)

##**8. 总结**
非对称加密一般不会单独拿来使用，他并不是为了取代对称加密而出现的，非对称加密速度比对称加密慢很多，极端情况下会慢1000 倍，所以一般不会用来加密大量数据，通常我们经常会将对称加密和非对称加密两种技术联合起来使用，例如用非对称加密来给称加密里的秘钥进行加密（即秘钥交换）。