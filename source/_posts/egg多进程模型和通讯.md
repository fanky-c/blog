---
title: egg多进程模型和通讯
date: 2021-06-23 19:42:36
tags:
   - node多进程
   - cluster
   - egg进程通讯
   - egg多进程
---

### **名称解释**

#### **cluster是啥**

cluster可以通过一个父进程管理一坨子进程的方式来实现集群的功能。

#### **cluster具体内容**

1. 在服务器上同时启动多个进程(**一个master和多个worker; master不做具体的工作，只负责启动其他进程; worker它们接收请求，对外提供服务**)

2. 每个进程里都跑的是同一份源代码（好比把以前一个进程的工作分给多个进程去做）

3. 这些进程可以同时监听一个端口

   

nodejs实现多进程代码示例：

```js

const cluster = require('cluster');

const http = require('http');

const numCPUs = require('os').cpus().length;



if (cluster.isMaster) {

  // Fork workers.

  for (let i = 0; i < numCPUs; i++) {

​    cluster.fork();

  }

  cluster.on('exit', function(worker, code, signal) {

​    console.log('worker ' + worker.process.pid + ' died');

  });

} else {

  // Workers can share any TCP connection

  // In this case it is an HTTP server

  http.createServer(function(req, res) {

​    res.writeHead(200);

​    res.end("hello world\n");

  }).listen(8000);

}

```



### **egg框架多进程模型**

#### **进程守护**

#####  node进程退出分类和解决方案

