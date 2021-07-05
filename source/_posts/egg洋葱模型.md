---
title: egg洋葱模型
date: 2021-07-05 22:01:27
tags:
   - egg
   - 洋葱模型
   - 中间件
---

### 模型介绍
洋葱圈模型：所有的请求经过一个中间件的时候都会执行两次，对比Express形式的中间件，洋葱圈的模型可以非常方便的实现后置处理逻辑。 代表有egg、koa框架

<img src="/img/middle2.png" height = "auto" style="margin: 0 0 20px 0;" align=center />
<img src="/img/middle1.png" height = "auto" align=center />


### 代码执行顺

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