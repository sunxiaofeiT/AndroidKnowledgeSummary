# AIDL

`AIDL（Android 接口定义语言）` 是 Android 提供的一种进程间通信 (IPC) 机制。

我们可以利用它定义客户端与服务使用进程间通信 (IPC) 进行相互通信时都认可的编程接口。

在 Android 上，一个进程通常无法访问另一个进程的内存。 尽管如此，进程需要将其对象分解成操作系统能够识别的原语，并将对象编组成跨越边界的对象。

编写执行这一编组操作的代码是一项繁琐的工作，因此 Android 会使用 AIDL 来处理。

通过这种机制，我们只需要写好 `aidl` 接口文件，编译时系统会帮我们生成 `Binder` 接口。

## AIDL 支持的四种数据类型

- Java 的基本数据类型
- List 和 Map 
    - 元素必须是 AIDL 支持的数据类型。
    - Server 端具体的类里则必须是 ArrayList 或者 HashMap
- 其他 AIDL 生成的接口
- 实现 Parcelable 的实体

## AIDL 如何编写

AIDL 的编写主要为以下三部分：

- 创建 AIDL  
    - 创建要操作的实体类，实现 Parcelable 接口，以便序列化/反序列化
    - 新建 aidl 文件夹，在其中创建接口 aidl 文件以及实体类的映射 aidl 文件
    - Make project ，生成 Binder 的 Java 文件
- 服务端  
    - 创建 Service，在其中创建上面生成的 Binder 对象实例，实现接口定义的方法在 onBind() 中返回
- 客户端 
    - 实现 ServiceConnection 接口，在其中拿到 AIDL 类
    - bindService()
    - 调用 AIDL 类中定义好的操作请求