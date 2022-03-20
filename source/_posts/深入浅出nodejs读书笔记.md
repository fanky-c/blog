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
### 函数数编程

**高阶函数则是可以把函数作为参数，或是将函数作为返回值的函数。**高阶函数在JavaScript中比比皆是，其中ECMAScript5中提供的一些数组方法（forEach()、map()、reduce()、reduceRight()、filter()、every()、some()）十分典型。

**偏函数用法是指创建一个调用另外一个部分——参数或变量已经预置的函数——的函数的用法。**
```js
// 这种通过指定部分参数来产生一个新的定制函数的形式就是偏函数
let isType = function (type) {
  return function (obj) {
    return toString.call(obj) == '[object ' + type + ']';
  };
};

let isString = isType('String');
let isFunction = isType('Function');
```

### 异步编程优势和难点
#### A、优势
**Node带来的最大特性莫过于基于事件驱动的非阻塞I/O模型，这是它的灵魂所在。非阻塞I/O可以使CPU与I/O并不相互依赖等待，让资源得到更好的利用**

<img src="/img/node6.jpeg" style="max-width:95%" />

利用事件循环的方式，**JavaScript线程像一个分配任务和处理结果的大管家，I/O线程池里的各个I/O线程都是小二，负责兢兢业业地完成分配来的任务，小二与管家之间互不依赖，所以可以保持整体的高效率。**这个利用事件循环的经典调度方式在很多地方都存在应用，最典型的是UI编程，如iOS应用开发等。

Node是为了解决编程模型中阻塞I/O的性能问题的，采用了单线程模型，**这导致Node更像一个处理I/O密集问题的能手，而CPU密集型则取决于管家的能耐如何**

由于事件循环模型需要应对海量请求，海量请求同时作用在单线程上，就需要防止任何一个计算耗费过多的CPU时间片。**至于是计算密集型，还是I/O密集型，只要计算不影响异步I/O的调度，那就不构成问题。**

#### B、缺点
1. 异常处理（最大缺点）
   > 异步I/O的实现主要包含两个阶段：**提交请求和处理结果。这两个阶段中间有事件循环的调度，两者彼此不关联。异步方法则通常在第一个阶段提交请求后立即返回，因为异常并不一定发生在这个阶段，try/catch的功效在此处不会发挥任何作用**
   
   ```js
     // callback被存放起来，直到下一个事件循环（Tick）才会取出来执行
     var async = function (callback) {
       process.nextTick(callback);
     };

     try {
       async(callback);
     } catch (e) {
       // TODO
     }     
   ```

2. 函数嵌套过深, 目前node较高版本已支持async/await模式解决了函数嵌套问题
3. 多线程编程
   > 浏览器提出了Web Workers，它通过将JavaScript执行与UI渲染分离，可以很好地利用多核CPU为大量计算服务。同时前端Web Workers也是一个利用消息机制合理使用多核CPU的理想模型
   <img src="/img/node7.jpeg" style="max-width:95%" />

   Node借鉴了浏览器web workers这个模式，**child_process是其基础API, cluster模块是更深层次的应用。**借助Web Workers的模式，开发人员要更多地去面临跨线程的编程，这对于以往的JavaScript编程经验是较少考虑的。


4. 异步转同步， 目前node较高版本已支持async/await模式

### 异步编程解决方案
#### 1. 事件发布/订阅模式
> 事件侦听器模式也是一种钩子（hook）机制，利用钩子导出内部数据或状态给外部的调用者。 Node自身提供的events模块是发布/订阅模式的一个简单实现，Node中部分模块都继承自它。

EventProxy的原理
```js
// 核心代码

```

#### 2. Promise/Deferred模式

#### 3. async/await模式

