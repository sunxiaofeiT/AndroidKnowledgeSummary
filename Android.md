# Android

## 基础知识

### 相关概念

#### 任务栈

> 一个应用对应一个任务栈，存储了此应用的所有 `Activity`。

- 任务栈
    1. 程序打开时就创建了一个任务栈, 用于存储当前程序的activity,所有的activity属于一个任务栈。 
    2. 一个任务栈包含了一个activity的集合, 去有序的选择哪一个activity和用户进行交互:只有在任务栈栈顶的activity才可以跟用户进行交互。 
    3. 任务栈可以移动到后台, 并且保留了每一个activity的状态. 并且有序的给用户列出它们的任务, 而且还不丢失它们状态信息。 
    4. 退出应用程序时：当把所有的任务栈中所有的activity清除出栈时,任务栈会被销毁,程序退出。

- 缺点：
    - 每开启一次页面都会在任务栈中添加一个Activity,而只有任务栈中的Activity全部清除出栈时，任务栈被销毁，程序才会退出,这样就造成了用，户体验差, 需要点击多次返回才可以把程序退出了。 
    - 每开启一次页面都会在任务栈中添加一个Activity还会造成数据冗余, 重复数据太多, 会导致内存溢出的问题(OOM)。

#### Task

> 因为任务栈的缺点，引入了 `启动模式` ，由 `Task` 进行 `Activity` 的管理。

`Task` 是一个具有栈结构的对象，它管理一个应用的多个 `Activity`，启动一个应用就启动了一个管理此应用的 `Task`。

#### 启动模式(Launch Model)

##### 模式说明

- standard(默认)
    - 不管有没有实例，都生成新的实例
- singleTop
    - 检查此时栈顶Activity是否是目标Activity，如果是，不再生成新的实例，直接跳转，如果不是生成新的实例。
- singleTask
    - 检查栈中是否存在Activity实例，如果存在，使其上方的Activity统统出栈，使其为栈顶Activity显示。
- singleInstance
    - 设置为此模式的Activity在start时将存在一个新的栈中。
    - 比如activity的start顺序为 A->B->C,在C界面返回到的是A界面。

##### 配置方法

AndroidManifest.xml 中Activity的android:launchMode属性。

###### 使用场景
 
- singleTop适合接收通知启动的内容显示页面。例如，某个新闻客户端的新闻内容页面，如果收到10个新闻推送，每次都打开一个新闻内容页面是很烦人的。

- singleTask适合作为程序入口点。例如浏览器的主界面。不管从多少个应用启动浏览器，只会启动主界面一次，其余情况都会走onNewIntent，并且会清空主界面上面的其他页面。之前打开过的页面，打开之前的页面就ok，不再新建。

- singleInstance适合需要与程序分离开的页面。例如闹铃提醒，将闹铃提醒与闹铃设置分离。singleInstance不要用于中间页面，如果用于中间页面，跳转会有问题，比如：A -> B (singleInstance) -> C，完全退出后，在此启动，首先打开的是B。


## 四大组件（生命周期，使用场景，如何启动）

### 概念

四大组件的概念。

#### Activity

> APP中一个 `activity` 相等于一个页面，它包含一些组件，主要用于和用户交互，一个应用程序中可以有很多个 `activity`。

#### BroadcastReceiver

> APP可以使用它对本身外部的事件进行过滤，对那些本身感兴趣的事件做出反应，例如电话呼入、数据网络连接时等。`BroadcastReceiver`本身没有页面，但是它可以启动一个`Acticity`或者`Service`来响应收到的事件信息，也可以通过`NotificationManager`来发出一个通知。

#### ContentProvider

> 内容提供者主要用于在不同应用程序之间实现数据共享的功能，它提供了一套完整的机制，允许一个程序访问另一个程序中的数据，同时还能保证被访问数据的安全性。只有需要在多个应用程序间共享数据时才需要内容提供者。例如：通讯录数据被多个应用程序使用，且必须存储在一个内容提供者中。它的好处：统一数据访问方式。

#### Service

> 是Android中实现程序后台运行的解决方案，它非常适合去执行那些不需要和用户交互而且还要长期运行的任务（一边打电话，后台挂着QQ）。服务的运行不依赖于任何用户界面，即使程序被切换到后台，或者用户打开了另一个应用程序，服务扔然能够保持正常运行，不过服务并不是运行在一个独立的进程当中，而是依赖于创建服务时所在的应用程序进程。当某个应用程序进程被杀掉后，所有依赖于该进程的服务也会停止运行（正在听音乐，然后把音乐程序退出）。


