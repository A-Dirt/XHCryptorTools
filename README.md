# XHCryptorTools

原文:http://www.jianshu.com/p/86f9a1ef3f24

框架从 CryptoExercise（苹果3.0时的包）进行提取扩展
iOS 系统自带相关函数说明，框架主要使用前两种：
SecKeyEncrypt 使用公钥对数据加密
SecKeyDecrypt 使用私钥对数据解密
SecKeyRawVerify 使用公钥对数字签名进行验证
SecKeyRawSign 使用私钥生成数字签名

普遍的加密方法：客户端用RSA的公钥加密AES的秘钥，服务器端用私钥解开获得的AES的秘钥，客户端再与服务器端进行AES加密的数据传输，即HTTPS协议传输的原理

加密解密概念

对称加密算法：加密解密都使用相同的秘钥，速度快，适合对大数据加密，方法有DES，3DES，AES等

非对称加密算法
非对称加密算法需要两个密钥：公开密钥（publickey）和私有密钥（privatekey）
公开密钥与私有密钥是一对，可逆的加密算法，用公钥加密，用私钥解密，用私钥加密，用公钥解密，速度慢，适合对小数据加密，方法有RSA

散列算法（加密后不能解密，上面都是可以解密的）
用于密码的密文存储，服务器端是判断加密后的数据
不可逆加密方法：MD5、SHA1、SHA256、SHA512

RSA算法原理：

1.找出两个“很大”的质数：P & Q（上百位）
	N = P Q
	M = (P – 1) (Q – 1)
2.找出整数E，E与M互质，即除了1之外，没有其他公约数
3.找出整数D，使得 ED 除以 M 余 1，即 (E * D) % M = 1
4.经过上述准备工作之后，可以得到：E是公钥，负责加密D是私钥，负责解密N负责公钥和私钥之间的联系
5.加密算法，假定对X进行加密(X ^ E) % N = Y（6）解密算法，根据费尔马小定义，可以使用以下公式完成解密(Y ^ D) % N = X

使用方法

XHCryptorTools *tools = [[XHCryptorTools alloc] init];

// 1. 加载公钥
NSString *pubPath = [[NSBundle mainBundle] pathForResource:@"rsacert.der" ofType:nil];
[tools loadPublicKeyWithFilePath:pubPath];

// 2. 使用公钥加密，加密内容最大长度 117
NSString *result = [tools RSAEncryptString:@"abcdefghijklmnopqrstuvwxyz1234567890abcdefghijklmnopqrstuvwxyz1234567890abcdefghijklmnopqrstuvwxyz1234567890abcdefghi"];
NSLog(@"RSA 加密 %@", result);

// 3. 加载私钥，并指定导出 p12 时设置的密码
NSString *privatePath = [[NSBundle mainBundle] pathForResource:@"p.p12" ofType:nil];
[tools loadPrivateKey:privatePath password:@"123"];
// 4. 使用私钥解密
NSLog(@"解密结果 %@", [tools RSADecryptString:result]);


公钥、私钥生成

公钥：就是签名机构签完给我们颁发的，放在网站的根目录上，可以分发
私钥：一般保存在中心服务器
加密解密使用了两种文件 .p12是私钥 .der是公钥，终端命令生成步骤如下：

1.创建私钥，生成安全强度是512（也可以是1024）的RAS私钥，.pem是base64的证书文件
openssl genrsa -out private.pem 512
2.生成一个证书请求，生成证书请求文件.csr
openssl req -new -key private.pem -out rsacert.csr

终端提示如下：

国家名字、代码
省的名字
城市的名字
公司的名字
公司的单位
我的名字
电子邮件
以及两个附加信息可以跳过

生成证书请求界面.png
3.签名，找证书颁发机构签名，证明证书合法有效的，也可以自签名一个证书
生成证书并签名，有效期10年，生成一个.crt的一个base64公钥文件
openssl x509 -req -days 3650 -in rsacert.csr -signkey private.pem -out rsacert.crt
由于iOS开发时使用的时候不能是base64的，必须解成二进制文件！

4.解成.der公钥二进制文件，放程序做加密用
openssl x509 -outform der -in rsacert.crt -out rsacert.der

5.生成.p12二进制私钥文件
.pem 是base64的不能直接使用，必须导成.p12信息交换文件用来传递秘钥
openssl pkcs12 -export -out p.p12 -inkey private.pem -in rsacert.crt
输入一个导出密码（框架中loadPrivateKey:方法的password参数需要用的密码）：


输入导出密码界面.png