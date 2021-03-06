---
layout: post
book: true
background-image: http://ot1cc1u9t.bkt.clouddn.com/17-7-16/91630214.jpg
category: 操作系统
title: Select/poll/epoll
tags:
- epoll
---
流
==
首先我们来定义流的概念，一个流可以是文件，socket，pipe等等可以进行I/O操作的内核对象。

不管是文件，还是套接字socket，还是管道，我们都可以把他们看作流。

在 I/O 操作里,通过read，我们可以从流中读入数据；通过write,我们可以往流写入数据。

现在假定一个情形,我们需要从流中读数据,但是流中还没有数据,（典型的例子为：用户访问某WEB页面，和服务器建立了 socket 连接。客户端要从socket读数据，但是服务器还没有把数据传回来）,这时候该怎么办？

阻塞
---
比如某个时候你在等快递，但是你不知道快递什么时候过来，而且你没有别的事可以干（或者说接下来的事要等快递来了才能做）;

那么你可以去睡觉了，因为你知道快递把货送来时一定会给你打个电话（假定一定能叫醒你）。

非阻塞忙轮询
---
你知道快递员的手机号,然后每分钟给他挂个电话：“你到了没？”

很明显一般人不会用第二种做法,自己花费大量的精力,还占用了快递员大量的时间。

>第一种方法经济而简单，经济是指消耗很少的CPU时间，如果线程睡眠了，就掉出了系统的调度队列，暂时不会去瓜分CPU宝贵的时间片了。

缓冲区
---
缓冲区的引入是为了减少频繁I/O操作而引起频繁的系统调用），当操作一个流时，更多的是以缓冲区为单位进行操作。对于内核来说，也需要缓冲区,就是内核缓冲区。

假设有一个管道，进程A为管道的写入方，B为管道的读出方。

假设一开始内核缓冲区是空的，B作为读出方，被阻塞着，在等待数据写入。然后首先A往管道写入，这时候内核缓冲区由空的状态变到非空状态，内核就会产生一个事件告诉B该醒来了，这个事件姑且称之为 ***“缓冲区非空”***。

但是“缓冲区非空”事件通知B后，B却还没有读出数据；且内核许诺了不能把写入管道中的数据丢掉,这个时候，A写入的数据会滞留在内核缓冲区中。

如果内核也缓冲区满了，B仍未开始读数据，最终内核缓冲区会被填满，这个时候会产生一个I/O事件，告诉进程A，你该等等（阻塞）了，我们把这个事件定义为 ***“缓冲区满”***。

假设后来B终于开始读数据了，于是内核的缓冲区空了出来，这时候内核会告诉A，内核缓冲区有空位了，你可以从阻塞中醒来了，继续写数据了，我们把这个事件叫做 ***“缓冲区非满”***。

也许事件已经通知了A，但是A也没有数据写入了，而B继续读出数据，直到内核缓冲区空了。这个时候内核就告诉B，你需要阻塞了!我们把这个事件定为 ***“缓冲区空”***。

这四个情形涵盖了四个I/O事件，缓冲区满，缓冲区空，缓冲区非空，缓冲区非满（注都是说的内核缓冲区，且这四个术语都是捏造的，仅为解释其原理而造）。这四个I/O事件是进行阻塞同步的根本。。

阻塞I/O的缺点
---
阻塞I/O模式下，一个线程只能处理一个流的I/O事件。如果想要同时处理多个流，要么多进程(fork)，要么多线程(pthread_create)，很不幸这两种方法效率都不高。

于是再来考虑非阻塞忙轮询的I/O方式，我们发现我们可以同时处理多个流了。

我们只要不停的把所有流从头到尾检查一遍，然后又从头开始。这样就可以处理多个流了，但这样的做法显然不好，因为如果所有的流都没有数据，那么只会白白浪费CPU。

这里要补充一点，阻塞模式下，内核对于I/O事件的处理是阻塞或者唤醒，而非阻塞模式下则把I/O事件交给其他对象（后文介绍的select以及epoll）处理甚至直接忽略。

select/poll
===
为了避免CPU空转，可以引进了一个代理（一开始有一位叫做 select 的代理，后来又有一位叫做 poll 的代理，不过两者的本质是一样的）。这个代理比较厉害，可以同时观察许多流的I/O事件，在空闲的时候，会把当前线程阻塞掉，当有一个或多个流有I/O事件时，就从阻塞态中醒来，然后就会轮询一遍所有的流。

于是，如果没有I/O事件产生，我们的程序就会阻塞在select处。但是依然有个问题，我们从select那里仅仅知道了，有I/O事件发生了，但却并不知道是那几个流（可能有一个，多个，甚至全部），我们只能无差别轮询所有流，找出能读出数据，或者写入数据的流，对他们进行操作。

