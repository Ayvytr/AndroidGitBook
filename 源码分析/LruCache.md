# LruCache

LruCache是个泛型类，主要算法原理是把最近使用的对象用强引用存储在LinkedHashMap中。当缓存满时，把最近最少使用的对象从内存中移除，并提供了get和put方法来完成缓存的获取和添加操作。



关键原理：LinkedHashMap（数组+双向链表结构）初始化参数accessOrder=true实现访问顺序排序，然后LruCache可方便地删除最近最少使用的对象

```
accessOrder=false: 插入顺序排序
accessOrder=true: 访问顺序排序
```



调用put()时，会在map中添加数据，并调用trimToSize()：判断缓存是否已满。如果满了就用LinkedHashMap的迭代器删除队尾元素，即近期最少访问的元素。当调用get()方法访问缓存对象时，就会调用LinkedHashMap的get)方法获得对应集合元素，同时会更新该元素到队头。

