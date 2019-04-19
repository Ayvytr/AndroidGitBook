# Logcat



## 优先级

- **V** — 详细（最低优先级）
- **D** — 调试
- **I** — 信息
- **W** — 警告
- **E** — 错误
- **A** — 断言



## 通过代码记录日志

`常用的日志记录方法包括：

- `Log.v(String, String)`（详细）
- `Log.d(String, String)`（调试）
- `Log.i(String, String)`（信息）
- `Log.w(String, String)`（警告）
- `Log.e(String, String)`（错误）



例如，使用以下调用：

```
Log.i("MyActivity", "MyClass.getView() — get item number " + position);
```



logcat 输出类似于如下：

```
I/MyActivity( 1557): MyClass.getView() — get item number 1
```





## [通过Android Studio查看和过滤日志](https://developer.android.google.cn/studio/debug/am-logcat.html?hl=zh-CN)

