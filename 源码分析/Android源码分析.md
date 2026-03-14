# Android源码分析



# Actviity启动流程

## 核心流程

startActivity() → Instrumentation → AMS → 
ActivityThread → onCreate() → onStart() → onResume()



## 源码关键节点

```
// 1. 客户端进程
Activity.startActivity()
  ↓
Instrumentation.execStartActivity()
  ↓
ActivityManager.getService().startActivity()  // 跨进程调用AMS

// 2. 系统进程（AMS）
ActivityManagerService.startActivity()
  ↓
ActivityStarter.startActivityMayWait()
  ↓
ActivityStack.resumeTopActivity()

// 3. 回到客户端进程
ActivityThread.handleLaunchActivity()
  ↓
Instrumentation.callActivityOnCreate()
  ↓
Activity.performCreate()
```



# **Window 与 WindowManager**

### 核心关系

```
Activity ← PhoneWindow ← DecorView ← ContentView
    ↓           ↓           ↓
WindowManager ← WindowManagerImpl ← WindowManagerGlobal
```

### 窗口添加流程



```
// WindowManagerImpl.java
public void addView(View view, ViewGroup.LayoutParams params) {
    WindowManagerGlobal.getInstance().addView(view, params);
}

// WindowManagerGlobal.java
public void addView(View view, ViewGroup.LayoutParams params, Display display, Window parentWindow) {
    // 1. 创建 ViewRootImpl
    ViewRootImpl root = new ViewRootImpl(context, display);
    // 2. 添加 View
    root.setView(view, wparams, panelParentView);
}
```





# AMS

### 核心功能架构



```
AMS 管理四大组件生命周期
  ├── ActivityStack       // Activity 栈管理
  ├── ProcessList         // 进程管理
  ├── ActiveServices      // Service 管理
  └── BroadcastQueue      // 广播管理
```

### 进程管理源码



```
// AMS 进程优先级
static final int
    FOREGROUND_APP_ADJ = 0,     // 前台应用
    VISIBLE_APP_ADJ = 100,      // 可见应用
    PERCEPTIBLE_APP_ADJ = 200,  // 可感知应用
    CACHED_APP_MAX_ADJ = 906,   // 缓存应用
    CACHED_APP_MIN_ADJ = 999;   // 空进程
```



# WMS



### 窗口管理核心





```
// WindowManagerService.java
class WindowManagerService extends IWindowManager.Stub {
    // 窗口层级（Type）
    // TYPE_APPLICATION        应用窗口
    // TYPE_SYSTEM_ALERT       系统提示窗口
    // TYPE_TOAST              Toast窗口
    // TYPE_INPUT_METHOD       输入法窗口
    
    // 窗口状态
    // NO_SURFACE
    // DRAW_PENDING
    // COMMIT_DRAW_PENDING
    // READY_TO_SHOW
    // HAS_DRAWN
}
```



# ClassLoader 与热修复

## ClassLoader：它是 JVM 或 Android Dalvik/ART 虚拟机的一部分，负责在运行时将`.class`或`.dex`文件加载到内存，并转换成可执行的类或组件。



**核心机制**：

- 

  **双亲委托模型**：当一个类加载器需要加载类时，它首先会委托给其父加载器尝试加载。只有当所有父加载器都无法加载时，自己才会尝试。这保证了Java核心库类的安全性和唯一性。

- 

  **Android中的主要ClassLoader**：

  - 

    `BootClassLoader`： 加载系统框架级别的类（如`java.*`, `android.*`）。

  - 

    `PathClassLoader`： 通常用于加载应用程序自身的类和安装在系统中的应用。

  - 

    `DexClassLoader`： **可以加载存储在文件系统（如SD卡或应用私有目录）中的`.dex`或`.apk/.jar`文件**。这是实现热修复、插件化的关键。

**关键点**：在Android中，每个`ClassLoader`实例都有一个自己的“已加载类”的命名空间。**同一个类被不同的`ClassLoader`加载，会被虚拟机视为两个不同的类**。



## 热修复

**定义**：指在不重新安装或重启应用的情况下，动态修复线上Bug（通常替换有问题的类文件）的技术。

**核心目标**：快速修复线上崩溃、功能异常，无需用户下载完整新版本。



### 双亲委派源码





```
// ClassLoader.java
protected Class<?> loadClass(String name, boolean resolve) {
    // 1. 检查是否已加载
    Class<?> c = findLoadedClass(name);
    if (c == null) {
        try {
            if (parent != null) {
                // 2. 委托父类加载
                c = parent.loadClass(name, false);
            } else {
                c = findBootstrapClassOrNull(name);
            }
        } catch (ClassNotFoundException e) {
            // 父类找不到
        }
        
        if (c == null) {
            // 3. 自己加载
            c = findClass(name);
        }
    }
    return c;
}
```

### 热修复原理





```
// 1. 类加载替换
DexClassLoader → DexPathList → Element[] dexElements
// 插入补丁 dex 到数组最前

// 2. Instant Run
// 重启 Activity：替换 Resources
// 温重启：替换 ClassLoader
// 热重启：修改方法指针
```





# SharedPreference 源码缺陷



### 核心问题



```
// 1. 全量写入
EditorImpl.apply() {
    MemoryCommitResult mcr = commitToMemory();
    // 异步写入，但全量写入文件
    QueuedWork.queue(writeToDiskRunnable);
}

// 2. ANR 风险
ActivityThread.handleStopActivity() {
    // 等待 SP 写入完成
    QueuedWork.waitToFinish();
}
```

### 优化方案

- 

  使用 MMKV

- 

  使用 DataStore

- 

  拆分小文件





### 陷阱一：异步写入的“幽灵” - `apply()`的可靠性

`apply()`看似是异步无等待的“安全”写入，实则有巨大隐患。

- 

  **源码逻辑**：`apply()`会先将数据写入内存 Map，然后创建一个 `awaitCommit`的 `Runnable`放入 `QueuedWork`队列，最后**异步**将 Map 数据写入 XML 文件。

