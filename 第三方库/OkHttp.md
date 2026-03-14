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





## 核心类分析

### 1. **OkHttpClient - 配置中心**

java

java

下载

复制



```
// 建造者模式配置
OkHttpClient client = new OkHttpClient.Builder()
    .connectTimeout(10, TimeUnit.SECONDS)
    .readTimeout(30, TimeUnit.SECONDS)
    .writeTimeout(30, TimeUnit.SECONDS)
    .addInterceptor(new LoggingInterceptor())
    .cache(new Cache(cacheDir, cacheSize))
    .build();

// 关键字段
public final class OkHttpClient implements Cloneable, Call.Factory {
    final Dispatcher dispatcher;           // 调度器
    final List<Interceptor> interceptors;  // 应用拦截器
    final List<Interceptor> networkInterceptors; // 网络拦截器
    final ConnectionPool connectionPool;   // 连接池
    final Cache cache;                     // 缓存
    final Dns dns;                         // DNS 解析器
    final Proxy proxy;                     // 代理
    // ... 其他配置
}
```

**源码亮点**：

- 

  不可变对象设计，线程安全

- 

  深度克隆支持配置修改

- 

  默认配置优化（连接池大小、超时时间）

### 2. **Call - 请求抽象**

java

java

下载

复制



```
public interface Call extends Cloneable {
    Request request();
    Response execute() throws IOException;
    void enqueue(Callback responseCallback);
    void cancel();
    boolean isExecuted();
    boolean isCanceled();
    
    interface Factory {
        Call newCall(Request request);
    }
}

// 实际实现
final class RealCall implements Call {
    final OkHttpClient client;
    final Request originalRequest;
    final boolean forWebSocket;
    
    // 关键方法
    @Override public Response execute() throws IOException {
        synchronized (this) {
            if (executed) throw new IllegalStateException("Already Executed");
            executed = true;
        }
        try {
            client.dispatcher().executed(this);
            Response result = getResponseWithInterceptorChain();
            if (result == null) throw new IOException("Canceled");
            return result;
        } finally {
            client.dispatcher().finished(this);
        }
    }
}
```

### 3. **Dispatcher - 调度器**

java

java

下载

复制



```
public final class Dispatcher {
    // 三个队列
    private final Deque<AsyncCall> readyAsyncCalls = new ArrayDeque<>();
    private final Deque<AsyncCall> runningAsyncCalls = new ArrayDeque<>();
    private final Deque<RealCall> runningSyncCalls = new ArrayDeque<>();
    
    // 线程池
    private ExecutorService executorService;
    
    // 最大并发请求数（默认64）
    private int maxRequests = 64;
    // 每主机最大并发数（默认5）
    private int maxRequestsPerHost = 5;
    
    // 关键方法：执行异步请求
    synchronized void enqueue(AsyncCall call) {
        if (runningAsyncCalls.size() < maxRequests 
            && runningCallsForHost(call) < maxRequestsPerHost) {
            runningAsyncCalls.add(call);
            executorService().execute(call);
        } else {
            readyAsyncCalls.add(call);
        }
    }
}
```

**调度策略**：

1. 

   同步请求：直接在当前线程执行

2. 

   异步请求：由线程池执行，受并发数限制

3. 

   主机限制：防止对单个服务器过度请求

### 4. **Interceptor - 拦截器链**

java

java

下载

复制



```
public interface Interceptor {
    Response intercept(Chain chain) throws IOException;
    
    interface Chain {
        Request request();
        Response proceed(Request request) throws IOException;
        Connection connection();
    }
}

// 拦截器链实现
public Response getResponseWithInterceptorChain() throws IOException {
    // 构建完整的拦截器链
    List<Interceptor> interceptors = new ArrayList<>();
    interceptors.addAll(client.interceptors());                    // 应用拦截器
    interceptors.add(new RetryAndFollowUpInterceptor(client));     // 重试拦截器
    interceptors.add(new BridgeInterceptor(client.cookieJar()));   // 桥接拦截器
    interceptors.add(new CacheInterceptor(client.cache()));        // 缓存拦截器
    interceptors.add(new ConnectInterceptor(client));              // 连接拦截器
    interceptors.addAll(client.networkInterceptors());             // 网络拦截器
    interceptors.add(new CallServerInterceptor(forWebSocket));     // 服务调用拦截器
    
    Interceptor.Chain chain = new RealInterceptorChain(
        interceptors, null, null, null, 0, originalRequest);
    
    return chain.proceed(originalRequest);
}
```

------

## 三、拦截器详细分析

### 1. **RetryAndFollowUpInterceptor - 重试和重定向**

java

java

下载

复制



