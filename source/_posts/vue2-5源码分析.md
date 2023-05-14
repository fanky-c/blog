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

我们创建了变量arrayMethods，它继承自Array.prototype，具备其所有功能。未来，我们要使用arrayMethods去覆盖Array.prototype

接下来，在arrayMethods上使用Object.defineProperty方法将那些可以改变数组自身内容的方法（push、pop、shift、unshift、splice、sort和reverse）进行封装。

所以，当使用push方法的时候，其实调用的是arrayMethods.push，而arrayMethods.push是函数mutator，也就是说，实际上执行的是mutator函数。（我们就可以在mutator函数中做一些其他的事，比如说发送变化通知）

最后，在mutator中执行original（它是原生Array.prototype上的方法，例如Array.prototype.push）来做它应该做的事，比如push的功能。


#### 3.4 使用拦截器覆盖Array原型
有了拦截器之后，想要让它生效，就需要使用它去覆盖Array.prototype。但是我们又不能直接覆盖，因为这样会污染全局的Array，这并不是我们希望看到的结果。我们希望拦截操作只针对那些被侦测了变化的数据生效，也就是说希望拦截器只覆盖那些响应式数组的原型。

而将一个数据转换成响应式的，需要通过Observer，所以我们只需要在Observer中使用拦截器覆盖那些即将被转换成响应式Array类型数据的原型就好了：

```js
export class Observer {
  constructor(value){
    this.value = value;

    if(Array.isArray(value)){
       value.__proto__ = ArrayMethods;
    }else{
      this.walk(value);
    }
  }
}
```
通过 \__proto__ 可以很巧妙地实现覆盖value原型的功能

<img src="/img/vue3.png" style="max-width:95%" />

#### 3.5 将拦截器方法挂载到数组的属性上
虽然绝大多数浏览器都支持这种非标准的属性（在ES6之前并不是标准）来访问原型，但并不是所有浏览器都支持！因此，我们需要处理不能使用 \__proto__ 的情况。 

Vue的做法非常粗暴，如果不能使用 \__proto__，就直接将arrayMethods身上的这些方法设置到被侦测的数组上

```js
import { arrayMethods } from './array';

const hasProto = '__proto__' in {};
const arrayKeys = Object.getOwnPropertyNames(arrayMethods);

function protoAugment(target, src, keys){
    target.__proto__ = src;
}

function copyAugment(target, src, keys){
  for(let i=0; i<keys.length; i++){
      const key = keys[i];
      def(target, key, src[key]);
  }
}

export class Observer {
  constructor(value){
     this.value = value;
     if(Array.isArray(value)){
        const augment = hasProto ? protoAugment : copyAugment; // 增强
        augment(value, arrayMethods, arrayKeys);
     }else{
         this.walk(value);
     }
  }
}
```

在浏览器不支持 __proto__ 的情况下，会在数组上挂载一些方法。当用户使用这些方法时，其实执行的并不是浏览器原生提供的Array.prototype上的方法，而是拦截器中提供的方法。 

因为当访问一个对象的方法时，只有其自身不存在这个方法，才会去它的原型上找这个方法。

#### 3.6 如何收集依赖
可能你也发现了，如果只有一个拦截器，其实还是什么事都做不了。为什么会这样呢？因为我们之所以创建拦截器，本质上是为了得到一种能力，一种当数组的内容发生变化时得到通知的能力。

而现在我们虽然具备了这样的能力，但是通知谁呢？前面我们介绍Object时说过，答案肯定是通知Dep中的依赖（Watcher），但是依赖怎么收集呢？这就是本节要介绍的内容，如何收集数组的依赖！


**Array在getter中收集依赖，在拦截器中触发依赖。**

#### 3.7 依赖列表存在哪儿
Vue.js把Array的依赖存放在Observer中：

```js
export class Observer {
  constructor(value){
     this.value = value;
     this.dep = new Dep();

     if(Array.isArray(value)){
        const augment = hasProto ? protoAugment : copyAugment;
        augment(value, arrayMethods, arrayKeys);
     }else{
        this.walk(value);
     }
  }
}
```

#### 3.8 收集依赖
把Dep实例保存在Observer的属性上之后，我们可以在getter中像下面这样访问并收集依赖：

```js
function defineReactive(data, key, val){
  let childOb = observe(val); // 修改
  let dep = new Dep(); 
  Object.defineProperty(data, key, {
    enumerable: true,
    configruable: true,
    get: function(){
      dep.depend();
      if(childOb){
        childOb.dep.depend();
      }
      return val;
    },
    set: function(newVal){
      if(val === newVal) return;
      dep.notfiy();
      val = newVal;
    }
  })
}

/**
 * 尝试为value创建一个Observer实例，
 * 如果创建成功，直接返回新创建Observer实例，
 * 如果value已经存在一个Observer实例，则直接返回它
 */
export function observe(value, asRootData){
  if(!isObject(value)) return;
  let ob;
  if(hasOwn(value, '__ob__') && value.__ob__ instanceof Observer){
    ob = value.__ob__;
  }else{
    ob = new Observer(value);
  }
  return ob;
}
```

#### 3.9 在拦截器中获取Observer实例
因为Array拦截器是对原型的一种封装，所以可以在拦截器中访问到this（当前正在被操作的数组）。

而dep保存在Observer中，所以需要在this上读到Observer的实例：

```js
// 工具函数
function def (data, key, val, enumerable){
  Object.defineProperty(obj, key, {
     value: val,
     enumerable: !!enumerable,
     writable: true,
     configurable: true
  })
}

export class Observer {
  constructor(value){
    this.value = value;
    this.dep = new Dep();
    def(value, '__ob__', this); //新增

    if(Array.isArray(value)){
       const augment = hasProto ？ protoAugment : copyAugment;
       augment(value, arrayMethods, arrayKeys);
    }else{
       this.walk(value);
    }
  }
}
```

所有被侦测了变化的数据身上都会有一个 \__ob__ 属性来表示它们是响应式的。上一节中的observe函数就是通过 \__ob__ 属性来判断：如果value是响应，如果value是响应式的，则直接返回 \__ob__；如果不是响应式的，则使用new Observer来将数据转换成响应式数据。

当value身上被标记了 \__ob__ 之后，就可以通过value.\__ob__ 来访问Observer实例。如果是Array拦截器，因为拦截器是原型方法，所以可以直接通过this.\__ob__来访问Observer实例。例如：

```js
['push', 'pop', 'shift', 'unshift', 'splice', 'sort', 'reverse'].
forEach(function(method){
  const original = arrayProto[method];
  Object.defineProperty(arrayMethods, method, {
    value: function muator (...args){
      const ob = this.__ob__; // 新增
      return original.apply(this, args);
    },
    enumerable: false,
    writable: true,
    configurable: true 
  })
})
```

#### 3.10 向数组的依赖发送通知
当侦测到数组发生变化时，会向依赖发送通知。此时，首先要能访问到依赖。前面已经介绍过如何在拦截器中访问Observer实例，所以这里只需要在Observer实例中拿到dep属性，然后直接发送通知就可以了：

```js
['push', 'pop', 'shift', 'unshift', 'splice', 'sort', 'reverse'].
forEach(function(method){
  // 缓存原始方法
  const original = arrayProto[method];
  def(arrayMethods, method, function mutator(...args){
      const result = original.apply(this, args);
      const ob = this.__ob__;
      ob.dep.notfiy(); // 通知依赖
      return result; 
  });
})
```
在上面的代码中，我们调用了ob.dep.notify()去通知依赖（Watcher）数据发生了改变。

#### 3.11 侦测数组中元素的变化
我们要在Observer中新增一些处理，让它可以将Array也转换成响应式的:

```js
export class Observer {
  constructor(value){
    this.value = value;
    def(value, '__ob__', this);

    if(Array.isArray(valye)){
      this.observeArray(value);
    }else{
      this.walk(value);
    }
  }
  
  // 侦测Array中每一项
  ObserveArray(items){
     for(let i=0, l=items.length; i<l; i++){
       observe(item[i]);
     }
  }
}
```
这里新增了observeArray方法，其作用是循环Array中的每一项，执行observe函数来侦测变化。前面介绍过observe函数，其实就是将数组中的每个元素都执行一遍new Observer，这很明显是一个递归的过程。

现在只要将一个数据丢进去，Observer就会把这个数据的所有子数据转换成响应式的。接下来，我们介绍如何侦测数组中新增元素的变化。

