# ICP机制
IPC是Inter-Process Communication的缩写，进程间通讯。任何操作系统都有IPC。

- 进程：一个执行单元，PC和移动设备上指一个程序或者应用，一个进程包含一个或多个线程
- 线程：CPU调度的最小单元，有限的系统资源

在Android中最有特色的进程间通信方式是Binder。

Socket也可以实现两个终端的通信，同一个设备上的两个进程也可以通过Socket通信。

多进程情况：

- 一个应用因为自身原因需要采用多进程模式实现
- 两个应用之间通信

#Andorid中的多进程模式

单一应用指定多今晨，只有一种方式：给四大组件指定 android：process属性。

还有非常规的方法多进程方法，通过JNI在native层去fork一个新的进程。这是特殊情况。

指定process属性有两种方式：

- 不加包名有一个**“:”**号如 **“:remote1”**，产生一个进程名为 com.longge.test：remote1，这种命名方式表示该进程属于该应用的私有进程，其它应用和组件不可以和它跑在同一个进程中
- 完整的命名方式，如com.lomgge.test.remote2，该方式创建的进程属于全局进程，其它应用可以通过ShareUID的方式和它跑在同一个进程中。

Android系统为每个应用分配一个唯一的UID，具有相同UID的应用才能共享数据。
> 两个应用通过ShareUID跑在同一个进程中是有要求的，需要这两个应用具有相同的ShareUID并且签名相同，在这种情况下，它们可以互相访问对方的私有数据，比如data目录、组件信息等，不管它们是否跑在同一个进程中。如果跑在同一个进程中，除了共享data、组件信息还可以共享内存数据，它们看起来就像是同一个应用的两个部分。

所有运行在不同进程中的四大组件，只要它们之间需要通过内存来共享数据，都会共享失败，这也是多进程所带来的主要影响。

多进程会造成的问题：

- 静态成员和单例模式完全失效
- 线程同步机制完全失效
- SharedPreference的可靠性下降，SP不支持两个进程同时去执行写操作，否则会导致一定几率的数据丢失，因为SP底层通过读写XML来实现，并发写显然可能出问题，甚至并发/读写都可能出问题。
- Application会多次创建，一个组件跑在新的进程的时候，系统要在创建新的进程的同时分配独立的虚拟机，所以这就是重新启动一个应用的过程。

同一个应用间的多进程相当于两个不同应用之间采用了SharedUID的模式。

跨进程通讯的方式：

- 通过Intent来传递数据，共享文件和SharedPreference
- 基于Binder的Messenger和AIDL
- Socket

# IPC基础概念介绍

## Serializable接口
是java提供的序列化接口，空接口，为对象提供标准的序列化和反序列化

- 静态成员变量属于类，不属于对象，不会参与序列化
- transient关键字编辑的成员变量不参与序列化
- 开销大，序列化和反序列化都需要用到IO操作，实现起来简单



## Parcelable 接口
只要实现这个序列化接口，一个类的对象就可以通过实现序列化并可以通过Intent和Binder传递

Intent、Bundle、Bitmap都实现了序列化，List和Map也可以序列化，前提是它们里面的元素都是序列的。

- 效率高，主要用到内存中
- 实现起来麻烦

## Binder
Binder是Android中的一个类，实现了IBinder接口。从IPC角度来说，Binder是Android中的一种跨进程通信方式。也可以理解为一种虚拟的物理设备，设备驱动是/dev/binder，该通信方式在Linux中没有。

从Framework角度来说，Binder是ServiceManager连接各种Manager（ActivityManager、WindowManager等）和ManagerService的桥梁。

从应用层来说，Binder是客户端和服务端进行通信的媒介，当bindService的时候，服务端会返回一个包含了服务端业务调用的Binder对象，通过这个Binder对象，客户端就可以获取服务端提供的服务和数据。这里的服务包括普通服务和AIDL服务。

Android开发中，Binder主要用于Service中，包括**AIDL**和**Messenger**。普通Service的Binder不涉及进程间通信，较为简单，无法触及Binder的核心。Messenger的底层是AIDL。

所有可以在Binder传输的接口都要继承IInterface接口。

AIDL编译后生成的文件中的一些描述

- DESCRIPTOR： Binder的唯一标示，一般用当前Binder的类名标示，如"com.longge.aidi.IBookManager"
- asInterface(IBinder obj):用于将服务端的Binder对象转换成客户端所需要的AIDL接口类型的对象，这种转换过程区分进程。如果客户端和服务端在同一个进程中，此方法返回的是Stub对象本身，否则返回的是系统封装后的Stub.proxy对象。

			public static com.ryg.chapter_2.aidl.IBookManager asInterface(android.os.IBinder obj)
		{
		if ((obj==null)) {
		return null;
		}
		android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
		if (((iin!=null)&&(iin instanceof com.ryg.chapter_2.aidl.IBookManager))) {
		return ((com.ryg.chapter_2.aidl.IBookManager)iin);
		}
		return new com.ryg.chapter_2.aidl.IBookManager.Stub.Proxy(obj);
		}

- asBinder：返回当前的Binder对象
- onTransact： 这个方法运行在服务端的Binder线程池中

# Android中的IPC方式

## 使用Bundle
四大组件中的三大组件（Activity、Service、BroadcastReceiver）都支持Intent中传递Bundle，Bundle实现了Parcelable接口。可以方便的在不同进程间传输。

## 使用文件共享
共享文件也是不错的进程间通信方式。两个文件通过读写同一个文件来交换数据。比如A进程把数据写入文件，B进程通过读取这个文件来获取数据。

这种情况适合对数据同步要求不高的进程间通信，而且要妥善处理并发读/写问题。

SharedPreference是一种特例，由于系统对它的读写具有一定的缓存策略，即在内存中会有一份SharedPreference的缓存，因为在多进程模式下有很大几率丢失数据。不建议在进程间通信中使用SharedPreference。












