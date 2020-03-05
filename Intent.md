# Intent

## Intent是什么？

Intent一般作为参数来使用，协助完成 Android各个组件之间的通讯。

Intent主要包括7个属性：**Action（动作）**、**Data（数据）**、**Category（类别）**、**Type（数据类型）**、**Component（组件）**、**Extra（扩展信息）**、**Flag（标志位）**。其中最常用的是Action属性和Data属性。

## 表现形式：

* 启动Activity

  ```java
  startActivity(intent);
  startActivityForResult(intent, requestCode);
  ```

* 启动Service

  ```java
  startService(intent);
  ```

* 发送Broadcast

  ```java
  sendBroadcast(intent);
  sendOrderedBroadcast(intent, sendOrderedBroadcast);
  ```

## Intent种类

### 显式Intent

```java
//最常用方法
Intent intent = new Intent(this, SecondActivity.class);  
startActivity(intent);  

//setClass 或者 setClassName
Intent intent = new Intent();    
intent.setClass(this, SecondActivity.class);  
//或者intent.setClassName(this, "com.example.app.SecondActivity");  
//或者intent.setClassName(this.getPackageName(),"com.example.app.SecondActivity");            
startActivity(intent);  

//setComponent
Intent intent = new Intent();    
intent.setComponent(new ComponentName(getPackageName(), SecondActivity.class))
startActivity(intent);  
```



### 隐式Intent

隐式，不明确指定启动哪个Activity，而是设置Action、Data、Category，让系统来筛选出合适的Activity。筛选是根据所有的``来筛选。

```java
//打电话
Uri uri = Uri.parse("tel:10010");
Intent intent = new Intent(Intent.ACTION_DIAL, uri);
startActivity(intent);

//使用选择器
Intent sendIntent = new Intent(Intent.ACTION_SEND);
Intent chooser = Intent.createChooser(sendIntent, title);
// Verify the original intent will resolve to at least one activity
if (sendIntent.resolveActivity(getPackageManager()) != null) {
    startActivity(chooser);
}

```



