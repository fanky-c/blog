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
在 ES6 中，对象的使用变得更加方便了，可以在定义对象时通过属性简写、遍历作为属性名或省略对象函数属性的书写等方式来提高编码的效率：
```js
const name = 'xixi'
const people = {
    name,
    [getKey('family')]: 'zhang',
    sayHi() {
        console.log(`Hello ${this.name} ${this.family}`)
    }
}

people.sayHi() // Hello xixi zhang

function getKey(key) {
    return key
}
```

### 6. 函数参数
ES6 对函数参数进行了新的设计，主要添加了默认参数、不定参数和扩展参数：
```js
// 默认参数
function sayHi(name = 'xixi') {
    console.log(`Hello ${name}`)
}
sayHi() // Hello xixi

// 不定参数
function sayHi(...name) {
    console.log(name.reduce((a, b) => `Hello ${a} ${b}`))
}
sayHi('xixi', 'zhang') // Hello xixi zhang

// 扩展参数
let name = ['xixi', 'zhang']
function sayHi(name1, name2) {
    console.log(`Hello ${name1} ${name2}`)
}
sayHi(...name) // Hello xixi zhang
``` 
**不定参数和扩展参数可以认为恰好是相反的两个模式，不定参数是使用数组来表示多个参数，扩展参数则是将多个参数映射到一个数组。**

需要注意：不定参数的 ... 和数组复制的 ... 是有区别的，不定参数可以使用函数的形参来表示所有的参数组成的列表。以前的 arguments 变量也有类似的作用，但是 arguments 不是真正的数组，除了存放参数的列表外，arguments 还有 length 属性，严格来说 arguments 是一个类数组对象，而不定参数则是一个完全的数组，这也是不定参数相对于 arguments 的优势，更加方便我们使用，所以建议使用不定参数来代替 arguments。

### 6. 箭头函数
```js
// 箭头函数
[1, 2, 3].forEach(x => x*x)

(() => {
    console.log('Hello xixi')
})()
```
箭头函数没有完整的执行上下文，因此其 this 和外层的 this 相同，可以理解为它的执行上下文只有变量对象和作用域链，没有 this 值。

在 JavaScript 中，代码的执行上下文由变量对象、作用域链和 this 值组成，但箭头函数与外层执行上下文共享 this 值，如果需要创建具有独立上下文的函数，就不要使用箭头函数。

### 7. class
基本上，ES6 中的 class 可以看做是一个语法糖，它的绝大部分功能，ES5 都可以做到，新的 class 写法只是让对象原型的写法更加清晰，更像面向对象变成的语法而已。

```js
class Animal {
    constructor() {

    }
}

class People extends Animal {
    constructor(contents = {}) {
        super()
        this.name = contents.name
        this.family = contents.family
    }
    sayHi() {
        console.log(`Hello ${this.name} ${this.family}`)
    }
}

let me = new People({
    name: 'xixi',
    family: 'zhang'
})

me.sayHi() //Hello xixi zhang
```

### 9. 模块 module
ES6 引入了模块引用规范，这也是之前语言标准上没有的，这样现有的 JavaScript 模块化规范又多了一种选择：import/export

```js
import { sayHi } from './people'
export sayHi
```

### 10. 集合类型 Map + Set + WeakMap + WeakSet
#### Set
Set 本身是一个构造函数 ，用来生成 Set 数据结构，Set 类似于数组（但它不是数组），Set 的成员的值都是唯一的，没有重复的值，也常用它来去重（不可以传递对象）。像 Set 加入值的时候，不会发生类型转换，所以 5 和 “5” 是两个不同的值。
```js
const arr = new Set([1, 2, 3, 4, 5, 5, 5, 5])

console.log(arr)  //[1, 2, 3, 4, 5]
console.log(arr.size)  //5 
```

#### WeakSet
WeakSet 结构与 Set 类似，也是不重复的值的集合，但是，它与 Set 有两个区别：
1. 第一，WeakSet 的成员只能是对象，而不能是其他类型的值。
2. 第二，WeakSet 中的对象都是弱引用，即垃圾回收机制 不考虑 WeakSet 对该对象的引用，也就是说，如果其他对象都不在引用该对象，那么垃圾回收机制会自动回收该对象所占用的内存，不考虑该对象是否还存在于 WeakSet 中。因此 ES6 规定 WeakSet 不可遍历。

#### Map
传统的 JavaScript 的对象（Object），本质上是键值对的集合（Hash 结构），但是只能用字符串当做键，这给它的使用带来了很大的限制，而 ES6 提供了 Map 数据结构，它类似于对象，也是键值对的集合，但是“键” 的范围不限于字符串，各种类型的值（包括对象）都可以当做键。也就是说，Object 结构提供了“字符串-值”的对应，Map 结构提供了“值-值”的对应，是一种更完善的 Hash 结构实现。Map 的键实际上是跟内存地址绑定的，只要内存地址不一样，就视为两个键。

