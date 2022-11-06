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


### 3. Array
* Array.isArray，确定某个值到底是不是数组，不管它在哪个全局执行环境中创建
* push，接收任意数量的参数，把它们逐个添加到数组末尾，并返回修改后数组的长度
* pop，在数组尾部移除最后一项，减少数组的 length 值，然后返回移除的项
* shift，移除数组中的第一项并返回该项，同时将数组长度减 1
* unshift，在数组前端添加任意个项，并返回新数组的长度
* reverse，反转数组项的顺序，返回排序后的数组
* sort，默认情况下，按升序排序数组项，返回排序后的数组
* concat，创建当前数组的一个副本，将接收到的参数添加到这个副本的末尾，返回新构建的数组
* join，用指定的字符拼接数组，返回拼接好的字符串
* slice，基于当前数组中的一个或多个创建一个新数组，不影响原始数组
* splice，根据传入参数不同，可以删除、插入、替换数组元素，返回一个数组，该数组中包含从原始数组中删除的项，如果没有删除任何项，则返回空数组
* indexOf，返回根据给定元素找到的第一个索引值，如果不存在则返回 -1
* lastIndexOf，返回指定元素在数组中的最后一个索引值，如果不存在则返回 -1
* every ，对数组中的每一项运行给定函数 ，如果该函数对每一项都返回 true，则返回 true
* filter，对数组中的每一项运行给定函数 ，返回该函数会返回 true 的项组成的数组
* forEach，对数组中的每一项运行给定函数，没有返回值
* map，对数组中的每一项运行给定函数，返回每次函数调用的结果组成的数组
* some，对数组中的每一项运行给定函数，如果函数对任一项返回 true，则返回 true
* reduce，接收一个函数作为累加器，数组中的每个值（从左到右）开始缩减，最终缩减为一个值

### 4. String
* charAt，访问字符串中特定字符，返回给定位置的字符
* charCodeAt，访问字符串中特定字符，返回给定位置的字符的字符编码
* concat，将一个或多个字符串拼接起来，返回拼接得到的新字符串
* match，查找匹配的字符串，返回一个数组
* search，查找匹配的字符串，返回匹配项的索引，没有找到，返回 -1
* replace，替换字符串
* split，基于指定的分隔符将一个字符串分割成多个字符串，将结果放在一个数组中，并返回
* trim，创建一个字符串的副本，删除前置及后缀的所有空格，然后返回结果
* localeCompare，比较两个字符串
* slice，返回被操作字符串的一个子字符串
* substr，返回被操作字符串的一个子字符串
* substring，返回被操作字符串的一个子字符串


### 5. Function
* arguments.callee，该属性是一个指针，指向拥有这个 arguments 对象的函数
* arguments.caller，该属性保存着调用当前函数的函数的引用
* apply，在特定的作用域中调用函数，第一个参数是在其中运行函数的作用域，第二个是参数数组
* call，在特定的作用域中调用函数，第一个参数是在其中运行函数的作用域，其余参数直接传递给函数
* bind，创建一个函数的实例，其 this 值会被绑定到传给 bind 函数的值


### 6. JSON
* JSON.parse，解析字符串为 JSON 对象
* JSON.stringify，解析 JSON 对象为字符串
* JSON.valueOf，获取某个JSON 对象中的值
* JSON.toString，被调用时，会调用 Object 原型上的 toString 方法，会获的 JSON 对象的值并转为字符串，如果没有具体的值，则返回原型数组


### 7. 其他
* Date.now，返回当前时间戳
* Number.prototype.toFixed，按照指定的小数位返回数值的字符串表示

## 二、ES6
ECMAScript 6 于 2015 年 6 月 17 日正式发布，也被命名为 ECMAScript 2015。ES6 借鉴了 ES5 和其他语言的特性，并在此基础上进行了补充和增强，使得 JavaScript 语言规范更加高效、严谨、完善。

例如，字符串模板、集合、箭头函数、Promise、for...of 等均是借鉴其他语言的优秀特性而增加的功能点。class 类和 import/export 模块规范则可以认为是对原有标准缺失的补充。迭代器、生成器、解构赋值、函数参数等都可以认为是对原有标准特性的增强。


### 1. let、const
对于跨级作用域变量声明关键字 let、const，在使用时有几个需要注意的地方:
 1. let 和 const 都只能作为块级作用域变量的声明，且只能在块作用域内生效，块内声明的变量无法在块级外层 引用
 2. 使用 const 声明的变量必须进行初始化赋值，而且一旦赋值就不能再进行二次修改赋值
 3. 使用 let 和 const 在全局作用域下声明的变量不会作为属性添加到全局作用域对象里面

let 和 const 使用场景的区别：
 1. 模块内不变的引用和常量，一般使用 const 定义
 2. 可变的变量或引用使用 let 声明
 3. var 仅用于声明函数整个作用域内需要使用的变量

### 2. 字符串模板
如果没有字符串模板，我们依然需要像以前一样借助“字符串+操作符” 拼接或数组 join 方法来连接多个字符串变量。
**需要注意的是，字符串模板不会压缩内部的换行与空格，而是按照原有的格式输出，只将变量内容填充替换掉。**

### 3. 解构赋值
解构赋值主要分为数组解构和对象解构：
```js
let [a, b, c] = [1, 2]
let {one, two, three} = {two:2, three: 3, one: 1}

[a, b] = [b, a] // 交换a、b 变量的值
console.log(c) // undefined
```
数组解构是严格按照数组下标依次对应顺序赋值的，如果复制的常量个数不够，则对应下标的变量默认为 undefined；如果常量个数超出，则多余的会被舍弃，所以顺序很重要。

对象的解构赋值是根据对象引用的键名来赋值的，可以无视顺序。

### 4. 数组的新特性
```js
const arr = [1, 2, 3]
const newArr = [...arr]  //[1, 2, 3]
```
注意：这里 **... 进行的数组复制是浅拷贝**。

**ES6 数组的新增方法：**
1. Array.from，用于将类数组对象（包括 [array-like object] 和可遍历对象）转化为真正的数组
2. Array.of，可以将传入的一组参数值转换为数组
3. Array.prototype.copyWithin，可以在当前数组内部将指定位置的数组项复制到其他位置，然后返回当前数组，使用 copyWithin 方法会修改当前数组
4. Array.prototype.fill，使用给定值，填充一个数组，会改变原来的数组
5. Array.prototype.find，用于找出第一个符合条件的数组元素，有点类似于 filter
6. Array.prototype.findIndex，用来返回某个特定数组元素在数组中的位置
7. Array.prototype.entries，对数组中键值对进行遍历
8. Array.prototype.keys，对数组键名进行遍历
9. Array.prototype.values，对数组键值进行遍历
10. for...of 循环进行遍历
11. Array.prototype[Symbol.iterator]，用来获取遍历数组对象的迭代器


### 5. 增强对象

## 三、ES7+
2016 年，ECMAScript 7（或称为 ECMAScript 2016） 正式发布，整体来说，在 ES6 的版本逐渐稳定后，后期版本添加的主要内容已经不是太多了。



<br/>
[文章来源](https://zhuanlan.zhihu.com/p/531959101)