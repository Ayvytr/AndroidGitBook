# LruCache

LruCache 是 Android 中基于 LRU（最近最少使用）算法实现的缓存类。



LruCache是个泛型类，主要算法原理是把最近使用的对象用强引用存储在LinkedHashMap中。当缓存满时，把最近最少使用的对象从内存中移除，并提供了get和put方法来完成缓存的获取和添加操作。



关键原理：LinkedHashMap（数组+双向链表结构）初始化参数accessOrder=true实现访问顺序排序，然后LruCache可方便地删除最近最少使用的对象

```
accessOrder=false: 插入顺序排序
accessOrder=true: 访问顺序排序
```



调用put()时，会在map中添加数据，并调用trimToSize()：判断缓存是否已满。如果满了就用LinkedHashMap的迭代器删除队尾元素，即近期最少访问的元素。当调用get()方法访问缓存对象时，就会调用LinkedHashMap的get)方法获得对应集合元素，同时会更新该元素到队头。



# LruCache 源码分析

LruCache 是 Android 中基于 LRU（最近最少使用）算法实现的缓存类。下面是 LruCache 的核心源码分析：

## 📁 核心数据结构

LruCache 内部使用 **LinkedHashMap** 作为存储容器，这是实现 LRU 算法的关键：





```
public class LruCache<K, V> {
    // 核心数据结构：LinkedHashMap
    private final LinkedHashMap<K, V> map;
    
    // 当前缓存大小（通常指条目数，但可自定义计算方式）
    private int size;
    private int maxSize;  // 缓存最大容量
    
    // 统计信息
    private int putCount;      // put操作次数
    private int createCount;   // create调用次数
    private int evictionCount; // 淘汰条目次数
    private int hitCount;      // 缓存命中次数
    private int missCount;     // 缓存未命中次数
}
```

## 🔧 关键源码分析

### 1. 构造方法



```
public LruCache(int maxSize) {
    if (maxSize <= 0) {
        throw new IllegalArgumentException("maxSize <= 0");
    }
    this.maxSize = maxSize;
    // 关键：创建LinkedHashMap，第三个参数为true表示按访问顺序排序
    this.map = new LinkedHashMap<K, V>(0, 0.75f, true);
}
```

**关键点**：`LinkedHashMap`的第三个构造参数 `accessOrder`设置为 `true`，表示按**访问顺序**排序（最近访问的放在队尾），这是 LRU 算法的核心。

### 2. put() 方法 - 添加缓存





```
public final V put(K key, V value) {
    if (key == null || value == null) {
        throw new NullPointerException("key == null || value == null");
    }
    
    V previous;
    synchronized (this) {
        putCount++;
        // 计算新条目的"大小"（可通过sizeOf()自定义计算逻辑）
        size += safeSizeOf(key, value);
        // 放入map，返回之前的值（如果存在）
        previous = map.put(key, value);
        if (previous != null) {
            // 如果之前有值，需要减去旧值的大小
            size -= safeSizeOf(key, previous);
        }
    }
    
    if (previous != null) {
        // 通知旧值被替换
        entryRemoved(false, key, previous, value);
    }
    
    // 调整缓存大小，确保不超过maxSize
    trimToSize(maxSize);
    return previous;
}
```

### 3. get() 方法 - 获取缓存





```
public final V get(K key) {
    if (key == null) {
        throw new NullPointerException("key == null");
    }
    
    V mapValue;
    synchronized (this) {
        // 从LinkedHashMap获取值
        mapValue = map.get(key);
        if (mapValue != null) {
            hitCount++;  // 缓存命中
            return mapValue;  // 返回找到的值
        }
        missCount++;  // 缓存未命中
    }
    
    // 缓存未命中时，尝试创建值
    V createdValue = create(key);
    if (createdValue == null) {
        return null;
    }
    
    // 创建成功，尝试放入缓存
    synchronized (this) {
        createCount++;
        // 再次尝试获取，防止其他线程已创建
        mapValue = map.put(key, createdValue);
        
        if (mapValue != null) {
            // 存在竞争，恢复原来的值
            map.put(key, mapValue);
        } else {
            // 成功放入，计算大小
            size += safeSizeOf(key, createdValue);
        }
    }
    
    if (mapValue != null) {
        entryRemoved(false, key, createdValue, mapValue);
        return mapValue;
    } else {
        // 调整缓存大小
        trimToSize(maxSize);
        return createdValue;
    }
}
```

**LRU关键机制**：调用 `map.get(key)`时，由于创建 `LinkedHashMap`时设置了 `accessOrder=true`，被访问的条目会自动移到**链表尾部**（最近使用），而链表头部就是最近最少使用的条目。

### 4. trimToSize() 方法 - 核心淘汰逻辑



```
public void trimToSize(int maxSize) {
    while (true) {
        K key;
        V value;
        synchronized (this) {
            if (size < 0 || (map.isEmpty() && size != 0)) {
                throw new IllegalStateException(getClass().getName()
                        + ".sizeOf() is reporting inconsistent results!");
            }
            
            // 如果当前大小未超过限制，退出循环
            if (size <= maxSize || map.isEmpty()) {
                break;
            }
            
            // 关键：获取链表头部（最近最少使用）的条目
            Map.Entry<K, V> toEvict = map.entrySet().iterator().next();
            key = toEvict.getKey();
            value = toEvict.getValue();
            
            // 移除该条目
            map.remove(key);
            size -= safeSizeOf(key, value);
            evictionCount++;  // 淘汰计数增加
        }
        
        // 通知条目被移除
        entryRemoved(true, key, value, null);
    }
}
```

**淘汰策略**：当缓存大小超过 `maxSize`时，循环移除链表头部的条目（最近最少使用），直到满足大小限制。

### 5. remove() 方法 - 移除缓存





```
public final V remove(K key) {
    if (key == null) {
        throw new NullPointerException("key == null");
    }
    
    V previous;
    synchronized (this) {
        previous = map.remove(key);
        if (previous != null) {
            size -= safeSizeOf(key, previous);
        }
    }
    
    if (previous != null) {
        entryRemoved(false, key, previous, null);
    }
    
    return previous;
}
```

## 📊 自定义大小计算

LruCache 默认按条目数计算大小，但可以通过重写 `sizeOf()`方法自定义计算逻辑：





```
// 默认实现，每个条目大小为1
protected int sizeOf(K key, V value) {
    return 1;
}
```

**自定义示例**：



```
// 创建缓存，限制为4MB
int maxMemory = (int) (Runtime.getRuntime().maxMemory() / 1024);
int cacheSize = maxMemory / 8; // 使用最大内存的1/8

LruCache<String, Bitmap> memoryCache = new LruCache<String, Bitmap>(cacheSize) {
    @Override
    protected int sizeOf(String key, Bitmap bitmap) {
        // 按KB计算Bitmap大小
        return bitmap.getByteCount() / 1024;
    }
};
```

## 🔄 生命周期回调

LruCache 提供了可重写的回调方法，用于处理特定事件：



```
// 条目被移除时调用（包括淘汰、替换、移除）
protected void entryRemoved(boolean evicted, K key, V oldValue, V newValue) {
    // evicted: true表示因容量不足被淘汰，false表示被主动移除或替换
}

// 缓存未命中时创建新值（默认返回null，需重写）
protected V create(K key) {
    return null;
}
```

## 🎯 线程安全性

LruCache 的所有公共方法都通过 **synchronized** 关键字实现线程安全
