> 本文基于图解HTTP此书进行学习，感谢原作者以及翻译人员的辛勤劳动。

#  WebSocket

## websocket 是什么？

> WebSocket 协议是一种位于应用层的通信协议，由于 Http 有以下等固有瓶颈：
>
> * 判断页面是否更新，需要频繁地从客户端到服务器端进行确认，可能产生大量无用请求。
> * 一条连接上只可发送一个请求。
> * 请求只能从客户端开始。客户端不可以接收除响应以外的指令。
> * 请求 / 响应首部未经压缩就发送。首部信息越多延迟越大。
> * 发送冗长的首部。每次互相发送相同的首部造成的浪费较多。
> * 可任意选择数据压缩格式。非强制压缩发送。
>
> 等等。
>
> **PS：Web Socket != Socket！！**
>
> ![](https://raw.githubusercontent.com/HoshiSrar/Note_Images/main/img/20231124120509.png)
>
> 用于解决 Http 协议本身瓶颈，一系列技术或协议由此诞生，WebSocket 就是其中一种，它是建立在 HTTP 基础上的协议。

##  WebSocket 的设计与功能 

WebSocket，即 Web 浏览器与 Web 服务器之间全双工通信协议（意味着服务端和客户端任意一端均可发起请求）。其中，WebSocket 协议由 IETF 定为标准，WebSocket API 由 W3C 定为 标准。仍在开发中的 WebSocket 技术主要是为了解决 Ajax 和 Comet 里 XMLHttpRequest 附带的缺陷所引起的问题。

一旦 Web 服务器与客户端之间建立起 WebSocket 协议的通信连接， 之后所有的通信都依靠 Web Socket 协议进行。通信过程中可互相发送 JSON、XML、HTML或图片等任意格式的数据。 由于是建立在 HTTP 基础上的协议，因此连接的发起方仍是客户端， 而一旦确立 WebSocket 通信连接，不论服务器还是客户端，任意一方都可直接向对方发送报文。

**主要特点：**

* 推送功能

  支持由服务器向客户端推送数据的推送功能。这样，服务器可直接发 送数据，而不必等待客户端的请求。

* 减少通信量

  只要建立起 WebSocket 连接就会一直保持连接状态。和 HTTP 相比，不但每次连接时的总开销减少，而且由于 WebSocket 的首部信息很小，通信量也相应较少。

为了实现 WebSocket 通信，在 HTTP 连接建立之后，需要完成一次“握手”（Handshaking）的步骤。

> * 握手·请求
>
> 为了实现 WebSocket 通信，需要用到 HTTP 的 Upgrade 首部字 段，告知服务器通信协议发生改变，以达到握手的目的。
>
> ![](https://raw.githubusercontent.com/HoshiSrar/Note_Images/main/img/20231124123330.png)
>
> * 握手·响应
>
> ![](https://raw.githubusercontent.com/HoshiSrar/Note_Images/main/img/20231124123450.png)

## WebSocket API

JavaScript 可调用 “The WebSocket API”（http://www.w3.org/TR/websockets/，由 W3C 标准制定）内提供的 WebSocket 程序接口，以实现 WebSocket 协议下全双工通信。 以下为调用 WebSocket API 的一个例子，每 50ms 发送一次数据。

~~~javascript
var socket = new WebSocket('www.localhost:8080/updates');
socket.onopen = function () {
	setInterval(function() {
		if (socket.bufferedAmount == 0)
			socket.send(getUpdateData());
	}, 50);
};
~~~

