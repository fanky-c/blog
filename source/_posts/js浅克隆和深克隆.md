---
title: js浅克隆和深克隆
date: 2018-12-27 19:01:55
tags:
    - js拷贝
    - js数据类型
---

### js数据类型

#### 基本类型
1. Null、String、Bool、Number、Symbol、Undefined
   1. Symbol: 如果有一种机制，保证每个属性的名字都是独一无二的就好了，这样就从根本上防止属性名的冲突。这就是 ES6 引入Symbol的原因。[参考](http://es6.ruanyifeng.com/#docs/symbol)

#### 引用类型
1. Object
   1. Array、Date、RegExp、Function

#### 区别
1. 基本数据类型的特点：直接存储在栈(stack)中的数据
2. 引用数据类型的特点：存储的是该对象在栈中引用，真实的数据存放在堆内存里。

### 拷贝

#### 浅拷贝
1. Object.assign();
1. 自己实现方法:
```
 function shallowClone(source = {}){
  var target = {};
  for ( let i in source) {
    target[i] = source[i];
  }
  return target;   
 }
```
1. 缺点就对象虽然被复制但是还是会指向相同内存，修改任何一方都影响另一方。



#### 深拷贝
1. JSON.parse方法
```
const oldObj = {
  a: 1,
  b: [ 'e', 'f', 'g' ],
  c: { h: { i: 2 } }
};

const newObj = JSON.parse(JSON.stringify(oldObj));
```
  1. 缺点：无法拷贝Function、RegExp；循环引用错误； 


1. 自己实现方法
```
function checkType(obj){
    retrun Object.prototype.toString.call(obj).slice(8, -1);  //[object,String]
}


function deepClone(target) {
  let result, targetType = checkedType(target)

  if (targetType === 'object') {
    result = {}
  } else if (targetType === 'Array') {
    result = []
  } else {
    return target
  }

  for (let i in target) {
    let value = target[i]
    if (checkedType(value) ==='Object' || checkedType(value) ==='Array') {
      result[i] = deepClone(value)
    } else {
      result[i] = value;
    }
  }

  return result
}

```