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


### 四大组件（生命周期，使用场景，如何启动）

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

### Intent

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

## Android结构

![android架构](https://images0.cnblogs.com/blog/25064/201411/261107127775418.jpg)

Android系统架构由5部分组成，由下到上分别是：Linux Kernel、Android Runtime、Libraries、Application Framework、Applications。

### Linux Kernel
Android基于Linux 2.6提供核心系统服务，例如：安全、内存管理、进程管理、网络堆栈、驱动模型。Linux Kernel也作为硬件和软件之间的抽象层，它隐藏具体硬件细节而为上层提供统一的服务

### Android Runtime
Android包含一个核心库的集合，提供大部分在Java编程语言核心类库中可用的功能。每一个Android应用程序是Dalvik虚拟机中的实例，运行在他们自己的进程中。Dalvik虚拟机设计成，在一个设备可以高效地运行多个虚拟机。Dalvik虚拟机可执行文件格式是.dex，dex格式是专为Dalvik设计的一种压缩格式，适合内存和处理器速度有限的系统.

### Libraries
Android包含一个C/C++库的集合，供Android系统的各个组件使用。这些功能通过Android的应用程序框架（application framework）暴露给开发者。
下面列出一些核心库：
- 系统C库——标准C系统库（libc）的BSD衍生，调整为基于嵌入式Linux设备
- 媒体库——基于PacketVideo的OpenCORE。这些库支持播放和录制许多流行的音频和视频格式，以及静态图像文件，包括MPEG4、 H.264、 MP3、 AAC、 AMR、JPG、 PNG
- 界面管理——管理访问显示子系统和无缝组合多个应用程序的二维和三维图形层
- LibWebCore——新式的Web浏览器引擎,驱动Android浏览器和内嵌的web视图
- SGL——基本的2D图形引擎
- 3D库——基于OpenGL ES 1.0 APIs的实现。库使用硬件3D加速或包含高度优化的3D软件光栅
- FreeType ——位图和矢量字体渲染
- SQLite ——所有应用程序都可以使用的强大而轻量级的关系数据库引擎

### Application Framework
所有的应用程序其实是一组服务和系统，包括：
- 视图（View）——丰富的、可扩展的视图集合，可用于构建一个应用程序。包括包括列表、网格、文本框、按钮，甚至是内嵌的网页浏览器
- 内容提供者（Content Providers）——使应用程序能访问其他应用程序（如通讯录）的数据，或共享自己的数据
- 资源管理器（Resource Manager）——提供访问非代码资源，如本地化字符串、图形和布局文件
- 通知管理器（Notification Manager）——使所有的应用程序能够在状态栏显示自定义警告
- 活动管理器（Activity Manager）——管理应用程序生命周期,提供通用的导航回退功能

### Application
各种应用。


## 其他
### JNI
### AIDL
### Handler