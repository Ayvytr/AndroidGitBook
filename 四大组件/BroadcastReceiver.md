# BroadcastReceiver



在 Android 系统中，广播（Broadcast）是在组件之间传播数据的一种机制，这些组件可以位于不同的进程中，起到进程间通信的作用

**BroadcastReceiver** 是对发送出来的 **Broadcast** 进行过滤、接受和响应的组件。首先将要发送的消息和用于过滤的信息（Action，Category）装入一个 **Intent** 对象，然后通过调用 **Context.sendBroadcast()** 、 **sendOrderBroadcast()** 方法把 Intent 对象以广播形式发送出去。 广播发送出去后，所以已注册的 BroadcastReceiver 会检查注册时的 **IntentFilter** 是否与发送的 Intent 相匹配，若匹配则会调用 BroadcastReceiver 的 **onReceiver()** 方法

所以当我们定义一个 BroadcastReceiver 的时候，都需要实现 onReceiver() 方法。BroadcastReceiver 的生命周期很短，在执行 onReceiver() 方法时才有效，一旦执行完毕，该Receiver 的生命周期就结束了

Android中的广播分为两种类型，标准广播和有序广播

- 标准广播
  标准广播是一种完全异步执行的广播，在广播发出后所有的广播接收器会在同一时间接收到这条广播，之间没有先后顺序，效率比较高，且无法被截断
- 有序广播
  有序广播是一种同步执行的广播，在广播发出后同一时刻只有一个广播接收器能够接收到， 优先级高的广播接收器会优先接收，当优先级高的广播接收器的 onReceiver() 方法运行结束后，广播才会继续传递，且前面的广播接收器可以选择截断广播，这样后面的广播接收器就无法接收到这条广播了

### 注册方法

#### 静态注册

静态注册即在**清单文件**中为 **BroadcastReceiver** 进行注册，使用**< receiver >**标签声明，并在标签内用 **< intent-filter >** 标签设置过滤器。这种形式的 BroadcastReceiver 的生命周期伴随着整个应用，如果这种方式处理的是系统广播，那么不管应用是否在运行，该广播接收器都能接收到该广播

#### 动态注册

动态注册 BroadcastReceiver 是在代码中定义并设置好一个 **IntentFilter** 对象，然后在需要注册的地方调用 **Context.registerReceiver()** 方法，调用 **Context.unregisterReceiver()** 方法取消注册，此时就不需要在清单文件中注册 Receiver 了

#### 本地广播**LocalBroadcastManager**

本地广播是无法通过静态注册的方式来接收的，因为静态注册广播主要是为了在程序未启动的情况下也能接收广播，而本地广播是应用自己发送的，此时应用肯定是启动的了



#### 使用私有权限

使用动态注册广播接收器存在一个问题，即系统内的任何应用均可监听并触发我们的 Receiver 。通常情况下我们是不希望如此的

解决办法之一是在清单文件中为 **< receiver >** 标签添加一个 **android:exported="false"** 属性，标明该 Receiver 仅限应用内部使用。这样，系统中的其他应用就无法接触到该 Receiver 了