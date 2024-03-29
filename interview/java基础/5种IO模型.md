## 什么是I/O

**从计算机结构的视角来看的话， I/O 描述了计算机系统与外部设备之间通信的过程。**

**我们再先从应用程序的角度来解读一下 I/O。**

根据大学里学到的操作系统相关的知识：为了保证操作系统的稳定性和安全性，一个进程的地址空间划分为 **用户空间（User space）** 和 **内核空间（Kernel space ）** 。

像我们平常运行的应用程序都是运行在用户空间，只有内核空间才能进行系统态级别的资源有关的操作，比如如文件管理、进程通信、内存管理等等。也就是说，我们想要进行 IO 操作，一定是要依赖内核空间的能力。

并且，用户空间的程序不能直接访问内核空间。

当想要执行 IO 操作时，由于没有执行这些操作的权限，只能发起系统调用请求操作系统帮忙完成。

因此，用户进程想要执行 IO 操作的话，必须通过 **系统调用** 来间接访问内核空间

我们在平常开发过程中接触最多的就是 **磁盘 IO（读写文件）** 和 **网络 IO（网络请求和相应）**。

**从应用程序的视角来看的话，我们的应用程序对操作系统的内核发起 IO 调用（系统调用），操作系统负责的内核执行具体的 IO 操作。也就是说，我们的应用程序实际上只是发起了 IO 操作的调用而已，具体 IO 的执行是由操作系统的内核来完成的。**

当应用程序发起 I/O 调用后，会经历两个步骤：

1. 内核等待 I/O 设备准备好数据
2. 内核将数据从内核空间拷贝到用户空间。



## 常见I/O模型

 IO 模型一共有 5 种： **同步阻塞 I/O**、**同步非阻塞 I/O**、**I/O 多路复用**、**信号驱动 I/O** 和**异步 I/O**。



### 阻塞式IO

当进程在等待数据时，若该数据一直没有产生，则该进程将一直等待，直到等待的数据产生为止，这个过程中进程的状态是阻塞的。

### 非阻塞式IO

在非阻塞式I/O模型中，当进程等待内核的数据，而当该数据未到达的时候，进程会不断询问内核，直到内核准备好数据。

### IO复用模型

如果一个进程需要等到多种不同的消息，那么一般的做法就是开启多条线程，每个线程接收一类消息，如果每个线程都是采用阻塞式I/O模型，那么每个线程在消息未产生的时候就会阻塞，也就是说在多线程中使用阻塞式I/O。I/O复用就是基于上述的场景中，无需采用多线程监听消息的方式，进程直接监听所有的消息类型，这其中就涉及到select、poll、epoll等不同的方法。



### 信号驱动IO

在信号驱动式I/O模型中，与阻塞式和非阻塞式有了一个本质的区别，那就是用户态进程不再等待内核态的数据准备好，直接可以去做别的事情。

当需要等待数据的时候，首先用户态会向内核发送一个信号，告诉内核我要什么数据，然后用户态就不管了，做别的事情去了，而当内核态中的数据准备好之后，内核立马发给用户态一个信号，说”数据准备好了，快来查收“，用户态进程收到之后，立马调用recvfrom，等待数据从内核空间复制到用户空间，待完成之后recvfrom返回成功指示，用户态进程才处理别的事情。

> 信号驱动式I/O模型有种异步操作的赶脚，但是在将数据从内核复制到用户空间这段时间内用户态进程是阻塞的



### 异步IO模型

如上图，首先用户态进程告诉内核态需要什么数据（上图中通过aio_read），然后用户态进程就不管了，做别的事情，内核等待用户态需要的数据准备好，然后将数据复制到用户空间，此时才告诉用户态进程，”数据都已经准备好，请查收“，然后用户态进程直接处理用户空间的数据。

> 在复制数据到用户空间这个时间段内，用户态进程也是不阻塞的





### IO模型比较

![img](D:%5CProgram%20Files%5Ctypora%5CNotebook%5Csource%5C1470567-20180911212315919-1077504877.png)







## IO多路复用中的select，poll，epoll

多路复用：**同一个线程内同时处理多个IO请求的目的**，多路是多个IO请求，复用是一个线程。

IO多路复用是一种同步IO模型，一个线程监听多个IO事件，当有IO事件就绪时，就会通知线程去执行相应的读写操作，没有就绪事件时，就会阻塞交出cpu。多路是指网络链接，复用指的是复用同一线程。

#### select  

```c
fd_set数据结构定义如下，可以看出fd_set是一个整型数组，用于保存socket文件描述符
typedef long int __fd_mask;

/* fd_set for select and pselect.  */
typedef struct
  {
#ifdef __USE_XOPEN
    __fd_mask fds_bits[__FD_SETSIZE / __NFDBITS];

# define __FDS_BITS(set) ((set)->fds_bits)

#else
    __fd_mask __fds_bits[__FD_SETSIZE / __NFDBITS];

# define __FDS_BITS(set) ((set)->__fds_bits)

#endif
  } fd_set;
```



执行过程

