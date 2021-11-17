rocketmq是干什么的： 异步消息队列，挺像一个分布式的中端系统。类比下linux的中断系统或者中断。

### linux的中断系统

#### 硬中断

上半区处理硬件请求：特点是负责处理耗时端的工作，特点是快速执行。

cat /proc/interrupts 列出当前所以系统注册的中断，记录中断号，中断发生次数，中断设备名称

#### 软中断

下半区负责上半区没有完成的的工作。

软中断不只是包括硬件设备中断处理程序的下半部，一些内核自定义事件也属于软中断。

cat  /proc/softirqs 可以看到系统的软中断类型，比如有：网络接收中断，网络发送中断，定时中断，内核调度等、RCU 锁（内核里常用的一种锁）等

关于linux中断的介绍 https://zhuanlan.zhihu.com/p/338075214?utm_source=wechat_session&utm_medium=social&utm_oi=757528815902662656

top命令中的hi 和si指的就是硬中断和软中断的使用率





像不像“主从Rreactor 多线程模型”mainReactor只负责客户端的连接请求（新连接，断开等），subReactor负责已经成功连接的socket的IO读写



### rocketmq的schame

（要注意schame的演进和向前以及向后的兼容性）



### rocketmq与flink的融合

https://www.infoq.cn/article/ofEHzYoJ4Lyo4Qtfq3zB?utm_source=related_read_bottom&utm_medium=article





### unix管道哲学

类似unix中，使用pipe可以很好的处理数据，比如使用  tail ，sed，awk等

可不可以在日常开发中也利用管道的思想，将业务代码，db，缓存，索引等穿起来。kafka/rocketmq等异步队列可不可以做pipe将业务连接起来。

![1635228104(1)](C:\Users\236774\Desktop\学习\github\myTechNotes\myOwnTechNotesInWondow\异步消息队列\1635228104(1).jpg)

业务方得到一个input，输出一个output，然后向rocketmq发出一个消息。