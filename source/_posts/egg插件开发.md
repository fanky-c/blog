---
title: egg插件使用和开发
date: 2021-08-29 14:39:47
tags:
 - egg
 - plugin
 - egg插件
---

### 介绍
#### 有了中间件为啥还要插件？
1. 中间件是定位拦截用户请求， 并在它的前后做一件事情， 例如：鉴权、访问日志、安全检查
2. 中间件加载是有先后顺序的， 但是中间件自身缺无法管理这种顺序， 只能交给使用者
3. 一些非常复杂的初始化逻辑， 需要在应用启动的时候完成， 如果放在中间件就不合适

#### 中间件、插件、应用的关系
一个插件就是一个迷你版的应用， 包含：
1. 包含了：Service、中间件、配置、扩展等
2. 没有 Controller 和 Router
3. 没有plugin.js, 只能声明跟其他插件的依赖， 不能决定插件是否开启
4. 插件本身可以包含中间件


### 使用
#### 1. plugin.js参数的配置
1. {Boolean} enable - 是否开启此插件，默认为 true
2. {String} package - npm 模块名称，通过 npm 模块形式引入插件
3. {String} path - 插件绝对路径，跟 上一个参数package 配置互斥
4. {Array} env - 只有在指定运行环境才能开启，会覆盖插件自身 package.json 中的配置

#### 2. egg内置插件
关闭内置插件
```js
// config/plugin.js
exports.cors = {
  enable: false;
};

// 也可以简写为：
exports.cors = false;
```
3. package 和 path
* package 是 npm 方式引入，也是最常见的引入方式
* path 是绝对路径引入，如应用内部抽了一个插件，但还没达到开源发布独立 npm 的阶段，或者是应用自己覆盖了框架的一些插件

```js
// config/plugin.js
const path = require('path');
exports.mysql = {
  enable: true,
  path: path.join(__dirname, '../lib/plugin/bg-mysql'),
};
```

#### 3. 内置插件列表
1. 异常处理 onerror 
2. session实现 session
3. 安全 security
4. 日志分割 logrotator
5. 定时任务 schedule
6. 模板引擎 view
7. 文件流式上传 multpart 
8. 等等...
### 开发
#### 1. 脚手架快速开发
```js
mkdir egg-test && cd egg-test
npm init egg --type=plugin
npm i
npm test
```

### 插件使用场景
#### 1. 扩展内置对象接口
1. app/extend/request.js - 扩展 Koa#Request 类
2. app/extend/response.js - 扩展 Koa#Response 类
3. app/extend/context.js - 扩展 Koa#Context 类
4. app/extend/helper.js  - 扩展 Helper 类
5. app/extend/application.js - 扩展 Application 类
6. app/extend/agent.js - 扩展 Agent 类

#### 2. 插入自定义中间件
```js
// 启动前想读取一些本地配置
// ${plugin_root}/app.js
const fs = require('fs');
const path = require('path');

module.exports = app => {
  app.customData = fs.readFileSync(path.join(app.config.baseDir, 'data.bin'));

  app.coreLogger.info('read data ok');
};
```
#### 3. 设置定时任务
```js
// 1. package.json
{
  "name": "your-plugin",
  "eggPlugin": {
    "name": "your-plugin",
    "dependencies": [ "schedule" ]
  }
}


//2.  ${plugin_root}/app/schedule/
exports.schedule = {
  type: 'worker',
  cron: '0 0 3 * * *',
  // interval: '1h',
  // immediate: true,
};

exports.task = async ctx => {
  // your logic code
};
```

### 插件开发实践
#### 1. 之前插件存在问题
1. 在一个应用中同时使用同一个服务的不同实例（连接到两个不同的 MySQL 数据库）。
2. 从其他服务获取配置后动态初始化连接（从配置中心获取到 MySQL 服务地址后再建立连接）。
   
#### 2. 实现
如果让插件各自实现，可能会出现各种奇怪的配置方式和初始化方式，所以框架提供了 app.addSingleton(name, creator) 方法来统一这一类服务的创建。需要注意的是在使用 app.addSingleton(name, creator) 方法时，配置文件中一定要有 client 或者 clients 为 key 的配置作为传入 creator 函数 的 config。

```js
// egg-mysql/app.js
module.exports = app => {
  // 第一个参数 mysql 指定了挂载到 app 上的字段，我们可以通过 `app.mysql` 访问到 MySQL singleton 实例
  // 第二个参数 createMysql 接受两个参数(config, app)，并返回一个 MySQL 的实例
  app.addSingleton('mysql', createMysql);
}


async function createMysql(config, app) {
  // 异步获取 mysql 配置
  const mysqlConfig = await app.configManager.getMysqlConfig(config.mysql);
  assert(mysqlConfig.host && mysqlConfig.port && mysqlConfig.user && mysqlConfig.database);
  // 创建实例
  const client = new Mysql(mysqlConfig);

  // 做启动应用前的检查
  const rows = await client.query('select now() as currentTime;');
  app.coreLogger.info(`[egg-mysql] init instance success, rds currentTime: ${rows[0].currentTime}`);

  return client;
}
```


#### 3. 应用层面使用案例
##### 单实例
```js
// config/config.default.js
module.exports = {
  mysql: {
    client: {
      host: 'mysql.com',
      port: '3306',
      user: 'test_user',
      password: 'test_password',
      database: 'test',
    },
  },
};

// app/controller/post.js
class PostController extends Controller {
  async list() {
    const posts = await this.app.mysql.query(sql, values);
  },
}
```

##### 多实例
```js
// config/config.default.js
exports.mysql = {
  clients: {
    // clientId, access the client instance by app.mysql.get('clientId')
    db1: {
      user: 'user1',
      password: 'upassword1',
      database: 'db1',
    },
    db2: {
      user: 'user2',
      password: 'upassword2',
      database: 'db2',
    },
  },
  // default configuration for all databases
  default: {
    host: 'mysql.com',
    port: '3306',
  },
}

// app/controller/post.js
class PostController extends Controller {
  async list() {
    const posts = await this.app.mysql.get('db1').query(sql, values);
  },
}
```

##### 动态创建实例 
```js
// app.js
module.exports = app => {
  app.beforeStart(async () => {
    // 从配置中心获取 MySQL 的配置 { host, post, password, ... }
    const mysqlConfig = await app.configCenter.fetch('mysql');
    // 动态创建 MySQL 实例
    app.database = await app.mysql.createInstanceAsync(mysqlConfig);
  });
};


// app/controller/post.js
class PostController extends Controller {
  async list() {
    const posts = await this.app.database.query(sql, values);
  },
}
```

### 插件寻址规则
1. 优先级A: 如果配置了 path，直接按照 path 加载
2. 优先级B: 没有 path 根据 package 名去查找，查找的顺序依次是：
  * 应用根目录下的 node_modules
  * 应用依赖框架路径下的 node_modules
  * 当前路径下的 node_modules （主要是兼容单元测试场景）



<br >
[文章来源1](https://eggjs.org/zh-cn/basics/plugin.html)
[文章来源2](https://eggjs.org/zh-cn/advanced/plugin.html)