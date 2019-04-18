# Service

### [关于Android Service真正的完全详解，你需要知道的一切 ](https://blog.csdn.net/javazejian/article/details/52709857)

### [Android Service使用详解](https://www.jianshu.com/p/95ec2a23f300)

Service是Android中实现程序后台运行的解决方案，非常适合用于去执行哪些不需要和用户交互而且还要求长期运行的任务。不能运行在一个独立的进程当中，而是依赖与创建服务时所在的应用程序进程。只能在后台运行，并且可以和其他组件进行交互。

Service可以在很多场合使用，比如播放多媒体的时候用户启动了其他Activity，此时要在后台继续播放；比如检测SD卡上文件的变化；比如在后台记录你的地理信息位置的改变等等，总之服务是藏在后台的。

服务不会自动开启线程，我们需要在服务的内部手动创建子线程，并在这里执行具体的任务。关于多线程的知识：可以参考另外一篇文章：[Android多线程----异步消息处理机制之Handler详解](http://www.cnblogs.com/smyhvae/p/4003922.html)





![img](http://upload-images.jianshu.io/upload_images/1981935-bd709d5989105a12.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### Service简单概述

  Service是Android系统中的四大组件之一，主要有两个应用场景：后台运行和跨进程访问。Service可以在后台执行长时间运行操作而不提供用户界面，除非系统必须回收内存资源，否则系统不会停止或销毁服务。服务可由其他应用组件启动，而且即使用户切换到其他应用，服务仍将在后台继续运行。 此外，组件可以绑定到服务，以与之进行交互，甚至是执行进程间通信 (IPC)
 需要注意的是，Service是在主线程里执行操作的，可能会因为执行耗时操作而导致ANR



Service可以分为以下三种形式：

- 启动
  当应用组件通过调用 **startService()** 启动服务时，服务即处于“启动”状态。一旦启动，服务即可在后台无限期运行，即使启动服务的组件已被销毁也不受影响。 已启动的服务通常是执行单一操作，而且不会将结果返回给调用方
- 绑定
  当应用组件通过调用 **bindService()** 绑定到服务时，服务即处于“绑定”状态。绑定服务提供了一个客户端-服务器接口，允许组件与服务进行交互、发送请求、获取结果，甚至是利用进程间通信 (IPC) 跨进程执行这些操作。多个组件可以同时绑定服务，服务只会在组件与其绑定时运行，一旦该服务与所有组件之间的绑定全部取消，系统便会销毁它
- 启动且绑定
  服务既可以是启动服务，也允许绑定。此时需要同时实现以下回调方法：**onStartCommand()**和 **onBind()**。系统不会在所有客户端都取消绑定时销毁服务。为此，必须通过调用 **stopSelf()** 或 **stopService()** 显式停止服务

无论应用是处于启动状态还是绑定状态，或者处于启动且绑定状态，任何应用组件均可像使用 Activity 那样通过调用 Intent 来使用服务（即使此服务来自另一应用），也可以通过清单文件将服务声明为私有服务，阻止其他应用访问

要使用服务，必须继承Service类（或者Service类的现有子类），在子类中重写某些回调方法，以处理服务生命周期的某些关键方面并提供一种机制将组件绑定到服务

- onStartCommand()
  当组件通过调用 **startService()** 请求启动服务时，系统将调用此方法（如果是绑定服务则不会调用此方法）。一旦执行此方法，服务即会启动并可在后台无限期运行。 在指定任务完成后，通过调用 **stopSelf()** 或 **stopService()** 来停止服务
- onBind()
  当一个组件想通过调用 **bindService()** 与服务绑定时，系统将调用此方法（如果是启动服务则不会调用此方法）。在此方法的实现中，必须通过返回 **IBinder** 提供一个接口，供客户端用来与服务进行通信
- onCreate()
  首次创建服务时，系统将调用此方法来执行初始化操作（在调用 **onStartCommand()** 或 **onBind()** 之前）。如果在启动或绑定之前Service已在运行，则不会调用此方法
- onDestroy()
  当服务不再使用且将被销毁时，系统将调用此方法，这是服务接收的最后一个调用，在此方法中应清理占用的资源

仅当内存过低必须回收系统资源以供前台 Activity 使用时，系统才会强制停止服务。如果将服务绑定到前台 Activity，则它不太可能会终止，如果将服务声明为在前台运行，则它几乎永远不会终止。或者，如果服务已启动并要长时间运行，则系统会随着时间的推移降低服务在后台任务列表中的位置，而服务也将随之变得非常容易被终止。如果服务是启动服务，则必须将其设计为能够妥善处理系统对它的重启。 如果系统终止服务，那么一旦资源变得再次可用，系统便会重启服务（这还取决于 onStartCommand() 的返回值）



Service在清单文件中的声明
  前面说过Service分为启动状态和绑定状态两种，但无论哪种具体的Service启动类型，都是通过继承Service基类自定义而来，也都需要在AndroidManifest.xml中声明，那么在分析这两种状态之前，我们先来了解一下Service在AndroidManifest.xml中的声明语法，其格式如下：

<service android:enabled=["true" | "false"]
    android:exported=["true" | "false"]
    android:icon="drawable resource"
    android:isolatedProcess=["true" | "false"]
    android:label="string resource"
    android:name="string"
    android:permission="string"
    android:process="string" >
    . . .
</service>

android:exported：代表是否能被其他应用隐式调用，其默认值是由service中有无intent-filter决定的，如果有intent-filter，默认值为true，否则为false。为false的情况下，即使有intent-filter匹配，也无法打开，即无法被其他应用隐式调用。

android:name：对应Service类名

android:permission：是权限声明

android:process：是否需要在单独的进程中运行,当设置为android:process=”:remote”时，代表Service在单独的进程中运行。注意“：”很重要，它的意思是指要在当前进程名称前面附加上当前的包名，所以“remote”和”:remote”不是同一个意思，前者的进程名称为：remote，而后者的进程名称为：App-packageName:remote。

android:isolatedProcess ：设置 true 意味着，服务会在一个特殊的进程下运行，这个进程与系统其他进程分开且没有自己的权限。与其通信的唯一途径是通过服务的API(bind and start)。

android:enabled：是否可以被系统实例化，默认为 true因为父标签 也有 enable 属性，所以必须两个都为默认值 true 的情况下服务才会被激活，否则不会激活。 

首先要创建服务，必须创建 Service 的子类（或使用它的一个现有子类如IntentService）。

### Service绑定服务

  绑定服务是Service的另一种变形，当Service处于绑定状态时，其代表着客户端-服务器接口中的服务器。当其他组件（如 Activity）绑定到服务时（有时我们可能需要从Activity组建中去调用Service中的方法，此时Activity以绑定的方式挂靠到Service后，我们就可以轻松地方法到Service中的指定方法），组件（如Activity）可以向Service（也就是服务端）发送请求，或者调用Service（服务端）的方法，此时被绑定的Service（服务端）会接收信息并响应，甚至可以通过绑定服务进行执行进程间通信 (即IPC，这个后面再单独分析)。与启动服务不同的是绑定服务的生命周期通常只在为其他应用组件(如Activity)服务时处于活动状态，不会无限期在后台运行，也就是说宿主(如Activity)解除绑定后，绑定服务就会被销毁。那么在提供绑定的服务时，该如何实现呢？实际上我们必须提供一个 IBinder接口的实现类，该类用以提供客户端用来与服务进行交互的编程接口，该接口可以通过三种方法定义接口：

扩展 Binder 类 
  如果服务是提供给自有应用专用的，并且Service(服务端)与客户端相同的进程中运行（常见情况），则应通过扩展 Binder 类并从 onBind() 返回它的一个实例来创建接口。客户端收到 Binder 后，可利用它直接访问 Binder 实现中以及Service 中可用的公共方法。如果我们的服务只是自有应用的后台工作线程，则优先采用这种方法。 不采用该方式创建接口的唯一原因是，服务被其他应用或不同的进程调用。

使用 Messenger 
  Messenger可以翻译为信使，通过它可以在不同的进程中共传递Message对象(Handler中的Messager，因此 Handler 是 Messenger 的基础)，在Message中可以存放我们需要传递的数据，然后在进程间传递。如果需要让接口跨不同的进程工作，则可使用 Messenger 为服务创建接口，客户端就可利用 Message 对象向服务发送命令。同时客户端也可定义自有 Messenger，以便服务回传消息。这是执行进程间通信 (IPC) 的最简单方法，因为 Messenger 会在单一线程中创建包含所有请求的队列，也就是说Messenger是以串行的方式处理客户端发来的消息，这样我们就不必对服务进行线程安全设计了。

使用 AIDL 

由于Messenger是以串行的方式处理客户端发来的消息，如果当前有大量消息同时发送到Service(服务端)，Service仍然只能一个个处理，这也就是Messenger跨进程通信的缺点了，因此如果有大量并发请求，Messenger就会显得力不从心了，这时AIDL（Android 接口定义语言）就派上用场了， 但实际上Messenger 的跨进程方式其底层实现 就是AIDL，只不过android系统帮我们封装成透明的Messenger罢了 。因此，如果我们想让服务同时处理多个请求，则应该使用 AIDL。 在此情况下，服务必须具备多线程处理能力，并采用线程安全式设计。使用AIDL必须创建一个定义编程接口的 .aidl 文件。Android SDK 工具利用该文件生成一个实现接口并处理 IPC 的抽象类，随后可在服务内对其进行扩展。



以上3种实现方式，我们可以根据需求自由的选择，但需要注意的是大多数应用“都不会”使用 AIDL 来创建绑定服务，因为它可能要求具备多线程处理能力，并可能导致实现的复杂性增加。



### 管理服务的生命周期（从创建到销毁）有以下两种情况：

启动服务 
该服务在其他组件调用 startService() 时创建，然后无限期运行，且必须通过调用 stopSelf() 来自行停止运行。此外，其他组件也可以通过调用 stopService() 来停止服务。服务停止后，系统会将其销毁。

绑定服务 

该服务在另一个组件（客户端）调用 bindService() 时创建。然后，客户端通过 IBinder 接口与服务进行通信。客户端可以通过调用 unbindService() 关闭连接。多个客户端可以绑定到相同服务，而且当所有绑定全部取消后，系统即会销毁该服务。 （服务不必自行停止运行）



### IntentService

Service服务是Android四大组件之一，在Android中有着举足重轻的作用。Service服务是工作的UI线程中，当你的应用需要下载一个文件或者播放音乐等长期处于后台工作而有没有UI界面的时候，你肯定要用到Service+Thread来实现。因此你需要自己在Service服务里面实现一个Thread工作线程来下载文件或者播放音乐。然而你每次都需要自己去写一个Service+Thread来处理长期处于后台而没有UI界面的任务，这样显得很麻烦，没必要每次都去构建一个Service+Thread框架处理长期处于后台的任务。Google工程师给我们构建了一个方便开发者使用的这么一个框架---IntentService。

IntentService是一个基础类，用于处理Intent类型的异步任务请求。当客户端调用android.content.Context#startService(Intent)发送请求时，Service服务被启动，且在其内部构建一个工作线程来处理Intent请求。当工作线程执行结束，Service服务会自动停止。IntentService是一个抽象类，用户必须实现一个子类去继承它，且必须实现IntentService里面的抽象方法onHandleIntent来处理异步任务请求。

#### IntentService总结   

1. 子类需继承IntentService并且实现里面的onHandlerIntent抽象方法来处理intent类型的任务请求。
2. 子类需要重写默认的构造方法，且在构造方法中调用父类带参数的构造方法。
3. IntentService类内部利用HandlerThread+Handler构建了一个带有消息循环处理机制的后台工作线程，客户端只需调用Content#startService(Intent)将Intent任务请求放入后台工作队列中，且客户端无需关注服务是否结束，非常适合一次性的后台任务。比如浏览器下载文件，退出当前浏览器之后，下载任务依然存在后台，直到下载文件结束，服务自动销毁。
4. 只要当前IntentService服务没有被销毁，客户端就可以同时投放多个Intent异步任务请求，IntentService服务端这边是顺序执行当前后台工作队列中的Intent请求的，也就是每一时刻只能执行一个Intent请求，直到该Intent处理结束才处理下一个Intent。因为IntentService类内部利用HandlerThread+Handler构建的是一个单线程来处理异步任务。



