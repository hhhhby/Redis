# 用户空间和内核空间

 - 任何Linux发行版，其系统内核都是Linux。我们的应用都需要通过Linux内核与硬件交互。
<img width="1126" alt="image" src="https://github.com/hhhhby/Redis/assets/113978854/e0b49b5c-8506-4e0a-b36a-a4e33796e4da">

 - 为了避免用户应用导致冲突甚至内核崩溃，用户应用与内核是分离的：
   - 将寻址空间进行隔离，分为**内核空间**和**用户空间**
   - **用户空间**只能执行受限的命令（Ring3)，而且不能直接调用系统资源，必须通过内核提供的接口来访问
   - **内核空间**可以执行特权命令（Ring0)，调用一切系统资源


 - Linux系统为了提高IO效率，会在用户空间和内核空间都加入缓冲区：
   - 写数据时，要把用户缓冲数据拷贝到内核缓冲区，然后写入设备
   - 读数据时，要从设备读取数据到内核缓冲区，然后拷贝到用户缓冲区
  


# 阻塞IO(Bilockng IO)

 - 读取数据的流程（）
   - 1.用户空间读取数据，指令传到内核空间，开始等待数据
   - 2.内核空间去硬件设备准备数据，数据拷贝至内核空间的缓冲区
   - 3.用户空间的缓冲区拷贝内核空间的缓冲区读取数据
  
 - 阻塞IO，这种模式的性能不高
   <img width="845" alt="image" src="https://github.com/hhhhby/Redis/assets/113978854/13458326-90de-47c9-937d-24d9cd60518a">
   - 在等待数据的阶段与从内核拷贝数据到用户控件的阶段，进程都处于阻塞的状态



# 非阻塞IO
 - 非阻塞IO的recvfrom操作会**立即返回结果**而不是阻塞用户进程（没有数据会一直问）
<img width="795" alt="image" src="https://github.com/hhhhby/Redis/assets/113978854/eb837c3d-98ec-4bd0-9210-6bd7aac64d4b">
 - 在从内核拷贝数据到用户控件的阶段，进程处于阻塞的状态
 - 在非阻塞IO模型中，用户进程再第一个阶段是非阻塞，第二个阶段是阻塞状态。虽然是非阻塞，但性能并没有得到提高。而且忙等机制会导致CPU空转，CPU使用率暴增。

# IO多路复用

 - 无论是阻塞IO还是非阻塞IO，用户应用在一阶段都需要调用recvfrom来获取数据，差别在于无数据时的处理方案：
   - 如果调用recvfrom时，恰好**没有**数据，阻塞IO会使进程阻塞，非阻塞IO会使CPU空转，都不能充分发挥CPU的作用。
   - 如果调用recvfrom时，恰好**有**数据，则用户进程可以直接进入第二阶段，读取并处理数据
 - 比如服务器端处理客户端Socket请求时，在单线程情况下，只能依次处理每一个socket，如果正在处理的socket恰好未就绪（数据不可读或不可写），线程就会被阻塞，所有其他客户端socket都必须等待，性能自然会很差。


 - **文件描述符（File Descriptor）**：简称FD,是一个从0开始提增的无符号整数，用来关联Linux中的一个文件。在Linux中，一切皆文件，例如常规文件、视频、硬件设备等，当然也包括网络套接字（Socket）。
 - **IO多路复用**：利用单个线程来同时监听多个FD，并在某个FD可读、可写时得到通知，从而避免无效的等待，充分利用CPU资源。
<img width="784" alt="image" src="https://github.com/hhhhby/Redis/assets/113978854/dead7e70-a8ea-4202-9d9d-d2e1da0ff2e6">

 - 监听FD的方式、通知的方式又有多种实现，常见的有：
   - **select** 
   - **poll**
   - 上面两种是早期的方式，这两种方式只知道这么多FD中有一个或多个FD数据准备就绪，但是不知道是哪个FD，只能挨个询问。
   - **epoll**：这种方式知道哪个FD准备就绪。

