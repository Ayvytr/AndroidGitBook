# Kotlin 源码分析



# 协程

## 线程池实现了工作窃取算法



Kotlin 协程默认的 CPU 密集型调度器（`Dispatcher.Default`）使用了一种智能的线程池，能够自动平衡多个线程之间的工作量，从而最大化 CPU 的利用效率。



### 核心：工作窃取算法

这是一种**负载平衡策略**，用于解决多个“工人”（线程）处理一个“任务队列”时可能出现的忙闲不均问题。

- 

  **传统线程池的问题**：通常有一个全局的任务队列。所有工作线程都从这个队列里取任务执行。如果某些任务执行时间很长，而其他任务很短，就可能出现某些线程一直忙碌，而其他线程早早空闲下来，却无活可干的情况（如下图左）。

- 

  **工作窃取的解决方案**：它为**每个线程维护一个自己的专属任务队列**（双端队列，Deque）。线程优先从自己队列的**尾部**取出任务执行（LIFO，后进先出，有利于缓存局部性）。

  - 

    当某个线程**清空了自己的队列**后，它不会“躺平”，而是变成一个“小偷”：它会随机选择另一个线程，从那个线程队列的**头部**“偷”一个任务来执行（FIFO，先进先出，因为最老的任务最可能是一个较大的工作单元）。

  - 

    这样就实现了线程间工作的**自动再平衡**，减少了线程空闲时间。





这个调度器专门为**CPU 密集型计算**而优化，例如排序、复杂计算、数据转换等。工作窃取算法在这里能发挥最大功效，**确保所有 CPU 核心都高效运转**。大部分时间线程只操作自己的本地队列（无竞争），“窃取”时才发生少量跨线程交互。**这是 Kotlin 协程能够高效处理大量并发计算任务、发挥多核 CPU 优势的关键底层机制之一。**

### 

## 调度器

在Kotlin协程中，线程池的管理主要是通过调度器（Dispatcher）来控制的。协程库提供了一些默认的调度器，它们内部使用了线程池，同时也允许我们自定义线程池。

1. 

   默认调度器：

   - 

     `Dispatchers.Default`：用于CPU密集型任务，它使用JVM上的共享线程池，其大小默认为CPU核心数（至少为2）。

   - 

     `Dispatchers.IO`：用于IO密集型任务，它使用基于`Dispatchers.Default`的线程池，但允许同时运行的线程数更多（默认大小为64）。

   - 

     `Dispatchers.Main`：在主线程上运行，例如Android的UI线程。

   - 

     `Dispatchers.Unconfined`：不限制在特定线程上，立即执行，直到第一个挂起点，之后在恢复的线程中执行。

2. 

   自定义线程池：

   我们可以使用`Executors`创建一个线程池，然后通过`asCoroutineDispatcher`扩展函数将其转换为协程调度器。

3. 

   线程池的管理：

   - 

     使用`Executors`创建线程池（如固定线程池、缓存线程池等）。

   - 

     将线程池转换为协程调度器。

   - 

     在协程中使用该调度器。

4. 

   注意事项：

   - 

     避免创建过多的线程池，以免浪费资源。可以考虑在应用中使用共享的线程池。

   - 

     在不需要时，应该关闭线程池（特别是自定义的线程池），以释放资源。





## DefaultScheduler


    

    // 源码位置：kotlinx.coroutines.scheduling.DefaultScheduler
    internal object DefaultScheduler : ExperimentalCoroutineDispatcher() {
        override val executor: ExecutorService
            get() = coroutineScheduler
            
    // 核心线程池实现
    private val coroutineScheduler = CoroutineScheduler(
        corePoolSize = CORE_POOL_SIZE,          // CPU核心数
        maxPoolSize = MAX_POOL_SIZE,           // 1e6 很大
        idleWorkerKeepAliveNs = IDLE_WORKER_KEEP_ALIVE_NS, // 空闲线程存活时间
        schedulerName = "DefaultDispatcher"
    )
    
    // Android 上默认核心数配置
    private val CORE_POOL_SIZE = 
        (System.getProperty(PROPERTY_NAME)?.toIntOrNull() 
         ?: AVAILABLE_PROCESSORS).coerceAtLeast(2)
    }


## DefaultIoScheduler

```
internal object DefaultIoScheduler : ExecutorCoroutineDispatcher(), Executor {

    private val default = UnlimitedIoScheduler.limitedParallelism(
        systemProp(
            IO_PARALLELISM_PROPERTY_NAME,
            64.coerceAtLeast(AVAILABLE_PROCESSORS)
        )
    )
}
```



## 自定义线程池

```
fun main() = runBlocking {
    // 创建一个固定大小为4的线程池
    val threadPool = Executors.newFixedThreadPool(4)
    val customDispatcher = threadPool.asCoroutineDispatcher()

    // 使用自定义调度器启动协程
    val job = launch(customDispatcher) {
        println("Running in thread: ${Thread.currentThread().name}")
        delay(1000)
        println("Done")
    }

    job.join()

    // 关闭线程池，释放资源
    threadPool.shutdown()
    threadPool.awaitTermination(1, TimeUnit.SECONDS)
}
```



