# Glide

# 使用问题

1. 默认情况下，固定链接图片加载后，加载的是缓存的图片，不是网络最新图片，需要设置缓存策略为不使用本地缓存

2. 图片拉伸问题

   ```
   解决方案：（在我手机上没发现这问题，解决方案不确定是否有效）
   1、取消使用place holder：
   Glide.with(context).load(resId). into(imageView);
   2、使用place holder加上dontAnimate()：
   Glide.with(context).load(resId).placeholder(defaultId).dontAnimate().into(imageView);
   3、使用asBitmap加载：
   Glide.with(context).load(imageUrl).asBitmap().placeholder(defaultId).into(imageView);
   ```

3. Glide加载框架生命周期问题：可能Activity中加载图片需要用applicationContext

4. 可以在application中预先初始化：Glide.get(this)


# Glide

### 1. 总体架构

Glide 的架构设计遵循了典型的图片加载库的流程，但它的模块化设计非常清晰，主要分为以下几个模块：

-

**API 层**：提供给用户使用的入口，主要是 `Glide`类和 `RequestManager`。

-

**引擎层 (Engine)**：负责加载和缓存资源，是核心调度模块。

-

**解码层 (Decoder)**：负责将原始数据（流、文件等）解码成图片资源。

-

**变换层 (Transformation)**：负责对图片进行变换，如裁剪、圆形等。

-

**目标层 (Target)**：负责将图片展示到目标上，如 `ImageView`。

-

**缓存层 (Cache)**：包括内存缓存、磁盘缓存等。

------

### 2. 加载流程（以 into(ImageView) 为例）

**步骤1：初始化请求**

-

通过 `Glide.with(context)`返回一个 `RequestManager`，用于管理生命周期和请求。

-

通过 `load()`方法创建一个 `RequestBuilder`，设置加载的模型（URL、文件等）。

**步骤2：创建请求**

-

调用 `into(ImageView)`时，会构建一个 `Request`（具体为 `SingleRequest`）。

-

首先会检查是否已经有相同的请求正在执行，避免重复请求。

**步骤3：引擎调度**

-

请求被交给 `Engine`处理。`Engine`会依次检查：

    1.

    **活动资源 (Active Resources)**：当前正在使用的资源，用弱引用保存，避免被回收。

    2.

    **内存缓存 (Memory Cache)**：LRU 缓存，存储最近使用过的资源。

    3.

    **磁盘缓存**：包括原始数据缓存和转换后的缓存。

    4.

    **源数据**：从网络或本地文件加载原始数据。

**步骤4：解码和变换**

-

从缓存或源获取数据后，会经过 `Decoder`解码成 `Bitmap`或 `Drawable`。

-

然后根据设置的变换（如 `centerCrop`）进行转换。

**步骤5：设置到目标**

-

将最终资源设置到 `ImageView`上，并更新目标的状态。

**步骤6：生命周期管理**

-

Glide 通过 `RequestManager`绑定生命周期（Activity/Fragment），在页面销毁时自动取消请求，避免内存泄漏。

------

### 3. 缓存机制

Glide 采用多级缓存，提高加载速度并减少流量消耗。

-

**活动资源 (Active Resources)**：使用弱引用存储当前正在使用的资源，避免同一图片被重复加载到内存。

-

**内存缓存 (Memory Cache)**：使用 `LruResourceCache`（基于 LRU 算法）缓存最近使用的资源，当资源从活动资源移除时，会存入内存缓存。

-

**磁盘缓存 (Disk Cache)**：分为

    -

    **资源（Resource）缓存**：存储转换后的图片。

    -

    **数据（Data）缓存**：存储原始数据。

默认情况下，Glide 会同时缓存原始数据和转换后的数据，但可以通过 `DiskCacheStrategy`进行配置。

------

### 4. 生命周期管理

Glide 通过 `RequestManager`与 Android 的生命周期绑定，实现请求的自动管理。

-

在 Activity 或 Fragment 销毁时，会自动暂停或取消请求。

-