- 

  **核心陷阱**：

  1. 

     **Activity 生命周期可能等待**：在 `ActivityThread`的 `handlePauseActivity`、`handleStopActivity`等关键生命周期节点，会调用 `QueuedWork.waitToFinish()`，**阻塞主线程**直到所有通过 `apply()`提交的写入完成。这可能导致界面卡顿甚至 ANR。

  2. 

     **崩溃导致数据丢失**：`apply()`提交后，如果应用在文件写入完成前发生崩溃或被杀，**数据会永久丢失**，因为写入是异步的，没有事务性保证。

  3. 

     **无写入失败回调**：整个过程没有任何回调告知写入成功或失败。

**结论**：`apply()`是“发起即忘”，**不保证最终一致性**。对需要强一致性的数据（如用户设置开关），应使用有返回值的 `commit()`（注意在主线程用可能导致卡顿）。

------

### 陷阱二：主线程的“隐形炸弹” - 首次加载与全量读写

- 

  **首次加载阻塞**：首次调用 `getSharedPreferences`时，**会同步（可能阻塞调用线程）加载整个 XML 文件到内存 Map**。如果文件很大或主线程调用，直接导致界面卡顿。

- 

  **全量更新**：每次 `commit/apply`都是将**整个内存 Map 序列化为 XML 并完全覆盖写入文件**，而不是增量更新。即使你只修改一个键值，也会重写整个文件。文件越大，I/O 开销越大。

------

### 陷阱三：多进程的“假象” - `MODE_MULTI_PROCESS`

此模式**已废弃**且完全不可靠。

- 

  **源码逻辑**：它并非真正的跨进程内存共享。只是在每次 `getSharedPreferences`时，检查文件最后修改时间，如果发现被其他进程修改过，就**重新加载整个文件**。

- 

  **核心陷阱**：

  1. 

     **严重性能问题**：频繁的 I/O 和反序列化。

  2. 

     **数据竞争与丢失**：两个进程几乎同时写入时，**后写入的会覆盖先写入的**，因为没有锁机制。这是“丢失更新”的经典并发问题。

**绝对禁止**：任何需要多进程共享数据的场景，都应使用 `ContentProvider`、`MMKV`（多进程模式）或 `DataStore`（多进程）等方案。

------

### 陷阱四：类型安全的“幻觉” - 无编译时检查

- 

  **陷阱**：`getString(key, default)`等方法，如果 key 存储的实际类型与读取时声明的类型不符，会在运行时返回默认值，**静默失败**，难以调试。

- 

  **解决**：考虑对 Key 进行封装，或使用 `DataStore`的 `PreferenceDataStore`提供类型安全接口。

------

### 陷阱五：监听器的“内存泄漏”与“失效”

- 

  **内存泄漏风险**：注册的 `OnSharedPreferenceChangeListener`是**强引用**。如果不反注册，会持有 Context 或 Activity 导致无法回收。

- 

  **失效时机**：在某些旧版本 Android 上，`apply()`提交后，**监听器可能在主线程排队执行，而不是在写入线程**。并且，如果注册时使用 `null`作为 key 筛选，行为可能不一致。





# ContentProvider 组件初始化顺序

### 1. **启动顺序概览**

当一个 Android 应用启动时，组件的初始化顺序如下：



```
Application.attachBaseContext() → ContentProvider.onCreate() → Application.onCreate()
```

**关键点**：`ContentProvider`的初始化发生在 `Application.onCreate()`之前！

### 2. **详细初始化流程**



```
// 源码层面（简化）的调用链
ActivityThread.handleBindApplication()
    ├── 创建 Application 对象
    ├── 调用 Application.attachBaseContext()
    ├── 安装 ContentProviders（按顺序初始化所有已声明的 Provider）
    │   ├── 创建 ContentProvider 实例
    │   ├── 调用 ContentProvider.attachInfo()
    │   └── 调用 ContentProvider.onCreate()
    └── 调用 Application.onCreate()
```

**重要**：`ContentProvider`的初始化是在应用主线程中同步执行的！

------

## 二、初始化的触发条件

ContentProvider 不会自动初始化，它只在以下情况下被初始化：

### 1. **首次访问时**（按需初始化）



```
// 当代码首次调用 getContentResolver().query()/insert()/update()/delete() 时
Cursor cursor = getContentResolver().query(
    uri, 
    projection, 
    selection, 
    selectionArgs, 
    sortOrder
);
// 如果对应的 ContentProvider 尚未初始化，系统会在此刻初始化它
```

### 2. **应用启动时自动初始化**（Android 7.0+ 优化）

Android 7.0 (API 24) 开始，系统会在应用启动时**自动初始化所有在清单文件中声明的 ContentProvider**，无论它们是否被使用。

**例外**：可以通过设置 `android:initOrder`控制初始化顺序，但无法完全禁用自动初始化。

### 3. **特殊系统组件的初始化**

一些系统组件会隐式初始化 ContentProvider：

- 

  `JobScheduler`触发 JobService

- 

  `BroadcastReceiver`处理广播

- 

  应用安装/更新后的首次启动

------

## 三、初始化顺序与依赖关系

### 1. **声明顺序决定初始化顺序**

在 `AndroidManifest.xml`中，ContentProvider 按照声明顺序初始化：

```
<application>
    <!-- 先初始化 AProvider -->
    <provider
        android:name=".AProvider"
        android:authorities="com.example.aprovider"
        android:initOrder="100"/>  <!-- 数值越大，优先级越高 -->
    
    <!-- 再初始化 BProvider -->
    <provider
        android:name=".BProvider"
        android:authorities="com.example.bprovider"
        android:initOrder="50"/>
    
    <!-- 最后初始化 CProvider -->
    <provider
        android:name=".CProvider"
        android:authorities="com.example.cprovider"/>
</application>
```

**初始化顺序**：AProvider → BProvider → CProvider（因为 `initOrder`值大的先初始化，未指定则按声明顺序）

### 2. **危险的隐式依赖**

