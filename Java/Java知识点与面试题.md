[Java基础知识总结](https://www.cnblogs.com/chenhao0302/p/7125874.html)





## JDK与JRE

SUN公司提供了一套Java开发环境，简称JDK(Java Development Kit)，它是整个Java的核心，其中包括Java编译器、Java运行工具、Java文档生成工具、Java打包工具等。

为了满足用户日新月异的需求，JDK的版本也在不断地升级。在1995年，Java诞生之初就提供了最早的版本JDK1.0，随后相继推出了JDK1.1、JDK1.2、JDK1.3、JDK1.4、JDK5.0、JDK6.0、JDK7.0、JDK8.0

SUN公司除了提供JDK，还提供了一种JRE(Java Runtime Environment)工具，它是Java运行环境，是提供给普通用户使用的。由于用户只需要运行事先编写好的程序，不需要自己动手编写程序，因此JRE工具中只包含Java运行工具，不包含Java编译工具。值得一提的是，为了方便使用，SUN公司在其JDK工具中自带了一个JRE工具，也就是说开发环境中包含运行环境，这样一来，开发人员只需要在计算机上安装JDK即可，不需要专门安装JRE工具了。



**Java语言是跨平台的，而JVM不是跨平台的。**

## [JDK、JRE、JVM的区别与联系](https://alleniverson.gitbooks.io/java-basic-introduction/content/%E7%AC%AC1%E7%AB%A0%20Java%E5%BC%80%E5%8F%91%E5%85%A5%E9%97%A8/JDK%E3%80%81JRE%E3%80%81JVM%E7%9A%84%E5%8C%BA%E5%88%AB%E4%B8%8E%E8%81%94%E7%B3%BB.html)



## Java类初始化流程

初始化顺序：

**父类静态成员变量**
**子类静态成员变量**
**子类Main开始执行**
**父类静态代码块**

**父类构造代码块**

**父类成员变量**
**父类成员变量**
**父类的构造方法**
**子类静态代码块**

**子类构造代码块**

**子类成员变量**
**子类成员变量**
**子类的构造方法执行**

```java
public class Zi extends Fu {

    private B b = new B();
    private int k = printInit("子类成员变量k初始化");


    public Zi() {
        System.out.println("子类的构造方法执行 " + "k = " + k + "  " + "j=" + j);
    }

    private static int x2 = printInit("子类静态成员变量x2初始化");

    public static void main(String[] args) {
        System.out.println("子类Main开始执行");
        new Zi();
    }

}

class A {
    A() {
        System.out.println("父类成员变量a初始化");
    }

    {
        System.out.println("父类构造代码块");
    }

    static {
        System.out.println("父类静态代码块");
    }
}

class B {
    B() {
        System.out.println("子类成员变量b初始化");
    }

    {
        System.out.println("子类构造代码块");
    }

    static {
        System.out.println("子类静态代码块");
    }
}

class Fu {

    private A a = new A();
    private int i = printInit("父类成员变量i初始化");
    protected int j;

    Fu() {
        System.out.println("父类的构造方法执行 " + "j = " + j);
        j = 99;
    }

    private static int x1 = printInit("父类静态成员变量x1初始化");

    static int printInit(String s) {
        System.out.println(s);
        return 9;
    }

    //虽然它是静态方法,但是不去调用不会自动执行
    static void Sup() {
        System.out.println("AAAAAA");
    }
}

```

## Java类加载机制

[jvm之java类加载机制和类加载器(ClassLoader)的详解](https://blog.csdn.net/m0_38075425/article/details/81627349)

[深入探讨Java类加载机制](https://www.cnblogs.com/xdouby/p/5829423.html)



引导类加载器

扩展类加载器

系统类加载器

自定义类加载器



**双亲委派机制：**

**JVM在加载类时默认采用的是双亲委派机制。通俗的讲，就是某个特定的类加载器在接到加载类的请求时，首先将加载任务委托给父加载器，依次递归，如果父加载器可以完成类加载任务，就成功返回；只有父加载器无法完成此加载任务时，才自己去加载。**



## [Java 虚拟机](https://blog.csdn.net/qq_41701956/article/details/81664921)



## GC是在什么时候，对什么东西，做了什么事情？

- 什么时候

　　从字面上翻译过来就是什么时候触发我们的GC机制

　　①在程序空闲的时候。这个回答无力吐槽

　　②程序不可预知的时候/手动调用system.gc()。关于手动调用不推荐

　　③Java堆内存不足时,GC会被调用。当应用线程在运行,并在运行过程中创建新对象,若这时内存空间不足,JVM就会强制地调用GC线程,以便回收内存用于新的分配。若GC一次之后仍不能满足内存分配的要求,JVM会再进行两次GC作进一步的尝试,若仍无法满足要求,则 JVM将报“out of memory”的错误,Java应用将停止。就是

　　这时候如果你们讲出新生代和老年代的话或许会更细的了解一下Minor GC、Full GC、OOM什么时候触发！

　　创建对象是新生代的Eden空间调用Minor GC；当升到老年代的对象大于老年代剩余空间Full GC；GC与非GC时间耗时超过了GCTimeRatio的限制引发OOM。

- 什么东西

　　从字面的意思翻译过来就是能被GC回收的对象都有哪些特征

　　①超出作用域的对象/引用计数为空的对象。

　　引用计数算法：给对象中添加一个引用计数器，每当有一个地方引用它时，计数器就加1；当引用失效时，计数器值就减1；任何时刻计数器都为0的对象就是不可能再被使用的。

　　②从GC Root开始搜索，且搜索不到的对象

　　跟搜索算法：以一系列名为 GC Root的对象作为起点，从这些节点开始往下搜索，搜索走过的路径称为引用链，当一个对象到GC Roots没有任何引用链的时候，则就证明此对象是不可用的。

　　这里会提出一个思考，什么样的对象能成为GC Root ： 虚拟机中的引用的对象、方法区中的类静态属性引用的对象、方法区中常量引用的对象、本地方法栈中jni的引用对象。

　　③从root搜索不到，而且经过第一次标记、清理后，仍然没有复活的对象。

- 做什么

　　不同年代、不同种类的收集器很多，不过总体的作用是删除不使用的对象，腾出内存空间。补充一些诸如停止其他线程执行、运行finalize等的说明。

ok  现在来回答一下我们最上面的问题，上面时候容易发生内存泄露

　　①静态集合类像HashMap、Vector等

　　②各种连接，数据库连接，网络连接，IO连接等没有显示调用close关闭，不被GC回收导致内存泄露。

　　③监听器的使用，在释放对象的同时没有相应删除监听器的时候也可能导致内存泄露。



## 元空间

**JDK7之前，**字符串常量池被存储在永久代（默认大小是4m）中，因此导致性能问题和OOM；实际上1.7时，存储在永久代的部分数据就已经转移到了Java Heap或者是 Native Heap；

**JDK8已经直接取消了Perm区域，以元空间替代**

由于类的元数据分配在本地内存中，元空间的最大可分配空间就是系统可用内存空间。

元空间虚拟机采用了组块分配的形式，会导致内存碎片存在。

JDK 8 中永久代向元空间的转换的原因：

- 1、字符串存在永久代中，容易出现性能问题和内存溢出。
- 2、类及方法的信息等比较难确定其大小，因此对于永久代的大小指定比较困难，太小容易出现永久代溢出，太大则容易导致老年代溢出。
- 3、永久代会为 GC 带来不必要的复杂度，并且回收效率偏低。
- 4、Oracle 可能会将HotSpot 与 JRockit 合二为一。



## [JVM的新生代、老年代、MinorGC、MajorGC](https://www.cnblogs.com/ygj0930/p/6522828.html)

![img](https://images2015.cnblogs.com/blog/1018541/201703/1018541-20170308200940484-1226739905.jpg)



新生代：主要是用来存放新生的对象。一般占据堆的1/3空间。由于频繁创建对象，所以新生代会频繁触发MinorGC进行垃圾回收。

​         新生代又分为 Eden区、ServivorFrom、ServivorTo三个区。

​         Eden区：Java新对象的出生地（如果新创建的对象占用内存很大，则直接分配到老年代）。当Eden区内存不够的时候就会触发MinorGC，对新生代区进行一次垃圾回收。

​         ServivorTo：保留了一次MinorGC过程中的幸存者。

​         ServivorFrom：上一次GC的幸存者，作为这一次GC的被扫描者。

​         MinorGC的过程：MinorGC采用复制算法。首先，把Eden和ServivorFrom区域中存活的对象复制到ServicorTo区域（如果有对象的年龄以及达到了老年的标准，则赋值到老年代区），同时把这些对象的年龄+1（如果ServicorTo不够位置了就放到老年区）；然后，清空Eden和ServicorFrom中的对象；最后，ServicorTo和ServicorFrom互换，原ServicorTo成为下一次GC时的ServicorFrom区。

​    ![img](https://images2015.cnblogs.com/blog/1018541/201703/1018541-20170308194923281-651017951.jpg)



  二：老年代：主要存放应用程序中生命周期长的内存对象。

​    老年代的对象比较稳定，所以MajorGC不会频繁执行。在进行MajorGC前一般都先进行了一次MinorGC，使得有新生代的对象晋身入老年代，导致空间不够用时才触发。当无法找到足够大的连续空间分配给新创建的较大对象时也会提前触发一次MajorGC进行垃圾回收腾出空间。

​    MajorGC采用标记—清除算法：首先扫描一次所有老年代，标记出存活的对象，然后回收没有标记的对象。MajorGC的耗时比较长，因为要扫描再回收。MajorGC会产生内存碎片，为了减少内存损耗，我们一般需要进行合并或者标记出来方便下次直接分配。

​     当老年代也满了装不下的时候，就会抛出OOM（Out of Memory）异常。



   三：永久代

​    指内存的永久保存区域，主要存放Class和Meta（元数据）的信息,Class在被加载的时候被放入永久区域. 它和和存放实例的区域不同,GC不会在主程序运行期对永久区域进行清理。所以这也导致了永久代的区域会随着加载的Class的增多而胀满，最终抛出OOM异常。

​    在Java8中，永久代已经被移除，被一个称为“元数据区”（元空间）的区域所取代。

​    元空间的本质和永久代类似，都是对JVM规范中方法区的实现。不过元空间与永久代之间最大的区别在于：元空间并不在虚拟机中，而是使用本地内存。因此，默认情况下，元空间的大小仅受本地内存限制。类的元数据放入 native memory, 字符串池和类的静态变量放入java堆中. 这样可以加载多少类的元数据就不再由MaxPermSize控制, 而由系统的实际可用空间来控制.

​    采用元空间而不用永久代的几点原因：（参考：http://www.cnblogs.com/paddix/p/5309550.html）

　　1、为了解决永久代的OOM问题，元数据和class对象存在永久代中，容易出现性能问题和内存溢出。

　　2、类及方法的信息等比较难确定其大小，因此对于永久代的大小指定比较困难，太小容易出现永久代溢出，太大则容易导致老年代溢出（因为堆空间有限，此消彼长）。

　　3、永久代会为 GC 带来不必要的复杂度，并且回收效率偏低。

　　4、Oracle 可能会将HotSpot 与 JRockit 合二为一。





