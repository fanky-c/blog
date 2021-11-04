---
title: Vue内部运行机制
date: 2019-07-15 21:56:08
tags:
    - vue
    - vue运行机制
---

### new Vue() 内部流程图

<img src="/img/newVue.png" width = "700" height = "auto" alt="离线日志" align=center />

#### 初始化及挂载
在 new Vue() 之后。 Vue 会调用 _init 函数进行初始化，也就是这里的 init 过程，它会初始化生命周期、事件、 props、 methods、 data、 computed 与 watch 等。其中最重要的是通过 Object.defineProperty 设置 setter 与 getter 函数，用来实现「响应式」以及「依赖收集」。
> 初始化之后调用 $mount 会挂载组件，如果是运行时编译，即不存在 render function 但是存在 template 的情况，需要进行「编译」步骤。


#### 编译(如果用了vue-loader则直接生成render function字符串)
compile编译可以分成 parse、optimize 与 generate 三个阶段，最终需要得到 render function。
** 如果是用vue-loader实现预编译就直接生成了render function 字符串；如果是运行时编译就不会生成render function。**
> 1. parse 会用正则等方式解析 template 模板中的指令、class、style等数据，形成AST。
> 2. optimize 的主要作用是标记 static 静态节点，这是 Vue 在编译过程中的一处优化，后面当 update 更新界面时，会有一个 patch 的过程， diff 算法会直接跳过静态节点，从而减少了比较的过程，优化了 patch 的性能
> 3. generate 是将 AST 转化成 render function 字符串的过程，得到结果是 render 的字符串以及 staticRenderFns 字符串。


#### 响应式
1. getter 跟 setter 已经在之前介绍过了，在 init 的时候通过 Object.defineProperty 进行了绑定，它使得当被设置的对象被读取的时候会执行 getter 函数，而在当被赋值的时候会执行 setter 函数。
2. 当 render function 被渲染的时候，因为会读取所需对象的值，所以会触发 getter 函数进行「依赖收集」，「依赖收集」的目的是将观察者 Watcher 对象存放到当前闭包中的订阅者 Dep 的 subs 中。
3. 在修改对象的值的时候，会触发对应的 setter， setter 通知之前「依赖收集」得到的 Dep 中的每一个 Watcher，告诉它们自己的值改变了，需要重新渲染视图。这时候这些 Watcher 就会开始调用 update 来更新视图，当然这中间还有一个 patch 的过程以及使用队列来异步更新的策略。


#### Virtual DOM
render function 会被转化成 VNode 节点。Virtual DOM 其实就是一棵以 JavaScript 对象（ VNode 节点）作为基础的树，用对象属性来描述节点，实际上它只是一层对真实 DOM 的抽象。最终可以通过一系列操作使这棵树映射到真实环境上。由于 Virtual DOM 是以 JavaScript 对象为基础而不依赖真实平台环境，所以使它具有了跨平台的能力，比如说浏览器平台、Weex、Node 等
```js
{
    tag: 'div',                 /*说明这是一个div标签*/
    children: [                 /*存放该标签的子节点*/
        {
            tag: 'a',           /*说明这是一个a标签*/
            text: 'click me'    /*标签的内容*/
        }
    ]
}
```
渲染后成真实DOM
```html
<div>
    <a>click me</a>
</div>
```
实际上的节点有更多的属性来标志节点，比如 isStatic （代表是否为静态节点）、 isComment （代表是否为注释节点）等


#### 更新视图
前面提到通过 setter -> Watcher -> update 的流程来修改对应的视图，那么最终是如何更新视图的呢？

当数据变化后，执行 render function 就可以得到一个新的 VNode 节点，我们如果想要得到新的视图，最简单粗暴的方法就是直接解析这个新的 VNode 节点，然后用 innerHTML 直接全部渲染到真实 DOM 中。但是其实我们只对其中的一小块内容进行了修改，这样做似乎有些「浪费」。

