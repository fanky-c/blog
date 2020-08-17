---
title: nodejs相关
date: 2020-08-17 21:26:46
tags:
    - nodejs平滑重启、
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