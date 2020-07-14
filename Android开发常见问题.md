# [Android开发问题和经验总结（不断更新中）](https://github.com/Ayvytr/AndroidGitBook/Android开发常见问题.md)



安卓常见问题，Bug等以及解答，欢迎大家更新和指正！！！

# Android 系统

## 防止OOM：

8.0之后对内存使用方式做了修改，这两个操作就可以避免全部的oom：
minSdk  26    ndk {abiFilters 'arm64-v8a'}

## Android 9 Http请求：Cleartext HTTP traffic to ... not permitted

### 方案1：改用https

### 方案2：targetSdkVersion 降到27以下

### 方案3：更改网络安全配置（1）

1.在res文件夹下创建一个xml文件夹，然后创建一个network_security_config.xml文件，文件内容如下：

```
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config cleartextTrafficPermitted="true" />
</network-security-config>
```



2.接着，在AndroidManifest.xml文件下的application标签增加以下属性：

```
<application
	android:networkSecurityConfig="@xml/network_security_config"
/>
```

### 方案4：更改网络安全配置（2）：直接在AndroidManifest.xml配置文件的<application>标签中直接插入android:usesCleartextTraffic="true"



# View 问题

## TextView

singleLine过时了，使用lines代替。maxLines=“1”不能限制文本只显示一行

### string资源字符串自定义部分文本颜色

```
//string.xml中配置：
<Data><![CDATA[染色<font color="#ff0000">%s</font>染色]]></Data>

//TextView设置：
TextView.setText(Html.fromHtml(getString(id, "format")))

```



## EditText

Java代码中调用了setFilters，会覆盖布局中maxLength属性，除非同时设置了LengthFilter

## CheckBox

### button属性失效问题

button对应的Selector drawable，把选中和未选中状态单独放到一个drawable中，不然和selected等混在一个文件会失效

## RecyclerView

### java.lang.IndexOutOfBoundsException: Inconsistency detected. Invalid item position...

RecyclerView绑定的 List 对象在更新数据之前进行了 clear，而这时紧接着迅速上滑 RecyclerView，就会造成崩溃，属于RecyclerView内部异常。



### RecyclerView在多层嵌套的内容中，即使设置了wrap_content或者match_parent,但是数据项还是只显示1行，Item高度也不是wrap_content

使用可以让RecyclerView显示全的控件进行包裹，推荐使用NestedScrollView。（本人在项目中遇到这个问题，Recyclerview外层是多层LinearLayout, CardView等布局，还要在外层加上下拉刷新的功能。网上多种解决方案都试过，也没有采用inflate(layoutId, null, false)加载Item的方法，因为封装过Adapter再改代价很大。最后的解决方案是：在RecyclerView外层布局包裹NestedScrollView，禁用RecyclerView滑动，使用NestedScrollView的滑动即可[本人使用的下拉刷新是SmartRefreshLayout，内部使用NestedScrollView可以正常滑动，问题解决]）



### ScrollView嵌套RecyclerView的显示不全，滑动卡顿，夺取了焦点导致页面启动时RecyclerView上方布局没显示问题

有问题的代码：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical">

            <!--页面启动时，TextView未显示，焦点被RecyclerView获取-->
            <TextView
                android:layout_width="match_parent"
                android:layout_height="@dimen/_200dp"
                android:gravity="center"
                android:text="Header"
                android:textColor="@color/aero"
                android:textSize="@dimen/_50sp"/>

            <!--没有android:nestedScrollingEnabled="false"时，滑动冲突，-->
            <!--有这个属性时，ScrollView无法滑动，整个ScrollView只能滑动一点-->
            <androidx.recyclerview.widget.RecyclerView
                android:id="@+id/rv"
                android:nestedScrollingEnabled="false"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                app:layoutManager="androidx.recyclerview.widget.LinearLayoutManager"/>

        </LinearLayout>
    </ScrollView>
</LinearLayout>