也可以close函数来关闭调度器，以便释放资源。例如：

val threadPool = Executors.newFixedThreadPool(4)
val customDispatcher = threadPool.asCoroutineDispatcher()

// 使用完成后关闭
customDispatcher.close() // 这会关闭底层的线程池





```
// 自定义协程线程池示例
val customDispatcher = Executors.newFixedThreadPool(4)
    .asCoroutineDispatcher()

// 或使用更灵活的配置
val myDispatcher = Executors.newFixedThreadPool(
    4,  // 核心线程数
    ThreadFactory { runnable ->
        Thread(runnable, "MyCoroutinePool-${counter.incrementAndGet()}")
            .apply { isDaemon = true }  // 守护线程
    }
).asCoroutineDispatcher()

// 使用
GlobalScope.launch(myDispatcher) {
    println("Running on: ${Thread.currentThread().name}")
}
```



## 工作窃取（Work-Stealing）算法



```
// Kotlin 协程线程池实现了工作窃取算法
// 源码：kotlinx.coroutines.scheduling.CoroutineScheduler

internal class CoroutineScheduler(
    corePoolSize: Int,
    maxPoolSize: Int,
    idleWorkerKeepAliveNs: Long = 1000L * 1000,  // 1秒
    schedulerName: String = "CoroutineScheduler"
) : Executor, Closeable {
    
    // 工作窃取队列
    private val globalCpuQueue = GlobalQueue()
    private val localQueues = arrayOfNulls<LocalQueue>(maxPoolSize)
    
    fun dispatch(block: Runnable, taskContext: TaskContext = NonBlockingContext) {
        val currentWorker = currentWorker()
        if (currentWorker != null) {
            // 尝试本地执行
            currentWorker.submit(block, taskContext)
        } else {
            // 提交到全局队列
            globalCpuQueue.addLast(block)
            signalCpuWork()  // 唤醒空闲线程
        }
    }
}
```





## 线程池配置与调优

### 1. 系统属性配置



```
// 启动 JVM 时设置系统属性
// 配置 Default 调度器的核心线程数
-Dkotlinx.coroutines.scheduler.core.pool.size=8

// 配置 IO 调度器的最大并行度
-Dkotlinx.coroutines.io.parallelism=128

// 配置线程空闲存活时间（纳秒）
-Dkotlinx.coroutines.scheduler.keep.alive.sec=60
```

### 2. 代码层面配置



```
// 1. 自定义调度器
val customDispatcher = Executors.newFixedThreadPool(10).asCoroutineDispatcher()

// 2. 使用 limitedParallelism 限制并发
val limitedDispatcher = Dispatchers.IO.limitedParallelism(8)

// 3. 共享线程池
object AppDispatchers {
    val Database = Executors.newFixedThreadPool(4).asCoroutineDispatcher()
    val Network = Executors.newFixedThreadPool(8).asCoroutineDispatcher()
    val Computation = Executors.newWorkStealingPool().asCoroutineDispatcher()
}

// 4. 优雅关闭
suspend fun cleanup() {
    (AppDispatchers.Database.executor as? ExecutorService)?.shutdown()
    (AppDispatchers.Network.executor as? ExecutorService)?.awaitTermination(10, TimeUnit.SECONDS)
}
```



## 高级配置示例

```
// 复杂线程池配置
fun createCustomDispatcher(
    name: String,
    coreSize: Int = Runtime.getRuntime().availableProcessors(),
    maxSize: Int = 64,
    keepAliveSeconds: Long = 60L
): CoroutineDispatcher {
    
    val threadFactory = ThreadFactory { r ->
        Thread(r, "$name-${atomicCounter.incrementAndGet()}").apply {
            isDaemon = true
            priority = Thread.NORM_PRIORITY
        }
    }
    
    val executor = ThreadPoolExecutor(
        coreSize,                    // 核心线程数
        maxSize,                     // 最大线程数
        keepAliveSeconds, TimeUnit.SECONDS,  // 空闲时间
        LinkedBlockingQueue(1000),   // 任务队列
        threadFactory,
        ThreadPoolExecutor.CallerRunsPolicy()  // 拒绝策略
    ).apply {
        allowCoreThreadTimeOut(true)  // 核心线程也可超时
    }
    
    return executor.asCoroutineDispatcher().apply {
        // 注册关闭钩子
        Runtime.getRuntime().addShutdownHook(Thread {
            executor.shutdown()
            executor.awaitTermination(10, TimeUnit.SECONDS)
        })
    }
}
```







## 调试与监控

### 1. 线程池状态监控

```
// 监控线程池状态
fun monitorDispatcher(dispatcher: CoroutineDispatcher) {
    val executor = (dispatcher as? ExecutorCoroutineDispatcher)?.executor
    if (executor is ThreadPoolExecutor) {
        with(executor) {
            println("""
                活跃线程: ${activeCount}
                核心线程: ${corePoolSize}
                最大线程: ${maximumPoolSize}
                队列大小: ${queue.size}
                完成任务: ${completedTaskCount}
            """.trimIndent())
        }
    }
}

// 在协程中获取当前线程信息
suspend fun printThreadInfo() = withContext(Dispatchers.IO) {
    val thread = Thread.currentThread()
    println("""
        线程: ${thread.name}
        线程组: ${thread.threadGroup?.name}
        线程ID: ${thread.id}
        是否守护线程: ${thread.isDaemon}
        优先级: ${thread.priority}
    """.trimIndent())
}
```