```
public final class RetryAndFollowUpInterceptor implements Interceptor {
    @Override public Response intercept(Chain chain) throws IOException {
        Request request = chain.request();
        RealInterceptorChain realChain = (RealInterceptorChain) chain;
        
        int followUpCount = 0;
        Response priorResponse = null;
        
        while (true) {
            // 尝试连接
            try {
                Response response = realChain.proceed(request);
                // 检查是否需要重试或重定向
                Request followUp = followUpRequest(response);
                
                if (followUp == null) {
                    return response;  // 请求完成
                }
                
                // 检查重试次数限制
                if (++followUpCount > MAX_FOLLOW_UPS) {
                    throw new ProtocolException("Too many follow-up requests: " + followUpCount);
                }
                
                // 准备下一次请求
                request = followUp;
                priorResponse = response;
                
            } catch (IOException e) {
                // 检查是否可恢复的异常
                if (!recover(e, request)) {
                    throw e;
                }
                // 重试
                continue;
            }
        }
    }
}
```

**处理场景**：

- 

  网络异常重试（IOException）

- 

  HTTP 重定向（3xx 状态码）

- 

  认证重试（401/407）

- 

  连接超时重试

### 2. **BridgeInterceptor - 桥接拦截器**

java

java

下载

复制



```
public final class BridgeInterceptor implements Interceptor {
    @Override public Response intercept(Chain chain) throws IOException {
        Request userRequest = chain.request();
        Request.Builder requestBuilder = userRequest.newBuilder();
        
        // 添加默认请求头
        if (userRequest.header("Host") == null) {
            requestBuilder.header("Host", hostHeader(userRequest.url()));
        }
        if (userRequest.header("Connection") == null) {
            requestBuilder.header("Connection", "Keep-Alive");
        }
        if (userRequest.header("Accept-Encoding") == null) {
            requestBuilder.header("Accept-Encoding", "gzip");
        }
        // ... 添加其他默认头
        
        // 处理 Cookie
        List<Cookie> cookies = cookieJar.loadForRequest(userRequest.url());
        if (!cookies.isEmpty()) {
            requestBuilder.header("Cookie", cookieHeader(cookies));
        }
        
        // 执行请求
        Response networkResponse = chain.proceed(requestBuilder.build());
        
        // 处理响应
        HttpHeaders.receiveHeaders(cookieJar, userRequest.url(), networkResponse.headers());
        
        Response.Builder responseBuilder = networkResponse.newBuilder()
            .request(userRequest);
        
        // 处理 GZIP 压缩
        if (transparentGzip 
            && "gzip".equalsIgnoreCase(networkResponse.header("Content-Encoding"))
            && HttpHeaders.hasBody(networkResponse)) {
            GzipSource responseBody = new GzipSource(networkResponse.body().source());
            Headers strippedHeaders = networkResponse.headers().newBuilder()
                .removeAll("Content-Encoding")
                .removeAll("Content-Length")
                .build();
            responseBuilder.headers(strippedHeaders);
            responseBuilder.body(new RealResponseBody(
                strippedHeaders, Okio.buffer(responseBody)));
        }
        
        return responseBuilder.build();
    }
}
```

### 3. **CacheInterceptor - 缓存拦截器**

java

java

下载

复制



```
public final class CacheInterceptor implements Interceptor {
    @Override public Response intercept(Chain chain) throws IOException {
        // 1. 尝试从缓存获取响应
        Response cacheCandidate = cache != null 
            ? cache.get(chain.request()) 
            : null;
        
        long now = System.currentTimeMillis();
        
        // 2. 缓存策略决策
        CacheStrategy strategy = new CacheStrategy.Factory(
            now, chain.request(), cacheCandidate).get();
        
        Request networkRequest = strategy.networkRequest;
        Response cacheResponse = strategy.cacheResponse;
        
        // 3. 记录缓存使用情况
        if (cache != null) {
            cache.trackResponse(strategy);
        }
        
        // 4. 如果缓存无效且不允许网络请求
        if (cacheCandidate != null && cacheResponse == null) {
            closeQuietly(cacheCandidate.body());
        }
        
        // 5. 如果禁止网络，直接返回缓存（可能为504）
        if (networkRequest == null && cacheResponse == null) {
            return new Response.Builder()
                .request(chain.request())
                .protocol(Protocol.HTTP_1_1)
                .code(504)
                .message("Unsatisfiable Request (only-if-cached)")
                .body(Util.EMPTY_RESPONSE)
                .sentRequestAtMillis(-1L)
                .receivedResponseAtMillis(System.currentTimeMillis())
                .build();
        }
        
        // 6. 不使用网络，直接返回缓存
        if (networkRequest == null) {
            return cacheResponse.newBuilder()
                .cacheResponse(stripBody(cacheResponse))
                .build();
        }
        
        Response networkResponse = null;
        try {
            // 7. 执行网络请求
            networkResponse = chain.proceed(networkRequest);
        } finally {
            // 8. 如果网络请求失败，但缓存可用
            if (networkResponse == null && cacheCandidate != null) {
                closeQuietly(cacheCandidate.body());
            }
        }
        
        // 9. 如果收到304（Not Modified），更新缓存
        if (cacheResponse != null) {
            if (networkResponse.code() == HTTP_NOT_MODIFIED) {
                Response response = cacheResponse.newBuilder()
                    .headers(combine(cacheResponse.headers(), networkResponse.headers()))
                    .sentRequestAtMillis(networkResponse.sentRequestAtMillis())
                    .receivedResponseAtMillis(networkResponse.receivedResponseAtMillis())
                    .cacheResponse(stripBody(cacheResponse))
                    .networkResponse(stripBody(networkResponse))
                    .build();
                networkResponse.body().close();
                
                // 更新缓存
                cache.trackConditionalCacheHit();
                cache.update(cacheResponse, response);
                return response;
            } else {
                closeQuietly(cacheResponse.body());
            }
        }
        
        // 10. 构建最终响应
        Response response = networkResponse.newBuilder()
            .cacheResponse(stripBody(cacheResponse))
            .networkResponse(stripBody(networkResponse))
            .build();
        
        // 11. 缓存响应
        if (cache != null) {
            if (HttpHeaders.hasBody(response) && CacheStrategy.isCacheable(response, networkRequest)) {
                CacheRequest cacheRequest = cache.put(response);
                return cacheWritingResponse(cacheRequest, response);
            }
            
            if (HttpMethod.invalidatesCache(networkRequest.method())) {
                try {
                    cache.remove(networkRequest);
                } catch (IOException ignored) {
                }
            }
        }
        
        return response;
    }
}
```