```



滑动卡顿问题：

```java
recyclerView.setHasFixedSize(true);
recyclerView.setNestedScrollingEnabled(false);
或者
android:nestedScrollingEnabled="false"
```

RecyclerView夺取焦点问题：外层布局需要增加属性（但是某些时候这个属性不起作用）

```
android:focusable="true"
android:focusableInTouchMode="true"
```

RecyclerView夺取焦点问题解决方案2：ScrollView的直接子View使用

```
//截获RecyclerView焦点
android:descendantFocusability="blocksDescendants" 
```

```
//android:descendantFocusability属性：
beforeDescendants：viewgroup会优先其子类控件而获取到焦点
afterDescendants：viewgroup只有当其子类控件不需要获取焦点时才获取焦点
blocksDescendants：viewgroup会覆盖子类控件而直接获得焦点

```



显示不全的问题：使用NestedScrollView代替ScrollView

//没有问题的代码：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <androidx.core.widget.NestedScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <LinearLayout
            android:focusable="true"
            android:focusableInTouchMode="true"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical">

            <!--页面启动时，TextView未显示，焦点被RecyclerView获取-->
            <TextView
                android:layout_width="match_parent"
                android:layout_height="@dimen/_200dp"
                android:gravity="center"
                android:text="Header"
                android:textColor="@color/aero"
                android:textSize="@dimen/_50sp"/>

            <!--没有android:nestedScrollingEnabled="false"时，滑动冲突，-->
            <!--有这个属性时，ScrollView无法滑动，整个ScrollView只能滑动一点-->
            <androidx.recyclerview.widget.RecyclerView
                android:id="@+id/rv"
                android:nestedScrollingEnabled="false"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                app:layoutManager="androidx.recyclerview.widget.LinearLayoutManager"/>

        </LinearLayout>
    </androidx.core.widget.NestedScrollView>
</LinearLayout>

```

### LinearLayout嵌套RecyclerView的显示不全，滑动卡顿，夺取了焦点导致页面启动时RecyclerView上方布局没显示问题（真实情况应是ScrollView嵌套LinearLayout嵌套RecyclerView会出现）

附上之前出问题的代码（问题已经改了）：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:orientation="vertical">

    <include layout="@layout/public_include_title_and_stausview" />

    <ScrollView
        android:layout_width="match_parent"
        android:layout_weight="1"
        android:overScrollMode="never"
        android:layout_height="0dp">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:paddingTop="15dp"
            android:orientation="vertical">

            <LinearLayout
                android:id="@+id/ll_quote_company"
                android:layout_width="match_parent"
                android:orientation="vertical"
                android:layout_height="wrap_content">

                <TextView
                    android:paddingLeft="15dp"
                    android:paddingRight="15dp"
                    android:text="@string/init_quote_top_notice"
                    android:textColor="@color/public_color_FA960C"
                    android:textSize="@dimen/public_font_16sp"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content" />

                <android.support.v7.widget.CardView
                    android:layout_margin="@dimen/public_height_12dp"
                    android:layout_width="match_parent"
                    app:cardCornerRadius="10dp"
                    app:cardBackgroundColor="@color/public_bg_3"
                    android:layout_height="wrap_content">

                    <LinearLayout
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:paddingLeft="15dp"
                        android:paddingRight="15dp"
                        android:paddingTop="18dp"
                        android:orientation="vertical">

                        <TextView
                            android:layout_width="wrap_content"
                            android:text="@string/quote_company"
                            android:textSize="@dimen/public_font_18sp"
                            android:textColor="@color/public_text_main"
                            android:layout_marginBottom="15dp"
                            android:layout_height="wrap_content" />

                        <include
                            layout="@layout/layout_wabill_detail_line"
                            android:layout_width="match_parent"
                            android:layout_height="wrap_content" />

                        <RelativeLayout
                            android:layout_width="match_parent"
                            android:layout_height="wrap_content">

                        <android.support.v7.widget.RecyclerView
                            android:id="@+id/rv"
                            android:layout_width="match_parent"
                            tools:listitem="@layout/item_select_quote"
                            tools:itemCount="1"
                            android:nestedScrollingEnabled="false"
                            app:layoutManager="android.support.v7.widget.GridLayoutManager"
                            app:spanCount="1"
                            android:layout_height="wrap_content" />
                        </RelativeLayout>
                    </LinearLayout>

                </android.support.v7.widget.CardView>

            </LinearLayout>

            <include layout="@layout/layout_quote_order_info" />
        </LinearLayout>
    </ScrollView>

    <RelativeLayout
        android:id="@+id/rl_bottom"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@color/public_bg_3"
        android:padding="@dimen/public_pad_top_10dp">

        <include layout="@layout/layout_btn_left_right" />
    </RelativeLayout>

