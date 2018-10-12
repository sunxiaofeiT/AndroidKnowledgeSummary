# Intent 

Intent这个英语单词的本意是“目的、意向、意图”。

Android中提供了Intent机制来协助应用间的交互与通讯，或者采用更准确的说法是，Intent不仅可用于应用程序之间，也可用于应用程序内部的activity, service和broadcast receiver之间的交互。

## Intent Filter

Intent Filter 是指意图过滤，不出现在代码中，而是出现在 android Manifest 文件中，以 `<intent-filter>` 的形式。

> 有一个例外是broadcast receiver的intent-filter是使用Context.registerReceiver()来动态设定的，其中intent filter也是在代码中创建的

一个 `隐式intent` 有 action，data，category 等字段,一个 `隐式intent` 为了能够被某个 `intent filter` 接收，必须通过所有的 filter，一个 `intent` 为了被某个组件接收，则必须通过它所有的 `intent filter` 中的一个。

Intent是一种运行时绑定（runtime binding) 机制，它能在程序运行的过程中连接两个不同的组件。通过Intent，你的程序可以向 Android 表达某种请求或者意愿，Android 会根据意愿的内容选择适当的组件来响应。 

> activity、service和broadcast receiver之间是通过Intent进行通信的，而另外一个组件Content Provider本身就是一种通信机制，不需要通过Intent.

## Intent 相关属性

## Intent 组成部分

- component(组件)：目标组件
- action（动作）：用来表现意图的行动
- category（类别）：用来表现动作的类别
- data（数据）：表示与动作要操纵的数据
- type（数据类型）：对于data范例的描写
- extras（扩展信息）：扩展信息
- Flags（标志位）：期望这个意图的运行模式

## Intent类型

- 显式Intent（直接类型）
- 隐式Intent（间接类型）。

> 直接使用component声明intent目标的为显示Intent，使用action、category等属性为隐式Intent。
> 官方建议使用隐式 Intent。

## Intent Action

当日常生活中，描述一个意愿或愿望的时候，总是有一个动词在其中。比如：我想“做”三个俯卧撑；我要“写” 一封情书，等等。

在Intent中，Action就是描述做、写等动作的，当你指明了一个 Action，执行者就会依照这个动作的指示，接受相关输入，表现对应行为，产生符合的输出。在Intent类中，定义了一批量的动作，比如 ACTION_VIEW，ACTION_PICK 等，基本涵盖了常用动作。加的动作越多，越精确。

Action 是一个用户定义的字符串，用于描述一个 Android 应用程序组件，一个 Intent Filter 可以包含多个 Action。在 AndroidManifest.xml 的Activity 定义时，可以在其 <intent-filter >节点指定一个 Action列表用于标识 Activity 所能接受的“动作”

## Intent Category

Category属性也是作为<intent-filter>子元素来声明的。

> Action 和category通常是放在一起用的。

## Intent 例子

```xml
<activity 
    android:name=".SecondActivity">
    <intent-filter>
        <action android:name="com.example.smyh006intent01.MY_ACTION"/>
        <category android:name="android.intent.category.DEFAULT" />
    </intent-filter>            
</activity>
```

上方代码，表示 SecondActicity 可以匹配第4行的 MY_ACTION 这个动作，此时，如果在其他的 Acticity 通过这个 action 的条件来查找，那 SecondActicity 就具备了这个条件。类似于相亲时，我要求对方有哪些条件，然后对方这个 SecondActicity 恰巧满足了这个条件。

也就是说：只有 `<action>` 和 `<category>` 中的内容同时能够匹配上 Intent 中指定的 action 和 category 时，这个活动才能响应 Intent。如果使用的是DEFAULT 这种默认的 category，在稍后调用 startActivity() 方法的时候会自动将这个 category 添加到 Intent 中。

> 如果没有指定的 category，则必须使用默认的 DEFAULT（即上方第5行代码）。
> 如果有多个 activity 匹配成功，则会让用户进行选择。（在某个应用中选择分享的时候可以选择多个应用）
> 每个 Intent 只能指定一个 Action，但是能设置多个 Category，类别越多，动作越具体，意图越明确。

## Intent data

表示与动作要操作的数据。

Data 属性是Android要访问的数据，和 action 和 Category 声明方式相同，也是在 `<intent-filter>` 中。
多个组件匹配成功显示优先级高的，相同显示列表。

> Data是用一个 uri 对象来表示的，uri 代表数据的地址，属于一种标识符。
> 通常情况下，我们使用 action+data 属性的组合来描述一个意图：做什么。

## Intern 设置优先级

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

## Intent Type

对于 `data` 类型的描写。

> Type属性用于明确指定Data属性的数据类型或MIME类型，但是通常来说，当Intent不指定Data属性时，Type属性才会起作用，否则Android系统将会根据Data属性值来分析数据的类型，所以无需指定Type属性.
> 所以，data 和 type 一般只需要一个。使用etData方法会把type属性设置为null，相反使用setType方法会把data设置为null，如果想要两个属性同时设置，要使用Intent.setDataAndType()方法。

## Intent extras

扩展信息，是其它所有附加信息的集合。使用 extras 可以为组件提供扩展信息，比如，如果要执行“发送电子邮件”这个动作，可以将电子邮件的标题、正文等保存在extras 里，传给电子邮件发送组件。

## Intent Flags

> TODO

期望这个intent的运行模式。