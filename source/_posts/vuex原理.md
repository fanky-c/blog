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

#### Vuex是怎样把store注入到Vue实例中？
```js
/*
   暴露给外部的插件install方法，供Vue.use调用安装vuex。做了一下2件事情：
      1. 保存Vue 同时检测是否重复安装
      2. 将VueInit(Vuex的init钩子)混淆进Vue的beforeCreate(vue2.0) / _init(vue1.0)
*/

/*暴露给外部的插件install方法，供Vue.use调用安装插件*/
export function install (_Vue) {
  if (Vue) {
    /*避免重复安装（Vue.use内部也会检测一次是否重复安装同一个插件）*/
    if (process.env.NODE_ENV !== 'production') {
      console.error(
        '[vuex] already installed. Vue.use(Vuex) should be called only once.'
      )
    }
    return
  }
  /*保存Vue，同时用于检测是否重复安装*/
  Vue = _Vue

  /*将vuexInit混淆进Vue的beforeCreate(Vue2.0)或_init方法(Vue1.0)*/
  applyMixin(Vue)
}

function applyMixin (Vue) {
  var version = Number(Vue.version.split('.')[0]);

  if (version >= 2) {
    Vue.mixin({ beforeCreate: vuexInit });
  } else {
    var _init = Vue.prototype._init;
    Vue.prototype._init = function (options) {
      if ( options === void 0 ) options = {};
      options.init = options.init
        ? [vuexInit].concat(options.init)
        : vuexInit;
      _init.call(this, options);
    };
  }

function vuexInit () {
    var options = this.$options;
    // store injection
    if (options.store) {
      this.$store = typeof options.store === 'function'
        ? options.store()
        : options.store;
    } else if (options.parent && options.parent.$store) {
      this.$store = options.parent.$store;
    }
  }
}
Vuex源码：https://github.com/vuejs/vuex/blob/dev/dist/vuex.esm.js
```
