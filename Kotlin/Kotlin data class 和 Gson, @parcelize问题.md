# Kotlin data class 和 Gson, @parcelize问题



gson 扩展方法

```kotlin
inline fun <reified T> Gson.fromJson(json: String) = fromJson(json, T::class.java)
```



@parcelize 需要在build.gradle android内设置属性

​	

```
androidExtensions {
    experimental = true
}
```



例子：

```kotlin
//正确的data class写法
@Parcelize
data class Student(
    var age: Int = 0,
    var name: String? = null,
    var toy: Toy? = null
) : Parcelable {
}

@Parcelize
data class Toy(
    var price: Int = 0,
    var name: String? = null
) : Parcelable {
}

//错误的data class写法，类体内的name不会序列化，别的地方使用获取到的是null，kotlin bytecode，decompile文件可以看到
@parcelize
data class Student(
	var age: Int = 0): Parcelable {
    var name: String? = null
}


gson解析
val student2 = gson.fromJson<Student>("{\n" +
                    "\t\"age\":10,\n" +
                    "\t\"name\":\"json name\",\n" +
                    "\t\"toy\":{\n" +
                    "\t\t\"name\":\"toy\",\n" +
                    "\t\t\"price\":1000\n" +
                    "\t}\n" +
                    "}", Student::class.java)
            intent.putExtra("student2", student2)
```



