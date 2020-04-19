---
title: CommonJS和ES6 Module
date: 2020-04-16 10:09:23
tags:
    - commonjs
    - es6 module
---

### 运行原理 
#### CommonJS
##### 模块查找、加载、解析


##### 模块重复引用和循环引用
1. 重复引用问题。
   1. 模块加载是以绝对路径为key写入cache（require.cache）中，缓存可以解决重复查找和重复加载的问题。
2. 循环引用问题。
   1. 我们可以看到，在 CommonJS 规范中，当遇到 require() 语句时，会执行 require 模块中的代码，并缓存执行的结果，当下次再次加载时不会重复执行，而是直接取缓存的结果。正因为此，出现循环依赖时才不会出现无限循环调用的情况。虽然这种模块加载机制可以避免出现循环依赖时报错的情况，但稍不注意就很可能使得代码并不是像我们想象的那样去执行。

#### ES6 Module
##### 查找、加载、解析
##### 模块重复引用和循环引用
1. 重复引用问题。
   1. 模块加载是以绝对路径为key写入cache（require.cache）中，缓存可以解决重复查找和重复加载的问题。
1. 循环引用问题。
   1. 跟 CommonJS 模块一样，ES6 不会再去执行重复加载的模块，又由于 ES6 动态输出绑定的特性，能保证 ES6 在任何时候都能获取其它模块当前的最新值。

### 两者区别
#### 加载运行的时机
* Commonjs是运行时候加载。因为导出是个对象，只有在运行时候才能生成，所以使用时可以放在地方。对外是动态定义。

* ES6 Module是编译时候加载。编译时候就确定模块依赖关系以及输入、输出的变量。对外导出只是个静态定义。所以使用的时候import必须放在文件的最开始，且前面不允许有其他逻辑代码。


#### 模块导出的值不一样
* commonjs是值的拷贝， 注意浅拷贝基本数据类型和对象数据类型区别
```js
// dep.js
let a = 1
module.exports = {
    a: a
}
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