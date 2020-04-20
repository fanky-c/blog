---
title: Set和Map
date: 2020-04-20 20:28:33
tags:
    - Set
    - Map
    - WeakSet
    - WeakMap
---

### Set
#### 定义
1. set类似数组， 成员是唯一且无序，也就是值不能重复。
2. 可以遍历， 方法有：add、delete、has、clear等

### WeakSet
#### 定义
1. 成员都是对象
2. 成员都是弱引用，可以随时消失。 可以保持DOM节点，不易造成内存泄露
3. 不能遍历

### Map
#### 定义
1. 本质上是键值对的集合，类似集合
2. 可以遍历，方法很多可以跟各种数据格式转换

#### 类型转换
1. map ==> array
```js
const map = new Map([[1, 1], [2, 2], [3, 3]])
console.log([...map])	// [[1, 1], [2, 2], [3, 3]]
```
2. array ==> map
```js
const map = new Map([[1, 1], [2, 2], [3, 3]])
console.log(map)	// Map {1 => 1, 2 => 2, 3 => 3}
```
3. map ==> object
```js
function mapToObj(map) {
    let obj = Object.create(null)
    for (let [key, value] of map) {
        obj[key] = value
    }
    return obj
}
const map = new Map().set('name', 'An').set('des', 'JS')
mapToObj(map)  // {name: "An", des: "JS"}
```
4. object ==> map
```js
function objToMap(obj) {
    let map = new Map()
    for (let key of Object.keys(obj)) {
        map.set(key, obj[key])
    }
    return map
}

objToMap({'name': 'An', 'des': 'JS'}) // Map {"name" => "An", "des" => "JS"}
```
5. map ==> json
```js
function mapToJson(map) {
    return JSON.stringify([...map])
}

let map = new Map().set('name', 'An').set('des', 'JS')
mapToJson(map)	// [["name","An"],["des","JS"]]
```

6. json ==> map
```js
function jsonToStrMap(jsonStr) {
  return objToMap(JSON.parse(jsonStr));
}

jsonToStrMap('{"name": "An", "des": "JS"}') // Map {"name" => "An", "des" => "JS"}
```

### WeakMap
#### 定义
1. WeakMap 对象是一组键值对的集合，其中的键是弱引用对象，而值可以是任意。
2. 不能遍历