# View

```java
//无异常，但是onResume后子线程更新ui会异常
@Override
protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        tv = findViewById(R.id.tv);
        new Thread(new Runnable() {
            @Override
            public void run() {
                tv.setText("Test");
            }
        }).start();
}
```



 为什么最开始的在`onCreate()`里子线程对`UI`的操作没有报错呢？是因为`ViewRootImp`此时还没有创建，还未进行当前线程的判断； 现在，我们寻找`ViewRootImp`在何时创建，从`Activity`启动过程中寻找源码,通过分析可以查看ActivityThread.handleResumeActivity还未完成Activity的onResume





## 主线程更新UI的方法

```
Handler(mainLooper).post{
    tv.setText()
}

runOnUiThread {
    tv.setText()
}

view.post{
    tv.setText()
}


//AsyncTask也可,onPreExecute,onProgressUpdate,onCancelled,onPostExecute都可以，都有@MainThread标志
```



## View 绘制流程

### 三级绘制流程



```
// ViewRootImpl.java
public void performTraversals() {
    // 1. 测量
    performMeasure();
    // 2. 布局  
    performLayout();
    // 3. 绘制
    performDraw();
}

private void performMeasure() {
    mView.measure();
    // 回调 onMeasure()
}

private void performLayout() {
    mView.layout();
    // 回调 onLayout()  
}

private void performDraw() {
    mView.draw();
    // 回调 onDraw()
}
```



## View的MeasureSpec。MeasureSpec是一个32位的int值，其中高2位表示测量模式（Mode），低30位表示测量大小（Size）。它用于在测量过程中将父容器的约束传递给子View。

三种模式：

1. 

   **EXACTLY（精确模式）**：对应数值为0。表示父容器已经确定了子View的确切大小。在这种情况下，子View的测量大小即为MeasureSpec中指定的Size值。对应布局参数中的具体数值（如100dp）或MATCH_PARENT。

2. 

   **AT_MOST（最大模式）**：对应数值为-2147483648（即0x80000000，二进制最高两位为10）。表示子View不能超过MeasureSpec中指定的Size值，具体大小需要根据子View的内容来计算。对应布局参数中的WRAP_CONTENT。

3. 

   **UNSPECIFIED（未指定模式）**：对应数值为1073741824（即0x40000000，二进制最高两位为01）。表示父容器对子View没有任何限制，子View可以想要多大就多大。通常用于系统内部，例如ScrollView测量子View时。



在自定义View时，我们通常需要根据MeasureSpec来测量并设置View的宽高。一般的处理方式如下：

```
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    int width = measureDimension(desiredWidth, widthMeasureSpec);
    int height = measureDimension(desiredHeight, heightMeasureSpec);
    setMeasuredDimension(width, height);
}

private int measureDimension(int desiredSize, int measureSpec) {
    int result;
    int specMode = MeasureSpec.getMode(measureSpec);
    int specSize = MeasureSpec.getSize(measureSpec);

    if (specMode == MeasureSpec.EXACTLY) {
        // 父容器已经指定了确切大小，直接使用
        result = specSize;
    } else {
        // 先使用我们期望的大小
        result = desiredSize;
        if (specMode == MeasureSpec.AT_MOST) {
            // 父容器指定了一个最大尺寸，我们取期望大小和指定大小的最小值
            result = Math.min(result, specSize);
        }
        // 如果是UNSPECIFIED，则直接使用期望大小
    }
    return result;
}
```

注意：上述代码只是一个示例，实际处理时还需要考虑padding、margin等因素。



另外，在自定义ViewGroup时，我们需要测量每个子View，并根据子View的测量结果来设置自己的大小。这时需要调用每个子View的measure方法，传入根据当前ViewGroup的MeasureSpec和子View的LayoutParams计算得到的子View的MeasureSpec。

计算子View的MeasureSpec的规则通常如下（以宽度为例，高度同理）：

1. 

   如果子View的LayoutParams.width是具体数值，则子View的宽度MeasureSpec为EXACTLY，大小为该数值。

