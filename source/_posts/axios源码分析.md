---
title: axios源码分析
date: 2022-11-28 21:24:09
tags:
 - axios
---

## xhr、ajax、axios和fetch区别
1. xhr: 现代浏览器，最开始与服务器交换数据，都是通过XMLHttpRequest对象。它可以使用JSON、XML、HTML和text文本等格式发送和接收数据。
2. ajax: 为了方便操作dom并避免一些浏览器兼容问题，产生了jquery， 它里面的AJAX请求也兼容了不同的浏览器，可以直接使用.get、.pist。它就是对XMLHttpRequest对象的一层封装
3. Axios是一个基于promise的HTTP库，可以用在浏览器和 node.js 中。它本质也是对原生XMLHttpRequest的封装，只不过它是Promise的实现版本，符合最新的ES规范。
4. Fetch API提供了一个 JavaScript 接口，用于访问和操作HTTP管道的部分，例如请求和响应。它还提供了一个全局fetch()方法，该方法提供了一种简单，合理的方式来跨网络异步获取资源


## axios的特性
1. 基于Promise
2. 支持浏览器和nodejs环境
3. 可添加请求、响应拦截器和转换请求和响应数据
4. 请求可以取消、中断
5. 自动转换JSON数据
6. 客户端支持方法XSRF

## 带着问题去看代码
1. axios拦截器执行顺序？


## 源码分析
### 1、源码目录结构
```js
├── /lib/                          // 项目源码目
  └── /adapters/                     // 定义发送请求的适配器
      ├── http.js                       // node环境http对象
      ├── xhr.js                        // 浏览器环境XML对象
  └── /cancel/                       // 定义取消请求功能
  └── /helpers/                      // 一些辅助方法
  └── /core/                         // 一些核心功能
      ├──Axios.js                      // axios实例构造函数                 
      ├── createError.js               // 抛出错误
      ├── dispatchRequest.js           // 用来调用http请求适配器方法发送请求
      ├── InterceptorManager.js        // 拦截器管理器
      ├── mergeConfig.js               // 合并参数
      ├── settle.js                    // 根据http响应状态，改变Promise的状态
      ├── transformData.js             // 转数据格式
 └── axios.js                        // 入口，创建构造函数
 └── defaults.js                     // 默认配置
 └── utils.js                        // 公用工具函数
```

### 2、入口文件/lib/axios.js 
```js
function createInstance(defaultConfig) {
  // 根据默认配置构建个上下文对象，包括默认配置和请求、响应拦截器对象
  const context = new Axios(defaultConfig);

  // 创建实例 bind后返回的是一个函数，并且上下文指向context
  const instance = bind(Axios.prototype.request, context);

  // Copy axios.prototype to instance
  // 
  utils.extend(instance, Axios.prototype, context, {allOwnKeys: true});

  // Copy context to instance
  // 
  utils.extend(instance, context, null, {allOwnKeys: true});

  // Factory for creating new instances
  // 
  instance.create = function create(instanceConfig) {
    return createInstance(mergeConfig(defaultConfig, instanceConfig));
  };

  return instance;
}

const axios = createInstance(defaults);

// ...
// ...


axios.default = axios;

// this module should only have a default export
export default axios
```

### 3. Axios.prototype.request
```js
class Axios {
  constructor(instanceConfig) {
    this.defaults = instanceConfig;
    this.interceptors = {
      request: new InterceptorManager(),
      response: new InterceptorManager()
    };
  }

  request(configOrUrl, config) {
    // 代码
    // ...
    // ...
    const requestInterceptorChain = [];
    let synchronousRequestInterceptors = true;
    this.interceptors.request.forEach(function unshiftRequestInterceptors(interceptor) {
      if (typeof interceptor.runWhen === 'function' && interceptor.runWhen(config) === false) {
        return;
      }

      synchronousRequestInterceptors = synchronousRequestInterceptors && interceptor.synchronous;
      
      // 后添加的请求拦截器保存在数组的前面
      requestInterceptorChain.unshift(interceptor.fulfilled, interceptor.rejected);
    });

    const responseInterceptorChain = [];
    this.interceptors.response.forEach(function pushResponseInterceptors(interceptor) {

      // 后添加的响应拦截器保存在数组的后面
      responseInterceptorChain.push(interceptor.fulfilled, interceptor.rejected);
    });

    let promise;
    let i = 0;
    let len;

    if (!synchronousRequestInterceptors) {
      const chain = [dispatchRequest.bind(this), undefined];
      chain.unshift.apply(chain, requestInterceptorChain);
      chain.push.apply(chain, responseInterceptorChain);
      len = chain.length;

      promise = Promise.resolve(config);

      while (i < len) {
        promise = promise.then(chain[i++], chain[i++]);
      }

      return promise;
    }

    len = requestInterceptorChain.length;

    let newConfig = config;

    i = 0;

    while (i < len) {
      const onFulfilled = requestInterceptorChain[i++];
      const onRejected = requestInterceptorChain[i++];
      try {
        newConfig = onFulfilled(newConfig);
      } catch (error) {
        onRejected.call(this, error);
        break;
      }
    }

    try {
      promise = dispatchRequest.call(this, newConfig);
    } catch (error) {
      return Promise.reject(error);
    }

    i = 0;
    len = responseInterceptorChain.length;
     
    // 通过promise的then()串连起所有的请求拦截器/请求方法/响应拦截器
    while (i < len) {
      promise = promise.then(responseInterceptorChain[i++], responseInterceptorChain[i++]);
    }

    return promise;    
  }
}
export default Axios;
```

### 拦截器实现
```js
class InterceptorManager {
  constructor() {
    this.handlers = [];
  }
  // 添加拦截器 添加成功和失败回调方法
  use(fulfilled, rejected, options) {
    this.handlers.push({
      fulfilled,
      rejected,
      synchronous: options ? options.synchronous : false,
      runWhen: options ? options.runWhen : null
    });
    return this.handlers.length - 1;
  }

  // 注销指定拦截器
  eject(id) {
    if (this.handlers[id]) {
      this.handlers[id] = null;
    }
  }

  // 注销所有的拦截器
  clear() {
    if (this.handlers) {
      this.handlers = [];
    }
  }
  // 遍历执行
  forEach(fn) {
    utils.forEach(this.handlers, function forEachHandler(h) {
      if (h !== null) {
        fn(h);
      }
    });
  }
}

export default InterceptorManager;
```

<br />

[文章来源于](https://juejin.cn/post/7016255507392364557)