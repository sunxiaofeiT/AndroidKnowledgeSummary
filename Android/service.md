# Service

## 相关概念

**Local Service  VS Remote Service**
Local Service：不少人又称之为 `本地服务`，是指 `Client - Service` 同处于一个进程(APP)
Remote Service：又称之为 `远程服务`，一般是指 `Service` 处于单独的一个进程中(与 `Actiity` 处于不同的进程)。

## service 特性

1. `Service` 本身都是运行在其所在进程的主线程（如果 `Service` 与 `Clinet` 同属于一个进程，则是运行于UI线程），但 `Service` 一般都是需要进行"长期"操作，所以经常写法是在自定义 `Service` 中处理"长期"操作时需要新建线程，以免阻塞UI线程或导致 `ANR(Application Not Responding)`；

2. `Service` 一旦创建，需要停止时都需要显示调用相应的方法, `Started Service` 需要调用 `stopService(..)` 或 `Service` 本身调用 `stopSelf(..)`,`Bound Service` 需要调用 `unbindService(..)`. 

## Service 生命周期

`service` 启动方式有两种:
1. 普通的后台不可交互的 `service`, 通过 `startService()` 方式进行启动.
2. 可交互的后台 `service`，可交互是指前台页面可以调用后台服务的方法。通过 `bindService()` 方式进行启动。

>- 可交互的后台服务实现步骤是和不可交互的后台服务实现步骤是一样的，区别在于启动的方式和获得Service的代理对象。
>- 不同的启动方式他们的生命周期是不一样.

## 普通 service

普通 `service` 又被称作 `started service`。

生命周期是： `onCreate()--> onStartConmon()--> onDestroy()`。

**注意：**
- 当 `startService` 被首次调用时候，会调用 `startService()` 和 `onStartCommand()`。
- 再次调用 `startService(Intent serviceIntent)` 时,`onCreate()` 不会再被调用，只有 `onStartCommand(Intent intent, int flags, int startId)` 会被再次调用.
- 无论执行过多少次 `startService(Intent serviceIntent)`，调用一次 `stopService()`，`onDestroy()` 就会被调用，从而销毁服务。
- 当通过 `startService()` 启动时候，通过 `intent` 传值，在 `onStartCommand()` 方法中获取值的时候，一定要先判断 `intent` 是否为 null。
- `onBind(...)` 函数是 Service 基类中的唯一抽象方法，子类都必须重写实现，此函数的返回值是针对 `Bound Service` 类型的 Service 才有用的，在 `Started Service` 类型中，此函数直接返回 null 即可。
- 当 `service` 被强制销毁的时候，`onDestory()` 是不是被执行的。

**onStartCommand(Intent intent, int flags, int startId)**

其中参数 `flags` 默认情况下是 0，对应的常量名为 `START_STICKY_COMPATIBILITY`。`startId` 是一个唯一的整型，用于表示此次 Client 执行 `startService(...)` 的请求标识，在多次 `startService(...)` 的情况下，呈现 0,1,2....递增。

此函数具有一个 int 型的返回值，具体的可选值及含义如下：

1. `START_NOT_STICKY`：当 Service 因为内存不足而被系统 kill 后，接下来未来的某个时间内，即使系统内存足够可用，系统也不会尝试重新创建此 Service。除非程序中 Client 明确再次调用 startService(...) 启动此Service。

2. `START_STICKY`：当Service因为内存不足而被系统kill后，接下来未来的某个时间内，当系统内存足够可用的情况下，系统将会尝试重新创建此Service，一旦创建成功后将回调 `onStartCommand(...)` 方法，但其中的 Intent 将是 null，pendingintent除外。

3. `START_REDELIVER_INTENT`：与 `START_STICKY`唯一不同的是，回调 `onStartCommand(...)` 方法时，其中的Intent将是非空，将是最后一次调用 startService(...) 中的intent。

**started service生命周期及进程相关**

1. 对于同一类型的Service，Service实例一次永远只存在一个，而不管Client是否是相同的组件，也不管Client是否处于相同的进程中。
2. Service通过startService(..)启动Service后，此时Service的生命周期与Client本身的什么周期是没有任何关系的，只有Client调用stopService(..)或Service本身调用stopSelf(..)才能停止此Service。当然，当用户强制kill掉Service进程或系统因内存不足也可能kill掉此Service。
3. Client A 通过startService(..)启动Service后,可以在其他Client（如Client B、Client C）通过调用stopService(..)结束此Service。
4. Client调用stopService(..)时，如果当前Service没有启动，也不会出现任何报错或问题，也就是说，stopService(..)无需做当前Service是否有效的判断。
5. startService(Intent serviceIntent)，其中的intent既可以是显式Intent，也可以是隐式Intent，当Client与Service同处于一个App时，一般推荐使用显示Intent。当处于不同App时，只能使用隐式Intent。
6. 当Service需要运行在单独的进程中，AndroidManifest.xml声明时需要通过android:process指明此进程名称，当此Service需要对其他App开放时，android:exported属性值需要设置为true(当然，在有intent-filter时默认值就是true)。

