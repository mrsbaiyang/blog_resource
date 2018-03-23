---
title: websocket http
date: 2018-03-23 20:52:11
tags: http
categories: 技术

---
&emsp;&emsp;前端时间和斌哥吃饭，我就斌哥他们做游戏用的是什么通信协议，斌哥说他们用的是websocket。关于websocket的名字我也是听说过的，但也只是了解到了它是http之上的协议，也没有做过深入的理解。这两天闲下来写，所以就查找了websocket了的资料。
&emsp;&emsp;既然提到了websocket，http就不能绕过去的。websocket的出现就是为了弥补http的不足。我们都知道一个事实，http协议只能由客户端发起，就是说服务器不能主动向客户端发起请求。但是websocket就是一个**全双工**协议，允许服务器主动发送信息给客户端。websocket本身是html5规范的一部分，就是来解决服务器给浏览器发消息的问题（原来只能依靠不断的轮询服务器才能做到的事情），但websocket也不局限于浏览器的使用，任何实现了websocket协议的客户端和服务端都可以用此通信。那么websocket到底和http有啥关系呢？websocket和http都是从tcp之上演化而来的，对于websocket来说，它依赖了http协议进行一次握手（关于为啥非得用http握手我就不太清楚了，可能是最先html5的规范并且肯定要在浏览器使用的原因吧），握手成功后，数据就从tcp通道传输，与http无关了。
&emsp;&emsp;下面我们来聊一聊http，http的历史发展我就不再多说了，网上很多资料写的很清楚，
<!-- more -->[http://www.360doc.com/content/16/0816/07/30578693_583526011.shtml](http://www.360doc.com/content/16/0816/07/30578693_583526011.shtml "http的前世今生")。这里聊聊我的一些困惑。首先现在最最流行的肯定就是http2了，而http2有很多优秀的特性，像服务端推送、header压缩、二进制格式、多路复用等。其他的特性还很容易理解，这里我最困惑的是**多路复用**，因为我知道http1.1就已经有了keep-alive属性，本身已经做到了不是每次都去创建http连接，而是复用上次的连接，而tcp本身的特性也决定了，只能在一次请求完成以后才能进行下一次请求，这也才有了“**head of line blocking**”问题，在我的理解当中，除非tcp本身解决此问题，否则在其之上的更不可能解决的吧？？
&emsp;&emsp;在查找资料的过程中，我也了解到了不少知识，http1.1本身也曾试着解决“head of line blocking”问题，所以提出了http piplelining解决方案，但可能其本身提出的并不是和合理，所以各大浏览器厂商支持并不好[http://www.cnblogs.com/gzchenjiajun-php/articles/4992795.html](http://www.cnblogs.com/gzchenjiajun-php/articles/4992795.html)。这更让我想知道http2是如何解决的呢？方式很简单，将消息分解为更小的独立部分（stream）并通过连接发送。其实tcp本身的特性是没变的，只不过是将消息分解成多份，每部分之间不会相互阻塞，原来发送一个大的请求阻塞的其他请求的情况大大减少。requset stream和response stream是在一个tcp连接上交织在一起的，多个stream构成一个完整的request或者response，但是服务器或者浏览器只能处理完整的请求或者响应，不可能处理一部分的，所以还有另一个事情需要浏览器和服务器去做，那就是将stream重新组装，发送给浏览器渲染或者server的handler处理。以下是我的对于http2对request和response处理的理解：![requset、response分解组装](https://i.imgur.com/ZzO0cGi.png)
***特别注意：以上的请求是发生在一个tcp连接上的，以及request的stream和response的stream都是交织的，在这里是想展示分解、组装的过程。***
&emsp;&emsp;这里我提出一点我的疑问，这个分解组装的过程应该也是增大了客户端和服务端的工作量了吧，但可能是这点消耗对于好处来说是无关紧要的吧。以上是我对于http2以及websocket的理解，不对的话欢迎指教。