#### 3.12 侦测新增元素的变化
数组中有一些方法是可以新增数组内容的，比如push，而新增的内容也需要转换成响应式来侦测变化，否则会出现修改数据时无法触发消息等问题。因此，我们必须侦测数组中新增元素的变化。

**其实现方式其实并不难，只要能获取新增的元素并使用Observer来侦测它们就行。**

##### 3.12.1 获取新增元素
想要获取新增元素，我们需要在拦截器中对数组方法的类型进行判断。如果操作数组的方法是push、unshift和splice（可以新增数组元素的方法），则把参数中新增的元素拿过来，用Observer来侦测：

```js
['pop', 'push', 'shift', 'splice', 'sort', 'reverse'].
forEach(function(method){
  const original = arrayProto[method];
  def(arrayMethod, method, function muator(...args){
     const result = original.apply(this, args);
     const ob = this.__ob__;
     let inserted;
     switch(method){
         case 'push':
         case 'unshift':
          inserted = true;
          break;
         case 'splice':
          inserted = args.slice(2);
          break;  
     }
     ob.dep.notify();
     return result;
  });
})
```
我们通过switch对method进行判断，如果method是push、unshift、splice这种可以新增数组元素的方法，那么从args中将新增元素取出来，暂存在inserted中。

接下来，我们要使用Observer把inserted中的元素转换成响应式的。

##### 3.12.2 使用Observer侦测新增元素
前面介绍过Observer会将自身的实例附加到value的 \__ob__ 属性上。所有被侦测了变化的数据都有一个 \__ob__ 属性，数组元素也不例外。

因此，我们可以在拦截器中通过this访问到 \__ob__，然后调用 \__ob__ 上的observeArray方法就可以了：

```js
['push', 'pop', 'shift', 'unshift', 'splice', 'sort', 'reverse'].
forEach(function(method){
   const original = arrayProto[method];
   def(arrayMethods, method, function mutator(...args){
     const result = original.apply(this, args);
     const ob = this.__ob__;
     let inserted;
     switch(method){
        case 'push':
        case 'unshift':
          inserted = true;
          break;
         case 'splice':
           inserted = args.slice(2);
           break; 
     }
     if(inserted) ob.observerArray(inserted); 
     ob.dep.notfiy();
     return result;
   })
})
```

#### 3.15 关于Array的问题

```js
this.list[0] = 1; //  即修改数组中第一个元素的值时，无法侦测到数组的变化
this.list.lengt = 0; // 这个清空数组操作也无法侦测到数组的变化，所以也不会触发re-render或watch等。
```

因为Vue.js的实现方式决定了无法对上面举的两个例子做拦截，也就没有办法响应。在ES6之前，无法做到模拟数组的原生行为，所以拦截不到也是没有办法的事情。ES6提供了元编程的能力，所以有能力拦截，我猜测未来Vue.js很有可能会使用ES6提供的Proxy来实现这部分功能，从而解决这个问题。

#### 3.13 总结
Array追踪变化的方式和Object不一样。因为它是通过方法来改变内容的，所以我们通过创建拦截器去覆盖数组原型的方式来追踪变化。

为了不污染全局Array.prototype，我们在Observer中只针对那些需要侦测变化的数组使用 \__proto__ 来覆盖原型方法，但 \__proto__ 在ES6之前并不是标准属性，不是所有浏览器都支持它。因此，针对不支持 \__proto__ 属性的浏览器，我们直接循环拦截器，把拦截器中的方法直接设置到数组身上来拦截Array.prototype上的原生方法。

Array收集依赖的方式和Object一样，都是在getter中收集。但是由于使用依赖的位置不同，数组要在拦截器中向依赖发消息，所以依赖不能像Object那样保存在defineReactive中，而是把依赖保存在了Observer实例上。

在Observer中，我们对每个侦测了变化的数据都标上印记 \__ob__，并把this（Observer实例）保存在 \__ob__ 上。这主要有两个作用，一方面是为了标记数据是否被侦测了变化（保证同一个数据只被侦测一次），另一方面可以很方便地通过数据取到 \__ob__，从而拿到Observer实例上保存的依赖。当拦截到数组发生变化时，向依赖发送通知。

除了侦测数组自身的变化外，数组中元素发生的变化也要侦测。我们在Observer中判断如果当前被侦测的数据是数组，则调用observeArray方法将数组中的每一个元素都转换成响应式的并侦测变化。

除了侦测已有数据外，当用户使用push等方法向数组中新增数据时，新增的数据也要进行变化侦测。我们使用当前操作数组的方法来进行判断，如果是push、unshift和splice方法，则从参数中将新增数据提取出来，然后使用observeArray对新增数据进行变化侦测。

由于在ES6之前，JavaScript并没有提供元编程的能力，所以对于数组类型的数据，一些语法无法追踪到变化，只能拦截原型上的方法，而无法拦截数组特有的语法，例如使用length清空数组的操作就无法拦截。

### 4、变化侦测相关的API实现原理
#### 4.1 vm.$watch
##### 4.1.1 用法
用于观察一个表达式或computed函数在Vue.js实例上的变化。回调函数调用时，会从参数得到新数据（new value）和旧数据（old value）。表达式只接受以点分隔的路径，例如a.b.c。如果是一个比较复杂的表达式，可以用函数代替表达式。

```js
let unwatch = vm.$watch('a.b.c', function (newVal, oldVal) {
    // 做点什么
});

unwatch(); // vm.$watch返回一个取消观察函数，用来停止触发回调：
```

**deep。为了发现对象内部值的变化，可以在选项参数中指定deep: true：**

```js
vm.$watch('someObj', callback, {
  deep: true
});
vm.someObj.a = 12; 
// 回调函数讲被触发
// 这里需要注意的是，监听数组的变动不需要这么做。
```
**immediate。在选项参数中指定immediate: true，将立即以表达式的当前值触发回调：**

```js
vm.$watch('a', callback, {
  immediate: true
});
// 立即以 a 的当前值触发回调
```
##### 4.1.2 原理
vm.$watch其实是对Watcher的一种封装，但vm.$watch中的参数deep和immediate是Watcher中所没有的。下面我们来看一看vm.$watch到底是怎么实现的：

```js
Vue.prototype.$watch = function(expOrFn, cb, options){
  const vm = this;
  options = options || {};
  const watcher = new Watcher(vm, expOrFn, cb, options);

  if(options.immediate){
      cb.call(vm, watcher.value);
  }

  return function unwatchFn(){
     watcher.teardown();
  }
}
```

先执行new Watcher来实现vm.$watch的基本功能。这里有一个细节需要注意，expOrFn是支持函数的，而我们在第2章中并没有介绍。这里我们需要对Watcher进行一个简单的修改：

```js
export default class Watcher {
   constructor(vm, expOrFn, cb){
     this.vm = vm;
     if(typeof expOrFn === 'function'){
       this.getter = expOrFn;
     }else{
       this.getter = parsePath(expOrFn);
     }
     this.cb = cb;
     this.value = this.get();
   }
}
```
当expOrFn是函数时，会发生很神奇的事情。它不只可以动态返回数据，其中读取的所有数据也都会被Watcher观察。当expOrFn是字符串类型的keypath时，Watcher会读取这个keypath所指向的数据并观察这个数据的变化。而当expOrFn是函数时，Watcher会同时观察expOrFn函数中读取的所有Vue.js实例上的响应式数据。**也就是说，如果函数从Vue.js实例上读取了两个数据，那么Watcher会同时观察这两个数据的变化，当其中任意一个发生变化时，Watcher都会得到通知。(Vue.js中计算属性 Computed 的实现原理与expOrFn支持函数有很大的关系)**

现在要在Watcher中添加该方法来实现unwatch的功能：

首先，需要在Watcher中记录自己都订阅了谁，也就是watcher实例被收集进了哪些Dep里。然后当Watcher不想继续订阅这些Dep时，循环自己记录的订阅列表来通知它们（Dep）将自己从它们（Dep）的依赖列表中移除掉。

```js
export default class Wather {
  constructor(vm, expOrFn, cb){
     this.vm = vm;
     this.deps = [];
     this.depIds = new Set();
     if(typeof expOrFn === 'function'){
       this.getter = expOrFn;
     }else{
       this.getter = parsePath(expOrFn);
     }
     this.cb = cb;
     this.value = this.get();
  }

  addDep(dep){
    const id = dep.id;
    if(!this.depIds.has(id)){
       this.depIds.add(id);
       this.deps.push(id);
       dep.addSub(this);
    }
  }

  teardown(){
    let i = this.deps.length;
    while(i--){
      this.deps[i].removeSub(this);
    }
  }
}
```

