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

### 3.1 JSBridge 的通信原理

#### 3.1.1 JavaScript 调用 Native

#### 3.1.2 Native 调用 JavaScript

### 3.2 JSBridge 接口实现

## 4. JSBridge 如何引用

### 4.1 由 Native 端进行注入

### 4.2 JavaScript 端引用




<br />
[文章来源于](https://juejin.cn/post/6844903585268891662)