![img](D:/Program%20Files/typora/Notebook/source/e2bf6fc915be2b8d0b15d128a77cee65.png)

流程：

> 1. 用户线程调用select，将fd_set从用户空间拷贝到内核空间
> 2. 内核在内核空间对fd_set遍历一遍，检查是否有就绪的socket描述符，如果没有的话，就会进入休眠，直到有就绪的socket描述符
> 3. 内核返回select的结果给用户线程，即就绪的文件描述符数量
> 4. 用户拿到就绪文件描述符数量后，再次对fd_set进行遍历，找出就绪的文件描述符
> 5. 用户线程对就绪的文件描述符进行读写操作

优点

- 所有平台都支持，良好的跨平台性

缺点

- 每次调用select，都需要将fd_set从用户空间拷贝到内核空间，当fd很多时，这个开销很大

- 最大连接数（支持的最大文件描述符数量）有限制，一般为1024

- 每次有活跃的socket描述符时，都需要遍历一次fd_set，造成大量的时间开销，时间复杂度是O(n)

- 将fd_set从用户空间拷贝到内核空间，内核空间也需要对fd_set遍历一遍

  

#### poll

数据结构

数据结构定义如下，链表存储

```c
/* Data structure describing a polling request.  */
struct pollfd
  {
    int fd;			/* File descriptor to poll.  */
    short int events;		/* Types of events poller cares about.  */
    short int revents;		/* Types of events that actually occurred.  */
  };
```


与select的异同点

相同点：

- 内核线程都需要遍历文件描述符，并且当内核返回就绪的文件描述符数量后，还需要遍历一次找出就绪的文件描述符
- 需要将文件描述符数组或链表从用户空间拷贝到内核空间
- 性能开销会随文件描述符的数量而线性增大
  

不同点：

- select存储的数据结构是文件描述符数组，poll采用链表
- select有最大连接数限制，poll没有最大限制，因为poll采用链表存储
  

执行过程（基本与select类型）

> - 用户线程调用poll系统调用，并将文件描述符链表拷贝到内核空间
> - 内核对文件描述符遍历一遍，如果没有就绪的描述符，则内核开始休眠，直到有就绪的文件描述符
> - 返回给用户线程就绪的文件描述符数量
> - 用户线程再遍历一次文件描述符链表，找出就绪的文件描述符
> - 用户线程对就绪的文件描述符进行读写操作

#### epoll

核心代码

```c
#include <sys/epoll.h>
    
// 数据结构
// 每一个epoll对象都有一个独立的eventpoll结构体
// 红黑树用于存放通过epoll_ctl方法向epoll对象中添加进来的事件
// epoll_wait检查是否有事件发生时，只需要检查eventpoll对象中的rdlist双链表中是否有epitem元素即可
struct eventpoll {
    ...
    /*红黑树的根节点，这颗树存储着所有添加到epoll中的需要监控的事件*/
    struct rb_root  rbr;
    /*双链表存储所有就绪的文件描述符*/
    struct list_head rdlist;
    ...
};
```

> // API
> int epoll_create(int size); // 内核中间加一个 eventpoll 对象，把所有需要监听的 socket 都放到 eventpoll 对象中
> int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event); // epoll_ctl 负责把 socket 增加、删除到内核红黑树
> int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);// epoll_wait 检测双链表中是否有就绪的文件描述符，如果有，则返回

核心点：

- epoll_create创建eventpoll对象（红黑树，双链表）
- 一棵红黑树，存储监听的所有文件描述符，并且通过epoll_ctl将文件描述符添加、删除到红黑树
- 一个双链表，存储就绪的文件描述符列表，epoll_wait调用时，检测此链表中是否有数据，有的话直接返回
- 所有添加到eventpoll中的事件都与设备驱动程序建立回调关系
  

缺点

- 只能工作在linux下
  

优点

- 时间复杂度为O(1)，当有事件就绪时，epoll_wait只需要检测就绪链表中有没有数据，如果有的话就直接返回
- 不需要从用户空间到内核空间频繁拷贝文件描述符集合，使用了内存映射(mmap)技术
- 当有就绪事件发生时采用回调的形式通知用户线程









## select poll epoll比较

![image-20210427140909162](D:/Program%20Files/typora/Notebook/source/image-20210427140909162.png)



> 注：
>
> - epoll 的水平触发（LT）和边缘触发（ET）的区别
>
>   LT模式：只要文件描述符还有数据可读，每次epoll_wait就会返回它的事件（**只要有数据就触发**）
>
>   ET模式：只有数据流到来的时候才触发，不管缓冲区是否还有数据（**只有数据流到来才会触发**）
>
>   在绝大多少情况下，ET模式并不会比LT模式更为高效，同时，ET模式带来了不好理解的语意，这样容易造成编程上面的复杂逻辑和坑点。因此，建议还是采用LT模式来编程更为舒爽。
>
> - 应用场景
>
>   1. 连接数较少并且都很活跃,用select和poll效率更高
>   2. 连接数较多并且都不很活跃,使用epoll效率更高
>
>   
>
>   