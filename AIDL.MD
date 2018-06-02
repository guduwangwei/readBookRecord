####`IPC`

###### `Linux内存空间`

Linux 分为 内核空间和 用户空间，用户空间和用户空间和相互隔离的，如果两个用户空间要通信 。就必须要借助* 内核空间中转。(数据需要copy两次)

![](https://pic4.zhimg.com/80/v2-38e2ea1d22660b237e17d2a7f298f3d6_hd.jpg)



* 动态内核加载和内存映射

   动态内核加载机制：模块具有独立功能的程序，可以被单独编译，但是不能单独运行。它在运行时被链接到内核作为内核的一部分运行。这样，Android系统就可以动态天津爱一个内核模块到内核空间，用户进程之间通过这个内核模块作为通信的桥梁

  > Android系统中 ，这个内核模块就Binder驱动

  内存映射： 

  ​     将用户空间的一块内存 映射到内核空间 ，这样 用户空间的更改直接反应到内核空间， 内核空间的更改也可反应到用户空间

----



Binder IPC 正是基于内存映射（mmap）来实现的，但是 mmap() 通常是用在有物理介质的文件系统上的。

比如进程中的用户区域是不能直接和物理设备打交道的，如果想要把磁盘上的数据读取到进程的用户区域，需要两次拷贝（磁盘-->内核空间-->用户空间）；通常在这种场景下 mmap() 就能发挥作用，通过在物理介质和用户空间之间建立映射，减少数据的拷贝次数，用内存读写取代I/O读写，提高文件读取效率。

而 Binder 并不存在物理介质，因此 Binder 驱动使用 mmap() 并不是为了在物理介质和用户空间之间建立映射，而是用来在内核空间创建数据接收的缓存空间。

一次完整的 Binder IPC 通信过程通常是这样：

1. 首先 Binder 驱动在内核空间创建一个数据接收缓存区；
2. 接着在内核空间开辟一块内核缓存区，建立**内核缓存区**和**内核中数据接收缓存区**之间的映射关系，以及**内核中数据接收缓存区**和**接收进程用户空间地址**的映射关系；
3. 发送方进程通过系统调用 copy*from*user() 将数据 copy 到内核中的**内核缓存区**，由于内核缓存区和接收进程的用户空间存在内存映射，因此也就相当于把数据发送到了接收进程的用户空间，这样便完成了一次进程间的通信。

如下图：

![](https://pic4.zhimg.com/80/v2-cbd7d2befbed12d4c8896f236df96dbf_hd.jpg)

------



![流](https://github.com/guduwangwei/learn_framework/blob/master/pic/WX20180506-220938@2x.png?raw=true)

#### AIDL

*binder驱动*：任意服务端`Binder`对象被创建时。同时会在`Binder`驱动中创建一个`mRemote`对象。该对象的类型也是`Binder`类

#### `BookManagerService`

````java
package com.example.wangwei.aidldemo;

import android.app.Service;
import android.content.Intent;
import android.os.Binder;
import android.os.IBinder;
import android.os.RemoteException;
import android.support.annotation.Nullable;

import java.util.ArrayList;
import java.util.List;

/**
 * Created by wangwei on 2018/5/6.
 服务端
 */

public class BookManagerService extends Service {

    private ArrayList<Book> mBookList = new ArrayList<>();
    private Binder mBinder = new BookManager.Stub() {
      

        @Override
        public Book addBookIn(Book book) throws RemoteException {
            book.setPrice(2333);
            mBookList.add(book);
            return book;
        }

        @Override
        public Book addBookOut(Book book) throws RemoteException {
            book.setPrice(2333);
            mBookList.add(book);
            return book;

        }

        @Override
        public Book addBookInout(Book book) throws RemoteException {
            book.setPrice(2333);
            mBookList.add(book);
            return book;

        }
    };


    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return mBinder;
    }

    @Override
    public void onCreate() {
        super.onCreate();
        mBookList.add(new Book("book1", 1));
        mBookList.add(new Book("book2", 2));

    }
}

````

注册

 <service android:name=".BookManagerService" android:process="com.wangwei.exmaple.BookManagerService">

##### `Book.aidl`

````
package com.example.wangwei.aidldemo;

// Declare any non-default types here with import statements

parcelable Book;

````

#### `BookManager.aidl`

````
// BookManager.aidl
package com.example.wangwei.aidldemo;

// Declare any non-default types here with import statements
import com.example.wangwei.aidldemo.Book;
interface BookManager {

  List<Book> getBooks();
  Book getBook();

  int getBookCount();
  Book addBookIn(in Book book);
  Book addBookOut(out Book book);
  Book addBookInout(inout Book book);
}

````

#### `MainActivity.java`

````java
    
    Intent intent = new Intent(this,BookManagerService.class);
    ServiceConnection serviceConnection = new ServiceConnection() {
            @Override
            public void onServiceConnected(ComponentName name, IBinder service) {
                //服务端对象
                bookManagerService = BookManager.Stub.asInterface(service);
            }

            @Override
            public void onServiceDisconnected(ComponentName name) {
                Log.i("ManActivity", "connect");
            }
        };
 // 通过 bookManagerService就可以调用服务端的方法了
````



#### 原理分析 

###### 自动生成的java类

````java
/*
 * This file is auto-generated.  DO NOT MODIFY.
 * Original file: /Users/wangwei/Downloads/studioPro/AIDLDemo/app/src/main/aidl/com/example/wangwei/aidldemo/BookManager.aidl
 */
package com.example.wangwei.aidldemo;

public interface BookManager extends android.os.IInterface {
    /**
     * Local-side IPC implementation stub class.
     */
    public static abstract class Stub extends android.os.Binder implements com.example.wangwei.aidldemo.BookManager {
        private static final java.lang.String DESCRIPTOR = "com.example.wangwei.aidldemo.BookManager";

        /**
         * Construct the stub at attach it to the interface.
         */
        public Stub() {
            this.attachInterface(this, DESCRIPTOR);
        }

        /**
         * Cast an IBinder object into an com.example.wangwei.aidldemo.BookManager interface,
         * generating a proxy if needed.
         */
        public static com.example.wangwei.aidldemo.BookManager asInterface(android.os.IBinder obj) {
            if ((obj == null)) {
                return null;
            }
            android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
            if (((iin != null) && (iin instanceof com.example.wangwei.aidldemo.BookManager))) {
                return ((com.example.wangwei.aidldemo.BookManager) iin);
            }
            return new com.example.wangwei.aidldemo.BookManager.Stub.Proxy(obj);
        }

        @Override
        public android.os.IBinder asBinder() {
            return this;
        }

        @Override
        public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException {
            switch (code) {
                case INTERFACE_TRANSACTION: {
                    reply.writeString(DESCRIPTOR);
                    return true;
                }
                case TRANSACTION_addBookIn: {
                    data.enforceInterface(DESCRIPTOR);
                    com.example.wangwei.aidldemo.Book _arg0;
                    if ((0 != data.readInt())) {
                        _arg0 = com.example.wangwei.aidldemo.Book.CREATOR.createFromParcel(data);
                    } else {
                        _arg0 = null;
                    }
                    com.example.wangwei.aidldemo.Book _result = this.addBookIn(_arg0);
                    reply.writeNoException();
                    if ((_result != null)) {
                        reply.writeInt(1);
                        _result.writeToParcel(reply, android.os.Parcelable.PARCELABLE_WRITE_RETURN_VALUE);
                    } else {
                        reply.writeInt(0);
                    }
                    return true;
                }
                case TRANSACTION_addBookOut: {
                    data.enforceInterface(DESCRIPTOR);
                    com.example.wangwei.aidldemo.Book _arg0;
                    _arg0 = new com.example.wangwei.aidldemo.Book();
                    com.example.wangwei.aidldemo.Book _result = this.addBookOut(_arg0);
                    reply.writeNoException();
                    if ((_result != null)) {
                        reply.writeInt(1);
                        _result.writeToParcel(reply, android.os.Parcelable.PARCELABLE_WRITE_RETURN_VALUE);
                    } else {
                        reply.writeInt(0);
                    }
                    if ((_arg0 != null)) {
                        reply.writeInt(1);
                        _arg0.writeToParcel(reply, android.os.Parcelable.PARCELABLE_WRITE_RETURN_VALUE);
                    } else {
                        reply.writeInt(0);
                    }
                    return true;
                }
                case TRANSACTION_addBookInout: {
                    data.enforceInterface(DESCRIPTOR);
                    com.example.wangwei.aidldemo.Book _arg0;
                    if ((0 != data.readInt())) {
                        _arg0 = com.example.wangwei.aidldemo.Book.CREATOR.createFromParcel(data);
                    } else {
                        _arg0 = null;
                    }
                    com.example.wangwei.aidldemo.Book _result = this.addBookInout(_arg0);
                    reply.writeNoException();
                    if ((_result != null)) {
                        reply.writeInt(1);
                        _result.writeToParcel(reply, android.os.Parcelable.PARCELABLE_WRITE_RETURN_VALUE);
                    } else {
                        reply.writeInt(0);
                    }
                    if ((_arg0 != null)) {
                        reply.writeInt(1);
                        _arg0.writeToParcel(reply, android.os.Parcelable.PARCELABLE_WRITE_RETURN_VALUE);
                    } else {
                        reply.writeInt(0);
                    }
                    return true;
                }
            }
            return super.onTransact(code, data, reply, flags);
        }

        private static class Proxy implements com.example.wangwei.aidldemo.BookManager {
            private android.os.IBinder mRemote;

            Proxy(android.os.IBinder remote) {
                mRemote = remote;
            }

            @Override
            public android.os.IBinder asBinder() {
                return mRemote;
            }

            public java.lang.String getInterfaceDescriptor() {
                return DESCRIPTOR;
            }

            @Override
            public com.example.wangwei.aidldemo.Book addBookIn(com.example.wangwei.aidldemo.Book book) throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                com.example.wangwei.aidldemo.Book _result;
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    if ((book != null)) {
                        _data.writeInt(1);
                        book.writeToParcel(_data, 0);
                    } else {
                        _data.writeInt(0);
                    }
                    mRemote.transact(Stub.TRANSACTION_addBookIn, _data, _reply, 0);
                    _reply.readException();
                    if ((0 != _reply.readInt())) {
                        _result = com.example.wangwei.aidldemo.Book.CREATOR.createFromParcel(_reply);
                    } else {
                        _result = null;
                    }
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
                return _result;
            }

            @Override
            public com.example.wangwei.aidldemo.Book addBookOut(com.example.wangwei.aidldemo.Book book) throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                com.example.wangwei.aidldemo.Book _result;
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    mRemote.transact(Stub.TRANSACTION_addBookOut, _data, _reply, 0);
                    _reply.readException();
                    if ((0 != _reply.readInt())) {
                        _result = com.example.wangwei.aidldemo.Book.CREATOR.createFromParcel(_reply);
                    } else {
                        _result = null;
                    }
                    if ((0 != _reply.readInt())) {
                        book.readFromParcel(_reply);
                    }
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
                return _result;
            }

            @Override
            public com.example.wangwei.aidldemo.Book addBookInout(com.example.wangwei.aidldemo.Book book) throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                com.example.wangwei.aidldemo.Book _result;
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    if ((book != null)) {
                        _data.writeInt(1);
                        book.writeToParcel(_data, 0);
                    } else {
                        _data.writeInt(0);
                    }
                    mRemote.transact(Stub.TRANSACTION_addBookInout, _data, _reply, 0);
                    _reply.readException();
                    if ((0 != _reply.readInt())) {
                        _result = com.example.wangwei.aidldemo.Book.CREATOR.createFromParcel(_reply);
                    } else {
                        _result = null;
                    }
                    if ((0 != _reply.readInt())) {
                        book.readFromParcel(_reply);
                    }
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
                return _result;
            }
        }

        static final int TRANSACTION_addBookIn = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
        static final int TRANSACTION_addBookOut = (android.os.IBinder.FIRST_CALL_TRANSACTION + 1);
        static final int TRANSACTION_addBookInout = (android.os.IBinder.FIRST_CALL_TRANSACTION + 2);
    }

    public com.example.wangwei.aidldemo.Book addBookIn(com.example.wangwei.aidldemo.Book book) throws android.os.RemoteException;

    public com.example.wangwei.aidldemo.Book addBookOut(com.example.wangwei.aidldemo.Book book) throws android.os.RemoteException;

    public com.example.wangwei.aidldemo.Book addBookInout(com.example.wangwei.aidldemo.Book book) throws android.os.RemoteException;
}

````