执行this.depIds.add来记录当前Watcher已经订阅了这个Dep。

然后执行this.deps.push(dep)记录自己都订阅了哪些Dep。

最后，触发dep.addSub(this)来将自己订阅到Dep中。

在Watcher中新增addDep方法后，Dep中收集依赖的逻辑也需要有所改变：

```js
let uid = 0;

export default class Dep {
   constructor(){
      this.id = uid++;
      this.subs = [];
   }

   depend(){
     if(window.target){
       this.addSub(window.target);  // 废弃
       window.target.addDep(this); // 新增
     }
   }

   removeSub(sub){
    const index = this.subs.indexOf(sub);
    if(index > -1){
       return this.subs.splice(index, 1)
    }
   }

}
```

##### 4.1.3 deep参数的实现原理
要想实现deep的功能，其实就是除了要触发当前这个被监听数据的收集依赖的逻辑之外，还要把当前监听的这个值在内的所有子值都触发一遍收集依赖逻辑。这就可以实现当前这个依赖的所有子数据发生变化时，通知当前Watcher了。

```js
export default class Wather {
   constructor(vm, expOrFn, cb, options){
     this.vm = vm;

     // 新增
     if(options){
        this.deep = !!options.deep;
     }else{
        this.deep = false;
     }

     this.deps = [];
     this.depIds = new Set();
     if(typeof expOrFn === 'function'){
       this.getter = expOrFn;
     }else{
       this.getter = parsePath(expOrFn);
     }
     this.cb = cb;
     this.value = this.get();
   }

   get(){
    window.target = this;
    let value = this.getter.call(vm, vm);
    if(this.deep){
       traverse(value);
    }
    window.target = undefined;
    return value
   }
}
```

一定要在window.target = undefined之前去触发子值的收集依赖逻辑，这样才能保证子集收集的依赖是当前这个Watcher。如果在window.target = undefined之后去触发收集依赖的逻辑，那么其实当前的Watcher并不会被收集到子值的依赖列表中，也就无法实现deep的功能。

接下来，要递归value的所有子值来触发它们收集依赖的功能：

```js
const seenObjects = new Set();

export function traverse (val){
  _traverse(val, seenObjects);
  seenObjects.clear();
}

function _traverse(val, seen){
   let i, keys;
   const isA = Array.isArray(val);
   if((!isA && !isObject(val)) || Object.isFrozen(val)){
       return;
   }
   if(val.__ob__){
    const depId = val.__ob__.dep.id;
    if(seen.has(depId)){
      return;
    }
    seen.add(depId);
   }

   if(isA){
      i = val.length;
      while(i--) __traverse(val[i], seen);
   }else{
      keys = Object.keys(val);
      i = keys.length;
      while(i--) __traverse(val[keys[i]], seen); 
   }
}
```

#### 4.2 vm.$set
##### 4.2.1 用法
```js
vm.$set(target, key, value);
// {Object | Array } target
// {String | Number} key
// {any} value

// 返回值 {Function} unwatch
```

用法：在object上设置一个属性，如果object是响应式的，Vue.js会保证属性被创建后也是响应式的，并且触发视图更新。这个方法主要用来避开Vue.js不能侦测属性被添加的限制。

原因：只有已经存在的属性的变化会被追踪到，新增的属性无法被追踪到。因为在ES6之前，JavaScript并没有提供元编程的能力，所以根本无法侦测object什么时候被添加了一个新属性。

```js
/**
 * 新增的这个属性也不是响应式的，Vue.js根本不知道这个obj新增了属性，
 * 就好像Vue.js无法知道我们使用array.length = 0清空了数组一样。
 */ 
var vm = new Vue({
    el: '#el',
    template: '#demo-template',
    data: {
      obj: { }
    },    
    methods: {
      action () {
        this.obj.name = 'berwin' // 
      }
    }
})
```

##### 4.2.2 原理

```js
import { set } from '../observer/index';
Vue.prototype.$set = set;

// index文件
export function set (target, key, val){ } 
```

**1、Array处理：**

```js
export function set (target, key, val){
  if(Array.isArray(target) && isValidArrayIndex(key)){
      target.length = Math.max(target.length, key);
      target.splice(key, 1, val);
      return val;
  }
}
```

如果target是数组并且key是一个有效的索引值，就先设置length属性。这样如果我们传递的索引值大于当前数组的length，就需要让target的length等于索引值。

通过splice方法把val设置到target中的指定位置（参数中提供的索引值的位置）。当我们使用splice方法把val设置到target中的时候，数组拦截器会侦测到target发生了变化，并且会自动帮助我们把这个新增的val转换成响应式的。

**2、key已经存在target中**
```js
export function set(target, key, val){
  if(Array.isArray(target) && isValidArrayIndex(key)){
      target.length = Math.max(target.length, key);
      target.splice(key, 1, val);
      return val;
  }

  if(key in target && !(key in Object.prototype)){
      target[key] = val;
      return val;
  }
}
```

由于key已经存在于target中，所以其实这个key已经被侦测了变化。也就是说，这种情况属于修改数据，直接用key和val改数据就好了。

**3、处理新增属性**
```js
export function set(target, key, val){
  if(Array.isArray(target) && isValidArrayIndex(key)){
      target.length = Math.max(target.length, key);
      target.splice(key, 1, val);
      return val;
  }

  if(key in target && !(key in Object.prototype)){
      target[key] = val;
      return val;
  }

  // 新增
  const ob = target.__ob__;
  if(target.isVue && (ob && ob.vmCount)){
      process.env.NODE_ENV !== 'production' && warn(
        'Avoid adding reactive properties to a Vue instance or its root $data ' +
        'at runtime - declare it upfront in the data option.'
      );
      return val
  }
  if(!ob){
    target[key] = val;
    return val;
  }
  defineReactive(ob.value, key, val);
  ob.dep.notify();
  return val;
}
```

要处理文档中所说的“target不能是Vue.js实例或Vue.js实例的根数据对象”的情况。

实现这个功能并不难，只需要使用target._isVue来判断target是不是Vue.js实例，使用ob.vmCount来判断它是不是根数据对象即可。

那么，什么是根数据？this.$data就是根数据。

接下来，我们处理target不是响应式的情况。如果target身上没有 \__ob__ 属性，说明它并不是响应式的，并不需要做什么特殊处理，只需要通过key和val在target上设置就行了


#### 4.3 vm.$delete
##### 4.3.1 用法
```js
vm.$delete(target, key);

// {Object | Array} target
// {String | Numbet} key
```

删除对象的属性。如果对象是响应式的，需要确保删除能触发更新视图。这个方法主要用于避开Vue.js不能检测到属性被删除的限制，但是你应该很少会使用它。

强烈不推荐这样写代码：

```js
delete this.obj.name;
this.obj.__ob__.dep.notify(); // 手动向依赖发生变化通知
```

##### 4.3.2 原理

```js
import { del } from '../observer/index';
Vue.prototype.$delete = del;

// index
export function del (target, key){
  const ob = target.__ob__;
  delete target[key];
  ob.dep.notify();
}
```

**1、处理Array**

```js
export function del (target, key){
  if(Array.isArray(target) && isValidArrayIndex(key)){
    target.splice(key, 1);
    return;
  }
  const ob = (target).__ob__;
  delete target[key];
  ob.dep.notify();  
}
```

只需要使用splice将参数key所指定的索引位置的元素删除即可。因为使用了splice方法，数组拦截器会自动向依赖发送通知。

vm.$delete也不可以在Vue.js实例或Vue.js实例的根数据对象上使用:
```js
export function del (target, key){
   if(Array.isArray(target) && isValidArrayIndex(key)){
      target.splice(key, 1);
      return;
   }
   const ob = target.__ob__;

   // 新增
   if(target._isVue || (ob && ob.vmCount)){
      process.env.NODE_ENV !== 'production' && warn(
        'Avoid deleting properties on a Vue instance or its root $data ' +
        '- just set it to null.'        
      );
      return;
   }

   // 如果key不是target自身属性， 则终止
   if(!hasOwn(target, key))){
    return;
   }

   delete target[key];
   
   // 如果ob不存在，则终止
   if(!ob){
     return;
   }

   ob.dep.notify();
} 
```

#### 4.4 总结
待补充吧


## 4、虚拟DOM
Vue.js 2.0引入了虚拟DOM，比Vue.js 1.0的初始渲染速度提升了2~4倍，并大大降低了内存消耗。