#### 4. 流程控制库
1. 尾触发和next(Connect中间件)
   > 除了事件和Promise外，还有一类方法是需要手工调用才能持续执行后续调用的，我们将此类方法叫做尾触发，常见的关键词是next

   ```js
     var app = connect();
     // Middleware
     app.use(connect.staticCache());
     app.use(connect.static(__dirname + '/public'));
     app.use(connect.cookieParser());
     app.use(connect.session());
     app.use(connect.query());
     app.use(connect.bodyParser());
     app.use(connect.csrf());
     app.listen(3001);
   ```

   **connect核心代码**
```js
function app(req, res){ app.handle(req, res); }

// use方法
app.use = function(route, fn){
 // some code
 this.stack.push({ route: route, handle: fn });
 return this;
};

function createServer() {
 function app(req, res){ app.handle(req, res); }
 utils.merge(app, proto);
 utils.merge(app, EventEmitter.prototype);
 app.route = '/';
 app.stack = [];
 for (var i = 0; i < arguments.length; ++i) {
   app.use(arguments[i]);
 }
 return app;
};

app.listen = function(){
 var server = http.createServer(this);
 return server.listen.apply(server, arguments);
};

app.handle = function(req, res, out) {
 // some code
 next();
};

// 原始的next()方法较为复杂，下面是简化后的内容，
// 其原理十分简单，取出队列中的中间件并执行，同时传入当前方法以实现递归调用，达到持续触发的目的
function next(err){
  // some code
  // next callback
  layer = stack[index++];
  layer.hander(req, res, next);
}

```


1. async流程库： 异步的串行执行
   ```js
     async.series([
       function (callback) {
         fs.readFile('file1.txt', 'utf-8', callback);
       },
       function (callback) {
         fs.readFile('file2.txt', 'utf-8', callback);
       }
     ], function (err, results) {
       // results => [file1.txt, file2.txt]
     });
   ```

### 异步并发控制
> 目前社区中有bigpipe、async异步流程库解决方案    

bigpipe核心实现

  ```js
        /**
         * 推入方法，参数。最后一个参数为回调函数
         * @param {Function} method异步方法
         * @param {Mix} args参数列表，最后一个参数为回调函数
         */
        Bagpipe.prototype.push = function (method) {
          var args = [].slice.call(arguments, 1);
          var callback = args[args.length -1];
          if (typeof callback ! == 'function') {
            args.push(function () {});
          }
          if (this.options.disabled || this.limit < 1) {
            method.apply(null, args);
            return this;
          }

          // 队列长度也超过限制值时
          if (this.queue.length < this.queueLength || ! this.options.refuse) {
            this.queue.push({
              method: method,
              args: args
            });
          } else {
            var err = new Error('Too much async call in queue');
            err.name = 'TooMuchAsyncCallError';
            callback(err);
          }

          if (this.queue.length > 1) {
            this.emit('full', this.queue.length);
          }
          this.next();
          return this;
        };

        /*
         * 继续执行队列中的后续动作
         */
        Bagpipe.prototype.next = function () {
          var that = this;
          if (that.active < that.limit && that.queue.length) {
            var req = that.queue.shift();
            that.run(req.method, req.args);
          }
        };  
        
        /*
         * 执行队列中的方法
         */
        Bagpipe.prototype.run = function (method, args) {
          var that = this;
          that.active++;
          var callback = args[args.length -1];
          var timer = null;
          var called = false;

          // inject logic
          args[args.length -1] = function (err) {
            // anyway, clear the timer
            if (timer) {
              clearTimeout(timer);
              timer = null;
            }
            // if timeout, don't execute
            if (! called) {
              that._next();
              callback.apply(null, arguments);
            } else {
              // pass the outdated error
              if (err) {
              that.emit('outdated', err);
              }
            }
          };

          var timeout = that.options.timeout;
          if (timeout) {
            timer = setTimeout(function () {
              // set called as true
              called = true;
              that._next();
              // pass the exception
              var err = new Error(timeout + 'ms timeout');
              err.name = 'BagpipeTimeoutError';
              err.data = {
                name: method.name,
                method: method.toString(),
                args: args.slice(0, -1)
              };
              callback(err);
            }, timeout);
          }
          method.apply(null, args);
        };
  ```