```
// AProvider.kt
class AProvider : ContentProvider() {
    override fun onCreate(): Boolean {
        // 假设这里初始化了某些全局状态
        GlobalConfig.init()
        return true
    }
}

// BProvider.kt  
class BProvider : ContentProvider() {
    override fun onCreate(): Boolean {
        // 错误：依赖 AProvider 初始化的状态
        // 但 BProvider 可能比 AProvider 先初始化！
        if (GlobalConfig.isInitialized) {  // 可能为 false
            doSomething()
        }
        return true
    }
}
```

------

## 四、初始化对启动性能的影响

### 1. **启动延迟问题**

由于 ContentProvider 在主线程同步初始化，如果有耗时操作，会**直接影响应用启动速度**：



```
class MyProvider : ContentProvider() {
    override fun onCreate(): Boolean {
        // 以下操作都会阻塞主线程，导致启动变慢：
        
        // 1. 数据库初始化（尤其大数据库）
        val db = Room.databaseBuilder(...).build()
        
        // 2. 网络请求
        val response = Retrofit.create(...).getData().execute()
        
        // 3. 大量文件 I/O
        val data = File("large_file.json").readText()
        
        // 4. 复杂计算
        val result = calculateHugeData()
        
        return true
    }
}
```

### 2. **启动性能数据**

假设应用有 3 个 ContentProvider：



```
应用启动时间线：
├── 进程创建：50ms
├── Application.attachBaseContext()：20ms
├── ContentProvider A 初始化：120ms ← 阻塞主线程
├── ContentProvider B 初始化：80ms  ← 阻塞主线程
├── ContentProvider C 初始化：200ms ← 阻塞主线程
└── Application.onCreate()：100ms
总启动时间：470ms，其中 ContentProvider 占 400ms！
```

------

## 五、如何优化 ContentProvider 初始化

### 1. **延迟初始化（Lazy Initialization）**



```
class OptimizedProvider : ContentProvider() {
    private val initializationLock = Any()
    private var isInitialized = false
    
    override fun onCreate(): Boolean {
        // 只做必要的轻量级初始化
        return true
    }
    
    override fun query(...): Cursor? {
        ensureInitialized()  // 在首次访问时完成初始化
        return doQuery(...)
    }
    
    private fun ensureInitialized() {
        synchronized(initializationLock) {
            if (!isInitialized) {
                // 将耗时初始化移到后台线程
                CoroutineScope(Dispatchers.IO).launch {
                    performHeavyInitialization()
                    isInitialized = true
                }
            }
        }
    }
    
    private suspend fun performHeavyInitialization() {
        // 在后台线程执行耗时操作
        withContext(Dispatchers.IO) {
            // 数据库初始化、网络请求等
        }
    }
}
```

### 2. **使用异步初始化框架**



```
// 使用 Android Startup 库
class MyProviderInitializer : Initializer<Unit> {
    override fun create(context: Context) {
        // 在后台线程初始化
        CoroutineScope(Dispatchers.IO).launch {
            initDatabase()
            prefetchData()
        }
    }
    
    override fun dependencies(): List<Class<out Initializer<*>>> {
        // 声明依赖的其他 Initializer
        return listOf(WorkManagerInitializer::class.java)
    }
}
```

### 3. **合并多个 Provider**



```
// 避免多个轻量级 Provider，合并为一个
class UnifiedProvider : ContentProvider() {
    private val delegateMap = mapOf(
        "users" to UsersDelegate(),
        "posts" to PostsDelegate(),
        "settings" to SettingsDelegate()
    )
    
    override fun onCreate(): Boolean {
        // 只初始化一次
        return true
    }
    
    override fun query(...): Cursor? {
        val type = detectUriType(uri)
        return delegateMap[type]?.query(...)
    }
}
```

### 4. **完全避免 ContentProvider**

对于仅应用内部使用的数据存储，考虑替代方案：



```
// 使用 SharedPreferences 替代
val prefs = context.getSharedPreferences("my_prefs", Context.MODE_PRIVATE)

// 或使用 Room 数据库（Room 内部会创建 ContentProvider，但可优化）
@Database(entities = [User::class], version = 1, exportSchema = false)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao
    
    companion object {
        // 使用单例延迟初始化
        @Volatile
        private var INSTANCE: AppDatabase? = null
        
        fun getInstance(context: Context): AppDatabase {
            return INSTANCE ?: synchronized(this) {
                INSTANCE ?: buildDatabase(context).also { INSTANCE = it }
            }
        }
        
        private fun buildDatabase(context: Context): AppDatabase {
            return Room.databaseBuilder(
                context.applicationContext,
                AppDatabase::class.java,
                "app_database"
            ).apply {
                // 允许在主线程查询（仅用于调试）
                if (BuildConfig.DEBUG) {
                    allowMainThreadQueries()
                }
                // 异步预加载
                addCallback(object : RoomDatabase.Callback() {
                    override fun onCreate(db: SupportSQLiteDatabase) {
                        super.onCreate(db)
                        // 在后台线程执行初始化
                        GlobalScope.launch(Dispatchers.IO) {
                            // 预填充数据等
                        }
                    }
                })
            }.build()
        }
    }
}
```

------

## 六、常见陷阱与最佳实践

### 1. **避免的陷阱**



```
// 陷阱1：在 onCreate() 中执行网络请求
override fun onCreate(): Boolean {
    val response = apiService.fetchData().execute()  // 阻塞主线程！
    return true
}

// 陷阱2：初始化其他组件
override fun onCreate(): Boolean {
    // 错误：可能依赖其他未初始化的 Provider
    val otherData = context.contentResolver.query(otherUri, ...)
    return true
}

// 陷阱3：假设初始化顺序
override fun onCreate(): Boolean {
    // 错误：假设 SharedPreferences 已初始化
    val prefs = PreferenceManager.getDefaultSharedPreferences(context)
    return true
}
```

### 2. **最佳实践**

