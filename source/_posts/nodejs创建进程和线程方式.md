---
title: nodejs创建进程和线程方式
date: 2025-02-21 17:46:24
tags:
 - child_process
 - cluster
 - worker_threads
 - poolifier
---

## 1、创建进程
### 方式一 child_process

child_process 提供四种方法来创建子进程：
* spawn() --  适用于长时间运行的进程
* exec() -- 适用于短时间执行的 shell 命令
* execFile() --  直接运行可执行文件
* fork() --  专门用于创建 Node.js 子进程，并支持进程间通信（IPC）

使用 spawn() 创建子进程示例：

```js
const { spawn } = require('child_process');
const child = spawn('node', ['child.js']); // 运行 child.js 作为子进程
child.stdout.on('data', (data) => {
    console.log(`子进程输出: ${data}`);
});
child.on('close', (code) => {
    console.log(`子进程退出，退出码 ${code}`);
});
```

使用 fork() 创建子进程示例：
```js
// 主进程文件
const { fork } = require('child_process');
const child = fork('child.js');
child.on('message', (msg) => {
    console.log(`主进程收到消息: ${msg}`);
});
child.send('Hello from parent');

// 子进程child.js 文件
process.on('message', (msg) => {
    console.log(`子进程收到消息: ${msg}`);
    process.send('Hello from child');
});
```




### 方式二 cluster

cluster 模块用于创建多进程来共享服务器端口

示例：创建多个 Worker 进程：

```js
const cluster = require('cluster');
const http = require('http');
const os = require('os');

if (cluster.isMaster) {
    const numCPUs = os.cpus().length;
    console.log(`主进程 ${process.pid} 正在运行`);

    // 创建多个 Worker 进程
    for (let i = 0; i < numCPUs; i++) {
        cluster.fork();
    }

    cluster.on('exit', (worker, code, signal) => {
        console.log(`工作进程 ${worker.process.pid} 退出`);
    });
} else {
    // 工作进程创建 HTTP 服务器
    http.createServer((req, res) => {
        res.writeHead(200);
        res.end(`Hello from Worker ${process.pid}\n`);
    }).listen(8000);

    console.log(`工作进程 ${process.pid} 已启动`);
}
```

特点：
* cluster.isMaster 确保只有主进程负责管理 Worker。
* cluster.fork() 创建多个 Worker（默认和 CPU 核心数一致）。
* 各个 Worker 进程共享同一端口，提高性能。

### child_process 和 cluster 区别

<table border="1" cellspacing="0">
    <tr>
        <th>维度</th>
        <th>child_process</th>
        <th>cluster</th>
    </tr>
    <tr>
        <td><b>主要用途</b></td>
        <td>创建独立的子进程，适用于任务分工</td>
        <td>创建多个工作进程（Worker），用于负载均衡</td>
    </tr>
    <tr>
        <td><b>进程关系</b></td>
        <td>父进程可以创建多个子进程，但默认不共享端口</td>
        <td>Master 进程自动管理多个 Worker 进程，Worker 共享端口</td>
    </tr>
    <tr>
        <td><b>进程间通信 (IPC)</b></td>
        <td>需要手动使用 <code>process.send()</code> 进行数据传输</td>
        <td>内部自动处理进程间通信</td>
    </tr>
    <tr>
        <td><b>端口管理</b></td>
        <td>每个子进程通常绑定不同的端口</td>
        <td>所有 Worker 进程可以共享同一个端口</td>
    </tr>
    <tr>
        <td><b>应用场景</b></td>
        <td>任务处理（如爬虫、数据计算）</td>
        <td>多进程 Web 服务器，提高并发能力</td>
    </tr>
    <tr>
        <td><b>代码复杂度</b></td>
        <td>需要手动管理进程和通信</td>
        <td>更简洁，适用于创建多进程服务器</td>
    </tr>
</table>


## 2、创建线程

Node.js 的主线程是单线程的，但可以使用 Worker Threads 在后台创建多个线程执行计算密集型任务，而不阻塞主线程。

### 方式一 worker_threads

从 Node.js 10 开始，worker_threads 允许在 Node.js 中使用真正的多线程

```js
const { Worker, isMainThread, parentPort } = require('worker_threads');

if (isMainThread) {
    const worker = new Worker(__filename);
    worker.on('message', (msg) => {
        console.log(`收到子线程消息: ${msg}`);
    });

    worker.postMessage('Hello from Main Thread');
} else {
    parentPort.on('message', (msg) => {
        console.log(`收到主线程消息: ${msg}`);
        parentPort.postMessage('Hello from Worker Thread');
    });
}
```

### 方式二 poolifier 线程池

poolifier 是一个线程池管理库，可以优化 Worker 线程的管理。

```js
const { DynamicPool } = require('poolifier');

const pool = new DynamicPool(4, (msg) => {
    return `处理任务: ${msg}`;
});

async function run() {
    const result = await pool.execute('任务数据');
    console.log(result);
}

run();
```

优点：
* poolifier 提供线程池机制，避免频繁创建和销毁 Worker 线程，提高性能。

## 3、process 和 threads 区别

<table border="1" cellspacing="0">
  <thead>
    <tr>
      <th>方案</th>
      <th>模块</th>
      <th>适用场景</th>
      <th>优势</th>
      <th>劣势</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>多进程</td>
      <td>child_process / cluster</td>
      <td>处理多个请求，利用多核 CPU</td>
      <td>进程隔离，不影响主线程</td>
      <td>进程开销大，占用更多内存</td>
    </tr>
    <tr>
      <td>多线程</td>
      <td>worker_threads</td>
      <td>计算密集型任务，如加密、压缩</td>
      <td>共享内存，开销小</td>
      <td>线程同步需要管理</td>
    </tr>
  </tbody>
</table>

* 如果是 I/O 密集型任务（如 Web 服务器），建议使用 cluster 创建多个进程处理请求。
* 如果是 CPU 密集型任务（如数学计算），建议使用 worker_threads 进行多线程计算。