**缓存策略**：

- 

  **强制缓存**：Cache-Control: max-age, s-maxage

- 

  **协商缓存**：ETag, Last-Modified

- 

  **启发式缓存**：根据 Date, Age 等头部计算

### 4. **ConnectInterceptor - 连接拦截器**

java

java

下载

复制



```
public final class ConnectInterceptor implements Interceptor {
    @Override public Response intercept(Chain chain) throws IOException {
        RealInterceptorChain realChain = (RealInterceptorChain) chain;
        Request request = realChain.request();
        Transmitter transmitter = realChain.transmitter();
        
        // 是否需要建立新连接
        boolean doExtensiveHealthChecks = !request.method().equals("GET");
        Exchange exchange = transmitter.newExchange(chain, doExtensiveHealthChecks);
        
        return realChain.proceed(request, transmitter, exchange);
    }
}
```

**连接建立流程**：

1. 

   从连接池获取可用连接

2. 

   如果没有可用连接，创建新连接

3. 

   执行 TCP 握手和 TLS 握手

4. 

   建立 HTTP/2 连接（如果支持）

### 5. **CallServerInterceptor - 服务调用拦截器**

java

java

下载

复制



```
public final class CallServerInterceptor implements Interceptor {
    @Override public Response intercept(Chain chain) throws IOException {
        RealInterceptorChain realChain = (RealInterceptorChain) chain;
        Exchange exchange = realChain.exchange();
        Request request = realChain.request();
        
        // 1. 写入请求头
        long sentRequestMillis = System.currentTimeMillis();
        exchange.writeRequestHeaders(request);
        
        // 2. 如果有请求体，写入请求体
        boolean responseHeadersStarted = false;
        Response.Builder responseBuilder = null;
        
        if (HttpMethod.permitsRequestBody(request.method()) && request.body() != null) {
            // 如果有 Expect: 100-continue 头部
            if ("100-continue".equalsIgnoreCase(request.header("Expect"))) {
                exchange.flushRequest();
                responseHeadersStarted = true;
                exchange.responseHeadersStart();
                responseBuilder = exchange.readResponseHeaders(true);
            }
            
            if (responseBuilder == null) {
                // 写入请求体
                Sink requestBodyOut = exchange.createRequestBody(request, false);
                BufferedSink bufferedRequestBody = Okio.buffer(requestBodyOut);
                request.body().writeTo(bufferedRequestBody);
                bufferedRequestBody.close();
            } else {
                // 服务器拒绝了 100-continue
                exchange.noRequestBody();
                if (!exchange.connection().isMultiplexed()) {
                    exchange.noNewExchangesOnConnection();
                }
            }
        } else {
            exchange.noRequestBody();
        }
        
        if (request.body() == null || !request.body().isDuplex()) {
            exchange.finishRequest();
        }
        
        // 3. 读取响应头
        if (!responseHeadersStarted) {
            exchange.responseHeadersStart();
        }
        
        if (responseBuilder == null) {
            responseBuilder = exchange.readResponseHeaders(false);
        }
        
        Response response = responseBuilder
            .request(request)
            .handshake(exchange.connection().handshake())
            .sentRequestAtMillis(sentRequestMillis)
            .receivedResponseAtMillis(System.currentTimeMillis())
            .build();
        
        // 4. 读取响应体
        int code = response.code();
        if (code == 100) {
            // 继续读取真正的响应
            response = exchange.readResponseHeaders(false)
                .request(request)
                .handshake(exchange.connection().handshake())
                .sentRequestAtMillis(sentRequestMillis)
                .receivedResponseAtMillis(System.currentTimeMillis())
                .build();
            code = response.code();
        }
        
        exchange.responseHeadersEnd(response);
        
        // 5. 构建最终响应
        if (forWebSocket && code == 101) {
            // WebSocket 升级响应
            response = response.newBuilder()
                .body(Util.EMPTY_RESPONSE)
                .build();
        } else {
            response = response.newBuilder()
                .body(exchange.openResponseBody(response))
                .build();
        }
        
        // 6. 处理连接关闭
        if ("close".equalsIgnoreCase(response.request().header("Connection"))
            || "close".equalsIgnoreCase(response.header("Connection"))) {
            exchange.noNewExchangesOnConnection();
        }
        
        // 7. 处理 204 和 205 状态码
        if ((code == 204 || code == 205) && response.body().contentLength() > 0) {
            throw new ProtocolException(
                "HTTP " + code + " had non-zero Content-Length: " + response.body().contentLength());
        }
        
        return response;
    }
}
```

