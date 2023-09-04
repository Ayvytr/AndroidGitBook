# [Handler](https://juejin.im/post/5c74b64a6fb9a049be5e22fc)

Handler：实现线程间通信

实例化 Handler 的时候 Handler 会去检查当前线程的 Looper 是否存在，如果不存在则会报异常，也就是说**在创建 Handler 之前一定需要先创建 Looper** 



Handler 提供 send()系列 post()系列的方法让我们来发送消息。

不过不管我们调用什么方法，最终都会走到 `MessageQueue.enqueueMessage(Message,long)` 方法。

以 `sendEmptyMessage(int)`  方法为例：

```
//Handler
sendEmptyMessage(int)
  -> sendEmptyMessageDelayed(int,int)
    -> sendMessageAtTime(Message,long)
      -> enqueueMessage(MessageQueue,Message,long)
  			-> queue.enqueueMessage(Message, long);
```



## Handler 的背后有着 Looper 以及 MessageQueue 的协助，分工明确。

- Looper ：**负责关联线程以及消息的分发**在该线程下**从 MessageQueue 获取 Message，分发给 Handler ；
- MessageQueue ：**是个队列，负责消息的存储与管理**，负责管理由 Handler 发送过来的 Message ；
- Handler : **负责发送并处理消息**，面向开发者，提供 API，并隐藏背后实现的细节。



**Handler 发送的消息由 MessageQueue 存储管理，并由 Loopler 负责回调消息到 handleMessage()。**

**线程的转换由 Looper 完成，handleMessage() 所在线程由 Looper.loop() 调用者所在线程决定。**



## Looper.loop() 为什么不会卡主线程

主线程的MessageQueue没有消息时，便阻塞在loop的queue.next()中的nativePollOnce()，此时主线程会释放cpu资源进入休眠状
态，直到下个消息到达或者有事务发生。这里采用的epoll机制，是一种IO多路
复用机制，可以同时监控多个描述符，当某个描述符就绪（读或写就绪），则立刻通知相应程序进行读或写操作。



主线程中的Looper从消息队列读取消息，当读完所有消息时，主线程阻塞。子线程往消息队列发送消息，并且往管道文件写数据
后，主线程被唤醒，并从管道文件读取数据，当消息读取完毕，再次睡眠。因此loop的循环并不会对cpu性能有过多的消耗。



## Handler 的延伸

由于 Handler 的特性，它在 Android 里的应用非常广泛，比如： AsyncTask、HandlerThread、Messenger、IdleHandler 和 IntentService 等。



### 3.1 Handler 引起的内存泄露原因以及最佳解决方案

Handler 允许我们发送**延时消息**，如果在延时期间用户关闭了 Activity，那么该 Activity 会泄露。

这个泄露是因为 Message 会持有 Handler，而又因为 **Java 的特性，内部类会持有外部类**，使得 Activity 会被 Handler 持有，这样最终就导致 Activity 泄露。

解决该问题的最有效的方法是：**将 Handler 定义成静态的内部类，在内部持有 Activity 的弱引用，并及时移除所有消息**。

示例代码如下：

```
private static class SafeHandler extends Handler {

    private WeakReference<HandlerActivity> ref;

    public SafeHandler(HandlerActivity activity) {
        this.ref = new WeakReference(activity);
    }

    @Override
    public void handleMessage(final Message msg) {
        HandlerActivity activity = ref.get();
        if (activity != null) {
            activity.handleMessage(msg);
        }
    }
}
复制代码
```

并且再在 `Activity.onDestroy()` 前移除消息，加一层保障：

```
@Override
protected void onDestroy() {
  safeHandler.removeCallbacksAndMessages(null);
  super.onDestroy();
}
复制代码
```

这样双重保障，就能完全避免内存泄露了。

**注意：单纯的在 onDestroy 移除消息并不保险，因为 onDestroy 并不一定执行。**



### 3.4 Handler 里藏着的 Callback 能干什么？

在 Handler 的构造方法中有几个 要求传入 Callback ，那它是什么，又能做什么呢？

来看看 `Handler.dispatchMessage(msg)`  方法：

```
public void dispatchMessage(Message msg) {
  //这里的 callback 是 Runnable
  if (msg.callback != null) {
    handleCallback(msg);
  } else {
    //如果 callback 处理了该 msg 并且返回 true， 就不会再回调 handleMessage
    if (mCallback != null) {
      if (mCallback.handleMessage(msg)) {
        return;
      }
    }
    handleMessage(msg);
  }
}
复制代码
```

可以看到 Handler.Callback 有**优先处理消息的权利** ，当一条消息被 Callback 处理**并拦截（返回 true）**，那么 Handler 的 `handleMessage(msg)` 方法就不会被调用了；如果 Callback 处理了消息，但是并没有拦截，那么就意味着**一个消息可以同时被 Callback 以及 Handler 处理**。

这个就很有意思了，这有什么作用呢？

**我们可以利用 Callback 这个拦截机制来拦截 Handler 的消息！**

场景：Hook [ActivityThread.mH](http://ActivityThread.mH) ， 在 ActivityThread 中有个成员变量 `mH` ，它是个 Handler，又是个极其重要的类，几乎所有的插件化框架都使用了这个方法。



### 3.5 创建 Message 实例的最佳方式

由于 Handler 极为常用，所以为了节省开销，Android 给 Message 设计了回收机制，所以我们在使用的时候尽量复用 Message ，减少内存消耗：

1. `Message.obtain();`   
2. 通过 Handler 的公有方法 `handler.obtainMessage();` 。



### 3.6 子线程里弹 Toast 的正确姿势

当我们尝试在子线程里直接去弹 Toast 的时候，会 crash ：

```
java.lang.RuntimeException: Can't create handler inside thread that has not called Looper.prepare()
复制代码
```

**本质上是因为 Toast 的实现依赖于 Handler**，按子线程使用 Handler 的要求修改即可（见【2.1】），同理的还有 Dialog。

正确示例代码如下：

```
new Thread(new Runnable() {
  @Override
  public void run() {
    Looper.prepare();
    Toast.makeText(HandlerActivity.this, "不会崩溃啦！", Toast.LENGTH_SHORT).show();
    Looper.loop();
  }
}).start();
复制代码
```



### 3.7 妙用 Looper 机制

我们可以利用 Looper 的机制来帮助我们做一些事情：

1. 将 Runnable post 到主线程执行；
2. 利用 Looper 判断当前线程是否是主线程。



```
public final class MainThread {

    private MainThread() {
    }

    private static final Handler HANDLER = new Handler(Looper.getMainLooper());

    public static void run(@NonNull Runnable runnable) {
        if (isMainThread()) {
            runnable.run();
        }else{
            HANDLER.post(runnable);
        }
    }

    public static boolean isMainThread() {
        return Looper.myLooper() == Looper.getMainLooper();
    }

}
```


