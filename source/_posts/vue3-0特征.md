---
title: vue3.0特征
date: 2018-12-07 11:31:24
tags:
  - vue3.0
  - vue 
---
### 新特征
1. 编译时候会对原生元素(div)和组件元素(component)进行分析对比，运行时候原生元素直接生成vitrual dom，组件元素就直接生成组件代码。
2. 优化slots生成，生成新的组件依赖关系，避免不必要的组件渲染。之前版本如果slots模板自身发生变化，会通知调用的父组件更新。现在利用scoped(范围、作用域)只更新子组件本身了。
3. 静态属性的提取。之前v2.x只对静态内容做直接输出，但是静态属性可以和静态内容一样不做比对直接输出。
```
<div id="test" class="test">{{ msg }}</div>
```
#### 数据监听由object.defineProperty变为Proxy。
  1. Object.defineProperty() 方法会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性，并返回这个对象。
  ```
    obj：必需。目标对象 
    prop：必需。需定义或修改的属性的名字
    descriptor：必需。目标属性所拥有的特性
  ```
  2. Proxy 对象用于定义基本操作的自定义行为（如属性查找，赋值，枚举，函数调用等）:[参考](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy)
     1. Proxy是lazy by default，初始化整个对象数据，当用某个属性用到才会被监听。


#### runTime资源会更小（源码支持tree-shaking）  
   1. 源码支持tree-shaking结构, 之前的所有的功能都挂在vue对象上，vue3.x把不是必须的功能编译打包的时候按需引入。最最最基本核心代码大概10kg gziped压缩。
   ```
   //Vue2.0
   import Vue from 'vue'
   Vue.nextTick(() => {})
   const obj =  Vue.observable({})
   
   //Vue3.0
   import { nextTick, observable } from 'vue'
   nextTick(() => {})
   const obj = observable({})
   ```

#### 时间分片
1. cpu运行web程序机制是：主队列（主线程）需要完成所有主要任务（脚本的加载、渲染等），才能响应用户的操作。 这样用户的体验取决于Vue组件加载或者重新渲染的时间。
2. Vue3的时间分片做法：将脚本运算过程切换成小段，并在每段执行完之后查看是否有用户输入处理。这样无论多少次渲染和重新渲染，应用程序都保持这响应状态。

#### flow变为type-script
#### 编译器重构
   1. 插件化设计，便于开发者参与其中。
   2. 带位置的信息parser(source maps), 具体某个模板具体某一块出错信息。
   3. 更好的IDE工具链（eslint、vscode）