

## 网络相关命令：

### netstat

```
netstat -ano 查看所有端口状态
netstat -ano | grep "8004" 查看8004端口状态
```

 



## TCP相关

Nagle Algorithm的问题

 为了解决tcp中传输小包的问题，例如每次传输一个字符，tcp如果也马上发到网络中，效率比较低（因为tcp的header最少也是20字节，还需要对方为了这一个字节回复一个ack，而这个ack又有可能在网络中丢失）TCP总是希望尽可能的发送足够大的数据。（一个连接会设置MSS参数，因此，TCP/IP希望每次都能够以MSS尺寸的数据块来发送数据）。Nagle算法就是为了尽可能发送大块数据，避免网络中充斥着许多小数据块。

 若设置这个算法，最晚会等待200ms，frame才会发出去（所以想想，对于一个get/post请求，有可能需要等待200ms,才能发出去，那么是不是可以关闭这个算法，可以节约200ms，但这可能会引起网络阻塞的）

TCP调优 ： https://blog.liu-kevin.com/2021/01/04/linuxzhi-wang-luo/

linux中关于tcp的文档

https://man7.org/linux/man-pages/man7/tcp.7.html

### tcp中的定时器

#### （1）保活定时器

目的：探测对方是不是正常，是不是不可达，是不是已经崩溃，是不是崩溃后重启，

#### （2）坚持定时器

#### （3）超时重传定时器

## DNS相关

## IP相关

### TSL相关

使用wireshark看tsl： https://www.jianshu.com/p/a3a25c6627ee

### IPV4

### IPV6



