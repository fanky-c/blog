---
title: javascript中toString()和valueOf()
date: 2019-05-14 18:00:58
tags:
   - toString()
   - valueOf()
---

### 介绍
1. 所有的对象原型都有这个2个方法。
2. Object.prototype会返回这2个方法。


### toString()

#### 介绍
1. toString() 返回对象字符串表示

#### 案例
```js
 3.toString(); // "3"
 '3'.toString(); // "3"
 true.toString(); // "true"
 {test: 3, app: "test"}.toString(); //"[object Object]"
 function(){console.log(3)}.toString(); // "function(){console.log(3)}"
 ['test', 'test2'].toString(); //"test,test2"  相当于调用Array.join(',')
 /\d/.toString();  // "/\d/"
```

### valueOf()

#### 介绍
1. valueOf() 如果对象存在任意原始值，它就默认将对象转换为表示它的原始值，如果对象是复合值，而且大多数对象无法真正表示为一个原始值，因此默认的valueOf( )方法简单地返回对象本身，而不是返回一个原始值。

#### 案例
```js
 3.valueOf(); // 3  返回原始值
 '3'.valueOf(); // "3"  返回原始值
 true.valueOf(); // true  返回原始值
 {test: 3, app: "test"}.valueOf(); //{test: 3, app: "test"} 返回自身
 function(){console.log(3)}.valueOf(); // function(){console.log(3)} 返回自身
 ['test', 'test2'].valueOf(); //["test,test2"]  返回自身
```


### 调用时机
1. 案例
```
//案例一
var a = '31';
console.log(+a);// 31

//案例二
var example = {test:'123'};
console.log(+example);// NaN

//案例三
console.log('test'+{}); //"test[object Object]"

//案例四
var example = {test:'1232'};
alert(example);// "[object Object]"
```
2. 结论： 
   1. 案例一二：一元加操作符在操作对象的时候，会先调用对象的valueOf方法来转换，最后再用Number( )方法转换
   2.  案例三：对象和字符串相加，肯定转换为字符串啊，所以调用了对象的toString方法，变为[object Object]了
   3. alert是调用toString()方法，和字符串和对象相加一样
   4. 


