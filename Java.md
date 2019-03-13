# Java

## Java概述

1991年4月，由James Gosling博士领导的绿色计划（Green Project）开始启动，此计划的目的是开发一种能够在各种消费性电子产品（如机顶盒、冰箱、收音机等）上运行的程序架构。这个计划的产品就是Java语言的前身：Oak（橡树）。Oak当时在消费品市场上并不算成功，但随着1995年互联网潮流的兴起，Oak迅速找到了最适合自己发展的市场定位并蜕变成为Java语言。

　　1995年5月23日，Oak语言改名为Java，并且在SunWorld大会上正式发布Java 1.0版本。Java语言第一次提出了“Write Once，Run Anywhere”的口号。

　　1996年1月23日，JDK 1.0发布，Java语言有了第一个正式版本的运行环境。JDK 1.0提供了一个纯解释执行的Java虚拟机实现（Sun Classic VM）。JDK 1.0版本的代表技术包括：Java虚拟机、Applet、AWT等。

　　1996年4月，10个最主要的操作系统供应商申明将在其产品中嵌入Java技术。同年9月，已有大约8.3万个网页应用了Java技术来制作。在1996年5月底，Sun公司于美国旧金山举行了首届JavaOne大会，从此JavaOne成为全世界数百万Java语言开发者每年一度的技术盛会。

　　1997年2月19日，Sun公司发布了JDK 1.1，Java技术的一些最基础的支撑点（如JDBC等）都是在JDK 1.1版本中发布的，JDK 1.1版的技术代表有：JAR文件格式、JDBC、JavaBeans、RMI。Java语法也有了一定的发展，如内部类（Inner Class）和反射（Reflection）都是在这个时候出现的。

　　直到1999年4月8日，JDK 1.1一共发布了1.1.0～1.1.8九个版本。从1.1.4之后，每个JDK版本都有一个自己的名字（工程代号），分别为：JDK 1.1.4 - Sparkler（宝石）、JDK 1.1.5 - Pumpkin（南瓜）、JDK 1.1.6 - Abigail（阿比盖尔，女子名）、JDK 1.1.7 - Brutus（布鲁图，古罗马政治家和将军）和JDK 1.1.8 – Chelsea（切尔西，城市名）。

　　1998年12月4日，JDK迎来了一个里程碑式的版本JDK 1.2，工程代号为Playground（竞技场），Sun在这个版本中把Java技术体系拆分为3个方向，分别是面向桌面应用开发的J2SE（Java 2 Platform， Standard Edition）、面向企业级开发的J2EE（Java 2 Platform， Enterprise Edition）和面向手机等移动终端开发的J2ME（Java 2 Platform， Micro Edition）。在这个版本中出现的代表性技术非常多，如EJB、Java Plug-in、Java IDL、Swing等，并且这个版本中Java虚拟机第一次内置了JIT（Just In Time）编译器（JDK 1.2中曾并存过3个虚拟机，Classic VM、HotSpot VM和Exact VM，其中Exact VM只在Solaris平台出现过；后面两个虚拟机都是内置JIT编译器的，而之前版本所带的Classic VM只能以外挂的形式使用JIT编译器）。在语言和API级别上，Java添加了strictfp关键字与现在Java编码之中极为常用的一系列Collections集合类。

　　在1999年3月和7月，分别有JDK 1.2.1和JDK 1.2.2两个小版本发布。

　　1999年4月27日，HotSpot虚拟机发布，HotSpot最初由一家名为“Longview Technologies”的小公司开发，因为HotSpot的优异表现，这家公司在1997年被Sun公司收购了。HotSpot虚拟机发布时是作为JDK 1.2的附加程序提供的，后来它成为了JDK 1.3及之后所有版本的Sun JDK的默认虚拟机。

　　2000年5月8日，工程代号为Kestrel（美洲红隼）的JDK 1.3发布，JDK 1.3相对于JDK 1.2的改进主要表现在一些类库上（如数学运算和新的Timer API等），JNDI服务从JDK 1.3开始被作为一项平台级服务提供（以前JNDI仅仅是一项扩展），使用CORBA IIOP来实现RMI的通信协议，等等。这个版本还对Java 2D做了很多改进，提供了大量新的Java 2D API，并且新添加了JavaSound类库。JDK 1.3有1个修正版本JDK 1.3.1，工程代号为Ladybird（瓢虫），于2001年5月17日发布。

