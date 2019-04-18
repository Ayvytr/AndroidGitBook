# SQLite

嵌入式数据库，可通过SQLiteOpenHelper.getWriteableDatabase获取实例，然后进行增删改查操作

**SQLiteOpenHelper**

| 方法名                                                       | 方法描述                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **SQLiteOpenHelper(Context context,String name,SQLiteDatabase.CursorFactory factory,int version)** | 构造方法，其中context 程序上下文环境 即：XXXActivity.this;name :数据库名字;factory:游标工厂，默认为null,即为使用默认工厂;version 数据库版本号 |
| **onCreate(SQLiteDatabase db)**                              | 创建数据库时调用                                             |
| **onUpgrade(SQLiteDatabase db,int oldVersion , int newVersion)** | 版本更新时调用                                               |
| **getReadableDatabase()**                                    | 创建或打开一个只读数据库                                     |
| **getWritableDatabase()**                                    | 创建或打开一个读写数据库                                     |

### 