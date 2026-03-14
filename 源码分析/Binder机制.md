# Binder机制

```
// 1. 一次 Binder 调用流程
Client Process → Binder Driver → Server Process
    ↓               ↓              ↓
Proxy          跨进程通信      Stub/Impl

// 2. 关键源码类
IBinder           // 通信接口
Parcel            // 数据容器
ServiceManager    // 服务管家
BpBinder/BnBinder // 代理/本地对象
```

## 高频问题

- 

  **Binder 为什么高效？**

  

  ```
  // 内核驱动层：一次拷贝 + mmap内存映射
  static long binder_ioctl(struct file *filp, unsigned int cmd, unsigned long arg) {
      switch (cmd) {
          case BINDER_WRITE_READ:
              // 通过 copy_from_user 从用户空间读取数据
              // 通过 mmap 建立内存映射，减少拷贝次数
              break;
      }
  }
  ```

- 

  **Binder 与 Socket/AIDL 的关系？**

  - 

    AIDL 是 Binder 的接口描述语言

  - 

    Socket 是通用 IPC，Binder 为 Android 优化

- 

  **如何突破 Binder 传输限制？（1MB）**

  

  

  ```
  // 解决方案
  // 1. 分段传输
  // 2. 使用 ashmem 共享内存
  // 3. 文件描述符传递
  ```







## AIDL

- AIDL 是 Binder 的接口描述语言
- Socket 是通用 IPC，Binder 为 Android 优化



步骤：

1. 定义AIDL接口
2. 实现接口
3. 在Service中返回实现类的实例
4. 客户端绑定服务并调用远程方法





## AIDL 基础使用

### 1. 创建 AIDL 文件

java

java

下载

复制



```
// IBookManager.aidl
package com.example.aidl;

// 声明非默认支持的数据类型
import com.example.aidl.Book;

// 定义接口
interface IBookManager {
    // 基本类型参数
    int getBookCount();
    
    // 自定义对象参数（必须实现 Parcelable）
    Book getBook(in int bookId);
    
    // 输入输出参数标识
    void addBook(in Book book);
    void removeBook(out Book book);
    void updateBook(inout Book book);
    
    // 带返回值的复杂操作
    List<Book> getAllBooks();
    
    // 回调接口
    void registerListener(IOnNewBookListener listener);
    void unregisterListener(IOnNewBookListener listener);
    
    // 异步方法（oneway）
    oneway void clearAllBooks();
}
```

### 2. 定义 Parcelable 对象





```
// Book.java
package com.example.aidl;

import android.os.Parcel;
import android.os.Parcelable;

public class Book implements Parcelable {
    private int id;
    private String name;
    private double price;
    
    // 构造方法
    public Book() {}
    public Book(int id, String name, double price) {
        this.id = id;
        this.name = name;
        this.price = price;
    }
    
    // Parcelable 实现
    protected Book(Parcel in) {
        id = in.readInt();
        name = in.readString();
        price = in.readDouble();
    }
    
    public static final Creator<Book> CREATOR = new Creator<Book>() {
        @Override
        public Book createFromParcel(Parcel in) {
            return new Book(in);
        }
        
        @Override
        public Book[] newArray(int size) {
            return new Book[size];
        }
    };
    
    @Override
    public int describeContents() {
        return 0;
    }
    
    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeInt(id);
        dest.writeString(name);
        dest.writeDouble(price);
    }
    
    // 必须为 AIDL 创建对应的 .aidl 文件
    // Book.aidl
    // parcelable Book;
}
```



```
// Book.aidl
package com.example.aidl;
parcelable Book;
```

### 3. 定义回调接口





```
// IOnNewBookListener.aidl
package com.example.aidl;

import com.example.aidl.Book;

// 回调接口
interface IOnNewBookListener {
    void onNewBookArrived(in Book newBook);
}
```



## 服务端实现

### 1. Service 实现