通过向 Activity 添加一个不可见的 `Fragment`（`SupportRequestManagerFragment`）来监听生命周期。

------

### 5. 线程池

Glide 使用了多个线程池来优化性能：

-

**磁盘缓存线程池**：用于读取磁盘缓存，通常是单线程，避免并发读写的开销。

-

**源线程池**：用于从网络或本地文件加载数据，默认最多 4 个线程。

-

**动画线程池**：用于加载 GIF 等动画资源。

------

### 6. 注册组件 (Registry)

Glide 3.x 到 4.x 引入了 `Registry`，用于注册各种组件（如 `ModelLoader`、`ResourceDecoder`、`Encoder`等），使得 Glide 的扩展性大大增强。用户可以通过注册自定义组件来支持新的数据源或解码方式。

------

### 7. 关键类说明

-

`Glide`：入口类，全局单例，负责初始化配置。

-

`RequestManager`：请求管理器，绑定生命周期，管理请求的启动、暂停、重启、清除等。

-

`RequestBuilder`：构建请求，设置加载选项。

-

`Engine`：引擎，负责加载和缓存资源。

-

`EngineJob`和 `DecodeJob`：负责在后台执行加载和解码任务。

-

`LruResourceCache`：内存缓存实现。

-

`DiskLruCacheWrapper`：磁盘缓存实现。

------

### 8. 注意事项

-

**内存占用**：Glide 默认使用 `ARGB_8888`格式，每像素占用 4 字节。大量图片加载时需要注意内存优化，比如使用 `override()`限制尺寸，或者使用 `RGB_565`。

-

**缓存策略**：根据业务场景调整缓存策略，例如，头像加载可以使用 `DiskCacheStrategy.AUTOMATIC`（默认），而背景图等可能不需要磁盘缓存。

-

**列表加载优化**：在列表快速滑动时，Glide 会自动暂停请求，停止加载，以减少卡顿。

------

### 9. 与 Picasso 的对比

-

**缓存机制**：Glide 默认缓存转换后的图片，而 Picasso 只缓存原始图片。Glide 的缓存更节省 CPU 但占用更多磁盘空间。

-

**内存占用**：Glide 默认使用 `RGB_565`（在 4.0 以前）或 `ARGB_8888`（4.0 以后），而 Picasso 使用 `ARGB_8888`，但 Glide 会根据图片是否包含透明通道自动选择格式。

-

**功能**：Glide 支持 GIF 和视频缩略图，而 Picasso 不支持。



## 关键流程源码解析

### 1. **with() 生命周期绑定**





```
public static RequestManager with(Activity activity) {
    return getRetriever(activity).get(activity);
}
```

-

内部通过 `RequestManagerRetriever`获取或创建 `RequestManager`。

-

**生命周期绑定**：向 Activity/Fragment 添加一个**无 UI 的 SupportFragment**，监听生命周期，实现请求的自动暂停/恢复/销毁。

### 2. **load() 只是设置模型**

-

返回 `RequestBuilder`，此时并未开始加载，只是保存了数据源（URL、URI、资源ID等）。

### 3. **into() 启动加载（核心）**

`into(ImageView)`的调用链：





复制

```
RequestBuilder.into() 
→ RequestBuilder.buildRequest() 
→ RequestBuilder.into(Target) 
→ RequestManager.track() 
→ Engine.load()
```

#### **Engine.load() 关键步骤**：

1.

**生成 Key**：





   ```
   EngineKey key = keyFactory.buildKey(model, width, height, transformations, resourceClass, transcodeClass, options);
   ```

根据请求参数生成唯一缓存 Key（包括 URL、尺寸、变换等）。

2.

**四级缓存检查**（按顺序）：

    -

    **活动资源 (Active Resources)**：`Map<Key, EngineResource<?>>`，存储当前正在使用的资源，采用**弱引用**，避免同一图片被重复加载到内存。

    -

    **内存缓存 (Memory Cache)**：`LruResourceCache`，LRU 算法，存储最近使用但当前未显示的图片。

    -

    **资源重用 (Bitmap Pool)**：`BitmapPool`，复用已释放的 Bitmap 内存，避免频繁 GC。

    -

    **磁盘缓存 (Disk Cache)**：分为 `Resource`（转换后）和 `Data`（原始数据）两级。

