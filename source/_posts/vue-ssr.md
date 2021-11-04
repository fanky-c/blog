---
title: vue-ssr
date: 2018-07-06 16:34:25
tags:
    - vue
    - 同构    
---


## 通过个人项目分享下面几个方面：

* 构建流程
* 运行流程
* SSR的特点和利弊表

## 流程图
![流程图](/img/vue-ssr.png)

## 目录结构

```	
|- src 程序源文件
    |- api  客户端和服务器api文件
    |- assets  页面基础静态资源
    |- commons 公共JS、CSS
    |- components 公共的vue模块
    |- config  配置项
    |- directive 公共的vue指令
    |- filter  全局filter方法   
    |- router  路由控制 
    |- util    工具方法     
    |- store   状态存放          
    |- pages   各个页面入口
    |- plugin  扩展方法
    |- vue$methods  vue实例方法    
    |- app.js  创建vue实例
    |- App.vue 总入口
    |- entry-client.js 浏览器入口文件
    |- entry-server.js 服务端入口文件
    |- index.template.html 项目页面
|- server.js 创建服务端渲染器
|- node_modules node包文件
|- public 公共静态资源
|- build 开发环境和生产环境打包配置
    |- setup-dev-server 开发环境启动服务器
    |- webpack.client.config  打包浏览器端配置
    |- webpack.server.config  打包服务器端配置
|- dist 打包之后文件所在位置
|- doc 项目说明文档
    |- dev              
|- test 开发期间功能测试(暂时未添加)
    |- e2e 
    |- unit
```
*目录结构说明*
1，app.js入口文件，它的作用就是构建一个Vue的实例以供服务端(entry-server.js)和客户端(entry-client.js)使用。
2，entry-client.js  就是挂在Vue实例到DOM上。
3，entry-server.js 就是主要负责调用组件内定义的获取数据的方法，获取到SSR渲染所需数据，并存储到上下文环境中。这个函数会在每一次的渲染中重复的调用。


## webpack构建流程
然后我们的服务端代码和客户端代码通过webpack分别打包，生成Server Bundle和Client Bundle。
前者会运行在服务器上通过node生成预渲染的HTML字符串，发送到我们的客户端以便完成初始化渲染；
而客户端bundle就自由了，初始化渲染完全不依赖它了。客户端拿到服务端返回的HTML字符串后，会去“激活”这些静态HTML，是其变成由Vue动态管理的DOM，以便响应后续数据的变化。

## Vue SSR运行流程
* 构建一个vue的实例。 app.js创建vue实例，用一个工厂函数去封装它，以便每一个用户的请求都能够返回一个新的实例，也就是官网说到的避免交叉污染了。
* 服务端的entry.server.js拿到当前路由匹配的组件。调用组件中asyncData方法（也可以是自己定义的其他方法），此方法就是去调用我们vuex store中的方法去异步获取数据。
* node服务器把刚刚构建好的Vue实例渲染成HTML字符串，然后将拿到的数据混入我们的HTML字符串中，最后发送到我们客户端。
* 这里客户端拿到存在window中的数据混入我们客户端的vuex中，然后就是我们熟悉的客户端执行流程了。


## Vue SSR核心
* 数据预期和状态：在服务器端渲染(SSR)期间，我们本质上是在渲染我们应用程序的"快照"，所以如果应用程序依赖于一些异步数据，那么在开始渲染过程之前，需要先预取和解析好这些数据。
* 本项目中用vuex存放预取数据，也可以自己实现EventBus来实现。


## Vue SSR注意事项
* 在SSR中，创建Vue实例、创建store和创建router都是套了一层工厂函数的，目的就是避免数据的交叉污染。
* 在服务端只能执行生命周期中的created和beforeCreate，原因是在服务端是无法操纵dom的，所以可想而知其他的周期也就是不能执行的了。
* 服务器要求html书写规范，它实际输出的是一大串字符串，并不会像浏览器那样智能添加标签，不规范书写会导致混淆后的HTML和浏览器渲染的HTML不匹配。例如table > tbody > tr > td。
* asyncData()方法在子路由中获取不到异步数据，要放在顶级路由组件中。
* SSR服务端请求不带cookie。解决方案参考（https://www.mmxiaowu.com/article/596cbb2d436eb550a5423c30）



## Vue SSR优缺点：
### 优点：
   - SEO
   - 首屏渲染

### 缺点：
  - 服务器压力大
  - 性能问题
  - 学习和开发成本大


参考资料：https://ssr.vuejs.org/zh/  