---
title: js异步
date: 2018-08-12 20:49:47
tags: 
    - 异步
    - settimeout
    - Promise
    - async await    
---

### js异步使用方法：

#### setTimeout/setInterval

###### setTimeout

```js
function fn2 (f) {
  setTimeout(() => {
    console.log('Function 2')
    f()
  }, 500)
}
 
fn2(fn3)
```


#### 事件发布/订阅

##### 

```js
class AsyncFunArr {
  constructor (...arr) {
    this.funcArr = [...arr]
  }
 
  next () {
    const fn = this.funcArr.shift()
    if (typeof fn === 'function') fn()
  }
 
  run () {
    this.next()
  }
}
 
const asyncFunArr = new AsyncFunArr(fn1, fn2, fn3);


//队列方法
function fn1 () {
  console.log('Function 1')
  asyncFunArr.next()
}
 
function fn2 () {
  setTimeout(() => {
    console.log('Function 2')
    asyncFunArr.next()
  }, 500)
}
 
function fn3 () {
  console.log('Function 3')
  asyncFunArr.next()
}
```

#### Promise

##### Promise概念：
Promise 对象是一个代理对象（代理一个值），被代理的值在Promise对象创建时可能是未知的。它允许你为异步操作的成功和失败分别绑定相应的处理方法（handlers）。 这让异步方法可以像同步方法那样返回值，但并不是立即返回最终执行结果，而是一个能代表未来出现的结果的promise对象。

一个 Promise有以下几种状态:
* pending: 初始状态，既不是成功，也不是失败状态。
* fulfilled: 意味着操作成功完成。
* rejected: 意味着操作失败。

##### Promise方法：
* Promise.all(iterable)==>这个方法返回一个新的promise对象，该promise对象在iterable参数对象里所有的promise对象都成功的时候才会触发成功，一旦有任何一个iterable里面的promise对象失败则立即触发该promise对象的失败
```js
var p1 = new Promise(function (resolve, reject) {
    setTimeout(resolve, 500, 'P1');
});
var p2 = new Promise(function (resolve, reject) {
    setTimeout(resolve, 600, 'P2');
});
// 同时执行p1和p2，并在它们都完成后执行then:
Promise.all([p1, p2]).then(function (results) {
    console.log(results); // 获得一个Array: ['P1', 'P2']
});
```
* Promise.race(iterable)==>当iterable参数里的任意一个子promise被成功或失败后，父promise马上也会用子promise的成功返回值或失败详情作为参数调用父promise绑定的相应句柄，并返回该promise对象
```js
var p1 = new Promise(function (resolve, reject) {
    setTimeout(resolve, 500, 'P1');
});
var p2 = new Promise(function (resolve, reject) {
    setTimeout(resolve, 600, 'P2');
});
Promise.race([p1, p2]).then(function (result) {
    console.log(result); // 'P1' P1比P2先返回，然后就丢弃p2
});
```
* Promise.reject(reason)==>返回一个状态为失败的Promise对象

* Promise.resolve(value)==>返回一个状态由给定value决定的Promise对象

##### Promise原型方法：
* Promise.prototype.catch(onRejected)==>添加一个拒绝(rejection) 回调到当前 promise, 返回一个新的promise

* Promise.prototype.then(onFulfilled, onRejected)==>添加解决(fulfillment)和拒绝(rejection)回调到当前 promise, 返回一个新的 promise, 将以回调的返回值来resolve.
```js
job1.then(job2).then(job3).catch(handleError); //job1、job2和job3都是Promise对象。
```

* Promise.prototype.finally(onFinally)==>添加一个事件处理回调于当前promise对象，并且在原promise对象解析完毕后，返回一个新的promise对象。回调会在当前promise运行完毕后被调用，无论当前promise的状态是完成(fulfilled)还是失败(rejected)




#### async/await

##### async 
* 它作为一个关键字放到函数前面，用于表示函数是一个异步函数，因为async就是异步的意思， 异步函数也就意味着该函数的执行不会阻塞后面代码的执行。
* async 函数返回的是一个promise 对象，如果要获取到promise 返回值，我们应该用then方法。
```js
async function timeout() {
    return 'hello world'
}
timeout().then(result => {
    console.log(result);
})
console.log('虽然在后面，但是我先执行');
```

##### await
* await是等待的意思，那么它等待什么呢，它后面跟着什么呢？其实它后面可以放任何表达式，不过我们更多的是放一个返回promise 对象的表达式。注意await 关键字只能放到async 函数里面。
```js
// 2s 之后返回双倍的值
function doubleAfter2seconds(num) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve(2 * num)
        }, 2000);
    } )
}

//2s秒 输出 2 * 30
async function testResult() {
    let result = await doubleAfter2seconds(30);
    console.log(result);
}

//6s 输出 2 * 110
async function testResult() {
    let first = await doubleAfter2seconds(30);
    let second = await doubleAfter2seconds(50);
    let third = await doubleAfter2seconds(30);
    console.log(first + second + third);
}
```
