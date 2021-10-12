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