## 指定Java调用的类名

要用到注解 @file:JvmName(),放在package之前——Hello就是Java中可以调用的类名

```
@file:JvmName("Hello")
package kotlin
```



### @file:JvmName("Utils") 和 @file:JvmMultifileClass 一起使用的场景：

```
//在需要合并的每个Kotlin文件加入如下属性：
@file:JvmName("Utils")
@file:JvmMultifileClass
```

```
demo:

// oldutils.kt
@file:JvmName("Utils")
@file:JvmMultifileClass
package demo

fun foo() {
}

//___________________________________________________________
// newutils.kt
@file:JvmName("Utils")
@file:JvmMultifileClass
package demo
fun bar() {
}

//__________________________________________________________
// build后，在Java文件中的调用方法：
Utils.foo();
Utils.bar();

```



## [编写和Kotlin代码文档（相当于Javadoc）](https://www.kotlincn.net/docs/reference/kotlin-doc.html)

## 生成文档

Kotlin 的文档生成工具 [Dokka](https://github.com/Kotlin/dokka)。其使用说明请参见 [Dokka README](https://github.com/Kotlin/dokka/blob/master/README.md)。

Dokka 有 Gradle、Maven 和 Ant 的插件，因此你可以将文档生成集成到你的构建过程中。

