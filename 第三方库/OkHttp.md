# OkHttp



**参考：[彻底理解OkHttp - OkHttp 源码解析及OkHttp的设计思想](https://www.jianshu.com/p/cb444f49a777)**





## 用法

1. 创建OkHttp实例，可通过`OkHttpClient.Builder`添加拦截器等自定配置

2. 请求网络，可以同步/异步

   ```java
   Request request = new Request.Builder().url(url).build();
   //同步
   Response response = okHttpClient.newCall(request).execute()；
   response.body().string();
   
   //异步
   okHttpClient.newCall(request).enqueue();
   ```

   



## OkHttp大致执行流程

通过OkHttpClient.newCall内部的RealCall发起网络请求（execute同步请求和enqueue异步请求，异步请求有DIspatcher进行分发），`getResponseWithInterceptorChain`，`RetryAndFollowUpInterceptors`, 以及`BridgeInterceptor`, `CacheInterceptor`, `ConnectInterceptor`, `NetworkInterceptor`, `CallServerInterceptor`进行请求处理，最后返回请求结果，请求结束。