```
// BookManagerService.java
public class BookManagerService extends Service {
    private static final String TAG = "BookManagerService";
    
    // CopyOnWriteArrayList 支持并发读写
    private CopyOnWriteArrayList<Book> mBookList = new CopyOnWriteArrayList<>();
    
    // 回调监听器列表，使用 RemoteCallbackList 管理
    private RemoteCallbackList<IOnNewBookListener> mListenerList = new RemoteCallbackList<>();
    
    // 死亡代理
    private IBinder.DeathRecipient mDeathRecipient = new IBinder.DeathRecipient() {
        @Override
        public void binderDied() {
            Log.w(TAG, "Binder died!");
        }
    };
    
    // Binder 实现类
    private Binder mBinder = new IBookManager.Stub() {
        @Override
        public int getBookCount() {
            return mBookList.size();
        }
        
        @Override
        public Book getBook(int bookId) {
            for (Book book : mBookList) {
                if (book.getId() == bookId) {
                    return book;
                }
            }
            return null;
        }
        
        @Override
        public void addBook(Book book) {
            mBookList.add(book);
            
            // 通知所有监听器
            final int N = mListenerList.beginBroadcast();
            for (int i = 0; i < N; i++) {
                IOnNewBookListener listener = mListenerList.getBroadcastItem(i);
                if (listener != null) {
                    try {
                        listener.onNewBookArrived(book);
                    } catch (RemoteException e) {
                        e.printStackTrace();
                    }
                }
            }
            mListenerList.finishBroadcast();
        }
        
        @Override
        public void removeBook(Book book) {
            mBookList.remove(book);
        }
        
        @Override
        public void updateBook(Book book) {
            // 更新书籍信息
        }
        
        @Override
        public List<Book> getAllBooks() {
            return mBookList;
        }
        
        @Override
        public void registerListener(IOnNewBookListener listener) {
            mListenerList.register(listener);
            
            // 绑定死亡代理
            try {
                listener.asBinder().linkToDeath(mDeathRecipient, 0);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
            
            Log.d(TAG, "registerListener, current size:" + mListenerList.getRegisteredCallbackCount());
        }
        
        @Override
        public void unregisterListener(IOnNewBookListener listener) {
            mListenerList.unregister(listener);
            Log.d(TAG, "unregisterListener, current size:" + mListenerList.getRegisteredCallbackCount());
        }
        
        @Override
        public void clearAllBooks() {
            mBookList.clear();
            Log.d(TAG, "All books cleared");
        }
    };
    
    @Override
    public void onCreate() {
        super.onCreate();
        
        // 初始化数据
        mBookList.add(new Book(1, "Android开发艺术探索", 58.8));
        mBookList.add(new Book(2, "第一行代码", 79.0));
    }
    
    @Override
    public IBinder onBind(Intent intent) {
        return mBinder;
    }
    
    @Override
    public void onDestroy() {
        super.onDestroy();
        // 清理资源
        mListenerList.kill();
    }
}
```

### 2. AndroidManifest.xml 配置



```
<service
    android:name=".BookManagerService"
    android:enabled="true"
    android:exported="true"
    android:process=":remote">  <!-- 运行在独立进程 -->
    
    <intent-filter>
        <action android:name="com.example.aidl.BOOK_SERVICE" />
        <category android:name="android.intent.category.DEFAULT" />
    </intent-filter>
</service>
```



## 客户端实现

### 1. 连接服务

java

java

下载

复制