　　自从JDK 1.3开始，Sun维持了一个习惯：大约每隔两年发布一个JDK的主版本，以动物命名，期间发布的各个修正版本则以昆虫作为工程名称。

　　2002年2月13日，JDK 1.4发布，工程代号为Merlin（灰背隼）。JDK 1.4是Java真正走向成熟的一个版本，Compaq、Fujitsu、SAS、Symbian、IBM等著名公司都有参与甚至实现自己独立的JDK 1.4。哪怕是在十多年后的今天，仍然有许多主流应用（Spring、Hibernate、Struts等）能直接运行在JDK 1.4之上，或者继续发布能运行在JDK 1.4上的版本。JDK 1.4同样发布了很多新的技术特性，如正则表达式、异常链、NIO、日志类、XML解析器和XSLT转换器等。

　　JDK 1.4有两个后续修正版：

　　2002年9月16日发布的工程代号为Grasshopper（蚱蜢）的JDK 1.4.1

　　2003年6月26日发布的工程代号为Mantis（螳螂）的JDK 1.4.2。

　　2002年前后还发生了一件与Java没有直接关系，但事实上对Java的发展进程影响很大的事件，那就是微软公司的.NET Framework发布了。这个无论是技术实现上还是目标用户上都与Java有很多相近之处的技术平台给Java带来了很多讨论、比较和竞争，.NET平台和Java平台之间声势浩大的孰优孰劣的论战到目前为止都在继续。

　　2004年9月30日，JDK 1.5发布，工程代号Tiger（老虎）。从JDK 1.2以来，Java在语法层面上的变换一直很小，而JDK 1.5在Java语法易用性上做出了非常大的改进。例如，自动装箱、泛型、动态注解、枚举、可变长参数、遍历循环（foreach循环）等语法特性都是在JDK 1.5中加入的。在虚拟机和API层面上，这个版本改进了Java的内存模型（Java Memory Model，JMM）、提供了java.util.concurrent并发包等。另外，JDK 1.5是官方声明可以支持Windows 9x平台的最后一个JDK版本。

　　2006年12月11日，JDK 1.6发布，工程代号Mustang（野马）。在这个版本中，Sun终结了从JDK 1.2开始已经有8年历史的J2EE、J2SE、J2ME的命名方式，启用Java SE 6、Java EE 6、Java ME 6的命名方式。JDK 1.6的改进包括：提供动态语言支持（通过内置Mozilla Java Rhino引擎实现）、提供编译API和微型HTTP服务器API等。同时，这个版本对Java虚拟机内部做了大量改进，包括锁与同步、垃圾收集、类加载等方面的算法都有相当多的改动。

　　在2006年11月13日的JavaOne大会上，Sun公司宣布最终会将Java开源，并在随后的一年多时间内，陆续将JDK的各个部分在GPL v2（GNU General Public License v2）协议下公开了源码，并建立了OpenJDK组织对这些源码进行独立管理。除了极少量的产权代码（Encumbered Code，这部分代码大多是Sun本身也无权限进行开源处理的）外，OpenJDK几乎包括了Sun JDK的全部代码，OpenJDK的质量主管曾经表示，在JDK 1.7中，Sun JDK和OpenJDK除了代码文件头的版权注释之外，代码基本上完全一样，所以OpenJDK 7与Sun JDK 1.7本质上就是同一套代码库开发的产品。

　　JDK 1.6发布以后，由于代码复杂性的增加、JDK开源、开发JavaFX、经济危机及Sun收购案等原因，Sun在JDK发展以外的事情上耗费了很多资源，JDK的更新没有再维持两年发布一个主版本的发展速度。JDK 1.6到目前为止一共发布了37个Update版本，最新的版本为Java SE 6 Update 37，于2012年10月16日发布。

　　2009年2月19日，工程代号为Dolphin（海豚）的JDK 1.7完成了其第一个里程碑版本。根据JDK 1.7的功能规划，一共设置了10个里程碑。最后一个里程碑版本原计划于2010年9月9日结束，但由于各种原因，JDK 1.7最终无法按计划完成。

