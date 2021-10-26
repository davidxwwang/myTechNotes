# netty是什么

netty是封装了java nio的一个三方库

## 典型应用

（1）Rocketmq的网络通信

（2）gRPC的网络通信模块、Dubbo的网络通信模块

（3）其他的网络通信模块，基于TCP协议定制自己的私有协议

为什么会出现这个网络通信的三方库

有许多的服务组件需要tcp的网络通信模块，在java中是有原生的nio方法，但是难以应对复杂的网络环境。客户端面临断连重连、网络闪断、半包读写、失败缓存、网络拥塞和异常码流的处理等等。 NIO 编程的特点是功能开发相对容易，但是可靠性能力补齐工作量和难度都非常大。

# 前置知识

## java中的nio

（1）ServerSocketChannel

```
 A selectable channel for stream-oriented listening sockets. 服务端的channel
```

（2）SocketChannel

```
A selectable channel for stream-oriented connecting sockets. 表示一个socket连接channel，双向
```

（3）Selector

```
A multiplexor of {@link SelectableChannel} objects.channel的多路复用器，channel都注册到Selector上
```

（4）SelectionKey

```
A token representing the registration of a {@link SelectableChannel} with a
* {@link Selector}.（表示绑在Selector上的SelectableChannel，与Channel一对一）
```

（5）ByteBuffer

```
A byte buffer.
```

（6）mmap（memory-mapped files）



## 其他

（1）Reactor模式

![v2-7eefba893a65706eb6bbe4115cbd0b83_720w](C:\Users\236774\Desktop\学习\github\myTechNotes\netty.assets\v2-7eefba893a65706eb6bbe4115cbd0b83_720w.jpg)

为什么要有worker group，感觉只需要boss group就可以了啊，别忘记现在的处理器都是多核心的啊！！！

## 涉及到的思想

Reactor模式：像不像银行大堂的服务模式：

有客户去办理业务，首先是大堂经理统一处理，她会告诉你办理业务对应的服务人员，然后你去找那个服务人员办理具体业务，这样办事效率大大提高了。业务办理人员不用感知有客户来，只有大堂经理感知有客户来。

divide-and-conquer





![1634106363(1)](C:\Users\236774\Desktop\学习\github\myTechNotes\netty.assets\1634106363(1).jpg)



## Netty中的主要概念

### IO事件

- InBound事件：

  处理inBound事件实现 ChannelInboundHandler。

- OutBound事件。

  处理Outbound 事件的 handler 需要实现 ChannelOutboundHandler。

  eg：对于客户端来讲， connect 和 write 就是 **out** 事件， read 就是 **in** 事件；对于服务端来讲，  write 就是 **out** 事件， read 就是 **in** 事件

  

  ### DefaultChannelPipeline

  与channel一对一,包含了一系列的ChannelHandler

  ```
  A list of ChannelHandlers which handles or intercepts inbound events and outbound operations of a Channel. ChannelPipeline implements an advanced form of the Intercepting Filter  pattern to give a user full control over how an event is handled and how the ChannelHandlers in a pipeline interact with each other.
  ```

  ![1634200096(1)](C:\Users\236774\Desktop\学习\github\myTechNotes\netty.assets\1634200096(1).jpg)





# 有用的文献：



Doug Lea 关于nio的文章： http://gee.cs.oswego.edu/dl/cpjslides/nio.pdf

https://zhuanlan.zhihu.com/p/181239748



https://mp.weixin.qq.com/s/7PojZyAdB0DflYtNEh3IBw