2. 

   如果子View的LayoutParams.width是MATCH_PARENT，则：

   - 

     如果父容器是EXACTLY模式，则子View为EXACTLY，大小为父容器剩余大小。

   - 

     如果父容器是AT_MOST模式，则子View为AT_MOST，大小不超过父容器剩余大小。

   - 

     如果父容器是UNSPECIFIED模式，则子View为UNSPECIFIED，大小不受限制（0）。

3. 

   如果子View的LayoutParams.width是WRAP_CONTENT，则：

   - 

     如果父容器是EXACTLY模式，则子View为AT_MOST，大小不超过父容器剩余大小。

   - 

     如果父容器是AT_MOST模式，则子View为AT_MOST，大小不超过父容器剩余大小。

   - 

     如果父容器是UNSPECIFIED模式，则子View为UNSPECIFIED，大小不受限制（0）。



## 三种模式详解

### 1. **EXACTLY（精确模式）**

- 

  **模式值**：`0`（对应常量 `MeasureSpec.EXACTLY`）

- 

  **含义**：父容器已确定了子 View 的确切尺寸，子 View 必须使用这个大小

- 

  **对应场景**：

  - 

    Layout 中指定了具体数值（`100dp`）

  - 

    Layout 参数为 `match_parent`且父容器有确定尺寸

- 

  **处理方式**：直接使用 MeasureSpec 中的 size 作为最终尺寸

### 2. **AT_MOST（最大模式）**

- 

  **模式值**：`-2147483648`（`0x80000000`，二进制 `10`开头）

- 

  **含义**：子 View 不能超过指定尺寸，但可以根据内容需要更小

- 

  **对应场景**：Layout 参数为 `wrap_content`

- 

  **处理方式**：计算内容所需尺寸，但不超过 MeasureSpec 中的 size

### 3. **UNSPECIFIED（未指定模式）**

- 

  **模式值**：`1073741824`（`0x40000000`，二进制 `01`开头）

- 

  **含义**：父容器对子 View 无任何限制，子 View 可以任意大小

- 

  **对应场景**：

  - 

    `ScrollView`测量子 View 高度时

  - 

    系统内部测量

  - 

    自定义测量场景

- 

  **处理方式**：使用 View 自身需要的大小



## 核心计算规则

### 1. **MeasureSpec 的生成（父容器决定子 View 的约束）**

这是最重要的规则，决定了父容器如何根据自身约束和子 View 的 LayoutParams 生成子 View 的 MeasureSpec：



```
// 伪代码逻辑（实际在 ViewGroup.getChildMeasureSpec 中实现）
public static int getChildMeasureSpec(int parentSpec, int padding, int childDimension) {
    int specMode = MeasureSpec.getMode(parentSpec);
    int specSize = MeasureSpec.getSize(parentSpec);
    
    int size = Math.max(0, specSize - padding);
    
    int resultSize = 0;
    int resultMode = 0;
    
    if (childDimension >= 0) {
        // 子 View 指定了具体数值（如 100dp）
        resultSize = childDimension;
        resultMode = MeasureSpec.EXACTLY;
    } else if (childDimension == LayoutParams.MATCH_PARENT) {
        // 子 View 是 match_parent
        if (specMode == MeasureSpec.EXACTLY || specMode == MeasureSpec.AT_MOST) {
            resultSize = size;        // 父容器给多大就用多大
            resultMode = specMode;    // 继承父容器的模式
        } else { // UNSPECIFIED
            resultSize = 0;           // 大小未定
            resultMode = MeasureSpec.UNSPECIFIED;
        }
    } else if (childDimension == LayoutParams.WRAP_CONTENT) {
        // 子 View 是 wrap_content
        if (specMode == MeasureSpec.EXACTLY || specMode == MeasureSpec.AT_MOST) {
            resultSize = size;        // 不超过父容器给的空间
            resultMode = MeasureSpec.AT_MOST;  // 但可以更小
        } else { // UNSPECIFIED
            resultSize = 0;           // 大小未定
            resultMode = MeasureSpec.UNSPECIFIED;
        }
    }
    
    return MeasureSpec.makeMeasureSpec(resultSize, resultMode);
}
```

