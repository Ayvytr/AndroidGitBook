# Java

## Java特点

1.简单 

Java最初是为对家用电器进行集成控制而设计的一种语言，因此它必须简单明了。Java语言的简单性主要体现在以下三个方面： 

1) Java的风格类似于C++，因而C++程序员是非常熟悉的。从某种意义上讲，Java语言是C及C++语言的一个变种，因此，C++程序员可以很快就掌握Java编程技术。 

2) Java摒弃了C++中容易引发程序错误的地方，如指针和内存管理。 

3) Java提供了丰富的类库。 

2.面向对象 

面向对象可以说是Java最重要的特性。Java语言的设计完全是面向对象的，它不支持类似C语言那样的面向过程的程序设计技术。Java支持静态和动态风格的代码继承及重用。单从面向对象的特性来看，Java类似于Small Talk，但其它特性、尤其是适用于分布式计算环境的特性远远超越了Small Talk。 

3.分布式 

Java包括一个支持HTTP和FTP等基于TCP/IP协议的子库。因此，Java应用程序可凭借URL打开并访问网络上的对象，其访问方式与访问本地文件系统几乎完全相同。为分布环境尤其是Internet提供的动态内容无疑是一项非常宏伟的任务，但Java的语法特性却使我们很容易地实现这项目标。 

4.健壮 

Java致力于检查程序在编译和运行时的错误。类型检查帮助检查出许多开发早期出现的错误。Java自已操纵内存减少了内存出错的可能性。Java还实现了真数组，避免了覆盖数据的可能。这些功能特征大大缩短了开发Java应用程序的周期。Java提供Null指针检测数组边界检测异常出口字节代码校验。 

5.结构中立 

另外，为了建立Java作为网络的一个整体，Java将它的程序编译成一种结构中立的中间文件格式。只要有Java运行系统的机器都能执行这种中间代码。现在，Java运行系统有Solaris2.4(SPARC),Win32系统(Windows95和WindowsNT)等.Java源程序被编译成一种高层次的与机器无关的byte-code格式语言，这种语言被设计在虚拟机上运行，由机器相关的运行调试器实现执行。 

6.安全 

Java的安全性可从两个方面得到保证。一方面，在Java语言里，象指针和释放内存等C++功能被删除，避免了非法内存操作。另一方面，当Java用来创建浏览器时，语言功能和浏览器本身提供的功能结合起来，使它更安全。Java语言在你的机器上执行前，要经过很多次的测试。它经过代码校验，检查代码段的格式，检测指针操作，对象操作是否过分以及试图改变一个对象的类型。 

7.可移植的 

这句话一直是Java程序设计师们的精神指标，也是Java之所以能够受到程序设计师们喜爱的原因之一，最大的功臣就是JVM的技术。大多数编译器产生的目标代码只能运行在一 种CPU上(如Intel的x86系列)，即使那些能支持多种CPU的编译器也不能同时产生适合多 种CPU的目标代码。如果你需要在三种CPU( 如x86、SPARC 和MIPS)上运行同一程序, 就必须编译三次。 

但JAVA编译器就不同了。JAVA编译器产生的目标代码(J-Code) 是针对一种并不 存在的CPU--JAVA虚拟机(JAVA Virtual Machine)，而不是某一实际的CPU。JAVA虚拟机能掩盖不同CPU之间的差别，使J-Code能运行于任何具有JAVA虚拟机的机器上。 

虚拟机的概念并不AVA 所 特 有 的：加州大学几年前就提出了PASCAL虚拟机的概念；广泛用于Unix服务器的Perl脚本也是产生与机器无关的中间代码用于执行。但针对Internet应用而设计的JAVA虚拟机的特别之处在于它能产生安全的不受病毒威胁的目标代码。正是由于Internet对安全特性的特别要求才使得JVM能够迅速被人们接受。 当今主 流的操作系统如OS/2、MacOS、Windows95/NT都已经或很快提供对J-Code的支持。 

作为一种虚拟的CPU，JAVA 虚拟机对于源代码(Source Code) 来说是独立的。我们不仅可以用JAVA语言来生成J-Code，也可以用Ada95来生成。事实上，已经有了针对若干种源代码的J-Code 编译器，包括Basic、Lisp 和Forth。源代码一经转换成J-Code以后，JAVA虚拟机就能够执行而不区分它是由哪种源代码生成的。这样做的结果就是CPU可移植性。 将源程序编译为J-Code的好处在于可运行于各种机器上，而缺点是它不如本机代码运行的速度快。 

