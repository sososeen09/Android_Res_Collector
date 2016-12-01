
# 2 Service
服务，分为两种工作状态，

- **启动状态：** 调用startService()方法，主要用于执行后台计算任务，会在后台一直存在直到被系统杀死，跟启动它的组件生命周期无关。通常情况下，不会返回结果给调用者。可以在后台下载或者上传文件，结束时候调用stopSelf()方法来关闭服务。
- **绑定状态：** 调用bindService()方法，主要用于其他组件和Service的交互

生命周期：
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

一个Service可以被多个客户进行绑定，只有所有的对象都执行了  unbindService() 方法后该 Service 才会销毁,不过如果有一个客户执行了 startService() 方法,那么这个时候如果所有的绑定客户都执行了 unbindService()该 Service 也不会 销毁。



## 2.1 StartService/stopService
## 2.2 bindService/unbindService
## 2.3 跨进程,Binder/Aidl ##
## 2.4 IntentService ##

