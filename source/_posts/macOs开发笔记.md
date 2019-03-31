---
title: macOs开发笔记
date: 2018-12-05 21:52:39
tags: 
    - mac
    - xcode
    - oc
    - ios
---

* 入门参考：https://www.jianshu.com/p/be92a0a0f1a0
* 个人练习：https://github.com/fanky-c/OcTest

### 1: objective-c与javascript交互

####  用 Objective-C 取得与设定 JavaScript 对象
```
js:
window.location.href = 'http://spring-studio.net';

oc:
[[webView windowScriptObject] setValue:@"http://spring-studio.net"forKeyPath:@"location.href"];
```
1. JS 虽然是 OO，但是并没有 class，所以将 JS 对象传到 Obj C 程序里头，除了基本字串会转换成 NSString、基本数字会转成 NSNumber，像是 Array 等其他对象，在 Objective-C 中，都是 WebScriptObject 这个 Class。意思就是，JS 的 Array 不会帮你转换成 NSArray。
2. 从 JS 里头传一个空对象给 Objective-C 程序，用的不是 Objective-C 里头原本表示「没有东西」的方式，像是 NULL、nil、NSNull 等，而是专属 WebKit 使用的 WebUndefined。

####  用 Objective C 调用 JavaScript function
1. js调用oc
```
oc:
[[webView windowScriptObject] evaluateWebScript:@"function x(x) { return x + 1;}"];

js:
window.x(1);
```

2. oc调用js
```
oc:
[[webView windowScriptObject] evaluateWebScript:[NSString stringWithFormat:@"showPromoAd('%@','%@');",self.word, computerSerialNumber()]];

js:
function showPromoAd(word, id){};
```

#### DOM
1. WebKit 里头，所有的 DOM 对象都继承自 DOMObject，DOMObject 又继承自 WebScriptObject，所以我们在取得了某个 DOM 对象之后，也可以从 Objective-C 程序中，要求这个 DOM 对象执行 JS 程序
```
js:
document.querySelector('#s').focus();

oc:
DOMDocument *document = [[webView mainFrame] DOMDocument];
[[document querySelector:@"#s"] callWebScriptMethod:@"focus"withArguments:nil];
```

####  用 JavaScript 存取 Objective C 的 Value
1. 要让网页中的 JS 程序可以调用 Objective-C 对象，方法是把某个 Objective-C 对象注册成 JS 中 window 对象的属性。之后，JS 便也可以调用这个对象的 method，也可以取得这个对象的各种 Value，只要是 KVC 可以取得的 Value，像是 NSString、NSNumber、NSDate、NSArray、NSDictionary、NSValue…等。JS 传 Array 到 Objective-C 时，还需要特别做些处理才能变成 NSArray，从 Obj C 传一个 NSArray 到 JS 时，会自动变成 JS Array。

2. 首先我们要注意的是将 Objective-C 对象注册给 window 对象的时机，由于每次重新载入网页，window 对象的内容都会有所变动－毕竟每个网页都会有不同的 JS 程序，所以，我们需要在适当的时机做这件事情。我们首先要指定 WebView 的 frame loading delegate（用 setFrameLoadDelegate:），并且实作 webView:didClearWindowObject:forFrame:，WebView 只要更新了 windowScriptObject，就会调用这一段程序。

####  用 JavaScript 调用 Objective C method
1. Objective-C 的语法沿袭自 SmallTalk，Objective-C 的 selector，与 JS 的 function 语法有相当的差异。WebKit 预设的实事是，如果我们要在 JS 调用 Objective-C selector，就是把所有的参数往后面摆，并且把所有的冒号改成底线，而原来 selector 如果有底线的话，又要另外处理。


### 2: jsbridge

#### OC端
1. 引入依赖文件
```
#import "WebViewJavascriptBridge.h"
...
@property WebViewJavascriptBridge* bridge;
```
2. 实例化一个对象
```
self.bridge = [WebViewJavascriptBridge bridgeForWebView:webView];
```
3. OC端 注册一个方法+ 调用该方法的函数
```
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

#### JS端
```js
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

#### 原理
1. 在windows下面添加webviewJsbridge对象
2. 然后动态的在webview中创建一个Iframe
3. oc内部通过iframe的src惊喜通信，每次注册方法和调用方法动态的改变src
4. 调用 js 中的方法_handleMessageFromObjC 进行数据交互

### 3: ios本地缓存数据

#### 直接写文件方式
1. 可以存储的对象有NSString、NSArray、NSDictionary、NSData、NSNumber，数据全部存放在一个属性列表文件（*.plist文件）中。

#### NSUserDefaults
1. 一般应用来存储应用设置信息，文件放在perference目录下。

#### 归档操作（NSkeyedArchiver）
1. 不同于前面两种，它可以把自定义对象存放在文件中。

#### coreData
1. coreData是苹果官方iOS5之后推出的综合型数据库。

#### FMDB
1. FMDB是iOS平台的SQLite数据库框架，FMDB以OC的方式封装了SQLite的C语言API，使用起来更加面向对象，省去了很多麻烦、冗余的C语言代码。


### 4: 数据之间通信


### 5: 文件操作
1. NSFileHandle：文件内容的读取和写入
2. NSFileManager：文件的删除和创建等
3. [参考](https://github.com/fanky-c/OcTest/blob/master/IOS%E6%96%87%E4%BB%B6%E6%93%8D%E4%BD%9C/IOS%E6%96%87%E4%BB%B6%E6%93%8D%E4%BD%9C/ViewController.m)

### 6: 多线程

#### PThread(c框架，极少使用)
1. [github参考](https://github.com/fanky-c/OcTest/blob/master/pThread/pThread/ViewController.m)

#### NSThread
1. 三种创建方式
```
//方式一
NSThread *thread1 = [[NSThread alloc] initWithTarget: self selector:@selector(runThread1) object:nil];
[thread1 setName:@"name_thread1"]; //在selector方法里面获取当前线程[NSThread currentThread].name;
[thread1 setThreadPriority:0.5]; //优先级：0-1
[thread1 start];


//方式二
[NSThread detachNewThreadSelector:@selector(runThread1) toTarget:self withObject:nil];


//方式三
//回到主线程：performSelectorOnMainThread
[self performSelectorInBackground:@selector(runThread1) withObject:nil];
```

2. [github参考](https://github.com/fanky-c/OcTest/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8BNSThread/%E5%A4%9A%E7%BA%BF%E7%A8%8BNSThread/TicketMangager.m)

#### GCD
1. 介绍
*类似前端的new worker(), 把耗时较长的任务交给子线程处理，主线程处理ui操作，任务完成worker.postMessage()告诉给主线程。*

1.


#### NSOperation


#### dispatch_semaphore（信号量）
```
- (NSInteger)methodSync {
    __block NSInteger result = 0;
    dispatch_semaphore_t sema = dispatch_semaphore_create(0);
    [self methodAsync:^(NSInteger value) {
        result = value;
        dispatch_semaphore_signal(sema);
    }];
   // 这里本来同步方法会立即返回，但信号量=0使得线程阻塞
   // 当异步方法回调之后，发送信号，信号量变为1，这里的阻塞将被解除，从而返回正确的结果
    dispatch_semaphore_wait(sema, DISPATCH_TIME_FOREVER);
    NSLog(@"methodSync 结束 result:%ld", (long)result);
    return result;
}
```