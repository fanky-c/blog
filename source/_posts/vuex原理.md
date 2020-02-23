---
title: vuex原理
date: 2020-02-21 13:17:45
tags:
   - vuex原理
---

### 什么是vuex
vuex是用于vue状态管理库， 将共享的数据抽离到全局， 以单例的方式存放， 形成单向数据流，同时利用vue响应式机制进行高效的管理和更新。


### vuex流程
Vue Component进行 Dispatch 操作   ==>  触发Vuex Action（异步获取服务器数据， action无法直接改变state）  ==>  通过Mutation改变state   ==>  根据state的变化渲染Vue Component视图


### vuex使用
```js
//store.js
Vue.use(Vuex)

//将store放入到Vue创建时的option中（注入到vue实例中
new Vue({
   el: '#app',
   store
}) 
```

### vuex原理
