# RecyclerView

# DiffUtil

**最大的用处就是在RecyclerView刷新时**，不再无脑`mAdapter.notifyDataSetChanged()`

它会自动计算新老数据集的差异，并根据差异情况，自动调用以下四个方法

	adapter.notifyItemRangeInserted(position, count);
	adapter.notifyItemRangeRemoved(position, count);
	adapter.notifyItemMoved(fromPosition, toPosition);
	adapter.notifyItemRangeChanged(position, count, payload);


### 1. 基本用法

实现**DiffUtil.Callback**，然后**调用DiffUtil.calculateDiff()方法**，计算出新老数据集转化的最小更新集，就是DiffUtil.DiffResult对象。 然后调用`DiffUtil.DiffResult.dispatchUpdatesTo（RecyclerView.Adapter adapter）`方法，传入RecyclerView的Adapter，最后，更新数据集即可。

```java
//示例步骤：
public class StudentDiffCallback extends DiffUtil.Callback {
    private List<Student> oldList;
    private List<Student> list;

    public StudentDiffCallback(List<Student> oldList, List<Student> list) {
        this.oldList = oldList;
        this.list = list;
    }

    @Override
    public int getOldListSize() {
        return oldList.size();
    }

    @Override
    public int getNewListSize() {
        return list.size();
    }

    @Override
    public boolean areItemsTheSame(int i, int i1) {
        return oldList.get(i).id == list.get(i1).id;
    }

    @Override
    public boolean areContentsTheSame(int i, int i1) {
        Student oldStudent = oldList.get(i);
        Student student = list.get(i1);
        return oldStudent.id == student.id &&
                oldStudent.age == student.age &&
                oldStudent.name.equals(student.name);
    }
}

		//使用步骤
        DiffUtil.DiffResult diffResult = DiffUtil.calculateDiff(new StudentDiffCallback(adapter.getDatas(), list));
        diffResult.dispatchUpdatesTo(adapter);
        adapter.updateList(list);
```



### 2.高级用法

高级用法只涉及到两个方法， 我们需要分别实现DiffUtil.Callback.getChangePayload(int oldItemPosition, int newItemPosition)， 返回的Object就是表示Item改变了哪些内容。

再配合RecyclerView.Adapter.onBindViewHolder(VH holder, int position, List<Object> payloads)方法， 完成定向刷新。

payloads 参数 是一个从（notifyItemChanged(int, Object)或notifyItemRangeChanged(int, int, Object)）里得到的合并list。 
如果payloads list 不为空，那么当前绑定了旧数据的ViewHolder 和Adapter， 可以使用 payload的数据进行一次 高效的部分更新。 

如果payload 是空的，Adapter必须进行一次完整绑定（调用两参方法）。 

```java
//示例代码
    @Nullable
    @Override
    public Object getChangePayload(int oldItemPosition, int newItemPosition) {
        // 定向刷新中的部分更新,效率最高
        //只是没有了ItemChange的白光一闪动画，（反正我也觉得不太重要）
        TestBean oldBean = mOldDatas.get(oldItemPosition);
        TestBean newBean = mNewDatas.get(newItemPosition);

        //这里就不用比较核心字段了,一定相等
        Bundle payload = new Bundle();
        if (!oldBean.getDesc().equals(newBean.getDesc())) {
            payload.putString("KEY_DESC", newBean.getDesc());
        }
        if (oldBean.getPic() != newBean.getPic()) {
            payload.putInt("KEY_PIC", newBean.getPic());
        }

        if (payload.size() == 0)//如果没有变化 就传空
            return null;
        return payload;//
    }

	//在Adapter里如下重写三参的onBindViewHolder：
    @Override
   	public void onBindViewHolder(DiffVH holder, int position, List<Object> payloads) 	{
        if (payloads.isEmpty()) {
            onBindViewHolder(holder, position);
        } else {
            //文艺青年中的文青
            Bundle payload = (Bundle) payloads.get(0);
            TestBean bean = mDatas.get(position);
            for (String key : payload.keySet()) {
                switch (key) {
                    case "KEY_DESC":
                        //这里可以用payload里的数据，不过data也是新的 也可以用
                        holder.tv2.setText(bean.getDesc());
                        break;
                    case "KEY_PIC":
                        holder.iv.setImageResource(payload.getInt(key));
                        break;
                    default:
                        break;
                }
            }
        }
    }

//如果嫌麻烦可以这样写，当然这需要更新所有内容：
    @Nullable
    @Override
    public Object getChangePayload(int oldItemPosition, int newItemPosition) {
        return list.get(newItemPosition);
    }


    @Override
    public void onBindViewHolder(@NonNull ViewHolder holder, int position, @NonNull List<Object> payloads) {
        if (payloads.isEmpty()) {
            onBindViewHolder(holder, position);
        } else {
            convert(holder, getItemAt(position));
        }
    }
```

### 注意1： 没有重写DiffUtil.Callback.getChangePayload(int oldItemPosition, int newItemPosition) 时，更新列表时，有数据更新的item会闪白光，如果不想要白光的话，重写这个方法即可。

### 注意2：如果数据量多比较耗时，计算新老数据集差异的逻辑可以写在子线程，然后再主线程更新，减少主线程开销。



* [RecyclerView的好伴侣：详解DiffUtil](https://blog.csdn.net/zxt0601/article/details/52562770)





#  RecyclerView源码分析

## 四级缓存



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



