---
title: Nodejs进程和线程相关
date: 2019-11-18 11:08:54
tags:
 - nodejs
 - nodejs进程、nodejs线程
---
### 进程
#### 介绍
进程是资源分配的最小单位。我们启动一个服务、运行一个实例，就是开一个服务进程，Node.js 里通过 node app.js 开启一个服务进程，多进程就是进程的复制（fork），fork 出来的每个进程都拥有自己的独立空间地址、数据栈，一个进程无法访问另外一个进程里定义的变量、数据结构，只有建立了 IPC 通信，进程之间才可数据共享。

#### Node.js默认主进程创建
1. node app.js
```js
const server = http.createServer();
server.listen(3000,()=>{
    process.title='测试进程';
    console.log('进程id',process.pid);
})
```
#### Node.js多进程创建
##### child_process模块
1，api使用
  1. child_process.spawn()：适用于返回大量数据，例如图像处理，二进制数据处理。
  2. child_process.exec()：适用于小量数据，maxBuffer 默认值为 200 * 1024 超出这个默认值将会导致程序崩溃，数据量过大可采用 spawn。
  3. child_process.execFile()：类似 child_process.exec()，区别是不能通过 shell 来执行，不支持像 I/O 重定向和文件查找这样的行为。
  4. child_process.fork()：衍生新的进程，进程之间是相互独立的，每个进程都有自己的 V8 实例、内存，系统资源是有限的，不建议衍生太多的子进程出来，通常根据系统 * CPU 核心数 * 设置（cpu核心数的获取方式 constcpus=require('os').cpus()）。

2，fork开启子进程代码Demo
```js
//app.js文件
const http = require('http');
const fork = require('child_process').fork;

const server = http.createServer((req, res) =>{
    if(req.url === '/test'){
        const fork = require('./fork.js');
        fork.send('开启一个新的子进程');

        // 当一个子进程使用 process.send() 发送消息时会触发 'message' 事件
        fork.on('message', (sum)=>{
            res.send(`Sum is ${sum}`);
            fork.kill();
        });

        // 子进程监听到一些错误消息退出
        fork.on('close', (code, signal)=>{
            console.log(`收到close事件，子进程收到信号 ${signal} 而终止，退出码 ${code}`);
            fork.kill();
        });

    }else{
        res.send(`ok`);
    }
})

server.listen(3001, ()=>{
    console.log(`server started at http://127.0.0.1:3001`)
});


//fork.js文件
const computation = () =>{
    let sum = 0;
    console.log(`计算开始`);
    console.time(`计算耗时开始`);
    for(let i = 0; i < 100000; i++){
        sum += 1;
    }
    console.log(`计算结束`);
    console.time(`计算耗时结束`);
    return sum;
}


process.on('message', msg => {
   console.log(`${msg}; process.pid:${process.pid}`);
   const sum = computation();
   process.send(sum);  //发送消息给父进程
});

```


##### cluster模块
1，cluster开启子进程代码Demo
```js
//app.js
const http = require('http');
const cpusNum = require('os').cpus().length;
const cluster = require('cluster');

if(cluster.isMaster){
   console.log(`Master process id is: ${process.pid}`);
   for(let i = 0; i < cpusNum; i++){
      cluster.fork();
   }

   cluster.on('exit', (work, code, signal)=>{
      console.log(`worker process died,id: ${worker.process.pid}`);
   })
}else{
   http.createServer((req, res)=>{
       res.writeHead(200);
       res.end(`hello world`);
   }).listen(3001);
}

```

> 在单核 CPU 系统之上我们采用 单进程 + 单线程 的模式来开发。在多核 CPU 系统之上，可以通过 child_process.fork 开启多个进程（Node.js 在 v0.8 版本之后新增了Cluster 来实现多进程架构） ，即 多进程 + 单线程 模式。注意：开启多进程不是为了解决高并发，主要是解决了单进程模式下 Node.js CPU 利用率不足的情况，充分利用多核 CPU 的性能。

#### Node.js进程通信原理
##### IPC(进程间通信)
1. ipc创建过程
  1. 主进程  ==> 生成工作进程
  2. 工作进程  ==> 连接IPC   
  3. 主进程  ==> 监听/接受IPC

#### process模块
##### 介绍
Node.js 中的进程 Process 是一个全局对象，无需 require 直接使用，给我们提供了当前进程中的相关信息：
1. process.env：环境变量，例如通过 process.env.NODE_ENV 获取不同环境项目配置信息
1. process.nextTick：这个在谈及 EventLoop 时经常为会提到
1. process.pid：获取当前进程id
1. process.ppid：当前进程对应的父进程
1. process.cwd()：获取当前进程工作目录，
1. process.platform：获取当前进程运行的操作系统平台
1. process.uptime()：当前进程已运行时间，例如：pm2 守护进程的 uptime 值
1. 进程事件： process.on(‘uncaughtException’,cb) 捕获异常信息、 process.on(‘exit’,cb）进程推出监听
1. 三个标准流： process.stdout 标准输出、 process.stdin 标准输入、 process.stderr 标准错误输出
1. process.title 指定进程名称，有的时候需要给进程指定一个名称


### 线程
#### 介绍
线程是操作系统能够进行运算调度的最小单位，首先我们要清楚线程是隶属于进程的，被包含于进程之中。一个线程只能隶属于一个进程，但是一个进程是可以拥有多个线程的。

#### nodejs线程
1. Node.js 虽然是单线程模型，但是其基于事件驱动、异步非阻塞模式，可以应用于高并发场景，避免了线程创建、线程之间上下文切换所产生的资源开销。
2. 当你的项目中需要有大量计算，CPU 耗时的操作时候，要注意考虑开启多进程来完成了。
3. Node.js 开发过程中，错误会引起整个应用退出，应用的健壮性值得考验，尤其是错误的异常抛出，以及进程守护（Pm2、Forever）是必须要做的。
4. 单线程无法利用多核CPU，但是后来Node.js 提供的API以及一些第三方工具相应都得到了解决。


[资料来源于1](https://mp.weixin.qq.com/s/VzXnnfn4gCBMd5wea3LRIg)
[资料来源于2](https://www.shengshunyan.xyz/2021/03/31/Node.js%E4%B8%AD%E7%9A%84%E5%A4%9A%E8%BF%9B%E7%A8%8B%E5%92%8C%E5%A4%9A%E7%BA%BF%E7%A8%8B/#cluster)