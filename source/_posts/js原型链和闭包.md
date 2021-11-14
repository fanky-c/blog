---
title: js原型链和闭包
date: 2021-11-04 22:03:03
tags:
 - 原型链
 - 原型
---


### 原型和原型链

#### 原型概念
每一个js对象(null除外)在创建的时候就会与之关联另一个对象，这个对象就是我们所说的原型，每一个对象都会从原型继承属性和方法
#### 原型链概念
每个实例对象（object）都有一个私有属性（称之为 __proto__ ）指向它的构造函数的原型对象（prototype）。该原型对象也有一个自己的原型对象（__proto__），层层向上直到一个对象的原型对象为 null。根据定义，null 没有原型，并作为这个原型链中的最后一个环节。

#### 原理
##### 1. prototype
```js
function Person() {

}

Person.prototype.name = 'fanky';
let person1 = new Person();
let person2 = new Person();
console.log(person1.name) // fanky
console.log(person2.name) // fanky
```
> **函数的prototype属性指向一个对象， 这个对象正是调用该构造函数而创建的实例原型， 也就是person1 和 person2**

##### 2. \__proto__
```js
function Person() {

}
let person = new Person();
console.log(person.__proto__ === Person.prototype); // true
```
每一个js对象(除了 null )都具有的一个属性，叫\__proto__，这个属性会指向该对象的原型。
##### 3. constructor
```js
function Person() {

}
console.log(Person === Person.prototype.constructor); // true
```

> 每个原型都有一个 constructor 属性指向关联的构造函数

##### 4. 实例和原型
```js
// 构造函数
function Person() {

}

// 原型
Person.prototype.name = 'fanky';

// 实例
let person = new Person();
person.name = 'Daisy';
console.log(person.name) // Daisy

delete person.name;
console.log(person.name) // fanky
```
> 但是当我们删除了 person 的 name 属性时，读取 person.name，从 person 对象中找不到 name 属性就会从 person 的原型也就是 person.\__proto__ ，也就是 Person.prototype中查找，幸运的是我们找到了 name 属性，结果为 fanky

但是万一还没有找到呢？原型的原型又是什么呢？

##### 5. 原型的原型
```js
let obj = new Object();
obj.name = 'fanky'
console.log(obj.name); // fanky
```
> 其实原型对象就是通过 Object 构造函数生成的，结合之前所讲，实例的 __proto__ 指向构造函数的 prototype 


##### 6. 原型链
那 Object.prototype 的原型呢: 
```js
console.log(Object.prototype.__proto__ === null) // true
```
<img src="/img/prototype.png"  alt="原型链" height="auto"/>
***图中由相互关联的原型组成的链状结构就是原型链，也就是蓝色的这条线***

##### 7. hasOwnProperty
在原型链上查询属性比较耗时，对性能有影响，试图访问不存在的属性时会遍历整个原型链。
```js
const object1 = {};
object1.property1 = 42;
console.log(object1.hasOwnProperty('property1'));
// expected output: true

console.log(object1.hasOwnProperty('toString'));
// expected output: false
```

#### 作用
##### 继承属性
```js
let f = function () {
   this.a = 1;
   this.b = 2;
}
let o = new f(); // {a: 1, b: 2}
f.prototype.b = 3;
f.prototype.c = 4;
console.log(o.b); // 3
console.log(o.c); // 4
```
##### 继承方法
```js
var o = {
  a: 2,
  m: function(){
    return this.a + 1;
  }
};
console.log(o.m()); // 3

var p = Object.create(o); //p是一个继承自 o 的对象
p.a = 4;
console.log(p.m()); // 5
// 调用 p.m 时，'this' 指向了 p
// 又因为 p 继承了 o 的 m 函数
// 所以，此时的 'this.a' 即 p.a，就是 p 的自身属性 'a'
```

<br />
[文章来源于](https://github.com/mqyqingfeng/blog/issues/2)


