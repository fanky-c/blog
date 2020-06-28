---
title: egg学习文档
date: 2020-06-27 20:31:36
tags:
    - Egg.js
    - Node.js框架
---
### Egg介绍

#### Egg设计原则
##### 插件机制
1. 可以通过插件根据自己的业务定制配置。 例如MySQL数据库封装成了 egg-mysql, Nunjucks 模板封装成了 egg-view-nunjucks.

##### 约定优先于配置
1. 插件开发、配置、使用约定
```js
// package.json
{
  "dependencies": {
    "egg": "^2.0.0",
    "egg-mysql": "^3.0.0"
  }
}

// config/plugin.js
// config/plugin.${env}.js/plugin.js
module.exports = {
  mysql: {
    enable: true,
    package: 'egg-mysql',
  },
}
```
2. 项目目录结构约定
```js
egg-project
├── package.json
├── app.js (可选)
├── agent.js (可选)
├── app
|   ├── router.js  //用于配置 URL 路由规则
│   ├── controller  //用于解析用户的输入，处理后返回相应的结果
│   |   └── home.js
│   ├── service (可选)  //用于编写业务逻辑层
│   |   └── user.js
│   ├── middleware (可选) //用于编写中间件
│   |   └── response_time.js
│   ├── schedule (可选)  //用于定时任务
│   |   └── my_task.js
│   ├── public (可选)  //放置静态资源
│   |   └── reset.css
│   ├── view (可选)    //用于放置模板文件
│   |   └── home.tpl
│   └── extend (可选)   //用于框架的扩展
│       ├── helper.js (可选)
│       ├── request.js (可选)
│       ├── response.js (可选)
│       ├── context.js (可选)
│       ├── application.js (可选)
│       └── agent.js (可选)
├── config
|   ├── plugin.js  //用于配置需要加载的插件
|   ├── plugin.local.js
|   ├── config.default.js //用于编写配置文件
│   ├── config.prod.js
|   ├── config.test.js (可选)
|   ├── config.local.js (可选)
|   └── config.unittest.js (可选)
└── test
    ├── middleware
    |   └── response_time.test.js
    └── controller
        └── home.test.js
```

### Egg功能
#### Egg内置对象
##### 继承Koa
###### 1. Application
1. 介绍
   * Application 对象几乎可以在编写应用时的任何一个地方获取到

2. 获取方式
```js
// app.js
module.exports = app => {
  app.cache = new Cache();
};

// app/controller/user.js
class UserController extends Controller {
  async fetch() {
    //第一种方法  
    this.ctx.body = this.app.cache.get(this.ctx.query.id);

    //第二种方法： 和Koa一样 通过Context对象
    //this.ctx.body = this.ctx.app.cache.get(this.ctx.query.id);
  }
}
```

###### 2. Context
1. 介绍：
   * Context 是一个请求级别的对象，继承自 Koa.Context。在每一次收到用户请求时，框架会实例化一个 Context 对象，这个对象封装了这次用户请求的信息，并提供了许多便捷的方法来获取请求参数或者设置响应信息。框架会将所有的 Service 挂载到 Context 实例上，一些插件也会将一些其他的方法和对象挂载到它上面

2. 获取方式
```js
// app.js
module.exports = app => {
  app.beforeStart(async () => {
    const ctx = app.createAnonymousContext();
    // preload before app start
    await ctx.service.posts.load();
  });
}

// app/middleware/index.js
...

// app/controller/home.js
const Controller = require('egg').Controller;
class HomeController extends Controller {
  async index() {
    const { ctx } = this;
    await ctx.render('index');
  }
}


// app/schedule/refresh.js
exports.task = async ctx => {
  await ctx.service.posts.refresh();
};
```

###### 3. Request & Response
1. 介绍
   * Request & Response是一个请求级别的对象，继承自 Koa.Request & Koa.Response
2. 获取方式
```js
// app/controller/user.js
class UserController extends Controller {
  async fetch() {
    const { app, ctx } = this;
    const id = ctx.request.query.id;
    ctx.response.body = app.cache.get(id);
  }
}
```

##### Egg框架扩展
######  1. Controller
######  2. Service
######  3. Helper
######  4. Config
######  5. Logger
######  6. Subscription


*文章来源* [https://eggjs.org/zh-cn/basics/objects.html]