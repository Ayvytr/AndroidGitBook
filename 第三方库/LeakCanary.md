# LeakCanary



## 一、基本用法

### 1. 依赖配置



```
dependencies {
  // 最新版本（以2.x为例）
  debugImplementation 'com.squareup.leakcanary:leakcanary-android:2.12'
}
```

### 2. 自动安装（2.x版本）

**2.x版本**默认自动安装，无需手动初始化：

- 通过`ContentProvider`自动初始化
- 仅`debug`构建变体生效
- 应用启动时自动开始监视

**1.x版本**需手动初始化（已过时）：

```
class MyApplication : Application() {
  override fun onCreate() {
    super.onCreate()
    if (LeakCanary.isInAnalyzerProcess(this)) return
    LeakCanary.install(this)
  }
}
```

### 3. 检测范围

默认自动监视：

- 销毁的`Activity`
- 销毁的`Fragment`和`View`（通过`androidx.fragment`集成）
- 清除的`ViewModel`

### 4. 手动监视对象



```
class MyService : Service() {
  override fun onDestroy() {
    super.onDestroy()
    // 手动标记需监视的对象
    AppWatcher.objectWatcher.watch(
      watchedObject = this,
      description = "MyService received Service#onDestroy() callback"
    )
  }
}
```

### 5. 测试中检测内存泄漏



```
@After
fun tearDown() {
  val retainedObjects = AppWatcher.objectWatcher.retainedObjects
  if (retainedObjects.isNotEmpty()) {
    throw AssertionError("发现内存泄漏: $retainedObjects")
  }
}
```



## 工作原理：

1. RefWatcher.watch() 创建一个 KeyedWeakReference 到要被监控的对象。
2. 然后在后台线程检查引用是否被清除，如果没有，调用GC。
3. 如果引用还是未被清除，把 heap 内存 dump 到 APP 对应的文件系统中的一个 .hprof 文件中。
4. 在另外一个进程中的 HeapAnalyzerService 有一个 HeapAnalyzer 使用HAHA 解析这个文件。
5. 得益于唯一的 reference key, HeapAnalyzer 找到 KeyedWeakReference，定位内存泄漏。
6. HeapAnalyzer 计算 到 GC roots 的最短强引用路径，并确定是否是泄漏。如果是的话，建立导致泄漏的引用链。
7. 引用链传递到 APP 进程中的 DisplayLeakService， 并以通知的形式展示出来。







## 二、核心架构与源码分析

### 1. 整体架构



```
App Process (主进程)
├── ObjectWatcher - 监视对象是否被回收
├── InternalLeakCanary - 触发堆转储
└── HeapDumpTrigger - 控制转储时机

:leakcanary Process (分析进程)
├── HeapAnalyzer - 分析堆转储文件
├── Shark - 解析HPROF文件
└── LeakTrace - 生成泄漏链
```

### 2. 核心流程

**步骤1：对象监视（ObjectWatcher）**



```
// 核心原理：WeakReference + ReferenceQueue
class ObjectWatcher {
  private val watchedObjects = mutableMapOf<String, KeyedWeakReference>()
  private val queue = ReferenceQueue<Any>()
  
  fun watch(watchedObject: Any, description: String) {
    val key = UUID.randomUUID().toString()
    val reference = KeyedWeakReference(watchedObject, key, description, queue)
    watchedObjects[key] = reference
    
    // 5秒后检查是否被回收
    checkRetainedExecutor.execute {
      moveToRetained(key)
    }
  }
}
```

**步骤2：触发GC并检查**



```
fun moveToRetained(key: String) {
  // 1. 移除已被回收的弱引用
  removeWeaklyReachableObjects()
  
  // 2. 如果对象仍存在，说明可能泄漏
  val retained = watchedObjects[key] != null
  
  if (retained) {
    // 触发GC
    Runtime.getRuntime().gc()
    Thread.sleep(100)
    System.runFinalization()
    
    // 再次检查
    removeWeaklyReachableObjects()
    
    // 如果仍存在，确认为泄漏
    if (watchedObjects[key] != null) {
      onObjectRetained()
    }
  }
}
```

**步骤3：堆转储（Heap Dump）**



```
class HeapDumpTrigger {
  fun dumpHeap(retainedReferenceCount: Int) {
    // 1. 检查条件：是否达到阈值、是否有足够存储空间
    if (!checkCanPerformHeapDump()) return
    
    // 2. 暂停应用（避免转储过程中状态变化）
    Debug.startAllocCounting()
    
    // 3. 执行堆转储
    val heapDumpFile = heapDumper.dumpHeap()
    
    // 4. 恢复应用
    Debug.stopAllocCounting()
    
    // 5. 启动分析服务
    if (heapDumpFile != null) {
      HeapAnalyzerService.runAnalysis(context, heapDumpFile)
    }
  }
}
```

**步骤4：HPROF文件分析**



```
class HeapAnalyzer {
  fun analyze(file: File): HeapAnalysis {
    // 1. 解析HPROF文件
    val heapGraph = HprofHeapGraph.indexHprof(file)
    
    // 2. 查找泄漏引用
    val leakingRefs = findLeakingReferences(heapGraph)
    
    // 3. 构建泄漏链
    val leakTraces = buildLeakTraces(heapGraph, leakingRefs)
    
    return HeapAnalysis(
      heapDumpFile = file,
      createdAt = System.currentTimeMillis(),
      applicationLeaks = leakTraces
    )
  }
}
```

### 3. 关键类解析