### 2. **自定义 View 中的测量处理**

在自定义 View 的 `onMeasure()`中，需要根据 MeasureSpec 计算并设置自己的尺寸：



```
override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
    // 1. 获取模式和尺寸
    val widthMode = MeasureSpec.getMode(widthMeasureSpec)
    val widthSize = MeasureSpec.getSize(widthMeasureSpec)
    val heightMode = MeasureSpec.getMode(heightMeasureSpec)
    val heightSize = MeasureSpec.getSize(heightMeasureSpec)
    
    // 2. 根据模式计算最终尺寸
    val measuredWidth = when (widthMode) {
        MeasureSpec.EXACTLY -> {
            // 精确模式：必须使用指定尺寸
            widthSize
        }
        MeasureSpec.AT_MOST -> {
            // 最大模式：计算内容宽度，但不能超过限制
            val desiredWidth = calculateDesiredWidth()
            desiredWidth.coerceAtMost(widthSize)
        }
        else -> { // UNSPECIFIED
            // 未指定：使用内容所需尺寸
            calculateDesiredWidth()
        }
    }
    
    val measuredHeight = when (heightMode) {
        MeasureSpec.EXACTLY -> heightSize
        MeasureSpec.AT_MOST -> {
            val desiredHeight = calculateDesiredHeight()
            desiredHeight.coerceAtMost(heightSize)
        }
        else -> calculateDesiredHeight()
    }
    
    // 3. 必须调用此方法设置测量结果
    setMeasuredDimension(measuredWidth, measuredHeight)
}
```

### 3. **实际测量示例**



```
// 示例：一个固定宽高比的正方形 View
override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
    val widthMode = MeasureSpec.getMode(widthMeasureSpec)
    val widthSize = MeasureSpec.getSize(widthMeasureSpec)
    
    val heightMode = MeasureSpec.getMode(heightMeasureSpec)
    val heightSize = MeasureSpec.getSize(heightMeasureSpec)
    
    var finalWidth = 0
    var finalHeight = 0
    
    when (widthMode) {
        MeasureSpec.EXACTLY -> {
            finalWidth = widthSize
            // 高度 = 宽度（正方形）
            finalHeight = widthSize
        }
        MeasureSpec.AT_MOST -> {
            val desiredSize = 200 // 期望大小
            finalWidth = desiredSize.coerceAtMost(widthSize)
            finalHeight = finalWidth
        }
        MeasureSpec.UNSPECIFIED -> {
            finalWidth = 200
            finalHeight = 200
        }
    }
    
    // 检查高度约束
    if (heightMode == MeasureSpec.EXACTLY) {
        finalHeight = heightSize
        // 如果高度是确定的，调整宽度保持正方形
        finalWidth = finalHeight
    } else if (heightMode == MeasureSpec.AT_MOST) {
        finalHeight = finalHeight.coerceAtMost(heightSize)
        finalWidth = finalHeight
    }
    
    setMeasuredDimension(finalWidth, finalHeight)
}
```



## 核心区别概览

| 特性             | **`invalidate()`**               | **`requestLayout()`**                                        |
| ---------------- | -------------------------------- | ------------------------------------------------------------ |
| **触发目的**     | 请求**重绘**（重新绘制内容）     | 请求**重新测量和布局**                                       |
| **调用流程**     | `invalidate()`→ `onDraw()`       | `requestLayout()`→ `onMeasure()`→ `onLayout()`→ (可能) `onDraw()` |
| **影响范围**     | 当前 View 及其子 View 的绘制区域 | 当前 View 到 ViewRootImpl 的整条路径                         |
| **性能开销**     | 相对较小（只重绘）               | 较大（涉及测量、布局）                                       |
| **常见使用场景** | 内容变化但尺寸不变               | 尺寸或位置变化                                               |

## 详细解析

### 1. **`invalidate()`- 请求重绘**

**触发条件**：当 View 的**内容**发生变化，但**尺寸和位置不变**时调用。



