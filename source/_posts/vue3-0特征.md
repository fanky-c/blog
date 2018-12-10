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
4. 数据监听由object.defineProperty变为Proxy。
  1. Object.defineProperty() 方法会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性，并返回这个对象。
  ```
    obj：必需。目标对象 
    prop：必需。需定义或修改的属性的名字
    descriptor：必需。目标属性所拥有的特性
  ```
  2. Proxy 对象用于定义基本操作的自定义行为（如属性查找，赋值，枚举，函数调用等）:[参考](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy)
  3. Proxy是lazy by default，初始化很多数据，当用到才会被监听。
5. runTime会更小  
   1. tree-skaking结构, 之前的所有的功能都挂在vue对象上，vue3.x把不是必须的功能编译打包的时候按需引入。最最最基本核心代码大概10kg gziped压缩。
6. flow变为type-script
7. 编译器重构
   1. 插件化设计，便于开发者参与其中。
   2. 带位置的信息parser(source maps), 具体某个模板具体某一块出错信息。
   3. 更好的IDE工具链（eslint、vscode）