### 1、虚拟DOM简介
#### 1.1 虚拟DOM背景
虚拟DOM是随着时代发展而诞生的产物。

在Web早期，页面的交互效果比现在简单得多，没有很复杂的状态需要管理，也不太需要频繁地操作DOM，使用jQuery来开发就可以满足我们的需求。

随着时代的发展，页面上的功能越来越多，我们需要实现的需求也越来越复杂，程序中需要维护的状态也越来越多，DOM操作也越来越频繁。

**命令式操作DOM：当状态变得越来越多，DOM操作越来越频繁时，我们就会发现如果像之前那样使用jQuery来开发页面，那么代码中会有相当多的代码是在操作DOM，程序中的状态也很难管理，代码中的逻辑也很混乱。**

**声明式操作DOM: 我们通过描述状态和DOM之间的映射关系是怎样的，就可以将状态渲染成视图。关于状态到视图的转换过程，框架会帮我们做，不需要我们自己手动去操作DOM。**

然而通常程序在运行时，状态会不断发生变化（引起状态变化的原因有很多，有可能是用户点击了某个按钮，也可能是某个Ajax请求，这些行为都是异步发生的。理论上，所有异步行为都有可能引起状态变化）。**每当状态发生变化时，都需要重新渲染。如何确定状态中发生了什么变化以及需要在哪里更新DOM？**

在这种情况下，最简单粗暴的解决方式是，既不需要关心状态发生了什么变化，也不需要关心在哪里更新DOM，我们只需要把所有DOM全删了，然后使用状态重新生成一份DOM，并将其输出到页面上显示出来就好了。

但是访问DOM是非常昂贵的。按照上面说的方式做，会造成相当多的性能浪费。状态变化通常只有有限的几个节点需要重新渲染，所以我们不仅需要找出哪里需要更新，还需要尽可能少地访问DOM。

业界3大框架解决方案：

1. Angular中就是脏检查的流程
2. React中使用虚拟DOM
3. Vue.js 1.0通过细粒度的绑定


**虚拟DOM的解决方式是通过状态生成一个虚拟节点树，然后使用虚拟节点树进行渲染。在渲染之前，会使用新生成的虚拟节点树和上一次生成的虚拟节点树进行对比，只渲染不同的部分。**


#### 1.2 为什么要引入虚拟DOM 
事实上，Angular和React的变化侦测有一个共同点，那就是它们都不知道哪些状态（state）变了。因此，就需要进行比较暴力的比对，React是通过虚拟DOM的比对，Angular是使用脏检查的流程。

Vue.js的变化侦测和它们都不一样，它在一定程度上知道具体哪些状态发生了变化，这样就可以通过更细粒度的绑定来更新视图。也就是说，在Vue.js中，当状态发生变化时，它在一定程度上知道哪些节点使用了这个状态，从而对这些节点进行更新操作，根本不需要比对。事实上，在Vue.js 1.0的时候就是这样实现的。

**但是这样做其实也有一定的代价。因为粒度太细，每一个绑定都会有一个对应的watcher来观察状态的变化，这样就会有一些内存开销以及一些依赖追踪的开销。当状态被越多的节点使用时，开销就越大。对于一个大型项目来说，这个开销是非常大的。**

**因此，Vue.js 2.0开始选择了一个中等粒度的解决方案，那就是引入了虚拟DOM。组件级别是一个watcher实例，就是说即便一个组件内有10个节点使用了某个状态，但其实也只有一个watcher在观察这个状态的变化。所以当这个状态发生变化时，只能通知到组件，然后组件内部通过虚拟DOM去进行比对与渲染。这是一个比较折中的方案。**

**Vue.js之所以能随意调整绑定的粒度，本质上还要归功于变化侦测。**

#### 1.3 vue.js中的虚拟dom
在Vue.js中，我们使用模板来描述状态与DOM之间的映射关系。**Vue.js通过编译将模板转换成渲染函数（render），执行渲染函数就可以得到一个虚拟节点树，使用这个虚拟节点树就可以渲染页面：**

<img src="/img/vue4.jpeg" style="max-width:95%" />

由于DOM操作比较慢，所以这些DOM操作在性能上会有一定的浪费，避免这些不必要的DOM操作会提升很大一部分性能。

为了避免不必要的DOM操作，虚拟DOM在虚拟节点映射到视图的过程中，将虚拟节点与上一次渲染视图所使用的旧虚拟节点（oldVnode）做对比，找出真正需要更新的节点来进行DOM操作，从而避免操作其他无任何改动的DOM。

虚拟DOM的执行流程：

<img src="/img/vue5.jpeg" style="max-width:95%" />

虚拟DOM在Vue.js中所做的事情其实并没有想象中那么复杂，它主要做了两件事：

1. 提供与真实DOM节点所对应的虚拟节点vnode。
2. 将虚拟节点vnode和旧虚拟节点oldVnode进行比对，然后更新视图

#### 1.4 总结
虚拟DOM是将状态映射成视图的众多解决方案中的一种，它的运作原理是使用状态生成虚拟节点，然后使用虚拟节点渲染视图。

之所以需要先使用状态生成虚拟节点，是因为如果直接用状态生成真实DOM，会有一定程度的性能浪费。而先创建虚拟节点再渲染视图，就可以将虚拟节点缓存，然后使用新创建的虚拟节点和上一次渲染时缓存的虚拟节点进行对比，然后根据对比结果只更新需要更新的真实DOM节点，从而避免不必要的DOM操作，节省一定的性能开销。

Vue.js中通过模板来描述状态与视图之间的映射关系，所以它会先将模板编译成渲染函数，然后执行渲染函数生成虚拟节点，最后使用虚拟节点更新视图。

因此，虚拟DOM在Vue.js中所做的事是提供虚拟节点vnode和对新旧两个vnode进行比对，并根据比对结果进行DOM操作来更新视图。


### 2、VNode
#### 2.1 VNode简介
在Vue.js中存在一个VNode类，使用它可以实例化不同类型的vnode实例，而不同类型的vnode实例各自表示不同类型的DOM元素。

例如，DOM元素有元素节点、文本节点和注释节点等，vnode实例也会对应着有元素节点、文本节点和注释节点等。

```js
export default class VNode {
  constructor(tag, data, children, text, elm, context, componentOptions, asyncFactory){
      this.tag = tag;
      this.data = data;
      this.children = children;
      this.text = text;
      this.elm = elm;
      this.ns = undefined;
      this.context = context;
      this.functionalContext = undefined;
      this.functionalOptions = undefined;
      this.functionalScopeId = undefined;
      this.key = data && data.key;
      this.componentOptions = componentOptions;
      this.componentInstance = undefined;
      this.parent = undefined;
      this.raw = false;
      this.isStatic = false;
      this.isRootInsert = true;
      this.isComment = false;
      this.isCloned = false;
      this.isOnce = false;
      this.asyncFactory = asyncFactory;
      this.asyncMeta = undefined;
      this.isAsyncPlaceholder = false;
  }

  get child(){
    return this.commponentInstance;
  }
}
```

简单地说，vnode可以理解成节点描述对象，它描述了应该怎样去创建真实的DOM节点。

vnode表示一个真实的DOM元素，所有真实的DOM节点都使用vnode创建并插入到页面中:

**Vnode --create--> DOM  --insert--> 视图**

#### 2.2 VNode作用
由于每次渲染视图时都是先创建vnode，然后使用它创建真实DOM插入到页面中，所以可以将上一次渲染视图时所创建的vnode缓存起来，之后每当需要重新渲染视图时，将新创建的vnode和上一次缓存的vnode进行对比，查看它们之间有哪些不一样的地方，找出这些不一样的地方并基于此去修改真实的DOM。

**Vue.js目前对状态的侦测策略采用了中等粒度。当状态发生变化时，只通知到组件级别，然后组件内使用虚拟DOM来渲染视图。**

也就是说，只要组件使用的众多状态中有一个发生了变化，那么整个组件就要重新渲染。

如果组件只有一个节点发生了变化，那么重新渲染整个组件的所有节点，很明显会造成很大的性能浪费。因此，对vnode进行缓存，并将上一次缓存的vnode和当前新创建的vnode进行对比，只更新发生变化的节点就变得尤为重要。这也是vnode最重要的一个作用。

#### 2.3 VNode类型
vnode的类型有以下几种：
1. 注释节点
2. 文本节点
3. 元素节点
4. 组件节点
5. 函数式组件
6. 克隆节点

##### 2.3.1 注释节点

```html
<!-- 注释节点 -->
```

```js
{
  text: '注释节点',
  isComment: true
}
```

