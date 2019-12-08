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


### Websocket协议内容

### Websocket使用
#### 客户端使用
```js
const ws = new WebSocket("wss://127.0.0.1"); //ws默认端口:80， wss默认端口:443

ws.onopen = function(evt) { 
  console.log("Connection open ..."); 
  ws.send("Hello WebSockets!");
};

ws.onmessage = function(evt) {
  console.log( "Received Message: " + evt.data);
  ws.close();
};

ws.onclose = function(evt) {
  console.log("Connection closed.");
};
```

### 服务器使用
1. 常用的Node实现有:Socket.IO、WebSocket-Node。