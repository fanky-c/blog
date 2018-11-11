---
title: es6扩展运算符，三个点...
date: 2018-11-11 19:01:24
tags: 
   - es6
   - ...
---

### 含义

扩展运算符用...表示，它好比rest参数的逆运算，讲一个数组转为用逗号分隔的参数序列。
```
console.log(...[1, 2, 3])
// 1 2 3
console.log(1, ...[2, 3, 4], 5)
// 1 2 3 4 5
[...document.querySelectorAll('div')]
//[<div>,<div>]
```
### 用途

#### 1，函数调参,将一个数组，变为参数序列。
```
function f(v, w, x, y, z) { }
var args = [0, 1];
f(-1, ...args, 2, ...[3]);
```

#### 2，替换数组和apply相关方法
* 由于扩展运算符可以展开数组，所以不再需要apply方法，将数组转为函数的参数了。
* 取代apply用法一：
```

// ES5 的写法
function f(x, y, z) {
// ...
}
var args = [0, 1, 2];
f.apply(null, args);
// ES6 的写法
function f(x, y, z) {
// ...
}
var args = [0, 1, 2];
f(...args);
```
* 取代apply，应用Math.max方法用法二：
```
// ES5 的写法
Math.max.apply(null, [14, 3, 77])
// ES6 的写法
Math.max(...[14, 3, 77])
//  等同于
Math.max(14, 3, 77);
```
* 数组合并和添加尾部三：
```
// ES5
[1, 2].concat(more)
// ES6
[1, 2, ...more]
var arr1 = ['a', 'b'];
var arr2 = ['c'];
var arr3 = ['d', 'e'];
// ES5 的合并数组
arr1.concat(arr2, arr3);
// [ 'a', 'b', 'c', 'd', 'e' ]
// ES6 的合并数组
[...arr1, ...arr2, ...arr3]
// [ 'a', 'b', 'c', 'd', 'e' ]
// ES5 的写法
var arr1 = [0, 1, 2];
var arr2 = [3, 4, 5];
Array.prototype.push.apply(arr1, arr2);
// ES6 的写法
var arr1 = [0, 1, 2];
var arr2 = [3, 4, 5];
arr1.push(...arr2);
```

#### 3，与解构赋值结合（只能放在最后）
```
const [first, ...rest] = [1, 2, 3, 4, 5];
first // 1
rest // [2, 3, 4, 5]
const [...butLast, last] = [1, 2, 3, 4, 5];
//  报错
const [first, ...middle, last] = [1, 2, 3, 4, 5];
//  报错
```

#### 4，将字符串转为真正的数组
```
[...'hello']
// [ "h", "e", "l", "l", "o" ]
```

#### 5,实现了 Iterator 接口的对象
```
var nodeList = document.querySelectorAll('div');
var array = [...nodeList];
```
任何 Iterator 接口的对象，都可以用扩展运算符转为真正的数组。


```
let arrayLike = {
'0': 'a',
'1': 'b',
'2': 'c',
length: 3
};
// TypeError: Cannot spread non-iterable object.
let arr = [...arrayLike];
```
对于那些没有部署 Iterator 接口的类似数组的对象，扩展运算符就无法将其转为真正的数组，但是可以使用Array.from方法将arrayLike转为真正的数组。


[参考](https://blog.csdn.net/qq_30100043/article/details/53391308)