| 场景                 | 推荐做法                       | 不推荐做法                              |
| -------------------- | ------------------------------ | --------------------------------------- |
| **数据库初始化**     | 延迟初始化，在首次访问时初始化 | 在 Provider.onCreate() 中初始化大数据库 |
| **网络请求**         | 完全避免，或使用异步回调       | 同步网络请求                            |
| **多 Provider 依赖** | 通过 Application 类协调        | Provider 间直接依赖                     |
| **耗时计算**         | 使用工作线程                   | 在主线程执行                            |
| **文件 I/O**         | 延迟到首次访问时               | 启动时读取大文件                        |

### 3. **监控初始化性能**



```
// 使用 Firebase Performance Monitoring
class MonitoredProvider : ContentProvider() {
    private var initTrace: Trace? = null
    
    override fun onCreate(): Boolean {
        initTrace = Firebase.performance.newTrace("provider_init")
        initTrace?.start()
        
        // 初始化代码...
        
        initTrace?.stop()
        return true
    }
}

// 或使用 Systrace
override fun onCreate(): Boolean {
    Trace.beginSection("MyProvider.onCreate")
    try {
        // 初始化代码...
        return true
    } finally {
        Trace.endSection()
    }
}
```

------

## 七、特殊场景分析

### 1. **多进程应用中的初始化**



```
// 每个进程都会初始化自己的 ContentProvider 实例！
<provider
    android:name=".MyProvider"
    android:authorities="com.example.myprovider"
    android:process=":background"/>  <!-- 会在独立进程初始化 -->
```

**注意**：每个进程（包括主进程、:background 进程等）都会调用一次 `onCreate()`。

### 2. **Library 中的 ContentProvider**

许多三方库会声明自己的 ContentProvider，导致应用启动时初始化多个 Provider：



```
<!-- 常见的库 Provider -->
<provider
    android:name="com.facebook.FacebookContentProvider"
    android:authorities="com.facebook.app.FacebookContentProvider12345"
    android:exported="true"/>

<provider
    android:name="com.google.firebase.provider.FirebaseInitProvider"
    android:authorities="${applicationId}.firebaseinitprovider"
    android:exported="false"
    android:initOrder="100"/>  <!-- 高优先级，最先初始化 -->
    
<provider
    android:name="androidx.startup.InitializationProvider"
    android:authorities="${applicationId}.androidx-startup"
    android:exported="false"/>
```



# Application

## 一、Application 的初始化流程

### 1. **创建时机：进程启动时**

`Application`是在应用进程创建时由系统创建的，比任何 Activity 都早。

java

java

下载

复制



```
// 源码路径：frameworks/base/core/java/android/app/ActivityThread.java
public final class ActivityThread {
    private void handleBindApplication(AppBindData data) {
        // 1. 创建 LoadedApk（应用的包信息）
        data.info = getPackageInfoNoCheck(data.appInfo, data.compatInfo);
        
        // 2. 创建 Instrumentation
        if (data.instrumentationName != null) {
            InstrumentationInfo ii = ...;
            mInstrumentation = (Instrumentation) cl.loadClass(ii.name).newInstance();
        } else {
            mInstrumentation = new Instrumentation();
        }
        
        // 3. 创建 Application
        Application app = data.info.makeApplication(data.restrictedBackupMode, null);
        
        // 4. 初始化 ContentProvider
        installContentProviders(app, data.providers);
        
        // 5. 调用 Application.onCreate()
        mInstrumentation.callApplicationOnCreate(app);
    }
}
```

### 2. **创建过程：makeApplication 方法**

java

java

下载

复制



```
// 源码路径：frameworks/base/core/java/android/app/LoadedApk.java
public Application makeApplication(boolean forceDefaultAppClass, Instrumentation instrumentation) {
    // 单例：一个进程只有一个 Application 实例
    if (mApplication != null) {
        return mApplication;
    }
    
    // 获取 Application 类名
    String appClass = mApplicationInfo.className;
    if (forceDefaultAppClass || appClass == null) {
        appClass = "android.app.Application";  // 默认类
    }
    
    try {
        // 1. 创建 ClassLoader
        java.lang.ClassLoader cl = getClassLoader();
        if (!mPackageName.equals("android")) {
            initializeJavaContextClassLoader();
        }
        
        // 2. 创建 ContextImpl（真正的 Context 实现）
        ContextImpl appContext = ContextImpl.createAppContext(this, /*activityThread*/);
        
        // 3. 实例化 Application
        Application app = instrumentation.newApplication(cl, appClass, appContext);
        
        // 4. 设置 Application 到 ContextImpl
        appContext.setOuterContext(app);
        
        // 5. 缓存 Application 实例
        mApplication = app;
        
        return app;
    } catch (Exception e) {
        // 异常处理
    }
}
```

### 3. **Instrumentation 创建 Application**

java

java

下载

复制



```
// 源码路径：frameworks/base/core/java/android/app/Instrumentation.java
static public Application newApplication(Class<?> clazz, Context context) 
        throws InstantiationException, IllegalAccessException, ClassNotFoundException {
    // 反射创建 Application 实例
    Application app = (Application) clazz.newInstance();
    
    // 关键：调用 attach 方法
    app.attach(context);
    return app;
}
```

------

## 二、Application 的核心方法分析

### 1. **attach(Context context) 方法**

java

java

下载

复制



```
// 源码路径：frameworks/base/core/java/android/app/Application.java
final void attach(Context context) {
    // 1. 调用父类 ContextWrapper 的 attachBaseContext
    attachBaseContext(context);
    
    // 2. 设置进程信息
    mLoadedApk = ContextImpl.getImpl(context).mPackageInfo;
    
    // 3. 初始化组件回调分发器
    mComponentCallbacks = new ArrayMap<>();
    mActivityLifecycleCallbacks = new ArrayMap<>();
    mAssistCallbacks = new ArrayMap<>();
    
    // 4. 设置资源管理器
    mResourcesManager = ResourcesManager.getInstance();
}
```

**注意**：`attach()`是 `package`权限，应用无法重写。开发者只能在 `attachBaseContext()`中执行初始化。

