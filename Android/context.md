# Context

Context是一个抽象基类，翻译为上下文，也可以理解为环境，是提供一些程序的运行环境基础信息。

Context 有两个子类:

- ContextWrapper: 上下文功能的封装类。
    - contextThemeWrapper: 是一个带主题的封装类
        - Activity(直接子类): 需要主题
    - Service
    - Application

- ContextImpl 则是上下文功能的实现类。

Context 一共有三种类型，分别是 Application、Activity 和 Service。 

这三个类虽然分别各种承担着不同的作用，但它们都属于 Context 的一种，而它们具体 Context 的功能则是由 ContextImpl 类去实现的，因此在绝大多数场景下，Activity、Service 和 Application 这三种类型的 Context 都是可以通用的。

不过有几种场景比较特殊，比如启动 Activity，还有弹出 Dialog。 

出于安全原因的考虑，Android 是不允许 Activity 或 Dialog 凭空出现的，一个 Activity 的启动必须要建立在另一个 Activity 的基础之上，也就是以此形成的返回栈。而 Dialog 则必须在一个 Activity 上面弹出（除非是 System Alert 类型的 Dialog），因此在这种场景下，我们只能使用 Activity 类型的 Context，否则将会出错。

> activity 和 service 以及 application 的 context 是不同的，但是大多数情况下 context 是通用的。
> 只有 activity 需要主题，service 不需要主题。
 
getApplicationContext() 和 getApplication() 方法得到的对象都是同一个 application 对象，只是对象的类型不一样。

Context 数量 = Activity 数量 + Service 数量 + 1(1为Application)