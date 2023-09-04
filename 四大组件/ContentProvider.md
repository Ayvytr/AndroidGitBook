# [ContentProvider](存储/ContentProvider.md)

外界（包括当前进程的其他组件）无法直接访问 ContentProvider 的，而是需要通过 ContentResolver 来间接访问。这种设计的优点是 **统一管理应用依赖的 ContentProvider，而不需要关心真正操作的 ContentProvider 实现类。**

UriMatcher 是用于自定义 ContentProvider 的工具类，**主要作用是根据 Uri 匹配对应的数据表。**

**ContentObserver 用于监听 ContentProvider 中指定 Uri 标识数据的变化（增 / 删 / 改）**，onChange实现监听逻辑。需要使用registerContentObserver，unregisterContentObserver分别注册和取消注册



## 使用



继承ContentProvider，重写onCreate，getType，增删改查6个方法，实现逻辑。onCreate在主线程启动，增删改查在Binder线程，getType在调用线程。



## 批量操作

bulkInsert

applyBatch



ContentProvider底层数据通信是采用了Binder，而Binder传输限制1Mb，即使超过Binder缓冲区的限制，bulkInsert也不会报TransactionTooLargeException，是因为ContentResolver内部做了RemoteException的捕获消化，并直接return 0，且并没有log输出