同体系结构无关的特性使得Java应用程序可以在配备了Java解释器和运行环境的任何计算机系统上运行，这成为Java应用软件便于移植的良好基础。但仅仅如此还不够。如果基本数据类型设计依赖于具体实现，也将为程序的移植带来很大不便。例如在Windows3.1中整数(Integer)为16bits，在Windows95中整数为32bits，在DECAlpha中整数为64bits，在Intel486中为32bits。通过定义独立于平台的基本数据类型及其运算，Java数据得以在任何硬件平台上保持一致。Java语言的基本数据类型及其表示方式如下：byte8-bit二进制补码short16-bit二进制补码int32-bit二进制补码long64-bit二进制补码float32-bitIEEE754浮点数double32-bitIEEE754浮点数char16-bitUnicode字符在任何Java解释器中，数据类型都是依据以上标准具体实现的。因为几乎目前使用的所有CPU都能支持以上数据类型、8～64位整数格式的补码运算和单/双精度浮点运算。Java编译器本身就是用Java语言编写的。Java运算系统的编制依据POSIX方便移植的限制，用ANSIC语言写成。Java语言规范中也没有任何"同具体实现相关"的内容。 

8.解释的 

Java解释器(运行系统)能直接运行目标代码指令。链接程序通常比编译程序所需资源少，所以程序员可以在创建源程序上花上更多的时间。 

9.高性能 

如果解释器速度不慢，Java可以在运行时直接将目标代码翻译成机器指令。Sun用直接解释器一秒钟内可调用300,000个过程。翻译目标代码的速度与C/C++的性能没什么区别。 

10.多线程 

多线程功能使得在一个程序里可同时执行多个小任务。线程－－有时也称小进程－－是一个大进程里分出来的小的独立的进程。因为Java实现的多线程技术，所以比C和C++更键壮。多线程带来的更大的好处是更好的交互性能和实时控制性能。当然实时控制性能还取决于系统本身(UNIX,Windows,Macintosh等)，在开发难易程度和性能上都比单线程要好。任何用过当前浏览器的人，都感觉为调一副图片而等待是一件很烦恼的事情。在Java里，你可用一个单线程来调一副图片，而你可以访问HTML里的其它信息而不必等它。 

11.动态 

Java的动态特性是其面向对象设计方法的发展。它允许程序动态地装入运行过程中所需要的类，这是C++语言进行面向对象程序设计所无法实现的。在C++程序设计过程中，每当在类中增加一个实例变量或一种成员函数后，引用该类的所有子类都必须重新编译，否则将导致程序崩溃。Java从如下几方面采取措来解决这个问题。Java编译器不是将对实例变量和成员函数的引用编译为数值引用，而是将符号引用信息在字节码中保存下传递给解释器，再由解释器在完成动态连接类后，将符号引用信息转换为数值偏移量。这样，一个在存储器生成的对象不在编译过程中决定，而是延迟到运行时由解释器确定的。这样，对类中的变量和方法进行更新时就不至于影响现存的代码。解释执行字节码时，这种符号信息的查找和转换过程仅在一个新的名字出现时才进行一次，随后代码便可以全速执行。在运行时确定引用的好处是可以使用已被更新的类，而不必担心会影响原有的代码。如果程序连接了网络中另一系统中的某一类，该类的所有者也可以自由地对该类进行更新，而不会使任何引用该类的程序崩溃。Java还简化了使用一个升级的或全新的协议的方法。如果你的系统运行Java程序时遇到了不知怎样处理的程序，没关系，Java能自动下载你所需要的功能程序。四.与C和C++语言的异同 Java提供了一个功能强大语言的所有功能，但几乎没有一点含混特征。C++安全性不好，但C和C++还是被大家所接受，所以Java设计成C++形式，让大家很容易学习。Java去掉了C++语言的许多功能，让Java的语言功能很精炼，并增加了一个很有用的功能，Java去掉了以下几个C和C++功能和特征：指针运算结构typedefs#define需要释放内存全局变量的定义这个功能都是很容易引起错误的地方。 

12. Unicode 

Java使用Unicode作为它的标准字符，这项特性使得Java的程序能在不同语言的平台上都能撰写和执行。简单的说，你可以把程序中的变量、类别名称使用中文来表示<注>，当你的程序移植到其它语言平台时，还是可以正常的执行。Java也是目前所有计算机语言当中，唯一天生使用Unicode的语言。

## Java基础

### 单根继承结构



在Java中（事实上还包括除C++以外的所有OOP语言），所有的类最终都继承自单一的基类，这个终极基类的名字就是Object。 
  单根继承结构的好处：

在单根继承结构中所有对象都具有一个共用接口，所以它们归根到底都是相同的基本类型。
单根继承结构保证所有对象都具备某些功能。

单根继承结构使垃圾回收器的实现变得容易得多，而垃圾回收器正是相对C++的重要改进之一。由于所有对象都保证具有其类型信息，因此不会因无法确定对象的类型而陷入僵尸。这对于系统级操作（如异常处理）显得尤其重要，并且给编程带来了更大的灵活性。