## 内存控制
### V8的垃圾回收机制和内存限制
#### 1. V8的内存限制
> 至于V8为何要限制堆的大小，表层原因为V8最初为浏览器而设计，不太可能遇到用大量内存的场景。对于网页来说，V8的限制值已经绰绰有余。深层原因是V8的垃圾回收机制的限制。按官方的说法，以1.5 GB的垃圾回收堆内存为例，V8做一次小的垃圾回收需要50毫秒以上，做一次非增量式的垃圾回收甚至要1秒以上。这是垃圾回收中引起JavaScript线程暂停执行的时间，在这样的时间花销下，应用的性能和响应能力都会直线下降


* 关于V8，作为虚拟机，V8的性能表现优异。
* Node中通过JavaScript使用内存时就会发现只能使用部分内存（64位系统下约为1.4 GB,32位系统下约为0.7 GB）。

**手动打开node内存限制**  
```js
// 设置老生代内存空间的最大值
node --max-old-space-size=1700 test.js // 单位为MB
// 或者 设置新生代内存空间的大小
node --max-new-space-size=1024 test.js // 单位为KB
```


#### 2. V8的对象分配
在V8中，所有的JavaScript对象都是通过**堆来进行分配的**。
```js
// heapTotal和heapUsed是V8的堆内存使用情况，
// heapTotal: 前者是已申请到的堆内存，
// heapUsed: 后者是当前使用的量。
// 至于rss为常驻内存
$ node
> process.memoryUsage();
{ rss: 14958592,
  heapTotal: 7195904,
  heapUsed: 2821496 
}
```

#### 3. V8的垃圾回收机制
> V8的垃圾回收策略主要基于分代式垃圾回收机制。在自动垃圾回收的演变过程中，人们发现没有一种垃圾回收算法能够胜任所有的场景。因为在实际的应用中，对象的生存周期长短不一，不同的算法只能针对特定情况具有最好的效果

1. a、v8的内存分代
在V8中，主要将内存分为新生代和老生代两代。新生代中的对象为存活时间较短的对象，老生代中的对象为存活时间较长或常驻内存的对象
<img src="/img/node8.jpeg" style="max-width:95%" />

2. b、Scavenge算法
在Scavenge的具体实现中，主要采用了Cheney算法； Cheney算法是一种采用复制的方式实现的垃圾回收算法。它将堆内存一分为二，每一部分空间称为semispace。在这两个semispace空间中，只有一个处于使用中，另一个处于闲置状态。处于使用状态的semispace空间称为From空间，处于闲置状态的空间称为To空间。当我们分配对象时，先是在From空间中进行分配。当开始进行垃圾回收时，会检查From空间中的存活对象，这些存活对象将被复制到To空间中，而非存活对象占用的空间将会被释放。

<img src="/img/node9.jpeg" style="max-width:95%" />

当一个对象经过多次复制依然存活时，它将会被认为是生命周期较长的对象。这种较长生命周期的对象随后会被移动到老生代中，采用新的算法进行管理。对象从新生代中移动到老生代中的过程称为晋升。
**对象晋升的条件主要有两个：1、一个是对象是否经历过Scavenge回收，2、一个是To空间的内存（to空间超过25%）占用比超过限制。**



3. c、Mark-Sweep & Mark-Compact (标记清除 & 标记整理)
> 对于老生代中的对象，由于存活对象占较大比重，再采用Scavenge的方式会有两个问题：一个是存活对象较多，复制存活对象的效率将会很低；另一个问题依然是浪费一半空间的问题。这两个问题导致应对生命周期较长的对象时Scavenge会显得捉襟见肘。为此，V8在老生代中主要采用了Mark-Sweep和Mark-Compact相结合的方式进行垃圾回收

  1. **Mark-Sweep是标记清除的意思，它分为标记和清除两个阶段。**Mark-Sweep只清理死亡对象。活对象在新生代中只占较小部分，死对象在老生代中只占较小部分，这是两种回收方式能高效处理的原因
  <img src="/img/node11.jpeg" style="max-width:95%" />

  2. **Mark-Compact是标记整理的意思，是在Mark-Sweep的基础上演变而来的。**它们的差别在于对象在标记为死亡后，在整理的过程中，将活着的对象往一端移动，移动完成后，直接清理掉边界外的内存
  <img src="/img/node12.jpeg" style="max-width:95%" />


