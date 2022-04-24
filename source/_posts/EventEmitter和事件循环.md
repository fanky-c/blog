---
title: EventEmitter和事件循环
date: 2022-04-19 20:39:35
tags:
 - node事件循环
 - EventEmitter
---

## Node事件循环
>Node.js 是单进程单线程应用程序，但是因为 V8 引擎提供的异步执行回调接口，通过这些接口可以处理大量的并发，所以性能非常高。
Node.js 几乎每一个 API 都是支持回调函数的。
Node.js 基本上所有的事件机制都是用设计模式中观察者模式实现。
Node.js 单线程类似进入一个while(true)的事件循环，直到没有事件观察者退出，每个异步事件都生成一个事件观察者，如果有事件发生就调用该回调函数.

### 事件驱动程序
Node.js 使用事件驱动模型，当web server接收到请求，就把它关闭然后进行处理，然后去服务下一个web请求。
当这个请求完成，它被放回处理队列，当到达队列开头，这个结果被返回给用户。
这个模型非常高效可扩展性非常强，因为 webserver 一直接受请求而不等待任何读写操作。（这也称之为非阻塞式IO或者事件驱动IO）
在事件驱动模型中，会生成一个主循环来监听事件，当检测到事件时触发回调函数。
<img src="/img/event_loop.jpeg" style="max-width:95%" />

### 实例
```js
// 引入 events 模块
var events = require('events');
// 创建 eventEmitter 对象
var eventEmitter = new events.EventEmitter();
 
// 创建事件处理程序
var connectHandler = function connected() {
   console.log('连接成功。');
  
   // 触发 data_received 事件 
   eventEmitter.emit('data_received');
}
 
// 绑定 connection 事件处理程序
eventEmitter.on('connection', connectHandler);
 
// 使用匿名函数绑定 data_received 事件
eventEmitter.on('data_received', function(){
   console.log('数据接收成功。');
});
 
// 触发 connection 事件 
eventEmitter.emit('connection');
 
console.log("程序执行完毕。");

/**
 * node main.js
   连接成功。
   数据接收成功。
   程序执行完毕。
 * /
```



## Node.js EventEmitter
> EventEmitter是Node.js的内置模块events提供的一个类，它是Node事件流的核心；Node.js 所有的异步 I/O 操作在完成时都会发送一个事件到事件队列。
Node.js 里面的许多对象都会分发事件：一个 net.Server 对象会在每次有新连接时触发一个事件， 一个 fs.readStream 对象会在文件被打开的时候触发一个事件。 所有这些产生事件的对象都是 events.EventEmitter 的实例。

### EventEmitter类
events 模块只提供了一个对象： events.EventEmitter。**EventEmitter 的核心就是事件触发与事件监听器功能的封装。**
```js
//event.js 文件
var EventEmitter = require('events').EventEmitter; 
var event = new EventEmitter(); 
event.on('some_event', function() { 
    console.log('some_event 事件触发'); 
}); 
setTimeout(function() { 
    event.emit('some_event'); 
}, 1000);

//node event.js 
// some_event 事件触发
```

### 方法和事件
#### 方法
1. addListener(event, listener)：为指定事件添加一个监听器到监听器数组的尾部(on的同名函数)。
2. on(event, listener)：为指定事件注册一个监听器，接受一个字符串 event 和一个回调函数
3. once(event, listener)：为指定事件注册一个单次监听器，即 监听器最多只会触发一次，触发后立刻解除该监听器
4. removeListener(event, listener)：移除指定事件的某个监听器，监听器必须是该事件已经注册过的监听器。
5. emit(event, [arg1], [arg2], [...])：按监听器的顺序执行执行每个监听器，如果事件有注册监听返回 true，否则返回 false。

#### 事件
1. newListener
2. removeListener

#### 实例
```js
var events = require('events');
var eventEmitter = new events.EventEmitter();

// 监听器 #1
var listener1 = function listener1() {
   console.log('监听器 listener1 执行。');
}

// 监听器 #2
var listener2 = function listener2() {
  console.log('监听器 listener2 执行。');
}

// 绑定 connection 事件，处理函数为 listener1 
eventEmitter.addListener('connection', listener1);

// 绑定 connection 事件，处理函数为 listener2
eventEmitter.on('connection', listener2);

var eventListeners = eventEmitter.listenerCount('connection');
console.log(eventListeners + " 个监听器监听连接事件。");

// 处理 connection 事件 
eventEmitter.emit('connection');

// 移除监绑定的 listener1 函数
eventEmitter.removeListener('connection', listener1);
console.log("listener1 不再受监听。");

// 触发连接事件
eventEmitter.emit('connection');

eventListeners = eventEmitter.listenerCount('connection');
console.log(eventListeners + " 个监听器监听连接事件。");

console.log("程序执行完毕。");
/**
 * node main.js
   2 个监听器监听连接事件。
   监听器 listener1 执行。
   监听器 listener2 执行。
   listener1 不再受监听。
   监听器 listener2 执行。
   1 个监听器监听连接事件。
   程序执行完毕。
 * /
```

#### error 事件
EventEmitter 定义了一个特殊的事件 error，它包含了错误的语义，我们在遇到 异常的时候通常会触发 error 事件。
当 error 被触发时，EventEmitter 规定如果没有响 应的监听器，Node.js 会把它当作异常，退出程序并输出错误信息。
我们一般要为会触发 error 事件的对象设置监听器，避免遇到错误后整个程序崩溃。

### node模块继承EventEmitter
大多数时候我们不会直接使用 EventEmitter，而是在对象中继承它。包括 fs、net、 http 在内的，只要是支持事件响应的核心模块都是 EventEmitter 的子类。

为什么要这样做呢？原因有两点：
1. 具有某个实体功能的对象实现事件符合语义， 事件的监听和发生应该是一个对象的方法。
2. JavaScript 的对象机制是基于原型的，支持 部分多重继承，继承 EventEmitter 不会打乱对象原有的继承关系。