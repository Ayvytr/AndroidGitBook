# Android优化

## 常见内存泄漏场景

```java

```

### 1. 需要手动关闭的对象没有关闭

#### 1.1. try/catch/finally中网络文件等流的手动关闭

- HTTP
- File
- ContendProvider
- Bitmap
- Uri
- Socket

这些都是java基础啦，就不一一介绍了。我们可以用RxJava进行封装，让它变成可观察的流；在Go语言中，可以使用Defer这样的方法来减少迷之缩进；在okhttp中，使用了[引用计数](https://link.juejin.im/?target=http%3A%2F%2Fwww.jianshu.com%2Fp%2F92a61357164b)的技术对流进行管理

#### 1.2. `onDestroy()` 或者 `onPause()`中未及时关闭对象

泄露实例：

- 线程泄漏：当你执行耗时任务，在`onDestroy()`的时候考虑调用`Thread.close()`，如果对线程的控制不够强的话，可以使用RxJava自动建立线程池进行控制，并在生命周期结束时取消订阅；
- Handler泄露：当退出activity时，要注意所在Handler消息队列中的Message是否全部处理完成，可以考虑`removeCallbacksAndMessages(null)`手动关闭
- 广播泄露：手动注册广播时，记住退出的时候要`unregisterReceiver()`
- 第三方SDK/开源框架泄露：ShareSDK, JPush等第三方SDK需要按照文档控制生命周期,它们有时候要求你继承它们丑陋的activity，其实也为了帮你控制生命周期
- 各种callBack/Listener的泄露，要及时设置为Null，特别是static的callback
- EventBus等观察者模式的框架需要手动解除注册
- 某些Service也要及时关闭，比如图片上传，当上传成功后，要`stopself()`
- Webview需要手动调用`WebView.onPause()`以及`WebView.destory()`

比如常见的ButterKnife

```
  @Override public void onDestroyView() {
    super.onDestroyView();
    ButterKnife.reset(this);
  }
```

再比如ShareSDK（此垃圾再也不用）

```
protected void onDestroy() {
        ShareSDK.stopSDK(this);
        super.onDestroy();
    }
```

> 使用开源的框架（比如帮你写好的图片下载队列，REST解析等）可能会帮助你快速的解决这个问题，但是知其然并知其所以然，也要了解它们的生命周期

### 2. Static的使用

#### 2.1 static class/method/variable 的区别，你真的懂了吗？

(1). **Static inner class 与 non static inner class 的区别**

static inner class 即`静态内部类`，它只会出现在类的内部，在某个类中写一个静态内部类其实同你在IDE里新建一个`.java` 文件是完全一样的。

以下为它们的对比

| class对比                       | static inner class             | non-static inner class               |
| ------------------------------- | ------------------------------ | ------------------------------------ |
| 与外部class引用关系             | 如果没有传入参数，就无引用关系 | 自动获得强引用（implicit reference） |
| 被调用时需要外部实例            | 不需要（比如Bulider类）        | 需要                                 |
| 能否调用外部class中的变量与方法 | 不能                           | 能                                   |
| 生命周期                        | 自主的生命周期                 | 依赖于外部类，甚至比外部类更长       |



可以看到，在生命周期中，埋下了内存泄漏的隐患，如果它的生命周期比activity更长，那么可能会发生泄露，更可怕的是，有可能会产生难以预防的空指针问题。

这个泄露的例子，详见内存管理(2)的文章。

(2). static inner method

静态内部方法，也就是虚函数：可以被直接调用，而不用去依赖它所在的类，比如你需要随机数，只用调用`Math.random()`即可，而不用实例化`Math`这个对象。在工具类（Utils）中，建议用static修饰方法。static方法的调用不会泄露内存。

(3). static inner variable

慎重使用静态变量，静态变量是被分配给当前的Class的，而不是一个独立的实例，当ClassLoader停止加载这个Class时，它才会回收。在Android中，需要手动置空才会卸掉ClassLoader，才能出现GC。

> static 变量称为静态变量或者类变量，它由类的所有实例共享。
> Classes are only unloaded if all classes associated with a ClassLoader can be garbage collected, which is rare but will not be impossible in Android.
> 高效的场景：“全局常量”，“单例”与“远程接口”。

这段谷歌博客上的著名代码演示了一次内存泄露的，当你旋转屏幕后，Drawable就会泄露。

```
private static Drawable sBackground;

@Override
protected void onCreate(Bundle state) {
  super.onCreate(state);

  TextView label = new TextView(this);
  label.setText("Leaks are bad");

  if (sBackground == null) {
    sBackground = getDrawable(R.drawable.large_bitmap);
  }
  label.setBackgroundDrawable(sBackground);

  setContentView(label);
}
```

注意，我在实际试验(CM12)中，没有GC时，导出的堆是有泄露的，而手动GC后，是**不会发生**内存泄露的，希望各位自己做实验，发一下反馈（评论已经有了相关反馈）。



MAT

总的来说，还是少用，就算可能是新的设备支持更加先进的GC，但是还是要注意控制内存。

#### 2.2. 使用内部匿名类要注意什么？

匿名内部类实际上就是non-static inner class，比如某些初学者经常一个`new Handler`就写出来了，它对外部类有一个强引用。建议单独写出来这个类并继承，并加入static修饰。

#### 2.3. 单例模式（Singleton）是不是内存泄漏？

在单例模式中，只有一个对象被产生，看起来一直占用了内存，但是这个**不意味**就是浪费了内存，内存本来就是用来装东西的，只要这个对象一直被高效的利用就不能叫做泄露。但是也不要偷懒，一个劲的全整成了单例，越多的单例会让内存占用过多，放在Application中初始化的内容也越多，意味着APP打开白屏的时间会更久，而且软件维护起来也变得复杂。

- 好的例子：GlobalContext，SmsReceiver动态注册，EventBus



可以看到我们在 Activity 中继承 AsyncTask 自定义了一个非静态内部类，在 doInbackground() 方法中做了耗时的操作，然后在 onCreate() 中启动 MyAsyncTask。如果在耗时操作结束之前，Activity 被销毁了，这时候因为 MyAsyncTask 持有 Activity 的强引用，便会导致 Activity 的内存无法被回收，这时候便会产生内存泄露。

public class MainActivity extends AppCompatActivity {

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    new MyAscnyTask().execute();
}

class MyAscnyTask extends AsyncTask<Void, Integer, String>{
    @Override
    protected String doInBackground(Void... params) {
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "";
    }
}
```
}

//解决方案：非静态内部类改为静态内部类
public class MainActivity extends AppCompatActivity {

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    new MyAscnyTask().execute();
}

static class MyAscnyTask extends AsyncTask<Void, Integer, String>{
    @Override
    protected String doInBackground(Void... params) {
        try {
            Thread.sleep(50000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "";
    }
}
```
}