##### 2.3.2 文本节点
当文本类型的vnode被创建时，它只有一个text属性：

```js
export function createTextVNode(val){
  return new VNode(undefined, undefined, undefined, String(val));
}

{
  text: 'hello world'
}
```

##### 2.3.3 克隆节点
克隆节点是将现有节点的属性复制到新节点中，让新创建的节点和被克隆节点的属性保持一致，从而实现克隆效果。它的作用是优化静态节点和插槽节点（slot node）。

以静态节点为例，当组件内的某个状态发生变化后，当前组件会通过虚拟DOM重新渲染视图，静态节点因为它的内容不会改变，所以除了首次渲染需要执行渲染函数获取vnode之外，后续更新不需要执行渲染函数重新生成vnode。因此，这时就会使用创建克隆节点的方法将vnode克隆一份，使用克隆节点进行渲染。这样就不需要重新执行渲染函数生成新的静态节点的vnode，从而提升一定程度的性能

```js
export function cloneVNode(vnode, depp){
  const cloned = new VNode(
    vnode.tag,
    vnode.data,
    vnode.children,
    vnode.text,
    vnode.elm,
    vnode.context,
    vnode.componentOptions,
    vnode.asyncFactory
  )
  cloned.ns = vnode.ns;
  cloned.isStatic = vnode.isStatic;
  cloned.key = vnode.key;
  clone.isCloned = true; // 这是克隆节点与被克隆节点的区别
  if(deep && vnode.children){
    cloned.children = cloneVNode(vnode.children);
  }
  return cloned;
}
```

##### 2.3.4 元素节点
元素节点通常会存在以下4种有效属性。
1. tag：顾名思义，tag就是一个节点的名称，例如p、ul、li和div等。
2. data：该属性包含了一些节点上的数据，比如attrs、class和style等。
3. children：当前节点的子节点列表。
4. context：它是当前组件的Vue.js实例。

```html
<p><span>Hello</span><span>Berwin</span></p>
```

```js
{
  children: [VNode, VNode],
  context: {...},
  data: {...},
  tag: 'p',
  ...
}
```

##### 2.3.5 组件节点
组件节点和元素节点类似，有以下两个独有的属性。

1. componentOptions：顾名思义，就是组件节点的选项参数，其中包含propsData、tag和children等信息
2. componentInstance：组件的实例，也是Vue.js的实例。事实上，在Vue.js中，每个组件都是一个Vue.js实例

```js
<child></child>

// 对应vnode
{
   componentOptions: {...},
   commponentInstance: {...},
   context: {...},
   data: {...},
   tag: 'vue-component-child',
   ...
}
```

##### 2.3.6 函数式组件
函数式组件和组件节点类似，它有两个独有的属性functionalContext和functionalOptions


### 3、patch
虚拟DOM最核心的部分是patch，它可以将vnode渲染成真实的DOM。

之所以要这么做，主要是因为DOM操作的执行速度远不如JavaScript的运算速度快。因此，把大量的DOM操作搬运到JavaScript中，使用patching算法来计算出真正需要更新的节点，最大限度地减少DOM操作，从而显著提升性能。这本质上其实是使用JavaScript的运算成本来替换DOM操作的执行成本，而JavaScript的运算速度要比DOM快很多，这样做很划算，所以才会有虚拟DOM。

#### 3.1 patch介绍
对现有DOM进行修改需要做三件事：

1. 创建新增的节点；
2. 删除已经废弃的节点；
3. 修改需要更新的节点。


之所以需要通过算法来比对两个节点之间的差异，并针对不同的节点进行更新，主要是为了性能考虑。

我们完全可以把整个旧节点从DOM中删除，然后使用最新的状态（state）重新生成一份全新的节点并插入到DOM中，这种方式完全可以实现功能。

##### 3.1.1 新增节点
首先，新增节点的一个很明显的场景就是，当oldVnode不存在而vnode存在时，就需要使用vnode生成真实的DOM元素并将其插入到视图当中去。

1. **1、这通常会发生在首次渲染中。**因为首次渲染时，DOM中不存在任何节点，所以oldVnode是不存在的。
<img src="/img/vue6.jpeg" style="max-width:95%" />
<img src="/img/vue7.jpeg" style="max-width:95%" />

2. **2、还有一种情况也需要新增节点。**当vnode和oldVnode完全不是同一个节点时，需要使用vnode生成真实的DOM元素并将其插入到视图当中。
<img src="/img/vue8.jpeg" style="max-width:95%" />


##### 3.1.2 删除节点
就是当一个节点只在oldVnode中存在时，我们需要把它从DOM中删除。因为渲染视图时，需要以vnode为标准，所以vnode中不存在的节点都属于被废弃的节点，而被废弃的节点需要从DOM中删除。

##### 3.1.3 更新节点
新旧两个节点是同一个节点。当新旧两个节点是相同的节点时，我们需要对这两个节点进行比较细致的比对，然后对oldVnode在视图中所对应的真实节点进行更新。

举个简单的例子，当新旧两个节点是同一个文本节点，但是两个节点的文本不一样时，我们需要重新设置oldVnode在视图中所对应的真实DOM节点的文本。

<img src="/img/vue9.jpeg" style="max-width:95%" />

##### 3.1.4 总结
<img src="/img/vue10.jpeg" style="max-width:95%" />

#### 3.2 创建节点
只有三种类型的节点会被创建并插入到DOM中：元素节点、注释节点和文本节点。

创建一个节点并将其渲染到视图的全过程：

<img src="/img/vue11.jpeg" style="max-width:95%" />

#### 3.2 删除节点

```js
// removeVnodes 删除一组指定的节点
function removeVnodes(vnodes, startIdx, endIdx){
  for(; startIdx <= endIdx; ++startIdx){
    const ch = vnodes[startIdx];
    if(isDef(ch)){
        // removeNode 删除单个节点
        removeNode(ch.elm);
    }
  }
}

const nodeOps = {
  removeChild(node, child){
     node.removeChild(child);
  }
}

function removeNode(el){
   const parent = nodeOps.parentNode(el);
   if(!isDef(parent)){
    nodeOps.removeChild(parent, el);
   }
}
```
有同学可能会对nodeOps感到奇怪，为什么不直接使用parent.removeChild(child)删除节点，而是将这个节点操作封装成函数放在nodeOps里呢？

其实这涉及跨平台渲染的知识，我们知道阿里开发的Weex可以让我们使用相同的组件模型为iOS和Android编写原生渲染的应用。也就是说，我们写的Vue.js组件可以分别在iOS和Android环境中进行原生渲染。

**跨平台渲染的本质是在设计框架的时候，要让框架的渲染机制和DOM解耦。**只要把框架更新DOM时的节点操作进行封装，就可以实现跨平台渲染，在不同平台下调用节点的操作。

如果我们把这些平台下节点操作的封装看成渲染引擎，那么将这些渲染引擎所提供的节点操作的API和框架的运行时对接一下，就可以实现将框架中的代码进行原生渲染的目的。


#### 3.3 更新节点
只有两个节点是同一个节点时，才需要更新元素节点，而更新节点并不是很暴力地使用新节点覆盖旧节点，而是通过比对找出新旧两个节点不一样的地方，针对那些不一样的地方进行更新。

##### 3.3.1 静态节点
在更新节点时，首先需要判断新旧两个虚拟节点是否是静态节点，如果是，就不需要进行更新操作，可以直接跳过更新节点的过程。

**静态节点指的是那些一旦渲染到界面上之后，无论日后状态如何变化，都不会发生任何变化的节点。**

```html
<span>我是静态节点，我不需要改变</span>
```

##### 3.3.2 新虚拟节点有文本属性
当新旧两个虚拟节点（vnode和oldVnode）不是静态节点，并且有不同的属性时，要以新虚拟节点（vnode）为准来更新视图。根据新节点（vnode）是否有text属性，更新节点可以分为两种不同的情况。

简单来说，就是当新虚拟节点有文本属性，并且和旧虚拟节点的文本属性不一样时，我们可以直接把视图中的真实DOM节点的内容改成新虚拟节点的文本。

##### 3.3.3 新虚拟节点无文本属性
如果新创建的虚拟节点没有text属性，那么它就是一个元素节点。元素节点通常会有子节点，也就是children属性，但也有可能没有子节点，所以存在两种不同的情况。

1. 一种情况，有children情况
   a、如果旧虚拟节点也有children属性，那么我们要对新旧两个虚拟节点的children进行一个更详细的对比并更新。 <br>

   b、如果旧虚拟节点没有children属性，那么说明旧虚拟节点要么是一个空标签，要么是有文本的文本节点。