### 内部类和静态内部类

Java普通内部类能访问其外围对象（enclosing object）的所有成员，而不需要任何特殊条件 。C++嵌套类的设计只是单纯的名字隐藏机制，与外围对象没有联系，也没有隐含的访问权。在Java中，当某个类创建一个内部类对象时，此内部类对象必定会秘密地捕获一个指向那个外围类的对象的引用。然后，在你访问此外围类的成员时，就是用那个引用来选择外围类的成员。这些细节是由编译器处理的。Java的迭代器复用了这个特性。

内部类声明为`static`时，**不再包含外围对象的引用.this**，称为**嵌套类**（与C++嵌套类大致相似，只不过在C++中那些类不能访问私有成员，而在Java中可以访问）。 
- 创建嵌套类，不需要外围对象。 
- 不能从嵌套类的对象中访问**非静态的外围对象**。



**闭包就是能够读取其他函数内部变量的函数。闭包可以理解成“定义在一个[函数](https://baike.baidu.com/item/%E5%87%BD%E6%95%B0/301912)内部的函数“。在本质上，闭包是将函数内部和函数外部连接起来的桥梁。Java中的闭包：内部类，局部内部类，匿名内部类**



1. 标识符：就是给类，接口，方法，变量等起名字时使用的字符序列

   由英文大小写字母、数字字符、$（美元符号）、_（下划线）组成

   - 不能以数字开头
   - 不能是Java中的关键字
   - 区分大小写

2. 变量作用域和生存期：作用域从变量定义的位置开始，到该变量所在的那对大括号结束；生存期：从定义的位置开始直到它所在的作用域结束

3. 基本类型和包装类

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



### 变量和常量

### 顺序和分支，循环

### 数组

## [JVM内存结构 VS Java内存模型 VS Java对象模型](https://blog.csdn.net/qq_36907589/article/details/80839385)


### 

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



## 面向对象

### 三大特征

1、封装

　　封装就是隐藏对象的属性和实现细节，仅对外公开接口，控制在程序中属性的读和修改的访问级别，将抽象得到的数据和行为（或功能）相结合，**形成一个有机的整体**，也就是将数据与操作数据的源代码进行有机的结合，形成“类”，其中数据和函数都是类的成员。

　　封装的目的是增强安全性和简化编程，使用者不必了解具体的实现细节，而只是要通过外部接口，以特定的访问权限来使用类的成员。

2、继承

　　继承是面向对象的基本特征之一，继承机制允许创建分等级层次的类。**继承就是子类继承父类的特征和行为**，使得子类对象（实例）具有父类的实例域和方法，或子类从父类继承方法，使得子类具有父类相同的行为。

3、多态 

　　多态同一个行为具有多个不同表现形式或形态的能力。是指一个类实例（对象）的相同方法在不同情形有不同表现形式。多态机制使具有不同内部结构的对象可以共享相同的外部接口。这意味着，虽然针对不同对象的具体操作不同，但通过一个公共的类，它们（那些操作）可以通过相同的方式予以调用。

**Java提供两种多态机制：重载与重写**

### 五大基本原则

1、单一职责原则（SRP）

　　一个类应该有且只有一个去改变它的理由，这意味着一个类应该只有一项工作。

　　比如在职员类里，将工程师、销售人员、销售经理这些情况都放在职员类里考虑，其结果将会非常混乱，在这个假设下，职员类里的每个方法都要if else判断是哪种情况，从类结构上来说将会十分臃肿。

2、开放封闭原则（OCP）

　　对象或实体应该对扩展开放，对修改封闭。

　　更改封闭即是在我们对模块进行扩展时，勿需对源有程序代码和DLL进行修改或重新编译文件！这个原则对我们在设计类的时候很有帮助，坚持这个原则就必须尽量考虑接口封装，抽象机制和多态技术！

3、里氏替换原则（LSP）

　　在对象 x 为类型 T 时 q(x) 成立，那么当 S 是 T 的子类时，对象 y 为类型 S 时 q(y) 也应成立。（即对父类的调用同样适用于子类）

4、依赖倒置原则（DIP）

　　高层次的模块不应该依赖于低层次的模块，他们都应该依赖于抽象。具体实现应该依赖于抽象，而不是抽象依赖于实现。

5、接口隔离原则（ISP）

　　不应强迫客户端实现一个它用不上的接口，或是说客户端不应该被迫依赖它们不使用的方法，使用多个专门的接口比使用单个接口要好的多！

　　比如，为了减少接口的定义，将许多类似的方法都放在一个接口中，最后会发现，维护和实现接口的时候花了太多精力，而接口所定义的操作相当于对客户端的一种承诺，这种承诺当然是越少越好，越精练越好，过多的承诺带来的就是你的大量精力和时间去维护！



**多态的优点：**

消除类型之间的耦合关系

可替换性

可扩充性

接口性

灵活性

简化性



**多态存在的三个必要条件：**

- 继承
- 重写（子类继承父类后对父类方法进行重新定义）
- 父类引用指向子类对象

　　简言之，**多态其实是在继承的基础上的。**



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

在Java中，**任何基本类型都不能作为类型参数**。因此**不能创建ArrayList<int> 或 HashMap<int, int>之类的东西**。但是可以利用自动包装机制和基本类型的包装器来解决，**自动包装机制将自动地实现int 到 Integer的双向转换**。

```java
// ListOfInt.java
import java.util.*;
public class ListOfInt{
    public static void main(String[] args){
        // 编译错误：意外的类型
        // List<int> li = new ArrayList<int>();
        // Map<int, Interger> m = new HashMap<int, Integer>();
        List<Integer> li = new ArrayList<Integer>();
        for(int i = 0; i < 5; i++){
            li.add(i);      // int --> Integer
        }
        for(int i : li){    // Integer --> int
            System.out.print(i + " ");
        }
    }
}/* Output:
0 1 2 3 4
*/

```



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

![img](http://www.superalloy.biz/Container.jpg)

### List,Set,Map对比

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

## List（有序、可重复）

### ArrayList

　　ArrayList是基于数组的，在初始化ArrayList时，会构建空数组（Object[] elementData={}）。ArrayList是一个无序的，它是按照添加的先后顺序排列，当然，他也提供了sort方法，如果需要对ArrayList进行排序，只需要调用这个方法，提供Comparator比较器即可

### LinkedList

　　LinkedList是基于链表的，它是一个双向链表，每个节点维护了一个prev和next指针。同时对于这个链表，维护了first和last指针，first指向第一个元素，last指向最后一个元素。LinkedList是一个无序的链表，按照插入的先后顺序排序，不提供sort方法对内部元素排序。

**ArrayList 和 LinkedList对比：**
都可自动扩容。
ArrayList底层是数组结构，即连续存储空间，所以读取元素快。因可自动扩容，所以可以把ArrayList当作“可自动扩充自身尺寸的数组”看待。
LinkedList是链表结构，所以插入元素快。 
LinkedList具有能够直接实现栈（Stack）的所有功能的方法，因此可以直接将LinkedList作为栈使用。LinkdedList也提供了支持队列（Queue）行为的方法，并且实现了Queue接口，所以也可以用作Queue。



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

[HashTable和HashMap的区别详解](https://www.cnblogs.com/williamjie/p/9099141.html)

[HashMap、Hashtable、ConcurrentHashMap的原理与区别](https://www.cnblogs.com/heyonggang/p/9112731.html)

hashtable是线程安全的,即hashtable的方法都提供了同步机制；hashmap不是线程安全的,即不提供同步机制 ；hashtable不允许插入空值,hashmap允许!



## 异常 [详细参考](https://blog.csdn.net/zhanaolu4821/article/details/81012382)

Java分为两类：Error和Exception

1、Error（错误）：是程序中无法处理的错误，表示运行应用程序中出现了严重的错误。此类错误一般表示代码运行时JVM出现问题。通常有Virtual MachineError（虚拟机运行错误）、NoClassDefFoundError（类定义错误）等。比如说当jvm耗完可用内存时，将出现OutOfMemoryError。此类错误发生时，JVM将终止线程。

这些错误是不可查的，非代码性错误。因此，当此类错误发生时，应用不应该去处理此类错误。

2、Exception（异常）：程序本身可以捕获并且可以处理的异常。

Exception又分为两类：运行时异常和编译异常。

1、运行时异常(不受检异常)：RuntimeException类极其子类表示JVM在运行期间可能出现的错误。比如说试图使用空值对象的引用（NullPointerException）、数组下标越界（ArrayIndexOutBoundException）。此类异常属于不可查异常，一般是由程序逻辑错误引起的，在程序中可以选择捕获处理，也可以不处理。

2、编译异常(受检异常)：Exception中除RuntimeException极其子类之外的异常。如果程序中出现此类异常，比如说IOException，必须对该异常进行处理，否则编译不通过。在程序中，通常不会自定义该类异常，而是直接使用系统提供的异常类。

## 字符串

**String对象是不可变的**

### [String，StringBuilder，StringBuffer三者的区别](https://www.cnblogs.com/su-feng/p/6659064.html)

1. 执行速度，**在这方面运行速度快慢为：StringBuilder > StringBuffer > String**
2. **在线程安全上，StringBuilder是线程不安全的，而StringBuffer是线程安全的**

**String：适用于少量的字符串操作的情况**

**StringBuilder：适用于单线程下在字符缓冲区进行大量操作的情况**

**StringBuffer：适用多线程下在字符缓冲区进行大量操作的情况**


