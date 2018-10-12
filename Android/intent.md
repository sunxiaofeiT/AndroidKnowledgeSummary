# Intent 

Intent在Android中被翻译为”意图”.

是三种应用程序基本组件-Activity，Service和broadcast receiver之间相互激活的手段。

在调用Intent名称时使用ComponentName也就是类的全名时称作`显式调用`。  
这种方式一般用于应用程序的内部调用，因为你不一定会知道别人写的类的全名。

Intent Filter是指意图过滤，不出现在代码中，而是出现在android Manifest文件中，以<intent-filter>的形式。

> 有一个例外是broadcast receiver的intent-filter是使用Context.registerReceiver()来动态设定的，其中intent filter也是在代码中创建的

一个 `隐式intent` 有 action，data，category 等字段,一个 `隐式intent` 为了能够被某个 `intent filter` 接收，必须通过3个测试，一个 `intent` 为了被某个组件接收，则必须通过它所有的 `intent filter` 中的一个。