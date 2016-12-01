
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

**IntentService的特征:**

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

##2.6 常见面试题
**1.Service 是否在 main thread 中执行, service 里面是否 能执行耗时的操作? **

默认情况,如果没有显示的指 service 所运行的进程, Service 和 activity 是运行在当前 app 所在进程的 main thread(UI 主线程)里面。 
service 里面不能执行耗时的操作(网络请求,拷贝数据库,大文件 ) 
特殊情况 ,可以在清单文件配置 service 执行所在的进程 ,让 service 在另 外的进程中执行
	
	<service android:name="com.baidu.location.f" android:enabled="true" android:process=":remote" >
	</service>
**2.Activity 怎么和 Service 绑定,怎么在 Activity 中启动自己对应的 Service? **

Activity 通过 bindService(Intent service, ServiceConnection conn, int flags)跟 Service 进行绑定,当绑定成功的时候 Service 会将代理对象通过回调 的形式传给 conn,这样我们就拿到了 Service 提供的服务代理对象。 
在 Activity 中可以通过 startService 和 bindService 方法启动 Service。一 般情况下如果想获取 Service 的服务对象那么肯定需要通过 bindService()方 法,比如音乐播放器,第三方支付等。如果仅仅只是为了开启一个后台任务那么 可以使用 startService()方法。

**3.Service 的生命周期**
见本文上面

**4.什么是 IntentService?有何优点?**
见本文上面

**5.Activity、Intent、Service 是什么关系？**

他们都是 Android 开发中使用频率最高的类。其中 Activity 和 Service 都是 Android 四大组件之一。他俩都是 Context 类的子类 ContextWrapper 的子类, 因此他俩可以算是兄弟关系吧。不过兄弟俩各有各自的本领,Activity 负责用户 界面的显示和交互,Service 负责后台任务的处理。Activity 和 Service 之间可 以通过 Intent 传递数据,因此可以把 Intent 看作是通信使者。

**6.Service 和 Activity 在同一个线程吗？**

对于同一 app 来说默认情况下是在同一个线程中的,main Thread (UI Thread)。

**7.Service 里面可以弹吐司么 ？**

可以的。弹吐司有个条件就是得有一个 Context 上下文,而 Service 本身就是 Context 的子类,因此在 Service 里面弹吐司是完全可以的。比如我们在 Service 中完成下载任务后可以弹一个吐司通知用户。

**8.什么是 Service 以及描述下它的生命周期。Service 有哪 些启动方法,有什么区别,怎样停用 Service?**

在 Service 的生命周期中,被回调的方法比 Activity 少一些,只有 onCreate, onStart, onDestroy, 
onBind 和 onUnbind。 
通常有两种方式启动一个 Service,他们对 Service 生命周期的影响是不一样的。

通过 startService 
Service 会经历 onCreate 到 onStart,然后处于运行状态,stopService 
的时候调用 onDestroy 方法。 
如果是调用者自己直接退出而没有调用 stopService 的话,Service 会一直 
在后台运行。
通过 bindService 
Service 会运行 onCreate,然后是调用 onBind,这个时候调用者和 Service 绑定在一起。调用者退出了,Srevice 就会调用 onUnbind->onDestroyed 方 法。 
所谓绑定在一起就共存亡了。调用者也可以通过调用 unbindService 方法来 停止服务,这时候 Srevice 就会调用 onUnbind->onDestroyed 方法。 需要注意的是如果这几个方法交织在一起的话,会出现什么情况呢? 一个原则是 Service 的 onCreate 的方法只会被调用一次,就是你无论多少次的 startService 又 bindService,Service 只被创建一次。 
如果先是 bind 了,那么 start 的时候就直接运行 Service 的 onStart 方法,如 果先是 start,那么 bind 的时候就直接运行 onBind 方法。 
如果 service 运行期间调用了 bindService,这时候再调用 stopService 的话,service 是不会调用 onDestroy 方法的,service 就 stop 不掉了,只能调用 UnbindService, service 就会被销毁 
如果一个 service 通过 startService 被 start 之后,多次调用 startService 的 话,service 会多次调用 onStart 方法。多次调用 stopService 的话,service 只会调用一次 onDestroyed 方法。 
如果一个 service 通过 bindService 被 start 之后,多次调用 bindService 的话, service 只会调用一次 onBind 方法。 
多次调用 unbindService 的话会抛出异常。

**9.在 service 的生命周期方法 onstartConmand()可不可以执行网络操作?如何在 service 中执行网络操作?**