### 2. 性能分析配置



```
// 启用协程调试
System.setProperty("kotlinx.coroutines.debug", "on")

// 或启动参数
-Dkotlinx.coroutines.debug=on
-Dkotlinx.coroutines.stacktrace.recovery=true

// 创建带有名称的协程
val job = CoroutineScope(Dispatchers.IO + CoroutineName("NetworkRequest")).launch {
    // 协程会显示名称，便于调试
    println(coroutineContext[CoroutineName])
}
```

## 七、与虚拟线程（Project Loom）的集成



```
// 在支持虚拟线程的 JVM 上
@OptIn(ExperimentalCoroutinesApi::class)
fun useVirtualThreads() {
    // 创建虚拟线程执行器
    val executor = Executors.newVirtualThreadPerTaskExecutor()
    val dispatcher = executor.asCoroutineDispatcher()
    
    runBlocking(dispatcher) {
        // 每个协程在独立的虚拟线程上运行
        repeat(100_000) { i ->
            launch {
                Thread.sleep(1000)  // 不阻塞OS线程
                println("Task $i on ${Thread.currentThread()}")
            }
        }
    }
}
```





## 一、挂起与恢复的本质

### 1. **与传统线程阻塞的对比**

kotlin

kotlin

复制

```
// 线程阻塞（Thread Blocking）
fun blockingExample() {
    println("1. 线程开始执行")
    Thread.sleep(1000)  // 线程被阻塞，不释放CPU
    println("2. 线程恢复")
    // 问题：线程资源被占用，无法执行其他任务
}

// 协程挂起（Coroutine Suspending）
suspend fun suspendingExample() {
    println("1. 协程开始执行")
    delay(1000)  // 协程挂起，线程可以执行其他任务
    println("2. 协程恢复")
    // 优点：线程被释放，可执行其他协程
}
```

**关键区别**：

- 

  **线程阻塞**：线程不释放，CPU 时间片被浪费

- 

  **协程挂起**：协程暂停，线程可执行其他任务

------

## 二、挂起函数的工作原理

### 1. **挂起函数的编译转换（CPS 变换）**

kotlin

kotlin

复制

```
// 源代码
suspend fun fetchUser(id: Int): User {
    delay(1000)
    return User(id, "Alice")
}

// 编译器转换后（概念性展示）
fun fetchUser(id: Int, continuation: Continuation<User>): Any? {
    // 1. 创建状态机
    val sm = object : CoroutineImpl(continuation) {
        var label = 0
        var result: Any? = null
        
        override fun doResume(data: Any?, exception: Throwable?): Any? {
            when (label) {
                0 -> {
                    label = 1
                    // 2. 调用 delay，传入 Continuation
                    delay(1000, this)
                    return COROUTINE_SUSPENDED
                }
                1 -> {
                    // 3. delay 完成，继续执行
                    result = User(id, "Alice")
                    label = 2
                }
            }
            // 4. 恢复执行
            continuation.resume(result as User)
            return result
        }
    }
    
    return sm.doResume(null, null)
}
```

### 2. **CPS 变换详解**

CPS（Continuation-Passing Style）是函数式编程中的概念，编译器自动将挂起函数转换为这种形式：

kotlin

kotlin

复制

```
// 原始挂起函数签名
suspend fun foo(): T

// CPS 变换后签名
fun foo(continuation: Continuation<T>): Any?

// 返回值含义：
// 1. COROUTINE_SUSPENDED: 表示函数挂起
// 2. 其他值: 表示直接返回结果
// 3. 抛出异常: 表示执行失败
```

**`Continuation`接口**：

kotlin

kotlin

复制

```
interface Continuation<in T> {
    val context: CoroutineContext
    fun resumeWith(result: Result<T>)
}
```

------

## 三、状态机实现

### 1. **多挂起点的状态机**

kotlin

kotlin

复制

```
// 源代码：有多个挂起点的函数
suspend fun fetchData(): String {
    val user = fetchUser()      // 挂起点1
    val avatar = fetchAvatar()  // 挂起点2
    return "$user with $avatar"
}

// 编译器生成的状态机
fun fetchData(continuation: Continuation<String>): Any? {
    class FetchDataStateMachine(
        completion: Continuation<String>
    ) : ContinuationImpl(completion) {
        
        // 状态变量
        var label = 0
        var user: User? = null
        var avatar: Avatar? = null
        
        override fun invokeSuspend(result: Any?): Any? {
            when (label) {
                0 -> {
                    // 初始状态
                    label = 1
                    // 调用第一个挂起函数
                    val result = fetchUser(this)
                    if (result === COROUTINE_SUSPENDED) return COROUTINE_SUSPENDED
                    // 直接返回结果，继续执行
                    return doResume(result)
                }
                1 -> {
                    // 第一个挂起点恢复
                    user = result as User
                    label = 2
                    // 调用第二个挂起函数
                    val result = fetchAvatar(this)
                    if (result === COROUTINE_SUSPENDED) return COROUTINE_SUSPENDED
                    return doResume(result)
                }
                2 -> {
                    // 第二个挂起点恢复
                    avatar = result as Avatar
                    label = 3
                    // 返回最终结果
                    return "$user with $avatar"
                }
                else -> throw IllegalStateException("Already completed")
            }
        }
    }
}
```