那么我们为什么不能只修改那些「改变了的地方」呢？这个时候就要介绍我们的「patch」了。我们会将新的 VNode 与旧的 VNode 一起传入 patch 进行比较，经过 diff 算法得出它们的「差异」。最后我们只需要将这些「差异」的对应 DOM 进行修改即可

### 响应式系统依赖收集追踪原理

### 实现Virtual DOM下的一个VNode节点


### template模板怎么通过Compile编译
#### 1.parse，利用正则解析template、class、style等数据，生成ast语法树
AST语法树结构如下：
```js
{
    /* 标签属性的map，记录了标签上属性 */
    'attrsMap': {
        ':class': 'c',
        'class': 'demo',
        'v-if': 'isShow'
    },
    /* 解析得到的:class */
    'classBinding': 'c',
    /* 标签属性v-if */
    'if': 'isShow',
    /* v-if的条件 */
    'ifConditions': [
        {
            'exp': 'isShow'
        }
    ],
    /* 标签属性class */
    'staticClass': 'demo',
    /* 标签的tag */
    'tag': 'div',
    /* 子标签数组 */
    'children': [
        {
            'attrsMap': {
                'v-for': "item in sz"
            },
            /* for循环的参数 */
            'alias': "item",
            /* for循环的对象 */
            'for': 'sz',
            /* for循环是否已经被处理的标记位 */
            'forProcessed': true,
            'tag': 'span',
            'children': [
                {
                    /* 表达式，_s是一个转字符串的函数 */
                    'expression': '_s(item)',
                    'text': '{{item}}'
                }
            ]
        }
    ]
}
```
#### 2.optimize，优化静态内容
这个涉及到后面要讲 patch 的过程，因为 patch 的过程实际上是将 VNode 节点进行一层一层的比对，然后将「差异」更新到视图上。那么一些静态节点是不会根据数据变化而产生变化的，这些节点我们没有比对的需求，是不是可以跳过这些静态节点的比对，从而节省一些性能呢？
```js
{
    'attrsMap': {
        ':class': 'c',
        'class': 'demo',
        'v-if': 'isShow'
    },
    'classBinding': 'c',
    'if': 'isShow',
    'ifConditions': [
        'exp': 'isShow'
    ],
    'staticClass': 'demo',
    'tag': 'div',
    /* 静态标志 */
    'static': false,
    'children': [
        {
            'attrsMap': {
                'v-for': "item in sz"
            },
            'static': false,
            'alias': "item",
            'for': 'sz',
            'forProcessed': true,
            'tag': 'span',
            'children': [
                {
                    'expression': '_s(item)',
                    'text': '{{item}}',
                    'static': false
                }
            ]
        }
    ]
}
```
1. isStatic函数；判断的标准是当 type 为 2（表达式节点）则是非静态节点，当 type 为 3（文本节点）的时候则是静态节点，当然，如果存在 if 或者 for这样的条件的时候（表达式节点），也是非静态节点。
```js
function isStatic (node) {
    if (node.type === 2) {
        return false
    }
    if (node.type === 3) {
        return true
    }
    return (!node.if && !node.for);
}
```
2. markStatic函数；遍历所有节点通过 isStatic 来判断当前节点是否是静态节点，此外，会遍历当前节点的所有子节点，如果子节点是非静态节点，那么当前节点也是非静态节点。


#### 3.generate,生成render function字符串
generate 会将 AST 转化成 render funtion 字符串，最终得到 render 的字符串以及 staticRenderFns 字符串。
Vue.js编译得到结果
```js
with(this){
    return (isShow) ? 
    _c(
        'div',
        {
            staticClass: "demo",
            class: c
        },
        _l(
            (sz),
            function(item){
                return _c('span',[_v(_s(item))])
            }
        )
    )
    : _e()
}
```

### 数据状态更新时差异diff以及patch机制
首先说一下 patch 的核心 diff 算法，我们用 diff 算法可以比对出两颗树的「差异」，我们来看一下，假设我们现在有如下两颗树，它们分别是新老 VNode 节点，这时候到了 patch 的过程。

<img src="/img/diff.png" width = "700" height = "auto" alt="" align=center />


