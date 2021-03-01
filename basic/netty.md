---
title: netty基础
tags: netty
---

# IO模型

### 同步

任务完成之前不能做其他操作，必须等待

### 异步IO

任务完成之前可以做其他操作，必须等待

### 阻塞IO

任务完成之前挂起当前线程

### 非阻塞IO

不用挂起当前线程

# BIO

一个连接一个线程， 每个请求都需要独立的线程

当并发量大时，需要创建大量线程来处理连接，系统资源占用大

连接建立后，如果当前线程没有数据操作，则线程阻塞，造成资源浪费

# NIO

采用多路复用IO，通过一个线程管理多个socket，通过Selector.select()查询每个通道是否有事件发生，只有在真正有socket读写事件进行时才会使用IO资源，没有就一直阻塞

NIO三大核心：Buffer、Channel、Selector。传统IO基于字节流和字符流进行操作，NIO通过Buffer和Channel进行操作，数据从通道写入缓冲区或者从缓冲区写到通道。IO是面向流的，NIO面向缓冲区

## Channel

双向，类似IO中的Stream(单向)。一个channel对应一个buffer

-  FileChannel:是从文件中读取数据。
-  DatagramChannel:从UDP网络中读取或者写入数据。
-  SocketChannel:从TCP网络中读取或者写入数据。
-  ServerSocketChannel:监听来自TCP的连接，就像服务器一样。每一个连接都会有一个SocketChannel产生。

read(ByteBuffer dst)：从channel读取数据放到buffer

write(ByteBuffer src)：把buffer的数据写到channel

transferFrom：把数据从目标channel复制到当前channel

transferTo：    把数据从当前channel复制到目标channel

## Buffer

缓冲区，本质是一个可读写的内存块。对通道的读写必须经过Buffer

| 属性     | 描述                       |
| -------- | -------------------------- |
| capacity | 容量                       |
| limit    | 缓冲区的当前终点，可以更改 |
| position | 下一个要被操作的索引       |
| mark     | 标记                       |

### 常用方法

flip()：将Buffer设置为读模式，limit=position，position=0，之后向流中写数据

### 其他

ByteBuffer支持类型化的put和get，例如putInt、putLong，但是get的时候必须按照put的顺序，否则可能会抛出BufferUnderflowException异常

## Selector

Selector是一个抽象类，检测多个注册的通道上是否有事件发生

一个selector对应一个线程

### 流程

1. ServerSocketChannel注册OP_ACCEPT事件到Selector上，会有一个SelectionKey与之关联
2. Selector监听select()方法，根据事件进行处理
3. 当有客户端连接时，ServerSocketChannel对应的SelectionKey会返回OP_ACCEPT事件，通过accept返回SocketChannel，并注册读写事件到Selector上
4. 当有读写事件发生时，通过SelectionKey获取SocketChannel进行相应处理

![image-20200316114018024](netty/image-20200316114018024.png)

# 零拷贝

是从操作系统的角度来说的，因为内核缓冲区之间没有数据重复

## 传统IO读写

```java
        File file = new File("index.html");
        RandomAccessFile raf = new RandomAccessFile(file, "rw");

        byte[] arr = new byte[(int) file.length()];
        raf.read(arr);

        Socket socket = new ServerSocket(8080).accept();
        socket.getOutputStream().write(arr);
```

OS底层流程

![image-20200316175346823](netty/image-20200316175346823.png)

4次拷贝，3次切换

1. read导致用户态切换到内核态，同时第一次拷贝开始：DMA从硬盘读取文件放到内核缓冲区
2. 第二次拷贝：将内核缓冲区的数据拷贝到用户缓冲区，并且发生内核态到用户态的切换
3. 第三次拷贝：write将用户缓冲区的数据拷贝到socket缓冲区，并且发生用户态到内核态的切换
4. 第四次拷贝：使用DMA引擎异步的将数据从socket缓冲区拷贝到**网络协议引擎**

## mmap(内存映射)

mmap通过内存映射，将文件映射到内核缓冲区，并且用户空间可以共享内核空间的数据

![image-20200316180341313](netty/image-20200316180341313.png)

适合小数据量；3次数据拷贝，3次上下文切换

如图，user buffer和kernel buffer共享数据，只需要从内核缓冲区直接拷贝到socket缓冲区即可

## sendFile

适合大数据量；

Linux提供了sendFile函数

**Linux2.1**

![image-20200316181728118](netty/image-20200316181728118.png)

