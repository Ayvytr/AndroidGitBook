# WebView



### WebView常用方法

- void loadUrl(String url):加载网络链接 url

- boolean canGoBack():判断 WebView 当前是否可以返回上一页

- goBack():回退到上一页

- boolean canGoForward():判断 WebView 当前是否可以向前一页

- goForward():回退到前一页

- onPause():类似 Activity 生命周期，页面进入后台不可见状态

- pauseTimers():该方法面向全局整个应用程序的webview，它会暂停所有webview的layout，parsing，JavaScript Timer。当程序进入后台时，该方法的调用可以降低CPU功耗。

- onResume():在调用 onPause()后，可以调用该方法来恢复 WebView 的运行。

- resumeTimers():恢复pauseTimers时的所有操作。(注：pauseTimers和resumeTimers 方法必须一起使用，否则再使用其它场景下的 WebView 会有问题)

- destroy():销毁 WebView

- clearHistory():清除当前 WebView 访问的历史记录。

- clearCache(boolean includeDiskFiles):清空网页访问留下的缓存数据。需要注意的时，由于缓存是全局的，所以只要是WebView用到的缓存都会被清空，即便其他地方也会使用到。该方法接受一个参数，从命名即可看出作用。若设为false，则只清空内存里的资源缓存，而不清空磁盘里的。

- reload():重新加载当前请求

- setLayerType(int layerType, Paint paint):设置硬件加速、软件加速

- removeAllViews():清除子view。

- clearSslPreferences():清除ssl信息。

- clearMatches():清除网页查找的高亮匹配字符。

- removeJavascriptInterface(String interfaceName):删除interfaceName 对应的注入对象

- addJavascriptInterface(Object object,String interfaceName):注入 java 对象。

- setVerticalScrollBarEnabled(boolean verticalScrollBarEnabled):设置垂直方向滚动条。

- setHorizontalScrollBarEnabled(boolean horizontalScrollBarEnabled):设置横向滚动条。

- loadUrl(String url, Map<String, String> additionalHttpHeaders):加载制定url并携带http header数据。

- evaluateJavascript(String script, ValueCallback<String> resultCallback):Api 19 之后可以采用此方法之行 Js。

- stopLoading():停止 WebView 当前加载。

- clearView():在Android 4.3及其以上系统这个api被丢弃了， 并且这个api大多数情况下会有bug，经常不能清除掉之前的渲染数据。官方建议通过loadUrl("about:blank")来实现这个功能，阴雨需要重新加载一个页面自然时间会收到影响。

- freeMemory():释放内存，不过貌似不好用。

- clearFormData():清除自动完成填充的表单数据。需要注意的是，该方法仅仅清除当前表单域自动完成填充的表单数据，并不会清除WebView存储到本地的数据。

  



## WebView优化方案

### 1.WebView 动态加载 

 WebView 动态加载。就是不在xml中写WebView，写一个layout，然后把WebView add进去。

```
WebView mWebView = new WebView(getApplicationgContext());
LinearLayout mll = findViewById(R.id.xxx);
mll.addView(mWebView);
```

然后：

```
protected void onDestroy() {
      super.onDestroy();
      mWebView.removeAllViews();
      mWebView.destroy()
}
```

这里用的getApplicationContext()也是防止内存溢出，这种方法有一个问题。如果你需要在WebView中打开链接或者你打开的页面带有flash，获得你的WebView想弹出一个dialog，都会导致从ApplicationContext到ActivityContext的强制类型转换错误，从而导致你应用崩溃。这是因为在加载flash的时候，系统会首先把你的WebView作为父控件，然后在该控件上绘制flash，他想找一个Activity的Context来绘制他，但是你传入的是ApplicationContext。然后就崩溃了。。。

 

### 2.独立的web进程，与主进程隔开

这个方法被运用于类似qq，微信这样的超级app中，这也是解决任何webview内存问题屡试不爽的方法
 对于封装的webactivity，在`manifest.xml`中设置

```
<activity android:name=".webview.WebViewActivity" android:launchMode="singleTop" android:process=":remote" android:screenOrientation="unspecified" />
```

然后在关闭webactivity时销毁进程

```
@Overrideprotected void onDestroy() {                
     super.onDestroy(); 
     System.exit(0);
}
```

 关闭浏览器后便销毁整个进程，这样一般`95%`的情况下不会造成内存泄漏之类的问题，但这就涉及到[android进程间通讯](https://link.jianshu.com/?t=http://blog.csdn.net/hitlion2008/article/details/9824009)，比较不方便处理， 优劣参半，也是可选的一个方案. 





### WebView释放

完整的代码如下：

```
	public void destroy() {
        if (mWebView != null) {
            // 如果先调用destroy()方法，则会命中if (isDestroyed()) return;这一行代码，需要先onDetachedFromWindow()，再
            // destory()
            ViewParent parent = mWebView.getParent();
            if (parent != null) {
                ((ViewGroup) parent).removeView(mWebView);
            }

            mWebView.stopLoading();
            // 退出时调用此方法，移除绑定的服务，否则某些特定系统会报错
            mWebView.getSettings().setJavaScriptEnabled(false);
            mWebView.clearHistory();
            mWebView.clearView();
            mWebView.removeAllViews();

            try {
                mWebView.destroy();
            } catch (Throwable ex) {

            }
        }
    }
```



### 更多文章

* [Android WebView 全面干货指南](https://www.jianshu.com/p/fd61e8f4049e)

 

### 其他解决方案

* [**TBS腾讯浏览服务**](https://link.jianshu.com/?t=http://x5.tencent.com/index)  
* Hybrid方案