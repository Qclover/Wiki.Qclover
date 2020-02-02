## java基础

### 1.Gadget



#### 1.1setter与getter方法

我们在设置变量的属性时，我们通常会对数据进行封装，这样就可以增加了数据访问限制，增加了程序可维护性。而实现方法就是：用private去修饰一个变量，然后再用setter方法去设置该变量的值，然后在用getter方法去调用该变量的值。

作用：//私有的值，用setter和getter方法提供外界访问

格式为：

```
     getter（用于获取）：
       [非私有修饰符] 字段类型 get字段名称（首字母大写）()
      {
          return 字段名；
      }
```

```
setter（用于设置）：
 26     [非私有修饰符] void set字段名称（首字母大写）(字段类型 变量)
 27     {
 28         字段=变量；
 29     }
```

 例外：Boolean类型的是setter方法和is方法。

```
setter格式与上述相同，is方法只需把set编程is即可
```

```
public void setRs(boolean xs)
      {
          rs=xs;
      }
      public boolean isRs()
      {
          return rs;
      }
```

#### 2.Gadget和反序列化漏洞执行过程

commons.collections4-Gadget

基于org.apache.commons.collections4类库的Gadget，其通过构造一个特殊的PriorityQueue对象，将其序列化成字节流后，在字节流反序列化的过程中触发代码执行。

反序列化时首先从ObjectInputStream类的readObject方法中进入到PriorityQueue类的readObject方法里，其readObject方法中会进行堆化，堆化时队列中元素大于等于2时会进行堆排序，这时会调用自定义的比较器（TransformingComparator），TransformingComparator在比较次序时会将对象进行转换。转换时使用的transformer是基于ConstantTransformer对象和InstantiateTransformer对象构造的ChainedTransformer对象，ChainedTransformer对象在其转换方法（transform()）中会依次调用ConstantTransformer对象和InstantiateTransformer对象的transform方法，并将前一个对象transform方法的返回值作为参数传入后一个对象的transform方法中，InstantiateTransformer对象中的transform方法会基于参数（这里即ConstantTransformer.transform()的返回值com.sun.org.apache.xalan.internal.xsltc.trax.TrAXFilter）新建实例，则进入了TrAXFilter类的构造方法中，这里调用了TransformerImpl实例的newTransformer方法，又调用了getTransletInstance方法，加载_bytecodes中修改后的StubTransletPayload类字节码并生成实例，从而触发代码执行。

参考：[Java反序列化：基于CommonsCollections4的Gadget分析](https://www.freebuf.com/articles/others-articles/193445.html)

### 2.OGNL

2.1OGNL基本使用

   OGNL是Object-Graph Navigation Language（对象图导航语言）的缩写，它是一种功能强大的表达式语言通过简单一致的表达式语法，可以存取对象的任何属性，调用对象的方法，遍历整个对象的结构图，实现字段类型转化等功能、它使用相同的表达式去存取对象的属性。

**Struts2框架使用OGNL作为默认的表达式语言**

- OGNL是一种比EL强大很多倍的语言，支持对象方法调用，支持静态方法和字段访问，支持赋值操作等等。
- xwork提供了OGNL表达式。
- 其jar包为ognl-x.x.x.jar。

**OGNL的要素**

   OGNL有三大要素，分别是表达式、根对象、Context对象。

**表达式**

   表达式是OGNL的核心，带有语法含义的字符串，规定了操作的类型和操作的内容

**根对象（Root）**

Root对象可以理解为OGNL的操作对象。

OGNL称为对象图导航语 言，所谓对象图，即以任意一个对象为根，通过OGNL可以访问与这个对象关联的其它对象。

**Context对象**

   实际上OGNL的取值还需要一个上下文环境。设置了Root对象，OGNL可以对Root对象进行取值或写值等操作，Root对象所在环境就是OGNL的上下文环境(Context)。

上下文环境Context是一个Map类型的对象，在表达式中访问Context中的对象，需要使用"# "号加上对象名称，即#"对象名称"的形式。、

思维导图![OGNL](C:\Users\clover\Documents\文章\Java漏洞wiki\java基础\images\OGNL.png)

参考文章：https://blog.csdn.net/q547550831/article/details/53325094

相关OGNL漏洞：Struts 2漏洞（CVE-2018-11776/S2-057）

http://www.shungg.cn/post/229