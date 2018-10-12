# ContentProvider

## ContentProvider 是如何实现数据共享的

一个程序可以通过实现一个 `Content provider` 的抽象接口将自己的数据完全暴露出去，而且 `Content provider` 是以类似数据库中的表的方式将自己的数据暴露。

`Content provider` 存储和检索数据，通过它可以让所有的应用程序访问到，这也是应用程序之间唯一共享数据的方法。

要想使应用程序的数据公开化，可通过2种方法：
- 创建一个数据自己的 `Content Provider`
- 将你的数据添加到一个已经存在的 `Content Provider` 中，前提是有**相同数据类型**并且有写入 `Content Provider` 的权限

Android提供了 `Content Resolverr`，外界的程序可以通过 `Content Resolver` 接口访问 `Content Provider` 提供的数据。