　　从JDK 1.7最开始的功能规划来看，它本应是一个包含许多重要改进的JDK版本，其中的Lambda项目（Lambda表达式、函数式编程）、Jigsaw项目（虚拟机模块化支持）、动态语言支持、GarbageFirst收集器和Coin项目（语言细节进化）等子项目对于Java业界都会产生深远的影响。在JDK 1.7开发期间，Sun公司由于相继在技术竞争和商业竞争中都陷入泥潭，公司的股票市值跌至仅有高峰时期的3%，已无力推动JDK 1.7的研发工作按正常计划进行。为了尽快结束JDK 1.7长期“跳票”的问题，Oracle公司收购Sun公司后不久便宣布将实行“B计划”，大幅裁剪了JDK 1.7预定目标，以便保证JDK 1.7的正式版能够于2011年7月28日准时发布。“B计划”把不能按时完成的Lambda项目、Jigsaw项目和Coin项目的部分改进延迟到JDK 1.8之中。最终，JDK 1.7的主要改进包括：提供新的G1收集器（G1在发布时依然处于Experimental状态，直至2012年4月的Update 4中才正式“转正”）、加强对非Java语言的调用支持（JSR-292，这项特性到目前为止依然没有完全实现定型）、升级类加载架构等。

　　到目前为止，JDK 1.7已经发布了9个Update版本，最新的Java SE 7 Update 9于2012年10月16日发布。从Java SE 7 Update 4起，Oracle开始支持Mac OS X操作系统，并在Update 6中达到完全支持的程度，同时，在Update 6中还对ARM指令集架构提供了支持。至此，官方提供的JDK可以运行于Windows（不含Windows 9x）、Linux、Solaris和Mac OS平台上，支持ARM、x86、x64和Sparc指令集架构类型。

　　2009年4月20日，Oracle公司宣布正式以74亿美元的价格收购Sun公司，Java商标从此正式归Oracle所有（Java语言本身并不属于哪间公司所有，它由JCP组织进行管理，尽管JCP主要是由Sun公司或者说Oracle公司所领导的）。由于此前Oracle公司已经收购了另外一家大型的中间件企业BEA公司，在完成对Sun公司的收购之后，Oracle公司分别从BEA和Sun中取得了目前三大商业虚拟机的其中两个：JRockit和HotSpot，Oracle公司宣布在未来1～2年的时间内，将把这两个优秀的虚拟机互相取长补短，最终合二为一。可以预见在不久的将来，Java虚拟机技术将会产生相当巨大的变化。

　　2011年7月28日，Oracle公司发布Java SE 1.7

　　2014年3月18日，Oracle公司发表Java SE 1.8

　　Java语言有下面一些特点 :简单、面向对象、分布式、解释执行、鲁棒、安全、体系结构中立、可移植、高性能、多线程以及动态性。

## Java特点

Java语言的主要特点：
Java语言有下面一些特点 :简单、面向对象、分布式、解释执行、鲁棒、安全、体系结构中立、可移植、高性能、多线程以及动态性。

1.面向对象

Java语言的设计集中于对象及其接口 ,它提供了简单的类机制以及动态的接口模型。对象中封装了它的状态变量以及相应的方法 ,实现了模块化和信息隐藏 ;而类则提供了一类对象的原型 ,并且通过继承机制 ,子类可以使用父类所提供的方法 ,实现了代码的复用。

2.分布性

Java是面向网络的语言。通过它提供的类库可以处理 TCP/IP协议 ,用户 可以通过 URL地址在网络上很方便地访问其它对象。

3.简单性

Java语言是一种面向对象的语言 ,它通过提供最基本的方法来完成指定的任务 ,只需理解一些基本的概念 ,就可以用它编写出适合于各种情况的应用程序。 Java略去了运算符重载、多重继承等模糊的概念 ,并且通过实现自动垃圾收集大大简化了程序设计者的内存管理工作。另外 ,Java也适合于在小型机上运行 ,它的基本解释器及类的支持只有 40KB左右 ,加上标准类库和线程的支持也只有 215KB左右。库和线程的支持也只有 215KB左右。

4.鲁棒性

Java在编译和运行程序时 ,都要对可能出现的问题进行检查 ,以消除错误的产生。它提供自动垃圾收集来进行内存管理 ,防止程序员在管理内存时容易产生的错误。通过集成的面向对象的例外处理机制 ,在编译时,Java提示出可能出现但未被处理的例外 ,帮助程序员正确地进行选择以防止系统的崩溃。另外,Java在编译时还可捕获类型声明中的许多常见错误 ,防止动态运行时不匹配问题的出现。

5.可移植性

