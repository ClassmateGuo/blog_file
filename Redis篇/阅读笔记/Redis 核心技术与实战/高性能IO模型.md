# 为什么单线程Redis那么快？

## Redis是单线程的嘛？
通常我们说`Redis` 是单线程，主要是指`Redis` 的<font color="red">网络IO </font>和<font color="red">键值对读写</font>是由<font color="red">一个线程</font>来完成的。但是，`Redis` 的其他功能，比如<font color="red">持久化，异步删除，集群数据同步</font>等，都是由<font color="red">额外的线程</font>执行的。

## Redis 为什么用单线程？

**首先，了解下多线程的开销：**

作为程序员，能经常听到一种说法：<font color="red">“使用多线程，可以增加系统吞吐率，或是可以增加系统扩展性。”</font>

>这是当有<font color="red">合理资源分配</font>的情况下，可以增加系统中处理请求操作的资源实体，进而提升系统能够<font color="red">同时处理的请求数</font>（即吞吐率）。但是，通常情况下，当采用多线程后，<font color="red">没有良好的系统设计</font>，实际的结果是：<font color="red">刚开始</font>增加线程数的时候，系统吞吐率会增加，但是，<font color="red">在进一步</font>增加线程数的时候，系统吞吐率的增长反而会降低，甚至下降。

![系统吞吐率结果对比](.pic/2023-02-17-%E7%B3%BB%E7%BB%9F%E5%90%9E%E5%90%90%E7%8E%87%E7%BB%93%E6%9E%9C%E5%AF%B9%E6%AF%94.png)

**这是为什么呢？**

>因为，系统中通常会存在被多线程<font color="red">同时访问</font>的<font color="red">共享资源</font>，当有多个线程要来<font color="red">修改</font>这个共享资源时，为了保证共享资源的<font color="red">正确性</font>，就需要有<font color="red">额外的机制</font>进行保护，就会带来<font color="red">额外的开销</font>(比如锁之类的); 而且，采用多线程开发一般会<font color="red">引入同步原语来保护共享资源的并发访问</font>，这也会降低系统代码的<font color="red">易调试性和可维护性</font>。


为了避免这些问题，<font color="red">Redis 直接采用了单线程模式</font>。

## 所以，单线程Redis 为什么那么快？

 Redis 使用<font color="red">单线程模型</font>能达到<font color="red">每秒数十万级别</font>的处理能力，这是Redis <font color="red">多方面设计选择</font>的一个综合结果：

1. `Redis` 大部分操作都是在内存上；
2. 采用了高效的数据结构；
3. 采用了多路复用机制。


### 基本 IO 模型与阻塞点

首先，以`Get` 请求举个例子: 处理一个`Get` 请求，需要监听客户端请求（`bind/listen`），和客户端建立连接（`accept`），从 `socket` 中读取请求（`recv`），解析客户端发送请求（`parse`），根据请求类型读取键值数据（`get`），最后给客户端返回结果，即向 `socket` 中写回数据（`send`）。

下图显示了这一过程，其中，`bind/listen`、`accept`、`recv`、`parse` 和` send `属于<font color="red">网络 IO 处理</font>，而 <font color="red">get 属于键值数据操作</font>。既然 `Redis` 是单线程，那么，最基本的一种实现是在一个线程中依次执行上面说的这些操作:
![Get请求流程](.pic/2023-02-17-Get%E8%AF%B7%E6%B1%82%E6%B5%81%E7%A8%8B.png)

在上图的网络IO 操作中，`accept()` 和`recv()` 存在阻塞的风险：

- 当 `Redis` 监听到一个客户端有连接请求，但<font color="red">一直未能成功建立起连接时</font>，会阻塞在 `accept()` 函数这里，导致其他客户端无法和 `Redis` 建立连接。

- 当 `Redis` 通过 `recv()` 从一个客户端读取数据时，如果<font color="red">数据一直没有到达</font>，`Redis` 也会一直阻塞在 `recv()`。


### Socket 非阻塞模式

`Socket` 网络模型本身支持非阻塞模式设置。体现在三个关键的函数调用上.