3.

**加载策略**：

    -

    若缓存未命中，创建 `EngineJob`和 `DecodeJob`提交到线程池。

    -

    `DecodeJob`执行 `run()`方法，通过 `LoadPath`进行**数据获取 → 解码 → 转码 → 变换** 的处理链。



## 核心设计亮点

### 1. **三级缓存 + 活动资源**





```
// Engine.load() 缓存检查顺序
// 1. 活动资源
EngineResource<?> active = loadFromActiveResources(key);
// 2. 内存缓存
EngineResource<?> cached = loadFromCache(key);
// 3. 正在进行的请求（避免重复请求）
EngineJob<?> current = jobs.get(key);
// 4. 创建新任务
EngineJob<R> engineJob = engineJobFactory.build(...);
DecodeJob<R> decodeJob = decodeJobFactory.build(...);
```

**活动资源 (ActiveResources)** 的设计是其亮点之一，它存储当前正在使用的资源，防止被 LruCache 回收，同时避免同一图片重复加载。

### 2. **Bitmap 复用池 (BitmapPool)**



```
public interface BitmapPool {
    void put(Bitmap bitmap);
    Bitmap get(int width, int height, Bitmap.Config config);
}
```

-

存放被回收的 Bitmap 对象，新加载图片时尝试复用，**大幅减少内存分配和 GC 频率**。

-

实现类 `LruBitmapPool`使用 LRU 策略管理。

### 3. **灵活的注册机制 (Registry)**





```
registry.append(Photo.class, InputStream.class, new FlickrPhotoLoader.Factory());
```

-

通过 `Registry`注册组件（`ModelLoader`, `ResourceDecoder`, `Encoder`等），支持高度扩展。

-

例如，可自定义解码器支持新图片格式，或替换网络栈。

### 4. **生命周期自动管理**

-

通过 `RequestManagerFragment`监听宿主生命周期。

-

页面暂停时暂停网络请求，恢复时继续，销毁时释放资源。

### 5. **高效的线程模型**

-

**磁盘缓存读取**：固定线程池（1-2个线程），避免并发 I/O 竞争。

-

**网络请求**：自定义线程池，默认最多4个并发线程。

-

**图片解码**：根据 CPU 核心数动态调整线程数。

------

## 四、关键类说明

| 类                    | 职责                                                         |
| --------------------- | ------------------------------------------------------------ |
| `Glide`               | 全局入口，初始化配置                                         |
| `RequestManager`      | 管理请求生命周期，绑定到 Activity/Fragment                   |
| `RequestBuilder`      | 构建请求，设置参数（占位符、变换、缓存策略等）               |
| `Engine`              | 核心引擎，调度加载、缓存、复用                               |
| `EngineJob`           | 管理单个加载任务的执行、回调                                 |
| `DecodeJob`           | 执行解码任务，实现 `DataFetcher`, `Decoder`, `Transcoder`的调用链 |
| `LruResourceCache`    | 内存缓存（LRU 实现）                                         |
| `DiskLruCacheWrapper` | 磁盘缓存（基于 Jake Wharton 的 DiskLruCache）                |
| `BitmapPool`          | Bitmap 复用池                                                |

------

## 五、优化技巧（源码启示）

1.

**缓存策略**：

    -

    使用 `DiskCacheStrategy.ALL`缓存原始和转换后数据。

    -

    对频繁变化的图片（如头像）使用 `DiskCacheStrategy.NONE`。

2.

**内存优化**：

    -

    通过 `override(width, height)`精确控制加载尺寸。

    -

    使用 `Bitmap.Config.RGB_565`减少内存（无透明通道时）。

3.

**列表优化**：

    -

    在 RecyclerView 滑动时，Glide 自动暂停请求，停止后恢复。

    -

    可通过 `pauseRequests()`和 `resumeRequests()`手动控制。