4. d、3种垃圾回收算法的简单对比
<img src="/img/node10.jpeg" style="max-width:95%" />

5. e、增量标记
为了避免出现JavaScript应用逻辑与垃圾回收器看到的不一致的情况，垃圾回收的3种基本算法都需要将应用逻辑暂停下来，待执行完垃圾回收后再恢复执行应用逻辑，这种行为被称为“全停顿”（stop-the-world）
<img src="/img/node13.jpeg" style="max-width:95%" />

6. f、查看垃圾回收日志
   1. 查看垃圾回收日志的方式主要是在启动时添加--trace_gc参数
   2. 通过在Node启动时使用--prof参数，可以得到V8执行时的性能分析数据，其中包含了垃圾回收执行时占用的时间 

### 查看内存指标
>应用中存在一些全局性的对象是正常的，而且在正常的使用中，变量都会自动释放回收。但是也会存在一些我们认为会回收但是却没有被回收的对象，这会导致内存占用无限增长。一旦增长达到V8的内存限制，将会得到内存溢出错误，进而导致进程退出

#### 查看内存使用情况
1. a、查看进程内存占用：**process.memoryUsage()可以看到Node进程的内存占用情况**
  ```js
    var showMem = function () {
    var mem = process.memoryUsage();
    var format = function (bytes) {
      return (bytes / 1024 / 1024).toFixed(2) + ' MB';
    };
    // heapTotal：堆中总申请的内存量；heapUsed：堆中使用中内存量
    console.log('Process: heapTotal ' + format(mem.heapTotal) +
      ' heapUsed ' + format(mem.heapUsed) + ' rss ' + format(mem.rss));
  };
  ```

2. b、查看系统的内存占用
 os模块中的totalmem()和freemem()这两个方法用于查看操作系统的内存使用情况

#### 堆外内存
>通过process.memoryUsage()的结果可以看到，堆中的内存用量总是小于进程的常驻内存用量，这意味着Node中的内存使用并非都是通过V8进行分配的。我们将那些不是通过V8分配的内存称为堆外内存

```js
var useMem = function () {
 var size = 200 * 1024 * 1024;
 var buffer = new Buffer(size);
 for (var i = 0; i < size; i++) {
   buffer[i] = 0;
 }
 return buffer;
};
```
heapTotal与heapUsed的变化极小，唯一变化的是rss的值，并且该值已经远远超过V8的限制值。**这其中的原因是Buffer对象不同于其他对象，它不经过V8的内存分配机制，所以也不会有堆内存的大小限制。**

>为何Buffer对象并非通过V8分配？这在于Node并不同于浏览器的应用场景。在浏览器中，JavaScript直接处理字符串即可满足绝大多数的业务需求，而Node则需要处理网络流和文件I/O流，操作字符串远远不能满足传输的性能需求。

#### 总结
**node的内存构成主要由通过V8进行分配的部分和node自行分配部分，受V8的垃圾回收机制限制主要是V8的堆内存**

### 内存泄漏
>造成内存泄漏的原因有如下几个: **1、缓存 2、队列消费不及时 3、作用域未释放**

#### 1. 慎将内存当缓存使用
>一旦一个对象被当做缓存来使用，那就意味着它将会常驻在老生代中。缓存中存储的键越多，长期存活的对象也就越多，这将导致垃圾回收在进行扫描和整理时，对这些对象做无用功。**所以在Node中，任何试图拿内存当缓存的行为都应当被限制。当然，这种限制并不是不允许使用的意思，而是要小心为之**

