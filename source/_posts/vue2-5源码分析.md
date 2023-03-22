---
title: vue2.5源码分析
date: 2023-02-17 18:31:22
tags:
  - vue2.5.2
---


## 1、带着问题看书
1. vue响应式原理，为啥修改数据视图会自动更新？
2. 虚拟dom的概念和原理，为啥要引入vnode?
3. 模板编译原理，vue的模板是如何生效？
4. vue整体架构设计和项目结构？
5. 不同的生命周期钩子之间的区别，不同的生命周期之间vue内部到底做了什么？
6. vue提供全局的api内部原理？
7. 过滤器实现原理？

## 2、vue介绍
### 1、什么是渐进式框架
所谓渐进式框架（The Progressive Framework），就是把框架分层。

最核心的部分是视图层渲染，然后往外是组件机制，在这个基础上再加入路由机制，再加入状态管理，最外层是构建工具。 

所谓分层，就是说你既可以只用最核心的视图层渲染功能来快速开发一些需求，也可以使用一整套全家桶来开发大型应用。Vue.js有足够的灵活性来适应不同的需求，所以你可以根据自己的需求选择不同的层级。

<img src="/img/vue1.jpeg" style="max-width:95%" />

### 2、为啥要引入虚拟DOM
Vue.js引入虚拟DOM是有原因的。事实上，并不是引入虚拟DOM后，渲染速度变快了。准确地说，应该是80% 的场景下变得更快了，而剩下的20% 反而变慢了。

任何技术的引入都是在解决一些问题，而通常解决一个问题的同时会引发另外一个问题，这种情况更多的是做权衡，做取舍。

## 3、变化侦测
### 1、介绍
**从状态生成DOM，再输出到用户界面显示的一整套流程叫作渲染，应用在运行时会不断地进行重新渲染。**而响应式系统赋予框架重新渲染的能力，其重要组成部分是变化侦测。变化侦测是响应式系统的核心，没有它，就没有重新渲染。框架在运行时，视图也就无法随着状态的变化而变化。

### 2、Object的变化侦测
大部分人不会想到Object和Array的变化侦测采用不同的处理方式。事实上，它们的侦测方式确实不一样。

#### 2.1 什么是变化侦测
Vue.js会自动通过状态生成DOM，并将其输出到页面上显示出来，这个过程叫渲染。Vue.js的渲染过程是声明式的，我们通过模板来描述状态与DOM之间的映射关系。

变化侦测就是用来解决这个问题的，它分为两种类型：一种是“推”（push），另一种是“拉”（pull）。

Angular和React中的变化侦测都属于“拉”，这就是说当状态发生变化时，它不知道哪个状态变了，只知道状态有可能变了，然后会发送一个信号告诉框架，框架内部收到信号后，会进行一个暴力比对来找出哪些DOM节点需要重新渲染。这在Angular中是脏检查的流程，在React中使用的是虚拟DOM。

而Vue.js的变化侦测属于“推”。当状态发生变化时，Vue.js立刻就知道了，而且在一定程度上知道哪些状态变了。因此，它知道的信息更多，也就可以进行更细粒度的更新。

更细粒度的更新也有一定代价，因为粒度越细，每个状态所绑定的依赖就越多，依赖追踪在内存上的开销就会越大。

#### 2.2 如何追踪变化
**两种方法可以侦测到变化：使用Object.defineProperty 和 ES6的Proxy。**


```js
function defineReactive(data, key, val){
   Object.defineProperty(data, key, {
      enumerable: true,
      configurable: true,
      get: function(){
        return val;
      },
      set: function(newVal){
        if(val !== newVal){
           val = newVal;
        }
      }
   });
}
```

封装好之后，每当从data的key中读取数据时，get函数被触发；每当往data的key中设置数据时，set函数被触发。

#### 2.3 如何依赖收集
如果只是把Object.defineProperty进行封装，那其实并没什么实际用处，真正有用的是收集依赖。

下面例子中，该模板中使用了数据name，所以name它发生变化时，要向使用了它的地方发送通知。

```js
<temple>
 <h1>{{ name }}</h1>
</temple>
```

*在Vue.js 2.0中，模板使用数据等同于组件使用数据，所以当数据发生变化时，会将通知发送到组件，然后组件内部再通过虚拟DOM重新渲染。*

对于上面的问题，我的回答是：**先收集依赖，即把用到数据name的地方收集起来，然后等属性发生变化时，把之前收集好的依赖循环触发一遍就好了。总结起来，其实就一句话，在getter中收集依赖，在setter中触发依赖。**

#### 2.4 依赖收集在哪里
思考一下，首先想到的是每个key都有一个数组，用来存储当前key的依赖。假设依赖是一个函数，保存在window.target上，现在就可以把defineReactive函数稍微改造一下：