**使用场景**

- 比如**播放多媒体**的时候，用户通过 Activity 控制开始、暂停等操作，此时 Activity 就需要和后台 service 通信，告诉 service 要进行的操作。
- 比如**检测SD卡上文件的变化**
- 或者在后台**记录地理信息位置**的改变等等

## 可交互 Service

可交互 `service` 又被称作 `bound service`，通过 `bindService()` 方式进行绑定。

生命周期为：`bindService-->onCreate()-->onBind()-->unBind()-->onDestroy()`。

在 `activity` 中调用后台服务的方式时，需要在 `activity` 获取 `ServiceConnection` 对象，通过 `ServiceConnection` 来获取 `service` 中内部类的类对象，然后通过这个类对象就可以调用类中的方法，当然这个类需要继承 `Binder` 对象.

> `Started Service` 与 Client 生命周期无关。
> `Bound Service` 的主要特性在于 Service 的生命周期是依附于 Client 的生命周期的，当 Client 不存在时，Bound Service将执行 onDestroy，同时通过 Service中的 Binder对象可以较为方便进行 Client-Service 通信。

**Bound Service 的使用**

1. 自定义Service继承基类Service，并重写onBind(Intent intent)方法，此方法中需要返回具体的Binder对象；
2. Client通过实现 `ServiceConnection` 接口来自定义 ServiceConnection，并通过 `bindService (Intent service, ServiceConnection sc, int flags)` 方法将Service绑定到此Client上；
3. 自定义的ServiceConnection中实现 `onServiceConnected(ComponentName name, IBinder binder)` 方法，获取Service端 Binder 实例；
4. 通过获取的 Binder 实例进行 Service 端其他公共方法的调用，以完成 Client-Service 通信；
5. 当 Client 在恰当的生命周期（如onDestroy等）时，此时需要解绑之前已经绑定的Service，通过调用函数 `unbindService(ServiceConnection sc)`。

**onBind(Intent intent)方法返回的Binder对象的定义方式不同**

1. Extending the Binder class
这是Bound Service中最常见的一种使用方式，也是Bound Service中最简单的一种。
局限：Clinet与Service必须同属于同一个进程，不能实现进程间通信（IPC）。否则则会出现类似于“android.os.BinderProxy cannot be cast to xxx”错误。

> 需要注意的的是BroadcastReceiver不能作为Bound Service的Client，因为BroadcastReceiver的生命周期很短，当执行完onReceive(..)回调时，BroadcastReceiver生命周期完结。而Bound Service又与Client本身的生命周期相关，因此，Android中不允许BroadcastReceiver去bindService(..)，当有此类需求时，可以考虑通过startService(..)替代

2. Using a Messenger

Messenger，在此可以理解成”信使“，通过Messenger方式返回Binder对象可以不用考虑Clinet - Service是否属于同一个进程的问题，并且，可以实现Client - Service之间的双向通信。极大方便了此类业务需求的实现。

局限：不支持严格意义上的多线程并发处理，实际上是以队列去处理

> 1.MyMessengerService自定中，通过new Messenger(new ServerHandler())创建Messenger对象，在onBind(..)回调中，通过调用Messenger对象的getBinder()方法，将Binder返回；
> 2.Client在ServiceConnection的onServiceConnected(..)的回调中，通过new Messenger(binder)获取到Service传递过来的mServerMessenger；
> 3.接下来，就可以通过mServerMessenger.send(msg)方法向Service发送message，Service中的Messenger构造器中的Handler即可接收到此信息，在handleMessage(..)回调中处理；
> 4.至此只是完成了从Client发送消息到Service，同样的道理，想实现Service发送消息到Client，可以在客户端定义一个Handler，并得到相应的Messenger，在Clinet发送消息给Service时，通过msg.replyTo = mClientMessenger方式将Client信使传递给Service；
> 5.Service接收到Client信使后，获取此信使，并通过mClientMessenger.send(toClientMsg)方式将Service消息发送给Client。

3. AIDL（Android Interface Definition Language）

一般情况下，Messenger这种方式都是可以满足需求的，当然，通过自定义AIDL方式相对更加灵活。
这种方式需要自己在项目中自定义xxx.aidl文件，然后系统会自动在gen目录下生成相应的接口类文件，接下来整个的流程与Messenger方式差别不大，网上也有不少实例，在此不再具体给出。

> 无论哪种方式的Bound Service，在进行unbind(..)操作时，都需要注意当前Service是否处于已经绑定状态，否则可能会因为当前Service已经解绑后继续执行unbind(..)会导致崩溃。这点与Started Service区别很大（如前文所述：stopService(..)无需做当前Service是否有效的判断）。


## 前台Service

Android中 Service 接口中还提供了一个称之为”前台Service“的概念。通过 `Service.startForeground (int id, Notification notification)` 方法可以将此Service设置为前台Service。在UI显示上，`notification` 将是一个处于 `onGoing` 状态的通知，使得前台 Service 拥有更高的进程优先级，并且 Service 可以直接进行 notification 通信。

