---
title: js中分解长任务相关做法
date: 2025-04-01 20:49:49
tags:
  - 长任务
  - settimeout
  - requestAnimationFrame
  - requestIdleCallback
  - Web Workers
---

## 背景

如果让一个耗时且资源消耗大的任务占用主线程，很容易破坏网站的用户体验。无论应用程序变得多复杂，事件循环一次仍然只能处理一件事。如果你的代码占用了它，其他所有操作都将处于待机状态，通常用户很快就会察觉到。

**一个任务耗时过长，会导致页面卡顿、掉帧，甚至出现假死的现象！！！**

## 分解方案

### 1. setTimeout + 递归

#### a、逐步计算

```js
function sumNumbers(start, end, step, sum = 0) {
    if (start >= end) {
        console.log("Total Sum:", sum);
        return;
    }

    // 计算一部分
    let nextEnd = Math.min(start + step, end);
    for (let i = start; i < nextEnd; i++) {
        sum += i;
    }

    console.log(`Processed: ${start} to ${nextEnd}`);

    // 让出控制权，继续下一批计算
    setTimeout(() => sumNumbers(nextEnd, end, step, sum), 0);
}

sumNumbers(1, 1000000000, 1000000);
```

优点：

* 每次只计算一小部分，避免主线程长时间被占用。
* UI 仍然可以响应，用户不会感到页面卡顿。


#### b、分批处理数组

```js
function processArrayInChunks(arr, chunkSize, index = 0) {
    if (index >= arr.length) {
        console.log("Processing done!");
        return;
    }

    let end = Math.min(index + chunkSize, arr.length);
    for (let i = index; i < end; i++) {
        console.log("Processing:", arr[i]);
    }

    setTimeout(() => processArrayInChunks(arr, chunkSize, end), 0);
}

let bigArray = Array.from({ length: 10000 }, (_, i) => i);
processArrayInChunks(bigArray, 1000);
```

优点：

* 适用于超大数组，防止页面卡死
* 允许 UI 更新，用户可见进度

#### c、逐步渲染 DOM

```js
function renderListInChunks(container, data, chunkSize, index = 0) {
    if (index >= data.length) {
        console.log("Rendering complete!");
        return;
    }

    let fragment = document.createDocumentFragment();
    let end = Math.min(index + chunkSize, data.length);
    for (let i = index; i < end; i++) {
        let item = document.createElement("div");
        item.textContent = `Item ${data[i]}`;
        fragment.appendChild(item);
    }

    container.appendChild(fragment);

    setTimeout(() => renderListInChunks(container, data, chunkSize, end), 0);
}

let container = document.getElementById("list-container");
let data = Array.from({ length: 10000 }, (_, i) => i);
renderListInChunks(container, data, 100);
```

优点：

* 避免 UI 阻塞，每次渲染一部分，提高页面流畅度。
* 减少 DOM 操作成本，使用 documentFragment 批量插入。

### 2. Async/Await & Timeout

### 3. scheduler.postTask()

### 4. requestAnimationFrame

### 5. requestIdleCallback

### 6. MessageChannel

### 7. Web Workers



<br >
[文章来来源于](https://mp.weixin.qq.com/s/CYWm7y_qyanZb0c4s29sag)