# RxJava

## RxJava基础

### Observable 分类

1. Hot 多个订阅者时，Hot Observable与订阅者们是一对多关系，可以与多个订阅者共享信息
2. Cold  只有观察者订阅了，才开始执行发射数据流的代码。只能是一对一关系，各自事件独立

### Cold Observable转Hot Observable

1. publish 生成ConnectableObservable，调用connect才能真正执行。
2. Subject/Processor，Processor继承自Flowable，支持背压。Subject既是Observable，又是Observer

### Hot Observable转Cold Observable

1. ConnectableObservable.refCount
2. Observable.share

### Flowable (RxJava2 Observable不再支持背压，由Flowable支持阻塞式的背压)

### RxJava2 中其他3种被观察者： Single Completable, Maybe

### Subject和Processor

* Subject分类
  1. AsyncSubject 不论订阅发生时间，只发射最后一个数据
  2. BehaviorSubject 发送订阅之前的一个数据和订阅之后全部数据
  3. ReplaySubject 发射全部数据
  4. PublishSubject 发送订阅之后全部数据
* Processor 



## 创建操作符

* just
* from 
* create
* defer
* range
* interval
* timer
* empty
* error
* never

### Repeat

1. just(1),repeat(3)
2. repeatWhen
3. repeatUntil

### defer 观察者订阅时才撞见Observable，并且为每个观察者创建一个全新的Observable



## 线程操作

### Scheduler种类

1. single 定长为1的线程池
2. newThread
3. computation
4. io
5. trampoline 直接在当前线程运行
6. from 将 Executor转换成一个调度器实例



## 变换和过滤

### 变换

* map
* flatMap, comcatMap, flatMapIterable
* switchMap
* scan
* groupBy
* buffer
* window
* cast

### 过滤

* filter
* takeLast
* last
* lastOrDefault
* takeLastBuffer
* skip
* skipLast
* take
* first, takeFirst
* firstOrDefault
* elementAt
* elementAtOrDefault
* sample, throttleLast
* throttleFirst
* throttleWhenTImeout, debounce
* timeout
* distinct
* distinctUntilChanged
* ofType
* ignoreElements



## 条件和布尔操作符

### 条件操作符

* amb
* defaultIfEmpty
* skipUntil
* skipWhile
* takeUntil
* takeWhile takeWhileWithIndex

### 布尔

* all
* contains
* exists isEmpty
* sequenceEqual



## 合并，连接操作符

### 合并

* startWith
* merge
* mergeDelayError
* zip
* combineLatest
* join groupJoin
* switchOnNext

### 连接

* ConnectableObservable.connect
* Observable.publish
* Observable.replay
* COnnectableObservable.refCount



## 背压 异步场景下，被观察者发送事件速度远快于观察者处理的速度，导致下游Buffer溢出



## Disposable Transformer