```js
function defineReactive(data, key, val) {
  const dep = []; // 新增
  Object.defineProperty(data, key, {
    enumerable: true,
    configurable: true,
    get: function() {
      dep.push(window.target);  // 新增, window.target是啥东西？？？
      return val;
    },
    set: function(newVal){
      if(val === newVal) return;
      // 新增
      if(let i=0; i<dep.length; i++){
         dep(i)[newVal, val];
      }
      val = newVal;
    }
  });
}
```
这里我们新增了数组dep，用来存储被收集的依赖；然后在set被触发时，循环dep以触发收集到的依赖。

但是这样写有点耦合，我们把依赖收集的代码封装成一个Dep类，它专门帮助我们管理依赖。使用这个类，我们可以收集依赖、删除依赖或者向依赖发送通知等。其代码如下：

```js
export default class Dep {
  constructor(){
    this.subs = [];
  }
  
  addSub(sub){
    this.subs.push(sub);
  }
  
  removeSub(sub){
    remove(this.subs. sub);
  }
  
  depend(){
    if(window.target){
       this.addSub(window.target);
    }
  }

  notify(){
    const subs = this.subs.slice();
    for(let i=0; l=subs.length; i<l; i++){
      subs[i].update();
    }
  }
}

function remove(arr, item){
   if(arr.length){
     const index = arr.indexOf(item);
     if(index > -1){
       return arr.splice(index, 1);
     }
   }
}
```
改造一下defineReactive：

```js
function defineReactive(data, key, val){
   let dep = new Dep();
   Object.defineReactive(data, key, {
     enumerable: true,
     configurable： true,
     get: function(){
       dep.depend(); // 收集依赖
       return val;
     },
     set: function(newVal){
       if(val === newVal) return;
       val = newVal;
       dep.notfiy();
     }
   });
}
```

#### 2.5 依赖是谁
在上面的代码中，我们收集的依赖是window.target，那么它到底是什么？我们究竟要收集谁呢？

收集谁，换句话说，就是当属性发生变化后，通知谁。

我们要通知用到数据的地方，而使用这个数据的地方有很多，而且类型还不一样，既有可能是模板，也有可能是用户写的一个watch，这时需要抽象出一个能集中处理这些情况的类。然后，我们在依赖收集阶段只收集这个封装好的类的实例进来，通知也只通知它一个。接着，它再负责通知其他地方。所以，我们要抽象的这个东西需要先起一个好听的名字。嗯，就叫它Watcher吧。

现在就可以回答上面的问题了，收集谁？Watcher！
#### 2.6 什么是Watcher
Watcher是一个中介的角色，数据发生变化时通知它，然后它再通知其他地方。

```js
// 这段代码表示当data.a.b.c属性发生变化时，触发第二个参数中的函数。
vm.$watch('a.b.c', function(newVal, oldVal){
   
})
```

思考一下，怎么实现这个功能呢？好像只要把这个watcher实例添加到data.a.b.c属性的Dep中就行了。然后，当data.a.b.c的值发生变化时，通知Watcher。接着，Watcher再执行参数中的这个回调函数。

代码如下：

```js
export default class Watcher {
  constructor(vm, expOrFn, cb){
    this.vm = vm;
    this.getter = parsePath(expOrFn); // 解析data.a.b.c
    this.cb = cb;
    this.value = this.get();
  }

  get(){
    window.target = this;
    let value = this.getter.call(this.vm, this.vm);
    window.target = undefined;
    return value;
  }

  update(){
    const oldValue = this.value;
    this.value = this.get();
    this.cb.call(this.vm, this.value, oldValue)
  }
}

function parsePath(path){
  const bailER = /[^\w.$]/;
  if(bailER.test(path)){
    return;
  }
  const segments = path.split('.');
  return function (obj){
    for(let i=0; i<segments.length; i++){
       if(!obj) return;
       obj = obj[segments[i]];
    }
    return obj;
  }
}
```

#### 2.7 递归侦测所有key
现在，其实已经可以实现变化侦测的功能了，但是前面介绍的代码只能侦测数据中的某一个属性，我们希望把数据中的所有属性（包括子属性）都侦测到，所以要封装一个Observer类。这个类的作用是将一个数据内的所有属性（包括子属性）都转换成getter/setter的形式，然后去追踪它们的变化：

```js
/**
* Observer类会附加到每一个被侦测的object上。
* 一旦被附加上，Observer会将object的所有属性转换为getter/setter的形式
* 来收集属性的依赖，并且当属性发生变化时会通知这些依赖
*/

export class Observer {
  constructor(value){
    this.value = value;
    if(!Array.isArray(value)){
      this.wall(value);
    }
  }

  walk(obj){
    const keys = Object.keys(obj);
    for(let i = 0; i < keys.length; i++){
      defineReactive(obj, keys[i], obj[keys[i]]);
    }
  }
}

function defineReactive(data, key, val){
  // 递归子属性
   if(typeof val === 'object'){
     new Observer(val)
   }

   let dep = new Dep();
   Object.defineProperty(data, key, {
      enumerable: true,
      configruable: true,
      get(){
        dep.depend();
        return val;
      }
      set(newVal){
        if(val === newVal) return;
        val = newVal;
        dep.notfiy();
      }
   });
}
```

