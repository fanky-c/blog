---
title: Websocket协议
date: 2019-12-08 14:29:39
tags:
    - websocket协议
---


### Websocket介绍
HTML5开始提供的一种浏览器与服务器进行全双工通讯的网络技术，属于应用层协议。基于TCP这些应用程序需要与服务器进行双向通信，而不依赖于打开多个HTTP连接。


### Websocket优缺点
#### 优点
1. 支持双向通信，实时性更强。
2. 可以发送文本，也可以发送二进制数据。
3. 较少的控制开销，通信性能更高。http1.x协议每次通信都需要携带完整的头部；websocket协议头部包含的较小。
4. 允许跨域

#### 缺点
1. 对于客户端来说兼容性差。
2. 对服务器来说开发者要求更高，长连接需要服务业务更加稳定（不能随便把进程和框架crash）。


### Websocket协议流程
#### 握手阶段
1. WebSocket客户端的握手是一个HTTP Upgrade请求
2. Sec-WebSocket-Key（请求头）以及Sec-WebSocket-Accept（响应头）。 目的：Websocket协议需要保证客户端发起的Websocket连接请求只会被能理解Websocket协议的服务器所识别。
```js
//相比传统的http多了这个2个值
Upgrade: websocket
Connection: Upgrade
```



#### 数据传输阶段
客户端与服务器之间互相传输数据的的基本单位根据规格说明书里我们称为“Messages”。在实际网络中，这些Message由一个或多个Frames组成。解决了http的被动性、无状态性。


### Websocket使用
#### 客户端使用
```js
const ws = new WebSocket("wss://127.0.0.1"); //ws默认端口:80， wss默认端口:443

//建立连接
ws.onopen = function(evt) { 
  console.log("Connection open ..."); 
  ws.send("Hello WebSockets!");
};

//接受服务器信息调用
ws.onmessage = function(evt) {
  console.log( "Received Message: " + evt.data);
  ws.close();
};

//连接出错时候调用
ws.onerror = function(){

}

//连接终止是调用
ws.onclose = function(evt) {
  console.log("Connection closed.");
};
```

#### 服务器使用
1. 常用的Node实现有:Socket.IO、WebSocket-Node。


#### 检测心跳
1. 在使用websocket的过程中，有时候会遇到客户端网络关闭的情况，而这时候在服务端并没有触发onclose事件。这样会：
   1. 多余的连接
   2. 服务端会继续给客户端发数据


#### 身份认证
1. 大体上Websocket的身份认证都是发生在握手阶段，通过请求中的内容来认证。一个常见的例子是在url中附带参数token。

#### 解决ws与wss共存
1. nginx配置


#### websocket和http2服务器推送的区别
1. websocket是全双工同学， 消息可以直接推送给webapp, 也就是说webapp有API来获取服务器推送的数据
2. http2虽然也支持server push，但是服务器只会主动把资源推送到客户端缓存，并不直接推送到webapp本身。 也可以说webapp本身没有直接响应这些数据的API接口。