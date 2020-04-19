---
title: CommonJS和ES6 Module
date: 2020-04-16 10:09:23
tags:
    - commonjs
    - es6 module
---

### 运行原理 
#### CommonJS
##### 模块查找
1. 内置模块


##### 模块加载
1. 缓存。 模块加载是以绝对路径为key写入cache（require.cache）中,可以解决重复查找和重复加载的问题
2. 循环引用问题。

#### ES6 Module
##### 查找、加载、解析

### 两者区别
#### 加载运行的时机
* commonjs是运行时候加载。因为导出是个对象，只有在运行时候才能生成，所以使用时可以放在地方。对外是动态定义。

* ES6 Module是编译时候加载。编译时候就确定模块依赖关系以及输入、输出的变量。对外导出只是个静态定义。所以使用的时候import必须放在文件的最开始，且前面不允许有其他逻辑代码。


#### 模块导出的值不一样
* commonjs是值的拷贝
```js
// dep.js
export let a = 1
setTimeout(() => a += 1, 500)

// app.js
import { a } from 'dep'
setTimeout(function () {
  console.log(a) // 输出：1
}, 1000)
```

* es6是值的引用
```js
// dep.js
export let a = 1
setTimeout(() => a += 1, 500)

// app.js
import { a } from 'dep'
setTimeout(function () {
  console.log(a)   //输出：2
}, 1000)
```