可以直接在 Service 中执行网络操作,在 onStartCommand()方法中可以执行网络操作

**10.如何提高service的优先级？**

Android 系统对于内存管理有自己的一套方法，为了保障系统有序稳定的运信， 
系统内部会自动分配，控制程序的内存使用。当系统觉得当前的资源非常有限的时候， 
为了保 证一些优先级高的程序能运行，就会杀掉一些他认为不重要的程序或者服务来释放内存。 
这样就能保证真正对用户有用的程序仍然再运行。如果你的 Service 碰上了这种情况，多半会先被杀掉。 
但如果你增加 Service 的优先级就能让他多留一会， 
我们可以用 setForeground(true) 来设置 Service 的优先级。 
为什么是 foreground ? 默认启动的 Service 是被标记为 background，当前运行的 Activity 
一般被标记为 foreground，也就是说你给 Service 设置了 foreground 那么他就和正在运行的 
Activity 类似优先级得到了一定的提高。当让这并不能保证你得 Service 
永远不被杀掉，只是提高了他的优先级。

**11.service 如何定时执行？**

当启动service进行后台任务的时候，我们一般的 做法是启动一个线程，然后通过sleep方法来控制进行定时的任务，如轮询操作，消息推送。这样容易被系统回收。 
service被回收是我们不能控制的，但是我们可以控制service的重启活动。在service的onStartCommand 
方法中可以返回一个参数来控制重启活动 
使用AlarmManager，根据AlarmManager的工作原理，alarmmanager会定时的发出一条广播，然后在自己的项目里面注册这个广播，重写onReceive方法，在这个方法里面启动一个service，然后在service里面进行网络的访问操作，当获取到新消息的时候进行推送，同时再设置一个alarmmanager进行下一次的轮询，当本次轮询结束的时候可以stopself结束改service。这样即使这一次的轮询失败了，也不会影响到下一次的轮询。这样就能保证推送任务不会中断

**12.Service 的 onStartCommand 方法有几种返回值?各代表什么意思?**

有四种返回值,不同值代表的意思如下: 

- START_STICKY:如果 service 进程被 kill 掉,保留 service 的状态为开始状态,但不保留递送的 intent 对象。随 后系统会尝试重新创建 service,由于服务状态为开始状态,所以创建服务后一定会调用 onStartCommand(Intent,int,int)方法。如果在此期间没有任何启动命令被传递到 service,那么参数 Intent 将为 null。 
- START_NOT_STICKY:“非粘性的”。使用这个返回值时,如果在执行完 onStartCommand 后,服务被异常 kill 掉,系统不会自动重启该服务。 
- START_REDELIVER_INTENT:重传 Intent。使用这个返回值时,如果在执行完 onStartCommand 后,服务被异 常 kill 掉,系统会自动重启该服务,并将 Intent 的值传入。 
- START_STICKY_COMPATIBILITY:START_STICKY 的兼容版本,但不保证服务被 kill 后一定能重启。

**13.Service 的 onRebind(Intent)方法在什么情况下会执行?**

如果在 onUnbind()方法返回 true 的情况下会执行,否则不执行。

**14.Activity 调用 Service 中的方法都有哪些方式?**

Binder： 
通过 Binder 接口的形式实现,当 Activity 绑定 Service 成功的时候 Activity 会在 ServiceConnection 的类 的 onServiceConnected()回调方法中获取到 Service 的 onBind()方法 return 过来的 Binder 的子类，然后通过对象调用方法。 
Aidl: 
aidl 比较适合当客户端和服务端不在同一个应用下的场景。

Messenger： 
它引用了一个Handler对象，以便others能够向它发送消息(使用mMessenger.send(Message msg)方法)。该类允许跨进程间基于Message的通信(即两个进程间可以通过Message进行通信)，在服务端使用Handler创建一个Messenger，客户端持有这个Messenger就可以与服务端通信了。一个Messeger不能同时双向发送，两个就就能双向发送了

BroadCastReceiver

**15.Service 如何给 Activity 发送 Message?**

Service 和 Activity 如果想互发 Message 就必须使用使用 Messenger 机制。

**16.IntentService 适用场景**

IntentService 内置的是 HandlerThread 作为异步线程，每一个交给 IntentService 的任务都将以队列的方式逐个被执行到，一旦队列中有某个任务执行时间过长，那么就会导致后续的任务都会被延迟处理
正在运行的 IntentService 的程序相比起纯粹的后台程序更不容易被系统杀死，该程序的优先级是介于前台程序与纯后台程序之间的