### 2. **onCreate() 方法**

java

java

下载

复制



```
// Application.java
public void onCreate() {
    // 空实现，等待子类重写
}
```

**重点**：Application 的 `onCreate()`调用顺序在所有 ContentProvider 的 `onCreate()`之后。

### 3. **onTrimMemory() 和 onLowMemory()**

java

java

下载

复制



```
public void onTrimMemory(int level) {
    // 1. 转发给所有已注册的 ComponentCallbacks
    Object[] callbacks = collectComponentCallbacks();
    if (callbacks != null) {
        for (int i=0; i<callbacks.length; i++) {
            ((ComponentCallbacks)callbacks[i]).onTrimMemory(level);
        }
    }
    
    // 2. 调用 ActivityLifecycleCallbacks
    for (ActivityLifecycleCallbacks cb : mActivityLifecycleCallbacks.values()) {
        cb.onTrimMemory(level);
    }
}

public void onLowMemory() {
    // 类似实现
}
```

------

## 三、Application 的设计模式

### 1. **单例模式**

java

java

下载

复制



```
// Application 是进程内单例
public class MyApplication extends Application {
    private static MyApplication sInstance;
    
    public static MyApplication getInstance() {
        return sInstance;
    }
    
    @Override
    public void onCreate() {
        super.onCreate();
        sInstance = this;  // 保存单例
    }
}
```

**注意**：多进程应用中，每个进程都有独立的 Application 实例。

### 2. **门面模式（Facade）**

Application 是 Context 的门面，将复杂操作封装：

java

java

下载

复制



```
// 门面模式的体现
public abstract class Application extends ContextWrapper 
    implements ComponentCallbacks2 {
    
    // 封装资源访问
    public Resources getResources() {
        return super.getResources();
    }
    
    // 封装系统服务
    public Object getSystemService(String name) {
        return super.getSystemService(name);
    }
    
    // 封装组件管理
    public void registerComponentCallbacks(ComponentCallbacks callback) {
        synchronized (mComponentCallbacks) {
            mComponentCallbacks.put(callback, Boolean.TRUE);
        }
    }
}
```

### 3. **观察者模式**

Application 管理各种回调：

java

java

下载

复制



```
// 注册 Activity 生命周期监听
public void registerActivityLifecycleCallbacks(ActivityLifecycleCallbacks callback) {
    synchronized (mActivityLifecycleCallbacks) {
        mActivityLifecycleCallbacks.put(callback, Boolean.TRUE);
    }
}

// 分发事件
private void dispatchActivityCreated(Activity activity, Bundle savedInstanceState) {
    Object[] callbacks = collectActivityLifecycleCallbacks();
    if (callbacks != null) {
        for (int i=0; i<callbacks.length; i++) {
            ((ActivityLifecycleCallbacks)callbacks[i]).onActivityCreated(
                activity, savedInstanceState);
        }
    }
}
```

------

## 四、Application 的生命周期

### 1. **完整生命周期时序**



```
1. 进程创建
2. Application 类加载
3. Application.attachBaseContext()  ← 开发者可重写
4. ContentProvider.onCreate()        ← 所有 ContentProvider
5. Application.onCreate()           ← 开发者可重写
6. Activity.onCreate()              ← 首个 Activity
...
7. 内存不足时：onTrimMemory() / onLowMemory()
8. 进程终止（无回调，直接 kill）
```

### 2. **多进程的生命周期**



```
// 每个进程都有独立的 Application 实例
<application
    android:name=".MyApplication"
    ...>
    
    <!-- 主进程 -->
    <activity android:name=".MainActivity"/>
    
    <!-- 独立进程，会创建新的 Application 实例 -->
    <service
        android:name=".MyService"
        android:process=":background"/>
        
    <!-- 全局进程，也会创建新的 Application 实例 -->
    <provider
        android:name=".MyProvider"
        android:process="com.example.process"/>
</application>
```

**验证代码**：

```
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        Log.d("Process", "进程: ${Process.myPid()}, 应用: ${this.hashCode()}")
    }
}
// 输出示例：
// 主进程: 1234, 应用: 111111
// 后台进程: 1235, 应用: 222222  ← 不同的 Application 实例
```

------

## 五、源码中的关键设计点

### 1. **Context 继承体系**

```
Context (接口)
    ↑
ContextWrapper (包装类，持有 ContextImpl)
    ↑
ContextThemeWrapper (主题相关)
    ↑
Application (应用级别 Context)
    ↑
Activity (Activity 级别 Context)
    ↑
Service (Service 级别 Context)
```

**Application 与 Activity Context 的区别**：



```
// Application.java
public Context getApplicationContext() {
    return this;  // 返回自身
}

// Activity.java
public Context getApplicationContext() {
    return getApplication();  // 返回 Application 实例
}
```

### 2. **资源管理机制**



```
// Application 持有一个 Resources 对象
private Resources mResources;

@Override
public Resources getResources() {
    if (mResources != null) {
        return mResources;
    }
    
    // 懒加载创建 Resources
    mResources = super.getResources();
    return mResources;
}
```

**注意**：应用内所有组件的 Resources 实际上共享同一个实例。

### 3. **组件回调的线程安全性**





```
// 使用 synchronized 保护回调集合
private ArrayMap<ComponentCallbacks, Boolean> mComponentCallbacks;

public void registerComponentCallbacks(ComponentCallbacks callback) {
    synchronized (mComponentCallbacks) {
        mComponentCallbacks.put(callback, Boolean.TRUE);
    }
}

public void unregisterComponentCallbacks(ComponentCallbacks callback) {
    synchronized (mComponentCallbacks) {
        mComponentCallbacks.remove(callback);
    }
}
```

------

## 六、Application 的性能陷阱

### 1. **Application 初始化阻塞**