操作方法：
1. set(key, value)，设置 key 所对应的键值，返回整个 Map 结构 ，如果 key 已经有值，则键值会被更新，否则就生成该键
2. get(key)，读取 key 对应的键值，如果在好不到 key，则返回 undefined

遍历方法：
1. keys()，返回键名的遍历器
2. values()，返回键值的遍历器
3. entries()，返回所有成员的遍历器
4. forEach()，遍历 Map 的所有成员

#### WeakMap
WeakMap 结构与 Map 结构类似，也用于生成键值对的集合，但 WeakMap 与 Map 有两个个区别：

1. WeakMap 只接受对象作为键名（null 除外），不接受其他类型的值作为键名
2. WeakMap 的键名所指向的对象不计入垃圾回收机制。它的键名所引用的对象都是弱引用，即垃圾回收机制不将该引用考虑在内，因此，只要所引用的对象的其他引用被清除了，垃圾回收机制就会释放该对象所占用的内存。也就是说， 一旦不再需要，WeakMap 里面的键名对象和所对应的键值对会自动消失，不用手动删除引用。基本上，如果要想对象中添加数据又不想干扰垃圾回收机制，便可以使用 WeakMap。一个典型的应用场景是，在网页的 DOM 元素上添加数据时就可以使用 WeakMap 结构，当该 DOM 元素被清除，其对应的 WeakMap 记录就会自动被移除。


注意：WeakMap 的专用场景就是它的键所对应的对象可能会在将来消失，WeakMap 结构有助于防止内存泄露。但是，WeakMap 弱引用的只是键名而不是键值，键值依然是正常引用的。

### 11. Symbol 类型
ES6 引入了一种新的原始数据类型 Symbol，表示独一无二的值，它是 JavaScript 语言的第 7 种数据类型，前 6 种分别是：Undefined、Null、Boolean、String、Number 和 Object。

Symbol 值通过 Symbol 函数生成，一般作为属性键值，并且能避免对象属性键的命名冲突。也就是说，对象的属性名现在可以有两种类型：一种是原来就有的字符串，另一种就是新增的 Symbol 类型。只要属性名属于 Symbol 类型，就是独一无二的，可以保证不会与其他属性名产生冲突。
```js
let s = Symbol('foo')
typeof s  //"symbol"
```

Symbol 函数前不能使用 new 命令，否则会报错，这是因为生产的 Symbol 是一个原始类型的值，不是对象。也就是说，由于 Symbol 值不是对象，所以不能添加属性。基本上，它是一种类似于字符串的数据类型。

**Symbol 函数的参数只表示对当前 Symbol 值的描述，因此相同参数的 Symbol 函数的返回值是不相等的。**

Symbol 值作为对象属性名时不能使用点运算符：
```js
let s = Symbol()

let obj = {
    [s]: function() {
        console.log('Hello')
    }
}

obj[s]()  // 'Hello'
```

### 12. Promise
Promise 代表一个异步操作的执行返回状态，这个执行返回状态在 Promise 对象创建时是未知的，它允许为异步操作的成功或失败指定处理方法。

####  Promise 的状态有三种：
1. Fulfilled，表示 Promise 执行成功
2. Rejected，表示 Promise 执行失败
3. Pending，表示 Promise 正在执行中

####  Promise 对象有两个特点：
1. 对象的状态不受外界影响
2. 一旦状态改变就不会再变，任何时候都可以得到这个结果

#### 缺点：
1. 无法取消 Promise，一旦新建它就会立即执行，无法中途取消
2. 如果不设置回调函数，Promise 内部抛出的错误不会反应到外部
3. 当处于 Pending 状态时，无法得知目前进展到哪一个阶段（刚开始还是即将完成）
   

#### 用法：
```js
var promise = new Promise(function(resolve, reject) {
    // ... some code

    if ( /*异步操作成功*/ ) {
        resolve(value)
    } else {
        reject(error)
    }
})

promise.then(function(value) {
    // success
}, function(error) {
    // failure
})
```

#### 实例方法：
1. Promise.prototype.then()，为 Promise 实例添加状态改变时的回调函数，返回一个新的 Promise 实例
2. Promise.prototype.catch()，用于指定发生错误时的回调函数，返回一个新的 Promise 实例
3. Promise.prototype.done()，总是处于回调链的尾端，保证捕捉到任何可能出现的错误
4. Promise.prototype.finally()，用于指定不管 Promise 对象最后状态如何都会执行的操作。

finally 与 done 的最大区别在于：finally 接受一个普通的回调函数作为参数，该函数不管怎样都必须执行。

####  Promise 对象方法：
1. Promise.all()，将多个 Promise 实例包装成一个新的 Promise 实例。
   1. 第一，只有 p1，p2，p3 的状态都编程 Fulfilled，p 的状态才会变成 Fulfilled，此时p1，p2，p3 的返回值组成一个数组，传递给 p 的回调函数。
   2. 第二，只要 p1，p2，p3 中有一个被 Rejected，p 的状态就变成 Rejected，此时第一个被 Rejected 的实例的返回值会传递给 p 的回调函数