## 一、`EngineKey`生成机制

### 1. **`EngineKey`的构成参数**



```
// EngineKey 构造方法
EngineKey(
    Object model,           // 数据源：URL、URI、File、ResourceId 等
    int width,             // 请求的目标宽度
    int height,            // 请求的目标高度
    Map<Class<?>, Transformation<?>> transformations, // 图片变换列表
    Class<?> resourceClass, // 资源类型：Bitmap、Drawable、GifDrawable 等
    Class<?> transcodeClass, // 转码目标类型：ImageView、Bitmap 等
    Options options         // Glide 选项：编码质量、解码格式等
)
```

### 2. **Key 生成的核心逻辑**

`KeyFactory.buildKey()`实际上就是调用 `EngineKey`的构造方法，将所有参数封装为一个 `EngineKey`对象。

**关键**：`EngineKey`重写了 `equals()`和 `hashCode()`方法，将所有成员变量都参与比较：





```
@Override
public boolean equals(Object o) {
    if (o instanceof EngineKey) {
        EngineKey other = (EngineKey) o;
        return 
            // 1. 比较数据源
            model.equals(other.model) &&
            // 2. 比较尺寸
            width == other.width &&
            height == other.height &&
            // 3. 比较变换
            transformations.equals(other.transformations) &&
            // 4. 比较资源类型
            resourceClass.equals(other.resourceClass) &&
            // 5. 比较目标类型
            transcodeClass.equals(other.transcodeClass) &&
            // 6. 比较Options
            options.equals(other.options);
    }
    return false;
}

@Override
public int hashCode() {
    if (hashCode == 0) {
        hashCode = model.hashCode();
        hashCode = 31 * hashCode + width;
        hashCode = 31 * hashCode + height;
        hashCode = 31 * hashCode + transformations.hashCode();
        hashCode = 31 * hashCode + resourceClass.hashCode();
        hashCode = 31 * hashCode + transcodeClass.hashCode();
        hashCode = 31 * hashCode + options.hashCode();
    }
    return hashCode;
}
```

### 3. **缓存命中的条件**

只有当**所有参数完全一致**时，才会命中缓存。例如：



```
// 这两个请求会生成不同的 Key，不会命中缓存
Glide.with(context)
    .load("https://example.com/image.jpg")
    .override(200, 200)
    .into(imageView1)

Glide.with(context)
    .load("https://example.com/image.jpg")
    .override(300, 300)  // 尺寸不同
    .into(imageView2)
```

------

## 二、相同文件不同 URL 的处理

### 1. **默认行为：视为不同资源**

如果两个 URL 不同但指向同一个文件，Glide 的**默认行为是分别缓存**，因为：





```
// URL 不同 → model 不同 → EngineKey 不同 → 缓存不命中
"https://example.com/image.jpg?v=1" ≠ "https://example.com/image.jpg?v=2"
```

这会导致：

-

**重复下载**：每个 URL 都会发起独立的网络请求

-

**重复缓存**：在内存和磁盘中存储多份相同的图片

-

**资源浪费**：流量、内存、存储空间都被浪费

### 2. **解决方案：自定义缓存 Key**

Glide 提供了多种方式解决这个问题：

#### **方案A：使用 `signature()`方法**



```
// 使用文件内容的哈希值作为签名
val file = File("/path/to/local/image.jpg")
val signature = ObjectKey(file.lastModified()) // 或使用文件 MD5

Glide.with(context)
    .load("https://example.com/image.jpg?v=1")
    .signature(signature)  // 添加签名
    .into(imageView)
```

**原理**：`signature`会参与 `EngineKey`的生成，如果签名相同，即使 URL 不同也可能命中缓存（还需其他参数相同）。

#### **方案B：自定义 `Key`实现**