1. a、缓存限制策略
   需要加入一种策略来限制缓存无限制增长，**例如LRU算法的缓存算法**
   ```js
        var LimitableMap = function (limit) {
          this.limit = limit || 10;
          this.map = {};
          this.keys = [];
        };

        var hasOwnProperty = Object.prototype.hasOwnProperty;

        LimitableMap.prototype.set = function (key, value) {
          var map = this.map;
          var keys = this.keys;
          if (! hasOwnProperty.call(map, key)) {
            if (keys.length === this.limit) {
              var firstKey = keys.shift();
              delete map[firstKey];
            }
            keys.push(key);
          }
          map[key] = value;
        };

        LimitableMap.prototype.get = function (key) {
          return this.map[key];
        };

        module.exports = LimitableMap;
   ```

2. b、缓存的解决方案
   1. 将缓存转移到外部，减少常驻内存的对象的数量，让垃圾回收更高效
   2. 进程之间可以共享缓存： 例如redis


#### 2. 关注队列状态
在大多数应用场景下，消费的速度远远大于生产的速度，内存泄漏不易产生。但是一旦消费速度低于生产速度，将会形成堆积。

**例子：**有的应用会收集日志。如果欠缺考虑，也许会采用数据库来记录日志。日志通常会是海量的，数据库构建在文件系统之上，写入效率远远低于文件直接写入，于是会形成数据库写入操作的堆积，而JavaScript中相关的作用域也不会得到释放，内存占用不会回落，从而出现内存泄漏

**解决方案：**
 1. 通过监控系统产生报警并通知相关人员
 2. 任意异步调用都应该包含超时机制，一旦在限定的时间内未完成响应，通过回调函数传达超时异常，启动拒绝模式


### 内存排查
1. 常见的内存排查工具
   1. v8-profiler：它可以用于对V8堆内存抓取快照和对CPU进行分析
   2. node-heapdump：它允许对V8堆内存抓取快照，用于事后分析。
   3. node-memwatch：

2. 总结
   **排查内存泄漏的原因主要通过对堆内存进行分析而找到**

### 大内存应用 
>在Node中，不可避免地还是会存在操作大文件的场景。由于Node的内存限制，操作大文件也需要小心，好在Node提供了stream模块用于处理大文件；Node中的大多数模块都有stream的应用，比如fs的createReadStream()和createWriteStream()方法可以分别用于创建文件的可读流和可写流，process模块中的stdin和stdout则分别是可读流和可写流的示例。

1. fs.createReadStream 和 fs.createWriteStream 使用

```js
var reader = fs.createReadStream('in.txt');
var writer = fs.createWriteStream('out.txt');
reader.on('data', function (chunk) {
 writer.write(chunk);
});
reader.on('end', function () {
 writer.end();
});

// 另外一种写法
var reader = fs.createReadStream('in.txt');
var writer = fs.createWriteStream('out.txt');
reader.pipe(writer);
```
可读流提供了管道方法pipe()，封装了data事件和写入操作。通过流的方式，上述代码不会受到V8内存限制的影响，有效地提高了程序的健壮性。

## 理解buffer
>由于应用场景不同，在Node中，应用需要处理网络协议、操作数据库、处理图片、接收上传文件等，在网络流和文件的操作中，还要处理大量二进制数据，JavaScript自有的字符串远远不能满足这些需求，于是Buffer对象应运而生

### Buffer的结构
1. a、介绍
 1. 1、由于Buffer太过常见，Node在进程启动时就已经加载了它，并将其放在全局对象（global）上。所以在使用Buffer时，无须通过require()即可直接使用；
 2. 2、Buffer是一个典型的JavaScript与C++结合的模块，它将性能相关部分用C++实现，将非性能相关的部分用JavaScript实现。
 3. 3、Buffer所占用的内存不是通过V8分配的，属于堆外内存。

