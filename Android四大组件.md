
# 2 Service

Service在后台长期运行的组件，通过Service可以实现跨进程通讯（IPC）。Service可以处理网络传输，播放音乐，文件I/O，或者与内容提供者交互。
通过Context启动Service。

##2.1 两种状态
服务，分为两种工作状态，

- **启动状态：** 调用startService(Intent service)方法，主要用于执行后台计算任务，会在后台一直存在直到被系统杀死，跟启动它的组件生命周期无关。通常情况下，不会返回结果给调用者。可以在后台下载或者上传文件，结束时候调用stopSelf()方法来关闭服务。
- **绑定状态：** 调用bindService(Intent service, ServiceConnection conn, int flags)方法，主要用于其他组件和Service的交互。类似于客户端与服务端的交互，可以发送请求得到结果，甚至是IPC。生命周期与绑定它的客户端相一致（单个客户端情况）。


##2.2 生命周期
由于Service的启动分为绑定模式和非绑定模式，以及这两种模式的混合模式，不同的启动方法生命周期方法也不同。

**非绑定模式 startService()：**

- **onCreate()**
- **onStartCommand()**
- **onDestroy()**

**绑定模式 bindService()：**
第一次调用的时候，

- **onCreate()**
- **onUnbind()**
- **onDestroy()**


> 注意：**Service 实例只会有一个**,也就是说如果当前要启动的 Service 已经存 在了那么就不会再次创建该 Service 当然也不会调用 onCreate()方法。 

如果是startService()启动，Service会一直运行知道它自己调用 stopSelf()方法，或者其他的组件调用stopService()方法来停止。

一个Service可以被多个客户进行绑定，只有所有的对象都执行了  unbindService() 方法后该 Service 才会销毁,不过如果有一个客户执行了 startService() 方法,那么这个时候如果所有的绑定客户都执行了 unbindService()该 Service 也不会 销毁。

![Service生命周期图](https://github.com/sososeen09/Android_Res_Collector/blob/master/Service%20lifecycle.png)




> **注意：**默认情况下，服务是在当前app进程（除非指定单独的进程）中的**主线程**（亲自尝试在子线程中startService，依然是运行在主线程）中运行。这意味着如果Service需要做CPU密集型工作或者其他的阻塞操作（如播放音乐或者网络请求）,应该在Service中创建一个新的线程。这样可以减少ANR风险，并且Activity可以做其他交互而不影响Activity的性能。

自己必须创建Service的子类，可以继承Service或者IntentService。

- **Service：**是所有Services的基类，如果继承这个类，最好创建一个子线程进行耗时工作，因为Service默认运行在当前app的主线程中。
- **IntentService：**是Service的子类，采用工作线程来处理请求。如果不要求Service同时处理多个任务，那么这个IntentService就是最佳选择。只需要实现 **onHandleIntent()** 方法，接收每一个请求的Intent。



几个重要的回调：

###2.2.1 onStartCommand()
如果其它组件如Activity调用 **startService()** ，系统会回调这个方法。一旦这个方法执行就会在后台运行。完成工作之后需要自己调用 **stopSelf() **或者 **stopService()** 方法来结束。如果只是需要绑定的话可以不用实现这个方法。

### 2.2.2 onBind()
如果其它组件调用bindService()方法启动Service会回调这个方法，必须返回一个IBinder接口用于客户端与Service交互。如果不想要绑定，可以返回null。

###2.2.3 onCreate()
当Service第一次创建的时候由系统调用，可以一次性设置一些东西。会在**onStartCommand()** 或者 **onBind()**之前调用。如果Service已经运行，不会调用这个方法。

###2.2.4 onDestroy()
系统回调这个方法，当Service不再使用的并被销毁的时候。在这个方法中清空资源，，例如线程、注册的监听、广播等。这个是Service最后接收的回调。


##2.3 强杀
如果系统内存很低，Service可能会被强制停止。如果Service绑定的Activity处在前台，就不容易被杀死。如果一个Service声明为前台服务，就基本上不会被杀死。运行的时间越长，越有可能被系统杀死。但是系统还会重新启动这个服务，在内存充足的时候。


##2.4 IntentService
由于多数情况下开启服务并不需要同时处理多个请求，那么采用 **IntentService** 处理耗时任务就比较合适。

**IntentService的作用:**

- 创建一个默认的工作线程来执行所有传递到 **onStartCommand()** 的Intent。
- 创建一个工作队列来一次传递一个Intent给 **onHandleIntent()** 的实现方法，所以不用担心多线程问题。
- 当所有的请求处理完毕的时候会停止服务而不需要你自己调用 **stopSelf()**方法
- 实现了**onBind()**方法默认返回null
- 提供了一个 **onStartCommand()** 方法的默认实现，把Intent传递到工作线程，然后再传递给 **onHandleIntent()** 方法。

所有需要做的仅仅是实现抽象方法 **onHandleIntent()**来进行工作。在这里可以进行耗时工作。

> **注意：** 根据以上的说明可知，IntentService并不适合同时处理多个任务的情况。它只是把任务插入到队列里，就像处理消息一样。

如果你自己想要重写 **onCreate()、onStartCommand()、onDestroy()** 方法，确保会调用super实现。例如， **onStartCommand()** 必须返回默认的实现，这是表示Intent如何传递给 **onHandleIntent()** 方法。

	@Override
	public int onStartCommand(Intent intent, int flags, int startId) {
	    Toast.makeText(this, "service starting", Toast.LENGTH_SHORT).show();
	    return super.onStartCommand(intent,flags,startId);
	}

除了 **onHandleIntent()**方法，另一个不需要调用父类的方法是 **onBind()** ,除非你想要允许绑定服务。

> **如果需要同时处理请求，最好自己还是继承Service。**


##2.5 Service的onStartCommand(Intent intent, int flags, int startId) 方法返回值

**onStartCommand()** 是有返回值的，这个返回值是用来描述当系统杀死服务之后如何继续这个服务。在IntentService中已经做了处理。

	#IntentService
    @Override
    public int onStartCommand(@Nullable Intent intent, int flags, int startId) {
        onStart(intent, startId);
        return mRedelivery ? START_REDELIVER_INTENT : START_NOT_STICKY;
    }

**onStartCommand()**的返回值必须是下面的其中之一：

- START_NOT_STICKY 意思是如果被系统杀死就不用再重启服务，除非后续传递了Intent来启动服务。这是一种比较安全的方式，避免在不必要的情况下运行服务。
- START_STICKY 被系统杀死之后系统会重启服务，重新创建，但是并不会再传递最后一个Intent，因此在回调 **onStartCommand()** 方法时传递的Intent为null，除非会面再由其他组件继续开启服务传递Intent。比较适合MediaPlayer或者其他类似的服务，并不执行任务，一直运行等待任务。
- START_REDELIVER_INTENT 被系统杀死之后系统会重新创建服务，而且 **onStartCommand()** 方法会传递最后一个传递的Intent，其他后续的Intent可以继续进行。这个比较适合于启动之后就要马上工作的情况，如下载文件。
- START_STICKY_COMPATIBILITY START_STICKY的兼容版本，不保证被杀死之后**onStartCommand()**方法会被继续调用。



## 2.3 跨进程,Binder/Aidl ##


