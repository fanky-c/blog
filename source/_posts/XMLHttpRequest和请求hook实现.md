---
title: XMLHttpRequest和请求hook实现
date: 2022-04-29 17:24:35
tags:
 - xhr
 - 请求hook
---

## xhr
### 构造方法
该构造函数用于初始化一个 XMLHttpRequest 实例对象。在调用下列任何其他方法之前，必须先调用该构造函数，或通过其他方式，得到一个实例对象。
```js
const request = new XMLHttpRequest();
```

### 标准属性
1. XMLHttpRequest.onreadystatechange：当 readyState 属性发生变化时
2. XMLHttpRequest.readyState

| 值 | 状态 | 描述 |
|---| --- | ---|
| 0 | UNSENT| 代理被创建，但尚未调用 open() 方法|
| 1 | OPENED | open() 方法已经被调用|
| 2 | HEADERS_RECEIVED | send() 方法已经被调用，并且头部和状态已经可获得|
| 3 | LOADING | 下载中； responseText 属性已经包含部分数据。|
| 4 | DONE | 下载操作已完成 |

3. XMLHttpRequest.response:返回一个 ArrayBuffer、Blob、Document，或 DOMString，具体是哪种类型取决于 XMLHttpRequest.responseType 的值。其中包含整个响应实体（response entity body）
4. XMLHttpRequest.responseText:返回一个 DOMString，该 DOMString 包含对请求的响应，如果请求未成功或尚未发送，则返回 null
5. XMLHttpRequest.responseType: 一个用于定义响应类型的枚举值(arraybuffer、blob、document、json、text)
6. XMLHttpRequest.status: 返回一个无符号短整型数字，代表请求的响应状态(在请求完成前/如果 XMLHttpRequest出错，status的值都为0)
7. XMLHttpRequest.timeout： 一个无符号长整型数字，表示该请求的最大请求时间（毫秒），若超出该时间，请求会自动终止。
8. XMLHttpRequestEventTarget.ontimeout:当请求超时调用的
9. XMLHttpRequest.withCredentials:一个布尔值，用来指定跨域 Access-Control 请求是否应当带有授权信息，如 cookie 或授权 header 头


### 标准方法
1. XMLHttpRequest.abort()：如果请求已被发出，则立刻中止请求。
2. XMLHttpRequest.getAllResponseHeaders(): 以字符串的形式返回所有用 CRLF(回车符（CR）和换行符（LF）是文本文件用于标记换行的控制字符或字节码（bytecode）) 分隔的响应头，如果没有收到响应，则返回 null。
3. XMLHttpRequest.getResponseHeader(): 返回包含指定响应头的字符串，如果响应尚未收到或响应中不存在该报头，则返回 null
4. XMLHttpRequest.open(): 初始化一个请求。该方法只能在 JavaScript 代码中使用，若要在 native code 中初始化请求，请使用 openRequest()。
5. XMLHttpRequest.overrideMimeType(): 覆写由服务器返回的 MIME 类型。
6. XMLHttpRequest.send(): 发送请求。如果请求是异步的（默认），那么该方法将在请求发送后立即返回。
7. XMLHttpRequest.setRequestHeader(): 设置 HTTP 请求头的值。必须在 open() 之后、send() 之前调用 setRequestHeader() 方法

### 标准事件
1. abort: 当 request 被停止时触发，例如当程序调用 XMLHttpRequest.abort() 时。也可以使用 onabort 属性。
2. error: 当 request 遭遇错误时触发。 也可以使用 onerror 属性
3. load: XMLHttpRequest请求成功完成时触发。 也可以使用 onload 属性
4. loadend: 当请求结束时触发, 无论请求成功 ( load) 还是失败 (abort 或 error)。也可以使用 onloadend 属性。
5. loadstart: 接收到响应数据时触发。也可以使用 onloadstart 属性。
6. progress: 当请求接收到更多数据时，周期性地触发。也可以使用 onprogress 属性。
7. timeout: 在预设时间内没有接收到响应时触发。 也可以使用 ontimeout 属性


### 代码封装
```js
function (opts) { 
const xhr = new XMLHttpRequest();
xhr.open(opts.type || 'GET', opts.url, true);
xhr.success = opts.success;
xhr.fail = opts.fail;
xhr.withCredentials = opts.withCredentials;
xhr.onreadystatechange = function () {
   if (xhr.readyState === 4) {
       const status = xhr.status;
       if (status >= 200) {
           opts.success && opts.success(xhr.responseText);
       } else {
           opts.fail && opts.fail(`Request failed, status: ${status}, responseText: ${xhr.responseText}`);
       }
   }
}
if (opts.type === 'POST') {
  if (opts.headers) {
      for (const key in opts.headers) {
          xhr.setRequestHeader(key, opts.headers[key]);
      }
  }
  xhr.send(opts.data);
} else {
  xhr.send();
}
return xhr;
}
```

## hook

<br >
[来源于](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest)