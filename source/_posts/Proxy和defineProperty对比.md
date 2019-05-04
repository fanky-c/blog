---
title: Proxy和defineProperty对比
date: 2018-12-21 18:55:12
tags:
    - 双向绑定
    - Proxy
    - defineProperty
---

### 双向绑定
1. 发布-订阅
2. 脏检测
3. 数据劫持
  1. Object.definePropery (es5）
  2. Proxy (es6)
  3. Object.observe(废弃)
4. 数据模型


### definePropery

#### 1. 不能检测数组长度变化，准确的说是通过改变Length而增加的长度不能检测。

> [length 和数字下标之间的关系](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array#Relationship_between_length_and_numerical_properties) —— JavaScript 数组的 length 属性和其数字下标之间有着紧密的联系。数组内置的几个方法（例如 join、slice、indexOf 等）都会考虑 length 的值。另外还有一些方法（例如 push、splice 等）还会改变 length 的值。[为什么defineProperty不能检测到数组长度的“变化”](https://iming.work/detail/5b0cb1f17f6fd30063942cd5)

  1. 当你利用索引直接设置一个项时，vm.items[indexOfItem] = newValue
    1. 解决方法：Vue.set(vm.items, indexOfItem, newValue) / vm.$set(vm.items, indexOfItem, newValue) / vm.items.splice(indexOfItem, 1, newValue)
  2. 当你修改数组的长度时，vm.items.length = newLength
    1. 解决方法：vm.items.splice(newLength)

#### 2. 为什么defineProperty不能检测到数组长度的“变化”
1. 

#### 3. vue对数组方法的hack
```
const arrayProto = Array.prototype
// 复制了数组构造函数的原型
// 这里需要注意的是数组构造函数的原型也是个数组
// 实例中指向原型的指针__proto__也是个数组
// 数组并没有索引，因为length = 0
// 相反的拥有属性，属性名为数组方法，值为对应的函数
export const arrayMethods = Object.create(arrayProto)

// 对以下方法重写
const methodsToPatch = [
  'push',
  'pop',
  'shift',
  'unshift',
  'splice',
  'sort',
  'reverse'
]

methodsToPatch.forEach(function (method) {
  // cache original method
  const original = arrayProto[method]
  // 这里的def很重要，其实也就是用object.defineProperty重新定义属性
  // 但这里的arrayMethods是个数组，这就是为什么上面我们解释
  // 数组构造函数原型是个空数组但是默认了属性方法
  // 所以这里的定义是很巧妙的
  def(arrayMethods, method, function mutator (...args) {
    const result = original.apply(this, args)
    // ob就是observe实例
    const ob = this.__ob__
    let inserted
    switch (method) {
      // 为什么对push和unshift单独处理？
      // 我们在上看解释过，这2中方法会增加数组的索引，但是新增的索引位需要手动observe的
      case 'push':
      case 'unshift':
        inserted = args
        break
      // 同理，splice的第三个参数，为新增的值，也需要手动observe
      case 'splice':
        inserted = args.slice(2)
        break
    }
    // 其余的方法都是在原有的索引上更新，初始化的时候已经observe过了
    if (inserted) ob.observeArray(inserted)
    // notify change
    // 然后通知所有的订阅者触发回调
    ob.dep.notify()
    return result
  })
})
```
#### 4. Vue 不能检测对象属性的添加或删除
1. 对于已经创建的实例，Vue 不能动态添加根级别的响应式属性
```
var vm = new Vue({
  data: {
    a: 1
  }
})
// `vm.a` 现在是响应式的

vm.b = 2
// `vm.b` 不是响应式的
```
1. 解决方法：
   1. Vue.set(vm.userProfile, key, value)
   2. vm.$set(vm.userProfile, key, value)
   3. Object.assign()

#### 5. definePropery缺点
1. 无法监听数组变化
2. 无法检测对象属性的添加和删除
3. 性能差。只能劫持对象的属性,因此我们需要对每个对象的每个属性进行遍历，如果属性值也是对象那么需要深度遍历


### Proxy

#### Proxy可以直接监听对象而非属性
```
const input = document.getElementById('input');
const p = document.getElementById('p');
const obj = {};

const newObj = new Proxy(obj, {
  get: function(target, key, receiver) {
    return Reflect.get(target, key, receiver);
  },
  set: function(target, key, value, receiver) {
    if (key === 'text') {
      input.value = value;
      p.innerHTML = value;
    }
    return Reflect.set(target, key, value, receiver);
  },
});

input.addEventListener('keyup', function(e) {
  newObj.text = e.target.value;
});
```

#### Proxy可以直接监听数组的变化

#### 优缺点
1. 优点：对数组和对象支持非常好，不需要hask；性能比definePropery好，初始化很多数据，当用到才会被监听。
2. 缺点：兼容性不太好，而且无法用polyfill磨平