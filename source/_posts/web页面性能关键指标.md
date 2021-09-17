---
title: web页面性能关键指标
date: 2021-08-23 20:21:46
tags:
 - web性能指标
 - fp
 - fcp
 - fmp
 - tti
---

### 核心指标
#### 1.FP && FCP
白屏时间：页面开始有内容的时间，在没有内容之前是白屏

#### 2.FSP
首屏时间： 可视区域内容已完全呈现的时间

#### 3.FCI
可交互时间：用户第一次可以与页面交互的时间
#### 4.TTl
可流畅交互时间：用户第一次可以持续与页面交互的时间

### 页面呈现过程相关指标
#### 1. 文档加载
##### time to first byte (TTFB)
TTFB: 浏览器从请求页面开始到接收第一字节的时间，这个时间段内包括 DNS 查找、TCP 连接和 SSL 连接。
##### domContentLoaded (DCL)
DCL: HTML 文档被完全加载和解析完成之后, 无需等待样式表、图像和子框架加载完成
##### load (L)
L:  页面所有资源都加载完毕后（比如图片，CSS）

#### 2. 内容呈现
##### first paint (FP)
FP: 从开始加载到浏览器首次绘制像素到屏幕上的时间，也就是页面在屏幕上首次发生视觉变化的时间; **这是开发人员关心页面加载的第一个关键时刻 --- 当浏览器开始呈现页面时**
##### first contentful paint (FCP)
FCP：浏览器首次绘制来自 DOM 的内容的时间，内容必须是文本、图片（包含背景图）、非白色的 canvas 或 SVG。 这是用户第一次开始看到页面内容，但仅仅有内容，并不意味着它是有用的内容（例如 Header、导航栏等），也不意味着有用户要消费的内容

##### largest meaningful paint (LMP)

##### largest contentful paint (LCP)
LCP: 可视区域中最大的内容元素呈现到屏幕上的时间，用以估算页面的主要内容对用户可见时间
##### first screen paint (FSP)
FSP: 页面从开始加载到首屏内容全部绘制完成的时间，用户可以看到首屏的全部内容
#### 3. 交互响应
##### first cpu idle (FCI)
FCI: 用户第一次可以与页面交互的时间
##### time to interactive (TTI)
TTI: 用户第一次可以持续与页面交互的时间


<br/>
[文章参考1](https://zhuanlan.zhihu.com/p/98880815)
[文章参考2](https://juejin.cn/post/6844904153869713416)