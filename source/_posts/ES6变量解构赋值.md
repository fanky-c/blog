---
title: ES6变量解构赋值
date: 2020-04-27 19:24:33
tags:
     - 解构赋值
---

### 数组的解构赋值
```js
let [foo, [[bar], baz]] = [1, [[2], 3]];
foo // 1
bar // 2
baz // 3

let [ , , third] = ["foo", "bar", "baz"];
third // "baz"

let [x, , y] = [1, 2, 3];
x // 1
y // 3

let [head, ...tail] = [1, 2, 3, 4];
head // 1
tail // [2, 3, 4]

let [x, y, ...z] = ['a'];
x // "a"
y // undefined
z // []
```

### 对象的解构赋值
```js
let { bar, foo, baz } = { foo: 'aaa', bar: 'bbb' };
foo // "aaa"
bar // "bbb"
baz // undefined

//用途
const { log } = console;
log('hello') // hello
```

### 字符串的解构赋值

### 函数参数的解构赋值

### 圆括号的解构赋值

### 数值和布尔值解构赋值