------

## 四、连接管理

### 1. **RealConnection - 真实连接**

java

java

下载

复制



```
public final class RealConnection extends Http2Connection.Listener implements Connection {
    private final ConnectionPool connectionPool;
    private final Route route;
    
    // Socket 相关
    private Socket rawSocket;
    private Socket socket;
    private Handshake handshake;
    private Protocol protocol;
    private Http2Connection http2Connection;
    
    // 流相关
    private int allocationLimit = 1;  // HTTP/1.x 为1，HTTP/2 为 Integer.MAX_VALUE
    private final List<Reference<Transmitter>> allocations = new ArrayList<>();
    
    // 关键方法：建立连接
    public void connect(int connectTimeout, int readTimeout, int writeTimeout,
                       boolean connectionRetryEnabled, Call call, EventListener eventListener) {
        
        // 1. 代理处理
        Proxy proxy = route.proxy();
        Address address = route.address();
        
        // 2. 根据代理类型选择 Socket
        if (proxy.type() == Proxy.Type.DIRECT || proxy.type() == Proxy.Type.HTTP) {
            socket = address.socketFactory().createSocket();
        } else {
            socket = new Socket(proxy);
        }
        
        // 3. 连接超时设置
        socket.setSoTimeout(readTimeout);
        
        // 4. 建立 TCP 连接
        Platform.get().connectSocket(socket, route.socketAddress(), connectTimeout);
        
        // 5. 建立源
        source = Okio.buffer(Okio.source(socket));
        sink = Okio.buffer(Okio.sink(socket));
        
        // 6. TLS 握手（如果需要）
        if (route.address().sslSocketFactory() != null) {
            connectTls(readTimeout, writeTimeout, call, eventListener);
        } else {
            protocol = Protocol.HTTP_1_1;
            socket = rawSocket;
        }
        
        // 7. 如果是 HTTP/2，建立 HTTP/2 连接
        if (protocol == Protocol.HTTP_2) {
            socket.setSoTimeout(0); // HTTP/2 使用自己的超时机制
            http2Connection = new Http2Connection.Builder(true)
                .socket(socket, route.address().url().host(), source, sink)
                .listener(this)
                .build();
            http2Connection.start();
        }
    }
}
```

### 2. **ConnectionPool - 连接池**

java

java

下载

复制



