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

（6）Reactor模式

![v2-7eefba893a65706eb6bbe4115cbd0b83_720w](C:\Users\236774\Desktop\v2-7eefba893a65706eb6bbe4115cbd0b83_720w.jpg)