2. 二种情况，无children情况
  当新创建的虚拟节点既没有text属性也没有children属性时，这说明这个新创建的节点是一个空节点，它下面既没有文本也没有子节点，这时如果旧虚拟节点（oldVnode）中有子节点就删除子节点，有文本就删除文本。有什么删什么，最后达到视图中是空标签的目的。


##### 3.3.4 总结
<img src="/img/vue12.jpeg" style="max-width:95%" />

<img src="/img/vue13.jpeg" style="max-width:95%" />

#### 3.4 更新子节点策略
之前我们详细讨论了更新节点的过程，其中讨论了当新节点的子节点和旧节点的子节点都存在并且不相同时，会进行子节点的更新操作。但我们并没有详细讨论子节点是如何更新的。

更新子节点大概可以分为4种操作：更新节点、新增节点、删除节点、移动节点位置。

##### 3.4.1 创建子节点
最上面的DOM节点是视图中的真实DOM节点。左下角的节点是新创建的虚拟节点。 右下角的节点是旧的虚拟节点。

<img src="/img/vue14.jpeg" style="max-width:95%" />

上图已经对前两个子节点进行了更新，当前正在处理第三个子节点。当在右下角的虚拟子节点中找不到与左下角的第三个节点相同的节点时，证明它是新增节点，这时候需要创建节点并插入到真实DOM中，插入的位置是所有未处理节点的前面，也就是虚线所指定的位置。

下图插入到未处理节点的前面：
<img src="/img/vue15.jpeg" style="max-width:95%" />

下图插入到已处理节点的后面：
<img src="/img/vue16.jpeg" style="max-width:95%" />

可能你现在又有疑问了，节点插入进真实DOM中后，真实DOM中的节点越来越多，为什么没看见删除节点的逻辑？ 关于删除节点的逻辑，我们将在后面详细介绍

##### 3.4.2 更新子节点
两个节点是同一个节点并且位置相同，这种情况下只需要进行更新节点的操作即可

<img src="/img/vue17.jpeg" style="max-width:95%" />

##### 3.4.3 移动子节点
移动节点通常发生在newChildren中的某个节点和oldChildren中的某个节点是同一个节点，但是位置不同，所以在真实的DOM中需要将这个节点的位置以新虚拟节点的位置为基准进行移动。

<img src="/img/vue18.jpeg" style="max-width:95%" />

上图表示正在处理第三个节点，这时在oldChildren中找到的相同节点是第四个节点。由于位置不同，所以需要移动节点，移动节点的位置是所有未处理节点的最前面。本例中，将第四个节点移动到所有未处理节点的最前面，就是将节点从第四个变成了第三个。

节点更新并且移动完位置后，开始进行下一轮循环，也就是开始处理newChildren中的第四个节点。

关于怎么分辨哪些节点是处理过的，哪些节点是未处理的，我们将在后面章节详细讨论

##### 3.4.4 删除子节点
删除子节点，本质上是删除那些oldChildren中存在但newChildren中不存在的节点

当newChildren中的所有节点都被循环了一遍后，也就是循环结束后，如果oldChildren中还有剩余的没有被处理的节点，那么这些节点就是被废弃、需要删除的节点。

##### 3.4.5 优化策略
通常情况下，并不是所有子节点的位置都会发生移动，一个列表中总有几个节点的位置是不变的。针对这些位置不变的或者说位置可以预测的节点，我们不需要循环来查找，因为我们有一个更快捷的查找方式

只需要尝试使用相同位置的两个节点来比对是否是同一个节点：如果恰巧是同一个节点，直接就可以进入更新节点的操作；如果尝试失败了，再用循环的方式来查找节点。

这样做可以很大程度地避免循环oldChildren来查找节点，从而使执行速度得到很大的提升。

如果我们把这种很快速的查找节点的方式称为快捷查找，那么它共有4种查找方式，分别是：

1. 新前（newChildren中所有未处理的第一个节点）与 旧前（oldChildren中所有未处理的第一个节点）
2. 新后（newChildren中所有未处理的最后一个节点）与 旧后（oldChildren中所有未处理的最后一个节点）
3. 新后 与 旧前
4. 新前 与 旧后


###### 1、新前和旧前
顾名思义，“新前”与“旧前”的意思就是尝试使用“新前”这个节点与“旧前”这个节点对比，对比它们俩是不是同一个节点。如果是同一个节点，则说明我们不费吹灰之力就在oldChildren中找到了这个虚拟节点，然后使用7.4节中介绍的更新节点操作将它们俩进行对比并更新视图

<img src="/img/vue20.jpeg" style="max-width:95%" />

由于“新前”与“旧前”的位置相同，所以并不需要执行移动节点的操作，只需要更新节点即可。


###### 2、新后和旧后
当“新前”与“旧前”对比后发现不是同一个节点，这时可以尝试用“新后”与“旧后”的方式来比对它们俩是否是同一个节点。

<img src="/img/vue21.jpeg" style="max-width:95%" />

由于“新后”与“旧后”这两个节点的位置相同，所以只需要执行更新节点的操作即可，不需要执行移动节点的操作。

###### 3、新后和旧前
“新后”与“旧前”的意思是使用“新后”这个节点与“旧前”这个节点进行对比，通过对比来分辨它们俩是不是同一个节点。如果是同一个节点，就对比它们俩并更新视图：

<img src="/img/vue22.jpeg" style="max-width:95%" />

如果“新后”与“旧前”是同一个节点，那么由于它们的位置不同，所以除了更新节点外，还需要执行移动节点的操作：

<img src="/img/vue23.jpeg" style="max-width:95%" />

当“新后”与“旧前”是同一个节点时，在真实DOM中除了做更新操作外，还需要将节点移动到oldChildren中所有未处理节点的最后面。

你可能对为什么移动到oldChildren中所有未处理节点的最后面感到困惑，接下来我们会详细介绍为什么移动到这个位置。

更新节点是以新虚拟节点为基准，子节点也不例外，所以在图7-21中，因为“新后”这个节点是最后一个节点，所以真实DOM中将节点移动到最后不难理解，让我们感到困惑的是为什么移动到oldChildren中所有未处理节点的最后面。

这里我们举个例子，如下图：

<img src="/img/vue24.jpeg" style="max-width:95%" />

当真实DOM子节点左右两侧已经有节点被更新，只有中间这部分节点未处理时，“新后”这个节点是未处理节点中的最后一个节点，所以真实DOM节点移动位置时，需要移动到oldChildren中所有未处理节点的最后面。只有移动到未处理节点的最后面，它的位置才与“新后”这个节点的位置相同。

如果对比之后发现这两个节点也不是同一个节点，则继续尝试对比“新前”与“旧后”是否是同一个节点。

###### 4、新前和旧后
“新前”与“旧后”的意思是使用“新前”与“旧后”这两个节点进行对比，对比它们是否是同一个节点，如果是同一个节点，则进行更新节点的操作。

<img src="/img/vue25.jpeg" style="max-width:95%" />

由于“新前”与“旧后”这两个节点的位置不同，所以除了更新节点的操作外，还需要进行移动节点的操作：

<img src="/img/vue26.jpeg" style="max-width:95%" />

从上图中可以看出，当“新前”与“旧后”是同一个节点时，在真实DOM中除了做更新操作外，还需要将节点移动到oldChildren中所有未处理节点的最前面。

将节点移动到oldChildren中所有未处理节点的最前面的原因，与前面介绍的“新后”与“旧前”的逻辑是一样的， 如下图：

<img src="/img/vue27.jpeg" style="max-width:95%" />

如上图所示，当真实的DOM节点中已经有节点被更新，并且更新到第二个节点时，我们发现oldChildren中对应的节点在第三个的位置上，这时需要将“旧后”这个节点更新并移动到第二个的位置上，所以只需要将节点移动到所有未处理节点的最前面，就能实现移动到第二个位置的目的。

也就是说，已更新过的节点都不用管。因为更新过的节点无论是节点的内容或者节点的位置，都是正确的，更新完后面就不需要再进行更改了。所以，我们只需要在所有未更新的节点区间内进行移动和更新操作即可。

如果前面这4种方式对比之后都没找到相同的节点，这时再通过循环的方式去oldChildren中详细找一圈，看看能否找到。

大部分情况下，通过前面这4种方式就可以找到相同的节点，所以节省了很多次循环操作。


###### 5、哪些节点是未处理过的
那么，怎样实现从两边向中间循环呢？