#### 2.8 关于Object的问题
前面介绍了Object类型数据的变化侦测原理，了解了数据的变化是通过getter/setter来追踪的。也正是由于这种追踪方式，有些语法中即便是数据发生了变化，Vue.js也追踪不到：

1. 新增属性：我们在obj上面新增了name属性，Vue.js无法侦测到这个变化，所以不会向依赖发送通知
2. 删除属性：我们在action方法中删除了obj中的name属性，而Vue.js无法侦测到这个变化，所以不会向依赖发送通知。

Vue.js通过Object.defineProperty来将对象的key转换成getter/setter的形式来追踪变化，但getter/setter只能追踪一个数据是否被修改，无法追踪新增属性和删除属性，所以才会导致上面例子中提到的问题。

但这也是没有办法的事，因为在ES6之前，JavaScript没有提供元编程的能力，无法侦测到一个新属性被添加到了对象中，也无法侦测到一个属性从对象中删除了。为了解决这个问题，Vue.js提供了两个API——vm.$set与vm.$delete，

#### 2.9 总结
变化侦测就是侦测数据的变化。当数据发生变化时，要能侦测到并发出通知。

Object可以通过Object.defineProperty将属性转换成getter/setter的形式来追踪变化。读取数据时会触发getter，修改数据时会触发setter。

我们需要在getter中收集有哪些依赖使用了数据。当setter被触发时，去通知getter中收集的依赖数据发生了变化。

收集依赖需要为依赖找一个存储依赖的地方，为此我们创建了Dep，它用来收集依赖、删除依赖和向依赖发送消息等。

所谓的依赖，其实就是Watcher。只有Watcher触发的getter才会收集依赖，哪个Watcher触发了getter，就把哪个Watcher收集到Dep中。当数据发生变化时，会循环依赖列表，把所有的Watcher都通知一遍。

Watcher的原理是先把自己设置到全局唯一的指定位置（例如window.target），然后读取数据。因为读取了数据，所以会触发这个数据的getter。接着，在getter中就会从全局唯一的那个位置读取当前正在读取数据的Watcher，并把这个Watcher收集到Dep中去。通过这样的方式，Watcher可以主动去订阅任意一个数据的变化。

此外，我们创建了Observer类，它的作用是把一个object中的所有数据（包括子数据）都转换成响应式的，也就是它会侦测object中所有数据（包括子数据）的变化。

**Data、Observer、Dep和Watcher之间的关系:**

<img src="/img/vue2.jpeg" style="max-width:95%" />

Data通过Observer转换成了getter/setter的形式来追踪变化;

当外界通过Watcher读取数据时，会触发getter从而将Watcher添加到依赖中;

当数据发生了变化时，会触发setter，从而向Dep中的依赖（Watcher）发送通知;

Watcher接收到通知后，会向外界发送通知，变化通知到外界后可能会触发视图更新，也有可能触发用户的某个回调函数等。

### 3、 Array的变化侦测
#### 3.1 介绍
这个例子使用了push方法来改变数组，并不会触发getter/setter

```js
this.list.push(1); 
```

正因为我们可以通过Array原型上的方法来改变数组的内容，所以Object那种通过getter/setter的实现方式就行不通了

#### 3.2 如何追踪变化
Object的变化是靠setter来追踪的，只要一个数据发生了变化，一定会触发setter。

在ES6之前，JavaScript并没有提供元编程的能力，也就是没有提供可以拦截原型方法的能力，但是这难不倒聪明的程序员们。我们可以用自定义的方法去覆盖原生的原型方法。  

我们可以用一个拦截器覆盖Array.prototype。之后，每当使用Array原型上的方法操作数组时，其实执行的都是拦截器中提供的方法，比如push方法。然后，在拦截器中使用原生Array的原型方法去操作数组。

#### 3.3 拦截器
拦截器其实就是一个和Array.prototype一样的Object，里面包含的属性一模一样，只不过这个Object中某些可以改变数组自身内容的方法是我们处理过的。

我们发现Array原型中可以改变数组自身内容的方法有7个，分别是push、pop、shift、unshift、splice、sort和reverse。

```js
const arrayProto = Array.prototype;
export const ArrayMethods = Object.create(arrayProto); // 新对象， 继承Array.prototype

['push', 'pop', 'shift', 'unshift', 'splice', 'sort', 'reverse'].forEach(function(method){
    // 缓存原始方法
    const original = arrayProto[method];

    Object.defineProperty(ArrayMethods, method, {
        value: function mutator(...args) {
            return original.apply(this, args);
        },
        enumerable: false,
        writable: true,
        configurable: true
    })
})
```
#### 3.4 使用拦截器覆盖Array原型

## 4、虚拟DOM

## 5、模板编译

## 6、整体流程