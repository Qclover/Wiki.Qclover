###Java反序列化

####1.什么是java反序列化漏洞
​      在java中会调用ObjectInputStream的readObject方法进行对象读取 ，java反序列化的过程如果目标对象的readObject进行一些复杂的操作,反序列化不可信数据时 ,被恶意代码利用极可能造成漏洞。如在若指定类的指定方法中解析json字符串有可被恶意利用(Gadget[setter或getter方法 ])或添加了存在漏洞的commons c0llections危险库,可能导致不安全的代码被利用造成反序列化漏洞

相关函数：json.parse.json()

ObjectInputStream.readObject()

修复

相关：[浅谈Java反序列化漏洞修复方案](https://github.com/Cryin/Paper/blob/master/浅谈Java反序列化漏洞修复方案.md)



#### 2.开源组件或框架

#####2.1 apache shiro漏洞

**2.1.1apache shiro（2018-ceve|RememeberMe-cookie加密） java反序列化 原理**

版本<=1.2.4

环境

- shiro-core-1.2.4.jar
- commons-collections4-4.0.jar 

Shiro提供了记住我（RememberMe）的功能 ，Shiro对rememberMe的cookie做了加密处理 ，`CookieRememberMeManaer`类中将cookie中rememberMe字段内容分别进行 序列化、AES加密、Base64编码 ，该类继承了`AbstractRememberMeManager`类，跟进`AbstractRememberMeManager`类，可以找到到AES的key ，在利用时只要

第一步：获取rememberMe的Cookie

第二步：base64解码。`CookieRememberMeManager`类的`getRememberedSerializedIdentity()`方法

```
byte[] decoded = Base64.decode(base64);
```

第三步：AES解密。base64解码后的字节，减去前面16个字节。

`AbstractRememberMeManager`类的`decrypt()`方法

第四步：反序列化

在反序列化时调用readObject() ，由于在反序列化的对象完全由外部rememberMe Cookie控制 ，一旦添加了有漏洞的common-collections包，就会造成任意命令执行 

freebuf.com/vuls/170344.html

xz.aliyun.com/t/2041

**2.1.2Shiro Padding Oracle Attack CBC字节翻转 反序列化 (最新)**

**利用及漏洞原理**

把所有的block块分开提交，可以通过控制前一块的block密文，从而控制后一块的明文，根据Padding的机制，可构造出一个bool条件，从而逐位得到明文，然后逐块得到所有明文。 

**利用条件及场景：padding错误和padding正确服务器可返回不一样的状态。** 

**Padding Oracle Attack**

明文分组和填充就是Padding Oracle
Attack的根源所在，但是这些需要一个前提，那就是应用程序对异常的处理。当提交的加密后的数据中出现错误的填充信息时，不够健壮的应用程序解密时报
错，直接抛出”填充错误”异常信息(这个错误信息在不同的应用中是不同的体现，在web一般是报500错误)。

攻击者就是利用这个异常来做一些事情，假设有这样一个场景，一个WEB程序接受一个加密后的字符串作为参数，这个参数包含用户名、密码。参数加密使
用的最安全的CBC模式，每一个block有一个初始化向量IV(注意：这个IV在服务器第一次生成这个密文的时候就产生了，并保存在服务器上，攻击者需
要在提交数据的时候也提交这个IV，后面我们会看到，攻击者实际上就是在”穷举”这个IV)。

当提交参数时，服务端的返回结果会有下面3种情况：
a. 参数是一串正确的密文，分组、填充、加密都是对的(程序运行本身没出问题)，包含的内容也是正确的(业务逻辑是对的)，那么服务端解密、检测用户权限都没有问题，返回HTTP 200。
b. 参数是一串错误的密文，包含不正确的bit填充(程序运行本身出现致命错误)，那么服务端解密时就会抛出异常，返回HTTP 500 server error。
c. 参数是一串正确的密文(程序运行本身没出问题)，包含的用户名是错误的(业务逻辑是错的)，那么服务端解密之后检测权限不通过，但是依旧会返回HTTP 200戒者HTTP 302，而不是HTTP 500。

攻击者无需关心用户名是否正确，只需要提交错误的密文(因为这里有4中变量情况，为了构造出二值逻辑推理，我们要定住其中2个情况，即让业务逻辑恒错，对Bit Padding的情况进行逻辑推理)，根据HTTP Code即可做出攻击。

cbc字节翻转攻击

攻击原理：只需修改第一组密文对应第二组'skctf'的位置的密文，就可以实现第二组明文的改变。即第10-14位。 

![1577345559724](C:\Users\clover\AppData\Local\Temp\1577345559724.png)

#####**2.2Fastjson 最新反序列化漏洞**

版本：<=1.2.47

通杀默认配置1.2.48版本以下

在autotype开启的情况下可以打到1.2.57。

利用成功条件：jdk 1.8.0.x低版本 Java `1.8.0_102` 进行

环境：1.使用python开启web服务

​             2.marshalsec快速开启rmi或ldap服务

**原理：**

   通过java.lang.Class，将JdbcRowSetImpl类加载到map缓存，从而绕过autotype的检测。因此将payload分两次发送，第一次加载，第二次执行。

​    当发送第一次请求时，Class是通过deserializers.findClass加载的，然后Class将JdbcRowSetImpl类加载进map中，然后第二次请求时，就这里就成功找到了JdbcRowSetImpl类，从而绕过检测。

​       加载JdbcRowSetImpl后，通过JdbcRowSetImpl中的调用链，通过jndi的lookup加载远程类。最终导致命令执行

坑点

rmi和ldap这种利用方式对版本是有要求的

![img](https://img2018.cnblogs.com/blog/1495879/201908/1495879-20190808212509909-1430686884.png)



##### 2.3Log4j<=1.2.17反序列化

**漏洞详情**

log4j是Apache开发的一个日志工具。可以将Web项目中的日志输出到控制台，文件，GUI组件，甚至是套接口服务器。本次出现漏洞就是因为log4j在启动套接口服务器后，对监听端口传入的反序列化数据没有进行过滤而造成的。

**漏洞分析**

见：[log4j=1.2.17-Socket-反序列化漏洞](http://qclover.cn/2020/01/04/log4j=1.2.17-Socket-反序列化漏洞.html)

####3.如何发现Java反序列化漏洞

流程如下：

① 通过检索源码中对反序列化函数的调用来静态寻找反序列化的输入点
可以搜索以下函数：

```
ObjectInputStream.readObject
ObjectInputStream.readUnshared
XMLDecoder.readObject
Yaml.load
XStream.fromXML
ObjectMapper.readValue
JSON.parseObject
```

小数点前面是类名，后面是方法名

② 确定了反序列化输入点后，再考察应用的Class Path中是否包含Apache Commons Collections等危险库（ysoserial所支持的其他库亦可）。

③ 若不包含危险库，则查看一些涉及命令、代码执行的代码区域，防止程序员代码不严谨，导致bug。

④ 若包含危险库，则使用ysoserial进行攻击复现。



参考：https://www.anquanke.com/post/id/87270