### 2. **局部变量保存与恢复**

kotlin

kotlin

复制

```
// 源代码
suspend fun processItems(items: List<String>) {
    var processed = 0
    for (item in items) {
        delay(100)  // 挂起点
        processed++
    }
    println("Processed: $processed")
}

// 编译器处理：将局部变量提升为状态机的字段
class ProcessItemsStateMachine : ContinuationImpl {
    var label = 0
    var processed: Int = 0
    var items: List<String>? = null
    var iterator: Iterator<String>? = null
    var item: String? = null
    
    // 状态机实现...
}
```

**关键点**：所有在挂起点之后使用的局部变量，都会被保存到状态机中。

------

## 四、挂起恢复的完整流程

### 1. **从创建到执行的完整流程**

kotlin

kotlin

复制

```
// 1. 创建协程
val job = CoroutineScope(Dispatchers.IO).launch {
    println("Start")
    val data = fetchData()  // 挂起点
    println("Data: $data")
}

// 实际执行流程
launch {
    // 1. 创建 Continuation
    val completion = object : Continuation<Unit> {
        override val context = ...
        override fun resumeWith(result: Result<Unit>) {
            // 协程完成时的回调
        }
    }
    
    // 2. 创建协程实例
    val coroutine = BuildersKt.createCoroutineUnintercepted({ 
        // 协程体
        println("Start")
        val data = fetchData()
        println("Data: $data")
    }, completion)
    
    // 3. 开始执行
    coroutine.resume(Unit)
}
```

### 2. **挂起与恢复的时间线**





```
时间线：
1. 主线程：启动协程
2. IO线程：执行协程体
3. 遇到挂起点：保存状态，返回 COROUTINE_SUSPENDED
4. IO线程：可执行其他任务
5. 延迟结束：调用 Continuation.resume()
6. IO线程：恢复协程执行
7. 完成：调用 completion.resumeWith()
```

# Flow、StateFlow、SharedFlow 深度解析

这三者是 Kotlin 协程中处理数据流的核心组件，理解它们的区别和适用场景非常重要。

###  **核心思想**

1. 

   **Flow** 是**声明式**的数据流，用于描述**如何产生数据**

2. 

   **StateFlow** 是**状态容器**，用于管理**当前的UI状态**

3. 

   **SharedFlow** 是**事件总线**，用于广播**一次性事件**



### **记忆口诀**

- 

  **Flow**：冷、懒、独 - 冷流、惰性、每个收集者独立

- 

  **StateFlow**：热、初、新 - 热流、有初始值、只关心最新

- 

  **SharedFlow**：热、共、事 - 热流、共享、适合事件

# Runtime.getRuntime().addShutdownHook()

## 不推荐在Android中使用，因为：
​        // 1. Android ART/Dalvik虚拟机行为与标准JVM不同
​        // 2. 进程可能被系统直接杀死，不会触发关闭钩子
​        // 3. 应在适当的生命周期中管理资源





这段代码是**注册一个JVM关闭钩子**，其核心作用是在**JVM正常关闭时，自动优雅地关闭线程池**，确保资源被正确释放。

```
Runtime.getRuntime().addShutdownHook(Thread {
    // 这段代码会在JVM关闭时被执行
    executor.shutdown()  // 1. 启动有序关闭
    executor.awaitTermination(10, TimeUnit.SECONDS)  // 2. 等待任务完成
})
```



## **实际应用场景**

```
// 场景1：命令行应用
fun main() {
    val executor = Executors.newFixedThreadPool(4)
    Runtime.getRuntime().addShutdownHook(Thread {
        println("正在关闭线程池...")
        executor.shutdown()
        if (!executor.awaitTermination(5, TimeUnit.SECONDS)) {
            println("线程池未在5秒内关闭，强制关闭")
            executor.shutdownNow()  // 尝试取消所有任务
        }
    })
    
    // 主程序逻辑
    executor.submit { /* 长时间任务 */ }
    
    // 程序运行...
    // 当用户按Ctrl+C时，关闭钩子会被触发
}

// 场景2：服务器应用
class ServerApplication {
    private val threadPool = Executors.newCachedThreadPool()
    
    init {
        // 注册关闭钩子
        Runtime.getRuntime().addShutdownHook(Thread {
            logger.info("开始优雅关闭...")
            gracefulShutdown()
        })
    }
    
    private fun gracefulShutdown() {
        threadPool.shutdown()
        try {
            if (!threadPool.awaitTermination(10, TimeUnit.SECONDS)) {
                logger.warn("强制关闭剩余任务")
                threadPool.shutdownNow()
            }
        } catch (e: InterruptedException) {
            threadPool.shutdownNow()
            Thread.currentThread().interrupt()
        }
    }
}
    
```



