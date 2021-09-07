---
title: node中使用redis做请求限流
date: 2021-08-14 16:04:04
tags:
   - node限流
   - redis
---

## 限流介绍
请求限流用于控制网络请求和传输量的技术， 用于健全node服务应用， **通常会开启接口速率限制，来控制一定周期内最大的请求量，用于保护服务应用免遭恶意请求和流量攻击**

## 限流常用算法 
### 1. 固定窗口算法
#### a. 介绍
 固定窗口算法： 在固定的时间范围内，窗口大小指允许通过最大请数。 也就是一段时间内最大允许请求量; 也就是在一个周期开始的时候，计数器清零，每个请求都将被计数，若计数达到上限，则不会再响应多余的请求，直到进入下一个周期

#### b. 代码实现
```js
// index.js
import { fixedWindow } from "./middleware/fixedWindow.js";
app.use(fixedWindow);

// middleware/ratelimit
import Redis from 'ioredis'

const FIX_WINDOW_SIZE = 60; // second
const FIX_WINDOW_MAX_REQUEST = 100;

const redisClient = new Redis(6379);

export const fixedWindow = async (ctx, next) => {
  const redisKey = `${ctx.ip}:ratelimit`;
  const curCount = await redisClient.get(redisKey);
  if (!curCount) {
    await redisClient.setex(redisKey, FIX_WINDOW_SIZE, 1);
    next();
    return;
  }
  if (Number(curCount) < FIX_WINDOW_MAX_REQUEST) {
    await redisClient.incr(redisKey);
    next();
  } else {
    ctx.status = 429;
    ctx.body = "you have too many requests";
  }
};
```
#### c. 缺点
<img src="/img/redis1.png" height = "auto" align=center />
**不平滑， 在2个窗口之间容易被攻击，图中9：03 - 9：04实际请求数已经超过了设置的最大上限**


### 2. 滑动窗口算法
#### a. 介绍
<img src="/img/redis2.jpeg" height = "auto" align=center />
滑动窗口算法：这种算法是对固定窗口算法一种优化（这种算法一定范围内只有一个窗口）， 在一个周期范围内有多个窗口进行计数， 窗口越多， 算法就越平滑。

#### b. 代码实现
```js
// 1. 在每次更新键值的时候，通过 EXPIRE 去更新 Redis Key 的过期时间
// 2. 在每次更新键值的时候，通过 HDEL 删除在滑动窗口之前的 Hash Key
// 3. 在每次更新键值的时候，我们需要通过 HGETALL 获取到所有 Key，然后进行进行判断：
//      3.1 存在 Key 在最小拆分窗口的周期时间内，HINCRBY 在原有 Key 的基础上去增加 1
//      3.2 不存在 Key 在最小拆分窗口的周期时间时，将当前时间的时间戳作为 HINCRBY 的 Key

```


### 3. 漏桶算法
#### a. 介绍
<img src="/img/redis4.png" height = "auto" align=center />
漏桶算法：api请求过程类比漏桶加水，漏桶流入速度不做限制， 而是限制漏桶流出速度； 当流入速度大于流出速度， 随着桶中水平面不断上涨， 漏桶水流就会溢出，


#### b. 代码实现


### 4. 令牌桶算法
#### a. 介绍
<img src="/img/redis3.png" height = "auto" align=center />
令牌桶算法：定义了一个集合（也就是桶）， 集合中容纳一定数量的令牌， api被请求一次消耗一个令牌； 如集合中没有令牌则不允许请求通过， 集合通过一定的速率去生成新的令牌， 以此达到限流的作用。


#### b. 代码实现
```js
//1. 当请求进来时，初始化一个令牌桶与过期时间，其中每次进行更新操作，都需要设置令牌桶的过期时间为需要补充的令牌数
// 2. 计算当前最新的令牌数为多少，判断是否请求能拿到令牌：
//   2.1 若请求能拿到令牌，则更新最新的令牌数与更新时间
//   2.2 若不能拿到令牌，将该请求抛弃


```
## redis-cell模块

## nginx 限制IP的连接和并发达到限流 


[参考文章1](https://vv13.cn/blog/post/Algorithm/20200902_%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E5%9C%A8Node.js%E4%B8%AD%E4%BD%BF%E7%94%A8Redis%E5%81%9A%E8%AF%B7%E6%B1%82%E9%99%90%E6%B5%81/)

[参考文章2](http://www.html.cn/qa/node-js/10803.html)