在 `Socket` 模型中，不同的操作调用后会返回不同的<font color="red">套接字</font>：
- `socket()` 返回<font color="red">主动套接字</font>
- `listen()` 返回<font color="red">监听套接字</font>（可设置非阻塞模式）
- `accept()` 返回<font color="red">已连接套接字</font>（可设置非阻塞模式）

对于<font color="red">监听套接字</font>，可以设置非阻塞模式：
- 当`Redis` 调用 `accept()` 但一直没有连接请求到达时，`Redis` 可以返回处理其他请求操作，不需要一直等待（注意，一旦调用`accept()` ，监听套接字就会存在）。

对于<font color="red">已连接套接字</font>，可以设置非阻塞模式：
- 当`Redis` 调用 `recv()` 后，如果已连接套接字上一直没有数据到达，`Redis` 线程同样可以返回处理其他操作。

>注意：虽然 `Redis` 线程可以不用继续等待，但是需要<font color="red">有机制继续监听</font>该<font color="red">监听套接字/已连接套接字</font>，并在有数据达到时通知 `Redis`。

**这个机制是什么呢？接着往下看👇。**

## 基于多路复用的高性能 I/O 模型

`Linux` 中的 IO 多路复用机制是指一个线程处理多个 IO 流（也就是 `select/epoll` 机制）

`Redis` 在只运行单线程的情况下， `select/epoll` 机制允许<font color="red">内核</font>中，同时存在多个监听套接字和已连接套接字。<font color="red">内核</font>会一直监听这些套接字上的<font color="red">连接请求或数据请求</font>，一旦有请求到达，就会交给 `Redis` 线程处理。

![Redis多路复用IO模型](.pic/2023-02-17-Redis%E5%A4%9A%E8%B7%AF%E5%A4%8D%E7%94%A8IO%E6%A8%A1%E5%9E%8B.png)

`Redis` 网络框架调用 `epoll` 机制，让<font color="red">内核</font>监听这些套接字。此时，`Redis` 线程<font color="red">不会阻塞</font>在某一个特定的监听或已连接套接字上，也就是说，<font color="red">不会阻塞</font>在某一个特定的客户端请求处理上。正因为此，`Redis` 可以同时和多个客户端连接并处理请求，从而提升<font color="red">并发性</font>。

### select/epoll基于事件的回调机制

为了在请求到达时能通知到 `Redis` 线程，`select/epoll` 提供了基于事件的回调机制，即针对不同事件的发生，调用相应的处理函数。

`select/epoll` 一旦监测到 `FD` 上有请求到达时，就会触发相应的事件。

这些事件会被放进一个<font color="red">事件队列</font>，`Redis` 单线程对该<font color="red">事件队列</font>不断进行处理。这样一来，`Redis` <font color="red">无需一直轮询</font>是否有请求实际发生，这就可以避免造成 `CPU` 资源浪费。同时，`Redis` 在对事件队列中的事件进行处理时，会调用相应的处理函数，这就实现了基于事件的回调。因为 `Redis` 一直在对事件队列进行处理，所以能及时响应客户端请求，提升 `Redis` 的响应性能。


**为了方便理解，以<font color="red">连接请求</font>和<font color="red">读数据请</font>求为例子，做个具体解释：**
> 
> 这两个请求分别对应 `Accept` 事件和 `Read` `事件，Redis` 分别对这两个事件注册 `accept` 和 `get` 回调函数。当 `Linux` 内核监听到有连接请求或读数据请求时，就会触发 `Accept` 事件和 `Read` 事件，此时，内核就会回调 `Redis` 相应的 `accept` 和 `get` 函数进行处理。

## 总结

`Redis` 单线程是指在它在对网络IO操作和数据读写的时候采用了一个线程，而采用单线程的原因是避免多线程的并发控制；单线程的Redis 性能高，主要体现在以下三点：<font color="red">大部分操作都在内存上</font>，<font color="red">高效的数据结构（哈希表，跳表）</font>，<font color="red">多路复用的I/O 模型</font>。多路复用I/O 模型主要避免了 `accept()` 和 `send()/recv()` 存在的阻塞风险。