数据不经过用户态，直接从内核缓冲区进入到socket缓冲区，3次数据拷贝，2次切换

**Linux2.4**

![image-20200316182043529](netty/image-20200316182043529.png)

利用DMA将数据直接从内核缓冲区拷贝到协议栈，只会拷贝一些length和offset信息到socket缓冲区，基本无损耗；

2次拷贝，2次切换

linux下一个transferTo即可

windows下一次只能发送2G，需要分段

## Netty中的零拷贝

1. Netty通过Direct Buffers，使用堆外内存进行Socket读写，不需要从堆内存拷贝到直接内存
2. Netty提供了组合Buffer，可以聚合多个ByteBuffer对象，避免了传统通过内存拷贝的方式将几个小Buffer合并为一个大Buffer
3. Netty的文件传输采用了transferTo方法，它可以将文件缓冲区的数据发送给目标channel，避免了传统通过循环write方式导致的内存拷贝问题

# Netty

## 为什么要使用

**NIO的缺点**

- 代码编写复杂
- Selector编写复杂
- 要处理断连重连、粘包
- 需要了解多线程和网络编程
- 有BUG

**Netty的好处**

- 非阻塞IO
- 封装了NIO，使用方便
- 零拷贝，效率更高
- 内存池ByteBuf
- 串行无锁化

## Reactor模型

### 单Reactor单线程

![image-20191214105703433](netty/image-20191214105703433.png)

1. Reactor通过select监控客户端事件，收到事件后通过dispatch分发
2. 如果是建立连接请求事件，则由Acceptor通过accept处理，然后创建一个Handler对象处理后续业务
3. 如果不是连接请求，则Reactor会分发调用对应的Handler来响应

优点：模型简单

缺点：只有一个线程，Handler在处理某个连接时整个进程无法处理其他连接事件

适用场景：客户端数量有限，业务处理迅速

### 单Reactor多线程

Acceptor线程负责处理连接事件，线程池负责处理读写事件

![image-20191214111715372](netty/image-20191214111715372.png)

1. 同上
2. 同上
3. 同上
4. Handler只负责响应事件，不做具体业务，read读取数据后，分发给worker线程池做业务处理
5. worker线程池会分配线程进行业务处理，并将结果发送给Handler
6. Handler将结果发给客户端

优点：充分利用多核CPU

缺点：Reactor承担所有事件的监听和响应，高并发情况下容易出现瓶颈

### 主从Reactor多线程

处理连接请求的主Reactor也是一个线程池

![image-20191214112140396](netty/image-20191214112140396.png)

1. Reactor主线程的MainReactor通过select监控连接事件，通过Acceptor处理连接事件
2. Acceptor处理连接事件后，MainReactor将连接分配Reactor子线程给SubReac处理
3. SubReactor将连接加入队列进行监听，并创建一个Handler
4. 当有新的事件发生时，SubReactor调用对应的Handler进行响应
5. Handler通过Read读取数据后，分发给worker线程池
6. 线程池分配线程完成业务处理，将结果发给Handler
7. Handler收到后通过send返回给客户端

优点：Reactor主线程和子线程分工明确，主线程只负责连接，子线程负责处理业务

## EventLoop

EventLoop负责处理Channel上的所有读写事件，一个Channel只会被一个EventLoop处理，而一个EventLoop可以处理多个Channel的事件

## 串行无锁化

`NioEventLoop`维护了一个任务队列，在创建`NioEventLoop`时被初始化，是用来实现串行无锁化的载体。同时封装了一个IO线程，用来处理客户端的连接、读写事件以及处理任务队列中的任务。

调用inEventLoop()判断当前线程是否为创建`NioEventLoop`时绑定的线程，如果是，就直接执行读写操作，如果不是，就把任务放在任务队列中

## ChannelHandler

处理入站和出站数据的应用程序逻辑的容器

## ChannelPipeline

提供了ChannelHandler链的容器

对发送端来说，事件的运动方向是从发送端到接收端是出站，即发送端的数据会经过pipeline中的一系列ChannelOutboundHandler，反之为入站

## 粘包/拆包

粘包：网卡将多个报文段合并发送

拆包：写入的数据块大于套接字缓冲区

### 解决

Netty提供了

1. 行拆包器：基于换行符
2. 固定长度拆包器FixedLengthFrameDecoder
3. 分隔符拆包器
4. 长度域拆包器，在自定义协议中添加长度域字段

## 编码器

- MessageToByteEncoder\<T>

## 解码器

- ByteToMessageDecoder
- MessageToMessageDecoder

















