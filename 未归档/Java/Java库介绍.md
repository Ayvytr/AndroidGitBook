# Java库

### java、javax、sun、org包有什么区别

java、javax、org、sun包都是jdk提供的类包，且都是在rt.jar中。rt.jar是JAVA基础类库（java核心框架中很重要的包），包含lang在内的大部分功能，而且rt.jar默认就在根classloader的加载路径里面，所以放在classpath是多此一举 。他们之间的区别具体如下： 
了解java核心框架看这里：java核心框架是什么样的

java.* 
java SE的标准库，是java标准的一部分，是对外承诺的java开发接口，通常要保持向后兼容，一般不会轻易修改。包括其他厂家的在内，所有jdk的实现，在java.*上都是一样的。

javax.* 
也是java标准的一部分，但是没有包含在标准库中，一般属于标准库的扩展。通常属于某个特定领域，不是一般性的api。 
所以以扩展的方式提供api，以避免jdk的标准库过大。当然某些早期的javax，后来被并入到标准库中，所有也应该属于新版本JDK的标准库。比如jmx，java 5以前是以扩展方式提供，但是jdk5以后就做为标准库的一部分了，所有javax.management也是jdk5的标准库的一部分。

com.sun.* 
是sun的hotspot虚拟机中java.* 和javax.*的实现类。因为包含在rt中，所以我们也可以调用。但是因为不是sun对外公开承诺的接口，所以根据根据实现的需要随时增减，因此在不同版本的hotspot中可能是不同的，而且在其他的jdk实现中是没有的，调用这些类，可能不会向后兼容，所以一般不推荐使用。

org.omg.* 

是由企业或者组织提供的java类库，大部分不是sun公司提供的，同com.sun.*，不具备向后兼容性，会根据需要随时增减。其中比较常用的是w3c提供的对XML、网页、服务器的类和接口。



## [Applet](http://www.runoob.com/java/java-applet-basics.html)

## 包含用于创建用户界面和绘制图形图像的所有分类

## Beans



## IO [IO体系的学习总结](https://blog.csdn.net/nightcurtis/article/details/51324105) 

