---
title: egg日志
date: 2021-08-29 14:43:06
tags:
 - egg 
 - egg-logger
---


## 使用场景
 1. 问题排查
 2. 应用运行状态监控

## 日志特性
1. 日志等级
2. 统一错误日志
3. 启动日志和运行日志分离
4. 多进程日志
5. 日志自动切割
6. 日志高性能写入
7. 日志扩展、自定义

## 日志操作
### 打印日志
1. 应用日志： 和业务无关， 例如应用启动
   app.logger.info()

2. 请求日志： 记录请求相关日志， 日志会带上请求相关信息
   ctx.logger.info('ctx.logger') 
   //2019-02-03 11:18:56,157 INFO 46536 [-/127.0.0.1/-/5ms GET /api/user] ctx.logger
      
3. this.logger: 在controller、 service等实例中获取， 会带上日志文件路径，便于快速定位问题
   this.logger.info('this.logger');
   //2019-02-03 11:18:56,158 INFO 46536 [-/127.0.0.1/-/5ms GET /api/user] [controller.user] this.logger

 ### 日志等级
 1. NONE(默认不输出)、DEBUF --> logger.debug、INFO --> logger.info 、WARN ---> looger.warn、ERROR ---> looger.error  


### 错误日志
ctx.logger.error(new Error('error')); --- > 存放路径common-error.log



### 日志输出方式
 1. 终端和日志同时会输出
 2. local 和 unittest 环境下为 baseDir，即项目源码的根目录。
 3. prod 和其他运行环境，都为 HOME，即用户目录，如 /home/admin。
   
### 框架内置日志标识
1. ${appInfo.name}-web.log：应用输出的日志，通过上述的 ctx.logger 等打印。
2. egg-web.log： 用于框架内核、插件日志，通过 app.coreLogger 打印。
3. common-error.log：所有 Logger 的错误日志会统一汇集到该文件。   


## 日志切割
### 按天切割
1. 这是框架的默认日志切割方式，在每日 00:01 按照 .log.YYYY-MM-DD 文件名进行切割。
### 按文件大小切割
```
// config/config.default.js
const path = require('path');

module.exports = appInfo => {
  const config = {};

  config.logrotator = {
    filesRotateBySize: [
      'egg-web.log',
    ],
    maxFileSize: 2 * 1024 * 1024 * 1024,
  };

  return config;
};
```
         