---
title: CommonJS和ES6 Module
date: 2020-04-16 10:09:23
tags:
    - commonjs
    - es6 module
---
### 使用介绍
#### CommonJS
##### 使用
1. 使用场景：**nodejs服务器端居多**
2. 导出
```js
// 导出一个对象
module.exports = {
    name: "蛙人",
    age: 24,
    sex: "male"
}

// 导出任意值
module.exports.name = "蛙人"
module.exports.sex = null
module.exports.age = undefined

// 混合导出
exports.name = "蛙人"
module.exports.age = 24
```
3. 导入
```js
// index.js
module.exports.name = "蛙人"
module.exports.age = 24

let data = require("./index.js")
console.log(data) // { name: "蛙人", age: 24 }

```

#### ES6 Module
##### 使用
1. 使用场景：后来ES6版本正式加入了ES Module模块，成为浏览器和服务器通用的模块解决方案
2. 导出
```js
// 导出变量
export const name = "蛙人"
export const age = 24

// 导出方法也可以
export function fn() {}
export const test = () => {}


// 如果有多个的话
const name = "蛙人"
const sex = "male"
export { name, sex }
```
3. 导入
```js
// index,js
export const name = "蛙人"
export const age = 24

import { name, age } from './index.js' //这里的花括号跟解构不一样
console.log(name, age) // "蛙人" 24

// 如果里面全是单个导出，我们就想全部直接导入则可以这样写
import * as all from './index.js'
console.log(all) // {name: "蛙人", age: 24}
```
4. 混合导入
```js
// index,js
export const name = "蛙人"
export const age = 24
export default {
    msg: "蛙人"
}

import msg, { name, age } from './index.js'
console.log(msg) // { msg: "蛙人" }
```
### 运行原理 
#### CommonJS
##### 模块查找、加载、解析
1. 对于核心模块，node将其已经编译成二进制代码，直接书写标识符fs、http就可以
2. 对于自己写的文件模块，需要用‘./’'../'开头，require会将这种相对路径转化为真实路径，找到模块
3. 对于第三方模块，也就是使用npm下载的包，就会用到paths这个变量，会依次查找当前路径下的node_modules文件夹，如果没有，则在父级目录查找no_modules，一直到根目录下，找到为止。

##### 模块重复引用和循环引用
1. 重复引用问题。
   1. 模块加载是以绝对路径为key写入cache（require.cache）中，缓存可以解决重复查找和重复加载的问题。
2. 循环引用问题。
   1. 我们可以看到，在 CommonJS 规范中，当遇到 require() 语句时，会执行 require 模块中的代码，并缓存执行的结果，当下次再次加载时不会重复执行，而是直接取缓存的结果。正因为此，出现循环依赖时才不会出现无限循环调用的情况。虽然这种模块加载机制可以避免出现循环依赖时报错的情况，但稍不注意就很可能使得代码并不是像我们想象的那样去执行。
   2. CommonJS通过模块缓存来解决：每一个模块都先加入缓存再执行，每次遇到require都先检查缓存，这样就不会出现死循环；借助缓存，输出的值也很简单就能找到了。

#### ES6 Module

##### 查找、加载、解析
##### 模块重复引用和循环引用
1. 重复引用问题。
   1. 模块加载是以绝对路径为key写入cache（require.cache）中，缓存可以解决重复查找和重复加载的问题。
1. 循环引用问题。
   1. 跟 CommonJS 模块一样，ES6 不会再去执行重复加载的模块。ES Module借助模块地图，已经进入过的模块标注为获取中，遇到import语句会去检查这个地图，已经标注为获取中的则不会进入，地图中的每一个节点是一个模块记录，上面有导出变量的内存地址，导入时会做一个连接——即指向同一块内存。

### 两者区别
#### 加载运行的时机
* Commonjs是运行时候加载。因为导出是个对象，只有在运行时候才能生成，所以使用时可以放在地方。对外是动态定义。

* ES6 Module是编译时候加载。编译时候就确定模块依赖关系以及输入、输出的变量。对外导出只是个静态定义。所以使用的时候import必须放在文件的最开始，且前面不允许有其他逻辑代码。


#### 模块导出的值不一样
* commonjs是值的拷贝（注意浅拷贝基本数据类型和对象数据类型区别）， 可以修改导出的值，这在代码出错时，不好排查引起变量污染
```js
// dep.js
let a = 1
module.exports = {
    a: a
}
setTimeout(() => a += 1, 500)

// app.js
let a = require('dep');
setTimeout(function () {
  console.log(a) // 输出：1
}, 1000)
```

* es6是值的引用, 值都是可读的，不能修改
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


<br />

[参考文章1](https://juejin.cn/post/6938581764432461854#heading-17)

[参考文章2](https://es6.ruanyifeng.com/#docs/module)