# 集合

**Kotlin 集合的"只读"特性是：**

1. 

   **编译时安全，运行时不保证** - 类型系统提供保护，但可绕过

2. 

   **视图，不是副本** - 只读接口只是可变集合的只读视图

3. 

   **与 Java 互操作是薄弱环节** - Java 看不到 Kotlin 的类型约束

4. 

   **需要主动防御** - 在关键位置使用防御性副本或真正不可变集合





# by lazy



## 一、`by lazy`的线程安全实现

Kotlin 标准库中的 `lazy`提供了三种线程安全模式，每种都有不同的实现和适用场景：

### 1. **三种模式概览**



```
// 1. 默认模式 - 双重检查锁
val lazyValue1: String by lazy { 
    println("computed!") 
    "Hello"
}

// 2. 发布模式 - 允许多线程初始化，接受第一个结果
val lazyValue2: String by lazy(LazyThreadSafetyMode.PUBLICATION) {
    println("computed in thread: ${Thread.currentThread().name}")
    "World"
}

// 3. 无锁模式 - 非线程安全
val lazyValue3: String by lazy(LazyThreadSafetyMode.NONE) {
    "No Thread Safe"
}
```

### 2. **源码实现分析**

#### **`SynchronizedLazyImpl`- 双重检查锁（默认）**



```
private class SynchronizedLazyImpl<out T>(
    initializer: () -> T,
    lock: Any? = null
) : Lazy<T>, Serializable {
    
    // 使用 @Volatile 保证可见性
    @Volatile private var initializer: (() -> T)? = initializer
    @Volatile private var _value: Any? = UNINITIALIZED_VALUE
    private val lock = lock ?: this
    
    // 双重检查锁实现
    override val value: T
        get() {
            val _v1 = _value
            // 第一次检查：避免已初始化时的锁开销
            if (_v1 !== UNINITIALIZED_VALUE) {
                @Suppress("UNCHECKED_CAST")
                return _v1 as T
            }
            
            return synchronized(lock) {
                val _v2 = _value
                // 第二次检查：防止重复初始化
                if (_v2 !== UNINITIALIZED_VALUE) {
                    @Suppress("UNCHECKED_CAST")
                    _v2 as T
                } else {
                    // 执行初始化
                    val typedValue = initializer!!()
                    _value = typedValue
                    initializer = null  // 释放初始化器引用
                    typedValue
                }
            }
        }
    
    override fun isInitialized(): Boolean = 
        _value !== UNINITIALIZED_VALUE
}
```

**性能优化点**：

1. 

   使用 `@Volatile`保证 `_value`的可见性

2. 

   第一次无锁检查，减少锁开销

3. 

   初始化后清空 `initializer`，释放内存

4. 

   支持自定义锁对象

#### **`SafePublicationLazyImpl`- 发布模式**



```
private class SafePublicationLazyImpl<out T>(
    initializer: () -> T
) : Lazy<T>, Serializable {
    
    @Volatile private var initializer: (() -> T)? = initializer
    @Volatile private var _value: Any? = UNINITIALIZED_VALUE
    
    // 使用 AtomicReferenceFieldUpdater 进行 CAS 操作
    override val value: T
        get() {
            val value = _value
            if (value !== UNINITIALIZED_VALUE) {
                @Suppress("UNCHECKED_CAST")
                return value as T
            }
            
            val initializerValue = initializer
            if (initializerValue != null) {
                // 执行初始化
                val newValue = initializerValue()
                // 使用 CAS 设置值
                if (valueUpdater.compareAndSet(this, UNINITIALIZED_VALUE, newValue)) {
                    initializer = null
                    return newValue
                }
            }
            // 如果 CAS 失败，说明其他线程已设置，返回已设置的值
            @Suppress("UNCHECKED_CAST")
            return _value as T
        }
    
    companion object {
        // 原子字段更新器
        private val valueUpdater = 
            java.util.concurrent.atomic.AtomicReferenceFieldUpdater.newUpdater(
                SafePublicationLazyImpl::class.java,
                Any::class.java,
                "_value"
            )
    }
}
```

**特点**：

- 

  允许多个线程同时执行初始化代码

- 

  只接受第一个完成的结果

- 

  适用于初始化成本高，重复初始化可接受的场景

#### **`UnsafeLazyImpl`- 无锁模式**



```
internal class UnsafeLazyImpl<out T>(
    initializer: () -> T
) : Lazy<T>, Serializable {
    
    private var initializer: (() -> T)? = initializer
    private var _value: Any? = UNINITIALIZED_VALUE
    
    override val value: T
        get() {
            if (_value === UNINITIALIZED_VALUE) {
                _value = initializer!!()
                initializer = null
            }
            @Suppress("UNCHECKED_CAST")
            return _value as T
        }
}
```

**注意**：这是 `internal`类，仅供 Kotlin 内部使用。

### 3. **性能对比与选择**

| 模式             | 线程安全   | 性能 | 适用场景                   |
| ---------------- | ---------- | ---- | -------------------------- |
| **SYNCHRONIZED** | ✅ 强一致   | 中等 | 默认选择，需要强一致性     |
| **PUBLICATION**  | ✅ 最终一致 | 较好 | 初始化成本高，允许多次执行 |
| **NONE**         | ❌ 不安全   | 最好 | 单线程、主线程、同步控制   |

