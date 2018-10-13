# Handler

Handler主要用于异步消息的处理： 有点类似辅助类，封装了消息投递、消息处理等接口。

当发出一个消息之后，首先进入一个消息队列，发送消息的函数即刻返回，而另外一个部分在消息队列中逐一将消息取出，然后对消息进行处理，也就是发送消息和接收消息不是同步的处理。 这种机制通常用来处理相对耗时比较长的操作。

## 原理

Handler 封装了消息的发送。其过程主要设计到了 Message、MessageQueue、Looper，我们可以理解成 handler 负责发送消息，消息存储到 MessageQueue，Looper 负责处理 handler 发送的消息，并直接把消息回传给 handler 自己。MessageQueue 就是一个存储消息的容器。

## Message

包含描述和任意数据对象的消息，用于发送给 Handler。Handler 发送消息其实就是向 MessageQueue 发送消息。

```java
//获取Message实例的方式
Message msg1 = Message.obtain();
//或
Message msg2 = Handler.obtainMessage();
```

在这里，我们看一下 POST 类的源码：

```java
public final boolean postDelayed(Runnable r, long delayMillis) {
    return sendMessageDelayed(getPostMessage(r), delayMillis);
}
 
private static Message getPostMessage(Runnable r) {
    Message m = Message.obtain();
    m.callback = r;
    return m;
}
```
注意到：在 postDelayed() 方法中，通过 getPostMessage() 方法将 Runnable 对象包装成了 Message 对象，所以对于 Handler 不论我们使用的是 POST 类方法还是 Send 类方法，发送的都是 Message。

接下来，看一下 Message 类：

```java
public final class Message implements Parcelable {
    public int what;    //定义的消息码，为了让接收者知道此消息是关于什么的
    public int arg1;    //用于发送 integer 的值
    public int arg2;    //用于发送 intege 的值
    public Object obj;  //用于传输任意类型的值
    ...
}
```

## MessageQueue

MessageQueue：顾名思义，消息队列。内部存储着一组消息。对外提供了插入和删除的操作。

MessageQueue 内部是以单链表的数据结构来存储消息列表的。

获取MessageQueue实例使用Looper.myQueue()方法。

enqueueMessage() 方法的源码：

```java
//插入Message操作
boolean enqueueMessage(Message msg, long when) {
    if (msg.target == null) {
        throw new IllegalArgumentException("Message must have a target.");
    }
    if (msg.isInUse()) {
        throw new IllegalStateException(msg + " This message is already in use.");
    }

    synchronized (this) {
        if (mQuitting) {
            IllegalStateException e = new IllegalStateException(msg.target + " sending message to a Handler on a dead thread");
            Log.w(TAG, e.getMessage(), e);
            msg.recycle();
            return false;
        }

        msg.markInUse();
        msg.when = when;
        Message p = mMessages;
        boolean needWake;
        if (p == null || when == 0 || when < p.when) {
            // New head, wake up the event queue if blocked.
            msg.next = p;
            mMessages = msg;
            needWake = mBlocked;
        } else {
            // Inserted within the middle of the queue.  Usually we don't have to wake
            // up the event queue unless there is a barrier at the head of the queue
            // and the message is the earliest asynchronous message in the queue.
            needWake = mBlocked && p.target == null && msg.isAsynchronous();
            Message prev;
            for (;;) {
                //从这里可看出MessageQueue内部是以单链表的数据结构来存储消息的
                prev = p;
                p = p.next;
                if (p == null || when < p.when) {
                    break;
                }
                if (needWake && p.isAsynchronous()) {
                    needWake = false;
                }
            }
            msg.next = p; // invariant: p == prev.next
            prev.next = msg;
        }
        // We can assume mPtr != 0 because mQuitting is false.
        if (needWake) {
            nativeWake(mPtr);
        }
    }
    return true;
}
```

## Looper

Looper:主要用于给一个线程轮询消息的。不断的从 MessageQueue 取消息，如果有消息就进行处理，如果没有就阻塞。

线程默认没有Looper,在创建Handler对象前，我们需要为线程创建 Looper。

使用 Looper.prepare() 方法创建 Looper，使用 Looper.loop() 方法运行消息队列。

```java
class LooperThread extends Thread {
    public Handler mHandler;
  
    public void run() {
        Looper.prepare();
        mHandler = new Handler() {
            public void handleMessage(Message msg) {
                // process incoming messages here
            }
        };
  
        Looper.loop();
    }
}
```

**注意，在主线程中创建 Handler 并没有创建 Looper，这是因为在 ActivityThread 的 main() 方法中初始化了 Looper。

下面是 MainThread(ActivityThread) 部分源码：

```java
public static void main(String[] args) {
    ...
    //初始化Looper
    Looper.prepareMainLooper();

    ActivityThread thread = new ActivityThread();
    thread.attach(false);

    if (sMainThreadHandler == null) {
        sMainThreadHandler = thread.getHandler();
    }

    if (false) {
        Looper.myLooper().setMessageLogging(new
        LogPrinter(Log.DEBUG, "ActivityThread"));
    }

    // End of event ActivityThreadMain.
    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
    Looper.loop();

    throw new RuntimeException("Main thread loop unexpectedly exited");
}
```

## 场景

### 延迟执行 Messages 或 Runnables

比如延时 Finish 掉启动 Activity。

### 将 A 线程的操作入队到 B 线程中

比如需要子线程中执行耗时操作，可能是读取文件或访问网络，但是我们只能在主线程中更新UI。所以使用 Handler 来将更新 UI 的操作从子线程切换到主线程中执行。

## 用法

1. 主线程创建 Handler，在子线程中通过 handler 的 post(Runnable) 方法更新 UI 信息。

2. 主线程创建 Handler，使用 handler.postDelayed( myRunnable, 1000)。

3. handler.sendMessage()

4. handler.sendEmptyMessage()

5. handler.removeCallback(runnable)

## Android 采用 handler 机制更新 UI 的原因

最根本的目的是解决多线程并发的问题：

假设如果在一个 Activity 当中，有多个线程去更新 UI，并且都没有加锁机制，那么会什么样子的问题？更新界面混乱。

如果对更新 UI 的操作都进行加锁处理的话又会产生什么样子的呢？性能下降。

出于对以上问题的考虑，Android 给我们提供了一套更新 UI 的机制，我们只要遵循这个机制就可以了，根本不用去关心多线程并发的问题，所有的更新 UI 的操作，都是在主线程的消息队列中去轮训处理的。