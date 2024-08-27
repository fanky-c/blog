---
title: jsbridge原理
date: 2023-10-31 17:50:55
tags:
 - jssdk
 - JSBridge
 - WebViewJavascriptBridge
---


<img src="/img/jsbridge1.webp" width="auto" />

<img src="/img/jsbridge2.awebp" width="auto" />

## 1. JSBridge 的起源
开发维护成本 和 更新成本 较低的 Web 技术成为混合开发中几乎不二的选择，而作为 Web 技术逻辑核心的 JavaScript 也理所应当肩负起与其他技术『桥接』的职责，并且作为移动不可缺少的一部分，**任何一个移动操作系统中都包含可运行 JavaScript 的容器，例如 WebView 和 JSCore。所以，运行 JavaScript 不用像运行其他语言时，要额外添加运行环境。**因此，基于上面种种原因，JSBridge 应运而生。

移动端混合开发中的 JSBridge，主要被应用在两种形式的技术方案上：

1. 基于 Web 的 Hybrid 解决方案：例如微信浏览器、各公司的 Hybrid 方案
2. 非基于 Web UI 但业务逻辑基于 JavaScript 的解决方案：例如 React-Native


## 2. JSBridge 的用途
JSBridge 就像其名称中的『Bridge』的意义一样，是 Native 和非 Native 之间的桥梁，**它的核心是 构建 Native 和非 Native 间消息通信的通道，而且是 双向通信的通道。**

所谓 双向通信的通道:

1. JS 向 Native 发送消息 : 调用相关功能、通知 Native 当前 JS 的相关状态等
2. Native 向 JS 发送消息 : 回溯调用结果、消息推送、通知 JS 当前 Native 的状态等

*消息都是单向的，那么调用 Native 功能时 Callback 怎么实现的？*

## 3. JSBridge 的实现原理

JavaScript 是运行在一个单独的 JS Context 中（例如，WebView 的 Webkit 引擎、JSCore）。由于这些 Context 与原生运行环境的天然隔离，我们可以将这种情况与 RPC（Remote Procedure Call，远程过程调用）通信进行类比，将 Native 与 JavaScript 的每次互相调用看做一次 RPC 调用。

在 JSBridge 的设计中，可以把前端看做 RPC 的客户端，把 Native 端看做 RPC 的服务器端，从而 JSBridge 要实现的主要逻辑就出现了：通信调用（Native 与 JS 通信） 和 句柄解析调用。

### 3.1 JSBridge 的通信原理

#### 3.1.1 JavaScript 调用 Native

JavaScript调用 Native 的方式，主要有两种：**注入 API 和 拦截 URL SCHEME。**

**方式一、注入API**
注入 API 方式的主要原理是，通过 WebView 提供的接口，向 JavaScript 的 Context（window）中注入对象或者方法，让 JavaScript 调用时，直接执行相应的 Native 代码逻辑，达到 JavaScript 调用 Native 的目的。

客户端注入：

```js
// wkwebview
@interface WKWebVIewVC ()<WKScriptMessageHandler>

@implementation WKWebVIewVC

- (void)viewDidLoad {
    [super viewDidLoad];

    WKWebViewConfiguration* configuration = [[WKWebViewConfiguration alloc] init];
    configuration.userContentController = [[WKUserContentController alloc] init];
    WKUserContentController *userCC = configuration.userContentController;
    // 注入对象，前端调用其方法时，Native 可以捕获到
    [userCC addScriptMessageHandler:self name:@"nativeBridge"];

    WKWebView wkWebView = [[WKWebView alloc] initWithFrame:self.view.frame configuration:configuration];

    // TODO 显示 WebView
}

- (void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message {
    if ([message.name isEqualToString:@"nativeBridge"]) {
        NSLog(@"前端传递的数据 %@: ",message.body);
        // Native 逻辑
    }
}


// android
publicclassJavaScriptInterfaceDemoActivityextendsActivity{
private WebView Wv;

    @Override
    public void onCreate(Bundle savedInstanceState){
        super.onCreate(savedInstanceState);

        Wv = (WebView)findViewById(R.id.webView);
        final JavaScriptInterface myJavaScriptInterface = new JavaScriptInterface(this);

        Wv.getSettings().setJavaScriptEnabled(true);
        Wv.addJavascriptInterface(myJavaScriptInterface, "nativeBridge");

        // TODO 显示 WebView

    }

    public class JavaScriptInterface{
         Context mContext;

         JavaScriptInterface(Context c) {
             mContext = c;
         }

         public void postMessage(String webMessage){
             // Native 逻辑
         }
     }
}

// harmonyOS
// 可以通过 Web 组件的 javaScriptProxy() 方法，或者 WebviewController.registerJavaScriptProxy() 方法
      Web({ src: this.url, controller: this.controller })
        .javaScriptAccess(true)
        .javaScriptProxy({
          object: this,
          name: "nativeBridge",
          methodList: ["postMessage"],
          controller: this.controller,
      })
```

