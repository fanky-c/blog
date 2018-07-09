---
title: vue源码分析中
date: 2018-07-09 17:20:26
tags:
    - vue
    - 虚拟 DOM
    - Virtual DOM
---

### Virtual DOM 产生背景

Virtual DOM 这个概念相信大部分人都不会陌生，它产生的前提是浏览器中的 DOM 是很“昂贵"的，为了更直观的感受，我们可以简单的把一个简单的 div 元素的属性都打印出来，如图所示：
![DOM元素](/blog/img/dom.png)

而 Virtual DOM 就是用一个原生的 JS 对象去描述一个 DOM 节点，所以它比创建一个 DOM 的代价要小很多。在 Vue.js 中，Virtual DOM 是用 VNode 这么一个 Class 去描述，它是定义在 src/core/vdom/vnode.js 中的*（源码结构目录在上篇文章已经谈到）*。

### 为什么Virtual DOM 更有好？
* 首先参考尤雨溪[网上都说操作真实 DOM 慢，但测试结果却比 React 更快，为什么？](https://www.zhihu.com/question/31809713)
* 如果没有 Virtual DOM，简单来想就是直接重置innerHTML。很多人都没有意识到，在一个大型列表所有数据都变了的情况下，重置 innerHTML 其实是一个还算合理的操作... 真正的问题是在 “全部重新渲染” 的思维模式下，即使只有一行数据变了，它也需要重置整个 innerHTML，这时候显然就有大量的浪费。
* 对innerHTML和Virtual进行对比:
	1，innerHTML: render html string O(template size) + 重新创建所有 DOM 元素 O(DOM size)
	2，Virtual  : render Virtual DOM + diff O(template size) + 必要的 DOM 更新 O(DOM change)

显然Virtual DOM render + diff 显然比渲染 html 字符串要慢，但是！它依然是纯 js 层面的计算，比起后面的 DOM 操作来说，依然便宜了太多。可以看到，innerHTML 的总计算量不管是 js 计算还是 DOM 操作都是和整个界面的大小相关，但 Virtual DOM 的计算量里面，只有 js 计算和界面大小相关，DOM 操作是和数据的变动量相关的。前面说了，和 DOM 操作比起来，js 计算是极其便宜的。这才是为什么要有 Virtual DOM：它保证了: 
- 1）不管你的数据变化多少，每次重绘的性能都可以接受；
- 2) 你依然可以用类似 innerHTML 的思路去写你的应用…	


### Virtual DOM到底是什么东西？
* 首先和dom没啥关系，类似java和javascript关系
* 它是一个数据结构，具体表现为有序的二叉树
* 在javascript中，它表现为Object对象
* 在UI映射方面，virtual Dom的对象节点跟DOM Tree每个位置属性一一对应
* Virtual DOM是DOM Tree某一时刻的快照

### Virtual DOM功能拆分


### Virtual DOM 在vue体现
其实 VNode 是对真实 DOM 
的一种抽象描述，它的核心定义无非就几个关键属性，标签名、数据、子节点、键值等，其它属性都是都是用来扩展 VNode 的灵活性以及实现一些特殊 feature 的。由于 VNode 只是用来映射到真实 DOM 的渲染，不需要包含操作 DOM 的方法，因此它是非常轻量和简单的。

Virtual DOM 除了它的数据结构的定义，映射到真实的 DOM 实际上要经历 VNode 的 create、diff、patch 等过程。那么在 Vue.js 中，VNode 的 create 是通过之前提到的 createElement 方法创建的，我们接下来分析这部分的实现。参考(https://ustbhuangyi.github.io/vue-analysis/data-driven/virtual-dom.html#%E6%80%BB%E7%BB%93)




