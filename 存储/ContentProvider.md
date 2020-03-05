# ContentProvider

ContentProvider一般为存储和获取数据提供统一的接口，可以在不同的应用程序之间共享数据。

之所以使用ContentProvider，主要有以下几个理由：
 1，ContentProvider提供了对底层数据存储方式的抽象。比如下图中，底层使用了SQLite数据库，在用了ContentProvider封装后，即使你把数据库换成MongoDB，也不会对上层数据使用层代码产生影响。

2，Android框架中的一些类需要ContentProvider类型数据。如果你想让你的数据可以使用在如SyncAdapter, Loader, CursorAdapter等类上，那么你就需要为你的数据做一层ContentProvider封装。

3，第三个原因也是最主要的原因，是ContentProvider为应用间的数据交互提供了一个安全的环境。它准许你把自己的应用数据根据需求开放给其他应用进行增、删、改、查，而不用担心直接开放数据库权限而带来的安全问题。



ContentProvider进行增，删，改，查的操作使用ContentResolver

ContentResolver来统一管理与不同ContentProvider间的操作。





### ContentProvider中的URI

ContentProvider中的URI有固定格式：

content://Authority/Path/Id



URI

Authority：

授权信息，用以区别不同的ContentProvider；

Path：

表名，用以区分ContentProvider中不同的数据表；

Id :

区分是哪一条数据