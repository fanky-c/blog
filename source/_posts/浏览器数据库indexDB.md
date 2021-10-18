---
title: 浏览器数据库IndexedDB
date: 2021-09-29 17:06:52
tags:
 - IndexedDB
---

### IndexedDB介绍
#### 概念
IndexedDB是浏览器提供的本地数据库， 它可以被网页脚本创建和操作

#### 特点
1. 键值对存储。 所有的数据都可以存储， 每一个数据都有主键， 主键是唯一的、不会重复
2. 异步。 IndexedDB操作不会锁死浏览器，用户依然可以进行其他操作； 和localstorage对比， 后者是同步的
3. 支持事务。 意味一系列操作之中， 只要有一步失败， 整个事务就会取消， 数据库回滚到之前的状态， 不存在只改写一部分数据
4. 同源策略。 受同源策略限制， 每一个数据库都有对应创建它的域名， 网页只能访问自身域名下数据库

#### IndexedDB基本概念
1. 数据库 -- IDBDatabase 对象
2. 对象仓库 -- IDBObjectStore 对象
3. 索引 -- IDBIndex 对象
4. 事务 -- IDBTransaction 对象
5. 操作请求 -- IDBRequest 对象
6. 指针 -- IDBCursor 对象
7. 主键集合 -- IDBKeyRange 对象

### 操作流程

<br>
[文章来源](https://www.ruanyifeng.com/blog/2018/07/indexeddb.html)