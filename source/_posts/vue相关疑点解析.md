---
title: vue相关疑点解析
date: 2020-01-08 17:46:19
tags:
    - vue中key
    - vue组件数据传递

---

### 如何理解vue中的key
#### 作用
* v-for遍历时， 用id作为key， 唯一标识节点加速虚拟DOM的渲染
```js
//有相同父元素的子元素必须有独特的key。重复的key会造成渲染错误。
<ul>
  <li v-for="item in items" :key="item.id">...</li>
</ul>
```
* 强制替换节点或者组件（component），而不重复使用它。
```js
<transition>
  <span :key="abc">{{ abc }}</span>
</transition>
//abc发生变化时，<span>会被替换，而不会patched(修复)，因此transition会被触发。    
```

#### 原理
##### 加速虚拟DOM的渲染
* 如果不用key，Vue会用一种算法（就地更新策略）：最小化element的移动，并且会尝试尽最大程度在同适当的地方对相同类型的element，做patch(修复)或者reuse（重用）。
* 如果使用了key，Vue会根据keys的顺序记录element，曾经拥有了key的element如果不再出现的话，会被直接remove或者destoryed。
* 如果使用key, 它会基于 key 的变化重新排列元素顺序，并且会移除 key 不存在的元素。

##### 强制替换节点或者组件
* 如果使用了key，Vue会根据keys的顺序记录element，曾经拥有了key的element如果不再出现的话，会被直接remove或者destoryed。
* 完整地触发组件的生命周期钩子
* 触发过渡


### vue中组件数据传递