使用select，同时处理的流越多，每一次轮询时间就越长。所以在高并发的情况下会很慢。
select 最不能忍受的是一个进程所打开的 TCP连接是有一定限制的，由FD_SETSIZE设置，默认值是1024。对于那些需要支持的上万连接数目的IM服务器来说显然太少了。

虽然可以选择多进程的解决方案（传统的Apache方案），不过虽然linux上面创建进程的代价比较小，但仍旧是不可忽视的，加上进程间数据同步远比不上线程间同步的高效，所以也不是一种完美的方案。

假设我们的服务器需要支持100万的并发连接，则在__FD_SETSIZE 为1024的情况下，则我们至少需要开辟1k个进程才能实现100万的并发连接。

select 是用数组存储监视的流，应用程序需要遍历整个数组才能发现哪些流发生了事件。而 poll使用了链表，对监视文件的数量没有限制，扩展方便，但每次一样要遍历才能发现哪些流有事件。

epoll
===
epoll可以理解为event poll，不同于忙轮询和无差别轮询，epoll之会把哪个流发生了怎样的I/O事件通知我们。

假如有100万个客户端同时与一个服务器进程保持着TCP连接。而每一时刻，通常只有几百上千个TCP连接是活跃的(事实上大部分场景都是这种情况)。如何实现这样的高并发？

在select/poll时代，服务器进程每次都把这100万个连接告诉操作系统(从用户态复制句柄数据结构到内核态)，让操作系统内核去查询这些套接字上是否有事件发生，轮询完后，再将句柄数据复制到用户态，让服务器应用程序轮询处理已发生的网络事件，这一过程资源消耗较大，因此，select/poll一般只能处理几千的并发连接。

epoll的设计和实现与select完全不同。epoll通过在Linux内核中申请一个简易的文件系统。把原先的select/poll调用分成了3个部分：

1. 调用epoll_create()建立一个epoll对象(在epoll文件系统中为这个句柄对象分配资源)
2. 调用epoll_ctl()。通过此调用向epoll对象中添加、删除、修改感兴趣的事件
3. 调用epoll_wait收集发生的事件的连接


这样，只需要在进程启动时建立一个epoll对象，然后在需要的时候向这个epoll对象中添加或者删除连接。当某一进程调用epoll_create方法时，Linux内核会创建一个eventpoll结构体，这个结构体中有两个成员与epoll的使用方式密切相关。如:

```
struct eventpoll{  
    ....  
    /*红黑树的根节点，这颗树中存储着所有添加到epoll中的需要监控的事件*/  
    struct rb_root  rbr;  
    /*双链表中则存放着将要通过epoll_wait返回给用户的满足条件的事件*/  
    struct list_head rdlist;  
    ....  
};  
```

每一个epoll对象都有一个独立的eventpoll结构体，用于存放通过epoll_ctl方法向epoll对象中添加进来的事件。这些事件都会挂载在红黑树中，如此，重复添加的事件就可以通过红黑树而高效的识别出来(红黑树的插入时间效率是l<sub>g</sub>n，其中n为树的高度)。

所有添加到epoll中的事件都会与设备(网卡)驱动程序建立回调关系，也就是说，当相应的事件发生时会调用这个回调方法。

在 epoll中，对于每一个事件，都会建立一个epitem结构体，如下所示:

```
struct epitem{  
    struct rb_node  rbn;//红黑树节点  
    struct list_head    rdllink;//双向链表节点  
    struct epoll_filefd  ffd;  //事件句柄信息  
    struct eventpoll *ep;    //指向其所属的eventpoll对象  
    struct epoll_event event; //期待发生的事件类型  
}  
```
当调用epoll_wait检查是否有事件发生时，只需要检查eventpoll对象中的rdlist双链表中是否有epitem元素即可。如果rdlist不为空，则把发生的事件复制到用户态，同时将事件数量返回给用户。

epoll 数据的结构图示如下:

![](/images/epoll.png)

>通过红黑树和双链表数据结构，并结合回调机制，造就了epoll的高效。

而且 epoll 没有 FD_SETSIZE 限制，它所支持的TCP连接数上限是最大可以打开文件的数目，这个数字一般远大于2048,举个例子，在1GB内存的机器上大约是10万左右，具体数目可以cat /proc/sys/fs/file-max 查看.

>在较小的并发中。select 方式在内存使用和即时效率上会稍优于 epoll, 但在现在硬件如此优越的情况下，这种差异可以不考虑到实际中，我们更关心在高并发时的情况。

***epool 的处理方式会更节省CPU使用率。当然，也会更省内存。***

传统的 Apache 服务器就是使用的 select 模型，Nginx 上可以配置成使用 epoll。