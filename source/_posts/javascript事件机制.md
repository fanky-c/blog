---
title: javascript事件机制
date: 2019-05-02 16:07:05
tags: 
    - js事件机制
---

### js事件机制三个阶段
#### 事件捕获阶段
```js
   document.getElementById("button").addEventListener("click",function(event){
            console.log("button");
            event.stopPropagation(); //阻止事件捕获
            event.stopImmediatePropagation(); //阻止其他事件（包括默认事件）
        },true);
```

#### 处于目标阶段
#### 事件冒泡阶段
1. 现代浏览器标准事件
```js
   document.getElementById("button").addEventListener("click",function(event){
            console.log("button");
            event.stopPropagation(); //阻止冒泡
        },false);
```