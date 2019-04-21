---
title: JS内存空间分配
date: 2019-04-18 17:40:38
tags:
    - js内存
    - 栈、堆
---
### js数据类型

#### 基本数据
  1. Null、Bollean、String、Number、Undefined、Symbol

#### 引用数据
  1. Object
     1. Array、Function、RegExp、Date

#### 内存分布
  ![内存分别](/blog/img/heap.png)
  1. 闭包中变量不会存放在栈。（函数 A 返回了一个函数 B，并且函数 B 中使用了函数 A 的变量，函数 B 就被称为闭包。）
  ```
    function A() {
    let a = 1
    function B() {
        console.log(a)
    }
    return B
    }
  ```


### js内存生命周期
1. 分配所需的内存
2. 使用分配的内存（读、写）
3. 不需要将内存释放（自动：对象不引用即释放\手动： a = null）


### 例子详解
#### 基本数据复制、修改
```
    var a = 20;
    var b = a;
    b = 30;
    // 这时a的值是多少？
```

#### 引用数据复制、修改
```
    var m = { a: 10, b: 20 }
    var n = m;
    n.a = 15;
    // 这时m.a的值是多少
```