---
title: egg洋葱模型
date: 2021-07-05 22:01:27
tags:
   - egg
   - 洋葱模型
   - 中间件
---

### 模型介绍
洋葱圈模型：所有的请求经过一个中间件的时候都会执行两次，对比Express形式的中间件，**洋葱圈的模型可以非常方便的实现后置处理逻辑**。 代表有egg、koa框架

<img src="/img/middle2.png" height = "auto" style="margin: 0 0 20px 0;" align=center />
<img src="/img/middle1.png" height = "auto" align=center />


### 代码执行顺序

```
const Koa = require('koa');

const app = new Koa();
const PORT = 3000;

// #1
app.use(async (ctx, next)=>{
    console.log(1)
    await next();
    console.log(1)
});
// #2
app.use(async (ctx, next) => {
    console.log(2)
    await next();
    console.log(2)
})

app.use(async (ctx, next) => {
    console.log(3)
})

app.listen(PORT);
console.log(`http://localhost:${PORT}`);

// 打印结果
// 1
// 2
// 3
// 2
// 1

```

### 使用案例


### 原理实现
1. 每个中间件都接收了一个next参数，在next函数运行之前的中间件代码会在一开始就执行，next函数之后的代码会在内部的中间件全部运行结束之后才执行。
2. 实现中间件的步骤 
   1. **首先我们要知道当前中间件的数组集合**
   2. **然后构建一个组合方法，对这些中间件按照洋葱的结构进行组合，并执行**


```
// middleware用来保存中间件
app.use = (fn) => {
    this.middleware.push(fn)
    return this
}

// compose组合函数来规定执行次序
function compose (middleware) {
  // context：上下文，next：传入的接下来要运行的函数
  return function (context, next) {
    function dispatch (i) {
      index = i
      // 中间件
      let fn = middleware[i]
      if (!fn) return Promise.resolve()
      try {
        // 我们这边假设和上文中的例子一样，有A、B、C三个中间件
        // 通过dispatch(0)发起了第一个中间件A的执行
        // A中间件执行之后，next作为dispatch(1)会被执行
        // 从而发起了下一个中间件B的执行，然后是中间件C被执行
        // 所有的中间件都执行了一遍后，执行Promise.resolve()
        // 最里面的中间件C的await next()运行结束，会继续执行console.log("C2")
        // 整个中间件C的运行结束又触发了Promise.resolve
        // 中间件B开始执行console.log("B2")
        // 同理，中间件A执行console.log("A2")
        return Promise.resolve(fn(context, () => {
          return dispatch(i + 1)
        }))
      } catch (err) {
        return Promise.reject(err)
      }
    }
    return dispatch(0)
  }
}
```

[文章来源参考](https://segmentfault.com/a/1190000013981513)