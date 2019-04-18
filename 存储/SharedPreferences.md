# SharedPreferences

适用范围**：**保存少量的数据，且这些数据的格式非常简单：字符串型、基本类型的值。比如应用程序的各种配置信息（如是否打开音效、是否使用震动效果、小游戏的玩家积分等），解锁口 令密码等

​    核心原理**：**保存基于XML文件存储的key-value键值对数据，通常用来存储一些简单的配置信息。通过DDMS的File Explorer面板，展开文件浏览树,很明显SharedPreferences数据总是存储在/data/data/<package name>/shared_prefs目录下。SharedPreferences对象本身只能获取数据而不支持存储和修改,存储修改是通过SharedPreferences.edit()获取的内部接口Editor对象实现。 SharedPreferences本身是一 个接口，程序无法直接创建SharedPreferences实例，只能通过Context提供的getSharedPreferences(String name, int mode)方法来获取SharedPreferences实例，该方法中name表示要操作的xml文件名，第二个参数具体如下：

​                 **Context.MODE_PRIVATE**: 指定该SharedPreferences数据只能被本应用程序读、写。

​                 **Context.MODE_WORLD_READABLE**:  指定该SharedPreferences数据能被其他应用程序读，但不能写。

​                 ***Context.MODE_WORLD_WRITEABLE***:  指定该SharedPreferences数据能被其他应用程序读，写

Editor有如下主要重要方法：

​                 **SharedPreferences.Editor clear():**清空SharedPreferences里所有数据

​                 **SharedPreferences.Editor putXxx(String key , xxx value):** 向SharedPreferences存入指定key对应的数据，其中xxx 可以是boolean,float,int等各种基本类型据

​                 **SharedPreferences.Editor remove():** 删除SharedPreferences中指定key对应的数据项

​                 **boolean commit():** 当Editor编辑完成后，使用该方法提交修改

