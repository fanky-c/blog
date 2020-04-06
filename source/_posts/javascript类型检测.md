---
title: javascript类型检测
date: 2019-05-22 16:21:03
tags:
    - js类型检测
    - typeof
    - instanceof
---

#### typeof

1. 基本类型（number、string、undefined、boolean）
```js
typeof 1 //
typeof '1' //
typeof true //  
typeof x // undefined 
if(typeof x == 'undefined') // true
if(x)  //直接报错
```
2. function(函数)
```js
typeof function(){}
typeof class C {}
typeof Math.sin 
```

3. object
```js
typeof null // 当时设计的原因
typeof [] //
typeof {} //
```

#### instanceof
1. 判断某个对象的是另外一个对象实例
```js

// A 的 __proto__ 指向 B 的 prototype 时，就认为 A 就是 B 的实例
[] instanceof Array; //true
({}) instanceof Object;//true
new Date() instanceof Date;//true

//无法判断继承
function Parent(){};
function Child(){};
function Other(){};
Child.prototype = new Parent();
let child = new Child();
child instanceof Child; // true
child instanceof Parent; // true
child instanceof Object; // true
child instanceof Other;  //false

//Child.prototype.__proto__ = Parent.prototype
//Parent.prototype.__proto__ = Object.prototype
//Object.prototype.__proto__ = null
```


#### Object.prototype.toString.call(obj).slice(8, -1)