前端调用方式：

```js
// ios
window.webkit.messageHandlers.nativeBridge.postMessage(message);

// android
window.nativeBridge.postMessage(message);
```


**方式二、拦截URL SCHEME**

**先解释一下 URL SCHEME：URL SCHEME是一种类似于url的链接，是为了方便app直接互相调用设计的，**形式和普通的 url 近似，主要区别是 protocol 和 host 一般是自定义的，例如: qunarhy://hy/url?url=ymfe.tech，protocol 是 qunarhy，host 则是 hy。

拦截 URL SCHEME 的主要流程是：Web 端通过某种方式（例如 iframe.src）发送 URL Scheme 请求，之后 Native 拦截到请求并根据 URL SCHEME（包括所带的参数）进行相关操作。

在时间过程中，这种方式有一定的 缺陷：

1. 使用 iframe.src 发送 URL SCHEME 会有 url 长度的隐患。
2. 创建请求，需要一定的耗时，比注入 API 的方式调用同样的功能，耗时会较长。

但是之前为什么很多方案使用这种方式呢？因为它 支持 iOS6。而现在的大环境下，iOS6 占比很小，基本上可以忽略，所以并不推荐为了 iOS6 使用这种 并不优雅 的方式。


#### 3.1.2 Native 调用 JavaScript

相比于 JavaScript 调用 Native， Native 调用 JavaScript 较为简单，毕竟不管是 iOS 的 UIWebView 还是 WKWebView，还是 Android 的 WebView 组件，都以子组件的形式存在于 View/Activity 中，直接调用相应的 API 即可

Native 调用 JavaScript，其实就是执行拼接 JavaScript 字符串，从外部调用 JavaScript 中的方法，因此 JavaScript 的方法必须在全局的 window 上。

```js
// wkwebview
[wkWebView evaluateJavaScript:javaScriptString completionHandler:completionHandler];

// android
webView.loadUrl("javascript:" + javaScriptString);

// android 新版本
webView.evaluateJavascript(javaScriptString, new ValueCallback<String>() {
    @Override
    publicvoidonReceiveValue(String value){

    }
});
```

### 3.2 JSBridge 接口实现

从上面的剖析中，可以得知，JSBridge 的接口主要功能有两个：调用 Native（给 Native 发消息） 和 接被 Native 调用（接收 Native 消息）。因此，JSBridge 可以设计如下：

```js
/**
 * 在上面的文章中，提到过 RPC 中有一个非常重要的环节是 句柄解析调用，
 * 这点在 JSBridge 中体现为 句柄与功能对应关系。
 * 同时，我们将句柄抽象为 桥名（BridgeName），最终演化为 一个 BridgeName 对应一个 Native 功能或者一类 Native 消息
 */
window.JSBridge = {
    // 调用 Native
    invoke: function(bridgeName, data) {
        // 判断环境，获取不同的 nativeBridge
        nativeBridge.postMessage({
            bridgeName: bridgeName,
            data: data || {}
        });
    },
    receiveMessage: function(msg) {
        var bridgeName = msg.bridgeName;
        var data = msg.data || {};
        // 具体逻辑
    }
};
```

JSBridge 大概的雏形出现了。现在终于可以着手解决这个问题了：消息都是单向的，那么调用 Native 功能时 Callback 怎么实现的？