在整个Java.io包中最重要的就是5个类和一个接口。5个类指的是File、OutputStream、InputStream、[Writer](https://www.baidu.com/s?wd=Writer&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)、Reader；一个接口指的是Serializable.掌握了这些IO的核心操作那么对于Java中的IO体系也就有了一个初步的认识了



Java I/O主要包括如下几个层次，包含三个部分：



***1.流式部分**――IO的主体部分；

***2.非流式部分**――主要包含一些辅助流式部分的类，如：File类、RandomAccessFile类和FileDescriptor等类；

***3.其他类**--文件读取部分的与安全相关的类，如：SerializablePermission类，以及与本地操作系统相关的文件系统的类，如：FileSystem类和Win32FileSystem类和WinNTFileSystem类。

   *主要的类如下：*

​     1. File（文件特征与管理）：用于文件或者目录的描述信息，例如生成新目录，修改文件名，删除文件，判断文件所在路径等。

​     2. InputStream（二进制格式操作）：抽象类，基于字节的输入操作，是所有输入流的父类。定义了所有输入流都具有的共同特征。

​     3. OutputStream（二进制格式操作）：抽象类。基于字节的输出操作。是所有输出流的父类。定义了所有输出流都具有的共同特征。

​     **Java中字符是采用Unicode标准，一个字符是16位，即一个字符使用两个字节来表示。为此，JAVA中引入了处理字符的流。**

​     4. Reader（文件格式操作）：抽象类，基于字符的输入操作。

​     5. Writer（文件格式操作）：抽象类，基于字符的输出操作。

        6. RandomAccessFile（随机文件操作）：它的功能丰富，**可以从文件的任意位置进行存取（输入输出）操作**。
        7. 

| 分 类         | 字节输入流           | 字节输出流            | 字符输入流        | 字符输出流         |
| ---------------- | -------------------- | --------------------- | ----------------- | ------------------ |
| 抽象基类   | InputStream          | OutputStream          | Reader            | Writer             |
| 访问文件   | FileInputStream      | FileOutputStream      | FileReader        | FileWriter         |
| 访问数组   | ByteArrayInputStream | ByteArrayOutputStream | CharArrayReader   | CharArrayWriter    |
| 访问管道   | PipedInputStream     | PipedOutputStream     | PipedReader       | PipedWriter        |
| 访问字符串 |                      |                       | StringReader      | StringWriter       |
| 缓冲流     | BufferedInputStream  | BufferedOutputStream  | BufferedReader    | BufferedWriter     |
| 转换流     |                      |                       | InputStreamReader | OutputStreamWriter |
| 对象流     | ObjectInputStream    | ObjectOutputStream    |                   |                    |
| 抽象基类   | FilterInputStream    | FilterOutputStream    | FilterReader      | FilterWriter       |
| 打印流     |                      | PrintStream           |                   | PrintWriter        |
| 推回输入流 | PushbackInputStream  |                       | PushbackReader    |                    |
| 特殊流     | DataInputStream      | DataOutputStream      |                   |                    |



### 分类

按照流的流向分，可以分为输入流和输出流

按照操作单元划分，可以划分为字节流和字符流

按照流的角色划分为节点流和处理流

InputStream OutputStream是字节流

Reader Writer是字符流



## NIO

## lang

**java.lang包是[java语言](https://www.baidu.com/s?wd=java%E8%AF%AD%E8%A8%80&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)的核心，它提供了java中的基础类。包括基本Object类、Class类、String类、基本类型的包装类、基本的数学类等等最基本的类。**



下面分别介绍其中比较常用的类：

一、类型
对象基类 
Object，是java.lang的根类，也是所有类的超类。

类 
Class，用来表示类和接口的类型。Class对象在类加载时由JVM调用类加载器中的defineClass方法自动构造。 
ClassLoader，负责加载类。 
Compiler，作为编译器的占位符，它不做任何事情，仅用来支持Java到本机代码的编译器及相关服务。

基本类型 
基本类型的包装类，包括Boolean、Character、Byte、Short、Integer、Long、Float、Double，其中数值类型均即成Number类。 
String，字符串类。

字符序列 
StringBuffer、StringBuilder，可变的字符序列。

枚举 
Enum，是所有枚举类型的公共基类。

包 
Package，包含了有关Java包（package）的信息。

无类型 
Void，标示关键字void的Class对象的引用，不可被实例化。

迭代器 
Iterable，可迭代接口，实现接口可以使用迭代器进行对象遍历。

二、工具
数学 
Math、StrictMath，提供了基本的数字操作，如指数、对数、平方根和三角函数。一般情况下，Math调用StrictMath的方法来完成实现。java中还有一个java.math包，这个包主要提供用于执行任意精度整数算法 (BigInteger) 和任意精度小数算法 (BigDecimal) 的类。

安全 
SecurityManager，允许应用程序实现安全策略的类。

注解 
Override，标记类中方法是实现/重写父类的方法。 
SuppressWarnings，取消对被标记的元素的警告。

反射



Ref

SoftReference

WeakReference等

三、系统

进程 
Process，进程抽象类。 
ProcessBuilder，用于创建操作系统进程。 
ProcessEnvironment，进程的运行环境参数。 
ProcessImpl，进行接口的实现类。

线程 
Thread，进程中的执行线程。 
ThreadGroup，线程组，表示一个线程的集合。它构成一个树状结构，可以包含其他线程组，除了根节点的线程组，每个线程组都具有父线程组。 
ThreadLocal，提供线程的变量。

运行 
Runnable，可运行接口，所有Thread都应实现它。 
Runtime，运行时类，将应用程序与其运行的环境相关联。 
RuntimePermission，用于运行时权限。 
System，系统级的很多属性和控制方法都放置在该类的内部。

堆栈 
StackTraceElement，堆栈跟踪中的元素，它的每个实例都表示单独的一个栈帧（表示一个方法调用）。

异常 

Throwable，异常基类，Java中所有异常都继承于它。



## math

3个主要的类：

​	BigInteger 

​	BigDecimal

​	RoundingMode



## net

## rmi

RPC (Remote Procedure Call):远程方法调用，用于一个进程调用另一个进程中的过程，从而提供了过程的分布能力。

RMI（Remote Method Invocation):远程方法调用，即在RPC的基础上有向前迈进了一步，提供分布式对象间的通讯。允许运行在一个java 虚拟机的对象调用运行在另一个java虚拟机上对象的方法。这两个虚拟机可以是运行在相同计算机上的不同进程中，也可以是运行在网络上的不同计算机中。



## security

## sql

## text

## time

## util

包含集合框架、遗留的 collection 类、事件模型、日期和时间设施、国际化和各种实用工具类（字符串标记生成器、随机数生成器和位[数组](https://baike.baidu.com/item/%E6%95%B0%E7%BB%84/3794097)、日期Date类、堆栈Stack类、向量Vector类等）。集合类、时间处理模式、日期时间工具等各类常用工具包

### concurrent

### function















