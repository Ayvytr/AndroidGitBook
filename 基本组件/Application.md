# Application

#### 作用

1. onCreate 初始化全局资源
2. registerComponentCallbacks() & unregisterComponentCallbacks() 注册和注销 `ComponentCallbacks2`回调接口

```java
registerComponentCallbacks(new ComponentCallbacks2() {
// 接口里方法下面会继续介绍
            @Override
            public void onTrimMemory(int level) {

            }

            @Override
            public void onLowMemory() {

            }

            @Override
            public void onConfigurationChanged(Configuration newConfig) {

            }
        });
```



3. onTrimMemory 通知 应用程序 当前内存使用情况（以内存级别进行识别）

   ![img](http://upload-images.jianshu.io/upload_images/944365-438db7cf11e0591b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4. onLowMemory 监听 `Android`系统整体内存较低时刻

   应用场景：Android 4.0前 检测内存使用情况，从而避免被系统直接杀掉 & 优化应用程序的性能体验

   类似于 OnTrimMemory（）

   特别注意：OnTrimMemory（） & OnLowMemory（） 关系

   OnTrimMemory（）是 OnLowMemory（） Android 4.0后的替代 API
   OnLowMemory（） = OnTrimMemory（）中的TRIM_MEMORY_COMPLETE级别

   若想兼容Android 4.0前，请使用OnLowMemory（）；否则直接使用OnTrimMemory（）即可

5. ### onConfigurationChanged 监听 应用程序 配置信息的改变，如屏幕旋转等

6. ### registerActivityLifecycleCallbacks（） & unregisterActivityLifecycleCallbacks（）注册 / 注销对 应用程序内 所有`Activity`的生命周期监听

7. ### onTerminate 应用程序结束时调用 但该方法只用于Android仿真机测试，在Android产品机是不会调用的

#### 使用场景

- 初始化 应用程序级别 的资源，如全局对象、环境配置变量等
- 数据共享、数据缓存，如设置全局共享变量、方法等
- 获取应用程序当前的内存使用情况，及时释放资源，从而避免被系统杀死
- 监听 应用程序 配置信息的改变，如屏幕旋转等
- 监听应用程序内 所有Activity的生命周期

#### 生命周期

```java
public class App extends Application {

    @Override
    public void onCreate() {
        // 程序创建的时候执行
        Log.d(TAG, "onCreate");
        super.onCreate();
    }
    @Override
    public void onTerminate() {
        // 程序终止的时候执行
        Log.d(TAG, "onTerminate");
        super.onTerminate();
    }
    @Override
    public void onLowMemory() {
        // 低内存的时候执行
        Log.d(TAG, "onLowMemory");
        super.onLowMemory();
    }
    @Override
    public void onTrimMemory(int level) {
        // 程序在内存清理的时候执行
        Log.d(TAG, "onTrimMemory");
        super.onTrimMemory(level);
    }
    @Override
    public void onConfigurationChanged(Configuration newConfig) {
        Log.d(TAG, "onConfigurationChanged");
        super.onConfigurationChanged(newConfig);
    }
    
}
```