</LinearLayout>
```

解决方案是在RecyclerView外包裹RelativeLayout。[原因：ScrollView嵌套RecyclerView解决以及原理详解 文中提到LinearLayout和RelativeLayout测量的不同](https://blog.csdn.net/a568478312/article/details/79881540)





## DiffUtil.Callback  RecyclerView动态更新/增减数据需要实现的类

部分(partial)绑定vs完整(full)绑定 
payloads 参数 是一个从（notifyItemChanged(int, Object)或notifyItemRangeChanged(int, int, Object)）里得到的合并list。 
如果payloads list 不为空，那么当前绑定了旧数据的ViewHolder 和Adapter， 可以使用 payload的数据进行一次 高效的部分更新。 
如果payload 是空的，Adapter必须进行一次完整绑定（调用两参方法）。 

需要实现的方法：DiffUtil.Callback：public Object getChangePayload(int oldItemPosition, int newItemPosition)， RecyclerView.Adapter：public void onBindViewHolder(VH holder, int position, List<Object> payloads)

## ScrollView

### 滚动到顶部或者底部  ：

scrollView.fullScroll(ScrollView.FOCUS_DOWN);滚动到底部

scrollView.fullScroll(ScrollView.FOCUS_UP);滚动到顶部

### 滚动到顶部的三种方式：

```
ScrollView.scrollTo(``0``,``0``);``//直接置顶，瞬间回到顶部，没有滚动过程，其中Y值可以设置为大于0的值，使Scrollview停在指定位置。
ScrollView.fullScroll(View.FOCUS_UP);``//类似于手动拖回顶部,有滚动过程
ScrollView.smoothScrollTo(``0``, ``0``);``//类似于手动拖回顶部,有滚动过程，其中Y值可以设置为大于0的值，使Scrollview停在指定位置。
```



## ViewPager

### Item切换重建（主要是和Fragment搭配使用时，切换到第三个页面，第一个页面销毁）

通过setOffscreenPageLimit防止频繁销毁（setOffscreenPageLimit注释：设置当前page左右两侧应该被保持的page数量，超过这个限制，page会被销毁重建（只是销毁视图），onDestroy-onCreateView,但不会执行onDestroy。尽量维持这个值小，特别是有复杂布局的时候，因为如果这个值很大，就会占用很多内存，如果只有3-4页的话，可以全部保持active，可以保持page切换的顺滑



# [Tools命名空间的使用](https://developer.android.com/studio/write/tool-attributes.html) xmlns:tools="http://schemas.android.com/tools"

`tools`命名空间中的各种XML属性，这些属性支持设计时功能（例如，在片段中显示哪种布局）或编译时行为（例如应用于XML资源的缩小模式）。构建应用程序时，构建工具会删除这些属性，因此不会影响APK大小或运行时行为。

## 错误处理属性

### tools:ignore

*用于：任何元素*

*使用者：Lint*

此属性接受以逗号分隔的lint问题ID列表，用于忽略指定的错误

例如，忽略`MissingTranslation`错误：

```xml
<string name="show_all_apps" tools:ignore="MissingTranslation">All</string>
```



### tools:targetApi

*用于：任何元素*

*使用者：Lint*

此属性与[`@TargetApi`](https://developer.android.com/reference/android/annotation/TargetApi.html)Java代码中的注释相同 ：它允许指定支持此元素的API级别（作为整数或代码名称）。

表名此元素（以及任何子元素）将仅在指定的API级别或更高级别上使用。用于阻止lint警告

例如，指定 [`GridLayout`](https://developer.android.com/reference/android/widget/GridLayout.html)仅适用于API级别14及更高版本：

```xml
<GridLayout xmlns:android="http://schemas.android.com/apk/res/android"    xmlns:tools="http://schemas.android.com/tools"    tools:targetApi="14" >
```



### tools:locale

*用于： <resources>*

*使用者：Lint，Android Studio编辑器*

这告诉工具给定`<resources>`元素中的资源的默认语言/语言环境是什么（因为工具否则假定为英语），以避免来自拼写检查器的警告。该值必须是有效的 [区域设置限定符](https://developer.android.com/guide/topics/resources/providing-resources.html#LocaleQualifier)。

例如，将其添加到`values/strings.xml`文件（默认字符串值），以指示用于默认字符串的语言是西班牙语而不是英语：

```
<resources xmlns:tools="http://schemas.android.com/tools"    tools:locale="es">
```



## 设计时视图属性——仅在Android Studio布局预览中可见的布局特征

### `tools:` 代替 `android:`

*用于： <View>*

*使用者：Android Studio布局编辑器*

可以使用`tools:`前缀而不是Android框架中的`android:`任何`<View>`属性在布局预览中插入示例数据。当在App运行之前未填充属性的值，但你希望在布局预览中预先看到效果时，这非常有用。

例如，如果`android:text`属性值是在运行时设置的，或者你希望看到布局的值不同于默认值，则可以添加 `tools:text`以仅为布局预览指定一些文本。

![img](https://developer.android.com/studio/images/write/tools-attribute-text_2x.png)

**图1.**该`tools:text`属性将“Google Voice”设置为布局预览的值

你可以添加`android:`命名空间属性（在运行时使用）和匹配`tools:`属性（仅在布局预览中覆盖运行时属性）。或者仅使用`tools:`属性用于布局预览。例如：

```
<Button
    android:id="@+id/button"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="First" />

