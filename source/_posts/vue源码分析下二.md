---
title: vue源码分析下二
date: 2019-01-04 10:41:56
tags:
    - vue
    - vue组件化
---

## 组件化介绍

1. 所谓组件化，就是把页面拆分成多个组件 (component)，每个组件依赖的 CSS、JavaScript、模板、图片等资源放在一起开发和维护。组件是资源独立的，组件在系统内部可复用，组件和组件之间可以嵌套。

## 组件化相关
### createComponent
1. 首先判断是否是真正html标签还是component。如果是html标签会是渲染普通VNode节点，负责渲染通过createComponent方法创建一个组件VNode
2. createComponent实现:
   1. 构造子类构造函数
   2. 安装组件钩子函数
   3. 实例化VNode

### Patch
1. 当我们通过 createComponent 创建了组件 VNode，接下来会走到 vm._update，执行 vm.__patch__ 比较新生成的VNode和旧的VNode，最后将差异（变化的节点）更新到真实的DOM树上。如果组件 patch 过程中又创建了子组件，那么DOM 的插入顺序是先子后父。
```
funciton patch(oldVnode,vnode,...){
    ...
}
//oldVnode: 旧的VNode或旧的真实DOM节点
//vnode: 新的VNode
//hydrating: 是否要和真实DOM混合
//removeOnly: 特殊的flag，用于<transition-group>
//parentElm: 父节点
//refElm: 新节点将插入到refElm之前
```

#### patch流程处理
1. 如果vnode不存在，但是oldVnode存在，说明是需要销毁旧节点，则调用invokeDestroyHook(oldVnode)来销毁oldVnode。
2. 如果vnode存在，但是oldVnode不存在，说明是需要创建新节点，则调用createElm来创建新节点。
3. 当vnode和oldVnode都存在时 
   1.
4. 返回VNode.elm


### 合并配置

### 生命周期

### 组件注册

#### 全局注册
```
Vue.component(tagName, options)
```

#### 局部注册
```
import HelloWorld from './components/HelloWorld'

export default {
  components: {
    HelloWorld
  }
}
```

### 异步组件

1. 方法一，普通异步组件：
```
Vue.component('async-example', function (resolve, reject) {
    // 这个特殊的 require 语法告诉 webpack
    // 自动将编译后的代码分割成不同的块，
    // 这些块将通过 Ajax 请求自动下载。  
   require(['./my-async-component'], resolve)
})
```
2. 方法二，promise异步组件
```
Vue.component(
  // 该 `import` 函数返回一个 `Promise` 对象。
  () => import('./my-async-component')
)
```
3. 方法三，高级异步组件
```
const AsyncComp = () => ({
  // 需要加载的组件。应当是一个 Promise
  component: import('./MyComp.vue'),
  // 加载中应当渲染的组件
  loading: LoadingComp,
  // 出错时渲染的组件
  error: ErrorComp,
  // 渲染加载中组件前的等待时间。默认：200ms。
  delay: 200,
  // 最长等待时间。超出此时间则渲染错误组件。默认：Infinity
  timeout: 3000
})
Vue.component('async-example', AsyncComp)
```
4. 总结：异步组件实现的本质是2次渲染，除了 0 delay 的高级异步组件第一次直接渲染成 loading 组件外，其它都是第一次渲染生成一个注释节点，当异步获取组件成功后，再通过forceRender强制重新渲染，这样就能正确渲染出我们异步加载的组件了。


[本文来源](https://ustbhuangyi.github.io/vue-analysis/components/)