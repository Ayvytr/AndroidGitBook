# [Handler](https://juejin.im/post/5c74b64a6fb9a049be5e22fc)

Handler 源码设计的核心要点：

1. **消息复用**：Message 对象池减少 GC

2. **线程隔离**：ThreadLocal 保证线程安全

3. **时间排序**：MessageQueue 按 when 排序

4. **异步唤醒**：Native 层 epoll 实现高效等待

5. **同步屏障**：优先处理异步消息，优化 UI 绘制。作用：临时阻塞普通的同步消息，优先处理异步消息。在Android中，同步屏障主要用于优化UI绘制，确保绘制消息（异步消息）能够优先得到处理。

6. **空闲处理**：IdleHandler 利用空闲时间执行非紧急任务





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





## 同步屏障源码详解

### 1. 同步屏障的插入

同步屏障通过调用MessageQueue的`postSyncBarrier()`方法插入。该方法会返回一个token，用于后续移除屏障。

```
// MessageQueue.java
public int postSyncBarrier() {
    return postSyncBarrier(SystemClock.uptimeMillis());
}

private int postSyncBarrier(long when) {
    synchronized (this) {
        final int token = mNextBarrierToken++;
        // 创建一个特殊的Message，target为null表示屏障
        final Message msg = Message.obtain();
        msg.markInUse();
        msg.when = when;
        msg.arg1 = token;

        Message prev = null;
        Message p = mMessages;
        if (when != 0) {
            while (p != null && p.when <= when) {
                prev = p;
                p = p.next;
            }
        }
        // 将屏障消息插入到消息队列的合适位置（按时间排序）
        if (prev != null) {
            msg.next = p;
            prev.next = msg;
        } else {
            msg.next = p;
            mMessages = msg;
        }
        return token;
    }
}
```

注意：同步屏障消息是一个特殊的消息，它的target为null（普通消息的target是发送它的Handler）。在消息队列中，屏障消息就像一堵墙，它后面的所有同步消息都会被阻塞，直到屏障被移除。

### 2. 同步屏障的处理

在MessageQueue的`next()`方法中，当遇到同步屏障时，会跳过所有同步消息，只寻找异步消息。

```
// MessageQueue.java - next()方法片段
Message msg = mMessages;
if (msg != null && msg.target == null) {
    // 遇到同步屏障，寻找队列中的异步消息
    do {
        prevMsg = msg;
        msg = msg.next;
    } while (msg != null && !msg.isAsynchronous());
}
```

如果找到异步消息，则返回该异步消息；否则，进入阻塞等待。这样，同步屏障就实现了对同步消息的阻塞，优先处理异步消息。

### 3. 同步屏障的移除

通过调用`removeSyncBarrier(int token)`移除指定的同步屏障。

```
// MessageQueue.java
public void removeSyncBarrier(int token) {
    synchronized (this) {
        Message prev = null;
        Message p = mMessages;
        // 寻找token对应的屏障消息
        while (p != null && (p.target != null || p.arg1 != token)) {
            prev = p;
            p = p.next;
        }
        if (p == null) {
            throw new IllegalStateException("The specified barrier token " + token + " was not found.");
        }
        // 从消息队列中移除屏障消息
        if (prev != null) {
            prev.next = p.next;
        } else {
            mMessages = p.next;
        }
        // 回收消息
        p.recycleUnchecked();
    }
}
```

### 4. 异步消息的发送

要利用同步屏障，需要发送异步消息。可以通过以下两种方式：

1. 

   使用异步Handler：

```
Handler asyncHandler = Handler.createAsync(Looper.getMainLooper());
```

1. 

   给Message设置异步标志：

```
Message msg = Message.obtain();
msg.setAsynchronous(true);
handler.sendMessage(msg);
```

### 5. 同步屏障的应用场景：UI绘制

在Android的UI刷新机制中，ViewRootImpl会在绘制流程开始前插入同步屏障，以确保绘制消息（异步消息）能够优先处理。

```
// ViewRootImpl.java
void scheduleTraversals() {
    if (!mTraversalScheduled) {
        mTraversalScheduled = true;
        // 插入同步屏障
        mTraversalBarrier = mHandler.getLooper().getQueue().postSyncBarrier();
        // 发送异步绘制消息
        mChoreographer.postCallback(
            Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);
    }
}
```

当绘制完成后，会移除同步屏障：

```
void unscheduleTraversals() {
    if (mTraversalScheduled) {
        mTraversalScheduled = false;
        // 移除同步屏障
        mHandler.getLooper().getQueue().removeSyncBarrier(mTraversalBarrier);
    }
}
```

这样，在绘制过程中，即使有其他同步消息（例如业务逻辑产生的消息）在消息队列中等待，也会被屏障阻塞，从而保证绘制的流畅性。



## IdleHandler 利用空闲时间执行非紧急任务

在MessageQueue的next()方法中，当没有消息需要立即处理时，会检查IdleHandler列表，并执行它们。

使用方式：

1. 

   实现MessageQueue.IdleHandler接口

2. 

   实现queueIdle()方法，返回true表示下次空闲时继续执行，false表示移除该IdleHandler

3. 

   通过Looper.myQueue().addIdleHandler()添加



```
Looper.myQueue().addIdleHandler(new MessageQueue.IdleHandler() {

@Override

public boolean queueIdle() {

// 执行非紧急任务

doSomeNonUrgentTask();

// 返回false表示只执行一次，返回true表示下次空闲时再次执行

return false;

}

});
```



Looper.myQueue().addIdleHandler(idleHandler)增加任务。如果返回true，我们需要在适当的时机（比如任务已经完成，或者页面销毁时）手动移除它，否则它会一直存在：

```
// 移除IdleHandler
Looper.myQueue().removeIdleHandler(idleHandler)
```

因此，在决定返回值时，需要根据任务的性质来决定。同时，为了避免内存泄漏，我们需要确保在不需要时移除那些返回true的IdleHandler。

另外，需要注意的是，如果一次空闲事件中，有多个IdleHandler，它们会按照添加的顺序依次执行。如果某个IdleHandler返回true，那么它会被保留，下次空闲时还会执行；如果返回false，则执行一次后就被移除。

所以，queueIdle()的返回值并不是控制任务是否重复执行，而是控制这个IdleHandler是否在下次空闲时再次被调用。在一次空闲事件中，每个IdleHandler只会被调用一次。
