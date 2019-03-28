# IPC 机制

## Android IPC 简介

IPC是Inter-Process Communication 的缩写，含义为进程间通信或是跨进程通信。

- 线程是CPU 调度的最小单元，是一种有限的系统资源。
- 一个进程可以包含多个线程，进程和线程的的关系是包含和被包含。
- Android 中最有特色的进程间通信就是Binder，同时Android 还支持Socket实现任意两个终端的通信，包括一个设备上的两个进程。

## Android 中的多进程模式

### 开启多进程模式

正常情况下，Android中的多进程是指一个应用中存在多个进程的情况。在Android中使用多进程只有一个方法，那就是给四大组件(Activity、Service、Reciver、ContentProvider)在AndroidMenifest中指定android:process属性。

```xml
<activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <activity android:name="com.hytc.ipc.SecondActivity"
            android:configChanges="screenLayout"
            android:label="@string/app_name"
            android:process=":remote"/>
        <activity android:name="com.hytc.ipc.ThirdActivity"
            android:configChanges="screenLayout"
            android:label="@string/app_name"
            android:process="com.hytc.ipc.remote"/>
```

> - 入口Activity是MainActivity，运行在默认进程中，进程名为包名
> - “: remote”    ":"的含义为在当前的进程名前面附加上当前的包名，该进程为当前应用的私有进程，其他应用的组件不可以和它跑到同一个进程中。
> - “com.hytv.ipc.remote”，是一种完整的命名方式，不会附加包名的信息。该进程属于全局进程，其他应用可以通过ShareUID方式和他跑在同一个进程中。

### 多进程模式的运行机制

使用多进程会造成以下几个方面的问题：

1. 静态成员和单例模式完全失效
2. 线程同步机制完全失效
3. SharePreference的可靠性下降
4. Application会多次创建

## IPC 基础概念介绍

### Serializable 接口

Serializable 是Java 所提供的一个序列化接口，它是一个空接口，为对象提供标准的序列化和反序列化操作。

```java
public class User  implements Serializable{
    public static final long serialVersionUID = 111L;
    private String name;
    private int age;
}
```

> serialVersionUID  用来辅助序列化和反序列化的过程，确定序列化类的版本和反序列化类的版本是否一致。

>- 静态成员变量属于类不属于对象，所以不会参加序列化过程
>- 用transientguan关键字标识的成员变量不参与序列化过程

### Parcelable 接口

Parcelable 是Android 所提供的一个序列化接口，实现这个接口，一个类的对象就可以实现序列化并可以通过Intent 、Binder 传递。

```java
public class User1  implements Parcelable{
    private String name;
    private int age;
    protected User1(Parcel in) {
        name = in.readString();
        age = in.readInt();
    }
    public static final Creator<User1> CREATOR = new Creator<User1>() {
        @Override
        public User1 createFromParcel(Parcel in) {
            return new User1(in);
        }
        @Override
        public User1[] newArray(int size) {
            return new User1[size];
        }
    };
    @Override
    public int describeContents() {
        return 0;
    }
    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeString(name);
        dest.writeInt(age);
    }
}
```

 ### Binder

Binder是Android 中的一个类，实现了IBinder接口。从IPC角度来说，Binder是Android中的一种跨进程通信方式。Android 开发中，Binder主要用在Service 中，包括AIDL和Messenger,其中普通Service中的Binder 不涉及进程间通信，较为简单，Messenger的底层为AIDL。

（有点复杂）

## Android 中的IPC方式

### 使用Bundle

四大组件中的三大组件（Activity、Service、Receiver）都支持在Intent中传递Bundle数据的，由于Bundle实现了Parcelable接口，所以方便在不同的进程间传递。

### 使用文件共享

两个进程通过读/写同一个文件来交换数据。

- 可以是文本文件，也可以是XML文件，读写双方约定好数据格式就可以
- 尽量避免并发读写的操作，使用线程同步
- 不建议在进程间通信中室友SharePreference,有很大几率丢失数据

### 使用Messenger

Messenger可以翻译为信使，通过他可以在不同进程中传递Message对象，在Message中放入我们需要传递的数据。Messenger是一种轻量级的IPC方案，它的底层实现为AIDL。实现一个Messenger的几个步骤：

1. #### 服务端进程

   首先在服务端创建一个Service来处理客户端的连接请求，同时创建一个Handler并通过她来创建一个Messenger对象，然后再Service的onBind 中返回这个Messenger 对象底层的Binder对象。

   ```java
   private final Messenger mMessenger = new Messenger( new MessengerHandler());
       @Nullable
       @Override
       public IBinder onBind(Intent intent) {
           return mMessenger.getBinder();
       }
   ```