<Button
    android:id="@+id/button2"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Second"
    tools:visibility="invisible"  />
```



### tools:context

*用于：任何根 <View>*

*使用者：Lint，Android Studio布局编辑器*

此属性声明默认情况下此布局与哪个Activity相关联。比如导入View.onClick方法到次Activity。



### tools:itemCount

*用于： <RecyclerView>*

*使用者：Android Studio布局编辑器*

对于给定的`RecyclerView`，此属性指定布局编辑器应在“ **预览”**窗口中呈现的条目数 。

例如：

```
<android.support.v7.widget.RecyclerView    
	android:id="@+id/recyclerView"    
	android:layout_width="match_parent"    
	android:layout_height="match_parent"    
	tools:itemCount="3"/>
```



### tools:layout

*用于： <fragment>*

*使用者：Android Studio布局编辑器*

此属性声明你希望布局预览在Fragment内绘制的布局（因为布局预览无法执行通常应用布局的活动代码）。例如：

```
<fragment android:name="com.example.master.ItemListFragment"    
	tools:layout="@layout/list_content" />
```



### tools:listitem / tools:listheader / tools:listfooter

*用于:( <AdapterView>及其子类，比如<ListView>）*

*使用者：Android Studio布局编辑器*

这些属性指定在列表的条目，页眉和页脚的布局预览中显示的布局。布局中的任何数据字段都填充了数字内容，例如“项目1”，以便列表项不重复。

例如：

```
<ListView xmlns:android="http://schemas.android.com/apk/res/android"    			xmlns:tools="http://schemas.android.com/tools"    
	android:id="@android:id/list"    
	android:layout_width="match_parent"    
	android:layout_height="match_parent"    
	tools:listitem="@layout/sample_list_item"    
	tools:listheader="@layout/sample_list_header"   
	tools:listfooter="@layout/sample_list_footer" />
