---
title: 从web前端看macOs开发
date: 2019-07-10 20:00:05
tags:
     - macOs开发
     - web和mac对比
---

### 1: Native与webview交互

#### A:Native取得window对象
```
js:
window.location.href = 'http://spring-studio.net';

oc:
[[webView windowScriptObject] setValue:@"http://spring-studio.net"forKeyPath:@"location.href"];
```


#### B:Native调用JavaScript Function
##### js调用Native
```
oc:
[[webView windowScriptObject] evaluateWebScript:@"function x(x) { return x + 1;}"];

js:
window.x(1);
```
##### Native调用js
```
oc:
[[webView windowScriptObject] evaluateWebScript:[NSString stringWithFormat:@"showPromoAd('%@','%@');",self.word, computerSerialNumber()]];

js:
function showPromoAd(word, id){
    console.log(word, id);
};
```

#### C:Native操作DOM
* WebKit 里头，所有的 DOM 对象都继承自 DOMObject，DOMObject 又继承自 WebScriptObject，所以我们在取得了某个 DOM 对象之后，也可以从 Objective-C 程序中，要求这个 DOM 对象执行 JS 程序
```
js:
document.querySelector('#s').focus();

oc:
DOMDocument *document = [[webView mainFrame] DOMDocument];
[[document querySelector:@"#s"] callWebScriptMethod:@"focus"withArguments:nil];
```

### 2:Native和web页面jsbridge通信

#### 原理
* 在windows下面添加webviewJsbridge对象
* 然后在webview中动态的创建一个Iframe
* Native内部通过iframe的src进行通信，每次注册方法和调用方法动态的改变src
* 调用 js 中的方法_handleMessageFromObjC 进行数据交互

#### 工作流
* JS 端加入 src 为 https://bridge_loaded 的 iframe
* Native 端检测到 Request，检测如果是 bridge_loaded 则通过当前的 WebView 组件注入 WebViewJavascriptBridge_JS 代码
* 注入代码成功之后会加入一个 messagingIframe，其 src 为 https://wvjb_queue_message
* 之后不论是 Native 端还是 JS 端都可以通过 registerHandler 方法注册一个两端约定好的 HandlerName 的处理，也都可以通过 callHandler 方法通过约定好的 HandlerName 调用另一端的处理（两端处理消息的实现逻辑对称）


#### 用法
//Native端

```
1. 引入依赖库文件
#import "WebViewJavascriptBridge.h"

2. 实例化一个对象
self.bridge = [WebViewJavascriptBridge bridgeForWebView:webView];

3. 注册前端方法+主动推送给前端
//供前端调用
[self.bridge registerHandler:@"OCTest" handler:^(id data, WVJBResponseCallback responseCallback) {
    NSLog(@"ObjC Echo called with: %@", data);
    responseCallback(data);
}];

//主动推送给前端
[self.bridge callHandler:@"JsTest" data:nil responseCallback:^(id responseData) {
    NSLog(@"ObjC received response: %@", responseData);
}];
```

//web端
```
function setupWebViewJavascriptBridge(callback) {
    if (window.WebViewJavascriptBridge) { return callback(WebViewJavascriptBridge); }
    if (window.WVJBCallbacks) { return window.WVJBCallbacks.push(callback); }
    window.WVJBCallbacks = [callback];
    var WVJBIframe = document.createElement('iframe');
    WVJBIframe.style.display = 'none';
    WVJBIframe.src = 'https://__bridge_loaded__';
    document.documentElement.appendChild(WVJBIframe);
    setTimeout(function() { document.documentElement.removeChild(WVJBIframe) }, 0)
}

setupWebViewJavascriptBridge(function(bridge) {
	 //主动调用
    bridge.registerHandler('JsTest', function(data, responseCallback) {
        console.log("JS Echo called with:", data)
        responseCallback(data)
    })
	//接受oc端主动推送的消息
    bridge.callHandler('OCTest', {'key':'value'}, function responseCallback(responseData) {
        console.log("JS received response:", responseData)
    })
})
```

### 3: Native和web本地缓存数据区别

#### web本地数据缓存方式
* cookie
* sessionStorage
* localStorage
* indexedDB
* pwa离线解决方案(serviceWorker子线程、无法操作DOM)
 
#### Native本地数据缓存
* NSUserDefaults
  * 一般应用来存储应用设置信息，文件放在perference目录下。
* FMDB
  * FMDB是iOS平台的SQLite数据库框架，FMDB以OC的方式封装了SQLite的C语言API，使用起来更加面向对象
* 直接写文件
  * Documents:(iTunes同步设备时会备份该目录)
  * tmp
  * Library
    *Caches (不会被同步)
    *Preference(应用设置，iTunes同步设备时会备份该目录) 

### 4: Native和web多页面之间通信区别

#### web多页面通信(同源多tab，不同源父子页面)
##### 原理
* 获取页面句柄，定向通信。
* 共享内存，完成相应的业务。

##### 方法
* window.open()、postMessage
* BroadcastChannel
* SharedWorker
* cookie、localStorage。(sessionStorage不支持多页面通信)

#### Native多页面通信
* 属性传值
* 单例传值
* NSUserDefaults(本地设置)
* 代理传值
* block传值
* NSNotificationCenter(通知中心)
```
//先监听
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(updateValue:) name:@"postValue" object:nil];

-(void)updateValue:(NSNotification*)not{
    self.lable.text = not.userInfo[@"val"];
}

//发送广播
 [[NSNotificationCenter defaultCenter] postNotificationName:@"postValue" object:nil userInfo:@{@"val":self.inputText}];
```


### 5: Native和web多线程和异步流程控制

#### web多线程
* new worker(work.js)，把耗时较长的任务交给子线程处理，主线程处理ui操作，任务完成worker.postMessage()告诉给主线程。
* worker和主线程不在同一上下文，使用限制：不能访问DOM、BOM、同源策略
* [demo](https://github.com/mdn/simple-web-worker)

#### web异步
* ajax
* setTimeout
* Promise


#### Native多线程和流程控制
* GCD
  * 同步/异步、串行/并行
  * dispath_get_global_queue & dispatch_get_main_queue
  * dispatch_group_async
  * dispatch_once
  * dispatch_afte，将任务添加到队列延迟执行，类似setTimeout。

* NSOperation
  * 实现方式:
     * NSInvocationOperation & NSBlockOperation（同步阻塞）
     * 自定义类继承NSOperation
     
* NSThread