首先，我们先准备4个变量：oldStartIdx、oldEndIdx、newStartIdx和newEndIdx。

这4个变量分别表示oldChildren的开始位置的下标（oldStartIdx）和结束位置的下标（oldEndIdx），以及newChildren的开始位置的下标（newStartIdx）和结束位置的下标（newEndIdx）。

在循环体内，每处理一个节点，就将下标向指定的方向移动一个位置，通常情况下是对新旧两个节点进行更新操作，就相当于一次性处理两个节点，将新旧两个节点的下标都向指定方向移动一个位置。

当开始位置大于等于结束位置时，说明所有节点都遍历过了，则结束循环：

```js
while(oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx){
  // todo
}
```


###### 6、小结
更新子节点的整体流程：

<img src="/img/vue19.jpeg" style="width:120%" />


名称解释：

1. oldStartVnode：oldChildren中所有未处理的第一个节点，与前文中提到的“旧前”是同一个节点。
2. oldEndVnode：oldChildren中所有未处理的最后一个节点，与前文中提到的“旧后”是同一个节点。
3. newStartVnode：newChildren中所有未处理的第一个节点，与前文中提到的“新前”是同一个节点。
4. newEndVnode：newChildren中所有未处理的最后一个节点，与前文中提到的“新后”是同一个节点。

在Vue.js的模板中，渲染列表时可以为节点设置一个属性key，这个属性可以标示一个节点的唯一ID。Vue.js官方非常推荐在渲染列表时使用这个属性，我也非常推荐使用它，为什么呢？

在更新子节点时，需要在oldChildren中循环去找一个节点。但是如果我们在模板中渲染列表时，为子节点设置了属性key，那么在上图中建立key与index索引的对应关系时，就生成了一个key对应着一个节点下标这样一个对象。也就是说，**如果在节点上设置了属性key，那么在oldChildren中找相同节点时，可以直接通过key拿到下标，从而获取节点。这样，我们根本不需要通过循环来查找节点。**



##### 3.4.6 总结
本章中，我们介绍了虚拟DOM中最关键的部分：patch。

通过patch可以对比新旧两个虚拟DOM，从而只针对发生了变化的节点进行更新视图的操作。本章详细介绍了如何对比新旧两个节点以及更新视图的过程。

## 5、模板编译
### 1、介绍
我们平时使用Vue.js进行开发时，会经常使用模板。模板赋予我们很多强大的能力，例如可以在模板中访问变量。

但在Vue.js中创建HTML并不是只有模板这一种途径，我们既可以手动写渲染函数来创建HTML，也可以在Vue.js中使用JSX来创建HTML。

**渲染函数是创建HTML最原始的方法。模板最终会通过编译转换成渲染函数，渲染函数执行后，会得到一份vnode用于虚拟DOM渲染。所以模板编译其实是配合虚拟DOM进行渲染，**这也是本书先介绍虚拟DOM后介绍模板编译的原因。

### 2、模板编译原理
模板编译在整个渲染过程中的位置：

<img src="/img/vue28.jpeg" style="width:90%" />

Vue.js提供了模板语法，允许我们声明式地描述状态和DOM之间的绑定关系，然后通过模板来生成真实DOM并将其呈现在用户界面上。

在底层实现上，Vue.js会将模板编译成虚拟DOM渲染函数。当应用内部的状态发生变化时，Vue.js可以结合响应式系统，聪明地找出最小数量的组件进行重新渲染以及最少量地进行DOM操作。

#### 2.1 概念

模板编译的主要目标就是生成渲染函数，如下图所示。**而渲染函数的作用是每次执行它，它就会使用当前最新的状态生成一份新的vnode，然后使用这个vnode进行渲染。**

<img src="/img/vue29.jpeg" style="width:90%" />

#### 2.2 模板编译 --> 渲染函数
将模板编译成渲染函数可以分两个步骤，先将模板解析成AST（Abstract Syntax Tree，抽象语法树），然后再使用AST生成渲染函数。

但是由于静态节点不需要总是重新渲染，所以在生成AST之后、生成渲染函数之前这个阶段，需要做一个操作，那就是遍历一遍AST，给所有静态节点做一个标记，这样在虚拟DOM中更新节点时，如果发现节点有这个标记，就不会重新渲染它。

所以，在大体逻辑上，模板编译分三部分内容：

1. 将模板解析为AST    --> 解析器
2. 遍历AST标记静态节点 --> 优化器
3. 使用AST生成渲染函数 --> 代码生成器

<img src="/img/vue30.jpeg" style="width:90%" />

#### 2.3 解析器
解析器的作用前面已经提到过，其目标很明确，只实现一个功能，那就是将模板解析成AST。

在解析器内部，分成了很多小解析器，其中包括过滤器解析器、文本解析器和HTML解析器。然后通过一条主线将这些解析器组装在一起。

AST其实和vnode有点类似，都是使用JavaScript中的对象来表示节点。

#### 2.4 优化器
优化器的目标是遍历AST，检测出所有静态子树（永远都不会发生变化的DOM节点）并给其打标记。

当AST中的静态子树被打上标记后，每次重新渲染时，就不需要为打上标记的静态节点创建新的虚拟节点，而是直接克隆已存在的虚拟节点。在虚拟DOM的更新操作中，如果发现两个节点是同一个节点，正常情况下会对这两个节点进行更新，但是如果这两个节点是静态节点，则可以直接跳过更新节点的流程。


#### 2.5 代码生成器

1、模板

```html
<p title="Brewin" @click="c">1</p>
```

2、生成后的代码字符串

```js
`with(this){return _c('p',{attrs:{"title":"Berwin"},on:{"click":c}},[_v("1")])}`

// 格式化后
with(this){
  return _c(
    'p',
    {
      attrs:{"title":"Berwin"},
      on:{"click":c}
    },
    [_v("1")]
  )
}
```
这样一个代码字符串最终导出到外界使用时，会将代码字符串放到函数里，这个函数叫作渲染函数。


```js
const code = `with(this){return 'Hello Berwin'}`
const hello = new Function(code)
  
hello()
// "Hello Berwin"
```
**渲染函数的作用是创建vnode。渲染函数之所以可以生成vnode，是因为代码字符串中会有很多函数调用（例如，上面生成的代码字符串中有两个函数调用 _c和 _v），这些函数是虚拟DOM提供的创建vnode的方法。vnode有很多种类型，不同的类型对应不同的创建方法，所以代码字符串中的 _c和 _v其实都是创建vnode的方法，只是创建的vnode的类型不同。例如，_c可以创建元素类型的vnode，而 _v可以创建文本类型的vnode。**

### 3、解析器
#### 3.1 解析器作用


解析器要实现的功能是将模板解析成AST。

其实AST并不是什么很神奇的东西，不要被它的名字吓倒。它只是用JavaScript中的对象来描述一个节点，一个对象表示一个节点，对象中的属性用来保存节点所需的各种数据。

```html
<div>
  <p>{{name}}</p>
</div>
```

AST

```js
{
  tag: "div"
  type: 1,
  staticRoot: false,
  static: false,
  plain: true,
  parent: undefined,
  attrsList: [],
  attrsMap: {},
  children: [
    {
      tag: "p"
      type: 1,
      staticRoot: false,
      static: false,
      plain: true,
      parent: {tag: "div", ...},
      attrsList: [],
      attrsMap: {},
      children: [{
        type: 2,
        text: "{{name}}",
        static: false,
        expression: "_s(name)"
      }]
    }
  ]
}
```

#### 3.2 解析器内部运行原理
解析器内部也分了好几个子解析器，比如HTML解析器、文本解析器以及过滤器解析器，其中最主要的是HTML解析器。顾名思义，HTML解析器的作用是解析HTML，它在解析HTML的过程中会不断触发各种钩子函数。这些钩子函数包括开始标签钩子函数、结束标签钩子函数、文本钩子函数以及注释钩子函数。

伪代码：
```js
parseHTML(template, {
  start(tag, attrs, unary){
    // 每当解析到标签的开始位置时，就会触发这个钩子函数
  },
  end(){
    //  每当解析到标签的结束位置时，就会触发这个钩子函数
  },
  chars(text){
    //  每当解析到文本时，就会触发这个钩子函数
  },
  comment(text){
    //  每当解析到注释时，就会触发这个钩子函数
  }
})
```

```html
<div><p>我是berwin</p></div>
```
当上面这个模板被HTML解析器解析时，所触发的钩子函数依次是：start、start、chars、end和end。