- 未捕获异常

  当代码抛出了异常没有被捕获到时，进程将会退出，此时 Node.js 提供了 `process.on('uncaughtException', handler)` 接口来捕获它，但是当一个 Worker 进程遇到 [未捕获的异常](https://nodejs.org/dist/latest-v6.x/docs/api/process.html#process_event_uncaughtexception) 时，它已经处于一个不确定状态，此时我们应该让这个进程优雅退出:

   流程图： **1.关闭异常 Worker 进程所有的 TCP Server  --> 2. Master 立刻 fork 一个新的 Worker 进程，保证在线的『工人』总数不变  --> 3. 异常 Worker 等待一段时间，处理完已经接受的请求后退出**

  ```
  +---------+                 +---------+
  |  Worker |                 |  Master |
  +---------+                 +----+----+
       | uncaughtException         |
       +------------+              |
       |            |              |                   +---------+
       | <----------+              |                   |  Worker |
       |                           |                   +----+----+
       |        disconnect         |   fork a new worker    |
       +-------------------------> + ---------------------> |
       |         wait...           |                        |
       |          exit             |                        |
       +-------------------------> |                        |
       |                           |                        |
      die                          |                        |
                                   |                        |
                                   |                        |
  ```

- OOM(内存溢出)、系统异常

  当一个进程出现异常导致 crash 或者 OOM 被系统杀死时，不像未捕获异常发生时我们还有机会让进程继续执行，只能够让当前进程直接退出，Master 立刻 fork 一个新的 Worker。  





#### **agent机制**

##### 背景

1. 有些工作其实不需要每个 Worker 都去做，如果都做，一来是浪费资源，更重要的是可能会导致多进程间资源访问冲突。举个例子：生产环境的日志文件我们一般会按照日期进行归档。

2. 对于这一类后台运行的逻辑，我们希望将它们放到一个单独的进程上去执行，这个进程就叫 Agent Worker，简称 Agent。Agent 好比是 Master 给其他 Worker 请的一个『秘书』，它不对外提供服务，只给 App Worker 打工，专门处理一些公共事务。

   ```
                   +--------+          +-------+
                   | Master |<-------->| Agent |
                   +--------+          +-------+
                   ^   ^    ^
                  /    |     \
                /      |       \
              /        |         \
            v          v          v
   +----------+   +----------+   +----------+
   | Worker 1 |   | Worker 2 |   | Worker 3 |
   +----------+   +----------+   +----------+
   ```



##### egg启动时序图

1. Master 启动后先 fork Agent 进程
2. Agent 初始化成功后，通过 IPC 通道通知 Master
3. Master 再 fork 多个 App Worker
4. App Worker 初始化成功，通知 Maste
5. 所有的进程初始化成功后，Master 通知 Agent 和 Worker 应用启动成功

```
+---------+           +---------+          +---------+
|  Master |           |  Agent  |          |  Worker |
+---------+           +----+----+          +----+----+
     |      fork agent     |                    |
     +-------------------->|                    |
     |      agent ready    |                    |
     |<--------------------+                    |
     |                     |     fork worker    |
     +----------------------------------------->|
     |     worker ready    |                    |
     |<-----------------------------------------+
     |      Egg ready      |                    |
     +-------------------->|                    |
     |      Egg ready      |                    |
     +----------------------------------------->|
```





#### **agent用法**

你可以在应用或插件根目录下的 `agent.js` 中实现你自己的逻辑（和[启动自定义](https://eggjs.org/zh-cn/basics/app-start.html) 用法类似，只是入口参数是 agent 对象）

```js
// agent.js
module.exports = agent => {
  // 在这里写你的初始化逻辑

  // 也可以通过 messenger 对象发送消息给 App Worker
  // 但需要等待 App Worker 启动成功后才能发送，不然很可能丢失
  agent.messenger.on('egg-ready', () => {
    const data = { ... };
    agent.messenger.sendToApp('xxx_action', data);
  });
};

// app.js
module.exports = app => {
  app.messenger.on('xxx_action', data => {
    // ...
  });
};
```



#### **master和agent和worker**

1. master 俗称包工头: 不做业务逻辑，只做进程管理、任务分配

2. agent  俗称秘书: 定时器、打印错误日志

3. worker 俗称工人: 具体业务代码

| 类型   | 进程数量            | 作用                         | 稳定性 | 是否运行业务代码 |
| ------ | ------------------- | ---------------------------- | ------ | ---------------- |
| Master | 1                   | 进程管理，进程间消息转发     | 非常高 | 否               |
| Agent  | 1                   | 后台运行工作（长连接客户端） | 高     | 少量             |
| Worker | 一般设置为 CPU 核数 | 执行业务代码                 | 一般   | 是               |

##### Master

​    Master 进程的稳定性是极高的，线上运行时我们只需要通过 [egg-scripts](https://github.com/eggjs/egg-scripts) 后台运行通过 `egg.startCluster` 启动的 Master 进程就可以了，不再需要使用 [pm2](https://github.com/Unitech/pm2) 等进程守护模块。

##### Agent

在大部分情况下，我们在写业务代码的时候完全不用考虑 Agent 进程的存在，但是当我们遇到一些场景，只想让代码运行在一个进程上的时候，Agent 进程就到了发挥作用的时候了。

由于 Agent 只有一个，而且会负责许多维持连接的脏活累活，因此它不能轻易挂掉和重启，所以 Agent 进程在监听到未捕获异常时不会退出，但是会打印出错误日志，**我们需要对日志中的未捕获异常提高警惕**。



##### Worker

Worker 进程负责处理真正的用户请求和[定时任务](https://eggjs.org/zh-cn/basics/schedule.html)的处理。而 Egg 的定时任务也提供了只让一个 Worker 进程运行的能力，**所以能够通过定时任务解决的问题就不要放到 Agent 上执行**。







### **egg框架进程间通讯（IPC）**

#### node进程通讯      

```js
'use strict';
const cluster = require('cluster');

if (cluster.isMaster) {
  const worker = cluster.fork();
  worker.send('hi there');
  worker.on('message', msg => {
    console.log(`msg: ${msg} from worker#${worker.id}`);
  });
} else if (cluster.isWorker) {
  process.on('message', (msg) => {
    process.send(msg);
  });
}
```

细心的你可能已经发现 cluster 的 IPC 通道只存在于 Master 和 Worker/Agent 之间，Worker 与 Agent 进程互相间是没有的。那么 Worker 之间想通讯该怎么办呢？是的，通过 Master 来转发

```
广播消息： agent => all workers
                  +--------+          +-------+
                  | Master |<---------| Agent |
                  +--------+          +-------+
                 /    |     \
                /     |      \
               /      |       \
              /       |        \
             v        v         v
  +----------+   +----------+   +----------+
  | Worker 1 |   | Worker 2 |   | Worker 3 |
  +----------+   +----------+   +----------+

指定接收方： one worker => another worker
                  +--------+          +-------+
                  | Master |----------| Agent |
                  +--------+          +-------+
                 ^    |
     send to    /     |
    worker 2   /      |
              /       |
             /        v
  +----------+   +----------+   +----------+
  | Worker 1 |   | Worker 2 |   | Worker 3 |
  +----------+   +----------+   +----------+
```

#### 发送

- `app.messenger.broadcast(action, data)`：发送给所有的 agent / app 进程（包括自己）

- ```
  app.messenger.sendToApp(action, data)
  ```

  发送给所有的 app 进程

  - 在 app 上调用该方法会发送给自己和其他的 app 进程
  - 在 agent 上调用该方法会发送给所有的 app 进程

- ```
  app.messenger.sendToAgent(action, data)
  ```

  发送给 agent 进程

  - 在 app 上调用该方法会发送给 agent 进程
  - 在 agent 上调用该方法会发送给 agent 自己

- ```
  agent.messenger.sendRandom(action, data)
  ```

  - app 上没有该方法（现在 Egg 的实现是等同于 sentToAgent）
  - agent 会随机发送消息给一个 app 进程（由 master 来控制发送给谁）

- `app.messenger.sendTo(pid, action, data)`: 发送给指定进程

  ```js
  // app.js
  module.exports = app => {
    // 注意，只有在 egg-ready 事件拿到之后才能发送消息
    app.messenger.once('egg-ready', () => {
      app.messenger.sendToAgent('agent-event', { foo: 'bar' });
      app.messenger.sendToApp('app-event', { foo: 'bar' });
    });
  }
  ```

  

#### 接收

在 messenger 上监听对应的 action 事件，就可以收到其他进程发送来的信息了。

```js
app.messenger.on(action, data => {
  // process data
});
app.messenger.once(action, data => {
  // process data
});
```



[文章来源参考](https://eggjs.org/zh-cn/core/cluster-and-ipc.html)


