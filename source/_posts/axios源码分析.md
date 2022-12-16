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

### 4. 拦截器实现
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

<img src="/img/axios1.png" width="90%" />

**由于请求的发送需要在请求拦截器之后，在响应拦截器之前，所以数组先放入request，接着在数组的前后分别加入请求和响应拦截器，由于加入请求拦截器的方法是unshift，所以最后通过promise进行请求的链式调用的时候，我们可以看到执行顺序是从左往右的，所以最后注册的请求拦截器会最先执行，而响应拦截的执行顺序和注册顺序是一样的。**

### 5、dispatchRequest

我们进入到核心请求方法dispatchRequest中

```js
function throwIfCancellationRequested(config) {
  if (config.cancelToken) {
    config.cancelToken.throwIfRequested();
  }

  if (config.signal && config.signal.aborted) {
    throw new CanceledError(null, config);
  }
}


export default function dispatchRequest(config) {
  // 请求取消请求
  throwIfCancellationRequested(config);
  
  // 请求头
  config.headers = AxiosHeaders.from(config.headers);

  // 转换数据
  config.data = transformData.call(
    config,
    config.transformRequest
  );

  if (['post', 'put', 'patch'].indexOf(config.method) !== -1) {
    config.headers.setContentType('application/x-www-form-urlencoded', false);
  }
   
  // 适配器
  const adapter = adapters.getAdapter(config.adapter || defaults.adapter);

  return adapter(config).then(function onAdapterResolution(response) {
    throwIfCancellationRequested(config);

    // 转换响应头数据
    response.data = transformData.call(
      config,
      config.transformResponse,
      response
    );

    response.headers = AxiosHeaders.from(response.headers);

    return response;
  }, function onAdapterRejection(reason) {
    if (!isCancel(reason)) {
      throwIfCancellationRequested(config);

       // 转换响应头数据
      if (reason && reason.response) {
        reason.response.data = transformData.call(
          config,
          config.transformResponse,
          reason.response
        );
        reason.response.headers = AxiosHeaders.from(reason.response.headers);
      }
    }

    return Promise.reject(reason);
  });
}
```

### 6、适配器adapter
```js
function getDefaultAdapter() {
  var adapter;
  // 判断XMLHttpRequest对象是否存在 存在则代表为浏览器环境
  if (typeof XMLHttpRequest !== 'undefined') {
    // For browsers use XHR adapter
    adapter = require('./adapters/xhr');
    // node环境 使用原生http发起请求
  } else if (typeof process !== 'undefined' && Object.prototype.toString.call(process) === '[object process]') {
    adapter = require('./adapters/http');
  }
  return adapter;
}
```

### 7、axios主动取消请求

**终止一个尚未完成的网络请求, status状态为canceled**

1、使用axios

```js
import { CancelToken } from axios;

// source为一个对象 结构为 { token, cancel }
// token用来表示某个请求，是个promise
// cancel是一个函数，当被调用时，则取消token注入的那个请求
const source = CancelToken.source();

axios
    .get('/user', {
        // 将token注入此次请求
        cancelToken: source.token, 
    })
    .catch(function (thrown) {
        // 判断是否是因为主动取消而导致的
        if (axios.isCancel(thrown)) {
            console.log('主动取消', thrown.message);
        } else {
            console.error(thrown);
        }
    });

// 这里调用cancel方法，则会中断该请求 无论请求是否成功返回
source.cancel('我主动取消请求')


/**
 * 方法二
 * let cancel = null;
 * axios
    .get('/user', {
        cancelToken: new CancelToken(function executor(c){
            cancel = c;
        }) 
    })
 * cancel('我主动取消请求')
 * /
```


2、CancelToken构造函数

```js
class CancelToken {
  constructor(executor) {
    if (typeof executor !== 'function') {
      throw new TypeError('executor must be a function.');
    }

    let resolvePromise;

    this.promise = new Promise(function promiseExecutor(resolve) {
     // 把resolve方法提出来 当resolvePromise执行时，this.promise状态会变为fulfilled
      resolvePromise = resolve;
    });

    const token = this;

   
    this.promise.then(cancel => {
      if (!token._listeners) return;

      let i = token._listeners.length;

      while (i-- > 0) {
        token._listeners[i](cancel);
      }
      token._listeners = null;
    });

    
    this.promise.then = onfulfilled => {
      let _resolve;
      const promise = new Promise(resolve => {
        token.subscribe(resolve);
        _resolve = resolve;
      }).then(onfulfilled);

      promise.cancel = function reject() {
        token.unsubscribe(_resolve);
      };

      return promise;
    };

    executor(function cancel(message, config, request) {
      if (token.reason) {
        // Cancellation has already been requested
        return;
      }

      token.reason = new CanceledError(message, config, request);
      resolvePromise(token.reason);
    });
  }

  /**
   * Throws a `CanceledError` if cancellation has been requested.
   */
  throwIfRequested() {
    if (this.reason) {
      throw this.reason;
    }
  }

  /**
   * Subscribe to the cancel signal
   */

  subscribe(listener) {
    if (this.reason) {
      listener(this.reason);
      return;
    }

    if (this._listeners) {
      this._listeners.push(listener);
    } else {
      this._listeners = [listener];
    }
  }

  /**
   * Unsubscribe from the cancel signal
   */

  unsubscribe(listener) {
    if (!this._listeners) {
      return;
    }
    const index = this._listeners.indexOf(listener);
    if (index !== -1) {
      this._listeners.splice(index, 1);
    }
  }

  /**
   * Returns an object that contains a new `CancelToken` and a function that, when called,
   * cancels the `CancelToken`.
   */
  static source() {
    let cancel;
    const token = new CancelToken(function executor(c) {
      cancel = c;
    });
    return {
      token,
      cancel
    };
  }
}

export default CancelToken;
```


3、请求中是如何处理的cancel
```js
// 订阅
if (config.cancelToken || config.signal) {
  // Handle cancellation
  // eslint-disable-next-line func-names
  onCanceled = cancel => {
    if (!request) {
      return;
    }
    reject(!cancel || cancel.type ? new CanceledError(null, config, request) : cancel);
    request.abort();
    request = null;
  };

  config.cancelToken && config.cancelToken.subscribe(onCanceled);
  if (config.signal) {
    config.signal.aborted 
           ? onCanceled() 
           : config.signal.addEventListener('abort', onCanceled);
  }
}


// 取消订阅
let onCanceled;
function done() {
  if (config.cancelToken) {
    config.cancelToken.unsubscribe(onCanceled);
  }

  if (config.signal) {
    config.signal.removeEventListener('abort', onCanceled);
  }
}
```


<img src="/img/axios2.png" width="90%" />

<br />

[文章来源于](https://juejin.cn/post/7016255507392364557)

[class static静态方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Classes/static)