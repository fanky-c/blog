---
title: nodejs相关
date: 2020-08-17 21:26:46
tags:
    - nodejs平滑重启
    - nodejs runTime架构
---

### nodejs平滑重启
1. pm2(restart, reload 以及 gracefulReload)
   1. restart: 直接关闭旧服务然后启动新服务，会造成已建立的连接失效
   2. reload: 平滑更新，先启动若干个新服务，同时停止旧服务接收请求。等待旧服务都停止服务后，关闭旧服务。和 cluster 的代码原理类似，有可能因为要等待连接关闭造成重启时间比较长。
   3. gracefulReload: 平滑更新，和 reload 的区别是 gracefulReload 会发送一个 shutdown 消息给旧服务，具体的停服逻辑可以由程序自己实现，比较灵活。

2. 多机器 + nginx负载均衡模块
   1. 更新服务时只需要逐台部署，保证同一时刻至少有一台机器在提供服务，nginx 就会将流量自动分配到正常服务的机器上

3. Node 本身的 cluster 模块
    1. 发一个重启信号给 Master，例如 kill -USR2 MASTER_PID
    2. Master 起 n 个新的服务，开始监听请求
    3. Master 停止原先旧服务的监听，并等待旧服务的所有连接结束
    4. 关闭旧服务

    ```js
        var cluster = require('cluster');
        var http = require('http');

        if (cluster.isMaster) {
            cluster.fork();

            cluster.on('exit', function(worker, code, signal) {
                console.log('worker ' + worker.process.pid + ' 退出');
            });

            cluster.on('listening', function(worker, code, signal) {
                console.log('worker ' + worker.process.pid + ' 开始服务');
            });

            cluster.on('disconnect', function(worker, code, signal) {
                console.log('worker ' + worker.process.pid + ' 停止服务');
            });

            process.on('SIGUSR2', function () {
                // 保存旧 worker 的列表，cluster.workers 是个 map
                var oldWorkers = Object.keys(cluster.workers).map(function (idx) {
                    return cluster.workers[idx];
                });

                // 起新服务
                cluster.fork();

                // 当新服务起起来之后，关闭所有的旧 worker
                cluster.once('listening', function (worker) {
                    oldWorkers.forEach(function (worker) {
                        // disconnect 会停止接收新请求，等待旧请求结束后再结束进程
                        worker.disconnect();
                    });
                });
            });
        } else {
            http.createServer(function(req, res) {
                // 模拟慢速请求
                setTimeout(function () {
                    res.writeHead(200);
                    res.end("hello world\n");
                }, 15000);
            }).listen(8000);
        }
    ```

### nodejs runtime架构

#### 什么是runtime
1. 程序分为几个状态，编辑时－>编译时->静态时->运行时
2. 比如有些错误在编译的时候是不会出现的，就是程序在语法上没有问题。但在运行时，因为缺少资源等因素可能出现运行时错误。叫做runtime error!
   
#### nodejs runtime架构
<img src="/img/noderuntime.jpeg"  alt="runtime" height="auto"/>

1. ***用户代码***：由程序员编写的 Javascript 应用程序代码。
2. ***Node.js API***： Node 提供的内置方法，可以在用户代码中使用（例如 用于使用 HTTP 方法的 HTTP  modules、crypto module、用于文件系统操作的 fs module、用于网络请求的 net 等……）。 有关 Node 提供的方法的完整列表，您可以在此处查看文档。 此外，您可以在此处找到源代码实现。 Node 的 API 是用 Javascript 编写的。
3. ***Bindings 和 C++扩展插件***： 在阅读 Node 时，您会看到 V8 是用 C++ 编写的，Libuv 是用 C 编写的，等等。 基本上，所有模块都是用 C 或 C++ 编写的，因为这些语言在处理底层任务和使用 OS API 方面非常出色和快速。 但是上层的Javascript代码怎么可能用其他语言去写代码呢？ 这就是 bindings 的作用。 它们充当两层之间的粘合剂，因此 Node 可以顺利使用 C 或 C++ 编写的低级代码。 那么，如果我们想自己添加一个C++模块应该怎么做呢？ 我们首先用 C++ 实现模块，然后为此编写 bindings代码。 我们编写的这段代码称为扩展插件。 更多信息可以在这里找到。
4. ***Node’s 依赖***: 这一层代表 Node 使用的底层库。 最大的依赖是谷歌的 V8 引擎和 Libuv。 其他库包括 OpenSSL（用于 SSL、TLS 和其他基本加密功能）、HTTP 解析器（用于解析 HTTP 请求和响应）、C-Ares（用于异步 DNS 请求）和 Zlib（用于快速压缩和解压缩）。
5. ***操作系统***: 这是表示上述库所使用的 OS API（系统调用）的最底层。 由于 OS-es 不同，这些库包括 Windows 和 Unix 变体的实现，这使得 Node 平台独立。



来源于：https://juejin.cn/post/6979790275879125006
