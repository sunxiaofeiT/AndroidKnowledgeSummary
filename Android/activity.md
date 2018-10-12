# Activity

## 生命周期

共有七个周期函数，按顺序分别是: onCreate(), onStart(), onRestart(), onResume(), onPause(),onStop(), onDestroy()。

onCreate(): 创建Activity时调用，设置在该方法中，还以Bundle的形式提供对以前存储的任何状态的访问。

onStart(): Activity变为在屏幕上对用户可见时调用。

onResume(): Activity开始与用户交互时调用(无论是启动还是重新启动一个活动，该方法总是被调用）。

onPause(): Activity被暂停或收回cpu和其他资源时调用，该方法用户保护活动状态的，也是保护现场。

onStop(): Activity被停止并转为不可见阶段及后续的生命周期事件时调用。

onRestart(): Activity被重新启动时调用。该活动仍然在栈中，而不是启动新的Activity。

onDestroy(): Activity被销毁时调用。

### 生命周期分类

- 完整生命周期，Activity从出现到消失，onCreate() => onDestory()

- 可见生命周期，可见但不一定可交互，onStart() => onStop()

- 前景生命周期, 处于栈的顶端，可见可交互，onResume() => onPause()


## Activity的启动过程

> 不同于Activity的生命周期

**app启动的过程有两种情况**
- 第一种是从桌面launcher上点击相应的应用图标
- 第二种是在activity中通过调用startActivity来启动一个新的activity。

我们创建一个新的项目，默认的根activity都是MainActivity，而所有的activity都是保存在堆栈中的。
我们启动一个新的activity，这个activity就会放在栈顶的activity上面，而我们从桌面点击应用图标的时候，由于launcher本身也是一个应用，当我们点击图标的时候，系统就会调用startActivitySately(),一般情况下，我们所启动的activity的相关信息都会保存在intent中，比如action，category等等。

我们在安装这个应用的时候，系统也会启动一个PackaManagerService的管理服务，这个管理服务会对AndroidManifest.xml文件进行解析，从而得到应用程序中的相关信息，比如service，activity，Broadcast等等，然后获得相关组件的信息。

当我们点击应用图标的时候，就会调用startActivitySately()方法，而这个方法内部则是调用startActivty(),而startActivity()方法最终还是会调用startActivityForResult()这个方法。
而在startActivityForResult()这个方法，因为startActivityForResult()方法是有返回结果的，所以系统就直接给一个-1，就表示不需要结果返回了。
而startActivityForResult()这个方法实际是通过Instrumentation类中的execStartActivity()方法来启动activity，Instrumentation这个类主要作用就是监控程序和系统之间的交互。
而在这个execStartActivity()方法中会获取ActivityManagerService的代理对象，通过这个代理对象进行启动activity。
启动会就会调用一个checkStartActivityResult()方法，如果说没有在配置清单中配置有这个组件，就会在这个方法中抛出异常了。
当然最后是调用的是Application.scheduleLaunchActivity()进行启动activity，而这个方法中通过获取得到一个ActivityClientRecord对象，而这个ActivityClientRecord通过handler来进行消息的发送，系统内部会将每一个activity组件使用ActivityClientRecord对象来进行描述，而ActivityClientRecord对象中保存有一个LoaderApk对象，通过这个对象调用handleLaunchActivity来启动activity组件，而页面的生命周期方法也就是在这个方法中进行调用。

## 保存Activity状态
`onSaveInstanceState(Bundle)` 会在 activity 转入后台状态之前被调用，也就是 `onStop()` 方法之前，`onPause()` 方法之后被调用