```
// MainActivity.java
public class MainActivity extends AppCompatActivity {
    private static final String TAG = "MainActivity";
    
    private IBookManager mBookManager;
    private boolean mBound = false;
    
    // 服务连接回调
    private ServiceConnection mConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            mBookManager = IBookManager.Stub.asInterface(service);
            mBound = true;
            
            // 绑定死亡代理
            try {
                service.linkToDeath(mDeathRecipient, 0);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
            
            // 注册监听器
            try {
                mBookManager.registerListener(mOnNewBookListener);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
            
            Log.d(TAG, "Service connected");
            fetchBooks();
        }
        
        @Override
        public void onServiceDisconnected(ComponentName name) {
            mBound = false;
            Log.e(TAG, "Service disconnected");
        }
    };
    
    // 死亡代理
    private IBinder.DeathRecipient mDeathRecipient = new IBinder.DeathRecipient() {
        @Override
        public void binderDied() {
            Log.w(TAG, "Binder died, reconnecting...");
            
            // 尝试重连
            if (mBookManager != null) {
                mBookManager.asBinder().unlinkToDeath(mDeathRecipient, 0);
                mBookManager = null;
            }
            
            // 重新绑定
            bindBookService();
        }
    };
    
    // 回调监听器实现
    private IOnNewBookListener mOnNewBookListener = new IOnNewBookListener.Stub() {
        @Override
        public void onNewBookArrived(Book newBook) throws RemoteException {
            // 注意：这个方法在 Binder 线程池中执行，需要切换到主线程更新UI
            runOnUiThread(() -> {
                Toast.makeText(MainActivity.this, 
                    "新书到达：" + newBook.getName(), 
                    Toast.LENGTH_SHORT).show();
            });
        }
    };
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        // 绑定服务
        bindBookService();
        
        // 初始化UI
        Button btnGetBooks = findViewById(R.id.btn_get_books);
        Button btnAddBook = findViewById(R.id.btn_add_book);
        
        btnGetBooks.setOnClickListener(v -> fetchBooks());
        btnAddBook.setOnClickListener(v -> addNewBook());
    }
    
    private void bindBookService() {
        Intent intent = new Intent();
        intent.setAction("com.example.aidl.BOOK_SERVICE");
        intent.setPackage(getPackageName());  // Android 5.0+ 需要显式声明
        intent.setComponent(new ComponentName("com.example.aidl", 
            "com.example.aidl.BookManagerService"));
        
        bindService(intent, mConnection, Context.BIND_AUTO_CREATE);
    }
    
    private void fetchBooks() {
        if (!mBound || mBookManager == null) {
            Toast.makeText(this, "服务未连接", Toast.LENGTH_SHORT).show();
            return;
        }
        
        // 异步调用，避免阻塞UI线程
        new Thread(() -> {
            try {
                List<Book> books = mBookManager.getAllBooks();
                
                // 切换到主线程更新UI
                runOnUiThread(() -> {
                    // 更新列表显示
                    updateBookList(books);
                });
            } catch (RemoteException e) {
                e.printStackTrace();
                runOnUiThread(() -> {
                    Toast.makeText(MainActivity.this, 
                        "获取书籍失败: " + e.getMessage(), 
                        Toast.LENGTH_SHORT).show();
                });
            }
        }).start();
    }
    
    private void addNewBook() {
        if (!mBound || mBookManager == null) {
            Toast.makeText(this, "服务未连接", Toast.LENGTH_SHORT).show();
            return;
        }
        
        new Thread(() -> {
            try {
                int newId = mBookManager.getBookCount() + 1;
                Book newBook = new Book(newId, "新书" + newId, 50.0);
                mBookManager.addBook(newBook);
                
                runOnUiThread(() -> {
                    Toast.makeText(MainActivity.this, 
                        "添加成功", 
                        Toast.LENGTH_SHORT).show();
                });
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }).start();
    }
    
    private void updateBookList(List<Book> books) {
        // 更新UI显示
        StringBuilder sb = new StringBuilder();
        for (Book book : books) {
            sb.append(book.getName()).append("\n");
        }
        TextView tvBooks = findViewById(R.id.tv_books);
        tvBooks.setText(sb.toString());
    }
    
    @Override
    protected void onDestroy() {
        super.onDestroy();
        
        // 解绑服务和监听器
        if (mBound) {
            if (mBookManager != null && mOnNewBookListener != null) {
                try {
                    mBookManager.unregisterListener(mOnNewBookListener);
                } catch (RemoteException e) {
                    e.printStackTrace();
                }
            }
            unbindService(mConnection);
            mBound = false;
        }
    }
}
```



## 高级特性

### 1. 权限验证



```
// 服务端添加权限验证
public class BookManagerService extends Service {
    
    @Override
    public IBinder onBind(Intent intent) {
        // 验证权限
        int check = checkCallingOrSelfPermission("com.example.permission.ACCESS_BOOK_SERVICE");
        if (check == PackageManager.PERMISSION_DENIED) {
            Log.e(TAG, "Permission denied");
            return null;
        }
        
        // 验证包名
        String[] packages = getPackageManager().getPackagesForUid(getCallingUid());
        if (packages != null && packages.length > 0) {
            String packageName = packages[0];
            if (!packageName.startsWith("com.example")) {
                Log.e(TAG, "Package not allowed: " + packageName);
                return null;
            }
        }
        
        return mBinder;
    }
}
```

### 2. 异步调用（oneway）





```
// 在 AIDL 中使用 oneway
interface IBookManager {
    // 异步调用，不阻塞客户端
    oneway void clearAllBooks();
    
    // 带回调的异步方法
    void asyncGetBooks(IBookListCallback callback);
}

// 回调接口
interface IBookListCallback {
    oneway void onBooksReceived(in List<Book> books);
    oneway void onError(in String error);
}
```

### 3. 定向 Tag 详解





