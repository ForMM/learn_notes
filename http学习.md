# http https学习

### HTTP请求过程

1. 浏览器根据访问的域名找到其IP地址。DNS查找过程：

   1. 浏览器缓存：首先搜索浏览器自身的DNS缓存（缓存的时间比较短，大概只有1分钟，且只能容纳1000条缓存），看自身的缓存中是否是有域名对应的条目，而且没有过期，如果有且没有过期则解析到此结束。
   2. 系统缓存：如果浏览器自身的缓存里面没有找到对应的条目，那么浏览器会搜索操作系统自身的DNS缓存，如果找到且没有过期则停止搜索解析到此结束。
   3. 路由器缓存：如果系统缓存也没有找到，则会向路由器发送查询请求。
   4. ISP（互联网服务提供商） DNS缓存：如果在路由缓存也没找到，最后要查的就是ISP缓存DNS的服务器。

2. 浏览器与web服务器建立一个tcp连接

   tcp的三次握手

3. 浏览器给web服务器发送http请求

   http请求包括请求行、请求头、请求数据

   - 请求行：请求方法、请求地址URL和HTTP协议版本，它们之间用空格分割。
     - 请求方法： HTTP/1.1 定义的请求方法有8种：GET（完整请求一个资源）、POST（提交表单）、PUT（上传文件）、DELETE（删除）、PATCH、HEAD（仅请求响应首部）、OPTIONS（返回请求的资源所支持的方法）、TRACE（追求一个资源请求中间所经过的代理）。最常的两种GET和POST，如果是RESTful接口的话一般会用到GET、POST、DELETE、PUT。
   - 请求头：
   - 

   请求头：

   ~~~html
   Accept: */*
   Accept-Encoding: gzip, deflate, br
   Accept-Language: zh-CN,zh;q=0.9
   Connection: keep-alive
   Content-Type: text/plain;charset=UTF-8
   Host: event.csdn.net
   Origin: https://blog.csdn.net
   Referer: https://blog.csdn.net/ailunlee/article/details/90600174
   User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.98 Safari/537.36
   ~~~

   ![http-2](.\img\http-2.png)

   请求数据

   ![http-3](.\img\http-3.png)

4. 服务器响应HTTP请求，返回HTML代码

   HTTP响应报文由状态行（status line）、相应头部（headers）、空行（blank line）和响应数据（response body）4个部分组成。

   - 状态行

     ![http-5](.\img\http-5.png)

   - 响应头部

     ![http-4](.\img\http-4.png)

   - 响应数据

    状态行由3部分组成，分别为：协议版本、状态码、状态码扫描。

5. 浏览器解析html代码，并请求html代码中的资源

6. 关闭tcp连接，浏览器对页面进行渲染呈现给用户

### HTTP底层原理





### java实现http的几种方式

1. HttpUrlConnection

   java中标准类，比较原生的实现方式。

2. HttpClient

   第三方工具包。httpClient有多种版本，具体实现请求的方式有两种，一种是创建HttpClient接口，另一种创建CloseableHttpClient抽象类。
   
   注意：需要代理服务器的话，加上代理服务地址，再发送请求。
   
   
   

### https请求过程详解

​	HTTPS的请求中使用了非对称加密算法。非对称加密，加密和解密过程使用不同的密钥，一个公钥，对外公开，一个私钥，仅是解密端拥有。由于公钥和私钥是分开的，非对称加密算法安全级别高，加密密文长度有限制，适用于对少量数据进行加密，速度较慢。

![http-1](.\img\http-1.png)

上述过程就是两次HTTP请求，其详细过程如下：

1. 客户端想服务器发起HTTPS的请求，连接到服务器的443端口；

2. 服务器将非对称加密的公钥传递给客户端，以证书的形式回传到客户端

3. 服务器接受到该公钥进行验证，就是验证2中证书，如果有问题，则HTTPS请求无法继续；如果没有问题，则上述公钥是合格的。（第一次HTTP请求）客户端这个时候随机生成一个私钥，成为client key,客户端私钥，用于对称加密数据的。使用前面的公钥对client key进行非对称加密；

4. 进行二次HTTP请求，将加密之后的client key传递给服务器；

5. 服务器使用私钥进行解密，得到client key,使用client key对数据进行对称加密

6. 将对称加密的数据传递给客户端，客户端使用非对称解密，得到服务器发送的数据，完成第二次HTTP请求。

### SSL协议请求过程分析

![http-6](.\img\http-6.png)

ClientHello过程：版本号、客户端随机数、密码套件信息、压缩方式、扩展信息等成员。

~~~
*** ClientHello, TLSv1.2
RandomCookie:  GMT: 1590739114 bytes = { 224, 123, 128, 208, 217, 216, 109, 177, 159, 189, 159, 121, 179, 143, 54, 96, 209, 247, 164, 149, 24, 136, 227, 191, 171, 233, 89, 198 }
Session ID:  {}
Cipher Suites: [TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256, TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256, TLS_RSA_WITH_AES_128_CBC_SHA256, TLS_ECDH_ECDSA_WITH_AES_128_CBC_SHA256, TLS_ECDH_RSA_WITH_AES_128_CBC_SHA256, 
TLS_DHE_RSA_WITH_AES_128_CBC_SHA256, TLS_DHE_DSS_WITH_AES_128_CBC_SHA256, TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA, TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA, TLS_RSA_WITH_AES_128_CBC_SHA, TLS_ECDH_ECDSA_WITH_AES_128_CBC_SHA, 
TLS_ECDH_RSA_WITH_AES_128_CBC_SHA, TLS_DHE_RSA_WITH_AES_128_CBC_SHA, TLS_DHE_DSS_WITH_AES_128_CBC_SHA, TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256, TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256, TLS_RSA_WITH_AES_128_GCM_SHA256, 
TLS_ECDH_ECDSA_WITH_AES_128_GCM_SHA256, TLS_ECDH_RSA_WITH_AES_128_GCM_SHA256, TLS_DHE_RSA_WITH_AES_128_GCM_SHA256, TLS_DHE_DSS_WITH_AES_128_GCM_SHA256, TLS_ECDHE_ECDSA_WITH_3DES_EDE_CBC_SHA, TLS_ECDHE_RSA_WITH_3DES_EDE_CBC_SHA, 
SSL_RSA_WITH_3DES_EDE_CBC_SHA, TLS_ECDH_ECDSA_WITH_3DES_EDE_CBC_SHA, TLS_ECDH_RSA_WITH_3DES_EDE_CBC_SHA, SSL_DHE_RSA_WITH_3DES_EDE_CBC_SHA, SSL_DHE_DSS_WITH_3DES_EDE_CBC_SHA, TLS_EMPTY_RENEGOTIATION_INFO_SCSV]
Compression Methods:  { 0 }
Extension elliptic_curves, curve names: {secp256r1, secp384r1, secp521r1, sect283k1, sect283r1, sect409k1, sect409r1, sect571k1, sect571r1, secp256k1}
Extension ec_point_formats, formats: [uncompressed]
Extension signature_algorithms, signature_algorithms: SHA512withECDSA, SHA512withRSA, SHA384withECDSA, SHA384withRSA, SHA256withECDSA, SHA256withRSA, SHA256withDSA, SHA1withECDSA, SHA1withRSA, SHA1withDSA
Extension server_name, server_name: [type=host_name (0), value=jcloud.gyuncai.cn]
***
*** ServerHello, TLSv1.2
RandomCookie:  GMT: -1031972689 bytes = { 61, 61, 171, 195, 167, 159, 124, 91, 175, 232, 148, 213, 22, 60, 144, 236, 49, 177, 84, 109, 78, 13, 138, 208, 84, 4, 243, 64 }
Session ID:  {247, 126, 138, 171, 46, 187, 24, 39, 120, 238, 182, 255, 68, 222, 7, 30, 154, 176, 241, 88, 106, 227, 141, 38, 48, 41, 9, 107, 40, 189, 131, 95}
Cipher Suite: TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
Compression Method: 0
Extension renegotiation_info, renegotiated_connection: <empty>
Extension server_name, server_name: 
Extension ec_point_formats, formats: [uncompressed, ansiX962_compressed_prime, ansiX962_compressed_char2]
***
~~~

~~~
Security.removeProvider("SunEC");
这个语句移除了一些加密组件，当启动时候本来会加载jdk下安全包的加密组件，这个语句移除了导致客户端的https请求支持的组件，我们这边不支持导致ssl协议连接握手失败。
~~~

https://guoxiaodong.blog.csdn.net/article/details/52469674?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-4.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-4.control

### 浏览器如何验证HTTPS证书的合法性

1. 验证浏览器中“受信任的根证书颁发机构”是否存在颁发该SSL证书的机构

2. 检查证书有没有被证书颁发机构吊销

3. 验证该网站的SSL证书是否过期

4. 审核该SSL证书的网站的域名是否与证书中的域名一致

5. 该网站有没有被列入欺诈网站黑名单

   https://blog.csdn.net/jasonhwang/article/details/2344768?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-4.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-4.control

### HTTPS和HTTP区别

1. HTTPS是密文传输，HTTP是明文传输；
2. 默认连接的端口号是不同的，HTTPS是443端口，而HTTP是80端口；
3. HTTPS请求的过程需要CA证书要验证身份以保证客户端请求到服务器端之后，传回的响应是来自于服务器端，而HTTP则不需要CA证书；
4. HTTPS=HTTP+加密+认证+完整性保护。

### 解决HTTPS握手失败

https://blog.csdn.net/u014644574/article/details/83381303?utm_medium=distribute.pc_relevant.none-task-blog-baidulandingword-10&spm=1001.2101.3001.4242

### Cookie和Session

**HTTP协议是无状态的协议。一旦数据交换完毕，客户端与服务端的链接就会关闭，再次交换数据需要建立新的链接。这就意味着服务器无法从链接上面跟踪会话。**Cookie和Session都为了用来保存状态信息，都是保存客户端状态的机制，它们都是为了解决HTTP无状态的问题而所做的努力。

1. Cookies是服务器在本地机器上存储的小段文本并随每一个请求发送至同一个服务器。Cookie最早在RFC2109中实现，后续RFC2965做了增强。网络服务器用HTTP头向客户端发送cookies，在客户终端，浏览器解析这些cookies并将它们保存为一个本地文件，它会自动将同一服务器的任何请求缚上这些cookies。Session并没有在HTTP的协议中定义；
2. Session是针对每一个用户的，变量的值保存在服务器上，用一个sessionID来区分是哪个用户session变量,这个值是通过用户的浏览器在访问的时候返回给服务器，当客户禁用cookie时，这个值也可能设置为由get来返回给服务器；

- session机制

  Session机制是一种服务器端的机制，服务器使用一种类似于散列表的结构（也可能就是使用散列表）来保存信息。

  当程序需要为某个客户端的请求创建一个session的时候，服务器首先检查这个客户端的请求里是否已包含了一个session标识 - 称为 session id，如果已包含一个session id则说明以前已经为此客户端创建过session，服务器就按照session id把这个 session检索出来使用（如果检索不到，可能会新建一个），如果客户端请求不包含session id，则为此客户端创建一个session并且生成一个与此session相关联的session id，session id的值应该是一个既不会重复，又不容易被找到规律以仿造的字符串，这个 session id将被在本次响应中返回给客户端保存。

- session实现机制

  使用cookie来实现：服务器给每个Session分配一个唯一的JSESSIONID，并通过Cookie发送给客户端。

  当客户端发起新的请求的时候，将在Cookie头中携带这个JSESSIONID。这样服务器能够找到这个客户端对应的Session。

  ![seesion-1](.\img\seesion-1.png)

- cookie流程

  服务器在响应消息中用Set-Cookie头将Cookie的内容回送给客户端，客户端在新的请求中将相同的内容携带在Cookie头中发送给服务器。从而实现会话的保持。

  ![cookie-1](.\img\cookie-1.png)

### web缓存

WEB缓存(cache)位于Web服务器和客户端之间。

缓存会根据请求保存输出内容的副本，例如html页面，图片，文件，当下一个请求来到的时候：如果是相同的URL，缓存直接使用副本响应访问请求，而不是向源服务器再次发送请求。

http协议中拓展消息头：

**Expires**：指示响应内容过期的时间，格林威治时间GMT

   **Cache-Control**：更细致的控制缓存的内容

   **Last-Modified**：响应中资源最后一次修改的时间

   **ETag**：响应中资源的校验值，在服务器上某个时段是唯一标识的。

   **Date**：服务器的时间

   **If-Modified-Since**：客户端存取的该资源最后一次修改的时间，同Last-Modified。

   **If-None-Match**：客户端存取的该资源的检验值，同ETag。