```
public final class ConnectionPool {
    // 空闲连接队列
    private final Deque<RealConnection> connections = new ArrayDeque<>();
    
    // 清理任务
    private final Runnable cleanupRunnable = () -> {
        while (true) {
            long waitNanos = cleanup(System.nanoTime());
            if (waitNanos == -1) return;
            if (waitNanos > 0) {
                long waitMillis = waitNanos / 1000000L;
                waitNanos -= (waitMillis * 1000000L);
                synchronized (ConnectionPool.this) {
                    try {
                        ConnectionPool.this.wait(waitMillis, (int) waitNanos);
                    } catch (InterruptedException ignored) {
                    }
                }
            }
        }
    };
    
    // 关键方法：获取连接
    RealConnection get(Address address, Transmitter transmitter, Route route) {
        for (RealConnection connection : connections) {
            // 1. 检查连接是否可用
            if (!connection.isEligible(address, route)) continue;
            
            // 2. 分配流
            transmitter.acquireConnectionNoEvents(connection);
            return connection;
        }
        return null;
    }
    
    // 清理算法
    long cleanup(long now) {
        int inUseConnectionCount = 0;
        int idleConnectionCount = 0;
        RealConnection longestIdleConnection = null;
        long longestIdleDurationNs = Long.MIN_VALUE;
        
        // 遍历所有连接
        for (Iterator<RealConnection> i = connections.iterator(); i.hasNext(); ) {
            RealConnection connection = i.next();
            
            // 统计使用中的连接
            if (pruneAndGetAllocationCount(connection, now) > 0) {
                inUseConnectionCount++;
                continue;
            }
            
            // 统计空闲连接
            idleConnectionCount++;
            
            // 计算空闲时间
            long idleDurationNs = now - connection.idleAtNanos;
            if (idleDurationNs > longestIdleDurationNs) {
                longestIdleDurationNs = idleDurationNs;
                longestIdleConnection = connection;
            }
        }
        
        // 清理决策
        if (longestIdleDurationNs >= keepAliveDurationNs
            || idleConnectionCount > maxIdleConnections) {
            // 移除最旧的空闲连接
            connections.remove(longestIdleConnection);
            closeQuietly(longestIdleConnection.socket());
            
            // 立即再次清理
            return 0;
        } else if (idleConnectionCount > 0) {
            // 等待直到连接过期
            return keepAliveDurationNs - longestIdleDurationNs;
        } else if (inUseConnectionCount > 0) {
            // 所有连接都在使用中，5分钟后再次检查
            return keepAliveDurationNs;
        } else {
            // 没有连接，停止清理
            return -1;
        }
    }
}
```

**连接池策略**：

- 

  默认最大空闲连接数：5

- 

  默认连接保持时间：5分钟

- 

  支持 HTTP/2 多路复用

------

## 五、HTTP/2 实现

### 1. **Http2Connection - HTTP/2 连接**

java

java

下载

复制



```
public final class Http2Connection {
    // 流管理
    private final Map<Integer, Http2Stream> streams = new LinkedHashMap<>();
    private int lastGoodStreamId;
    private int nextStreamId;
    
    // 设置帧
    private final Settings okHttpSettings = new Settings();
    
    // 关键方法：创建新流
    public Http2Stream newStream(int associatedStreamId, List<Header> requestHeaders,
                                boolean out) throws IOException {
        // 1. 分配流ID
        int streamId;
        synchronized (writer) {
            synchronized (this) {
                if (nextStreamId > 0) {
                    streamId = nextStreamId;
                    nextStreamId += 2;
                } else {
                    throw new ConnectionShutdownException();
                }
                
                // 2. 创建流
                Http2Stream stream = new Http2Stream(streamId, this, out, 
                    associatedStreamId, requestHeaders);
                streams.put(streamId, stream);
                
                // 3. 发送 HEADERS 帧
                if (out) {
                    writer.headers(outFinished, streamId, associatedStreamId, 
                        requestHeaders);
                }
                
                return stream;
            }
        }
    }
    
    // 多路复用：单个连接支持多个并发流
    public synchronized int maxConcurrentStreams() {
        return peerSettings.getMaxConcurrentStreams(Integer.MAX_VALUE);
    }
}
```

### 2. **头部压缩（HPACK）**

java

java

下载

复制



```
public final class Hpack {
    // 静态表（预定义的常用头部）
    static final List<Header> STATIC_HEADER_TABLE = Arrays.asList(
        new Header(":authority", ""),
        new Header(":method", "GET"),
        new Header(":method", "POST"),
        new Header(":path", "/"),
        new Header(":path", "/index.html"),
        // ... 61个预定义头部
    );
    
    // 动态表（运行时维护）
    private final List<Header> dynamicTable = new ArrayList<>();
    private int dynamicTableByteCount = 0;
    private int maxDynamicTableByteCount = 4096; // 默认4KB
    
    // 编码
    void writeHeaders(List<Header> headerBlock) throws IOException {
        for (int i = 0; i < headerBlock.size(); i++) {
            Header header = headerBlock.get(i);
            
            // 1. 尝试在静态表中查找完全匹配
            int staticIndex = HeaderTable.indexOf(STATIC_HEADER_TABLE, header);
            if (staticIndex != -1) {
                // 使用索引表示
                writer.writeInt(staticIndex, PREFIX_7_BITS, 0x80);
                continue;
            }
            
            // 2. 尝试在动态表中查找完全匹配
            int dynamicIndex = HeaderTable.indexOf(dynamicTable, header);
            if (dynamicIndex != -1) {
                // 使用索引表示（动态表索引从62开始）
                writer.writeInt(dynamicIndex + STATIC_HEADER_TABLE.size(), 
                    PREFIX_7_BITS, 0x80);
                continue;
            }
            
            // 3. 字面编码
            writer.writeByte(0x40); // 字面头部，增量索引
            writer.writeString(header.name);
            writer.writeString(header.value);
            
            // 4. 插入动态表
            insertIntoDynamicTable(header);
        }
    }
}
```

