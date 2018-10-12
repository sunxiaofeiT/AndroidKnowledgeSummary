# Android架构

![android架构](https://img.zhekoustore.cn/upload/1810/8890cd2aa9835f3a.jpg)

Android系统架构由5部分组成，由下到上分别是：Linux Kernel、Android Runtime、Libraries、Application Framework、Applications。

## Linux Kernel
Android基于Linux 2.6提供核心系统服务，例如：安全、内存管理、进程管理、网络堆栈、驱动模型。Linux Kernel也作为硬件和软件之间的抽象层，它隐藏具体硬件细节而为上层提供统一的服务

## Android Runtime
Android包含一个核心库的集合，提供大部分在Java编程语言核心类库中可用的功能。每一个Android应用程序是Dalvik虚拟机中的实例，运行在他们自己的进程中。Dalvik虚拟机设计成，在一个设备可以高效地运行多个虚拟机。Dalvik虚拟机可执行文件格式是.dex，dex格式是专为Dalvik设计的一种压缩格式，适合内存和处理器速度有限的系统.

## Libraries
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

## Application Framework
所有的应用程序其实是一组服务和系统，包括：
- 视图（View）——丰富的、可扩展的视图集合，可用于构建一个应用程序。包括包括列表、网格、文本框、按钮，甚至是内嵌的网页浏览器
- 内容提供者（Content Providers）——使应用程序能访问其他应用程序（如通讯录）的数据，或共享自己的数据
- 资源管理器（Resource Manager）——提供访问非代码资源，如本地化字符串、图形和布局文件
- 通知管理器（Notification Manager）——使所有的应用程序能够在状态栏显示自定义警告
- 活动管理器（Activity Manager）——管理应用程序生命周期,提供通用的导航回退功能

## Application
各种应用。

## JVM(Java虚拟机)和 Dalvik(Android虚拟机)的区别

### Java虚拟机：
1. java虚拟机基于栈。 基于栈的机器必须使用指令来载入和操作栈上数据，所需指令更多更多。
2. java虚拟机运行的是java字节码。（java类会被编译成一个或多个字节码.class文件）

### Dalvik虚拟机：
1. dalvik虚拟机是基于寄存器的。
2. Dalvik运行的是自定义的.dex字节码格式。(java类被编译成.class文件后，会通过一个dx工具将所有的.class文件转换成一个.dex文件，然后dalvik虚拟机会从其中读取指令和数据)
3. 常量池已被修改为只使用32位的索引，以简化解释器。
4. 一个应用，一个虚拟机实例，一个进程。(所有android应用的线程都是对应一个linux线程，都运行在自己的沙盒中，不同的应用在不同的进程中运行。每个android dalvik应用程序都被赋予了一个独立的linux PID(app_*))

## Android系统中GC什么情况下会出现内存泄露呢

答导致内存泄露主要的原因是，先前申请了内存空间而忘记了释放。

如果程序中存在无用对象的引用，那么这些对象就会驻留内存，消耗内存，因为无法让垃圾回收器GC验证这些对象是否不再需要。

如果存在对象的引用，这个对象就被定义为“有效的活动”，同时不会被释放。要确定对象所占内存将被回收，我们就要确认该对象不会再被使用。典型的做法是把对象数据成员设为null或者从集合中移除该对象。当出现以下情况时，会造成内存泄露:

1. 数据库的cursor没有关闭。

2. 构造adapter时，没有使用缓存contentview。

3. Bitmap对象不使用时，采用recycle()释放内存。

4. Activity中的对象的生命周期大于activity。

调试方法: DDMS==>HEAPSIZE==>dataobject==>[TotalSize]