## IO多路复用-select
 - select流程（1.表示用户空间，2.表示内核空间）：
   - 1.1 创建一个fd集合fd_set,(大小为1024，初始值都未0)
   - 1.2 假如要监听fd=1,2,5,将对应的fd的值设置为1
   - 1.3 执行select（5+1，rfds，null，null，3）
  
   - 
   <img width="286" alt="image" src="https://github.com/hhhhby/Redis/assets/113978854/b73375be-e184-42a3-8abe-a66822d1219d"></br>
  
   <img width="288" alt="image" src="https://github.com/hhhhby/Redis/assets/113978854/445fa60f-ad71-43d3-ba94-3b599a0abf36"></br>
   - 2.1 遍历fd_set，看看监听的fd有没有数据就绪
   - 2.2 如果没有，则休眠
   - 2.3 等待数据就行被唤醒或超时



 - select模式存在的问题：
   - 需要将整个fd_set从用户空间拷贝到内核空间，select结束还要再次拷贝会用户空间
   - select无法得知具体是哪个fd就绪，需要遍历整个fd_set
   - fd_set监听的fd数量不能超过1024
     
 ## IO多路复用-poll

 poll模式对select模式做了简单改进，但性能提升不明显。
  - 流程：
    - 创建pollfd数组，向其中添加关注的fd信息，数组大小自定义
    - 调用poll函数，将pollfd数组拷贝到内核空间，转链表存储，无上限
    - 内核遍历fd，判断是否就绪
    - 数据就绪或超时后，拷贝pollfd数组到用户空间，返回就绪fd数量n
    - 用户进程判断n是否大于0
    - 大于0则遍历pollfd数组，找到就绪的fd
  - 与select相比
    - fd_set大小固定，而pollfd在内核中采用链表，理论上无上限
    - 监听fd越多，每次遍历消耗时间也越久，性能反而会下降
      
## IO多路复用-epoll

 相对于前两种模式，epoll提供了三个函数：
```
struct eventpoll {
    struct rb_root rbr;      //一棵红黑树，每个节点记录要监听的FD,
    struct list_head rdlist; //一个链表，记录就绪的FD
};
//1.会在内核创建eventpoll结构体，返回对应的句柄epfd
int epoll_create(int_size);
//2.将一个fd添加到epoll的红黑树中，并设置ep_poll_callback
//callback触发时，就把对应的FD加入到rdlist这个就绪列表中
int epoll_ctl(
    int epfd,   //epoll实例的句柄
    int op,     //要执行的操作，包括：ADD、MOD、DEL          
    int fd,     //要监听的FD
    struct epoll_event *event  //要监听的事件类型：读、写、异常等
);
//3.检查rdlist列表是否为空，不为空则返回就绪的FD的数量
int epoll_wait(
    int epfd,     //eventpoll实例的句柄
    struct epoll_event *events,   //空event数组，用于接收就绪的FD
    int maxevents,                //events数组的最大长度
    int timeout  //超时时间，-1为永不超时，0不阻塞，大于0为阻塞时间
);
```
 - epoll模式特点
   - 基于epoll实例中的红黑树保存要监听的FD，理论上无上限，而且增删改查效率都非常高，性能不会随着监听的FD数量增多而下降
   - 每个FD只需要执行一次epoll_ctl添加到红黑树，以后每次epoll_wait无需传递任何参数，无需重复拷贝FD到内核空间
   - 内核会将就绪的FD直接拷贝到用户空间的指定位置，用户进程无需遍历所有FD就能知道就绪的FD是谁

## IO多路复用-事件通知机制
 
当FD有数据可读时，我们调用epoll_wait就可以得到通知。但是时间通知的模式有两种：
 - LevelTriggered：简称LT。当FD有数据可读时，会重复通知多次，直至数据处理完成。是Epoll的默认模式。
   -这种模式可能会导致惊群现象，如果有多个进程在同时监听一个FD，某个时刻FD就绪，一个进程开始执行处理数据，使用LT模式，它就会再通知多次，其他的进程也开始处理数据，第一个进程将数据处理完成，结果其他的还在处理。
 - EdgeTriggered：简称ET。当FD有数据可读时，只会被通知一次，不管数据是否处理完成。

 - 总结：
   - ET模式避免了LT模式可能出现的惊群现象
   - ET模式最好结合非阻塞IO读取FD数据，相比LT会复杂一些。
  
## IO多路复用-web服务流程