------

## 六、缓存实现

### 1. **Cache - 磁盘缓存**

java

java

下载

复制



```
public final class Cache implements Closeable, Flushable {
    final DiskLruCache cache;
    private int writeSuccessCount;
    private int writeAbortCount;
    
    // 关键方法：获取缓存响应
    Response get(Request request) {
        String key = key(request.url());
        DiskLruCache.Snapshot snapshot;
        Entry entry;
        try {
            snapshot = cache.get(key);
            if (snapshot == null) {
                return null;
            }
        } catch (IOException e) {
            return null;
        }
        
        try {
            entry = new Entry(snapshot.getSource(ENTRY_METADATA));
        } catch (IOException e) {
            Util.closeQuietly(snapshot);
            return null;
        }
        
        Response response = entry.response(snapshot);
        if (!entry.matches(request, response)) {
            Util.closeQuietly(response.body());
            return null;
        }
        
        return response;
    }
    
    // 缓存条目
    private static final class Entry {
        final String url;
        final Headers varyHeaders;
        final String requestMethod;
        final Protocol protocol;
        final int code;
        final String message;
        final Headers responseHeaders;
        final Handshake handshake;
        
        // 从磁盘读取
        Entry(Source in) throws IOException {
            try {
                BufferedSource source = Okio.buffer(in);
                url = source.readUtf8LineStrict();
                requestMethod = source.readUtf8LineStrict();
                varyHeaders = readHeaders(source);
                protocol = Protocol.get(source.readUtf8LineStrict());
                code = source.readInt();
                message = source.readUtf8LineStrict();
                responseHeaders = readHeaders(source);
                
                if (isHttps()) {
                    String blank = source.readUtf8LineStrict();
                    if (blank.length() > 0) {
                        throw new IOException("expected \"\" but was \"" + blank + "\"");
                    }
                    handshake = Handshake.get(source);
                } else {
                    handshake = null;
                }
            } finally {
                in.close();
            }
        }
    }
}
```

### 2. **缓存策略算法**

java

java

下载

复制



```
public final class CacheStrategy {
    public static class Factory {
        public CacheStrategy get() {
            CacheStrategy candidate = getCandidate();
            
            // 如果网络请求不为null且缓存响应不为null
            if (candidate.networkRequest != null && request.cacheControl().onlyIfCached()) {
                // 请求指定了 only-if-cached，但需要网络 → 返回504
                return new CacheStrategy(null, null);
            }
            
            return candidate;
        }
        
        private CacheStrategy getCandidate() {
            // 1. 如果没有缓存，强制网络请求
            if (cacheResponse == null) {
                return new CacheStrategy(request, null);
            }
            
            // 2. 如果是HTTPS且缓存缺少握手信息
            if (request.isHttps() && cacheResponse.handshake() == null) {
                return new CacheStrategy(request, null);
            }
            
            // 3. 检查缓存响应是否可缓存
            if (!isCacheable(cacheResponse, request)) {
                return new CacheStrategy(request, null);
            }
            
            CacheControl requestCaching = request.cacheControl();
            CacheControl responseCaching = cacheResponse.cacheControl();
            
            // 4. 检查请求是否禁止缓存
            if (requestCaching.noCache() || hasConditions(request)) {
                return new CacheStrategy(request, null);
            }
            
            // 5. 计算缓存年龄和新鲜度
            long ageMillis = cacheResponseAge();
            long freshMillis = computeFreshnessLifetime();
            
            if (requestCaching.maxAgeSeconds() != -1) {
                freshMillis = Math.min(freshMillis, 
                    SECONDS.toMillis(requestCaching.maxAgeSeconds()));
            }
            
            long minFreshMillis = 0;
            if (requestCaching.minFreshSeconds() != -1) {
                minFreshMillis = SECONDS.toMillis(requestCaching.minFreshSeconds());
            }
            
            long maxStaleMillis = 0;
            if (!responseCaching.mustRevalidate() 
                && requestCaching.maxStaleSeconds() != -1) {
                maxStaleMillis = SECONDS.toMillis(requestCaching.maxStaleSeconds());
            }
            
            // 6. 判断缓存是否新鲜
            if (!responseCaching.noCache() && ageMillis + minFreshMillis < freshMillis + maxStaleMillis) {
                Response.Builder builder = cacheResponse.newBuilder();
                if (ageMillis + minFreshMillis >= freshMillis) {
                    builder.addHeader("Warning", "110 - \"Response is stale\"");
                }
                long oneDayMillis = 24 * 60 * 60 * 1000L;
                if (ageMillis > oneDayMillis && isFreshnessLifetimeHeuristic()) {
                    builder.addHeader("Warning", "113 - \"Heuristic expiration\"");
                }
                return new CacheStrategy(null, builder.build());
            }
            
            // 7. 需要验证缓存
            String conditionName;
            String conditionValue;
            if (etag != null) {
                conditionName = "If-None-Match";
                conditionValue = etag;
            } else if (lastModified != null) {
                conditionName = "If-Modified-Since";
                conditionValue = lastModifiedString;
            } else if (servedDate != null) {
                conditionName = "If-Modified-Since";
                conditionValue = servedDateString;
            } else {
                return new CacheStrategy(request, null); // 无条件请求
            }
            
            Headers.Builder conditionalRequestHeaders = request.headers().newBuilder();
            conditionalRequestHeaders.add(conditionName, conditionValue);
            
            Request conditionalRequest = request.newBuilder()
                .headers(conditionalRequestHeaders.build())
                .build();
            return new CacheStrategy(conditionalRequest, cacheResponse);
        }
    }
}
```

