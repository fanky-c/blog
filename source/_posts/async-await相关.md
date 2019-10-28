---
title: async/await相关
date: 2019-10-27 17:08:30
tags:
   - async
   - await
---

### async函数

#### async介绍
1. 返回值是Promise， 可以直接使用 then() 方法进行调用。
2. 更好的语义。async 和 await 相较于 * 和 yield 更加语义化
3. 让异步代码看起来更像同步代码

#### async使用


### await函数
#### await使用
1. await这个关键字只能在使用async定义的函数里面使用。
2. 

#### await错误处理
1. 推荐await命令放在try/catch
2. Promise的catch()捕获错误


### async缺点以及优化方法

#### 缺点
大量使用await函数，每个await将等待前一个Promise完成，这样会导致代码变慢（而实际上你想要的是promises同时开始处理，只对结果做同步处理）。


#### 解决方案
方案： 在这里，我们将三个Promise对象存储在变量中，这样可以同时启动它们关联的进程。
```
function timeoutPromise(interval) {
  return new Promise((resolve, reject) => {
    setTimeout(function(){
      resolve("done");
    }, interval);
  });
};

//优化前
async function timeTest() {
  await timeoutPromise(3000);
  await timeoutPromise(3000);
  await timeoutPromise(3000);
}


//优化后
async function timeTest() {
  const timeoutPromise1 = timeoutPromise(3000);
  const timeoutPromise2 = timeoutPromise(3000);
  const timeoutPromise3 = timeoutPromise(3000);

  await timeoutPromise1;
  await timeoutPromise2;
  await timeoutPromise3;
}


//测试
let startTime = Date.now();
timeTest().then(() => {
  let finishTime = Date.now();
  let timeTaken = finishTime - startTime;
  alert("Time taken in milliseconds: " + timeTaken);
})

```