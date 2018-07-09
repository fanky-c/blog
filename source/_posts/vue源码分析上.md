---
title: vue源码分析上
date: 2018-07-08 21:22:55
tags:
    - vue
---


## 概论

本文主要讲vue源码分析 -- **模板编译**

*以v2.5.16版本为分析例子(https://github.com/vuejs/vue/tree/v2.5.16)*

### vue的核心可以分为三个大块

- 数据处理和双向绑定
- 模板编译
- 虚拟DOM


### vue的源码目录

* circleci：是一个持续集成与部署服务。vue使用了这个服务来部署项目，该文件夹下为circleci部署所需的配置文件
benchmarks：这个文件夹里面，都是作者对于vue的性能测试。其中在性能测试代码中，有一个api大家可以关注下，window.performance.now()，该api经常在衡量代码运行时间，运行效率时被用到

* example： 相当于vue的使用说明，里面都是尤大写的vue使用的小demo

* flow: flow是一个用来进行静态类型检查的工具。它的作用是让现有的JavaScript语法可以事先作类型的声明(定义)，然后在开发过程中去进行自动检查。vue使用了该工具。该文件夹中，都是作者定义的静态类型。

* test： 测试用例

* src:  vue的所有重要代码，都写在了这里面。详细说一下其下的各个文件夹。
  - compiler：模板解析模板编译的相关代码，也是本次组会要跟大家进行解读的部分
  - core 
    * components： 全局组件定义的代码
    * global-api： 添加在vue对象上的方法。比如Vue.use，Vue.enxtend等
    * instance： vue实例相关的内容，比如生命周期，事件等。
    * observer： vue双向绑定部分的代码
    * vdom： 虚拟dom相关的代码
    * util： vue的工具方法 
  - platforms：一个是web平台，我们常用的就是这个。一个是weex平台，weex是一个用于开发原生应用的框架，也可以把它视作vue-native。  
  - server：服务端渲染的相关代码。  
  - shared： vue共享的工具和方法。        
  - sfc： .vue 文件解析 


### 模板编译代码解读 

```js
export const createCompiler = createCompilerCreator(function baseCompile(
  template: string,
  options: CompilerOptions
): CompiledResult {
  const ast = parse(template.trim(), options)
  if (options.optimize !== false) {
    optimize(ast, options)
  }
  const code = generate(ast, options)
  return {
    ast,
    render: code.render,
    staticRenderFns: code.staticRenderFns
  }
})
``` 
vue模板编译的三个重要阶段都在上面的baseCompile函数中，这三个阶段分别是：
* 生成AST
* 优化静态内容
* 生成render。
  


文档参考：https://ustbhuangyi.github.io/vue-analysis/prepare/directory.html   