```



### tools:showIn

*用于：布局根<View>需要嵌入并一同显示预览的布局*

*使用者：Android Studio布局编辑器*

此属性将会把当前布局嵌入其引用的布局一同显示预览。例如：

```
<TextView xmlns:android="http://schemas.android.com/apk/res/android"    xmlns:tools="http://schemas.android.com/tools"    
    android:text="@string/hello_world"    
    android:layout_width="wrap_content"    
    android:layout_height="wrap_content"    
    tools:showIn="@layout/activity_main" />
```



## 资源收缩属性  

### [shrinkResources 的使用参考](https://blog.csdn.net/u011889786/article/details/56686492)



minifyEnabled 用来开启删除无用代码，比如没有引用到的代码

shrinkResources 用来开启删除无用资源，也就是没有被引用的文件（经过实测是drawable,layout，实际并不是彻底删除，而是保留文件名，但是没有内容，等等），但是因为需要知道是否被引用所以需要配合mififyEnable使用，只有当两者都为true的时候才会起到真正的删除无效代码和无引用资源的目的。例如：


```groovy
android {
    ...
    buildTypes {
        release {
            //收缩资源
            shrinkResources true
            //代码混淆
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
```



### tools:shrinkMode

*用于： <resources>*

*使用者：资源缩减构建工具*

此属性允许你指定构建工具是否应使用“安全模式”（保护安全并保留所有明确引用的资源以及*可能*通过调用动态 引用的资源[`Resources.getIdentifier()`](https://developer.android.com/reference/android/content/res/Resources.html#getIdentifier(java.lang.String, java.lang.String, java.lang.String))）或“严格模式”（仅保留资源）在代码或其他资源中明确引用）。

默认是使用安全模式（`shrinkMode="safe"`）。要改为使用严格模式，需更改为`shrinkMode="strict"`到`<resources>`节点，如下所示：

```xml
<?xml version="1.0" encoding="utf-8"?><resources xmlns:tools="http://schemas.android.com/tools"    tools:shrinkMode="strict" />
```

启用严格模式时，你可能需要使用[`tools:keep`](https://developer.android.com/studio/write/tool-attributes.html#toolskeep) 以保留已删除但实际需要的资源，并用于 [`tools:discard`](https://developer.android.com/studio/write/tool-attributes.html#toolsdiscard)显式删除更多资源。

有关更多信息，请参阅 [缩小资源](https://developer.android.com/studio/build/shrink-code.html#shrink-resources)。



### tools:keep

*用于： <resources>*

*使用者：资源缩减构建工具*

此属性用于指定你需要保留的资源（通常是因为它们在运行时以间接方式引用，例如通过将动态生成的资源名称传递给 [`Resources.getIdentifier()`](https://developer.android.com/reference/android/content/res/Resources.html#getIdentifier(java.lang.String, java.lang.String, java.lang.String))）。

用法：在资源目录（例如，at `res/raw/keep.xml`）中使用`<resources>`标记创建XML文件， 并将要保留在`tools:keep`属性中的每个资源指定为以逗号分隔的列表。可以将星号字符用作通配符。例如：

```xml
<?xml version="1.0" encoding="utf-8"?><resources xmlns:tools="http://schemas.android.com/tools"    tools:keep="@layout/used_1,@layout/used_2,@layout/*_3" />
```



### tools:discard

*用于： <resources>*

*使用者：资源缩减构建工具*

此属性用于指定你需要手动丢弃的资源（通常是因为资源*被*引用但不会影响你的应用，或者因为Gradle插件错误地推断出资源被引用）。

用法：在资源目录（例如，at `res/raw/keep.xml`）中使用`<resources>`标记创建XML文件， 并将要保留在`tools:discard`属性中的每个资源指定为以逗号分隔的列表。可以将星号字符用作通配符。例如：

```
<?xml version="1.0" encoding="utf-8"?><resources xmlns:tools="http://schemas.android.com/tools"    tools:discard="@layout/unused_1" />
```



# Style问题

### declare-styleable

#### format可选项 

```
"reference"//引用 
"color"//颜色 
"boolean"//布尔值 
"dimension"//尺寸值 
"float"//浮点值 
"integer"//整型值 
"string"//字符串 
"fraction"//百分数,比如200% 
"enum" 枚举值
```

```
//enum枚举例子
<attr name="status" format="enum">
    <enum name="content" value="-1"/>
    <enum name="loading" value="0"/>
    <enum name="error" value="1"/>
    <enum name="empty" value="2"/>
</attr>
```

# Activity等系统组件

### Activity切换时，短暂白屏问题，主要是启动singleTask或者singleInstance标志的Activity，同时关闭当前Activity，大概率出现这个问题

Activity theme加上  **<item name="android:windowDisablePreview">true</item> ** 即可。

这个标志应该也可以 <item name="android:windowIsTranslucent">true</item> ，然后onCreate应该需要去掉这个标志。

### moveTaskToBack(boolean) 点返回键时，把App移到后台，而不是退出

参数为false——代表只有当前activity是task根，指应用启动的第一个activity时，才有效;

参数为true——则忽略这个限制，任何activity都可以有效。

判断Activity是否是task根，Activity本身给出了相关方法：isTaskRoot()

把App移到后台，而不是退出，最简单无脑的使用方法：MainActivity，重写onBackPressed如下，即可实现

```
    override fun onBackPressed() {
        moveTaskToBack(true)
    }
```



## Activity设置为Dialog样式

方案：

1. Activity Theme设置为：

   不使用<item name="android:windowIsFloating">true</item>，而使用parent="@style/Theme.AppCompat.DayNight.Dialog"的原因是因为在页面上输入框会被软键盘遮住，软键盘不会顶起输入法

   ```
       <style name="ActivityDialogTheme" parent="@style/Theme.AppCompat.DayNight.Dialog">
           <item name="android:windowFrame">@null</item> <!-- 无windowFrame -->
           <!--        <item name="android:windowIsFloating">true</item>  &lt;!&ndash; 浮在activity之上 &ndash;&gt;-->
           <item name="android:windowIsTranslucent">false</item>  <!-- 半透明 -->
           <item name="android:windowNoTitle">true</item> <!-- 隐藏标题 -->
           <item name="windowNoTitle">true</item> <!-- 隐藏标题 -->
           <item name="android:background">@android:color/transparent</item>
           <item name="android:windowBackground">@android:color/transparent</item> <!-- 设置系统给定的透明值 -->
           <item name="android:backgroundDimEnabled">true</item>  <!-- 背景是否变暗-->
       </style>
   ```

2. 布局：android:minWidth或者android:layout_width设置为很大的dp，防止宽度太小的问题

   ```
   <?xml version="1.0" encoding="utf-8"?>
   <LinearLayout
       android:layout_width="match_parent"
       android:layout_height="match_parent"
       xmlns:android="http://schemas.android.com/apk/res/android">
   
       <LinearLayout
           xmlns:android="http://schemas.android.com/apk/res/android"
           android:layout_width="wrap_content"
           android:minWidth="1000dp"
           android:layout_height="326dp"
           android:layout_marginLeft="15dp"
           android:layout_marginRight="15dp"
           android:orientation="vertical"
           android:paddingLeft="15dp"
           android:paddingRight="15dp"
           android:background="@drawable/bg_hint_dialog_card">
   
           </LinearLayout>
   
       </LinearLayout>
   
   </LinearLayout>
   
   ```

3. 防止点击空白处页面关闭，Activity中调用：

   ```
   setFinishOnTouchOutside(false);
   ```

这个方案也有不完美的地方（应该算系统问题），布局较高，输入框靠下时，获取到焦点窗口上移时会被截断。但是不算大问题。



## MainActivity打开新的Activity，因点击触发异常后（直接onCreate抛异常App会直接死掉），返回当前Activity时，Fragment重叠问题

新Activity强退，导致MainActivity重新走了生命周期（onCreate-onStart-onResume）Activity保存了Fragment状态，在onCreate中savedInstanceState!=null，里头保存了Fragment状态：

```
android:support:fragments=android.support.v4.app.FragmentManagerState@ab76c10, android:fragments=android.app.FragmentManagerState@7661309}]
```



方案：

```
//方案1：super.onCreate(null); 传null解决
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        //解决Activity重建Fragment重叠问题
        super.onCreate(null);
    }