```
// 示例：改变颜色后请求重绘
view.setBackgroundColor(Color.RED)
view.invalidate()  // 标记为脏区域，需要重绘
```

**调用流程**：







```
invalidate() 
    → 标记当前View为"脏"区域
    → 向上传递到ViewRootImpl
    → 触发VSync信号
    → 执行performTraversals()中的draw阶段
    → 调用onDraw()重新绘制
```

**特点**：

- 

  只触发**绘制流程**，不触发测量和布局

- 

  可指定脏区域（`invalidate(Rect)`）进行局部重绘

- 

  多次调用会被合并，在下一帧统一处理

- 

  适用于：动画、文本更新、颜色变化等

### 2. **`requestLayout()`- 请求重新布局**

**触发条件**：当 View 的**尺寸或位置**需要改变时调用。



```
// 示例：修改布局参数后请求重新布局
view.layoutParams.width = 200
view.requestLayout()  // 请求重新测量和布局
```

**调用流程**：







```
requestLayout()
    → 向上传递到ViewRootImpl
    → 设置强制布局标志
    → 触发VSync信号
    → 执行performTraversals()的整个流程：
        1. measure()   → onMeasure()   (测量)
        2. layout()    → onLayout()    (布局)
        3. draw()      → onDraw()      (可能重绘)
```

**特点**：

- 

  触发**完整的测量和布局流程**

- 

  会遍历整个 View 树，从当前 View 到 ViewRootImpl

- 

  如果尺寸确实改变了，通常会**自动调用** `invalidate()`

- 

  适用于：动态改变 View 大小、修改 LayoutParams 等

## 实际调用链分析



```
// 假设我们修改了一个View
view.layoutParams.width = 300
view.requestLayout()

// 后续流程大致如下：
// 1. requestLayout() 标记需要重新布局
// 2. 向上传递到ViewRootImpl
// 3. ViewRootImpl.performTraversals() 被调用
// 4. 执行 measure() → 调用 onMeasure()
// 5. 执行 layout() → 调用 onLayout()
// 6. 如果尺寸变化，系统自动调用 invalidate()
// 7. 执行 draw() → 调用 onDraw()

// 对比：直接调用 invalidate()
view.setBackgroundColor(Color.BLUE)
view.invalidate()
// 只触发：draw() → onDraw()
```

## 源码层面分析

### `invalidate()`的关键实现





```
// View.java
public void invalidate() {
    invalidate(true);
}

private void invalidate(boolean invalidateCache) {
    // 1. 标记脏区域
    final AttachInfo ai = mAttachInfo;
    if (ai != null) {
        // 2. 向上传递
        final ViewParent p = mParent;
        if (p != null) {
            // 3. 最终调用到ViewRootImpl
            p.invalidateChild(this, dirty);
        }
    }
}
```

### `requestLayout()`的关键实现



```
// View.java
public void requestLayout() {
    // 1. 清除测量缓存
    mPrivateFlags |= PFLAG_FORCE_LAYOUT;
    
    // 2. 向上传递
    if (mParent != null && !mParent.isLayoutRequested()) {
        mParent.requestLayout();
    }
}
```

## 组合使用场景

### 场景1：先改变尺寸，再改变内容



```
// 正确做法
view.layoutParams.width = 400
view.requestLayout()  // 先请求重新布局

// 在布局完成后改变内容
view.post {
    view.setBackgroundColor(Color.RED)
    view.invalidate()  // 再请求重绘
}
```

### 场景2：自定义View中



```
class MyView : View {
    private var mText: String = "Hello"
    
    fun setText(text: String) {
        mText = text
        
        if (text.length > 10) {
            // 文本变长，可能需要更多空间
            requestLayout()  // 请求重新测量
        } else {
            // 文本变短，但内容变化
            invalidate()  // 只重绘即可
        }
    }
    
    override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
        // 测量逻辑...
        val desiredHeight = measureTextHeight(mText)
        setMeasuredDimension(width, desiredHeight)
    }
    
    override fun onDraw(canvas: Canvas) {
        // 绘制文本
        canvas.drawText(mText, 0, 0, paint)
    }
}
```