对于 JSBridge 的 Callback ，其实就是 RPC 框架的回调机制。当然也可以用更简单的 JSONP 机制解释:

> 当发送 JSONP 请求时，url 参数里会有 callback 参数，其值是 当前页面唯一 的，而同时以此参数值为 key 将回调函数存到 window 上，随后，服务器返回 script 中，也会以此参数值作为句柄，调用相应的回调函数。

由此可见，callback 参数这个 唯一标识 是这个回调逻辑的关键。这样，我们可以参照这个逻辑来实现 JSBridge：用一个自增的唯一 id，来标识并存储回调函数，并把此 id 以参数形式传递给 Native，而 Native 也以此 id 作为回溯的标识。这样，即可实现 Callback 回调逻辑。


```js
(function () {
    var id = 0,
        callbacks = {},
        registerFuncs = {};

    window.JSBridge = {
        // 调用 Native
        invoke: function(bridgeName, callback, data) {
            // 判断环境，获取不同的 nativeBridge
            var thisId = id ++; // 获取唯一 id
            callbacks[thisId] = callback; // 存储 Callback
            nativeBridge.postMessage({
                bridgeName: bridgeName,
                data: data || {},
                callbackId: thisId // 传到 Native 端
            });
        },
        receiveMessage: function(msg) {
            var bridgeName = msg.bridgeName;
            var data = msg.data || {};
            var callbackId = msg.callbackId;  // Native 将 callbackId 原封不动传回
            var responstId = msg.responstId;
            // 具体逻辑
            // bridgeName 和 callbackId 不会同时存在
            if (callbackId) {
                if (callbacks[callbackId]) { // 找到相应句柄
                    callbacks[callbackId](msg.data); // 执行调用
                }
            } else if (bridgeName) {
                if (registerFuncs[bridgeName]) { // 通过 bridgeName 找到句柄
                    var ret = {},
                        flag = false;
                    registerFuncs[bridgeName].forEach(function(callback) => {
                        callback(data, function(r) {
                            flag = true;
                            ret = Object.assign(ret, r);
                        });
                    });
                    if (flag) {
                        nativeBridge.postMessage({ // 回调 Native
                            responstId: responstId,
                            ret: ret
                        });
                    }
                }
            }
        },
        register: function(bridgeName, callback) {
            if (!registerFuncs[bridgeName])  {
                registerFuncs[bridgeName] = [];
            }
            registerFuncs[bridgeName].push(callback); // 存储回调
        }
    };
})();
```
> 这一节主要讲的是，JavaScript 端的 JSBridge 的实现，对于 Native 端涉及的并不多。在 Native 端配合实现 JSBridge 的 JavaScript 调用 Native 逻辑也很简单，**主要的代码逻辑是：接收到 JavaScript 消息 => 解析参数，拿到 bridgeName、data 和 callbackId => 根据 bridgeName 找到功能方法，以 data 为参数执行 => 执行返回值和 callbackId 一起回传前端。 Native 调用 JavaScript 也同样简单，直接自动生成一个唯一的 ResponseId，并存储句柄，然后和 data 一起发送给前端即可。**



## 4. JSBridge 如何引用

### 4.1 由 Native 端进行注入

注入方式和 Native 调用 JavaScript 类似，直接执行桥的全部代码。

它的优点在于：桥的版本很容易与 Native 保持一致，Native 端不用对不同版本的 JSBridge 进行兼容；与此同时，它的缺点是：注入时机不确定，需要实现注入失败后重试的机制，保证注入的成功率，同时 JavaScript 端在调用接口时，需要优先判断 JSBridge 是否已经注入成功。


### 4.2 JavaScript 端引用

**直接与 JavaScript 一起执行。**

与由 Native 端注入正好相反，它的优点在于：JavaScript 端可以确定 JSBridge 的存在，直接调用即可；缺点是：如果桥的实现方式有更改，JSBridge 需要兼容多版本的 Native Bridge 或者 Native Bridge 兼容多版本的 JSBridge。





<br />
[文章来源于](https://juejin.cn/post/6844903585268891662)