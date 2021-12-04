#### 系统方法



## linux系统中五种I/O操作模式

- 阻塞I/O（blocking I/O）
- 非阻塞I/O （nonblocking I/O）
- I/O复用 （I/O multiplexing）
- 信号驱动I/O （signal driven I/O (SIGIO)）
- 异步I/O （asynchronous I/O (the POSIX aio_functions)）

前四种都是同步I/O，只有最后一种是异步I/O。



### 写时复制（copy on write）

https://www.cnblogs.com/lengender-12/p/7054896.html

在linux程序中，fork()会产生一个和父进程完全相同的子进程，但子进程在此后多会exec系统调用，出于效率考虑，linux中引入了“写时复制”技术，也就是只有进程空间的各段的内容要发生变化时，才将父进程的内容复制一份给子进程。

在fork之后exec之前两个进程用的是相同的物理空间(内存区)，子进程的代码段、数据段、堆栈都是指向父进程的物理空间，也就是说，两者的虚拟空间不同，其对应的物理空间是一个。当父子进程中有更改相应段的行为发生时，再为子进程相应的段分配物理空间。如果不是因为exec，内核会给子进程的数据段、堆栈段分配相应的物理空间(至此两者都有各自的进程空间，互不影响)，而代码段继续共享父进程的物理空间(两者的代码完全相同)。而如果是因为exec，由于两者执行的代码不同，子进程的代码段也会分配单独的物理空间。

写时复制技术：内核只为新生成的子进程创建虚拟空间结构，它们复制于父进程的虚拟空间结构，但是不为这些段分配物理内存，它们共享父进程的物理空间，当父子进程中有更改相应的段的行为发生时，再为子进程相应的段分配物理空间。



### 零拷贝

#### 正常的复制

![](C:\Users\236774\Desktop\微信图片_20210809085245.png)

#### 零拷贝（消除了数据从内核到用户态之间的无用copy）

​		从上面的过程可以看出，数据白白从kernel模式到user模式走了一圈，浪费了2次copy(第一次，从kernel模式拷贝到user模式；第二次从user模式再拷贝回kernel模式，即上面4次过程的第2和3步骤)。而且上面的过程中kernel和user模式的上下文的切换也是4次。

​		幸运的是，开发者可以用“零拷贝”技术来去掉这些无谓的复制。**应用程序用Zero-Copy来请求kernel直接把disk的data传输给socket，而不是通过应用程序传输**。Zero-Copy大大提高了应用程序的性能，并且减少了kernel和user模式上下文的切换。

##### linux中零copy的两种方式

https://cloud.tencent.com/developer/article/1744444

1. mmap()

   将用户空间的地址和内核态的地址映射，这样可以将文件数据映射到内核地址空间，直接进行操作，操作完之后再刷回去。弥补了sendfile不能修改数据本身的问题。

2. sendfile() 

   ```javascript
   sendfile() copies data between one file descriptor and another.
   Because this copying is done within the kernel, sendfile() is more efficient than the combination of read(2) and write(2), which would require transferring data to and from user space.
   ```

   缺点：sendfile当传输时修改数据本身，就无能为力了

   零copy案例

   （1）kafka/rocketmq高吞吐：

   （2）netty

### C10K问题

**阻塞I/O**

阻塞：进程会一直阻塞，直到数据拷贝完成。

**非阻塞I/O**

当我们告诉内核，如果数据没有到来，你立马给我返回，不用等待数据了。

**I/O多路复用**

多路复用就是可以在一个线程中监测多个套接字，比如select，poll，epoll，当这些套接字（文件描述符）中的任意一个进入有数据到来，以上三个函数就会返回，之后进入数据处理状态。（就像前台一样）。



##### select系统方法（见https://zhuanlan.zhihu.com/p/57518857）

```
#include <sys/time.h>
#include <sys/types.h>
#include <unistd.h>

int select(int maxfd, fd_set *readfds, fd_set *writefds,
                  fd_set *exceptfds, struct timeval *timeout);
        
    // maxfds：是一个整数值，是指集合中所有文件描述符的范围，即所有文件描述符的最大值加1，不能错。在linux系统中，select的默认最大值为1024。
	// readfs是一个容器，里面可以容纳多个文件描述符，把需要监视的描述符放入这个集合中，当有文件描述符可读时，select就会返回一个大于0的值，表示有文件可读；
	// writefds是一个容器，里面可以容纳多个文件描述符，把需要监视的描述符放入这个集合中，当有文件描述符可写时，select就会返回一个大于0的值，表示有文件可写；、
```

```
select并不会告诉你说，我哪个文件描述符准备好了，他只会告诉你他的那些bit为位哪些是0，哪些是1。也就是说你需要自己用逻辑去判断你要的那个文件描是否准备好了

select的一个缺点，你需要去检测所有的套接字，看看这个套接字到底是谁来的数据。
```



#### poll系统调用（见https://www.cnblogs.com/Anker/archive/2013/08/15/3261006.html）

poll的机制与select类似。

与select在本质上没有多大差别，管理多个描述符也是进行轮询，根据描述符的状态进行处理，但是poll没有最大文件描述符数量的限制。poll和select同样存在一个缺点就是，包含大量文件描述符的数组被整体复制于用户态和内核的地址空间之间，而不论这些文件描述符是否就绪，它的开销随着文件描述符数量的增加而线性增大。

```
# include <poll.h>
int poll ( struct pollfd * fds, unsigned int nfds, int timeout);
// 返回： revents域不为0的文件描述符个数

```

```
struct pollfd {
	int fd;         /* 文件描述符 */
	short events;   /* 等待的事件 */
	short revents;  /* 实际发生了的事件,程序监控的就是这个域 */
} ; 
```

每一个pollfd结构体指定了一个被监视的文件描述符，可以传递多个结构体，指示poll()监视多个文件描述符。每个结构体的events域是监视该文件描述符的事件掩码，由用户来设置这个域。

##### epoll系统调用（https://zhuanlan.zhihu.com/p/165287735）

epoll将原先的一个select或poll调用分成了3部分

```
int epoll_create(int size);
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
int epoll_wait(int epfd, struct epoll_event *events,int maxevents, int timeout);
```

1. 调用epoll_create建立一个epoll对象(在epoll文件系统中给这个句柄分配资源)；

2. 调用epoll_ctl向epoll对象中添加这100万个连接的套接字；

3. 调用epoll_wait收集发生事件的连接（只返回发生了事件的那些个连接，高效许多）。

   **这样只需要在进程启动时建立1个epoll对象，并在需要的时候向它添加或删除连接就可以了**

##### kqueue



##### Reactor事件模型

http://gee.cs.oswego.edu/dl/cpjslides/nio.pdf