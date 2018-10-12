# ContentProvider

`Content Provider` 是Android系统提供的在多个应用之间共享数据的一种机制。

一个Content Provider类实现了一组标准的方法接口，从而能够让其他的应用保存或读取此Content Provider的各种数据类型。

每个ContentProvider都会对外提供一个公共的URI（包装成Uri对象），如果应用程序有数据需要共享，就需要使用ContentProvider为这些数据定义一个URI，然后其他应用程序就可以通过ContentProvider传入这个URI来对数据进行操作。

我们的APP可以通过实现一个Content Provider的抽象接口将自己的数据暴露出去，也可以通过ContentResolver接口访问Content Provider提供的数据。

ContentResolver支持CRUD(create, retrieve, update, and delete)操作。

Android系统提供了诸如：音频、视频、图片、通讯录等主要数据类型的Content Provider。我们也可以创建自己的Content Provider。

## URI

URI唯一标识了 Provider 中的数据，当应用程序访问 Content Provider 中的数据时，URI 将会是其中一个重要参数。

URI包含了两部分内容：
- 要操作的 Content Provider 对象
- 要操作的 Content Provider 中数据的类型

### URI组成

1. Scheme：在 Android 中 URI 的 Scheme 是不变的，即：Content://
2. Authority：用于标识 ContentProvider（API原文：A string that identifies the entire content provider.）；
3. Path：用来标识我们要操作的数据。零个或多个段，用正斜杠（/）分割;
4. ID：指定 ID 后，将操作指定 ID 的数据（ ID 也可以看成是 path 的一部分），ID 在 URI 中是可选的（API原文：A unique numeric identifier for a single row in the subset of data identified by the preceding path part.）。

### URI示例
（1）content://media/internal/images        ->  返回设备上存储的所有图片;
（2）content://media/internal/images /10    ->  返回设备上存储的ID为10的图片 

### 操作 URI 经常用到的 UriMatcher 和 ContentUris 两个类

UriMatcher：用于匹配Uri。
ContentUris：用于操作Uri路径后面的 ID 部分，如提供了方法 withAppendedId() 向 URI 中追加 ID。

**注意**

1. 访问 Content Provider 需要一定的操作权限。
2. 访问 Content Prvider 需要使用到 ContentResolver 对象。
3. ContentResolver 支持 query，insert，delete，update 操作。
4. 由 URI 确定 Content Provider 中要操作的具体数据。
5. Insert 时，要添加的数据可以使用 ContentValues 封装。

## ContentProvider 是如何实现数据共享的

一个程序可以通过实现一个 `Content provider` 的抽象接口将自己的数据完全暴露出去，而且 `Content provider` 是以类似数据库中的表的方式将自己的数据暴露。

`Content provider` 存储和检索数据，通过它可以让所有的应用程序访问到，这也是应用程序之间唯一共享数据的方法。

要想使应用程序的数据公开化，可通过2种方法：
- 创建一个数据自己的 `Content Provider`
- 将你的数据添加到一个已经存在的 `Content Provider` 中，前提是有**相同数据类型**并且有写入 `Content Provider` 的权限

Android提供了 `Content Resolverr`，外界的程序可以通过 `Content Resolver` 接口访问 `Content Provider` 提供的数据。