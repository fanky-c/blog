---
title: promise捕获异常
date: 2024-12-10 15:16:46
tags:
  - 捕获异常
  - 错误处理
  - promise
---

### 1、使用 .catch() 捕获异常

**特定**
1. 能捕获 Promise 执行中的任何错误，包括**显式调用reject()和代码抛出的异常**
2. 必须在 Promise 链的末尾调用 .catch() 以确保错误不会被忽略

```js
  new Promise((resolve, reject) => {
   reject(new Error('Something went wrong'));
  })
  .then((result) => {
    console.log('Success:', result);
  })
  .catch((error) => {
    console.error('Caught error:', error.message);
  });
```

### 2、在 .then() 的第二个参数中捕获错误

**特定**
1. 能捕获 Promise 的错误，但无法捕获链中后续的错误
2. 不如 .catch() 语义明确，推荐使用 .catch()

```js
  new Promise((resolve, reject) => {
    reject(new Error('Something went wrong'));
  })
  .then(
    (result) => {
      console.log('Success:', result);
    },
    (error) => {
      console.error('Caught error:', error.message);
    }
  );

```

### 3、使用 try...catch 包裹 async/await

**特定**
1. 适合处理复杂的异步操作
2. 必须确保所有 await 调用都在 try...catch 中

```js
async function fetchData() {
  try {
    const result = await new Promise((resolve, reject) => {
      reject(new Error('Error occurred in async/await'));
    });
    console.log(result);
  } catch (error) {
    console.error('Caught error:', error.message);
  }
}

fetchData();
```

### 4、封装统一的错误处理逻辑

**特定**
1. 避免大量重复的 try...catch
2. 返回统一的结果格式，便于处理

```js
function handlePromise(promise) {
  return promise
    .then((data) => [null, data])
    .catch((error) => [error, null]);
}

async function fetchData() {
  const [error, data] = await handlePromise(
    new Promise((resolve, reject) => {
      reject(new Error('Error in wrapped Promise'));
    })
  );
  if (error) {
    console.error('Caught error:', error.message);
    return;
  }
  console.log(data);
}

fetchData();
```

### 5、总结
1. 优先使用 .catch() 在 Promise 链末尾捕获错误
2. 对于复杂的异步操作，结合 async/await 和 try...catch 使用