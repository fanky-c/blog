---
title: js中的call、apply、bind的作用、应用场景以及区别
date: 2019-12-22 20:19:52
tags:
   - call
   - apply
   - bind
   - Array.prototype.slice.call
---

### call、apply、bind的作用与应用

#### apply
##### 介绍
* apply()方法接收两个参数：一个是在其中运行函数的作用域，另一个是参数数组。

##### 例子
```js
function sum(num1, num2){
    return num1 + num2;
}

function callSum1(num1, num2){
    return sum.apply(this, arguments);        // this== window   传入arguments对象
}

function callSum2(num1, num2){
    return sum.apply(this, [num1, num2]);    // this == window    传入数组
}

alert(callSum1(10,10));   //20
alert(callSum2(10, 10)); //20

//第一个参数（当前例子中this）出现的四种情况
（1） 不传，或者传null,undefined， 函数中的this指向window对象
（2） 传递另一个函数的函数名，函数中的this指向这个函数的引用
（3） 传递字符串、数值或布尔类型等基础类型，函数中的this指向其对应的包装对象，如 String、Number、Boolean
（4） 传递一个对象，函数中的this指向这个对象
```

#### call

##### 介绍
* call()方法接收多个参数：第一个是在其中运行函数的作用域，其他都是参数都直接传递给函数。

##### 例子
```js

window.color = "red";
var o = { color: "blue" };

function sayColor(){
    alert(this.color);
}

sayColor();                //red
//sayColor里面的this指向了this == window
sayColor.call(this);       //red
sayColor.call(window);     //red
//sayColor里面的this指向了o
sayColor.call(o);          // blue

```

##### [].slice.call(arguments) 等价于 Array.prototype.slice.call(arguments)
```js
// 类数组: 可以通过索引访问元素，并且拥有 length 属性; 没有数组的其他方法，例如 push ， forEach ， indexOf 等.
const likeArr =  {
    0: "test",
    1: "测试",
    length: 2
}

// 转换
[].slice.call(likeArr);

//
var args = [];
for (var i = 0; i < likeArr.length; i++) {
    args.push(likeArr[i]);
}
console.log(args); // ["test", "测试"];

/**
 * 原理
 */
// [].slice()
console.log([1,2,3].slice(0,1)) // [1]
console.log([1,2,3].slice()) // [1,2,3]

// Function.call()
function func(name, price) {
  this.name = name;
  this.price = price;
}
var food = {name:"apple",price:10};
func.call(food,"orange",15);
console.log(food); // {name: "orange", price: 15}


//为了提高性能，减少一层对原型链的追溯，一般我们会采用以下的写法
// []继承Array.prototype
[].slice.call(arguments) --> Array.prototype.slice.call(arguments)
```


##### Array.prototype.map.call()

作用：让**类数组对象（如 arguments、NodeList）**使用 map() 方法。

```js
function example() {
  const doubled = Array.prototype.map.call(arguments, (num) => num * 2);
  console.log(doubled); // [2, 4, 6]
}

example(1, 2, 3);
```

##### Object.prototype.toString.call()

作用：准确判断数据类型，比 typeof 更可靠

运行机制：
每个 JavaScript 内置对象（如 Array、Function、Date 等）都有一个内部的 [[Class]] 属性，该属性通过 Object.prototype.toString.call(value) 返回 [object Type] 格式的字符串。

```js
console.log(Object.prototype.toString.call([])); // "[object Array]"
console.log(Object.prototype.toString.call({})); // "[object Object]"
console.log(Object.prototype.toString.call(null)); // "[object Null]"
console.log(Object.prototype.toString.call(undefined)); // "[object Undefined]"
console.log(Object.prototype.toString.call(/\d/)); // "[object RegExp]"
console.log(Object.prototype.toString.call(new Date())); // "[object Date]"
```


#### bind
##### 介绍
* 这个方法会创建一个函数的实例，其this值会被绑定到传给bind()函数的值。 便于后期调用， 不同于call、apply立即调用。

##### 例子
```js
var bar=function(){
  console.log(this.x);
}
var foo={
     x:3
}
bar();
bar.bind(foo)();
 /*或*/
var func=bar.bind(foo);
func();

输出：
undefined
3
```