**选择建议**：



```
// 场景1：UI 组件初始化（主线程）
val viewModel: MyViewModel by lazy(LazyThreadSafetyMode.NONE) { 
    MyViewModel() 
}

// 场景2：网络客户端（多线程访问）
val httpClient: OkHttpClient by lazy { 
    OkHttpClient.Builder().build() 
}

// 场景3：昂贵的初始化（可接受重复计算）
val expensiveConfig: Config by lazy(LazyThreadSafetyMode.PUBLICATION) {
    computeExpensiveConfig()  // 多个线程可能同时计算
}
```



# 内联函数

## 一、`inline`函数的工作原理

### 1. **编译时展开**

kotlin

kotlin

复制

```
// Kotlin 源代码
inline fun <T> compute(value: T, block: (T) -> Unit) {
    println("Before")
    block(value)
    println("After")
}

fun main() {
    compute(10) { value ->
        println("Processing: $value")
    }
}

// 编译后（伪代码）
fun main() {
    // inline 函数体被插入到调用处
    println("Before")
    val value = 10
    println("Processing: $value")
    println("After")
}
```

**字节码对比**：





复制

```
# 非内联调用
INVOKESTATIC ComputeKt.compute
INVOKEINTERFACE Function1.invoke

# 内联调用
GETSTATIC System.out
LDC "Before"
INVOKEVIRTUAL PrintStream.println
...  # 内联的代码
```

### 2. **内联条件**

kotlin

kotlin

复制

```
// 可以被内联的函数
inline fun smallFunction() { /* 小函数体 */ }

// 不会被内联的情况
inline fun largeFunction() {
    // 1. 函数体过大（超过字节码限制）
    // 2. 递归调用
    // 3. 包含非局部返回
}
```

------

## 二、性能优势分析

### 1. **消除 Lambda 对象创建**

kotlin

kotlin

复制

```
// 测试1：高阶函数调用开销
fun measureLambdaAllocation() {
    val iterations = 1_000_000
    
    // 非内联版本
    fun processNonInline(data: List<Int>, action: (Int) -> Unit) {
        data.forEach(action)
    }
    
    // 内联版本
    inline fun processInline(data: List<Int>, action: (Int) -> Unit) {
        data.forEach(action)
    }
    
    val data = List(1000) { it }
    
    // 测试结果（近似）：
    // 非内联：每次调用创建 Lambda 对象，内存分配开销
    // 内联：无 Lambda 对象创建，直接展开代码
}
```

**性能数据**（参考基准测试）：

| 场景         | 非内联 (ns) | 内联 (ns) | 提升 |
| ------------ | ----------- | --------- | ---- |
| 简单 Lambda  | ~150        | ~5        | 30x  |
| 复杂 Lambda  | ~300        | ~10       | 30x  |
| 循环 1000 次 | ~15,000     | ~500      | 30x  |

### 2. **减少方法调用开销**

kotlin

kotlin

复制

```
// 方法调用栈对比
fun outer() {
    middle()  // 方法调用1
}

fun middle() {
    inner()   // 方法调用2
}

fun inner() {
    println("Hello")
}

// 内联后：
fun outer() {
    println("Hello")  // 直接执行
}
```

**开销分析**：





复制

```
方法调用开销（每次）：
1. 栈帧分配
2. 参数压栈
3. 返回地址保存
4. 局部变量表初始化
5. 方法表查找（虚方法）

内联消除：
- 所有方法调用开销
- 可能的内联优化
```

### 3. **支持 `reified`类型参数**

kotlin

kotlin

复制

```
// 非内联无法实现
fun <T> getTypeName(): String {
    return T::class.java.name  // 编译错误：类型擦除
}

// 内联可以实现
inline fun <reified T> getTypeName(): String {
    return T::class.java.name  // 可行：类型在调用处具体化
}
```

------

## 三、性能劣势分析

### 1. **代码膨胀（Code Bloat）**

kotlin

kotlin

复制

```
// 内联导致代码重复
inline fun <T> debugLog(tag: String, block: () -> T): T {
    val start = System.nanoTime()
    val result = block()
    val duration = System.nanoTime() - start
    println("$tag took ${duration}ns")
    return result
}

// 在 100 个地方调用
fun process1() = debugLog("process1") { compute1() }
fun process2() = debugLog("process2") { compute2() }
// ...
fun process100() = debugLog("process100") { compute100() }

// 结果：相同的日志代码被复制 100 次！
```

**代码膨胀影响**：

kotlin

kotlin

复制

```
// 原始大小
fun main() {
    repeat(1000) {
        process(compute())
    }
}

// 假设 process 是 100 字节的 inline 函数
// 内联后大小：1000 * 100 = 100KB
// 非内联大小：100 + 20（调用）* 1000 = 20KB
```

**实际测试数据**：

kotlin

kotlin

复制

```
// 测试代码膨胀
inline fun smallInline() { /* 20 bytes */ }
fun callManyTimes() {
    repeat(100_000) {
        smallInline()  // 产生 2MB 重复代码
    }
}
```