------

## 七、设计模式应用

### 1. **建造者模式（Builder）**

java

java

下载

复制



```
// OkHttpClient 配置
public class OkHttpClient implements Cloneable, Call.Factory {
    public static class Builder {
        Dispatcher dispatcher;
        Proxy proxy;
        List<Protocol> protocols;
        // ... 其他配置
        
        public Builder() {
            dispatcher = new Dispatcher();
            protocols = DEFAULT_PROTOCOLS;
            // ... 默认配置
        }
        
        public Builder connectTimeout(long timeout, TimeUnit unit) {
            connectTimeout = timeout;
            connectTimeoutUnit = unit;
            return this;
        }
        
        public OkHttpClient build() {
            return new OkHttpClient(this);
        }
    }
}
```

### 2. **工厂模式（Factory）**

java

java

下载

复制



```
// Call 工厂
public interface Call {
    interface Factory {
        Call newCall(Request request);
    }
}

// OkHttpClient 实现
@Override public Call newCall(Request request) {
    return RealCall.newRealCall(this, request, false /* for web socket */);
}
```

### 3. **责任链模式（Chain of Responsibility）**

java

java

下载

复制



```
// 拦截器链
public interface Interceptor {
    Response intercept(Chain chain) throws IOException;
    
    interface Chain {
        Request request();
        Response proceed(Request request) throws IOException;
    }
}

// 实现
public class RealInterceptorChain implements Interceptor.Chain {
    private final List<Interceptor> interceptors;
    private final int index;
    
    @Override public Response proceed(Request request) throws IOException {
        if (index >= interceptors.size()) throw new AssertionError();
        
        // 调用下一个拦截器
        RealInterceptorChain next = new RealInterceptorChain(
            interceptors, transmitter, exchange, index + 1, request);
        Interceptor interceptor = interceptors.get(index);
        Response response = interceptor.intercept(next);
        
        return response;
    }
}
```

### 4. **观察者模式（Observer）**

java

java

下载

复制



```
// EventListener 监听网络事件
public abstract class EventListener {
    public void callStart(Call call) {}
    public void dnsStart(Call call, String domainName) {}
    public void dnsEnd(Call call, String domainName, List<InetAddress> inetAddressList) {}
    // ... 其他事件
}

// 在关键位置触发事件
class RealCall {
    void notifyCallStart() {
        for (EventListener eventListener : eventListeners) {
            eventListener.callStart(this);
        }
    }
}
```

### 5. **享元模式（Flyweight）**

java

java

下载

复制



```
// ConnectionPool 连接复用
public final class ConnectionPool {
    private final Deque<RealConnection> connections = new ArrayDeque<>();
    
    // 复用连接
    RealConnection get(Address address, Transmitter transmitter, Route route) {
        for (RealConnection connection : connections) {
            if (connection.isEligible(address, route)) {
                transmitter.acquireConnectionNoEvents(connection);
                return connection;
            }
        }
        return null;
    }
}
```

------

## 八、性能优化技巧

### 1. **连接复用**

java

java

下载

复制



```
// 判断连接是否可复用
public boolean isEligible(Address address, Route route) {
    // 1. 连接已分配流数达到上限
    if (allocations.size() >= allocationLimit) return false;
    
    // 2. 非 HTTP/2 连接不能复用给不同的主机
    if (!route.address().url().host().equals(this.route().address().url().host())) {
        return false;
    }
    
    // 3. 必须完全匹配（包括代理、SSL 配置等）
    if (this.route().equals(route)) return true;
    
    // 4. HTTP/2 连接可以跨路由复用（相同主机）
    if (http2Connection != null && routeMatchesAny(route)) return true;
    
    return false;
}
```

### 2. **DNS 优化**

java

java

下载

复制



