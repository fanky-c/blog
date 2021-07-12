---
title: javascript中event loop原理
date: 2019-05-02 17:13:21
tags:
  - event loop
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
#### 任务队列图
![任务队列](/img/eventloop.png)

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
    console.log(5)
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
   3. 遇到promise 微任务， 直接执行 new Promise 输出 5， 将then函数的回调函数推入 micro Task 的 event queue 中，记为 promise1 
   4. 执行 console.log(10), 输出 10 
   5. 第一轮事件结束，主线程会去检查是否有微任务，会输出then函数 console.log(6)
2. 第二轮事件循环
   1. 首先setTimeout1，输出 2
   2. 然后碰到 promise 微任务， 直接执行 new Promise 输出 3, 同理将 回调函数推入 micro Task 的 event queue 中， 记为 promise2 
   3. 执行 微任务 promise2， 输出 4, 微任务执行完成，
   4. 检查event queue队列是否还有任务没执行，所以任务执行完成

### NodeJS中Event Loop
1. [参考](https://juejin.im/post/5c3d8956e51d4511dc72c200#heading-25)