### 2. **编译时间增加**

kotlin

kotlin

复制

```
// 编译过程对比
非内联函数：
1. 编译函数体
2. 生成调用指令

内联函数：
1. 编译函数体
2. 在每个调用点解析
3. 在每个调用点类型检查
4. 在每个调用点生成代码
5. 优化重复代码
```

**编译时间增长**：

- 

  小项目：可忽略

- 

  大型项目：可能增加 10-30% 编译时间

- 

  热重载：内联函数修改会导致更多文件重新编译

### 3. **调试困难**

kotlin

kotlin

复制

```
inline fun process(value: Int, transform: (Int) -> Int): Int {
    val result = transform(value)
    // 断点在这里
    return result * 2
}

fun main() {
    val x = process(5) { it * 3 }
    // 调试时：
    // 1. 调用栈不清晰
    // 2. 无法单步进入 transform
    // 3. 局部变量可能被优化掉
}
```

**调试问题**：

1. 

   调用栈展开，难以追踪

2. 

   局部变量可能被编译器优化

3. 

   断点可能无法准确定位

### 4. **无法使用高级特性**

kotlin

kotlin

复制

```
// 限制1：不能递归
inline fun recursive(n: Int) {
    if (n > 0) {
        recursive(n - 1)  // 编译错误：递归内联
    }
}

// 限制2：不能访问私有成员
private class PrivateData(val value: Int)

inline fun accessPrivate(data: PrivateData) {
    println(data.value)  // 如果内联到其他模块，无法访问私有成员
}
```

------

## 四、不同场景下的性能表现

### 1. **场景分析表**

| 场景             | 适合内联 | 理由               | 性能影响 |
| ---------------- | -------- | ------------------ | -------- |
| **简单工具函数** | ✅        | 函数体小，频繁调用 | 显著提升 |
| **集合操作**     | ✅        | 避免 Lambda 分配   | 中等提升 |
| **DSL 构建器**   | ✅        | 改善语法，消除开销 | 显著提升 |
| **日志/调试**    | ❌        | 代码膨胀严重       | 负面影响 |
| **复杂业务逻辑** | ❌        | 函数体大，调用少   | 负面影响 |
| **公共 API**     | ⚠️        | 需考虑二进制兼容性 | 不确定   |

### 2. **基准测试对比**

kotlin

kotlin

复制

```
@State(Scope.Benchmark)
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
open class InlineBenchmark {
    
    val data = (1..1000).toList()
    
    // 1. 非内联高阶函数
    fun processNonInline(list: List<Int>, action: (Int) -> Int): List<Int> {
        return list.map(action)
    }
    
    // 2. 内联高阶函数
    inline fun processInline(list: List<Int>, action: (Int) -> Int): List<Int> {
        return list.map(action)
    }
    
    // 3. 手写循环（基准）
    fun processManual(list: List<Int>): List<Int> {
        val result = mutableListOf<Int>()
        for (item in list) {
            result.add(item * 2)
        }
        return result
    }
    
    @Benchmark
    fun testNonInline(): List<Int> {
        return processNonInline(data) { it * 2 }
    }
    
    @Benchmark
    fun testInline(): List<Int> {
        return processInline(data) { it * 2 }
    }
    
    @Benchmark
    fun testManual(): List<Int> {
        return processManual(data)
    }
}
```

**预期结果**：





复制

```
Benchmark                 Mode  Cnt     Score    Error  Units
InlineBenchmark.testNonInline  avgt    5  15678.123 ± 123.456  ns/op
InlineBenchmark.testInline     avgt    5   5234.567 ±  89.012  ns/op
InlineBenchmark.testManual     avgt    5   5100.123 ±  78.901  ns/op
```

------

## 五、优化策略

### 1. **选择性内联**

kotlin

kotlin

复制

```
// 1. 只内联特定的 Lambda 参数
inline fun processData(
    data: List<Int>,
    noinline validator: (Int) -> Boolean,  // 不内联
    transform: (Int) -> Int               // 内联
) {
    data.filter(validator).map(transform)
}

// 2. 使用 @InlineOnly 注解
@InlineOnly
inline fun internalHelper(value: Int): Int {
    return value * 2  // 仅供内部使用
}
```

### 2. **避免过度内联**

kotlin

kotlin

复制

```
// 不好的实践：过度内联
inline fun processUser(
    userId: Int,
    onSuccess: (User) -> Unit,
    onError: (Throwable) -> Unit
) {
    try {
        val user = fetchUser(userId)  // 复杂操作
        onSuccess(user)
    } catch (e: Exception) {
        onError(e)
    }
}

// 好的实践：分离关注点
inline fun processUserInline(
    userId: Int,
    crossinline onSuccess: (User) -> Unit,
    noinline onError: (Throwable) -> Unit
) {
    processUserNonInline(userId, onSuccess, onError)
}

fun processUserNonInline(
    userId: Int,
    onSuccess: (User) -> Unit,
    onError: (Throwable) -> Unit
) {
    // 复杂逻辑，不内联
}
```

### 3. **使用内联类减少包装开销**

kotlin

kotlin

复制