```
// 异步 DNS 解析
public class Dns {
    public List<InetAddress> lookup(String hostname) throws UnknownHostException {
        try {
            return InetAddress.getAllByName(hostname);
        } catch (NullPointerException e) {
            UnknownHostException unknownHostException = 
                new UnknownHostException("Broken system behaviour");
            unknownHostException.initCause(e);
            throw unknownHostException;
        }
    }
}

// 自定义 DNS 实现（如 HTTP DNS）
public class HttpDns implements Dns {
    private final OkHttpClient httpClient;
    
    @Override
    public List<InetAddress> lookup(String hostname) throws UnknownHostException {
        // 1. 先尝试 HTTP DNS
        try {
            Request request = new Request.Builder()
                .url("https://dns.example.com/resolve?name=" + hostname)
                .build();
            Response response = httpClient.newCall(request).execute();
            List<InetAddress> addresses = parseDnsResponse(response);
            if (!addresses.isEmpty()) {
                return addresses;
            }
        } catch (Exception e) {
            // 失败时回退到系统 DNS
        }
        
        // 2. 系统 DNS 回退
        return SYSTEM.lookup(hostname);
    }
}
```

### 3. **请求体压缩**

java

java

下载

复制



```
// 自动 GZIP 压缩
public final class BridgeInterceptor implements Interceptor {
    @Override public Response intercept(Chain chain) throws IOException {
        Request userRequest = chain.request();
        Request.Builder requestBuilder = userRequest.newBuilder();
        
        // 自动添加 Accept-Encoding: gzip
        if (userRequest.header("Accept-Encoding") == null) {
            requestBuilder.header("Accept-Encoding", "gzip");
        }
        
        // 处理响应时自动解压
        if ("gzip".equalsIgnoreCase(networkResponse.header("Content-Encoding"))
            && HttpHeaders.hasBody(networkResponse)) {
            GzipSource responseBody = new GzipSource(networkResponse.body().source());
            Headers strippedHeaders = networkResponse.headers().newBuilder()
                .removeAll("Content-Encoding")
                .removeAll("Content-Length")
                .build();
            responseBuilder.headers(strippedHeaders);
            responseBuilder.body(new RealResponseBody(
                strippedHeaders, Okio.buffer(responseBody)));
        }
    }
}
```

### 4. **超时控制**

java

java

下载

复制



```
// 多层超时控制
public class RealCall implements Call {
    private void timeoutEnter() {
        long timeout = timeout.timeoutNanos();
        if (timeout != 0L) {
            boolean isNewTimeout = timeoutQueue.scheduleTimeout(this, timeout);
            if (isNewTimeout) {
                timeout.start();
            }
        }
    }
    
    // 读取超时
    private void checkTimeout() {
        if (timeout.isTimedOut()) {
            // 取消请求
            transmitter.timeout();
            // 抛出超时异常
            throw new SocketTimeoutException("timeout");
        }
    }
}
```

------

## 九、源码学习建议

### 1. **学习路径**





复制

```
1. 从使用开始 → 理解基本 API
2. 跟踪同步请求流程 → RealCall.execute()
3. 分析拦截器链 → getResponseWithInterceptorChain()
4. 深入研究关键拦截器 → CacheInterceptor、ConnectInterceptor
5. 理解连接管理 → RealConnection、ConnectionPool
6. 探索 HTTP/2 实现 → Http2Connection、Hpack
7. 学习缓存机制 → Cache、CacheStrategy
8. 研究异步处理 → Dispatcher、AsyncCall
```

### 2. **调试技巧**

kotlin

kotlin

复制

```
// 1. 添加日志拦截器
val client = OkHttpClient.Builder()
    .addInterceptor(HttpLoggingInterceptor().apply {
        level = HttpLoggingInterceptor.Level.BODY
    })
    .build()

// 2. 使用 EventListener 跟踪内部事件
val client = OkHttpClient.Builder()
    .eventListener(object : EventListener() {
        override fun callStart(call: Call) {
            println("Call started: ${call.request().url()}")
        }
        override fun dnsStart(call: Call, domainName: String) {
            println("DNS lookup: $domainName")
        }
        // ... 其他事件
    })
    .build()

// 3. 自定义拦截器分析
class DebugInterceptor : Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response {
        val request = chain.request()
        println("Request: ${request.method()} ${request.url()}")
        
        val startNs = System.nanoTime()
        val response = chain.proceed(request)
        val tookMs = TimeUnit.NANOSECONDS.toMillis(System.nanoTime() - startNs)
        
        println("Response: ${response.code()} in ${tookMs}ms")
        return response
    }
}
```

### 3. **关键断点位置**





复制

```
1. RealCall.execute() - 请求入口
2. Dispatcher.enqueue() - 异步请求调度
3. RealInterceptorChain.proceed() - 拦截器链流转
4. CacheInterceptor.intercept() - 缓存决策
5. ConnectInterceptor.intercept() - 连接建立
6. CallServerInterceptor.intercept() - 网络读写
7. ConnectionPool.get() - 连接获取
8. Http2Connection.newStream() - HTTP/2 流创建
```

### 