非静态内部类自动获得外部类的强引用，而且它的生命周期甚至比外部类更长，这便埋下了内存泄露的隐患。如果一个 Activity 的非静态内部类的生命周期比 Activity 更长，那么 Activity 的内存便无法被回收，也就是发生了内存泄露，而且还有可能发生难以预防的空指针问题。



#### 2.4. 为什么大神喜欢用static final来修饰常数？

static由于是所有实例共享的，说到共享一定要加锁，万一某个实例更改它后，其它的实例也会受到影响，所以加入`final`作为永久只读锁以防止常数被修改。

> 全局变量生命周期是classloader，有坑。你的activity在finish后变量并不会改变。
> 这个在面试中经常遇到，问你经过多次计算后，static的值是多少。比如在Android中有个坑，最常见的就是把一个sharedpreference赋值给一个static变量，然后又把sharedpreference改变后，再次调用这个static变量，就发现变量并没有改变，这个在debug中很难发现。

#### 2.5. 顺便说下final吧

- final 变量：是只读的；
- final 方法：是不能继承或者重写的。
- final 引用：引用不能修改，但是对象本身的属性可以修改；
- final class：不可继承；

```
final MyObject o = new MyObject();
o.setValue("foo"); // Works just fine
o = new MyObject(); // Doesn't work.
```

- 虚拟机并不会知道你的变量是否是final的，所以final与内存泄露无关。
- final不会让代码速度更快

### 3. Bitmap的使用

- 使用前注意配置Bitmap的Config，比如长宽，参数(565, 8888)，格式；
- 使用中注意缓存；
- 使用后注意recycle以清理native层的内存。

