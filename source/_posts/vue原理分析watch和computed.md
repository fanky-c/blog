---
title: vue原理分析watch和computed
date: 2020-04-12 16:15:14
tags:
    - computed原理
    - watch原理
    - vue原理
---

### 使用用法
#### watch（侦听属性）
```js
watch: {
    obj: {
        handler(val){

        },
        immediate: true, //初始立即执行， 不然只有当obj改变的时候才会触发handler
        deep: true //是否深度遍历，性能会差。建议用字符串“obj.a.b”
    }
}
```

#### computed（计算属性）
```js
computed: {
    data: function(){
        return {
            msg: 'hello world',
        }
    },
    //仅读取
    reverseMsg: function(){
        return this.msg.split('').reverse().join('');
    },
    //读取和设置
    //vm.reverseMsg2 = 'set'
    //console.log(this.msg) "hello worldset"
    reverseMsg2: {
        get: function(){
            return this.msg.split('').reverse().join('');
        },
        set: function(v){
            this.msg = v;
        }
    }
}

```

### 使用场景
#### watch
1. watch函数接受oldValue和newValue参数
2. 其他的操作依赖当前的值变化，不会产生新的值(data、props)

#### computed
1. 当模板中某个新的值依赖一个或者多个数据(data、props)时， 计算值会被缓存(依赖的属性值不变就不触发，源码中根据dirty字段判断要读取缓存)
2. computed函数不接受参数


### computed实现原理
#### 流程
对data的set操作  ==> Dep.notfiy()通知  ===> watcher ===> update更新视图。

#### 案例
```js

//因为other值的改变引起视图的变化，视图的变化肯定会读取sum的值。
//我们希望每次other的半会不会重新计算sum的值

<div id="app">
  <span @click="sumAdd">{{sum}}</span>
  <span @click="change">{{other}}</span>
</div>
<script>
  new Vue({
    el: "#app",
    data() {
      return {
        count: 1,
        other: 'Hello World'
      }
    },
    methods: {
      sumAdd() {
        this.count = 2;
      },
      change() {
        this.other = 'wtf'
      }
    },
    computed: {
      // 非常耗时的计算属性
      sum() {
        let i = 9999999999999999
        while(i > 0) {
            i--;
        }
        return this.count + 1
      },
    },
  })
</script>
```

#### 总结
##### 不缓存
1. 更新count的值就会重新算sum的值

##### 缓存
1. 更改other的值不会重新计算sum的值