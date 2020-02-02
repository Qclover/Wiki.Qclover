## Java-XXE漏洞

###1.XXE简介

XXE(XML外部实体注入、XML External Entity），在应用程序解析XML输入时，当允许引用外部实体时，可以构造恶意内容导致读取任意文件或SSRF、端口探测、DoS拒绝服务攻击、执行系统命令、攻击内部网站等。Java中的XXE支持sun.net.www.protocol里面的所有[协议](http://www.liuhaihua.cn/archives/tag/protocol)：[http](http://www.liuhaihua.cn/archives/tag/http)，[https](http://www.liuhaihua.cn/archives/tag/https)，file，[ftp](http://www.liuhaihua.cn/archives/tag/ftp)，[mail](http://www.liuhaihua.cn/archives/tag/mail)to，jar，[netdoc](http://qclover.cn/2019/01/13/java自定义通信协议及利用.html) 。一般利用file协议读取文件、利用http协议探测内网，没有回显时可组合利用file协议和ftp协议来读取文件。有关XXE漏洞利用可移至[PHP与JAVA之XXE漏洞详)解与审计](http://qclover.cn/2019/11/14/PHP与JAVA之XXE漏洞详解与审计.html)

###2.相关漏洞函数

JAVA解析XML的方法越来越多，常见有四种，即：[DOM](http://www.liuhaihua.cn/archives/tag/dom)、DOM4J、JDOM 和SAX。

相关函数：

```
javax.xml.parsers.DocumentBuilderFactory;
javax.xml.parsers.SAXParser
org.dom4j.io.SAXReader
org.xml.sax.XMLReader
```

###3.防御

setFeature:检验XML的合法性，限定URI和验证URI白名单

具体设置

```
DocumentBuilderFactory dbf =DocumentBuilderFactory.newInstance();
dbf.setExpandEntityReferences(false);

.setFeature("http://apache.org/xml/features/disallow-doctype-decl",true);

.setFeature("http://xml.org/sax/features/external-general-entities",false)

.setFeature("http://xml.org/sax/features/external-parameter-entities",false);

```

过滤用户提交的XML数据

关键词：<!DOCTYPE和<!ENTITY，或者，SYSTEM和PUBLIC。