```
public class MyApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        
        // 以下操作会阻塞应用启动
        // 1. 初始化大型第三方库
        Firebase.initializeApp(this);  // 可能耗时
        
        // 2. 同步网络请求
        val response = api.initialize().execute();  // 网络 I/O
        
        // 3. 加载大文件
        val config = loadConfigFromAssets();  // 文件 I/O
        
        // 4. 复杂计算
        precomputeData();  // CPU 密集型
    }
}
```

### 2. **内存泄漏风险**



```
public class MyApplication extends Application {
    // 静态引用导致 Activity 泄漏
    private static Context sContext;
    
    // 监听器集合未清理
    private List<Listener> mListeners = new ArrayList<>();
    
    // 单例持有 View 或 Activity
    private WeakHashMap<Activity, Object> mActivityMap;  // 正确：使用弱引用
}
```

------

## 七、Application 的最佳实践

### 1. **优化启动速度**



```
class MyApplication : Application() {
    
    override fun onCreate() {
        super.onCreate()
        
        // 1. 延迟初始化
        val startupTasks = listOf(
            StartupTask { initAnalytics() },      // 立即执行
            StartupTask(delay = 1000) { initAds() },  // 延迟1秒
            StartupTask(thread = IO) { initDatabase() }  // IO线程
        )
        
        StartupManager(this).execute(startupTasks)
    }
    
    // 2. 使用 ContentProvider 进行初始化
    class InitProvider : ContentProvider() {
        override fun onCreate(): Boolean {
            // 在 Application.onCreate() 之前执行
            ThirdPartyLib.init(context)
            return true
        }
    }
}
```

### 2. **使用 App Startup 库**



```
// 1. 定义 Initializer
class AnalyticsInitializer : Initializer<Analytics> {
    override fun create(context: Context): Analytics {
        return Analytics.initialize(context)
    }
    
    override fun dependencies(): List<Class<out Initializer<*>>> {
        return listOf(WorkManagerInitializer::class.java)
    }
}

// 2. 在 AndroidManifest 中声明
<provider
    android:name="androidx.startup.InitializationProvider"
    android:authorities="${applicationId}.androidx-startup"
    android:exported="false"
    tools:node="merge">
    <meta-data
        android:name="com.example.AnalyticsInitializer"
        android:value="androidx.startup" />
</provider>
```

### 3. **避免的常见错误**



```
// ❌ 错误：在 Application 中存储 UI 相关对象
private var currentActivity: Activity? = null  // 可能泄漏

// ✅ 正确：使用弱引用或事件总线
private val currentActivity = WeakReference<Activity>(null)

// ❌ 错误：过度使用 Application 单例
object AppSingleton {
    val user: User? = null
    val config: Config? = null
    val network: Network? = null  // 职责过多
}

// ✅ 正确：使用依赖注入
class AppModule {
    @Singleton
    @Provides
    fun provideUserRepository(): UserRepository {
        return UserRepository()
    }
}
```

### 4. **多进程处理**



```
class MyApplication : Application() {
    
    override fun onCreate() {
        super.onCreate()
        
        // 根据进程名称执行不同的初始化
        when (getProcessName()) {
            packageName -> {
                // 主进程：初始化 UI 相关
                initMainProcess()
            }
            "$packageName:background" -> {
                // 后台进程：初始化后台服务
                initBackgroundProcess()
            }
            else -> {
                // 其他进程
                initOtherProcess()
            }
        }
    }
    
    private fun getProcessName(): String? {
        val pid = Process.myPid()
        val activityManager = getSystemService(Context.ACTIVITY_SERVICE) as ActivityManager
        return activityManager.runningAppProcesses
            ?.firstOrNull { it.pid == pid }
            ?.processName
    }
}
```

------

## 八、Application 的高级用法

### 1. **监控 Activity 生命周期**



```
class MyApplication : Application() {
    
    private var mForegroundActivityCount = 0
    private var mIsAppInForeground = false
    
    override fun onCreate() {
        super.onCreate()
        
        registerActivityLifecycleCallbacks(object : ActivityLifecycleCallbacks {
            override fun onActivityStarted(activity: Activity) {
                if (mForegroundActivityCount == 0) {
                    mIsAppInForeground = true
                    onAppEnterForeground()
                }
                mForegroundActivityCount++
            }
            
            override fun onActivityStopped(activity: Activity) {
                mForegroundActivityCount--
                if (mForegroundActivityCount == 0) {
                    mIsAppInForeground = false
                    onAppEnterBackground()
                }
            }
            
            // 其他方法...
        })
    }
    
    fun isAppInForeground(): Boolean = mIsAppInForeground
}
```

### 2. **全局异常处理**



```
class MyApplication : Application() {
    
    override fun onCreate() {
        super.onCreate()
        
        // 设置默认未捕获异常处理器
        Thread.setDefaultUncaughtExceptionHandler { thread, throwable ->
            // 记录崩溃日志
            logCrash(throwable)
            
            // 上传崩溃信息
            uploadCrashReport(throwable)
            
            // 恢复默认处理
            val defaultHandler = Thread.getDefaultUncaughtExceptionHandler()
            defaultHandler?.uncaughtException(thread, throwable)
        }
        
        // 主线程异常处理
        Looper.getMainLooper().setMessageLogging { message ->
            if (message.startsWith("Dispatching to")) {
                // 监控主线程卡顿
                monitorMainThreadBlock()
            }
        }
    }
}
```

### 3. **热修复支持**



```
class HotfixApplication : Application() {
    
    override fun attachBaseContext(base: Context) {
        super.attachBaseContext(base)
        
        // 1. 加载热修复补丁
        if (BuildConfig.DEBUG) {
            HotfixManager.loadPatch(this)
        }
        
        // 2. 应用多 Dex 优化
        if (Build.VERSION.SDK_INT < Build.VERSION_CODES.LOLLIPOP) {
            MultiDex.install(this)
        }
    }
    
    override fun onCreate() {
        super.onCreate()
        
        // 3. 初始化热修复
        HotfixManager.applyPatch()
    }
}
```







# StrictMode 

