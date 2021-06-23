---
title: egg多进程模型和通讯
date: 2021-06-23 19:42:36
tags:
   - node多进程
   - cluster
   - egg进程通讯
   - egg多进程
---

### 名称解释
1. Cluster它可以通过一个父进程管理一坨子进程的方式来实现集群的功能。
  1. 在服务器上同时启动多个进程(一个master和多个worker; master不做具体的工作，只负责启动其他进程; worker它们接收请求，对外提供服务)
  2. 每个进程里都跑的是同一份源代码（好比把以前一个进程的工作分给多个进程去做）
  3. 这些进程可以同时监听一个端口

nodejs代码示例：
```js
const cluster = require('cluster');
const http = require('http');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  // Fork workers.
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }
  cluster.on('exit', function(worker, code, signal) {
    console.log('worker ' + worker.process.pid + ' died');
  });
} else {
  // Workers can share any TCP connection
  // In this case it is an HTTP server
  http.createServer(function(req, res) {
    res.writeHead(200);
    res.end("hello world\n");
  }).listen(8000);
}
```

### egg框架多进程模型
1. 框架进程分类
    a. master 俗称包工头: 不做业务逻辑，只做进程管理、任务分配
    b. agent  俗称秘书: 定时器、打印错误日志
    c. worker 俗称工人: 具体业务代码

2. node进程退出分类
    a. 未捕获异常
    b. OOM(内存溢出)、系统异常  


### egg框架进程间通讯（IPC）      




[文章来源参考](https://eggjs.org/zh-cn/core/cluster-and-ipc.html)
