---
title: javascript中event loop原理
date: 2019-05-02 17:13:21
tags:
  - event loop
  - 内存堆
  - 调用栈
---

### 介绍
Event Loop即事件循环，是指浏览器或Node的一种解决javaScript单线程运行时不会阻塞的一种机制，也就是我们经常使用异步的原理。

### js任务广义分类

#### 同步任务
1. 同步任务：进入主线程，一个一个执行。

#### 异步任务
1. 异步任务：进入  `event table ` , 注册回调函数 ` callback `, 任务完成之后，将 `callback` 移入  `event queue`， 等待主线程调用。

### js任务细致分类

#### 宏任务（MacroTask）
   1. setTimeout、setInterval、script、I/O、UI Rendering

#### 微任务（MicroTask）,优先于宏任务
   1. Process.nextTick（Node独有）、Promise

### 浏览器中Event Loop

#### 事件循环
<img src="/img/eventloop.webp" />
*** 事件循环 负责执行代码、收集和处理事件以及执行队列中的子任务: ***
1. Javascript 有一个主线程和执行栈，所有的任务都会被放到调用栈等待主线程执行
2. 同步任务会被放在调用栈中，按照顺序等待主线程依次执行
3. 主线程之外存在一个任务队列，所有任务在主线程中以执行栈的方式运行
4. 同步任务都在主线程上执行，栈中代码在执行的时候会调用 Web API，此时会产生一些异步任务
5. 异步任务会在有了结果（比如被监听的事件发生时）后，将注册的回调函数放入任务队列中
6. 执行栈中任务执行完毕后，此时主线程处于空闲状态，会从任务队列中获取任务进行处理

以上过程会不断重复，这就是浏览器的运行机制，也是Event Loop


**更加详细的流程：**

```js
JavaScript中的事件循环机制基于异步执行，用于处理非阻塞 I/O 操作、定时器等异步任务。以下是事件循环机制的基本流程：

1. 执行全局同步代码： 当 JavaScript 程序开始执行时，首先执行全局同步代码，将同步任务推入执行栈（Call Stack）中。

2. 执行栈（Call Stack）： 执行栈是一个存储函数调用的栈结构。每当调用一个函数，它会被推入执行栈的顶部；
当函数执行完毕，它会从栈顶弹出。执行栈是同步执行的。

3. 执行异步任务： 当遇到异步任务时，将异步任务的回调函数推入任务队列（Task Queue）中。
异步任务包括定时器、事件监听器、Ajax 请求等。

4. 事件循环开始： 当执行栈为空时，事件循环开始工作。
事件循环会检查任务队列是否有任务。如果有，将任务推入执行栈中执行。

5. 执行异步任务回调： 异步任务的回调函数在执行栈中执行。
如果异步任务中有新的异步任务，它们会被推入任务队列中，等待下一轮事件循环。

6. 重复： 重复以上过程，形成一个循环。每轮事件循环称为一个 tick。
```

#### 任务队列图
<img src="/img/event_loop.png"  alt="任务队列" height="auto"/>

#### 案例
```js
console.log(1);
setTimeout(() => {
    console.log(2)
    new Promise((resolve, reject) => {
        console.log(3)
        resolve()
    }).then(() => {
        console.log(4)
    })
}, 0)

new Promise((resolve, reject) => {
    console.log(5); // 为啥5会立即执行？---> 虽然位于Promise的构造函数中，但在这个例子构造函数没有任何触发异步操作的地方，所以它也会同步执行。
    resolve()
}).then(() => {
    console.log(6)
})

console.log(10)

// 1 5 10 6 2 3 4

```
1. 第一轮事件循环
   1. 进入主线程遇到console.log(1), 输出 1
   2. 遇到setTimeout,将其回调函数推入macro task的event queue， 并且标记任务setTimeout1
   3. 遇到promise 微任务， 直接执行 new Promise的构造函数 输出 5， 将then函数的回调函数推入 micro Task 的 event queue 中，记为 promise1
   4. 执行 console.log(10), 输出 10
   5. 第一轮事件结束，主线程会去检查是否有微任务，会输出then函数 console.log(6)
2. 第二轮事件循环
   1. 首先setTimeout1，输出 2
   2. 然后碰到 promise 微任务， 直接执行 new Promise 输出 3, 同理将 回调函数推入 micro Task 的 event queue 中， 记为 promise2
   3. 执行 微任务 promise2， 输出 4, 微任务执行完成，
   4. 检查event queue队列是否还有任务没执行，所以任务执行完成

### NodeJS中Event Loop

Node.js 的事件循环机制建立在 V8 引擎之上，并且采用了 libuv 库来处理事件循环。以下是 Node.js 事件循环机制的基本原理：

1、Node.js 的单线程： Node.js 是单线程的，但它通过事件循环机制实现了异步操作，使其能够处理大量并发请求。

2、事件循环的阶段： Node.js 的事件循环被分为不同的阶段。每个阶段都有一个 FIFO 队列，用于存放相应的回调函数。事件循环的阶段包括：

  1. timers（定时器）： 执行 setTimeout 和 setInterval 中到期的回调函数。
  2. pending callbacks： 执行某些系统操作的回调函数，如TCP错误之类的操作。
  3. idle, prepare： 仅用于内部操作。
  4. poll（轮询）： 处理 I/O 事件。当轮询队列不为空时，事件循环会遍历队列执行回调。如果轮询队列为空且有 setImmediate 回调，事件循环会直接进入下一阶段。
  5. check： 执行 setImmediate 的回调函数。
  6. close callbacks： 执行关闭的回调函数，例如 socket.on('close', ...)。

3、事件循环的执行过程： 当 Node.js 启动时，它会执行全局脚本，并且事件循环开始。每个阶段的执行都是一个循环。在每个循环中，事件循环会执行当前阶段的所有回调函数，然后移动到下一个阶段。

4、异步操作的触发： 在 Node.js 中，异步操作的触发通常包括 I/O 操作、定时器到期、以及其他异步操作（如 setImmediate、process.nextTick 等）。

5、触发异步操作后的回调： 当异步操作完成后，相应的回调函数会被推入到相应的事件队列中，等待事件循环执行。

6、事件队列和回调的执行： 每个阶段都有一个事件队列，用于存放相应阶段的回调函数。当事件循环进入某个阶段时，会依次执行该阶段队列中的回调函数。

总体来说，Node.js 事件循环机制通过事件队列和不同的阶段来实现异步非阻塞的执行模型。这使得 Node.js 能够高效地处理大量并发请求，而不会阻塞整个应用程序。

<br>
文章来源：
1. [参考1](https://juejin.im/post/5c3d8956e51d4511dc72c200#heading-25)
2. [参考2](https://mp.weixin.qq.com/s/omqXH1SxJyvl7N8y-6Zp3Q)