//方案2：按需清除Bundle中FragmentManagerState
```







# 输入法遮挡输入框问题

## 通用解决方法：AndroidManifest中Activity的windowSoftInputMode属性来调整

## 通用解决方法无法解决，主要是自定义软键盘的情况

ScrollView中的EditText被自定义输入法遮挡：

ScrollView子控件最后增加一个空布局，比如Space，默认隐藏；界面显示后，计算Space高度：软键盘高度-输入框底部距离当前Activity窗口底部距离，>0时，就是有遮挡问题。

监听自定义软键盘弹出/隐藏，或者输入框是否获得焦点，如果软键盘弹出/输入框获得焦点，显示Space，调用nestedScrollView.scrollTo(0, space.getHeight())，解决遮挡问题；软键盘隐藏时，隐藏Space





# Kotlin 专题

## Kotlin data class 和 Gson, @parcelize问题

gson 扩展方法

```kotlin
inline fun <reified T> Gson.fromJson(json: String) = fromJson(json, T::class.java)
```



@parcelize 需要在build.gradle android内设置属性

​	

```
androidExtensions {
    experimental = true
}
```



例子：

```kotlin
//正确的data class写法
@Parcelize
data class Student(
    var age: Int = 0,
    var name: String? = null,
    var toy: Toy? = null
) : Parcelable {
}

