---
title: IO  
date: 2018-08-16 18:28:30  
tags: aio bio nio select poll epoll 阻塞 非阻塞   
categories: 技术

---
### 先推荐两篇非常不错的文章   
[https://www.cnblogs.com/aspirant/p/6877350.html?utm_source=itdadao&utm_medium=referral](https://www.cnblogs.com/aspirant/p/6877350.html?utm_source=itdadao&utm_medium=referral)  
[https://www.cnblogs.com/aspirant/p/9166944.html](https://www.cnblogs.com/aspirant/p/9166944.html)

### 先给IO分个类
![](https://i.imgur.com/9MDfqPj.png)
我们平时总是提到的一些名词都在图里了，需要关注的是，NIO、IO多路复用，都属于同步IO。  
到这里我们就清晰了，我们最常提的select、poll、epoll其实都是同步IO。

### 再讲讲什么是IO操作
先同步两个概念：  

1. 内核空间（kernel）
2. 用户空间（user）

输入操作一般包含两个步骤：  

1. 等待数据准备好（waiting for data to be ready）。对于一个套接口上的操作，这一步骤关系到数据从网络到达，并将其复制到内核的某个缓冲区。  
2. 将数据从内核缓冲区复制到进程缓冲区（copying the data from the kernel to the process）。

<!-- more -->

### IO多路复用区分

1. select  
内核需要将消息传递到用户空间，都需要内核拷贝动作
2. poll  
同上
3. epoll  
epoll通过内核和用户空间共享一块内存来实现的。  

综上，在选择select，poll，epoll时要根据具体的使用场合以及这三种方式的自身特点。

1. 表面上看epoll的性能最好，但是在连接数少并且连接都十分活跃的情况下，select和poll的性能可能比epoll好，毕竟epoll的通知机制需要很多函数回调。

2. select低效是因为每次它都需要轮询。但低效也是相对的，视情况而定，也可通过良好的设计改善

### 水平触发 边缘触发
1. Level\_triggered(水平触发)  
当被监控的文件描述符上有可读写事件发生时，epoll\_wait()会通知处理程序去读写。如果这次没有把数据一次性全部读写完(如读写缓冲区太小)，那么下次调用 epoll\_wait()时，它还会通知你在上没读写完的文件描述符上继续读写，当然如果你一直不去读写，它会一直通知你！！！如果系统中有大量你不需要读写的就绪文件描述符，而它们每次都会返回，这样会大大降低处理程序检索自己关心的就绪文件描述符的效率！！！

2. Edge\_triggered(边缘触发)  
当被监控的文件描述符上有可读写事件发生时，epoll\_wait()会通知处理程序去读写。如果这次没有把数据全部读写完(如读写缓冲区太小)，那么下次调用epoll\_wait()时，它不会通知你，也就是它只会通知你一次，直到该文件描述符上出现第二次可读写事件才会通知你！！！这种模式比水平触发效率高，系统不会充斥大量你不关心的就绪文件描述符！！！

以上是copy的原话，貌似边缘触发就比水平触发什么都强？我们秉着存在即合理，肯定不是这样的。  
通过上面解释可以看到，边缘触发管线的是变化，如果没有变化，那就不会通知。也就是说，如果前一次没有把数据读完、并且之后也没有变化的话，系统不会通知，这样是不是就表明我们本来应该读的数据只读了一部分？？？而且另一部分能不能读到也是未知的？？？这样肯定是不正确的，所以，使用了**边缘触发**的话，肯定还需要其他的手段来保证数据正确？？？在这里我先打个问号，以后有了更深刻的了解后再解答。