```
class ContentAwareUrl(val url: String) {
    // 实现缓存 Key 的逻辑
    fun getCacheKey(): String {
        // 1. 提取文件标识（如从 URL 中提取文件名）
        val fileName = extractFileName(url)
        
        // 2. 或获取文件内容的哈希值
        val fileHash = getFileHash(url)
        
        return fileHash ?: fileName
    }
    
    override fun equals(other: Any?): Boolean {
        return other is ContentAwareUrl && 
               getCacheKey() == other.getCacheKey()
    }
    
    override fun hashCode(): Int {
        return getCacheKey().hashCode()
    }
}
```

然后通过自定义 `ModelLoader`使用这个 Key：



```
// 注册自定义 ModelLoader
@GlideModule
class MyAppGlideModule : AppGlideModule() {
    override fun registerComponents(
        context: Context,
        glide: Glide,
        registry: Registry
    ) {
        registry.append(
            ContentAwareUrl::class.java,
            InputStream::class.java,
            MyUrlLoader.Factory()
        )
    }
}
```

#### **方案C：重写 `GlideUrl`的 `getCacheKey()`**



```
val glideUrl = object : GlideUrl(url) {
    override fun getCacheKey(): String {
        // 去除查询参数，只保留路径部分
        val uri = URI(url)
        return uri.host + uri.path
    }
}
Glide.with(context).load(glideUrl).into(imageView)
```

**输出示例**：



```
原始URL: "https://img.example.com/photo.jpg?width=300&token=abc123"
缓存Key: "img.example.com/photo.jpg"  // 去除查询参数
```

#### **方案D：使用 `@Headers`注解（不推荐）**



```
@Headers("Cache-Control: public, max-age=86400")
@GET("image.jpg")
fun getImage(@Query("v") version: String): Call<ResponseBody>
```

这种方式**只影响 HTTP 缓存**，不影响 Glide 的缓存 Key。

------

## 三、最佳实践建议

| 场景                 | 推荐方案                    | 说明                               |
| -------------------- | --------------------------- | ---------------------------------- |
| **CDN 图片带版本号** | 方案C：重写 `getCacheKey()` | 去除版本参数，按文件路径缓存       |
| **相同文件不同域名** | 方案B：自定义 Key           | 提取文件哈希或唯一标识             |
| **本地文件修改检测** | 方案A：`signature()`        | 使用最后修改时间或 MD5             |
| **动态生成图片**     | 保留默认行为                | 不同参数生成不同图片，需要分别缓存 |

### 完整示例：处理带时间戳的 CDN URL



```
class CdnUrl(val originalUrl: String) {
    fun getStableKey(): String {
        return try {
            val uri = Uri.parse(originalUrl)
            // 移除查询参数，保留路径
            "${uri.scheme}://${uri.host}${uri.path}"
        } catch (e: Exception) {
            originalUrl
        }
    }
}

// 在 ModelLoader 中使用
class CdnUrlLoader : ModelLoader<CdnUrl, InputStream> {
    override fun buildLoadData(
        model: CdnUrl,
        width: Int,
        height: Int,
        options: Options
    ): ModelLoader.LoadData<InputStream>? {
        val glideUrl = GlideUrl(model.getStableKey())
        return ModelLoader.LoadData(
            glideUrl,
            HttpUrlFetcher(glideUrl, TIMEOUT, DefaultHttpUrlConnectionFactory())
        )
    }
    
    override fun handles(model: CdnUrl): Boolean = true
}
```

------

## 四、源码层面的优化思路

Glide 官方没有内置「相同文件不同 URL 去重」功能，因为：

1.

**性能考虑**：比较两个 URL 是否指向同一文件需要网络请求，代价太高

2.

**业务相关**：URL 去重逻辑高度依赖业务场景

3.

**缓存一致性**：可能引入复杂的缓存一致性问题

**建议**：

-

尽量在服务端使用**稳定的 URL**（不随意变更）

-

在客户端做**一层 URL 映射**，将动态 URL 转换为稳定标识

-

对频繁变化的图片（如用户头像）使用**短时间的磁盘缓存**策略





```
// 实用工具函数
fun String.toStableCacheKey(): String {
    return this
        .substringBefore("?")  // 去除查询参数
        .substringBefore("#")  // 去除锚点
        .replace(Regex("\\?.*"), "")
}
```