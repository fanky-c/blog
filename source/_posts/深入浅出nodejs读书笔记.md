---
title: 深入浅出nodejs读书笔记
date: 2022-02-09 18:15:29
tags:
    - 深入浅出nodejs
    - 读书笔记
---

## nodejs模块机制
### commonjs介绍
commonjs对模块的定义十分简单，主要分为**模块引用、模块定义、模块标识3个部分。**
* 模块引用 
```js
var math = require('math');
```
* 模块定义
```js
// 在模块中，上下文提供require()方法来引入外部模块。
// 对应引入的功能，上下文提供了exports对象用于导出当前模块的方法或者变量，并且它是唯一导出的出口。
// 在模块中，还存在一个module对象，它代表模块自身，而exports是module的属性.
exports.add = function () {
  var sum = 0,
    i = 0,
    args = arguments,
    l = args.length;
  while (i < l) {
    sum += args[i++];
  }
  return sum;
};
```
* 模块标识
模块标识其实就是传递给require()方法的参数，它必须是符合小驼峰命名的字符串，或者以．、.．开头的相对路径，或者绝对路径

### commonjs模块实现

node模块分为2类：**node提供的核心模块 和 用户编写的文件模块；**核心模块部分在Node源代码的编译过程中，编译进了二进制执行文件，node启动时候已经加载到内存，文件定位和编译执行这两个步骤可以省略掉所以速度最快。

加载模块过程：**路径分析 --> 文件定位 --> 编译执行**

Node对引入过的模块都会进行缓存，第二次优先从缓存加载；浏览器仅仅缓存文件，而Node缓存的是编译和执行之后的对象；核心模块的缓存检查先于文件模块。
#### 1. 路径分析
模块路径是Node在定位文件模块的具体文件时制定的查找策略，**具体表现为一个路径组成的数组**
```js
// 1. 创建module_path.js文件，其内容为console.log(module.paths);
// 2. 将其放到任意一个目录中然后执行node module_path.js。

[ '/home/zc/research/node_modules',
'/home/zc/node_modules',
'/home/node_modules',
'/node_modules' ]
```
根据上面的例子可以的模块路径生成规则：
❑ 当前文件目录下的node_modules目录。
❑ 父目录下的node_modules目录。
❑ 父目录的父目录下的node_modules目录。
❑ 沿路径向上逐级递归，直到根目录下的node_modules目录。

**总结：生成方式与JavaScript的原型链或作用域链的查找方式十分类似。在加载的过程中，Node会逐个尝试模块路径中的路径，直到找到目标文件为止。**
  

#### 2. 文件定位
* 文件扩展名分析
Node会按．js、.json、.node的次序补足扩展名，依次尝试； 在尝试过程中，调用fs模块同步阻塞式地判断文件是否存在。

* 目录分析和包
require()通过分析文件扩展名之后，可能没有查找到对应文件，但却得到一个目录，此时Node会将目录当做一个包来处理；优先找到目录下package.json文件下main属性当入口； 如果压根没有package.json文件，Node会将index当做默认文件名，然后依次查找index.js、index.json、index.node

#### 3. 编译执行
编译和执行是引入文件模块的最后一个阶段。定位到具体的文件后，**Node会新建一个模块对象，然后根据路径载入并编译。**
```js
//模块对象
function Module(id, parent) {
  this.id = id;
  this.exports = {};
  this.parent = parent;
  if (parent && parent.children) {
    parent.children.push(this);
  }

  this.filename = null;
  this.loaded = false;
  this.children = [];
}
```

**每一个编译成功的模块都会将其文件路径作为索引缓存在Module._cache对象上，以提高二次引入的性能**  

* 读取文件方式
  * .js文件， 通过fs模块同步读取
  * .node文件， 这是c/c++编写的扩展文件，通过dlopen()加载编译成文件
  * .json文件，通过fs模块同步读取， **然后json.parse()解析（反序列化：字节序列转化成对象的过程）**

* 模块编译过程
commonjs规范规定每个模块文件中存在着require、exports、module这3个变量，但是它们在模块文件中并没有定义，那么从何而来呢？甚至在Node的API文档中，我们知道每个模块中还有\__filename、\__dirname这两个变量的存在，它们又是从何而来的呢。

事实上，在编译的过程中，Node对获取的JavaScript文件内容进行了头尾包装。在头部添加了(function (exports, require, module, \__filename, \__dirname) {\n，在尾部添加了\n});。一个正常的JavaScript文件会被包装成如下的样子：
```js
(function (exports, require, module, __filename, __dirname) {
  var math = require('math');
  exports.area = function (radius) {
    return Math.PI * radius * radius;
  };
});
```
这样每个模块文件之间都进行了作用域隔离。包装之后的代码会通过vm原生模块的runInThisContext()方法执行（类似eval，只是具有明确上下文，不污染全局），返回一个具体的function对象；最后，将当前模块对象的exports属性、require()方法、module（模块对象自身），以及在文件定位中得到的完整文件路径和文件目录作为参数传递给这个function()执行。

上面是这些变量并没有定义在每个模块文件中却存在的原因。在执行之后，模块的exports属性被返回给了调用方。exports属性上的任何方法和属性都可以被外部调用到，但是模块中的其余变量或属性则不可直接被调用。



## 异步io

## 异步编程

## 内存控制

## 理解buffer

## 网络编程

## 进程
