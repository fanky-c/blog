---
title: 脚本async和defer的区别
date: 2021-05-13 17:48:00
tags:
 - async
 - defer
---

### 概念

 async和defer都是脚本异步加载的方式



### 原理

1.  async: 
<img src="/img/async.png" width = "700" height = "auto" alt="async" align=center />

1.  defer: 
<img src="/img/defer.png" width = "700" height = "auto" alt="defer" align=center />



### 共同点和区别

1. async: 异步加载， 加载完毕立即执行js代码， 有可能阻塞页面

2. defer：异步加载， 等dom树构建完成（DOMContentLoaded）才执行代码
