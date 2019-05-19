---
title: js继承
date: 2019-05-18 18:09:07
tags:
     - js继承
---

### 定义父类
```js
function Person(name, age){
   //属性 
   this.name = name || 'zhao';
   this.age = age || 25;
   
   //实例方法
   this.showName = function(){
      console.log(this.name + '正在睡觉');
   }
}
//原型属性和方法
Person.prototype.friends  = ['dog', 'cat'];
Person.prototype.showAge = function(){
     console.log('年龄是：'+this.age);
}
```
### 原型式继承
#### 使用
```js
function Ming(){

}
Ming.prototype = Person.prototype;

```
#### 问题
1. 只能继承父类原型对象方法和属性
2. 子类和父类原型共享
3. 无法给父构造函数传参



### 原型链继承
#### 使用
```js
function Ming(){
}
Ming.prototype = new Person();
//修改子构造函数的原型的构造器属性
Ming.prototype.constructor = Ming;

var subClass = new Ming();
console.log(subClass.name);
console.log(subClass.age);
console.log(subClass.friends);

// 当我们改变friends的时候, 父构造函数的原型对象的也会变化
subClass.friends.push('小王八');
console.log(subClass.friends); //["dog", "cat", "小王八"]
var father = new Person();
console.log(father.friends); //["dog", "cat", "小王八"]

```
#### 问题
1. 父子原型和实例对象共享
2. 不能给父构造函数传参


### 构造函数call、apply
#### 使用
```js
function Ming(name, age){
     Person.call(this, name, age); //Person.apply(this, [name, age]);
}

var subClass = new Ming('ming', 21);
subClass.showName();
subClass.friends.push('小王八');  //报错
console.log(subClass.friends)    //报错
```

#### 问题
1. 可以传参
2. 解决父子原型共享问题
3. 获取不到父类原型成员


### 组合继承(原型式+构造函数)
#### 使用
```js
function Ming(name, age){
     Person.call(this, name, age); //Person.apply(this, [name, age]);
}

Ming.prototype = Person.prototype;
Ming.prototype.constructor = Ming;
```

#### 问题
1. 解决传参
2. 继承父类原型对象上的方法和属性
3. 但是共享了父类原型对象


### 构造函数+深拷贝
#### 使用
```js
function Ming(name, age){
   Person.call(this, name, age);
}

//深拷贝
deepCopy(Ming.prototype,Person.prototype);

Ming.prototype.constructor = Ming;

//对象深拷贝
function deepCopy(source){
  let result = null;
  let type = checkType(source);
  
  if(type === 'Object'){
      result = {};
  }else if(type === 'Array'){
     result = [];
  }else{
      return result;
  }
  
  for(let i in source){
      let value = source[i];
      if(checkType(value) === 'Object' || checkType(value) === 'Array'){
        result[i] = deepCopy(value);
      }else{
        result[i] = value;
      }
  }
  return result;
}

//返回具体类型
function checkType(obj){
    return Object.prototype.toString.call(obj).slice(8, -1); //[object,String] => String
}

```