StrictMode 是 Android 开发中的一个工具类，用于检测在主线程中执行的不合规操作，例如磁盘读写、网络访问等。它帮助开发者识别和修复可能影响应用性能的问题，特别是那些可能导致应用无响应（ANR）的问题。

主要作用：

1. 

   检测主线程中的磁盘读写操作（违反模式时，可发出警告或崩溃）。

2. 

   检测主线程中的网络访问。

3. 

   检测Activity、Service、ContentProvider等组件的生命周期问题，如未关闭的Cursor、数据库连接等。

4. 

   检测慢函数调用（自定义耗时阈值）。

StrictMode 可以设置不同的策略（Policy），包括线程策略（ThreadPolicy）和虚拟机策略（VmPolicy）。

使用方式：

在Application的onCreate()或Activity的onCreate()中启用StrictMode。

示例代码（通常用于调试版本）：

```
if (BuildConfig.DEBUG) {
    StrictMode.setThreadPolicy(new StrictMode.ThreadPolicy.Builder()
        .detectDiskReads()   // 检测磁盘读
        .detectDiskWrites()  // 检测磁盘写
        .detectNetwork()     // 检测网络访问
        .penaltyLog()        // 违规时打印日志
        .build());

    StrictMode.setVmPolicy(new StrictMode.VmPolicy.Builder()
        .detectLeakedSqlLiteObjects()  // 检测未关闭的SQLite对象
        .detectLeakedClosableObjects() // 检测未关闭的Closable对象
        .penaltyLog()                   // 违规时打印日志
        .build());
}
```

注意：StrictMode 主要用于开发阶段，不应该在发布版本中启用，因为它可能会影响性能，并且可能会因为检测到违规而崩溃应用。

在Android 8.0（API 26）及以上版本，StrictMode 还增加了 detectUnbufferedIo() 等方法，以检测未缓冲的I/O操作。

另外，StrictMode 还允许自定义违规后的处理方式，比如弹出对话框（penaltyDialog）、崩溃（penaltyDeath）等。

通过使用StrictMode，开发者可以在开发阶段及时发现并修复潜在的性能问题，从而提升应用的用户体验。





## 一、StrictMode 的核心作用

### 1. **监控主线程的违规操作**

检测在 UI 线程上执行的不当操作：

| 检测类型       | 具体问题                   | 影响               |
| -------------- | -------------------------- | ------------------ |
| **磁盘读写**   | 主线程读写文件、数据库操作 | 界面卡顿、ANR      |
| **网络访问**   | 主线程发送 HTTP 请求       | 界面卡顿、ANR      |
| **慢函数调用** | 自定义耗时阈值检测         | 性能瓶颈识别       |
| **资源泄漏**   | 未关闭的 Cursor、文件流等  | 内存泄漏、资源耗尽 |

### 2. **检测内存泄漏**

监控 Activity、Service、BroadcastReceiver 等组件的内存泄漏：

- 

  Activity 被销毁后仍被持有引用

- 

  注册的监听器未正确反注册

- 

  静态变量持有 Context 引用

### 3. **虚拟机策略违规**

检测 JVM/ART 层面的问题：

- 

  SQLite 对象未关闭

- 

  可关闭对象（Closeable）未关闭

- 

  类实例计数异常

------

## 二、StrictMode 的使用方式

### 1. **基本配置**



```
// 通常在 Application.onCreate() 中配置
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        
        if (BuildConfig.DEBUG) {
            // 配置线程策略
            StrictMode.setThreadPolicy(
                StrictMode.ThreadPolicy.Builder()
                    .detectDiskReads()      // 检测磁盘读
                    .detectDiskWrites()     // 检测磁盘写
                    .detectNetwork()        // 检测网络访问
                    .detectCustomSlowCalls()// 检测自定义慢调用
                    .penaltyLog()           // 违规时打印日志
                    .penaltyFlashScreen()   // 违规时屏幕闪烁
                    .penaltyDialog()        // 违规时弹窗（不推荐生产环境）
                    .build()
            )
            
            // 配置虚拟机策略
            StrictMode.setVmPolicy(
                StrictMode.VmPolicy.Builder()
                    .detectLeakedSqlLiteObjects()    // 检测未关闭的 SQLite 对象
                    .detectLeakedClosableObjects()   // 检测未关闭的 Closeable
                    .detectActivityLeaks()          // 检测 Activity 泄漏
                    .detectLeakedRegistrationObjects() // 检测未反注册的监听器
                    .penaltyLog()                   // 违规时打印日志
                    .penaltyDeath()                 // 违规时崩溃（仅调试）
                    .build()
            )
        }
    }
}
```

### 2. **更严格的配置示例**



```
fun enableStrictMode() {
    // 线程策略
    val threadPolicy = StrictMode.ThreadPolicy.Builder()
        .detectAll()  // 检测所有线程违规
        .penaltyLog()
        .penaltyFlashScreen()
        .apply {
            // API 23+ 支持更多检测
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
                detectResourceMismatches()  // 检测资源不匹配
            }
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
                detectUnbufferedIo()  // 检测未缓冲的 I/O
            }
        }
        .build()
    
    // 虚拟机策略
    val vmPolicy = StrictMode.VmPolicy.Builder()
        .detectAll()  // 检测所有虚拟机违规
        .penaltyLog()
        .apply {
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.P) {
                detectNonSdkApiUsage()  // 检测非 SDK API 使用
            }
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q) {
                detectCredentialProtectedWhileLocked()  // 检测锁屏时访问凭证
            }
        }
        .build()
    
    StrictMode.setThreadPolicy(threadPolicy)
    StrictMode.setVmPolicy(vmPolicy)
}
```

------

## 三、StrictMode 的检测类型详解

### 1. **线程策略（ThreadPolicy）**

#### **A. 磁盘访问检测**



```
// 违规示例
fun readFileOnUiThread() {
    // 以下操作会在主线程触发 StrictMode 违规
    val file = File("/sdcard/test.txt")
    val content = file.readText()  // 磁盘读违规
    
    file.writeText("data")  // 磁盘写违规
}
```

