---
title: egg加载器
date: 2021-07-19 11:40:04
tags:
   - egg加载器
   - egg loader
---

### 加载器
Egg 在 Koa 的基础上进行增强最重要的就是基于一定的约定，根据功能差异将代码放到不同的目录下管理，对整体团队的开发成本提升有着明显的效果。Loader 实现了这套约定，并抽象了很多底层 API 可以进一步扩展


### 框架、插件和应用的关系
#### 1. 框架继承
```js
+-----------------------------------+--------+
|      app1, app2, app3, app4       |        |
+-----+--------------+--------------+        |
|     |              |  framework3  |        |
+     |  framework1  +--------------+ plugin |
|     |              |  framework2  |        |
+     +--------------+--------------+        |
|                   Egg             |        |
+-----------------------------------+--------|
|                   Koa                      |
+-----------------------------------+--------+
```

#### 2. 加载单元执行顺序
***文件按表格内的顺序自上而下加载***
<img src="/img/egg.png" height = "auto" align=center />
> - 按插件 => 框架 => 应用依次加载
> - 插件之间的顺序由依赖关系决定，被依赖方先加载，无依赖按 object key 配置顺序加载，具体可以查看插件章节
> - 框架按继承顺序加载，越底层越先加载。

#### 3. 加载单元生命周期
1. 配置文件即将加载，这是最后动态修改配置的时机（configWillLoad）
2. 配置文件加载完成（configDidLoad）
3. 文件加载完成（didLoad）
4. 插件启动完毕（willReady）
5. worker 准备就绪（didReady）
6. 应用启动完成（serverDidReady）
7. 应用即将关闭（beforeClose）

### 加载器函数
#### 1. loadFile
```
// 用于加载一个文件，比如加载 app/xx.js 就是使用这个方法
// app/xx.js
module.exports = app => {
  console.log(app.config);
};

// app.js
// 以 app/xx.js 为例，我们可以在 app.js 加载这个文件
const path = require('path');
module.exports = app => {
  app.loader.loadFile(path.join(app.config.baseDir, 'app/xx.js'));
};
```
#### 2. loadToApp
```
// 用于加载一个目录下的文件到 app，比如 app/controller/home.js 会加载到 app.controller.home。
// app.js
// 以下只是示例，加载 controller 请用 loadController
module.exports = app => {
  const directory = path.join(app.config.baseDir, 'app/controller');
  app.loader.loadToApp(directory, 'controller');
};
```

#### 3. loadToContext
```
// 与 loadToApp 有一点差异，loadToContext 是加载到 ctx 上而非 app，而且是懒加载。加载时会将文件都放到一个临时对象上，在调用 ctx API 时才实例化对象。
// 以下为示例，请使用 loadService
// app/service/user.js
const Service = require('egg').Service;
class UserService extends Service {

}
module.exports = UserService;

// app.js
// 获取所有的 loadUnit
const servicePaths = app.loader.getLoadUnits().map(unit => path.join(unit.path, 'app/service'));

app.loader.loadToContext(servicePaths, 'service', {
  // service 需要继承 app.Service，所以要拿到 app 参数
  // 设置 call 在加载时会调用函数返回 UserService
  call: true,
  // 将文件加载到 app.serviceClasses
  fieldClass: 'serviceClasses',
});
```

### CustomLoader
loadToContext 和 loadToApp 可被 customLoader 配置替代。
```
// app.js
module.exports = app => {
  const directory = path.join(app.config.baseDir, 'app/adapter');
  app.loader.loadToApp(directory, 'adapter');
};;

=====
变成：
=====

// config/config.default.js
module.exports = {
  customLoader: {
    // 定义在 app 上的属性名 app.adapter
    adapter: {
      // 相对于 app.config.baseDir
      directory: 'app/adapter',
      // 如果是 ctx 则使用 loadToContext
      inject: 'app',
      // 是否加载框架和插件的目录
      loadunit: false,
      // 还可以定义其他 LoaderOptions
    }
  },
};
```