**1. AppWatcher**



```
// 入口点，管理各种生命周期监听器
object AppWatcher {
  val objectWatcher = ObjectWatcher(
    clock = { SystemClock.uptimeMillis() },
    checkRetainedExecutor = { // 后台检查线程 }
  )
  
  // 自动安装的监听器
  internal val config: AppWatcherConfig = AppWatcherConfig(
    watchActivities = true,
    watchFragments = true,
    watchViewModels = true,
    // ...
  )
}
```

**2. KeyedWeakReference**



```
// 自定义WeakReference，携带唯一标识
class KeyedWeakReference(
  referent: Any,
  val key: String,
  val description: String,
  referenceQueue: ReferenceQueue<Any>
) : WeakReference<Any>(referent, referenceQueue) {
  val createdUptimeMillis = SystemClock.uptimeMillis()
  var clearedUptimeMillis: Long? = null
}
```

**3. InternalLeakCanary**



```
// 实际执行堆转储
internal object InternalLeakCanary {
  fun onHeapDump(heapDumpFile: File) {
    // 启动独立进程进行分析
    val intent = HeapAnalyzerService.createIntent(context, heapDumpFile)
    startService(intent)
  }
}
```

### 4. 泄漏链分析算法

LeakCanary使用**最短强引用路径**算法：

1. 

   **GC Roots查找**：从GC Root开始查找

2. 

   **支配树构建**：计算对象间的支配关系

3. 

   **最短路径计算**：找出泄漏对象到GC Root的最短路径

4. 

   **去重优化**：合并相同泄漏模式



```
// 泄漏链示例
┬───
│ GC Root: System class
│
├─ com.example.MyClass instance
│    Leaking: UNKNOWN
│    ↓ MyClass.leakedView
│               ~~~~~~~~~
├─ android.widget.TextView instance
│    Leaking: YES (View.mContext references destroyed activity)
```

### 5. 内存优化机制

**1. 阈值控制**



```
object HeapDumpTrigger {
  private const val RETainedObjectCountThreshold = 5
  private const val WAIT_FOR_OBJECT_THRESHOLD_MILLIS = 20_000L
}
```

**2. 避免频繁转储**

- 

  相同泄漏模式去重

- 

  最小间隔时间限制（默认1分钟）

- 

  磁盘空间检查

**3. 分析进程隔离**

- 

  独立进程（:leakcanary）进行分析

- 

  避免影响主进程性能

- 

  可独立配置堆大小

## 三、高级配置

### 1. 自定义配置



```
// 在Application中配置
class MyApp : Application() {
  override fun onCreate() {
    super.onCreate()
    
    LeakCanary.config = LeakCanary.config.copy(
      dumpHeap = BuildConfig.DEBUG,  // 仅Debug模式转储
      retainedVisibleThreshold = 3,  // 3个泄漏对象时触发
      referenceMatchers = AndroidReferenceMatchers.appDefaults
        .plus(
          ignoresStaticField(
            className = "com.example.StaticHolder",
            fieldName = "sLeaky"
          )
        )
    )
  }
}
```

### 2. 忽略特定泄漏



```
// 忽略已知的第三方库泄漏
LeakCanary.config = LeakCanary.config.copy(
  referenceMatchers = AndroidReferenceMatchers.appDefaults
    .plus(
      // 忽略Android系统已知泄漏
      ignoresInstanceField(
        className = "android.app.ActivityThread",
        fieldName = "mInitialApplication"
      )
    )
)
```

### 3. 自定义分析结果处理



```
class LeakUploader : OnHeapAnalyzedListener {
  override fun onHeapAnalyzed(heapAnalysis: HeapAnalysis) {
    // 上传到服务器
    uploadToServer(heapAnalysis)
    
    // 显示自定义通知
    showNotification(heapAnalysis)
  }
}

// 注册监听器
LeakCanary.config = LeakCanary.config.copy(
  onHeapAnalyzedListener = LeakUploader()
)
```

## 四、核心设计思想

### 1. 非侵入式监控

- 

  基于`ActivityLifecycleCallbacks`和`FragmentLifecycleCallbacks`

- 

  无源码侵入，仅依赖添加

### 2. 精准触发机制

- 

  延迟检测（默认5秒）

- 

  手动触发GC

- 

  阈值控制避免误报

### 3. 性能平衡

- 

  主进程仅监控，不分析

- 

  分析在独立进程进行

- 

  智能去重，避免重复分析

### 4. 可扩展架构

- 

  插件化设计

- 

  可自定义`ReferenceMatcher`

- 

  可替换`HeapDumper`和`HeapAnalyzer`

## 五、最佳实践

1. 

   **仅Debug版本启用**：避免生产环境性能影响

2. 

   **设置合理阈值**：避免频繁触发堆转储

3. 

   **及时修复泄漏**：关注"Application Leak"（应用泄漏）

4. 

   **忽略已知泄漏**：第三方库已知问题可配置忽略

5. 

   **结合CI/CD**：在自动化测试中集成LeakCanary检查

## 总结

LeakCanary通过**弱引用监控 + 延迟GC检查 + 独立进程分析**的架构，在保证检测准确性的同时最大限度减少对应用性能的影响。其核心价值在于：

- 

  **自动化**：自动监控、自动分析、自动报告

- 

  **准确性**：通过实际堆转储分析，避免误报

- 

  **可操作性**：提供完整的泄漏引用链，便于定位问题

- 

  **性能友好**：主进程轻量监控，复杂分析在独立进程进行