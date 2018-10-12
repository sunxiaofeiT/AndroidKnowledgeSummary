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