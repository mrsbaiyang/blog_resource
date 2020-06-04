---
title: parallelStream注意点  
date: 2019-4-11 17:22:36  
tags: java8 并行流  
categories: 技术

---
刚刚的项目中发现了一个bug，是由于使用parallelStream导致，在此记录下。  

### 现象
测试反馈，查询出来的数据不正确，有时候多一个有时候少一个。

### 定位问题
由此看来这肯定是一个比较让人困惑的问题，而能想到的这种随机性问题基本都和并发有关。所以我就想到了，我在一个循环rpc的中使用了parallelStream，如下所示：  
![](https://i.imgur.com/9tBLlMa.png)

### 解惑答疑
使用parallelStream只考虑到了ForkJoinPool最终会等待每个线程执行完毕，但是并没有考虑到线程安全问题，最终返回的结果就导致了list列表里出现了null。更深层次原因是由于ArrayList.add方法并不是线程安全的，如下所示：  
![](https://i.imgur.com/P8Vg2D5.png)  
是先增加了容器大小，然后再赋值，这个操作并不是一个原子操作，导致了此问题。