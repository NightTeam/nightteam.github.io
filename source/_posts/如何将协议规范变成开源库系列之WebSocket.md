---
title: 如何将协议规范变成开源库系列之 WebSocket
categories:
- 实用类库
tags:
- WebSocket
---

这是系列文章的第一篇，也是非常重要的一篇，希望大家能读懂我想要表达的意思。

## 系列文章开篇概述

相对于其他编程语言来说，Python 生态中最突出的就是第三方库。任何一个及格的 Python 开发者都使用过至少 5 款第三方库。

就爬虫领域而言，必将用到的例如网络请求库 Requests、网页解析库 Parsel 或 BeautifulSoup、数据库对象关系映射  Motor 或 SQLAlchemy、定时任务 Apscheduler、爬虫框架 Scrapy 等。


![](https://user-gold-cdn.xitu.io/2019/9/14/16d2dd8be3d2254d?w=1376&h=910&f=png&s=1290872)

这些开源库的使用方法想必大家已经非常熟练了，甚至还修炼出了自己的一套技巧，日常工作中敲起键盘肯定也是哒哒哒的响。

但是你有没有想过：

- 那个神奇的功能是如何实现的？
- 这个功能背后的逻辑是什么？
- 为什么要这样做而不是选择另一种写法？
- 编写这样的库需要用到哪些知识？
- 这个论点是否有明确的依据？


![](https://user-gold-cdn.xitu.io/2019/9/14/16d2dda24b0853ed?w=312&h=312&f=png&s=127644)
如果你从未这样想过，那说明你还没到达应该「渡劫」的时机；如果你曾提出过 3 个以上的疑问，那说明你即将到达那个重要的关口；如果你常常这么想，而且也尝试着寻找对应的答案，那么恭喜你，你现在正处于「渡劫」的关口之上。


![](https://user-gold-cdn.xitu.io/2019/9/14/16d2de0cdaef77c5?w=538&h=282&f=png&s=104780)
偶有群友会抛出这样的问题：初级工程师、中级工程师、高级工程师如何界定？

这个问题有两种不同的观点，第一个是看工作职级，第二个则是看个人能力。工作职级是一个浮动很大的参照物，例如阿里巴巴的高级研发和我司的高级研发，职级名称都是「高级研发」，但能力可能会有很大的差距。

个人能力又如何评定呢？

难不成看代码写的快还是写的慢吗？

当然不是！

个人能力应当从广度和深度两个方面进行考量，这并没有一个明确的标准。当两人能力差异很大的时候，外人可以轻松的分辨孰强孰弱。

自己怎样分辨个人能力的进与退呢？

这就回到了上面提到的那些问题：WHO WHAT WHERE WHY WHEN HOW？

我想通过这篇文章告诉你，不要做那个用库用得很熟练的人，要做那个创造库的人。计算机世界如此吸引人，就是因为我们可以在这个世界里尽情创造。

你想做一个创造者吗？

如果不想，那现在你就可以关掉浏览器窗口，回到 Hub 的世界里。



## 内容介绍

这是一套系列文章，这个系列将为大家解读常见库（例如 WebSocket、HTTP、ASCII、Base64、MD5、AES、RSA）的协议规范和对应的代码实现，帮助大家「知其然，知其所以然」。

### 目标

这次我们要学习的是 WebSocket 协议规范和代码实现，也可以理解为从 0 开始编写 [aiowebsocket](https://github.com/asyncins/aiowebsocket) 库。至于为什么选择它，那大概是因为全世界没有比我更熟悉的它的人了。

![](https://user-gold-cdn.xitu.io/2019/9/14/16d2de2d6c55bdcd?w=900&h=383&f=png&s=222353)

我是 aiowebsocket 库的作者，我花了 7 天编写这个库。写库的过程，让我深刻体会到造轮子和驾驶的区别，也让我有了飞速的进步。我希望用连载系列文章的形式帮助大家从驾驶者转换到创造者，拥有「编程思考」。

### 前置条件

WebSocket 是一种在单个 TCP 连接上进行全双工通信的协议，它的出现使客户端和服务器之间的数据交换变得更加简单。下图描述了双端交互的流程:

![](https://user-gold-cdn.xitu.io/2019/9/14/16d2de4a2f5a3b5d?w=978&h=600&f=png&s=74626)

WebSocket 通常被应用在实时性要求较高的场景，例如赛事数据、股票证券、网页聊天和在线绘图等。WebSocket 与 HTTP 协议完全不同，但同样被广泛应用。

无论是后端开发者、前端开发者、爬虫工程师或者信息安全工作者，都应该掌握 WebSocket 协议的知识。

我曾经发表过几篇关于 WebSocket 的文章：

- [【严选-高质量文章】开发者必知必会的 WebSocket 协议](https://juejin.im/post/5d4cbc0cf265da038f47fa37)
- [Python如何爬取实时变化的WebSocket数据](https://juejin.im/post/5c80b768f265da2dae514d4f)
- [WebSocket 从入门到写出开源库](https://juejin.im/post/5c7cdaabf265da2daf79c15f)

其中，《【严选-高质量文章】开发者必知必会的 WebSocket 协议》介绍了协议规范的相关知识。这篇文章的内容大体如下：

- WebSocket 协议来源
- WebSocket 协议的优点
- WebSocket 协议规范
- 一些实际代码演示

如果没有掌握 WebSocket 协议的朋友，我建议先去阅读这篇文章，尤其是对 [WebSocket 协议规范](https://juejin.im/post/5d4cbc0cf265da038f47fa37#heading-4)介绍的那部分。

要想将协议规范 RFC6455 变成开源库，第一步就是要熟悉整个协议规范，所以你需要阅读[【严选-高质量文章】开发者必知必会的 WebSocket 协议](https://juejin.im/post/5d4cbc0cf265da038f47fa37)。当然，有能力的同学直接阅读 RFC6455 也未尝不可。

接着还需要了解编程语言中内置库 Socket 的基础用法，例如 Python 中的 [socket](https://docs.python.org/3/library/socket.html?highlight=socket#module-socket) 或者更高级更潮的 [Streams](https://docs.python.org/3/library/asyncio-stream.html)、[Transports and Protocols](https://docs.python.org/3/library/asyncio-protocol.html)。如果你是 Go 开发者、Rust 开发者，请查找对应语言的内置库。

假设你已经熟悉了 RFC6455，你应该知道 Frame 打包和解包的时候需要用到位运算，正好我之前写过位运算相关的文章 [7分钟全面了解位运算](https://gitbook.cn/gitchat/activity/5d4d6d8f6f29256fa317e946)。

至于其它的，现用现学吧！

## Python 网络通信之 Streams

WebSocket，也可以理解为在 WEB 应用中使用的 Socket，这意味着本篇将会涉及到 Socket 编程。上面提到，Python 中与 Socket 相关的有 socket、Streams、Transports and Protocols。其中 socket 是同步的，而另外两个是异步的，这俩属于你常听到的 asyncio。

### Socket 通信过程

Socket 是端到端的通信，所以我们要搞清楚消息是怎么从一台机器发送到另一台机器的，这很重要。假设通信的两台机器为 Client 和 Server，Client 向 Server 发送消息的过程如下图所示：

![](https://user-gold-cdn.xitu.io/2019/9/14/16d2b6eccf4d7221?w=1240&h=672&f=jpeg&s=44507)

> Client 通过文件描述符的读写 API  read & write 来访问操作系统内核中的网络模块为当前套接字分配的发送  send buffer 和接收 recv buffer 缓存。
>
> Client 进程写消息到内核的发送缓存中，内核将发送缓存中的数据传送到物理硬件 NIC，也就是网络接口芯片 (Network Interface Circuit)。
>
> NIC 负责将翻译出来的模拟信号通过网络硬件传递到服务器硬件的 NIC。
>
> 服务器的 NIC 再将模拟信号转成字节数据存放到内核为套接字分配的接收缓存中，最终服务器进程从接收缓存中读取数据即为源客户端进程传递过来的 消息。

上述通信过程的描述和图片均出自钱文品的深入理解 RPC 交互流程。

我尝试寻找通信过程中每个步骤的依据（尤其是 send buffer to NIC to recv buffer），（我翻阅了 TCP 的 RFC 和 Kernel.org）但遗憾的是并未找到有力的证明（一定是我太菜了），如果有朋友知道，可以评论告诉我或发邮件 zenrusts@sina.com 告诉我，我可以扩展出另一篇文章。

### 创建 Streams

那么问题来了：在 Python 中，我们如何实现端到端的消息发送呢？

答：Python 提供了一些对象帮助我们实现这个需求，其中相对简单易用的是 Streams。

Streams 是 Python Asynchronous I/O 中提供的 High-level APIs。Python 官方文档对 Streams 的介绍如下：

> Streams are high-level async/await-ready primitives to work with network connections. Streams allow sending and receiving data without using callbacks or low-level protocols and transports.

我尬译一下：Streams 是用于网络连接的 high-level async/await-ready 原语。Streams 允许在不使用回调或 low-level protocols and transports 的情况下发送和接收数据。

Python 提供了 `asyncio.open_connection()` 让开发者创建 Streams，`asyncio.open_connection()` 将建立网络连接并返回 reader 和 writer 对象，这两个对象其实是 StreamReader 和 StreamWriter 类的实例。

开发者可以通过 StreamReader 从 IO 流中读取数据，通过 StreamWriter 将数据写入 IO 流。虽然文档并没有给出 IO 流的明确定义，但我猜它跟 buffer （也就是 send buffer to NIC to recv buffer 中的 buffer）有关，你也可以抽象的认为它就是 buffer。

有了 Streams，就有了端到端消息发送的完整实现。下面将通过一个例子来熟悉 Streams 的用法和用途。这是 Python 官方文档给出的双端示例，首先是 Server 端：

```
# TCP echo server using streams
# 本文出自「夜幕团队 NightTeam」 转载请联系并取得授权
import asyncio

async def handle_echo(reader, writer):
    data = await reader.read(100)
    message = data.decode()
    addr = writer.get_extra_info('peername')

    print(f"Received {message!r} from {addr!r}")

    print(f"Send: {message!r}")
    writer.write(data)
    await writer.drain()

    print("Close the connection")
    writer.close()

async def main():
    server = await asyncio.start_server(
        handle_echo, '127.0.0.1', 8888)

    addr = server.sockets[0].getsockname()
    print(f'Serving on {addr}')

    async with server:
        await server.serve_forever()

asyncio.run(main())
```

接着是 Client 端：

```
# TCP echo client using streams
# 本文出自「夜幕团队 NightTeam」 转载请联系并取得授权
import asyncio

async def tcp_echo_client(message):
    reader, writer = await asyncio.open_connection(
        '127.0.0.1', 8888)

    print(f'Send: {message!r}')
    writer.write(message.encode())

    data = await reader.read(100)
    print(f'Received: {data.decode()!r}')

    print('Close the connection')
    writer.close()

asyncio.run(tcp_echo_client('Hello World!'))
```

将示例分别写入到 server.py 和 client.py 中，然后按序运行。此时 server.py 的窗口会输出如下内容：

```
Serving on ('127.0.0.1', 8888)
Received 'Hello World!' from ('127.0.0.1', 59534)
Send: 'Hello World!'
Close the connection
```

从输出中得知，服务启动的 address 和 port 为 `('127.0.0.1', 8888)`，从 `('127.0.0.1', 59534)` 读取到内容为 `Hello World!` 的消息，接着将 `Hello World!` 返回给  `('127.0.0.1', 59534)` ，最后关闭连接。

client.py 的窗口输出内容如下：

```
Send: 'Hello World!'
Received: 'Hello World!'
Close the connection
```

在创建连接后，Client 向指定的端发送了内容为 `Hello World!`  的消息，接着从指定的端接收到内容为  `Hello World!`  的消息，最后关闭连接。

有些读者可能不太理解，为什么 Client Send  `Hello World!` ，而 Server 接收到之后也向 Client Send  `Hello World!` 。双端的 Send 和 Received 都是  `Hello World!` ，这很容易让新手懵逼。实际上这就是一个普通的回显服务器示例，也就是说当 Server 收到消息时，将消息内容原封不动的返回给 Client。

这样只是为了演示，并无它意，但这样的示例却会给新手带来困扰。

以上是一个简单的 Socket 编程示例，整体思路理解起来还是很轻松的，接下来我们将逐步解读示例中的代码：

    * client.py 中用 `asyncio.open_connection()` 连接指定的端，并获得 reader 和 writer 这两个对象。
    * 然后使用 writer 对象中的 `write()` 方法将 `Hello World!` 写入到 IO 流中，该消息会被发送到 Server。
    * 接着使用 reader 对象中的 `read()` 方法从 IO 流中读取消息，并将消息打印到终端。

看到这里，你或许会有另一个疑问：`write()` 只是将消息写入到 IO 流，并没有发送行为，那消息是如何传输到 Server 的呢？

由于无法直接跟进 CPython 源代码，所以我们无法得到确切的结果。但我们可以跟进 Python 代码，得知消息最后传输到 `transport.write()` ，如果你想知道更多，可以去看 Transports and Protocols 的介绍。你可以将这个过程抽象为上图的 Client to  send buffer to NIC to recv buffer to Server。

## 功能模块设计

通过上面的学习，现在你已经掌握了 WebSocket 协议规范和 Python Streams 的基本用法，接下来就可以设计一个 WebSocket 客户端库了。

根据 RFC6455 的约定，WebSocket 之前是 HTTP，通过「握手」来升级协议。协议升级后进入真正的 WebSocket 通信，通信包含发送（Send）和接收（Recv）。文本消息要在传输过程前转换为 Frames，而接受端读取到消息后要将 Frames 转换成文本。当然，期间会有一些异常产生，我们可能需要自定义异常，以快速定位问题所在。现在我们得出了几个模块：

    * 握手 - ShakeHands
    
    * 传输 - Transports
    
    * 帧处理 - Frames
    
    * 异常 - Exceptions

一切准备就绪后，就可以进入真正的编码环节了。

由于实战编码篇幅太长，我决定放到下一期，这期的内容，读者们可能需要花费一些时间吸收。


## 小结

开篇我强调了「创造能力」有多么重要，甚至抛出了一些不是很贴切的例子，但我就是想告诉你，不要做调参🐶。

然后我告诉你，本篇文章要讲解的是 WebSocket。

接着又跟你说，要掌握 WebSocket 协议，如果你无法独立啃完 RFC6455，还可以看我写过的几篇关于 WebSocket 文章和位运算文章。

过了几分钟，给你展示了 Socket 的通信过程，虽然没有强有力的依据，但你可以假设这是对的。

喝了一杯白开水之后，我向你展示了 Streams 的具体用法并为你解读代码的作用，重要的是将 Streams 与 Socket 通信过程进行了抽象。

这些前置条件都确定后，我又带着你草草地设计了 WebSocket 客户端的功能模块。

下一篇文章将进入代码实战环节，请做好环境（Python 3.6+）准备。

总之，要想越过前面这座山，就请跟我来！
