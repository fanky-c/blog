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
### 为什么要异步io
* 用户体验： 同步io会影响接口响应时间（M+N+..），异步io会快速响应接口(max(M,N,..))
* 资源分配，目前主流的方式：单线程串行执行 和 多线程并行执行
  * 多线程， 缺点：多线程编程经常面临锁、状态同步等问题； 优点：在多核cpu上能够有效提升cpu利用率
  * 单线程， 缺点：阻塞io导致硬件资源得不到充分利用；**在计算机资源中，通常I/O与CPU计算之间是可以并行进行的。但是同步的编程模型导致的问题是，I/O的进行会让后续任务等待，这造成资源不能被更好地利用。** 优点：同步利于程序员编码
  * **node单线程异步io方案：利用单线程，远离多线程死锁、状态同步等问题；利用异步I/O，让单线程远离阻塞，以更好地使用CPU；为了弥补单线程无法利用多核CPU的缺点，Node提供了类似前端浏览器中WebWorkers的子进程，该子进程可以通过工作进程高效地利用CPU和I/O。**

### 异步io现状
* 异步io和非阻塞io
  >异步与非阻塞听起来似乎是同一回事。从实际效果而言，异步和非阻塞都达到了我们并行I/O的目的。 但是从计算机内核I/O而言，异步/同步和阻塞/非阻塞实际上是两回事。操作系统内核对于I/O只有两种方式：阻塞与非阻塞。
  任意技术都并非完美的。阻塞I/O造成CPU等待浪费，非阻塞带来的麻烦却是需要轮询去确认是否完全完成数据获取，它会让CPU处理状态判断，是对CPU资源的浪费。这里我们且看轮询技术是如何演进的，以减小I/O状态判断的CPU损耗。

* 现存轮询技术
  * read: 它是最原始、性能最低的一种，通过重复调用来检查I/O的状态来完成完整数据的读取。在得到最终数据前，CPU一直耗用在等待上。
  <img src="/img/node5.jpeg" style="max-width:95%" />
  * select: 它是在read的基础上改进的一种方案，通过对文件描述符上的事件状态来进行判断。
  * poll: 该方案较select有所改进，采用链表的方式避免数组长度的限制，其次它能避免不需要的检查。但是当文件描述符较多的时候，它的性能还是十分低下的。
  * epoll: 该方案是Linux下效率最高的I/O事件通知机制，在进入轮询的时候如果没有检查到I/O事件，将会进行休眠，直到事件发生将它唤醒。它是真实利用了事件通知、执行回调的方式，而不是遍历查询，所以不会浪费CPU，执行效率较高。
  <img src="/img/node4.jpeg" style="max-width:95%" />

### node如何实现异步io
* 事件循环
  在进程启动时，Node便会创建一个类似于while(true)的循环，每执行一次循环体的过程我们称为Tick。
  <img src="/img/node1.jpeg" style="max-width:95%" />

* 观察者
  在每个Tick的过程中，如何判断是否有事件需要处理呢？**这里必须要引入的概念是观察者。每个事件循环中有一个或者多个观察者，而判断是否有事件要处理的过程就是向这些观察者询问是否有要处理的事件。**
  > 浏览器采用了类似的机制。事件可能来自用户的点击或者加载某些文件时产生，而这些产生的事件都有对应的观察者。在Node中，事件主要来源于网络请求、文件I/O等，这些事件对应的观察者有文件I/O观察者、网络I/O观察者等。观察者将事件进行了分类。
  > 事件循环是一个典型的生产者/消费者模型。异步I/O、网络请求等则是事件的生产者，源源不断为Node提供不同类型的事件，这些事件被传递到对应的观察者那里，事件循环则从观察者那里取出事件并处理

* 请求对象
  从JavaScript代码到系统内核之间都发生了什么？
  ```js
    fs.open = function(path, flags, mode, callback) {
    // ...
    binding.open(pathModule._makeLong(path),
                  stringToFlags(flags),
                  mode,
                  callback);
  };
  ```
  <img src="/img/node2.jpeg" style="max-width:95%" />
  JavaScript线程可以继续执行当前任务的后续操作。当前的I/O操作在线程池中等待执行，不管它是否阻塞I/O，都不会影响到JavaScript线程的后续执行，如此就达到了异步的目的。

* 执行回调
  **事件循环、观察者、请求对象、I/O线程池这四者共同构成了Node异步I/O模型的基本要素**
  <img src="/img/node3.jpeg" style="max-width:95%" />

* 总结
  我们可以提取出异步I/O的几个关键词：单线程、事件循环、观察者和I/O线程池。这里单线程与I/O线程池之间看起来有些悖论的样子。由于我们知道JavaScript是单线程的，所以按常识很容易理解为它不能充分利用多核CPU。事实上，在Node中，除了JavaScript是单线程外，Node自身其实是多线程的，只是I/O线程使用的CPU较少。另一个需要重视的观点则是，除了用户代码无法并行执行外，所有的I/O（磁盘I/O和网络I/O等）则是可以并行起来的。

### node中非io的异步api
* 定时器（setTimout/setInterval）
  调用setTimeout()或者setInterval()创建的定时器会被插入到定时器观察者内部的一个红黑树中。每次Tick执行时，会从该红黑树中迭代取出定时器对象，检查是否超过定时时间，如果超过，就形成一个事件，它的回调函数将立即执行。


* process.nextTick()
  每次调用process.nextTick()方法，只会将回调函数放入队列中，在下一轮Tick时取出执行。定时器中采用红黑树的操作时间复杂度为O(lg(n)), nextTick()的时间复杂度为O(1)。相较之下，process.nextTick()更高效。

* setImmediate()
  和process.nextTick功能一样

* 优先级对比
  process.nextTick > setImmediate； 这里的原因在于事件循环对观察者的检查是有先后顺序的，process.nextTick()属于idle观察者，setImmediate()属于check观察者。在每一个轮循环检查中，**idle观察者先于I/O观察者，I/O观察者先于check观察者**

### 事件驱动和高性能服务器
* 经典服务器模型
  * 同步式：一次只能处理一个请求，并且其余请求都处于等待状态。
  * 每进程/每请求：为每个请求启动一个进程，这样可以处理多个请求，但是它不具备扩展性，因为系统资源只有那么多。
  * 每线程/每请求：为每个请求启动一个线程来处理。尽管线程比进程要轻量，但是由于每个线程都占用一定内存，当大并发请求到来时，内存将会很快用光，导致服务器缓慢。(Apache服务)
  * **事件驱动：Node通过事件驱动的方式处理请求，无须为每一个请求创建额外的对应线程，可以省掉创建线程和销毁线程的开销，同时操作系统在调度任务时因为线程较少，上下文切换的代价很低。（node、nginx）**

## 异步编程

## 内存控制

## 理解buffer

## 网络编程

## 进程
