---
title: vue源码分析上
date: 2018-07-08 21:22:55
tags:
    - vue
    - 模板编译
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



#### a，生成AST

*AST简介*

AST即抽象语法树，我们的代码运行在浏览器的时候，浏览器首先就会把我们的代码解析为AST然后再进一步把语法树转化为字节码或者直接生成机器码。所以他对于浏览器很重要，同样的对于开发者来说，AST也很重要，我们通过ast可以精准定位到代码的任何地方，从而可以对代码进行一系列操作。比如代码的语法检查，压缩以及代码结构的改变等等。webpack、UglifyJS等工具的核心都是通过ast来操作代码的。语法树记录了其对应代码的所有信息。


*JavaScript Parser*

代码解析成AST，是通过js Parser来实现的，不同的parser，生成的AST格式是不同的。比如chrome和firefox，由于jsParser不同，其生成的AST格式也不同，优化AST的格式有时也是浏览器厂商提高浏览器效率的一种方式。

[这个网站](https://astexplorer.net/)可以即时看到不同parser解析出的ast

```js
const ast = parse(template.trim(), options)
``` 


#### b，优化静态内容

静态内容就是和数据没有关系，当数据更新时，不需要刷新的内容。比如{ { text } }就是非静态内容，当数据变化的时候，这里的内容也会被更新。
baseCompile函数中，通过下面这行代码实现了静态内容的优化

```js
if (options.optimize !== false) {
    optimize(ast, options)
  }
``` 
这其中的optimize函数，其实主要做了两件事:
* 标记静态&非静态节点
* 标记静态根结点  

```js
markStatic(root) // 标记所有静态&非静态节点
  
markStaticRoots(root, false) // 标记静态根节点
``` 

* 标记静态&非静态结点

其标记的思路是从最底层往上一步一步的去标记静态节点。先标记最底层的叶子结点，再往上标记其父节点。一旦当前节点被标记为非静态节点，那么他所有的父节点都会被标记为非静态节点。

每个结点是否为静态的判断依据是：isStatic(node)=true && 子节点是静态节点

isStatic函数:

```js
function isStatic (node: ASTNode): boolean {
  if (node.type === 2) { // expression
    return false
  }
  if (node.type === 3) { // text
    return true
  }
  return !!(node.pre || (
    !node.hasBindings && // no dynamic bindings
    !node.if && !node.for && // not v-if or v-for or v-else
    !isBuiltInTag(node.tag) && // not a built-in
    isPlatformReservedTag(node.tag) && // not a component
    !isDirectChildOfTemplateFor(node) &&
    Object.keys(node).every(isStaticKey) // node中的所有属性  是否isStaticKey()为true
  ))
}
``` 
*其实就是先判断当前结点是否为表达式，是的话直接返回false，再判断是会否为纯为本，是的话直接返回true，如果都不是，再去判断是否绑定了v-for、v-if等属性。*

* 标记静态根结点  

与标记静态&非静态结点的方向正好相反，标记静态跟结点是从根结点，到叶子节点，上往下去标记的。

每个结点是否为静态结点的标记依据是：

该结点为静态结点&&（结点有多个子节点||有一个子节点，但是这个子节点不是纯文本）

以上就是标记跟结点的实现思路。

至此所有静态/非静态信息就标记完了。这些标记在每次vue更新DOM时都会起到很重要的优化作用，因为遇到静态节点，就知道，其自身以及子节点内容都无需更新，可直接跳过，由此很大的提高了vue的更新速度。




#### c，生成render字符串

baseCompile函数中，下面这行为生成render字符串的代码

```js
const code = generate(ast, options)  
``` 




文档参考：https://ustbhuangyi.github.io/vue-analysis/prepare/directory.html   