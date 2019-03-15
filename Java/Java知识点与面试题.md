[Java基础知识总结](https://www.cnblogs.com/chenhao0302/p/7125874.html)





1. ## JDK与JRE

   SUN公司提供了一套Java开发环境，简称JDK(Java Development Kit)，它是整个Java的核心，其中包括Java编译器、Java运行工具、Java文档生成工具、Java打包工具等。

   为了满足用户日新月异的需求，JDK的版本也在不断地升级。在1995年，Java诞生之初就提供了最早的版本JDK1.0，随后相继推出了JDK1.1、JDK1.2、JDK1.3、JDK1.4、JDK5.0、JDK6.0、JDK7.0、JDK8.0

   SUN公司除了提供JDK，还提供了一种JRE(Java Runtime Environment)工具，它是Java运行环境，是提供给普通用户使用的。由于用户只需要运行事先编写好的程序，不需要自己动手编写程序，因此JRE工具中只包含Java运行工具，不包含Java编译工具。值得一提的是，为了方便使用，SUN公司在其JDK工具中自带了一个JRE工具，也就是说开发环境中包含运行环境，这样一来，开发人员只需要在计算机上安装JDK即可，不需要专门安装JRE工具了。

   

   **Java语言是跨平台的，而JVM不是跨平台的。**

2. [JDK、JRE、JVM的区别与联系](https://alleniverson.gitbooks.io/java-basic-introduction/content/%E7%AC%AC1%E7%AB%A0%20Java%E5%BC%80%E5%8F%91%E5%85%A5%E9%97%A8/JDK%E3%80%81JRE%E3%80%81JVM%E7%9A%84%E5%8C%BA%E5%88%AB%E4%B8%8E%E8%81%94%E7%B3%BB.html)



3. Java类初始化流程

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

   4. Java类加载机制

   [jvm之java类加载机制和类加载器(ClassLoader)的详解](https://blog.csdn.net/m0_38075425/article/details/81627349)

   [深入探讨Java类加载机制](https://www.cnblogs.com/xdouby/p/5829423.html)

   

   引导类加载器

   扩展类加载器

   系统类加载器

   自定义类加载器

   

   **双亲委派机制：**

   **JVM在加载类时默认采用的是双亲委派机制。通俗的讲，就是某个特定的类加载器在接到加载类的请求时，首先将加载任务委托给父加载器，依次递归，如果父加载器可以完成类加载任务，就成功返回；只有父加载器无法完成此加载任务时，才自己去加载。**

   

   5. [Java 虚拟机](https://blog.csdn.net/qq_41701956/article/details/81664921)