@Parcelize
data class Toy(
    var price: Int = 0,
    var name: String? = null
) : Parcelable {
}

//错误的data class写法，类体内的name不会序列化，别的地方使用获取到的是null，kotlin bytecode，decompile文件可以看到
@parcelize
data class Student(
	var age: Int = 0): Parcelable {
    var name: String? = null
}


gson解析
val student2 = gson.fromJson<Student>("{\n" +
                    "\t\"age\":10,\n" +
                    "\t\"name\":\"json name\",\n" +
                    "\t\"toy\":{\n" +
                    "\t\t\"name\":\"toy\",\n" +
                    "\t\t\"price\":1000\n" +
                    "\t}\n" +
                    "}", Student::class.java)
            intent.putExtra("student2", student2)
```



## 指定Java调用的类名，要用到注解 @file:JvmName(),放在package之前

也就是要写成这样，剩下的就直接改变代码里的类名就可以了。

```
@file:JvmName("Hello")
package kotlin
```



### @file:JvmName("Utils") 和 @file:JvmMultifileClass 一起使用的场景：

```
//在需要合并的每个Kotlin文件加入如下属性：
@file:JvmName("Utils")
@file:JvmMultifileClass
```

```
demo:

// oldutils.kt
@file:JvmName("Utils")
@file:JvmMultifileClass
package demo

fun foo() {
}

//___________________________________________________________
// newutils.kt
@file:JvmName("Utils")
@file:JvmMultifileClass
package demo
fun bar() {
}

//__________________________________________________________
// build后，在Java文件中的调用方法：
Utils.foo();
Utils.bar();

```



## [编写和Kotlin代码文档（相当于Javadoc）](https://www.kotlincn.net/docs/reference/kotlin-doc.html)

## 生成文档

Kotlin 的文档生成工具称为 [Dokka](https://github.com/Kotlin/dokka)。其使用说明请参见 [Dokka README](https://github.com/Kotlin/dokka/blob/master/README.md)。

Dokka 有 Gradle、Maven 和 Ant 的插件，因此你可以将文档生成集成到你的构建过程中。

