# View

## View 绘制流程

## 自定义控件

1. 组合控件。这种自定义控件不需要我们自己绘制，而是使用原生控件组合成的新控件。如标题栏。

2. 继承原有的控件。这种自定义控件在原生控件提供的方法外，可以自己添加一些方法。如制作圆角，圆形图片。

3. 完全自定义控件：这个View上所展现的内容全部都是我们自己绘制出来的。比如说制作水波纹进度条。

## View的绘制流程

OnMeasure() ——> OnLayout() ——> OnDraw()

- OnMeasure()：测量视图大小。从顶层父 View 到子 View 递归调用 measure() 方法，measure() 方法又回调 OnMeasure()。

- OnLayout()：确定View位置，进行页面布局。从顶层父 View 向子 View 的递归调用 view.layout() 方法的过程，即父 View 根据上一步 measure 子 View 所得到的布局大小和布局参数，将子 View 放在合适的位置上。

- OnDraw()：绘制视图。ViewRoot 创建一个 Canvas 对象，然后调用 OnDraw()。

    - 绘制视图的背景
    - 保存画布的图层(Layer)
    - 绘制View的内容
    - 绘制View子视图，如果没有就不用
    - 还原图层(Layer)
    - 绘制滚动条。

## View，ViewGroup 事件分发

1. Touch 事件分发中只有两个主角: ViewGroup 和 View。ViewGroup 包含 onInterceptTouchEvent、dispatchTouchEvent、onTouchEvent 三个相关事件。View 包含 dispatchTouchEvent、onTouchEvent 两个相关事件。其中 ViewGroup 又继承于 View。

2. ViewGroup 和View 组成了一个树状结构，根节点为 Activity 内部包含的一个 ViwGroup。

3. 触摸事件由 Action_Down、Action_Move、Aciton_UP 组成，其中一次完整的触摸事件中，Down 和 Up 都只有一个，Move 有若干个，可以为 0 个。

4. 当 Acitivty 接收到 Touch 事件时，将遍历子 View 进行 Down 事件的分发。ViewGroup 的遍历可以看成是递归的。分发的目的是为了找到真正要处理本次完整触摸事件的 View，这个 View 会在 onTouchuEvent() 结果返回 true。

5. 当某个子 View 返回 true 时，会中止 Down 事件的分发，同时在 ViewGroup 中记录该子 View。接下去的 Move 和 Up 事件将由该子 View 直接进行处理。由于子View 是保存在 ViewGroup 中的，多层 ViewGroup 的节点结构时，上级 ViewGroup 保存的会是真实处理事件的 View 所在的 ViewGroup 对象。

如 ViewGroup0-ViewGroup1-TextView 的结构中，TextView 返回了 true ，它将被保存在 ViewGroup1 中，而 ViewGroup1 也会返回 true，被保存在 ViewGroup0 中。当 Move 和 UP 事件来时，会先从 ViewGroup0 传递至 ViewGroup1 ，再由 ViewGroup1 传递至 TextView。

6. 当 ViewGroup 中所有子 View 都不捕获 Down 事件时，将触发 ViewGroup 自身的 onTouch() 事件。触发的方式是调用 super.dispatchTouchEvent() 函数，即父类 View 的 dispatchTouchEvent() 方法。在所有子 View 都不处理的情况下，触发 Acitivity 的 onTouchEvent() 方法。

7. onInterceptTouchEvent 有两个作用：
    1.拦截 Down 事件的分发。
    2.中止 Up 和 Move 事件向目标 View 传递，使得目标 View 所在的 ViewGroup 捕获 Up 和 Move 事件。

## 理解 Activity，View，Window 三者关系

Activity 像一个工匠（控制单元），Window 像窗户（承载模型），View 像窗花（显示视图），LayoutInflater 像剪刀，Xml 配置像窗花图纸。

1. Activity 构造的时候会初始化一个 Window，准确的说是 PhoneWindow。

2. 这个 PhoneWindow 有一个 ViewRoot，这个 ViewRoot 是一个 View 或者说 ViewGroup ，是最初始的根视图。

3. ViewRoot 通过 addView() 方法来一个个的添加 View。比如 TextView，Button 等。

4. 这些 View 的事件监听，是由 WindowManagerService 来接受消息，并且回调 Activity 函数。比如 onClickListener，onKeyDown 等。