diff 算法是通过同层的树节点进行比较而非对树进行逐层搜索遍历的方式，所以时间复杂度只有 O(n)，是一种相当高效的算法，如下图。

<img src="/img/path.png" width = "700" height = "auto" alt="" align=center />

这张图中的相同颜色的方块中的节点会进行比对，比对得到「差异」后将这些「差异」更新到视图上。因为只进行同层级的比对，所以十分高效。

```js
function patch (oldVnode, vnode, parentElm) {
    if (!oldVnode) 
    {
        //老VNode节点不存在，向父元素中添加新VNode节点
        addVnodes(parentElm, null, vnode, 0, vnode.length - 1);
    } 
    else if (!vnode) 
    {
        //没有新的VNode节点，就从父元素中把旧的VNode节点删除掉
        removeVnodes(parentElm, oldVnode, 0, oldVnode.length - 1);
    } 
    else 
    {  
        if (sameVnode(oldVNode, vnode)) {
            //新VNode和老VNode都存在且相同，比对 VNode
            patchVnode(oldVNode, vnode); 
        } 
        else 
        {
            //新VNode和老VNode都存在且不同，删除老节点，增加新节点
            removeVnodes(parentElm, oldVnode, 0, oldVnode.length - 1);
            addVnodes(parentElm, null, vnode, 0, vnode.length - 1);
        }
    }
}
```
#### patch四种情况
##### 1.老VNode节点不存在，就向父元素中添加新VNode节点。
##### 2.新VNode节点不存在，就从父元素中把旧的VNode节点删除掉。
##### 3.新VNode和老VNode都存在且相同，比对 VNode。
1. 怎么判断两个VNode是否为相同节点？
   1. sameVnode 其实很简单，只有当 key、 tag(div、p)、 isComment（是否为注释节点）、 data同时定义（或不定义），同时满足当标签类型为 input 的时候 type 相同（某些浏览器不支持动态修改<input\>类型，所以他们被视为不同类型）即可
      ```js
      function sameVnode () {
        return (
            a.key === b.key &&
            a.tag === b.tag &&
            a.isComment === b.isComment &&
            (!!a.data) === (!!b.data) &&
            sameInputType(a, b)
        )
      }

        function sameInputType (a, b) {
            if (a.tag !== 'input') return true
            let i
            const typeA = (i = a.data) && (i = i.attrs) && i.type
            const typeB = (i = b.data) && (i = i.attrs) && i.type
            return typeA === typeB
        }
      ```
    2. patchVnode过程
       1. test

##### 4.新VNode和老VNode都存在且不同，删除老节点，增加新节点。



### 批量异步更新策略和nextTick原理
#### 为啥什么要异步更新
通过前面几个章节我们介绍，相信大家已经明白了 Vue.js 是如何在我们修改 data 中的数据后修改视图了。简单回顾一下，这里面其实就是一个“setter -> Dep -> Watcher -> patch -> 视图”的过程。

Vue.js在默认情况下，每次触发某个数据的 setter 方法后，对应的 Watcher 对象其实会被 push 进一个队列 queue 中，在下一个 tick 的时候将这个队列 queue 全部拿出来 run（ Watcher 对象的一个方法，用来触发 patch 操作） 一遍

<img src="/img/vueWatcher.png" style="width:300px; margin:0 auto;display:block;" alt="离线日志" align=center />

那么什么是下一个 tick 呢？

#### nextTick
Vue.js 实现了一个 nextTick 函数，传入一个 cb ，这个 cb 会被存储到一个队列中，在下一个 tick 时触发队列中的所有 cb 事件。

因为目前浏览器平台并没有实现 nextTick 方法，所以 Vue.js 源码中分别用 Promise、setTimeout、setImmediate 等方式。目的是在当前调用栈执行完毕以后（不一定立即）才会去执行这个事件。


### Vuex状态管理的原理




[本文来源参考](https://juejin.im/book/5a36661851882538e2259c0f/section/5a37bbb35188257d167a4d64)