```
// in: 数据从客户端流向服务端
void addBook(in Book book);  // 客户端发送副本，服务端修改不影响客户端

// out: 数据从服务端流向客户端
void getNewBook(out Book book);  // 服务端填充数据，客户端接收

// inout: 双向传递
void updateBook(inout Book book);  // 客户端发送，服务端修改，客户端接收修改

// 默认是 in，但建议显式声明
```





## 简化版 Messenger 替代方案

如果不需要复杂接口，可以使用 Messenger：

java

java

下载

复制



```
// 服务端
public class MessengerService extends Service {
    private static final String TAG = "MessengerService";
    private static final int MSG_ADD_BOOK = 1;
    
    // 处理客户端消息
    class IncomingHandler extends Handler {
        @Override
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case MSG_ADD_BOOK:
                    // 处理添加书籍
                    Bundle data = msg.getData();
                    Book book = data.getParcelable("book");
                    addBook(book);
                    
                    // 回复客户端
                    Messenger client = msg.replyTo;
                    Message reply = Message.obtain(null, MSG_ADD_BOOK);
                    Bundle replyData = new Bundle();
                    replyData.putString("result", "success");
                    reply.setData(replyData);
                    
                    try {
                        client.send(reply);
                    } catch (RemoteException e) {
                        e.printStackTrace();
                    }
                    break;
                default:
                    super.handleMessage(msg);
            }
        }
    }
    
    final Messenger mMessenger = new Messenger(new IncomingHandler());
    
    @Override
    public IBinder onBind(Intent intent) {
        return mMessenger.getBinder();
    }
}

// 客户端
public class MessengerActivity extends AppCompatActivity {
    private Messenger mService;
    private boolean mBound;
    
    // 接收服务端回复
    private Handler mClientHandler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case MSG_ADD_BOOK:
                    String result = msg.getData().getString("result");
                    Toast.makeText(MessengerActivity.this, 
                        "添加结果: " + result, 
                        Toast.LENGTH_SHORT).show();
                    break;
            }
        }
    };
    
    private Messenger mClientMessenger = new Messenger(mClientHandler);
    
    private ServiceConnection mConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            mService = new Messenger(service);
            mBound = true;
        }
        
        @Override
        public void onServiceDisconnected(ComponentName name) {
            mBound = false;
        }
    };
    
    private void sendMessage() {
        if (!mBound) return;
        
        Message msg = Message.obtain(null, MSG_ADD_BOOK);
        Bundle data = new Bundle();
        data.putParcelable("book", new Book(1, "新书", 50.0));
        msg.setData(data);
        msg.replyTo = mClientMessenger;  // 设置回复 Messenger
        
        try {
            mService.send(msg);
        } catch (RemoteException e) {
            e.printStackTrace();
        }
    }
}
```



## 最佳实践总结

### 1. 项目结构



```
app/
├── src/main/
│   ├── aidl/
│   │   └── com/example/aidl/
│   │       ├── IBookManager.aidl
│   │       ├── Book.aidl
│   │       └── IOnNewBookListener.aidl
│   ├── java/
│   │   └── com/example/aidl/
│   │       ├── Book.java      # Parcelable 实现
│   │       ├── BookManagerService.java
│   │       └── MainActivity.java
│   └── AndroidManifest.xml
```

### 2. 编码规范



```
// 1. 使用 try-catch 处理 RemoteException
try {
    mBookManager.someMethod();
} catch (RemoteException e) {
    e.printStackTrace();
    // 处理连接异常
}

// 2. 在子线程中进行耗时 IPC 调用
new Thread(() -> {
    try {
        List<Book> books = mBookManager.getAllBooks();
        runOnUiThread(() -> updateUI(books));
    } catch (RemoteException e) {
        e.printStackTrace();
    }
}).start();

// 3. 使用死亡代理自动重连
service.linkToDeath(mDeathRecipient, 0);

// 4. 合理使用定向 Tag
// - 只读用 in
// - 只写用 out  
// - 读写用 inout
// - 避免不必要的 inout
```

### 3. 调试技巧





```
// 添加日志
private static final String TAG = "AIDLDemo";

// 检查 Binder 是否存活
if (mBookManager != null && mBookManager.asBinder().isBinderAlive()) {
    // 安全调用
}

// 使用 adb 命令调试
// 查看 Binder 引用计数
adb shell dumpsys activity service com.example.aidl/.BookManagerService

// 查看 Binder 事务
adb shell cat /sys/kernel/debug/binder/stats
```