# Android JNI

## 概念

JNI 全称 Java Native Interface，Java 本地化接口，可以通过 JNI 调用系统提供的 API。

操作系统，无论是 Linux，Windows 还是 Mac OS，或者一些汇编语言写的底层硬件驱动都是 C/C++ 写的。Java和C/C++不同 ，它不会直接编译成平台机器码，而是编译成虚拟机可以运行的Java字节码的.class文件，通过JIT技术即时编译成本地机器码，所以有效率就比不上C/C++代码，JNI技术就解决了这一痛点，JNI 可以说是 C 语言和 Java 语言交流的适配器、中间件。

![JNI调用示意图](https://img.zhekoustore.cn/upload/1810/92975c76cf9f92f2.png)

## JNI 与 NDK 区别

JNI：JNI是一套编程接口，用来实现Java代码与本地的C/C++代码进行交互。

NDK: NDK是Google开发的一套开发和编译工具集，可以生成动态链接库，主要用于Android的JNI开发。

## JNI 作用

1. 扩展：JNI扩展了JVM能力，驱动开发，例如开发一个wifi驱动，可以将手机设置为无限路由。

2. 高效： 本地代码效率高，游戏渲染，音频视频处理等方面使用JNI调用本地代码，C语言可以灵活操作内存。

3. 复用： 在文件压缩算法 7zip开源代码库，机器视觉 OpenCV开放算法库等方面可以复用C平台上的代码，不必在开发一套完整的Java体系，避免重复发明轮子。

4. 特殊： 产品的核心技术一般也采用JNI开发，不易破解。

## JNI在Android中作用

JNI可以调用本地代码库(即C/C++代码)，并通过 Dalvik 虚拟机与应用层和应用框架层进行交互，Android 中 JNI 代码主要位于应用层和应用框架层。

应用层: 该层是由 JNI 开发，主要使用标准 JNI 编程模型。

应用框架层: 使用的是 Android 中自定义的一套 JNI 编程模型，该自定义的 JNI 编程模型弥补了标准 JNI 编程模型的不足。

## JNI 数据类型映射

jni.h 头文件为了让 C/C++ 类型和 Java 原始类型相匹配的头文件定义。

当是 C++ 环境时，jobject, jclass, jstring, jarray 等都是继承自 _jobject 类，而在 C 语言环境是，则它的本质都是空类型指针 typedef void* jobject;