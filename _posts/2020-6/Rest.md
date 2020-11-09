---
title: Rest与Http  
date: 2020-06-04 14:02:52  
tags: Rest Http  
categories: 技术

---
## 背景描述
对于Rest其实我也写过一篇文章了，但有很多文章将Rest与Http放在一起说，讲着讲着Rest风格然后就跳到的讲Http的方法（Get、Post等）去了，让我产生了疑惑：Rest就是Http？

## 我不是个例
在网上找Rest文章时，也找到了有外国小哥与我有一样的疑虑，并且也有人给出了他回答：[https://stackoverflow.com/questions/2190836/what-is-the-difference-between-http-and-rest](https://stackoverflow.com/questions/2190836/what-is-the-difference-between-http-and-rest)。  
最重要的就是以下两句话：  

> REST is just a set of rules about how to use HTTP.  
> REST is the way HTTP should be used.

第一句话是提问者的理解，第二句话是回答者给的答案。  

## 总结
我认为，我们通常所说的Rest其实是默认使用了Http协议的：使用Restful风格的Http协议服务。首先我们先搞清，Rest并不是特定给Http使用的，只不过恰好它的风格刚好和Http的协议很契合，在Http协议上面应用最广泛，所以文章最常说的Rest就默认是使用Http，但是没有写明是Http的。下面的截图是我截取的维基百科的内容，个人认为还是很有重点的：
![](https://s1.ax1x.com/2020/06/10/toKHfg.png)
**是风格不是标准**，这句话我们看到了很多次了哈，再极端一点，如果以后Http协议将Rest风格纳入自己的协议当中，那到时候Rest就是风格，并且也是Http标准了哈。但我们反过来想想，为什么Http没有将Rest纳入自己的标准当中呢？我认为是，**标准就必须得遵守了，而风格只是一种建议**，新增这个标准对没有遵守这个标准的服务还是有很大影响的吧。

## 延伸
许多的文章也经常拿Rest与SOAP对比，这里我也找到了一篇不错的文章给大家分享：[https://www.ibm.com/developerworks/cn/webservices/0907_rest_soap/index.html](https://www.ibm.com/developerworks/cn/webservices/0907_rest_soap/index.html)。它对比了基于Http的Rest服务与SOAP服务的区别，还是有一些收获的。  
当前我们写Web服务其实不是Rest风格的，回想下方法的命名还是类似于saveXXXX、getXXX、listXXX等风格，其实我们目前的风格更多的像SOAP。