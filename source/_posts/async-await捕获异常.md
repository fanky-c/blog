---
title: async/await捕获异常
date: 2024-12-10 15:13:42
tags:
   - 异常捕获
   - 错误处理
   - async/await
---

### 1、try...catch捕获错误

**特点**
1. 适合局部错误处理
2. 明确处理每个函数可能的错误

```js
async function fetchData() {
  try {
    const response = await fetch('https://api.example.com/data');
    if (!response.ok) {
      throw new Error('Network response was not ok');
    }
    const data = await response.json();
    console.log(data);
  } catch (error) {
    console.error('Error occurred:', error.message);
  }
}

fetchData();
```

### 2、使用.catch方法处理错误

**概念**
可以将错误处理链式附加到 async 函数返回的 Promise 上

**特点**
1. 更简洁，但可能会遗漏内部的部分逻辑错误
2. 适合全局或较少特定需求的错误处理

```js
async function fetchData() {
  const response = await fetch('https://api.example.com/data');
  const data = await response.json();
  return data;
}

fetchData()
  .then((data) => console.log(data))
  .catch((error) => console.error('Error occurred:', error.message));
```

### 3、自定义错误封装

```js
function awaitWrap<T, U = any>(promise: Promise<T>): Promise<[U | null, T | null]> {
    return promise
        .then<[null, T]>((data: T) => [null, data])
        .catch<[U, null]>(err => [err, null])
}

(async()=>{
   const fetchData = () => {
      return new Promise((resolve, reject) => {
         setTimeout(() => {
            resolve('fetch data is me')
         }, 1000)
      })
   }
   const [err, data] = await awaitWrap(fetchData())
   console.log('err', err)  // err null
   console.log('data', data) // data fetch data is me
})()
```


### 4、最佳实践建议

1. 对单个异步任务，优先使用 try...catch 处理特定逻辑的错误。
2. 对于链式调用，使用 .catch 捕获错误。
3. 为未处理的 Promise 提供全局监控 （unhandledRejection），避免错误遗漏。
4. 封装错误处理逻辑，简化复杂场景的代码。
5. 针对框架（如 Express、Egg），可用中间件统一捕获异步错误。