---
title: webpack相关
date: 2020-02-14 16:17:30
tags:
     - webpack优化
     - webpack原理
---

## webpack核心概念

1. entry : webpack项目入口
2. module ： webpack一个模块对应一个文件， webpack从entry开始递归找出所有依赖模块
3. chunk ： 代码块， 一个chunk由多个模块组成， 主要用于代码合并和分割
4. loader ： 模块转换器， 用于将制定的模块内容转换为所需要的内容
5. plugins ： 拓展插件， 用于在webpack构建过程中特定的时机广播对应的事件， 插件可以监听这些事件发生，做特定的事情

## webpack流程

### 初始化
   1. 初始化相关参数： 从webpack.config.js和shell语句中读取

### 编译
   1. 开始编译： 用上一步读取到的配置参数初始化compiler对象， 加载所有的插件， 通过执行对象run方法进行编译
   2. 确定入口： 根据entry找出所有的入口文件
   3. 编译模块： 从入口文件出发， 调用所有的配置loader(从右到左，从下到上)对模块进行编译，再找出该模块依赖的模块，进行递归处理
   4. 完成编译： 经过上一步， 得到每个模块被编译之后的最终内容以及他们的依赖关系
### 输出
   1. 输出资源： 根据入口和模块之间的依赖关系， 组装一个个包含多个模块的chunk, 再将每个chunk转换一个单独的文件到输出列表中， 这一步是修改输出内容的最后一步
   2. 输出完成：根据上一步确定好输出内容之后， 根据webpack.config.js配置确定输出文件的路径和名称， 将文件写入磁盘(fs)

## webpack原理
1. 读取文件分析模块依赖
2. 对模块进行解析执行(深度遍历)
3. 对不同的模块使用相应的loader
4. 编译模块，生成抽象语法树AST
5. 循环遍历AST树，拼接输出js
> webpack输出的js文件：1. __webpack_require__是模块加载函数， 2. 每个模块都有唯一的id(0, 1, 2,...)， 3. __webpack_require__.e是异步加载模块函数（Promise实现）， 4. webpackJsonp用于从异步加载的文件中安装模块

## webpack模块打包原理
### webpack打包出来的代码为啥能在浏览器上运行
1. webpack对于ES模块/CommonJS模块的实现，是基于自己实现的webpack_require，所以代码能跑在浏览器中。
2. 从 webpack2 开始，已经内置了对 ES6、CommonJS、AMD 模块化语句的支持。但不包括新的ES6语法转为ES5代码，这部分工作还是留给了babel及其插件。
3. 在webpack中可以同时使用ES6模块和CommonJS模块。 因为 module.exports很像export default，所以ES6模块可以很方便兼容 CommonJS：import XXX from 'commonjs-module'。反过来CommonJS兼容ES6模块，需要额外加上default：require('es-module').default。

### webpack最终精简代码
```js
(function(modules) { 
    /**
     *  主要实现功能：
     *  1. 定义模块缓存
     *  2. 实现浏览器支持的require方法
     *     1. 根据moduleId判断是否已缓存模块， 如果有缓存就返回缓存模块
     *     2. 缓存模块相关信息
     *     3. 调用模块函数
     *     4. 标记已加载
     *     5. 返回module.exports
     *  3. 实现异步加载方法
     *     1. require.ensure() 通过回调函数执行接下来的流程
     *     2. webpack4 推荐 import(需要配合@babel/plugin-syntax-dynamic-import插件)  通过promise执行接下来的流程
     *  4. ...
     * **/
     function __webpack_require__(moduleId) {
        // 判断是否已缓存模块
        if(installedModules[moduleId]) {
            return installedModules[moduleId].exports;
        }
        //缓存模块
        var module = installedModules[moduleId] = {
            i: moduleId,
            l: false,
            exports: {}
        };
        //调用模块函数
        modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
        //标记模块为已加载
        module.l = true;
        // 返回module.exports
        return module.exports;
    }
})({
    "./src/bar.js":
    (function(module, exports, __webpack_require__) {
        //bar.js代码
    }),
    "./src/index.js":
    (function(module, exports, __webpack_require__) {
        //index.js代码
    })
});

```



## webpack优化
### 打包速度优化
#### 缩小文件查找范围
1. include指定查找范围、exclude排除文件查找范围

#### 减小编译的文件内容
1. 动态链接库DLL（webpack4可以用splitChunks, 不然代码会重复

#### 充分利用电脑多核性能
1. 多进程之HappyPack。 HappyPack就能让Webpack把任务分解给多个子进程去并发的执行，子进程处理完后再把结果发送给主进程，其中子进程的个数为cpu的个数减去1,需要在loader处修改如下

### 项目代码优化
#### 懒加载
1. require.ensure();
2. import();

#### tree shaking
1. 目的：消除无用到的代码， 推荐使用方法 { name } form './module.js' 
2. 原理：ES6引入模块是静态分析，所以编译的时候知道我们加载了哪些代码；进而分析哪些变量未用到，从而删除掉
3. 有一些副作用（babel转换代码、自执行函数、函数里面使用外部变量

#### 代码合并和分离(splitChunks)
1. 分离部分第三方库（vue、vuex、vue-router）功能，类似于动态链接库DLL
2. 合并公共代码（common

## webpack插件
### 插件相关概念
1. Tapable: webpack自己写的基础类Tapable, Compiler、Compilation等都是继承于Tabable类
2. Compiler: Webpack 环境所有的的配置信息，包含 options，loaders，plugins 这些信息
3. Compilation: 包含了当前的模块资源、编译生成资源、变化的文件等

### 插件创建流程
1. 创建一个javascript构造函数
2. 在他的原型定义apply方法
3. 指定触发webpack的事件钩子
4. 在钩子里面实现自己的业务逻辑
5. 功能实现完成，调用webpack提供的callback

### 使用案例
```js
//webpack1

function AddHeadTextPlugin(options) {
  this.options = options;
}

AddHeadTextPlugin.prototype.apply = function (compiler) {
  const self = this;
  compiler.plugin('emit', function(compilation, callback){
      let outputfile = compilation.options.output;
      let assets = compilation.assets;
      let keys = Object.keys(assets);

      callback();
  });

  compiler.plugin('done', function (compilation) {

  });

}

```