2. Promise.race()，将多个 Promise 实例包装成一个新的 Promise 实例。
   1. 只要 p1，p2，p3 中有一个实例率先改变状态，p 的状态就跟着改变，那个率先改变的 Promise 实例的返回值就传递给 p 的回调函数。
3. Promise.resolve()，将现有对象转为 Promise 对象，状态为 Resolved
4. Promise.reject()，将现有对象转为 Promise 对象，状态为 Rejected


### 13. Proxy
Proxy 用于修改某些操作的默认行为，可以用来拦截某个对象的属性访问方法，然后重载对象的 “ . ” 运算符。

Proxy 可以理解成在目标对象前架设一个“拦截”层，外界对该对象的访问都必须先通过这层拦截，因此提供了一种机制可以对外界的访问进行过滤和改写。
```js
let object = new Proxy({}, {
    get: function(target, key, receiver) {
        console.log(`getting ${key}`)
        return Reflect.get(target, key, receiver)
    },
    set: function(target, key, value, receiver) {
        console.log(`setting ${key}`)
        return Reflect.set(target, key, value, receiver)
    }
})

// 对比 Object.defineProperty
let object = {},
    value

Object.defineProperty(object, 'value', {
    get: function() {
        console.log('getting value')
        return value
    },
    set: function(newValue) {
        value = newValue
        console.log('setting: ' + newValue)
    },
    enumerable: true,
    configurable: true
})
```

### 14. Reflect
Reflect 对象的设计目的有以下几个：

第一，将 Object 对象的一些明显属于语言内部的方法（如 Object.defineProperty）放到 Reflect 对象上，现阶段，某些方法同时在 Object 和 Reflect 对象上部署，未来新的方法只在 Reflect 对象上部署。也就是说，从 Reflect 对象上可以获得语言内部的方法。

第二，修改某些 Object 方法的返回结果，让其变得更加合理。

第三，让 Object 操作都编程函数行为，某些 Object 操作是命令式，比如 name in obj 和 delete obj [name]，而 Reflect.has(obj, name) 和 Reflect.deleteProperty(obj, name) 让它们变成了函数行为。

第四，Reflect 对象的方法与 Proxy 对象的方法一一对应，只要是 Proxy 对象的方法，就能在 Reflect 对象上找到对应的方法，这就是 Proxy 对象可以方便的调用对应的 Reflect 方法来完成默认行为，作为修改行为的基础。也就是说，无论 Proxy 怎么修改默认行为，我们总可以在 Reflect 上获取到默认行为。

## 三、ES7+
2016 年，ECMAScript 7（或称为 ECMAScript 2016） 正式发布，整体来说，在 ES6 的版本逐渐稳定后，后期版本添加的主要内容已经不是太多了。

### 1. Array.prototype.includes
这个数组方法主要用来判断数组中是否包含某个元素
```js
let num = [1, 2, 3, 4, 5]

console.log(num.includes(1))  // true
```

### 2. 异步函数 async/await
async 函数返回一个 Promise 对象，可以使用 then 方法添加回调函数。当函数执行的时候，一旦遇到 await 就会先返回，等到异步操作完成，再接着执行函数体内后面的语句。

#### 基本用法
```js
function timeout(ms) {
    return new Promise(resolve => {
        setTimeout(resolve, ms)
    })
}

async function asyncPrint(value, ms) {
    await timeout(ms)
    console.log(value)
}

asyncPrint('Hello', 300)
```


async 函数返回一个 Promise 对象，async 函数内部 return 语句返回的值，会成为 then 方法回调函数的参数：
```js
async function f() {
    return 'Hello'
}
f().then(v => console.log(v))  // 'Hello'
```


async 函数内部抛出错误会导致返回的 Promise 对象变成 rejected 状态，抛出的错误对象会被 catch 方法回调函数接受到：
```js
async function f() {
    throw new Error('出错了')
}

f().then(
    v => console.log(v),
    e => console.log(1+e)
)   // ‘1Error: 出错了’
```
正常情况下，await 命令后面是一个 Promise 对象，如果不是，会被转为一个立即 resolve 的 Promise 对象。

await 命令后面的 Promise 对象如果变成 rejected 状态，则 reject 的参数会被 catch 方法的回调函数接收到。

只要一个 await 语句后面的 Promise 变成 rejected，那么整个 async 函数都会被中断执行。

#### 注意
await 命令后面的 Promise 对象的运行结果可能是 rejected，最好把 await 命令放在 try...catch 中。

多个 await 命令后面的异步操作如果不存在继发关系，最好让它们同时触发。

await 命令只能用在 async 函数中，如果用在普通函数中就会报错。

<br/>
[文章来源](https://zhuanlan.zhihu.com/p/531959101)