也就是说，解析器其实是从前向后解析的。解析到 <div\> 时，会触发一个标签开始的钩子函数start；然后解析到 <p\> 时，又触发一次钩子函数start；接着解析到我是Berwin这行文本，此时触发了文本钩子函数chars；然后解析到 </p\>，触发了标签结束的钩子函数end；接着继续解析到 </div\>，此时又触发一次标签结束的钩子函数end，解析结束。

因此，我们可以在钩子函数中构建AST节点。在start钩子函数中构建元素类型的节点，在chars钩子函数中构建文本类型的节点，在comment钩子函数中构建注释类型的节点。

```js
function createASTElement (tag, attrs, parent ){
    return {
      type: 1,
      tag,
      attrsList: attrs,
      parent,
      children: []
    }
}

parseHTML(template, {
  start(tag, attrs, unary){
     let element = createASTElement(tag, attrs, currentParent);
  },
  chars(text){
    let element = {
      type: 3,
      text
    }
  },
  comment(text){
    let element = {
      type: 3,
      text,
      isComment: true
    }
  }
})
```


基于HTML解析器的逻辑，我们可以在每次触发钩子函数start时，把当前构建的节点推入栈中；每当触发钩子函数end时，就从栈中弹出一个节点。

这样就可以保证每当触发钩子函数start时，栈的最后一个节点就是当前正在构建的节点的父节点，如下图：

<img src="/img/vue31.jpeg" style="width:90%" />

假如有个模板：
```html
<div>
  <h1>我是Berwin</h1>
  <p>我今年23岁</p>
</div>
```

上面模板解析成AST的过程如下：

<img src="/img/vue32.jpeg" style="width:90%" />
<img src="/img/vue33.jpeg" style="width:90%" />

❶ 模板的开始位置是div的开始标签，于是会触发钩子函数start。start触发后，会先构建一个div节点。此时发现栈是空的，这说明div节点是根节点，因为它没有父节点。最后，将div节点推入栈中，并将模板字符串中的div开始标签从模板中截取掉。

❷ 这时模板的开始位置是一些空格，这些空格会触发文本节点的钩子函数，在钩子函数里会忽略这些空格。同时会在模板中将这些空格截取掉。

❸ 这时模板的开始位置是h1的开始标签，于是会触发钩子函数start。与前面流程一样，start触发后，会先构建一个h1节点。此时发现栈的最后一个节点是div节点，这说明h1节点的父节点是div，于是将h1添加到div的子节点中，并且将h1节点推入栈中，同时从模板中将h1的开始标签截取掉。

❹ 这时模板的开始位置是一段文本，于是会触发钩子函数chars。chars触发后，会先构建一个文本节点，此时发现栈中的最后一个节点是h1，这说明文本节点的父节点是h1，于是将文本节点添加到h1节点的子节点中。由于文本节点没有子节点，所以文本节点不会被推入栈中。最后，将文本从模板中截取掉。

❺ 这时模板的开始位置是h1结束标签，于是会触发钩子函数end。end触发后，会把栈中最后一个节点弹出来。

❻ 与第 ❷ 步一样，这时模板的开始位置是一些空格，这些空格会触发文本节点的钩子函数，在钩子函数里会忽略这些空格。同时会在模板中将这些空格截取掉。

❼ 这时模板的开始位置是p开始标签，于是会触发钩子函数start。start触发后，会先构建一个p节点。由于第 ❺ 步已经从栈中弹出了一个节点，所以此时栈中的最后一个节点是div，这说明p节点的父节点是div。于是将p推入div的子节点中，最后将p推入到栈中，并将p的开始标签从模板中截取掉。

❽ 这时模板的开始位置又是一段文本，于是会触发钩子函数chars。当chars触发后，会先构建一个文本节点，此时发现栈中的最后一个节点是p节点，这说明文本节点的父节点是p节点。于是将文本节点推入p节点的子节点中，并将文本从模板中截取掉。

❾ 这时模板的开始位置是p的结束标签，于是会触发钩子函数end。当end触发后，会从栈中弹出一个节点出来，也就是把p标签从栈中弹出来，并将p的结束标签从模板中截取掉。

❿ 与第 ❷ 步和第 ❻ 步一样，这时模板的开始位置是一些空格，这些空格会触发文本节点的钩子函数并且在钩子函数里会忽略这些空格。同时会在模板中将这些空格截取掉。

⓫ 这时模板的开始位置是div的结束标签，于是会触发钩子函数end。其逻辑与之前一样，把栈中的最后一个节点弹出来，也就是把div弹了出来，并将div的结束标签从模板中截取掉。

⓬ 这时模板已经被截取空了，也就说明HTML解析器已经运行完毕。这时我们会发现栈已经空了，但是我们得到了一个完整的带层级关系的AST语法树。这个AST中清晰写明了每个节点的父节点、子节点及其节点类型。



#### 3.3 HTML解析器
我们发现构建AST非常依赖HTML解析器所执行的钩子函数以及钩子函数中所提供的参数，你一定会非常好奇HTML解析器是如何解析模板的。

##### 3.3.1 运行原理
事实上，解析HTML模板的过程就是循环的过程，简单来说就是用HTML模板字符串来循环，每轮循环都从HTML模板中截取一小段字符串，然后重复以上过程，直到HTML模板被截成一个空字符串时结束循环，解析完毕（如上图所示）。

```js
function parseHTML(html, options) {
  while (html) {
    // 截取模版字符串并触发钩子函数
    // todo...
  }
}
```

HTML模板：

```js
`<div>
  <p>{{name}}</p>
</div>`
```

第一轮循环时，截取一段字符串<div\>, 切触发钩子函数start, 截取后的结果为：

```js
`
  <p>{{name}}</p>
</div>`
```

第二轮循环时，截取出一段字符串

```js
`
  `
```

并且触发钩子函数chars，截取后的结果为：

```js
`<p>{{name}}</p>
</div>`
```

第三轮循环时，截取出一段字符串 <p\>，并且触发钩子函数start，截取后的结果为：

```js
`{{name}}</p>
</div>`
```

第四轮循环时，截取出一段字符串 {{name}}，并且触发钩子函数chars，截取后的结果为：

```js
`</p>
</div>`
```

第五轮循环时，截取出一段字符串 </p\>，并且触发钩子函数end，截取后的结果为：

```js
`
</div>`
```

第六轮循环时，截取出一段字符串：

```js
`
`
```

并且触发钩子函数chars，截取后的结果为：

```js
`</div>`
```

第七轮循环时，截取出一段字符串 </div\>，并且触发钩子函数end，截取后的结果为：

```js
``
```

这些被截取的片段分很多种类型，示例如下:

● 开始标签，例如 <div\>。

● 结束标签，例如 </div\>。

● HTML注释，例如 <!-- 我是注释 --\>。

● DOCTYPE，例如 <!DOCTYPE html\>。

● 条件注释，例如 <!--[if !IE]>--\>我是注释<!--<![endif]--\>。

● 文本，例如我是Berwin。



##### 3.3.2 截取开始标签
如何确定模板是不是以开始标签开头？

在HTML解析器中，想分辨出模板是否以开始标签开头并不难，我们需要先判断HTML模板是不是以 < 开头。

如何使用正则表达式来匹配模板以开始标签开头？我们看下面的代码：

```js
const ncname = '[a-zA-Z_][\\w\\-\\.]*';
const qnameCapture = `((?:${ncname}\\:)?${ncname})`;
const startTagOpen = new RegExp(`^<${qnameCapture}`);

// 以开始标签开始的模板
'<div></div>'.match(startTagOpen); // ["div", "div", index: 0, input: "<div></div>"]

// 以结束标签开始的模板
'</div><div>我是Berwin</div>'.match(startTagOpen); // null

// 以文本开始的模板
'我是Berwin</p>'.match(startTagOpen); // null
```

##### 3.3.3 截取结束标签


##### 3.3.4 截取注释


##### 3.3.5 截取条件注释


##### 3.3.6 截取DOCTYPE

##### 3.3.7 截取文本


##### 3.3.8 纯文本内容元素的处理

##### 3.3.9 使用栈维护DOM层级


### 4、优化器
解析器的作用是将HTML模板解析成AST，而优化器的作用是在AST中找出静态子树并打上标记。


### 5、代码生成器
代码生成器是模板编译的最后一步，它的作用是将AST转换成渲染函数中的内容，这个内容可以称为代码字符串。

代码字符串可以被包装在函数中执行，这个函数就是我们通常所说的渲染函数。

## 6、整体流程

