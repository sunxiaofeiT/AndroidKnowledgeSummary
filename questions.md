# Some Questiions

搜集面试经验中的问题。

## 生命周期相关

### 两个Activity之间跳转时必然会执行的是哪几个方法。

两个Activity之间跳转必然会执行的是下面几个方法。

- onCreate()//在Activity生命周期开始时调用。
- onRestoreInstanceState()//用来恢复UI状态。
- onRestart()//当Activity重新启动时调用。
- onStart()//当Activity对用户即将可见时调用。
- onResume()//当Activity与用户交互时，绘制界面。
- onSaveInstanceState()//即将移出栈顶保留UI状态时调用。
- onPause()//暂停当前活动Activity，提交持久数据的改变，停止动画或其他占用GPU资源的东西，由于下一个Activity在这个方法返回之前不会resume，所以这个方法的代码执行要快。
- onStop()//Activity不再可见时调用。
- onDestroy()//Activity销毁栈时被调用的最后一个方法。


### 横竖屏切换时候Activity的生命周期。

- 不设置Activity的android: configChanges时，切屏会重新调用各个生命周期，切横屏时会执行一次，切竖屏时会执行两次。
- 设置Activity的android: configChanges=“orientation”时，切屏还是会重新调用各个生命周期，切横、竖屏时只会执行一次。
- 设置Activity的android: configChanges=“orientation|keyboardHidden”时，切屏不会重新调用各个生命周期，只会执行onConfiguration方法



## 样式相关

### 如何将一个Activity设置成窗口的样式。

- 第一种方法，在styles.xml文件中，可以新建如下的类似Dialog的style。
```css
<style name=“Theme.FloatActivity” parent=“android:style/Theme.Dialog”> </style>。
```
- 在AndroidManifest.xml中在需要显示为窗口的Activity中添加如下属性:  android: theme=“@style/Theme.FloatActivity”即可。也可以直接添加对应需要展示为Dialog style的Activity的android: theme属性为android: theme=“@ android: style/Theme.Dialog”。


## Activity相关

### 两个Activity之间怎么传递数据？

可以在Intent对象中利用Extra来传递存储数据。
在Intent的对象请求中，使用putExtra(“键值对的名字”，”键值对的值”)；在另外一个Activity中将Intent中的请求数据取出来:

```java
Intent intent = getIntent();
String value = intent.getStringExtra(“testIntent”);
```


### 怎么让在启动一个Activity是就启动一个service？

首先定义好一个service，然后在Activity的onCreate里面进行连接并bindservice或者直接startService.


###  Activity怎么和service绑定，怎么在activity中启动自己对应的service？

- activity能进行绑定得益于Serviece的接口。为了支持Service的绑定，实现onBind方法。
- Service和Activity的连接可以用ServiceConnection来实现。需要实现一个新的ServiceConnection，重现onServiceConnected和OnServiceDisconnected方法，一旦连接建立，就能得到Service实例的引用。
- 执行绑定，调用bindService方法，传入一个选择了要绑定的Service的Intent(显示或隐式)和一个你实现了的ServiceConnection的实例


## 其他

### 进程保活（不死进程、唤醒应用）

当前业界的Android进程保活手段主要分为 **黑、白、灰** 三种。

#### 黑色保活
不同的app进程，用广播相互唤醒（包括利用系统提供的广播进行唤醒）

所谓黑色保活，就是利用不同的app进程使用广播来进行相互唤醒。举个3个比较常见的场景：
场景1：开机，网络切换、拍照、拍视频时候，利用系统产生的广播唤醒app
场景2：接入第三方SDK也会唤醒相应的app进程，如微信sdk会唤醒微信，支付宝sdk会唤醒支付宝。由此发散开去，就会直接触发了下面的 场景3
场景3：假如你手机里装了支付宝、淘宝、天猫、UC等阿里系的app，那么你打开任意一个阿里系的app后，有可能就顺便把其他阿里系的app给唤醒了。（只是拿阿里打个比方，其实BAT系都差不多）

#### 白色保活
启动前台Service
白色保活手段非常简单，就是调用系统api启动一个前台的Service进程，这样会在系统的通知栏生成一个Notification，用来让用户知道有这样一个app在运行着，哪怕当前的app退到了后台。如下方的LBE和QQ音乐这样.

#### 灰色保活
利用系统的漏洞启动前台Service。

灰色保活，这种保活手段是应用范围最广泛。它是利用系统的漏洞来启动一个前台的Service进程，与普通的启动方式区别在于，它不会在系统通知栏处出现一个Notification，看起来就如同运行着一个后台Service进程一样。这样做带来的好处就是，用户无法察觉到你运行着一个前台进程（因为看不到Notification）,但你的进程优先级又是高于普通后台进程的。那么如何利用系统的漏洞呢，大致的实现思路和代码如下：

思路一：API < 18，启动前台Service时直接传入new Notification()。
思路二：API >= 18，同时启动两个id相同的前台Service，然后再将后启动的Service做stop处理。

熟悉Android系统的童鞋都知道，系统出于体验和性能上的考虑，app在退到后台时系统并不会真正的kill掉这个进程，而是将其缓存起来。  
打开的应用越多，后台缓存的进程也越多。在系统内存不足的情况下，系统开始依据自身的一套进程回收机制来判断要kill掉哪些进程，以腾出内存来供给需要的app。  
这套杀进程回收内存的机制就叫 `Low Memory Killer` ，它是基于Linux内核的 `OOM Killer（Out-Of-Memory killer）`机制诞生。

进程的重要性，划分5级：
前台进程 (Foreground process)
可见进程 (Visible process)
服务进程 (Service process)
后台进程 (Background process)
空进程 (Empty process)
 
#### 什么是oom_adj
它是linux内核分配给每个系统进程的一个值，代表进程的优先级，进程回收机制就是根据这个优先级来决定是否进行回收。  

对于oom_adj的作用，你只需要记住以下几点即可：
- 进程的oom_adj越大，表示此进程优先级越低，越容易被杀回收；
- 越小，表示进程优先级越高，越不容易被杀回收
- 普通app进程的oom_adj>=0,系统进程的oom_adj才可能<0
- 有些手机厂商把这些知名的app放入了自己的白名单中，保证了进程不死来提高用户体验（如微信、QQ、陌陌都在小米的白名单中）。如果从白名单中移除，他们终究还是和普通app一样躲避不了被杀的命运，为了尽量避免被杀，还是老老实实去做好优化工作吧。
所以，进程保活的根本方案终究还是回到了性能优化上，进程永生不死终究是个彻头彻尾的伪命题！

### 说说 Activity，Intent，Service 是什么关系

一个Activity通常是一个单独的屏幕，每一个Activity都被实现为一个单独的类，这些类都是从Activity基类中继承而来的。Activity类会显示由视图控件组成的用户接口，并对视图控件的事件做出响应。

Intent的调用是用来进行屏幕之间的切换。Intent描述应用想要做什么。Intent数据结构中两个最重要的部分是动作和动作对应的数据，一个动作对应一个动作数据。

Service是运行在后台的代码，不能与用户交互，可以运行在自己的进程里，也可以运行在其他应用程序进程的上下文里。需要一个Activity或者其他Context对象来调用。

Activity跳转Activity，Activity启动Service，Service打开Activity都需要Intent表明意图，以及传递参数，Intent是这些组件间信号传递的承载者。