```
// 内联类：运行时无包装开销
@JvmInline
value class UserId(val value: Int)

inline fun processUser(userId: UserId) {
    println("Processing: ${userId.value}")
}
```

------

## 六、编译器优化细节

### 1. **智能内联决策**

编译器会根据多种因素决定是否内联：

kotlin

kotlin

复制

```
// 因素1：函数大小
inline fun small() { /* 编译决定内联 */ }
inline fun large() { /* 可能不内联，如果太大 */ }

// 因素2：调用频率
fun hotPath() {
    repeat(1000) {
        smallHelper()  // 频繁调用，倾向于内联
    }
}

// 因素3：Lambda 复杂度
inline fun withComplexLambda(block: () -> Unit) {
    // Lambda 很复杂时，内联可能不划算
}
```

### 2. **跨模块内联**

kotlin

kotlin

复制

```
// Module A
inline fun publicInline() { /* 可被其他模块内联 */ }

// Module B
fun caller() {
    publicInline()  // 内联展开
}
```

**限制**：

- 

  需要模块编译时保留内联信息

- 

  修改内联函数需要重新编译所有调用模块

### 3. **内联与泛型**

kotlin

kotlin

复制

```
// 内联泛型函数的优化
inline fun <T> List<T>.customMap(transform: (T) -> T): List<T> {
    val result = ArrayList<T>(size)
    for (item in this) {
        result.add(transform(item))
    }
    return result
}

// 调用具体类型时，编译器可能特化
val ints = listOf(1, 2, 3)
ints.customMap { it * 2 }  // 可能生成特化版本
```

------

## 七、实际项目中的最佳实践

### 1. **工具类函数**



```
// 适合内联：小、频繁使用
inline fun measureTime(block: () -> Unit): Long {
    val start = System.nanoTime()
    block()
    return System.nanoTime() - start
}

inline fun <T> T.applyIf(condition: Boolean, block: T.() -> Unit): T {
    if (condition) block()
    return this
}
```

### 2. **集合操作扩展**



```
// 适合：避免中间集合
inline fun <T, R> Iterable<T>.mapNotNull(
    crossinline transform: (T) -> R?
): List<R> {
    return mapNotNullTo(ArrayList(), transform)
}
```

### 3. **DSL 构建器**



```
// 适合：改善 API 体验
inline fun html(block: HTML.() -> Unit): HTML {
    val html = HTML()
    html.block()
    return html
}

// 使用
val page = html {
    head { /* ... */ }
    body { /* ... */ }
}
```

### 4. **避免内联的场景**



```
// 1. 大型工具类
object StringUtils {
    // 不要内联
    fun complexFormat(str: String): String { /* 100+ 行 */ }
}

// 2. 递归算法
fun factorial(n: Int): Int {
    return if (n <= 1) 1 else n * factorial(n - 1)
}

// 3. 公共 API
interface Repository {
    // 不要内联，影响二进制兼容性
    fun findById(id: Int): User
}
```

------

## 八、性能测量工具

### 1. **使用 JMH 基准测试**



```
dependencies {
    testImplementation("org.openjdk.jmh:jmh-core:1.35")
    testAnnotationProcessor("org.openjdk.jmh:jmh-generator-annprocess:1.35")
}
```

### 2. **使用 Profiler 分析**

```
// 1. 测量内存分配
Debug.startAllocCounting()
inlineFunction()
val allocCount = Debug.getGlobalAllocCount()

// 2. 使用 Android Profiler
// 查看内联前后的内存和CPU使用
```

### 3. **代码大小分析**



```
# 查看生成的字节码大小
javap -c -p MyClass.class | wc -l

# 使用 proguard 分析
./gradlew :app:minifyReleaseWithProguard
```



# reified

reified类型参数是 Kotlin 中的一个特性，它允许在泛型函数中保留类型信息，从而在运行时可以访问具体的类型。在 Java 中，由于类型擦除，泛型类型信息在运行时是不可用的。Kotlin 的 reified关键字通过内联函数（inline function）来突破这一限制。



## 一、类型擦除问题

在 Java 中，泛型是在编译时进行类型检查，在运行时，泛型类型信息会被擦除。例如：

```
// Java
public <T> void printType(T item) {
    System.out.println(T.class); // 编译错误，因为T在运行时不可知
}
```

这是因为 Java 的泛型是通过类型擦除来实现的，主要是为了向后兼容。

## 二、Kotlin 的 `reified`解决方案

Kotlin 通过内联函数和 `reified`类型参数，使得在函数体内可以访问具体的类型信息。具体原理如下：

1. 

   **内联函数**：内联函数在编译时会将函数体直接插入到每一个调用处，而不是进行普通的函数调用。

2. 

   **reified 类型参数**：当内联函数使用 `reified`修饰类型参数时，在函数体内可以使用该类型参数，因为在实际调用处，编译器知道具体的类型，并会在编译时将具体类型替换到内联函数体中。

## 三、实现原理

### 示例

```
inline fun <reified T> printType(item: T) {
    println(T::class.java)
}

fun main() {
    printType("Hello")  // 输出：class java.lang.String
    printType(123)      // 输出：class java.lang.Integer
}
```