与平台无关的特性使 Java程序可以方便地被移植到网络上的不同机器。同时 ,Java的类库中也实现了与不同平台的接口 ,使这些类库可以移植。另外,Java编译器是由 Java语言实现的 ,Java运行时系统由标准 C实现 ,这使得Java系统本身也具有可移植性。

6.体系结构中立

Java解释器生成与体系结构无关的字节码指令 ,只要安装了 Java运行时系统 ,Java程序就可在任意的处理器上运行。这些字节码指令对应于 Java虚拟机中的表示 ,Java解释器得到字节码后 ,对它进行转换 ,使之能够在不同的平台运行。

7.安全性

用于网络、分布环境下的 Java必须要防止病毒的入侵。 Java不支持指针,一切对内存的访问都必须通过对象的实例变量来实现 ,这样就防止程序员使用"特洛伊 "木马等欺骗手段访问对象的私有成员 ,同时也避免了指针操作中容易产生的错误。

8.解释执行

Java解释器直接对 Java字节码进行解释执行。字节码本身携带了许多编译时信息 ,使得连接过程更加简单。

9.动态性

Java的设计使它适合于一个不断发展的环境。在类库中可以自由地加入新的方法和实例变量而不会影响用户程序的执行。并且 Java通过接口来支持多重继承 ,使之比严格的类继承具有更灵活的方式和扩展性。

10.多线程

多线程机制使应用程序能够并行执行 ,而且同步机制保证了对共享数据的正确操作。通过使用多线程 ,程序设计者可以分别用不同的线程完成特定的行为 ,而不需要采用全局的事件循环机制 ,这样就很容易地实现网络上的实时交互行为。

11.高性能

和其它解释执行的语言如 BASIC、 TCL不同 ,Java字节码的设计使之能很容易地直接转换成对应于特定CPU的机器码 ,从而得到较高的性能。

## Java基础

1. 标识符：就是给类，接口，方法，变量等起名字时使用的字符序列

   由英文大小写字母、数字字符、$（美元符号）、_（下划线）组成

   - 不能以数字开头
   - 不能是Java中的关键字
   - 区分大小写

2. 变量作用域和生存期：作用域从变量定义的位置开始，到该变量所在的那对大括号结束；生存期：从定义的位置开始直到它所在的作用域结束

3. 数据类型

   **基本数据类型**：byte、short、int、long、float、double、char、boolean

4. ## JDK与JRE

   SUN公司提供了一套Java开发环境，简称JDK(Java Development Kit)，它是整个Java的核心，其中包括Java编译器、Java运行工具、Java文档生成工具、Java打包工具等。

   为了满足用户日新月异的需求，JDK的版本也在不断地升级。在1995年，Java诞生之初就提供了最早的版本JDK1.0，随后相继推出了JDK1.1、JDK1.2、JDK1.3、JDK1.4、JDK5.0、JDK6.0、JDK7.0、JDK8.0

   SUN公司除了提供JDK，还提供了一种JRE(Java Runtime Environment)工具，它是Java运行环境，是提供给普通用户使用的。由于用户只需要运行事先编写好的程序，不需要自己动手编写程序，因此JRE工具中只包含Java运行工具，不包含Java编译工具。值得一提的是，为了方便使用，SUN公司在其JDK工具中自带了一个JRE工具，也就是说开发环境中包含运行环境，这样一来，开发人员只需要在计算机上安装JDK即可，不需要专门安装JRE工具了。

   

   **Java语言是跨平台的，而JVM不是跨平台的。**