**解决**：使用工作线程



```
fun readFileProperly() {
    lifecycleScope.launch(Dispatchers.IO) {
        val content = File("/sdcard/test.txt").readText()
        withContext(Dispatchers.Main) {
            updateUI(content)
        }
    }
}
```

#### **B. 网络访问检测**



```
// 违规示例
fun fetchDataOnUiThread() {
    val response = OkHttpClient().newCall(Request.Builder()
        .url("https://api.example.com/data")
        .build()
    ).execute()  // 同步网络请求，触发违规
}
```

**解决**：使用异步请求



```
fun fetchDataProperly() {
    OkHttpClient().newCall(request).enqueue(object : Callback {
        override fun onResponse(call: Call, response: Response) {
            // 在回调线程，需要切回主线程更新UI
        }
    })
}
```

#### **C. 慢调用检测**



```
// 自定义慢操作检测
StrictMode.ThreadPolicy.Builder()
    .detectCustomSlowCalls()
    .penaltyLog()
    .build()

// 标记慢操作
StrictMode.noteSlowCall("数据库查询耗时")
```

### 2. **虚拟机策略（VmPolicy）**

#### **A. Activity 泄漏检测**



```
// 泄漏示例
object LeakySingleton {
    var leakedActivity: Activity? = null  // 静态引用导致泄漏
}

class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        LeakySingleton.leakedActivity = this  // 泄漏！
    }
}
```

**StrictMode 日志**：





```
E/StrictMode: class com.example.MainActivity instance leaked
```

#### **B. 资源未关闭检测**



```
// 违规示例
fun leakCursor() {
    val cursor = database.query("SELECT * FROM users", null)
    // 使用后未关闭 cursor → StrictMode 会检测到
    // cursor.close()  // 必须调用！
}
```

#### **C. 监听器未反注册**



```
class MyActivity : AppCompatActivity() {
    private val sensorManager by lazy { 
        getSystemService(SENSOR_SERVICE) as SensorManager 
    }
    
    override fun onResume() {
        super.onResume()
        sensorManager.registerListener(
            sensorListener, 
            sensorManager.getDefaultSensor(Sensor.TYPE_ACCELEROMETER),
            SensorManager.SENSOR_DELAY_NORMAL
        )
    }
    
    // 缺少 onPause 中反注册 → StrictMode 会检测到
    override fun onPause() {
        super.onPause()
        sensorManager.unregisterListener(sensorListener)  // 必须调用！
    }
}
```

------

## 四、StrictMode 的输出日志解析

### 1. **线程策略违规日志**





```
D/StrictMode: StrictMode policy violation; ~duration=212 ms: android.os.StrictMode$StrictModeDiskReadViolation: policy=31 violation=2
    at com.example.MyActivity.readFile(MyActivity.kt:25)
    at com.example.MyActivity.onCreate(MyActivity.kt:15)
```

**关键信息**：

- 

  `policy=31`：策略掩码

- 

  `violation=2`：违规类型（2=磁盘读）

- 

  `~duration=212 ms`：违规操作耗时

### 2. **虚拟机策略违规日志**



```
E/StrictMode: class com.example.MyActivity instance leaked
    LEAK: com.example.MyActivity instance
    mContext instance of android.view.ContextThemeWrapper
    mBase instance of android.app.ContextImpl
    mMainThread instance of android.app.ActivityThread
```

**泄露路径**：显示 Activity 如何被持有引用。



### 3. **违规类型代码**

| 违规码 | 含义         | 对应检测方法              |
| ------ | ------------ | ------------------------- |
| 1      | 磁盘写       | `detectDiskWrites()`      |
| 2      | 磁盘读       | `detectDiskReads()`       |
| 4      | 网络         | `detectNetwork()`         |
| 8      | 自定义慢调用 | `detectCustomSlowCalls()` |

------

## 五、高级用法与技巧

### 1. **临时关闭 StrictMode**



```
// 保存原有策略
val oldThreadPolicy = StrictMode.getThreadPolicy()
val oldVmPolicy = StrictMode.getVmPolicy()

// 临时关闭检测
StrictMode.setThreadPolicy(StrictMode.ThreadPolicy.LAX)
StrictMode.setVmPolicy(StrictMode.VmPolicy.LAX)

try {
    // 执行需要豁免的操作
    performLegacyOperation()
} finally {
    // 恢复策略
    StrictMode.setThreadPolicy(oldThreadPolicy)
    StrictMode.setVmPolicy(oldVmPolicy)
}
```

### 2. **条件性检测**



```
// 仅在特定条件下启用严格检测
fun setupStrictModeBasedOnCondition() {
    val policyBuilder = StrictMode.ThreadPolicy.Builder()
    
    if (shouldDetectDiskOperations) {
        policyBuilder.detectDiskReads().detectDiskWrites()
    }
    
    if (shouldDetectNetwork) {
        policyBuilder.detectNetwork()
    }
    
    StrictMode.setThreadPolicy(policyBuilder.penaltyLog().build())
}
```

### 3. **自定义违规处理器**



```
// 实现自定义违规处理
class CustomStrictModeViolationListener : StrictMode.OnThreadViolationListener {
    override fun onThreadViolation(violation: Violation) {
        // 自定义处理逻辑
        logToAnalytics(violation)
        showToastToDeveloper(violation)
        // 可以选择不抛出默认处罚
    }
}

// 使用自定义监听器
StrictMode.setThreadPolicy(
    StrictMode.ThreadPolicy.Builder()
        .detectAll()
        .penaltyListener(executor, CustomStrictModeViolationListener())
        .build()
)
```

### 4. **在特定线程上启用 StrictMode**



```
// 在工作线程上启用 StrictMode
fun startWorkerThread() {
    thread {
        // 在线程开始时设置策略
        StrictMode.setThreadPolicy(
            StrictMode.ThreadPolicy.Builder()
                .detectDiskReads()
                .penaltyLog()
                .build()
        )
        
        // 执行工作
        doBackgroundWork()
    }
}
```