> 2.3以后的bitmap不需要手动recycle了，内存已经在java层了。同时，Bitmap还有别人做好的轮子，比如PhotoView,Picasso，就可以方便的解决OOM问题。

### 4. 多线程

线程泄露可能是最严重的泄露问题了，第一它可能与Handler一样，转一转手机内存就没了，第二是当回调的时候，它极可能弹出`NullPointException`

##### 个人在实际使用的一个失败实例

上传图片时退出Activity，等到图片完成后，Toast就会抛出空指针异常。

```
//retrofit 1.9 bad sample 
RestAdapter adapter = new RestAdapter.Builder().setEndpoint(HeadlineService.END_POINT)
            .setLogLevel(RestAdapter.LogLevel.FULL)
            .build();
adapter.create(ImageService.class)
    .updateImage(new TypedFile("image/*", file), new TypedString(nickname),
        new TypedString(Build.MODEL), new TypedString(avatar),
        new Callback<UploadResult>() {
          @Override public void success(UploadResult uploadResult, Response response) {
            if (uploadResult.getStatus() == 1) {
              Log.d(TAG, "upload successfully!");
              Toast.makeText(getActivity(), "上传成功！", Toast.LENGTH_SHORT)
                  .show();
            } else {
              Log.e(TAG, "upload failed!");
              Toast.makeText(getActivity(), "上传失败！", Toast.LENGTH_SHORT)
                  .show();
            }
            bmp.recycle();
          }

          @Override public void failure(RetrofitError error) {
            bmp.recycle();
          }
        });
```

我是使用Retrofit框架进行上传的，retrofit内部自己维护它的线程与生命周期，当我退出Activity时，Retrofit内部的网络线程并没有停止；当图片上传成功回调的时候，却发现window已经没了，这样就会抛出异常。

解决方法：在Activity中使用耗时任务本来就不合适，使用Service可以更好的控制回调问题。

### 5. Context与ApplicationContext

| class    | Context                         | ApplicationContext                                           |
| -------- | ------------------------------- | ------------------------------------------------------------ |
| 生命周期 | 短                              | 非常长，几乎就是单例                                         |
| 适用场景 | Activity中需要UI/素材资源的地方 | 数据库，包管理，偏好设置，以及Picasso/Retrofit/ShareSDK/Webview等单例框架 |



Context的生命周期是一个Activiy，而ApplicationContext的生命周期是整个程序。我们最要注意的就是Context的内存泄露。

在Activiy的UI中要使用Context,而在其他的地方比如数据库、网络、系统服务的需要频繁调用Context的情况时，要使用ApplicationContext，以防止内存泄露。



```java
//传入的Context是Activity时，单例持有Activity的强引用，这样即使该Activity退出，Activity内存也不会被回收，这样就造成了内存泄漏。
//解决方案：使传入单例模式引用的对象的生命周期 = 应用生命周期
//将 context.getApplicationContext() 赋值给 mContext，此时单例引用的对象是 Application，而 Application 的生命周期本来就跟应用程序是一样的，也就不存在内存泄露。
public class SingleInstanceTest {

    private static SingleInstanceTest sInstance;
    private Context mContext;

    private SingleInstanceTest(Context context){
        this.mContext = context.getApplicationContext();
    }

    public static SingleInstanceTest newInstance(Context context){
        if(sInstance == null){
            sInstance = new SingleInstanceTest(context);
        }
        return sInstance;
    }
}
```





```java
//使用弱引用（WeakReference）进行保存Context实例
public class Sample {

    private WeakReference<Context> mWeakReference;

    public Sample(Context context){
        this.mWeakReference = new WeakReference<>(context);
    }

    public Context getContext() {
        if(mWeakReference.get() != null){
            return mWeakReference.get();
        }
        return null;
    }
}

// 外部调用
Sample sample = new Sample(MainActivity.this);
```



### 6. 其他

Handler 非静态内部Handler，应该把Handler改为静态内部类，持有Activity软引用进行使用，关闭Activity后要移除所有的消息

Thread Activity结束线程还在运行

TimerTask

AsyncTask 非静态内部AsyncTask持有Activity引用，应当改为静态内部类加持有Activity软引用

广播或其他资源注册后没有反注册

创建对象后没有关闭

集合

Adapter没有使用缓存，主要指ListView

IO，File， Cursor, ContentObserver, BroadCastRecelver



