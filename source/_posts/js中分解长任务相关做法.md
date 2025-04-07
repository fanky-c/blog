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

**setTimeout的 4ms 最小延迟。 也就是说，即使你设置了 0 毫秒，实际也要等待至少 4 毫秒之后才会被执行。 这是因为 JavaScript 引擎在处理事件循环时，会有一些额外的延迟。也是防止高频率的定时器拖垮页面性能。**

**虽然你写的是 setTimeout(fn, 0)，表示“立刻执行”，但实际上浏览器会有一个最小延迟阈值**

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

```js
async function sumNumbers(start, end, step, sum = 0) {
    for (let i = start; i < end; i += step) {
        for (let j = i; j < Math.min(i + step, end); j++) {
            sum += j;
        }

        console.log(`Processed: ${i} to ${Math.min(i + step, end)}`);

        await new Promise(resolve => setTimeout(resolve, 0)); // 让出主线程
    }

    console.log("Total Sum:", sum);
}

sumNumbers(1, 1000000000, 1000000);
```

原理：

* 每次累加 一部分数据，然后 await 一个 setTimeout(0) 让出主线程。
* 让浏览器有机会渲染 UI，防止页面冻结。

### 3. scheduler.postTask()

使用场景： 渐进式渲染（如大列表）、分批数据处理、避免页面卡顿、UI 优先、任务退让

不幸的是，整个调度器接口存在一个缺陷：目前它在所有浏览器中的支持情况并不理想。

```js
// 假设你要处理一个超大的数组，每次处理 1000 项，并用 scheduler.postTask() 在每一批之间让出主线程。
async function processInChunks(data, chunkSize) {
    let index = 0;
    while (index < data.length) {
        const chunk = data.slice(index, index + chunkSize);
        // 模拟处理任务（如计算、渲染）
        chunk.forEach(item => {
            console.log("Processing:", item);
        });
        index += chunkSize;
        // 让出主线程，等待浏览器调度下一次处理
        await scheduler.postTask(() => {}, { priority: 'user-visible' });
    }
    console.log("All chunks processed.");
}
const data = Array.from({ length: 10000 }, (_, i) => i);
processInChunks(data, 1000);
```

### 4. requestIdleCallback

requestIdleCallback() 是一个浏览器提供的 API，用来在浏览器空闲时执行代码。它的设计初衷是让我们可以安排一些**不重要但又需要执行的任务**（例如：打点、预处理数据、缓存更新等），而不会影响页面的流畅度。

```js
// 兼容
window.requestIdleCallback = window.requestIdleCallback || function (cb) {
  return setTimeout(() => {
    cb({
      timeRemaining: () => 0,
      didTimeout: true
    });
  }, 1);
};

window.cancelIdleCallback = window.cancelIdleCallback || function (id) {
  clearTimeout(id);
};

// usage
let bigArray = Array.from({ length: 10000 }, (_, i) => i);
function processChunk(deadline) {
  while (deadline.timeRemaining() > 0 && bigArray.length > 0) {
    let item = bigArray.shift();
    // 假设是计算、渲染等任务
    console.log("Processing:", item);
  }

  if (bigArray.length > 0) {
    requestIdleCallback(processChunk); // 继续处理下一批
  } else {
    console.log("All done!");
  }
}
requestIdleCallback(processChunk);
```

### 5. requestAnimationFrame

requestAnimationFrame() 是 JavaScript 中专为高性能动画渲染设计的 API。它会在浏览器下一次重绘前执行回调，确保动画平滑不卡顿，是做 动画、游戏循环、进度更新 的黄金工具。

注意事项

1. 不要滥用：不是所有长任务都适合 requestAnimationFrame，它适用于视觉相关的更新
2. 每一帧必须快速返回，不超过 16ms，否则掉帧
3. 可以与 cancelAnimationFrame 配对使用来停止动画

### 6. MessageChannel

我们可以用它像微任务一样分片处理大任务，它比 setTimeout 更快响应、不卡 UI。

```js
// 处理大数组，避免卡顿

const channel = new MessageChannel();
const taskQueue = [];

channel.port1.onmessage = () => {
  const task = taskQueue.shift();
  if (task) task();
};

function schedule(task) {
  taskQueue.push(task);
  channel.port2.postMessage(null); // 触发异步执行
}

// 用于处理一大批数据
let arr = Array.from({ length: 10000 }, (_, i) => i);

function processChunk() {
  const chunk = arr.splice(0, 500);
  chunk.forEach(i => {
    // 模拟工作
    console.log("Processing:", i);
  });

  if (arr.length > 0) {
    schedule(processChunk);
  } else {
    console.log("All done!");
  }
}

schedule(processChunk);
```

### 7. Web Workers

```js
// 主线程代码（main.js）
const worker = new Worker('worker.js');

const bigArray = Array.from({ length: 10000 }, (_, i) => i);

worker.postMessage({ type: 'start', data: bigArray });

worker.onmessage = (e) => {
  if (e.data.done) {
    console.log("处理完成");
  } else {
    console.log("处理中:", e.data.chunkResult);
  }
};

//Worker 脚本（worker.js）
self.onmessage = function (e) {
  if (e.data.type === 'start') {
    const data = e.data.data;
    const CHUNK_SIZE = 1000;
    let index = 0;

    function processNextChunk() {
      const chunk = data.slice(index, index + CHUNK_SIZE);
      const chunkResult = chunk.map(i => i * 2); // 模拟计算
      postMessage({ chunkResult });

      index += CHUNK_SIZE;
      if (index < data.length) {
        setTimeout(processNextChunk, 0); // 分片执行
      } else {
        postMessage({ done: true });
      }
    }
    processNextChunk();
  }
};
```


<br >
[文章来来源于](https://mp.weixin.qq.com/s/CYWm7y_qyanZb0c4s29sag)