2. b、buffer对象
Buffer对象类似于数组，它的元素为16进制的两位数，即0到255的数值。
```js
var str = "深入浅出node.js";
var buf = new Buffer(str, 'utf-8');
console.log(buf); // <Buffer e6 b7 b1 e5 85 a5 e6 b5 85 e5 87 ba 6e 6f 64 65 2e 6a 73>

// 
var buf = new Buffer(100);
console.log(buf.length); //  100
buf[10] = 123;
console.log(buf[10]); //  123
```

3. c、buffer的内存分配
  * Buffer对象的内存分配不是在V8的堆内存中，而是在Node的C++层面实现内存的申请的。因为处理大量的字节数据不能采用需要一点内存就向操作系统申请一点内存的方式，这可能造成大量的内存申请的系统调用，对操作系统有一定压力。为此Node在内存的使用上应用的是在C++层面申请内存、在JavaScript中分配内存的策略
  * slab机制介绍：xxxxx


4. d、总结
真正的内存是在Node的C++层面提供的，JavaScript层面只是使用它。当进行小而频繁的Buffer操作时，采用slab的机制进行**预先申请和事后分配，**使得JavaScript到操作系统之间不必有过多的内存申请方面的系统调用。对于大块的Buffer而言，则直接使用C++层面提供的内存，而无需细腻的分配操作。

### Buffer的转换
1. a、字符串转buffer
```js
//默认按UTF-8编码进行转码和存储
new Buffer(str, [encoding]);

buf.write(string, [offset], [length], [encoding]);
```

2. b、buffer转字符串
```js
buf.toString([encoding], [start], [end]);
```

### Buffer的拼接
1. buffer拼接
```js
var fs = require('fs');
var rs = fs.createReadStream('test.md');
var data = '';
rs.on("data", function (chunk){
 data += chunk; // data = data.toString() + chunk.toString();
});
rs.on("end", function () {
 console.log(data);
});
```
2. 中文乱码原因
   1. 模拟出现乱码：
   ```js
   // 将文件可读流的每次读取的Buffer长度限制为11
   var rs = fs.createReadStream('test.md', {highWaterMark: 11});
   ```
   <img src="/img/node14.jpeg" style="max-width:95%" />
   1. 上面的诗歌中，“月”、“是”、“望”、“低”4个字没有被正常输出，取而代之的是3个?。产生这个输出结果的原因在于文件可读流在读取时会逐个读取Buffer；
   2. 由于我们限定了Buffer对象的长度为11，因此只读流需要读取7次才能完成完整的读取；
   3. 上文提到的buf.toString()方法默认以UTF-8为编码，中文字在UTF-8下占3个字节。所以第一个Buffer对象在输出时，只能显示3个字符，Buffer中剩下的2个字节（e6 9c）将会以乱码的形式显示。第二个Buffer对象的第一个字节也不能形成文字，只能显示乱码。于是形成一些文字无法正常显示的问题；


   2. 临时解决方案
   ```js
    var rs = fs.createReadStream('test.md', { highWaterMark: 11});
    //该方法的作用是让data事件中传递的不再是一个Buffer对象，而是编码后的字符串
    fs.setEncoding('utf8');
   ```
   虽然string_decoder模块很奇妙，但是它也并非万能药，它目前只能处理UTF-8、Base64和UCS-2/UTF-16LE这3种编码。所以，通过setEncoding()的方式不可否认能解决大部分的乱码问题，但并不能从根本上解决该问题
   
3. 正确拼接buffer
   1. 通过iconv模块来转码
   2. 修改+=改成Buffer.concat()生成一个合并的Buffer对象
```js
var chunks = [];
var size = 0;
res.on('data', function (chunk) {
 chunks.push(chunk);
 size += chunk.length;
});
res.on('end', function () {
 var buf = Buffer.concat(chunks, size);
 var str = iconv.decode(buf, 'utf8');
 console.log(str);
});
```
### Buffer的性能

## 网络编程

## 进程

## 构建web应用