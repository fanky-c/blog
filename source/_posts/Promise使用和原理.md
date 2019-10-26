---
title: Promise使用和原理
date: 2019-10-26 16:57:23
tags:
  - Promise原理
  - Promise使用
---

### Promise是什么？
Promise是JS异步编程中的重要概念，异步抽象处理对象，是目前比较流行Javascript异步编程解决方案之一。这句话说的很明白了，Promise是一种用于解决异步问题的思路、方案或者对象方式。


### Promise用法
1. 首先，Promise是一个对象，因此，我们使用new的方式新建一个。
2. 然后给它传一个函数作为参数，这个函数呢也有两个参数，一个叫resolve(决定)，一个叫reject(拒绝)，这两个参数也是函数。
3. 紧接着，我们使用then来调用这个Promise。

参考代码
```js
new Promise(function (resolve, reject) {
  setTimeout(()=>{
    let num = Math.ceil(Math.random() * 10)
    if (num > 2) {
      resolve(num)
    } else {
      reject(num)
    }
  },2000);
}).then((res)=>{
  console.log(res); //reslove()
},(err)=>{
  console.log(err); //reject()
})
```

### Promise原理

在Promise的内部，有一个状态管理器的存在，有三种状态：pending、fulfilled、rejected。
  1. promise 对象初始化状态为 pending
  2. 当调用resolve(成功)，会由pending => fulfilled
  3. 当调用reject(失败)，会由pending => rejected

promsie状态 只能由 pending => fulfilled/rejected, 一旦修改就不能再变（记住，一定要记住)；  当状态为fulfilled（rejected反之）时，then的成功回调函数会被调用，并接受上面传来的num，进而进行操作；promise.then方法每次调用，都返回一个新的promise对象 所以可以链式写法（无论resolve还是reject都是这样）。


### Promise方法
#### then
then方法用于注册当状态变为fulfilled或者reject时的回调函数：

#### catch
1. catch能捕获reject(err)中返回的错误。
2. catch还能捕获链式调用then(()=>{}, ()=>{})中里面代码异常。
```js
const fn = new Promise(function (resolve, reject) {
  let num = Math.ceil(Math.random() * 10)
  if (num > 10) {
    resolve(num)
  } else {
    reject(num)
  }
})
fn.then((res)=>{
  console.log(res1)
}).catch((err)=>{
  console.log(`err==>${err}`)  //会提示res1未定义
})
```

#### resolve、reject
Promise.resolve 返回一个fulfilled状态的promise对象，Promise.reject 返回一个rejected状态的promise对象
```
Promise.resolve('hello').then(function(value){
    console.log(value);
});

Promise.resolve('hello');
// 相当于
const promise = new Promise(resolve => {
   resolve('hello');
});
```
#### all
1. 大概就是作为参数的几个promise对象一旦有一个的状态为rejected，则all的返回值就是rejected。
```
var   p1 = Promise.resolve(1),
      p2 = Promise.reject(2),
      p3 = Promise.resolve(3);
Promise.all([p1, p2, p3]).then((res)=>{
    //then方法不会被执行
    console.log(results);
}).catch((err)=>{
    //catch方法将会被执行，输出结果为：2
    console.log(err);
});
```
2. 当这几个作为参数的函数的返回状态为fulfilled时，至于输出的时间就要看谁跑的慢了：


#### race
##### 介绍：
promise.race()方法也可以处理一个promise实例数组但它和promise.all()不同，从字面意思上理解就是竞速，那么理解起来上就简单多了，也就是说在数组中的元素实例那个率先改变状态，就向下传递谁的状态和异步结果。但是，其余的还是会继续进行的。
```
let p1 = new Promise((resolve)=>{
  setTimeout(()=>{
    console.log('1s') //1s后输出
    resolve(1)
  },1000)
})
let p10 = new Promise((resolve)=>{
  setTimeout(()=>{
    console.log('10s') //10s后输出
    resolve(10) //不传递
  },10000)
})
let p5 = new Promise((resolve)=>{
  setTimeout(()=>{
    console.log('5s') //5s后输出
    resolve(5) //不传递
  },5000)
})
Promise.race([p1, p10, p5]).then((res)=>{
    console.log(res); // 最后输出
})

//输入结果
1s
1
5s
10s
```

##### 使用场景： 超时处理
图片超时
```
//请求某个图片资源
let requestImg = new Promise(function(resolve, reject){
        var img = new Image();
        img.onload = function(){
            resolve(img);
        }
    });
//延时函数，用于给请求计时
let timeOut = new Promise(function(resolve, reject){
        setTimeout(function(){
            reject('图片请求超时');
        }, 5000);
    });

Promise.race([requestImg, timeout]).then((res)=>{
    console.log(res);
}).catch((err)=>{
    console.log(err);
});
```
### Promise面试题
1. 1，Promise状态一旦改变，无法在发生变更
```
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('success')
    reject('error')
  }, 1000)
})
promise.then((res)=>{
  console.log(res)
},(err)=>{
  console.log(err)
})
//输出结果：success
```

2. 2，Promise的then方法的参数期望是函数，传入非函数则会发生值穿透。
```
Promise.resolve(1)
  .then(2)
  .then(Promise.resolve(3))
  .then(console.log)
//输出结果：1
```


3. 3，实现一个简单的Promise
```
function Promise(fn){
  var status = 'pending'
  function successNotify(){
      status = 'fulfilled'//状态变为fulfilled
      toDoThen.apply(undefined, arguments)//执行回调
  }
  function failNotify(){
      status = 'rejected'//状态变为rejected
      toDoThen.apply(undefined, arguments)//执行回调
  }
  function toDoThen(){
      setTimeout(()=>{ // 保证回调是异步执行的
          if(status === 'fulfilled'){
              for(let i =0; i< successArray.length;i ++)    {
                  successArray[i].apply(undefined, arguments)//执行then里面的回掉函数
              }
          }else if(status === 'rejected'){
              for(let i =0; i< failArray.length;i ++)    {
                  failArray[i].apply(undefined, arguments)//执行then里面的回掉函数
              }
          }
      })
  }
  var successArray = []
  var failArray = []
  fn.call(undefined, successNotify, failNotify)
  return {
      then: function(successFn, failFn){
          successArray.push(successFn)
          failArray.push(failFn)
          return undefined // 此处应该返回一个Promise
      }
  }
}
```
解题思路：Promise中的resolve和reject用于改变Promise的状态和传参，then中的参数必须是作为回调执行的函数。因此，当Promise改变状态之后会调用回调函数，根据状态的不同选择需要执行的回调函数。

[文章来源于掘金](https://juejin.im/post/5ad3fa47518825619d4d3a11#heading-0)