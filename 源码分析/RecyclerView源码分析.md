# RecyclerView源码分析



#### 1. 一级缓存：`mAttachedScrap`& `mChangedScrap`

- 

  **作用**：专门用于**屏幕内**的 Item 回收与复用。

- 

  **场景**：

  - 

    `mAttachedScrap`：在调用 `notifyItemMoved()`、`notifyItemRangeChanged()`等局部刷新方法，或轻度滑动时，**暂时离开屏幕但马上又可能回来的 Item** 会进入这里。它不经过 `onViewRecycled()`，状态完全保留，复用效率最高。

  - 

    `mChangedScrap`：专门存放那些被标记为“内容有更新”的 ViewHolder (调用了 `notifyItemChanged(position)`且没有设置 payload)。

- 

  **特点**：生命周期最短，与一次布局过程绑定。复用优先级最高。

#### 2. 二级缓存：`mCachedViews`(默认大小 = 2)

- 

  **作用**：存放**刚刚滚出屏幕**的 Item 的 ViewHolder。

- 

  **场景**：当 Item 完全离开屏幕时，会优先进入 `mCachedViews`。这是一个大小为 2（默认）的 ArrayList，按位置顺序存储。

- 

  **特点**：

  - 

    保存了完整的视图和数据，可以直接复用，无需重新绑定数据 (`onBindViewHolder`)。

  - 

    相当于一个“最近离开屏幕”的快速缓存。当反向滑动时，能立刻找到并显示。

  - 

    如果 `mCachedViews`已满，最老的 ViewHolder 会被移出，并降级到下一级缓存 `RecycledViewPool`。

#### 3. 三级缓存：`ViewCacheExtension`(开发者自定义缓存)

- 

  **作用**：提供给**开发者**的自定义缓存层。

- 

  **场景**：如果你有特殊需求，比如想自己管理特定类型的 ViewHolder 缓存，可以继承此类并实现相关方法。

- 

  **特点**：绝大多数情况下用不到。它是可选的，默认实现为 `null`。

#### 4. 四级缓存：`RecycledViewPool`(回收池，共享缓存)

- **作用**：`ViewHolder`的最终“蓄水池”和共享池。

- **场景**：当 `mCachedViews`满了，或者 Item 被移出 `mCachedViews`后，其 ViewHolder 在清除数据后会被放入这里。**同一个 RecyclerView 内部，以及通过 `setRecycledViewPool`共享的多个 RecyclerView 之间，都可以从这里获取复用的视图。**

- **特点**：

  - 

    按 `ViewHolder`的 `itemType`分类存储（通过 `getItemViewType`返回的类型）。

  - 

    每个类型默认最多缓存 5 个 ViewHolder。

  - 

    ViewHolder 进入这里时，会调用 `onViewRecycled()`清理数据。

  - 

    从 Pool 中取出的 ViewHolder **必须重新绑定数据**（会调用 `onBindViewHolder`）。



#### 和ListView对比



**ListView**：两级缓存，包括ActiveView和ScrapView，但复用仅限于同一位置，且缓存机制相对简单，容易造成频繁创建视图。



**RecyclerView**：四级缓存，且复用不依赖位置，而是通过ViewType匹配，更灵活，复用率更高。