5. [JDK、JRE、JVM的区别与联系](https://alleniverson.gitbooks.io/java-basic-introduction/content/%E7%AC%AC1%E7%AB%A0%20Java%E5%BC%80%E5%8F%91%E5%85%A5%E9%97%A8/JDK%E3%80%81JRE%E3%80%81JVM%E7%9A%84%E5%8C%BA%E5%88%AB%E4%B8%8E%E8%81%94%E7%B3%BB.html)

### 变量和常量

### 顺序和分支，循环

### 数组

## [JVM内存结构 VS Java内存模型 VS Java对象模型](https://blog.csdn.net/qq_36907589/article/details/80839385)


### 基本类型包装类

为了方便操作基本数据类型值，将其封装成了对象，在对象中定义了属性和行为丰富了该数据的操作。用于描述该对象的类就称为基本数据类型对象包装类。

- 将基本数据类型封装成对象的好处在于可以在对象中定义更多的功能方法操作该数据。
- 常用的操作之一：用于基本数据类型与字符串之间的转换。

| 基本数据类型 | 包装数据类型 |
| ------------ | ------------ |
| byte         | Byte         |
| short        | Short        |
| int          | Integer      |
| long         | Long         |
| float        | Float        |
| double       | Double       |
| char         | Character    |
| boolean      | Boolean      |

### BigInteger 

BigInteger：可以让超过Integer范围内的数据进行运算。超大整数相加的问题

| 方法声明                           | 功能描述           |
| ---------------------------------- | ------------------ |
| add(BigInteger val)                | 加                 |
| subtract(BigInteger val)           | 减                 |
| multiply(BigInteger val)           | 乘                 |
| divide(BigInteger val)             | 除                 |
| divideAndRemainder(BigInteger val) | 返回商和余数的数组 |
| BigInteger(String val)             | 构造方法           |

### BigDecimal

不可变的、任意精度的有符号十进制数。由于在运算的时候，float类型和double很容易丢失精度，演示案例。所以，为了能精确的表示、计算浮点数，Java提供了BigDecimal

| 方法声明                                              | 功能描述               |
| ----------------------------------------------------- | ---------------------- |
| BigDecimal(String val)                                | 构造方法               |
| add(BigDecimal augend)                                | 加                     |
| subtract(BigDecimal subtrahend)                       | 减                     |
| multiply(BigDecimal multiplicand)                     | 乘                     |
| divide(BigDecimal divisor)                            | 除                     |
| divide(BigDecimal divisor,int scale,int roundingMode) | 商、几位小数、如何取舍 |



## 面向对象：封装，继承，多态

继承，重写，重载，多态，抽象类，封装，接口

## 

## IO [详细参考](https://www.cnblogs.com/ylspace/p/8128112.html)

### Java中IO结构组成：流式部分：字节流和字符流；文件相关类：File, RandomAccessFile等；其他类



## 注解

元数据，即一种描述数据的数据。所以，可以说注解就是源代码的元数据。

### 按运行机制分

- 源码注解 只在源码中存在
- 编译时注解 在class中依然存在，如@Deprecated
- 运行时注解 运行阶段起作用，如@Autowired

### 按来源分

- JDK自带注解
- 三方注解 最常见
- 自定义注解

### 元注解

注解的注解

1.@Retention: 定义注解的保留策略
@Retention(RetentionPolicy.SOURCE)   //注解仅存在于源码中，在class字节码文件中不包含
@Retention(RetentionPolicy.CLASS)     // 默认的保留策略，注解会在class字节码文件中存在，但运行时无法获得，
@Retention(RetentionPolicy.RUNTIME)  // 注解会在class字节码文件中存在，在运行时可以通过反射获取到
首 先要明确生命周期长度 SOURCE < CLASS < RUNTIME ，所以前者能作用的地方后者一定也能作用。一般如果需要在运行时去动态获取注解信息，那只能用 RUNTIME 注解；如果要在编译时进行一些预处理操作，比如生成一些辅助代码（如 ButterKnife），就用 CLASS注解；如果只是做一些检查性的操作，比如 @Override 和 @SuppressWarnings，则可选用 SOURCE 注解。


2.@Target：定义注解的作用目标
源码为：
@Documented  
@Retention(RetentionPolicy.RUNTIME)  
@Target(ElementType.ANNOTATION_TYPE)  
public @interface Target {  
    ElementType[] value();  
}  
@Target(ElementType.TYPE)   //接口、类、枚举、注解
@Target(ElementType.FIELD) //字段、枚举的常量
@Target(ElementType.METHOD) //方法
@Target(ElementType.PARAMETER) //方法参数
@Target(ElementType.CONSTRUCTOR)  //构造函数
@Target(ElementType.LOCAL_VARIABLE)//局部变量
@Target(ElementType.ANNOTATION_TYPE)//注解
@Target(ElementType.PACKAGE) ///包    
3.@Document：说明该注解将被包含在javadoc中
4.@Inherited：说明子类可以继承父类中的该注解
举例：

## 网络编程 [总结参考](https://www.cnblogs.com/midiyu/p/7875574.html)



## 反射

JAVA反射机制是在运行状态中，对于任意一个实体类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意方法和属性；这种动态获取信息以及动态调用对象方法的功能称为java语言的反射机制。

JAVA反射（放射）机制：“程序运行时，允许改变程序结构或变量类型，这种语言称为[动态语言](https://baike.baidu.com/item/%E5%8A%A8%E6%80%81%E8%AF%AD%E8%A8%80/797407)”。从这个观点看，Perl，Python，Ruby是动态语言，C++，Java，C#不是动态语言。但是JAVA有着一个非常突出的动态相关机制：Reflection，用在Java身上指的是我们可以于运行时加载、探知、使用编译期间完全未知的classes。换句话说，Java程序可以加载一个运行时才得知名称的class，获悉其完整构造（但不包括methods定义），并生成其对象实体、或对其fields设值、或唤起其methods。



## 类加载机制

## 线程和并发

什么是进程？什么是线程？
进程：进程是并发执行程序在执行过程中资源分配和管理的基本单位（资源分配的最小单位）。进程可以理解为一个应用程序的执行过程，应用程序一旦执行，就是一个进程。每个进程都有自己独立的地址空间，每启动一个进程，系统就会为它分配地址空间，建立数据表来维护代码段、堆栈段和数据段。

线程：程序执行的最小单位。

 

为什么要有线程？
每个进程都有自己的地址空间，即进程空间，在网络或多用户换机下，一个服务器通常需要接收大量不确定数量用户的并发请求，为每一个请求都创建一个进程显然行不通（系统开销大响应用户请求效率低），因此操作系统中线程概念被引进。

 

 

进程与线程的区别？
1. 地址空间： 同一进程的所有线程共享本进程的地址空间，而不同的进程之间的地址空间是独立的。

2. 资源拥有： 同一进程的所有线程共享本进程的资源，如内存，CPU，IO等。进程之间的资源是独立的，无法共享。

3. 执行过程：每一个进程可以说就是一个可执行的应用程序，每一个独立的进程都有一个程序执行的入口，顺序执行序列。但是线程不能够独立执行，必须依存在应用程序中，由程序的多线程控制机制进行控制。

4. 健壮性： 因为同一进程的所以线程共享此线程的资源，因此当一个线程发生崩溃时，此进程也会发生崩溃。 但是各个进程之间的资源是独立的，因此当一个进程崩溃时，不会影响其他进程。因此进程比线程健壮。

 

线程执行开销小，但不利于资源的管理与保护。

进程的执行开销大，但可以进行资源的管理与保护。进程可以跨机器前移。

 

进程与线程的选择取决条件？
因为进程是资源分配的基本单位，线程是程序执行的最小单。以及进程与线程之间的健壮性来考虑。

1. 在程序中，如果需要频繁创建和销毁的使用线程。因为进程创建和销毁开销很大（需要不停的分配资源），但是线程频繁的调用只是改变CPU的执行，开销小。

2. 如果需要程序更加的稳定安全时，可以选择进程。如果追求速度，就选择线程。

### [Java中的多线程你只要看这一篇就够了](https://www.cnblogs.com/wxd0108/p/5479442.html)

### [Java线程状态](https://blog.csdn.net/ldx19980108/article/details/81675241)

NEW(初始)：线程被创建后尚未启动。
RUNNABLE(运行)：包括了操作系统线程状态中的Running和Ready，也就是处于此状态的线程可能正在运行，也可能正在等待系统资源，如等待CPU为它分配时间片。
BLOCKED(阻塞)：线程阻塞于锁。
WAITING(等待)：线程需要等待其他线程做出一些特定动作（通知或中断）。
TIME_WAITING(超时等待)：该状态不同于WAITING，它可以在指定的时间内自行返回。

TERMINATED(终止)：该线程已经执行完毕。

![img](https://img-blog.csdn.net/20180814203656288?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xkeDE5OTgwMTA4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### [线程同步的7种方式](http://www.cnblogs.com/XHJT/p/3897440.html) volatile 是不能达到同步效果的~

# 集合

## 集合与数组

**数组**（可以存储基本数据类型）是用来存现对象的一种容器，但是数组的长度固定，不适合在对象数量未知的情况下使用。

**集合**（只能存储对象，对象类型可以不一样）的长度可变，可在多数情况下使用。
注：数组我在前面的博客讲了大家可以看下

## 集合中接口和类的关系

**Collection**接口是集合类的根接口，Java中没有提供这个接口的直接的实现类。但是却让其被继承产生了两个接口，就是Set和List。Set中不能包含重复的元素。List是一个有序的集合，可以包含重复的元素，提供了按索引访问的方式。

**Map**是Java.util包中的另一个接口，它和Collection接口没有关系，是相互独立的，但是都属于集合类的一部分。Map包含了key-value对。Map不能包含重复的key，但是可以包含相同的value。

**Iterator**：所有的集合类，都实现了Iterator接口，这是一个用于遍历集合中元素的接口，主要包含以下三种方法：
1.**hasNext()**是否还有下一个元素。
2.**next()**返回下一个元素。
3.**remove()**删除当前元素。

### 层次图

图一这个比较简单

![这里写图片描述](http://img.blog.csdn.net/20170905084526091?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvTGl2ZW9yX0RpZQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

图二完整

![这里写图片描述](http://img.blog.csdn.net/20170905084554470?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvTGl2ZW9yX0RpZQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### list,set,map对比

| 接口       | 子接口      | 是否有序           | 是否允许元素重复                                          |
| ---------- | ----------- | ------------------ | --------------------------------------------------------- |
| Collection |             | 否                 |                                                           |
| List       | ArrayList   | 否                 | 是                                                        |
|            | LinkedList  | 否                 | 是                                                        |
|            | Vector      | 否                 | 是                                                        |
| Set        | AbstractSet | 否                 | 否                                                        |
|            | HashSet     | 否                 | 否                                                        |
|            | TreeSet     | 是（用二叉排序树） | 否                                                        |
| Map        | AbstractMap | 否                 | 使用key-value来映射和存储数据，key必须唯一，value可以重复 |
|            | HashMap     |                    | 否                                                        |
|            | TreeMap     | 是（用二叉排序树） | 使用key-value来映射和存储数据，key必须唯一，value可以重复 |

## list（有序、可重复）

### ArrayList

　　ArrayList是基于数组的，在初始化ArrayList时，会构建空数组（Object[] elementData={}）。ArrayList是一个无序的，它是按照添加的先后顺序排列，当然，他也提供了sort方法，如果需要对ArrayList进行排序，只需要调用这个方法，提供Comparator比较器即可

### LinkedList

　　LinkedList是基于链表的，它是一个双向链表，每个节点维护了一个prev和next指针。同时对于这个链表，维护了first和last指针，first指向第一个元素，last指向最后一个元素。LinkedList是一个无序的链表，按照插入的先后顺序排序，不提供sort方法对内部元素排序。

## Set（无序、不能重复）

### HashSet

　　HashSet是基于HashMap来实现的，操作很简单，更像是对HashMap做了一次“封装”，而且只使用了HashMap的key来实现各种特性，而HashMap的value始终都是PRESENT。

　　HashSet不允许重复（HashMap的key不允许重复，如果出现重复就覆盖），允许null值，非线程安全。

### TreeSet

　　基于 TreeMap 的 NavigableSet 实现。使用元素的自然顺序对元素进行排序，或者根据创建 set 时提供的 Comparator进行排序，具体取决于使用的构造方法。

## Map（键值对、键唯一、值不唯一）

　　Map集合中存储的是键值对，键不能重复，值可以重复。根据键得到值，对map集合遍历时先得到键的set集合，对set集合进行遍历，得到相应的值。

### HashMap

　　数组方式存储key/value，**线程非安全**，**允许null作为key和value**，key不可以重复，value允许重复，不保证元素迭代顺序是按照插入时的顺序，key的hash值是先计算key的hashcode值，然后再进行计算，每次容量扩容会重新计算所以key的hash值，会消耗资源，要求key必须重写equals和hashcode方法

　　默认初始容量16，加载因子0.75，扩容为旧容量乘2，查找元素快，如果key一样则比较value，如果value不一样，则按照链表结构存储value，就是一个key后面有多个value；

### Hashtable

　　Hashtable与HashMap类似，是HashMap的线程安全版，它支持线程的同步，即任一时刻只有一个线程能写Hashtable，因此也导致了Hashtale在写入时会比较慢，它继承自Dictionary类，不同的是它不允许记录的键或者值为null，同时效率较低。

### LinkedHashMap

LinkedHashMap保存了记录的插入顺序，在用Iteraor遍历LinkedHashMap时，先得到的记录肯定是先插入的，在遍历的时候会比HashMap慢，有HashMap的全部特性。

### TreeMap

　　基于红黑二叉树的NavigableMap的实现，线程非安全，不允许null，key不可以重复，value允许重复，存入TreeMap的元素应当实现Comparable接口或者实现Comparator接口，会按照排序后的顺序迭代元素，两个相比较的key不得抛出classCastException。主要用于存入元素的时候对元素进行自动排序，迭代输出的时候就按排序顺序输出

### Vector和ArrayList

　　1，vector是线程同步的，所以它也是线程安全的，而arraylist是线程异步的，是不安全的。如果不考虑到线程的安全因素，一般用arraylist效率比较高。

　　2，如果集合中的元素的数目大于目前集合数组的长度时，vector增长率为目前数组长度的100%，而arraylist增长率为目前数组长度的50%。如果在集合中使用数据量比较大的数据，用vector有一定的优势。

　　3，如果查找一个指定位置的数据，vector和arraylist使用的时间是相同的，如果频繁的访问数据，这个时候使用vector和arraylist都可以。而如果移动一个指定位置会导致后面的元素都发生移动，这个时候就应该考虑到使用linklist,因为它移动一个指定位置的数据时其它元素不移动。

　　ArrayList 和Vector是采用数组方式存储数据，此数组元素数大于实际存储的数据以便增加和插入元素，都允许直接序号索引元素，但是插入数据要涉及到数组元素移动等内存操作，所以索引数据快，插入数据慢，Vector由于使用了synchronized方法（线程安全）所以性能上比ArrayList要差，LinkedList使用双向链表实现存储，按序号索引数据需要进行向前或向后遍历，但是插入数据时只需要记录本项的前后项即可，所以插入数度较快。

### ArrayList和LinkedList

　　1.ArrayList是实现了基于动态数组的数据结构，LinkedList基于链表的数据结构。

　　2.对于随机访问get和set，ArrayList觉得优于LinkedList，因为LinkedList要移动指针。

　　3.对于新增和删除操作add和remove，LinedList比较占优势，因为ArrayList要移动数据。 这一点要看实际情况的。若只对单条数据插入或删除，ArrayList的速度反而优于LinkedList。但若是批量随机的插入删除数据，LinkedList的速度大大优于ArrayList. 因为ArrayList每插入一条数据，要移动插入点及之后的所有数据。

### HashMap与TreeMap

　　1、 HashMap通过hashcode对其内容进行快速查找，而TreeMap中所有的元素都保持着某种固定的顺序，如果你需要得到一个有序的结果你就应该使用TreeMap（HashMap中元素的排列顺序是不固定的）。

　　2、在Map 中插入、删除和定位元素，HashMap是最好的选择。但如果您要按自然顺序或自定义顺序遍历键，那么TreeMap会更好。使用HashMap要求添加的键类明确定义了hashCode()和 equals()的实现。

　　两个map中的元素一样，但顺序不一样，导致hashCode()不一样。

　　同样做测试：
　　　　在HashMap中，同样的值的map,顺序不同，equals时，false;
　　　　而在treeMap中，同样的值的map,顺序不同,equals时，true，说明，treeMap在equals()时是整理了顺序了的。

### HashTable与HashMap

　　1、同步性:Hashtable是线程安全的，也就是说是同步的，而HashMap是线程序不安全的，不是同步的。

　　2、HashMap允许存在一个为null的key，多个为null的value 。

　　3、hashtable的key和value都不允许为null。

## 线程安全的集合类

vector：就比arraylist多了个同步化机制（线程安全），因为效率较低，现在已经不太建议使用。在web应用中，特别是前台页面，往往效率（页面响应速度）是优先考虑的。

statck：堆栈类，先进后出

hashtable：就比hashmap多了个线程安全

enumeration：枚举，相当于迭代器

除了这些之外，其他的都是非线程安全的类和接口。

线程安全的类其方法是同步的，每次只能一个访问。是重量级对象，效率较低。

其他：

\1. hashtable跟hashmap的区别

hashtable是线程安全的,即hashtable的方法都提供了同步机制；hashmap不是线程安全的,即不提供同步机制 ；hashtable不允许插入空值,hashmap允许!



## 



## 更多问题



## 面试题

### [Java基础总结大全（实用）](https://www.cnblogs.com/javastu/p/5519569.html)

### [Java基础总结！精华版！](https://blog.csdn.net/Song_JiangTao/article/details/80642188)

