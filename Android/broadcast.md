# Broadcast Receiver

> [参考文章](https://www.cnblogs.com/engine1984/p/4140439.html)

## 广播的功能和特征

广播的生命周期很短，经过调用对象-->实现onReceive-->结束，整个过程就结束了。

从实现的复杂度和代码量来看，广播无疑是最迷你的 Android 组件，实现往往只需几行代码。

广播对象被构造出来后通常只执行 BroadcastReceiver.onReceive 方法，便结束了其生命周期。

Android中的四大组件是 Activity、Service、Broadcast和Content Provider。而Intent是一个对动作和行为的抽象描述，负责组件之间程序之间进行消息传递。那么Broadcast Receiver组件就提供了一种把Intent作为一个消息广播出去，由所有对其感兴趣的程序对其作出反应的机制。

和所有组件一样，广播对象也是在应用进程的主线程中被构造，所以广播对象的执行必须是要同步且快速的。也不推荐在里面开子线程，因为往往线程还未结束，广播对象就已经执行完毕被系统销毁。
如果需要完成一项比较耗时的工作 , 应该通过发送 Intent 给 Service, 由 Service 来完成。

每次广播到来时，会重新创建 BroadcastReceiver 对象，并且调用 onReceive() 方法，执行完以后，该对象即被销毁，当 onReceive() 方法在 10 秒内没有执行完毕，Android 会认为该程序无响应。

## BroadcastRceiver 生命周期

广播接收器的生命周期：关键在于 BroadcastReceiver 中的 onReceive() 方法，从 onReceive() 里的第一行代码开始，onReceive() 里的最后一行代码结束。

## 接收系统广播

广播接收器可以自由地对自己感兴趣的广播进行注册，这样当有相应的广播发出时，广播接收器就能收到该广播，并在内部处理相应的逻辑。

注册广播的方式有两种：

- 在代码中注册，动态注册，程序启动后才有效。
    这种注册方式一帮用于更新UI等吗，优先级较高。并且此注册方式在Activity销毁时需要解绑。否则会造成内存内漏，程序出错。

- 清单文件中注册，静态注册，程序不必启动。
    由于这种注册是常驻型，因此会消耗CPU的资源。

> 静态注册广播接收器可以实现开机启动。

## 发送广播

- 发送标准广播
    sendBroadcast()

- 发送有序广播
    sendOrderedBroadcast()  
    sendOrderedBroadcast()方法接收两个参数，第二个参数是一个与权限相关的字符串，这里传入null即可

在receiver中使用abortBroadcast()可以防止之后的接收器收到广播。

> 一个广播到来的时候，用什么方式提醒用户是最友好的呢？第一种方式是吐司，第二种方式是通知。注：不要使用对话框，以免中断了用户正在进行的操作。

## 本地广播

只能在本应用程序中发送和接收广播。

使用到了 `LocalBroadcastManager` 这个类来对广播进行管理。

localBroadcastManager.sendBroadcast(intent) //调用sendBroadcast()方法发送广播

> 本地广播是无法通过静态注册的方式来接收的。其实也完全可以理解，因为静态注册主要就是为了让程序在未启动的情况下也能收到广播。而发送本地广播时，我们的程序肯定是已经启动了，没有必要使用到静态注册的功能。

## 在manifest和代码中如何注册和使用 broadcast receiver 。

**在android的manifest中注册：**

```xml
<receiver android: name =“Receiver1”>

    <intent-filter>
        <!-- 和Intent中的action对应 -->
        <action android: name="com.forrest.action.mybroadcast"/>
    </intent-filter>

</receiver>
```

**在代码中注册**

```java
IntentFilter filter = new IntentFilter("com.forrest.action.mybroadcast");//和广播中Intent的action对应;
MyBroadcastReceiver br= new MyBroadcastReceiver();
registerReceiver(br, filter);
```