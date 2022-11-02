---
title: ES5和ES6以及ES7+
date: 2022-10-24 19:59:58
tags:
 - ES5
 - ES6
 - ES7
---

## 一、ES5
ECMAScript 5 于 2009 年 12 月发布的，内容主要包括严格模式、JSON 对象、新增 Object 接口、新增 Array 接口和 Function.prototype.bind 等。可以认为 ECMAScript 5 规范的推出在原来没有规范的 JavaScript 语法上添加了有限的限制标准，其中最重要的一条可能就是严格模式的推出。

### 1. 严格模式
* 未声明的全局变量赋值会抛出 ReferenceError，而不是默认去创建一个全局变量
* 默认支持的糟糕特性都会被禁用，使用 with、eval 等语句执行会抛出 SyntaxError
* 限制了函数中 arguments 使用，arguments.callee( ) 和 arguments.caller( ) 调用会提示 Uncaught ReferenceError
* 函数 arguments 严格定义为参数，不能与形参绑定

### 2. Object
* getPrototypeOf，返回一个对象的原型
* getOwnPropertyDescriptor，返回某个对象自有属性的属性描述符
* getOwnPropertyNames，返回一个数组，包括对象所有自由属性名称集合（包括不可枚举的属性）
* hasOwnProperty，给定属性存在于对象实例中，返回 true
* isPrototypeOf，在原型链中出现过的原型，返回 true
* create，创建一个拥有指定原型和若干指定属性的对象
* defineProperty，为对象定义一个新属性，或者修改已有的属性，并对属性重新设置 getter 和 setter，这里可以被用作数据绑定的对象劫持用途
* defineProperties，在一个对象上添加或修改一个或者多个自有属性，与 defineProperty 类
* seal，锁定对象，阻止修改现有属性的特性，并阻止添加新属性，但是可以修改已有属性的值
* freeze，冻结对象，阻止对对象的一切操作和更改，冻结对象将变成只读
* preventExtensions，让一个对象变得不可以扩展，也就是不能再添加新的属性
* keys，返回一个由给定对象的所有可以枚举自身属性的属性名组成的数组

## 二、ES6
ECMAScript 6 于 2015 年 6 月 17 日正式发布，也被命名为 ECMAScript 2015。ES6 借鉴了 ES5 和其他语言的特性，并在此基础上进行了补充和增强，使得 JavaScript 语言规范更加高效、严谨、完善。

例如，字符串模板、集合、箭头函数、Promise、for...of 等均是借鉴其他语言的优秀特性而增加的功能点。class 类和 import/export 模块规范则可以认为是对原有标准缺失的补充。迭代器、生成器、解构赋值、函数参数等都可以认为是对原有标准特性的增强。


## 三、ES7+
2016 年，ECMAScript 7（或称为 ECMAScript 2016） 正式发布，整体来说，在 ES6 的版本逐渐稳定后，后期版本添加的主要内容已经不是太多了。



<br/>
[文章来源](https://zhuanlan.zhihu.com/p/531959101)