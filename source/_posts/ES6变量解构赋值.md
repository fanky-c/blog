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
```js
let [a, b, c, d, e] = 'world';

//字符串有length属性
let {length : len} = 'hello';
len  // 5
```
### 函数参数的解构赋值

### 圆括号的解构赋值

### 数值和布尔值解构赋值


### 用途
#### 从函数返回多个值
```js
function example() {
  return {
    foo: 1,
    bar: 2
  };
}
let { foo, bar } = example();
```

#### 提取 JSON 数据
```js
let jsonData = {
  id: 42,
  status: "OK",
  data: [867, 5309]
};
let { id, status, data: number } = jsonData;
console.log(id, status, number);
// 42, "OK", [867, 5309]
```

#### 输入模块的指定方法
```js
//require加载方法
const { SourceMapConsumer, SourceNode } = require("source-map");
```