### Activity

#### 七个生命周期

共有七个周期函数，按顺序分别是: onCreate(), onStart(), onRestart(), onResume(), onPause(),onStop(), onDestroy()。

onCreate(): 创建Activity时调用，设置在该方法中，还以Bundle的形式提供对以前存储的任何状态的访问。

onStart(): Activity变为在屏幕上对用户可见时调用。

onResume(): Activity开始与用户交互时调用(无论是启动还是重新启动一个活动，该方法总是被调用）。

onPause(): Activity被暂停或收回cpu和其他资源时调用，该方法用户保护活动状态的，也是保护现场。

onStop(): Activity被停止并转为不可见阶段及后续的生命周期事件时调用。

onRestart(): Activity被重新启动时调用。该活动仍然在栈中，而不是启动新的Activity。

#### 生命周期分类

- 完整生命周期，Activity从出现到消失，onCreate() => onDestory()

- 可见生命周期，可见但不一定可交互，onStart() => onStop()

- 前景生命周期, 处于栈的顶端，可见可交互，onResume() => onPause()


### Service

#### 相关概念

**Local Service  VS Remote Service**
Local Service：不少人又称之为”本地服务“，是指Client - Service同处于一个进程；
Remote Service：又称之为”远程服务“，一般是指Service处于单独的一个进程中。

#### service 特性

1. Service本身都是运行在其所在进程的主线程（如果Service与Clinet同属于一个进程，则是运行于UI线程），但Service一般都是需要进行”长期“操作，所以经常写法是在自定义Service中处理”长期“操作时需要新建线程，以免阻塞UI线程或导致ANR；

2. Service一旦创建，需要停止时都需要显示调用相应的方法（Started Service需要调用stopService(..)或Service本身调用stopSelf(..)， Bound Service需要调用unbindService(..)），否则对于Started Service将处于一直运行状态，对于Bound Service，当Client生命周期结束时也将因此问题。也就是说，Service执行完毕后，必须人为的去停止它。

#### Started Service

> 继承Service类

- context.startService(Intent serviceIntent)启动Service
- context.stopService(Intent serviceIntent)停止此Service
- 在Service内部，也可以通过stopSelf(...)方式停止其本身。

**实现函数**

- onBind(...)函数是Service基类中的唯一抽象方法，子类都必须重写实现，此函数的返回值是针对Bound Service类型的Service才有用的，在Started Service类型中，此函数直接返回 null 即可
- onCreate(...)、onStartCommand(...)和onDestroy()都是Started Service相应生命周期阶段的回调函数

当Client调用startService(Intent serviceIntent)后，如果MyService是第一次启动，首先会执行 onCreate()回调，然后再执行onStartCommand(Intent intent, int flags, int startId)，当Client再次调用startService(Intent serviceIntent)，将只执行onStartCommand(Intent intent, int flags, int startId)，因为此时Service已经创建了，无需执行onCreate()回调。无论多少次的startService，只需要一次stopService()即可将此Service终止，执行onDestroy()函数（其实很好理解，因为onDestroy()与onCreate()回调是相对的）

**onStartCommand(Intent intent, int flags, int startId)**
其中参数flags默认情况下是0，对应的常量名为START_STICKY_COMPATIBILITY。startId是一个唯一的整型，用于表示此次Client执行startService(...)的请求请求标识，在多次startService(...)的情况下，呈现0,1,2....递增。另外，此函数具有一个int型的返回值，具体的可选值及含义如下：

START_NOT_STICKY：当Service因为内存不足而被系统kill后，接下来未来的某个时间内，即使系统内存足够可用，系统也不会尝试重新创建此Service。除非程序中Client明确再次调用startService(...)启动此Service。

START_STICKY：当Service因为内存不足而被系统kill后，接下来未来的某个时间内，当系统内存足够可用的情况下，系统将会尝试重新创建此Service，一旦创建成功后将回调onStartCommand(...)方法，但其中的Intent将是null，pendingintent除外。

START_REDELIVER_INTENT：与START_STICKY唯一不同的是，回调onStartCommand(...)方法时，其中的Intent将是非空，将是最后一次调用startService(...)中的intent。

**started service生命周期及进程相关**

1. onCreate(Client首次startService(..)) >> onStartCommand >> onStartCommand - optional ... >> onDestroy(Client调用stopService(..))
注：onStartCommand(..)可以多次被调用，onDestroy()与onCreate()想匹配，当用户强制kill掉进程时，onDestroy()是不会执行的。
2. 对于同一类型的Service，Service实例一次永远只存在一个，而不管Client是否是相同的组件，也不管Client是否处于相同的进程中。
3. Service通过startService(..)启动Service后，此时Service的生命周期与Client本身的什么周期是没有任何关系的，只有Client调用stopService(..)或Service本身调用stopSelf(..)才能停止此Service。当然，当用户强制kill掉Service进程或系统因内存不足也可能kill掉此Service。
4. Client A 通过startService(..)启动Service后,可以在其他Client（如Client B、Client C）通过调用stopService(..)结束此Service。
5. Client调用stopService(..)时，如果当前Service没有启动，也不会出现任何报错或问题，也就是说，stopService(..)无需做当前Service是否有效的判断。
6. startService(Intent serviceIntent)，其中的intent既可以是显式Intent，也可以是隐式Intent，当Client与Service同处于一个App时，一般推荐使用显示Intent。当处于不同App时，只能使用隐式Intent。
当Service需要运行在单独的进程中，AndroidManifest.xml声明时需要通过android:process指明此进程名称，当此Service需要对其他App开放时，android:exported属性值需要设置为true(当然，在有intent-filter时默认值就是true)。


#### Bound Service

> Bound Service的主要特性在于Service的生命周期是依附于Client的生命周期的，当Client不存在时，Bound Service将执行onDestroy，同时通过Service中的Binder对象可以较为方便进行Client-Service通信

**Bound Service 的使用**
1. 自定义Service继承基类Service，并重写onBind(Intent intent)方法，此方法中需要返回具体的Binder对象；
2. Client通过实现ServiceConnection接口来自定义ServiceConnection，并通过bindService (Intent service, ServiceConnection sc, int flags)方法将Service绑定到此Client上；
3. 自定义的ServiceConnection中实现onServiceConnected(ComponentName name, IBinder binder)方法，获取Service端Binder实例；
4. 通过获取的Binder实例进行Service端其他公共方法的调用，以完成Client-Service通信；
5. 当Client在恰当的生命周期（如onDestroy等）时，此时需要解绑之前已经绑定的Service，通过调用函数unbindService(ServiceConnection sc)。

**onBind(Intent intent)方法放回的Binder对象的定义方式不同**
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


#### 前台Service

Android中Service接口中还提供了一个称之为”前台Service“的概念。通过Service.startForeground (int id, Notification notification)方法可以将此Service设置为前台Service。在UI显示上，notification将是一个处于onGoing状态的通知，使得前台Service拥有更高的进程优先级，并且Service可以直接notification通信。

### Content Provider

`Content Provider` 是Android系统提供的在多个应用之间共享数据的一种机制。
一个Content Provider类实现了一组标准的方法接口，从而能够让其他的应用保存或读取此Content Provider的各种数据类型。

- 每个ContentProvider都会对外提供一个公共的URI（包装成Uri对象），如果应用程序有数据需要共享，就需要使用ContentProvider为这些数据定义一个URI，然后其他应用程序就可以通过ContentProvider传入这个URI来对数据进行操作。
- 我们的APP可以通过实现一个Content Provider的抽象接口将自己的数据暴露出去，也可以通过ContentResolver接口访问Content Provider提供的数据；
- ContentResolver支持CRUD(create, retrieve, update, and delete)操作；
- Android系统提供了诸如：音频、视频、图片、通讯录等主要数据类型的Content Provider。我们也可以创建自己的Content Provider。

**URI简介**
URI唯一标识了Provider中的数据，当应用程序访问Content Provider中的数据时，URI将会是其中一个重要参数。
URI包含了两部分内容：
- 要操作的Content Provider对象
- 要操作的Content Provider中数据的类型

**URI由以下几个部分组成**
（1）Scheme：在Android中URI的Scheme是不变的，即：Content://
（2）Authority：用于标识ContentProvider（API原文：A string that identifies the entire content provider.）；
（3）Path：用来标识我们要操作的数据。零个或多个段，用正斜杠（/）分割;
（4）ID：指定ID后，将操作指定ID的数据（ID也可以看成是path的一部分），ID在URI中是可选的（API原文：A unique numeric identifier for a single row in the subset of data identified by the preceding path part.）。

**URI示例**
（1）content://media/internal/images   返回设备上存储的所有图片;
（2）content://media/internal/images /10 返回设备上存储的ID为10的图片 

**操作URI经常会使用到UriMatcher和ContentUris两个类**
UriMatcher：用于匹配Uri；
ContentUris：用于操作Uri路径后面的ID部分，如提供了方法withAppendedId（）向URI中追加ID

**注意**
（1）访问Content Provider需要一定的操作权限；
（2）访问Content Prvider需要使用到ContentResolver对象；
（3）ContentResolver支持query，insert，delete，update操作；
（4）由URI确定Content Provider中要操作的具体数据；
（5）insert时，要添加的数据可以使用ContentValues封装。


### Broadcast Receiver

> [参考文章](https://www.cnblogs.com/engine1984/p/4140439.html)

**广播的功能和特征**
广播的生命周期很短，经过调用对象-->实现onReceive-->结束，整个过程就结束了。
从实现的复杂度和代码量来看，广播无疑是最迷你的Android 组件，实现往往只需几行代码。广播对象被构造出来后通常只执行BroadcastReceiver.onReceive方法，便结束了其生命周期。所以有的时候我们可以把它当做函数看也未必不可。
Android中的四大组件是 Activity、Service、Broadcast和Content Provider。而Intent是一个对动作和行为的抽象描述，负责组件之间程序之间进行消息传递。那么Broadcast Receiver组件就提供了一种把Intent作为一个消息广播出去，由所有对其感兴趣的程序对其作出反应的机制。
和所有组件一样，广播对象也是在应用进程的主线程中被构造，所以广播对象的执行必须是要同步且快速的。也不推荐在里面开子线程，因为往往线程还未结束，广播对象就已经执行完毕被系统销毁。如果需要完成一项比较耗时的工作 , 应该通过发送 Intent 给 Service, 由 Service 来完成。
每次广播到来时 , 会重新创建 BroadcastReceiver 对象 , 并且调用 onReceive() 方法 , 执行完以后 , 该对象即被销毁 . 当 onReceive() 方法在 10 秒内没有执行完毕， Android 会认为该程序无响应。

**接收系统广播**
广播接收器可以自由地对自己感兴趣的广播进行注册，这样当有相应的广播发出时，广播接收器就能收到该广播，并在内部处理相应的逻辑。

注册广播的方式有两种：
- 在代码中注册，动态注册，程序启动后才有效。
    这种注册方式一帮用于更新UI等吗，优先级较高。并且此注册方式在Activity销毁时需要解绑。否则会造成内存内漏，程序出错。
- 清单文件中注册，静态注册，程序不必启动。
    由于这种注册是常驻型，因此会消耗CPU的资源。

静态注册广播接收器可以实现开机启动。

**发送广播**

- 发送标准广播
sendBroadcast()

- 发送有序广播
sendOrderedBroadcast()  
sendOrderedBroadcast()方法接收两个参数，第二个参数是一个与权限相关的字符串，这里传入null即可

在receiver中使用abortBroadcast()可以防止之后的接收器收到广播。

> 广播接收器的生命周期：关键在于BroadcastReceiver中的onReceive()方法，从onReceive()里的第一行代码开始，onReceive()里的最后一行代码结束。
> 一个广播到来的时候，用什么方式提醒用户是最友好的呢？第一种方式是吐司，第二种方式是通知。注：不要使用对话框，以免中断了用户正在进行的操作。

**本地广播**
只能在本应用程序中发送和接收广播。
使用到了 `LocalBroadcastManager` 这个类来对广播进行管理。

localBroadcastManager.sendBroadcast(intent);//调用sendBroadcast()方法发送广播

> 本地广播是无法通过静态注册的方式来接收的。其实也完全可以理解，因为静态注册主要就是为了让程序在未启动的情况下也能收到广播。而发送本地广播时，我们的程序肯定是已经启动了，没有必要使用到静态注册的功能。


## Intent

Android中提供了Intent机制来协助应用间的交互与通讯，或者采用更准确的说法是，Intent不仅可用于应用程序之间，也可用于应用程序内部的activity, service和broadcast receiver之间的交互。Intent这个英语单词的本意是“目的、意向、意图”。
Intent是一种运行时绑定（runtime binding)机制，它能在程序运行的过程中连接两个不同的组件。通过Intent，你的程序可以向Android表达某种请求或者意愿，Android会根据意愿的内容选择适当的组件来响应。 

> activity、service和broadcast receiver之间是通过Intent进行通信的，而另外一个组件Content Provider本身就是一种通信机制，不需要通过Intent.

#### Intent的相关属性

**Intent由以下各个组成部分**
- component(组件)：目标组件
- action（动作）：用来表现意图的行动
- category（类别）：用来表现动作的类别
- data（数据）：表示与动作要操纵的数据
- type（数据类型）：对于data范例的描写
- extras（扩展信息）：扩展信息
- Flags（标志位）：期望这个意图的运行模式

**Intent类型**
Intent类型分为显式Intent（直接类型）、隐式Intent（间接类型）。官方建议使用隐式Intent。上述属性中，component属性为直接类型，其他均为间接类型。

> 直接使用component声明intent目标的为显示Intent，使用action、category等属性为隐式Intent。

**Intent Action**
当日常生活中，描述一个意愿或愿望的时候，总是有一个动词在其中。比如：我想“做”三个俯卧撑；我要“写” 一封情书，等等。在Intent中，Action就是描述做、写等动作的，当你指明了一个Action，执行者就会依照这个动作的指示，接受相关输入，表现对应行为，产生符合的输出。在Intent类中，定义了一批量的动作，比如ACTION_VIEW，ACTION_PICK等， 基本涵盖了常用动作。加的动作越多，越精确。

Action 是一个用户定义的字符串，用于描述一个 Android 应用程序组件，一个 Intent Filter 可以包含多个 Action。在 AndroidManifest.xml 的Activity 定义时，可以在其 <intent-filter >节点指定一个 Action列表用于标识 Activity 所能接受的“动作”

**Intent Category**
Category属性也是作为<intent-filter>子元素来声明的。

> Action 和category通常是放在一起用的。

**例**
```xml
<activity 
    android:name=".SecondActivity">
    <intent-filter>
        <action android:name="com.example.smyh006intent01.MY_ACTION"/>
        <category android:name="android.intent.category.DEFAULT" />
    </intent-filter>            
</activity>
```
上方代码，表示SecondActicity可以匹配第4行的MY_ACTION这个动作，此时，如果在其他的Acticity通过这个action的条件来查找，那SecondActicity就具备了这个条件。类似于相亲时，我要求对方有哪些条件，然后对方这个SecondActicity恰巧满足了这个条件（够通俗了吧）。
也就是说：只有<action>和<category>中的内容同时能够匹配上Intent中指定的action和category时，这个活动才能响应Intent。如果使用的是DEFAULT这种默认的category，在稍后调用startActivity()方法的时候会自动将这个category添加到Intent中

> 如果没有指定的category，则必须使用默认的DEFAULT（即上方第5行代码）。
> 如果有多个activity匹配成功，则会让用户进行选择。（在某个应用中选择分享的时候可以选择多个应用）
> 每个Intent只能指定一个Action，但是能设置多个Category，类别越多，动作越具体，意图越明确。

**Intent data**
表示与动作要操纵的数据
Data属性是Android要访问的数据，和action和Category声明方式相同，也是在<intent-filter>中。
多个组件匹配成功显示优先级高的，相同显示列表。

> Data是用一个uri对象来表示的，uri代表数据的地址，属于一种标识符。
> 通常情况下，我们使用action+data属性的组合来描述一个意图：做什么

**Intern设置优先级**
通过 `android:priority` 设置。
例如:
```xml
<activity 
    android:name=".SecondActivity">
    <intent-filter android:priority="-1">
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:scheme="http" android:host="www.baidu.com"/>                                  
    </intent-filter>            
</activity>
```

**Intent Type**
对于 `data` 类型的描写。

> Type属性用于明确指定Data属性的数据类型或MIME类型，但是通常来说，当Intent不指定Data属性时，Type属性才会起作用，否则Android系统将会根据Data属性值来分析数据的类型，所以无需指定Type属性.
> 所以，data 和 type 一般只需要一个。使用etData方法会把type属性设置为null，相反使用setType方法会把data设置为null，如果想要两个属性同时设置，要使用Intent.setDataAndType()方法。

**Intent extras**
扩展信息，是其它所有附加信息的集合。使用extras可以为组件提供扩展信息，比如，如果要执行“发送电子邮件”这个动作，可以将电子邮件的标题、正文等保存在extras里，传给电子邮件发送组件。

**Intent Flags**
期望这个intent的运行模式。




## 其他
### JNI
### AIDL
### Handler