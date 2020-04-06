---
title: javascript内存泄漏
date: 2020-03-29 19:34:27
tags:
    - 内存泄露场景
    - 内存泄露检测
---

### 内存的介绍
* 内存的生命周期： 申请内存 --> 使用内存 -->  释放内存


### 什么是内存泄露
* 内存泄露就是不在被应用需要的内存，由于某种原因，没有归还给操作系统。

### 垃圾回收算法
#### 引用计数法： 
1. 如果该对象没有引用就被回收。 
2. 缺点就是在循环引用情况下无法回收。
```js
  function  test（）{
       var  obj1  = {};
       var  obj2  = {};
       obj1.x  =  obj2 ; // obj1引用obj2
       obj2.x  =  obj1 ; // obj2引用obj1
   }
```
####  标记清除法
  1. 创建垃圾回收器对象， 浏览器宿主对象window， 检测它所有的子对象是否存在。
  2. 所有子对象递归检测，如果从window开始能到达标记为激活，就不视为垃圾
  3. 所有子对象递归检测，如果从window开始不能到达就视为垃圾，归还给操作系统


### 内存泄露的场景
#### 全局变量
```js
//案例
function test(){
    name = 'hello'; //name将变成全局变量，泄露到全局
}

//原因
全局变量是根据定义无法被垃圾回收机制。如果是临时需要存放大量数据的全局变量，
必须指定为null或者使用完重新分配内存

//解决方案
使用js严格模式


```

#### 被遗忘的定时器和回调函数
```js
//案例
setInterval(function(){
    var dom = document.getElementById('DOM');
    if(dom){
        dom.innerText = 'hello world';
    }
    //1， dom 无法被回收  2， 整个定时器一直在运行没有清除
}, 1000)


//解决方法
在定时器完成工作时候，手动清楚定时器
```

#### DOM外引用
```js
//案例
var refA = document.getElementById('test');
document.body.removeChild(refA); // dom删除了
console.log(refA, "refA");  // 但是还存在引用 能console出整个div 没有被回收

//原因: 
保留了DOM节点的引用,导致GC没有回收

//解决办法：
refA = null;
```


#### 闭包
```js
//闭包本身不会造成内存泄露，使用不当才会导致
//案例


//解决方法
在退出函数之前，将不使用的局部变量全部删除
```
### 内存泄露检测
#### 浏览器
1. chrome开发者工具memory 
2. 点击take snapshot进行对比

#### node环境
1. process.memoryUsage方法