2. ####  客户端进程

   首先绑定服务端的Service,绑定成功后用服务端返回的IBinder 对象创建一个Messenger,通过这个Messenger 就可以向服务端发送消息了，发消息的类型为Message。

   ```java
   private Messenger mService;
       private ServiceConnection mConnection = new ServiceConnection() {
           @Override
           public void onServiceConnected(ComponentName name, IBinder service) {
               mService = new Messenger(service);
               Message message = Message.obtain(null, 1);
               Bundle bundle = new Bundle();
               bundle.putString("msg","hello this is a client");
               message.setData(bundle);
               message.replyTo = mGetReplyMessenger;
               try {
                   mService.send(message);
               } catch (RemoteException e) {
                   e.printStackTrace();
               }
           }
           @Override
           public void onServiceDisconnected(ComponentName name) {
           }
       };
   ```

   >如果需要服务端能够回应客户端，就和服务端一样，我们还需要创建一个Handler 并创建一个新的Messenger,并把这个Messenger对象通过Message的replyTo参数传递给服务端，服务端通过这个replyTo参数就可以回应客户端。

### 使用AIDL

1. #### 服务端

   服务端首先要创建一个Service用来监听客户端的连接请求，然后创建一个AIDL文件，将暴露给客户端的接口在这个AIDL文件中声明，最后在Service中实现这个AIDL接口即可。

   ```java
   public class BookManagerService extends Service {
       private CopyOnWriteArrayList<Book> mBookList = new CopyOnWriteArrayList<>();
       private Binder mBinder = new IBookManager.Stub() {
           @Override
           public List<Book> getBookList() throws RemoteException {
               return mBookList;
           }
           @Override
           public void addBook(Book book) throws RemoteException {
               mBookList.add(book);
           }
       };
       @Override
       public void onCreate() {
           super.onCreate();
           mBookList.add(new Book("android","柯南"));
           mBookList.add(new Book("ios","柯南二号"));
       }
       @Nullable
       @Override
       public IBinder onBind(Intent intent) {
           return mBinder;
       }
   }
   ```

2. #### 客户端

   绑定服务端的Service，将服务端返回的Binder对象转成AIDL接口所属类型，接着就可以调用AIDL中的方法了。

   ```java
   private ServiceConnection mConnection = new ServiceConnection() {
           @Override
           public void onServiceConnected(ComponentName name, IBinder service) {
               IBookManager bookManager = IBookManager.Stub.asInterface(service);
               try {
                   List<Book> bookList = bookManager.getBookList();
                   Log.i(TAG, "onServiceConnected: "+bookList.getClass().getCanonicalName());
                   Log.i(TAG, "onServiceConnected: "+bookList.toString());
               } catch (RemoteException e) {
                   e.printStackTrace();
               }
           }
           @Override
           public void onServiceDisconnected(ComponentName name) {

           }
       };
   ```

3. AIDL 接口的创建

   ````
   //AIDL文件
   package com.hytc.aidl;
   import com.hytc.aidl.Book;
   interface IBookManager {
       List<Book> getBookList();
       void addBook(in Book book);
   }
   ````

   AIDL支持的数据类型

   - 基本数据类型
   - String和CharSquence
   - List(只支持ArrayList)
   - Map(只支持HashMap)
   - Parcelable:所有实现该接口的对象
   - AIDL: 所有的AIDL接口本身也可以在AIDL文件中使用

   如果AIDL文件中用到了自定义的Parcelable对象，必须新建一个和他同名的AIDL文件。

   ```java
   // Book.aidl
   package com.hytc.aidl;
   parcelable Book;
   ```

   **注意 : Make Project**


### IPC 方式的优缺点和使用场景

|       名称        |                    优点                    |                    缺点                    |                适用场景                |
| :-------------: | :--------------------------------------: | :--------------------------------------: | :--------------------------------: |
|     Bundle      |                   简单易用                   |            只能传输Bundle支持的数据类型             |             四大组件的进程间通信             |
|      文件共享       |                   简单易用                   |         不适合高并发场景，并且无法做到进程间的即时通信          |      无并发访问情形，交换简单的数据，实时性不高的场景      |
|      AIDL       |          功能强大，支持一对多并发通信，支持实时通信           |             使用稍复杂，需要处理好线程同步              |            一对多通信且有RPC需求            |
|    Messenger    |          功能一般，支持一对多的串行通信，支持实时通信          | 不能很好的处理高并发情形，不支持RPC，数据通过Message进行传输，因此只能传输Bundle支持的数据类型 | 低并发的一对多即时通信，无RPC需求，或者无需要返回结果的RPC需求 |
| ContentProvider | 在数据源访问方面功能强大，支持一对多并发数据共享，可通过Call方法扩展其他操作 |       可以理解为受约束的AIDL，主要提供数据源的CRUD操作       |              一对多的数据共享              |
|     Socket      |       功能强大，可以通过网络传输字节流，支持一对多并发实时通信       |           实现细节稍微有点繁琐，不支持直接的RPC           |               网络数据交换               |