<img width="1206" alt="image" src="https://github.com/hhhhby/Redis/assets/113978854/a197db48-9c5f-48e7-b3bb-2845c9dd0825">


# 信号驱动IO
   信号驱动IO是与内核建立SIGIO的信号关联并设置回调，当内核有FD就绪时，会发出SIGIO信号通知用户，期间用户应用可以执行其他业务，无需阻塞等待。
<img width="720" alt="image" src="https://github.com/hhhhby/Redis/assets/113978854/b702efbb-13cc-42c5-a224-666c8135aa02">
 - 缺点
   - 当有大量IO操作时，信号较多，SIGIO处理函数不能及时处理可能导致信号队列溢出
   - 内核空间与用户空间频繁信号交互性能也较低。

# 异步IO
   异步IO的整个过程都是非阻塞的，用户进程调用完异步API后就可以去做其它事情，内核等待数据就绪并拷贝到用户空间后才会递交信号，通知用户进程。
<img width="856" alt="image" src="https://github.com/hhhhby/Redis/assets/113978854/e49a7cb0-f53b-4482-a994-94f334bfbf17">
 - 可以看到，异步IO模型中，用户进程再两个阶段都是非阻塞状态。

# 五个IO模型的比较
<img width="773" alt="image" src="https://github.com/hhhhby/Redis/assets/113978854/a7faed97-2cc7-4969-9387-4181b89aaf99">
 - 前四种都是同步IO，之后最后一个是异步IO
 - IO操作是同步还是异步，关键看数据在内核空间与用户空间的拷贝过程（数据读写的IO操作），也就是阶段二是同步还是异步。





# Redis网络模型

 - Q:Redis是单线程还是多线程？
 - A:
   - 如果仅仅聊Redis的核心业务部分（命令处理），答案是单线程
   - 如果是聊整个Redis，那么答案是多线程
  
 - 在Redis版本迭代过程中，在两个重要的时间节点上引入了多线程的支持：
   - Redis v4.0：引入多线程异步处理一些耗时较长的任务，例如异步删除命令unlink
   - Redis v6.0：在核心网络模型中引入多线程，进一步提高对于多核CPU的利用率

 - Reids使用单线程的原因：
   - Redis是**纯内存操作**（执行速度快的重要原因！），执行速度非常快，它的性能瓶颈是**网络延迟**而不是执行速度，因此多线程并不会带来巨大的性能提升。
   - 多线程会导致**过多的上下文切换**，带来不必要的开销。
   - 引入多线程还会面临**线程安全问题**，必然要引入线程锁这样的安全手段，实现复杂度增高，而且性能也会大打折扣。

 - Redis通过IO多路复用来提高网络性能，并且支持**各种不同的多路复用**实现，并且将这些实现进行封装，提供了统一的高性能事件库API库AE 
 
 - Reids 6.0引入了多线程，目的是为了**提高IO读写效率**。因此在**解析客户端命令、写响应结果**时采用了多线程。核心的命令执行、IO多路复用模块依然是由主线程执行


<img width="1271" alt="image" src="https://github.com/hhhhby/Redis/assets/113978854/6a01dfe4-20c0-443d-a86d-92ee7662896f">

# Redis 通信协议

## RESP协议
在RESP中，通过首字节的字符来区分不同数据类型，常用的数据类型包括5种。
### 数据类型

 - 单行字符串：首字节是'+',后面跟上单行字符串，以CRLF（“\r\n”）结尾。例如返回“OK”:“+OK/r/n”
 - 错误（errors）：首字节是'-'，与单行字符串格式一样，只是字符串是异常信息。例如：“-Error Message\r\n”
 - 数值：首字节是':'，后面跟上数字格式的字符串，以CRLF结尾（与单行字符串相似，只是首字节不同）
 - 多行字符串：首字节是'$'，表示二进制安全的字符串，最大支持512MB
   - 如果大小为0，则代表空字符串:"$0\r\n\r\n"
   - 如果大小为-1，则代表不存在:"$-1\r\n"
   <img width="409" alt="image" src="https://github.com/hhhhby/Redis/assets/113978854/954baf8d-66d6-40a9-b37a-272861f27654">

 - 数组，首字节是'*',后面跟上数组元素个数，再跟上元素，元素数据类型不限
### 模拟Redis客户端
