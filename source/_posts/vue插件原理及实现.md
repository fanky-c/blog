---
title: vue插件原理及实现
date: 2019-05-26 12:25:55
tags:
    - vue
    - vue插件
---
### 插件使用场景
1. 添加全局方法或者属性，例如：[vue-custom-element](https://github.com/karol-f/vue-custom-element)
2. 添加全局资源：指令/过滤器/过渡等, 例如：[vue-touch](https://github.com/vuejs/vue-touch)
3. 通过全局混入来添加一些组件选项, 例如：[vue-router]()
4. 添加 Vue 实例方法，通过把它们添加到 Vue.prototype 上实现
5. 一个库，提供自己的 API，同时提供上面提到的一个或多个功能。 例如：[vue-route](https://github.com/vuejs/vue-router)

```js
MyPlugin.install = function (Vue, options) {
  // 1. 添加全局方法或属性
  Vue.myGlobalMethod = function () {
    // 逻辑...
  }

  // 2. 添加全局资源
  Vue.directive('my-directive', {
    bind (el, binding, vnode, oldVnode) {
      // 逻辑...
    }
    ...
  })

  // 3. 注入组件选项
  Vue.mixin({
    created: function () {
      // 逻辑...
    }
    ...
  })

  // 4. 添加实例方法
  Vue.prototype.$myMethod = function (methodOptions) {
    // 逻辑...
  }
}
```

### 插件使用
1. 通过Vue.use(myPlugin)使用， 本质是调用myPlugin.install(Vue)
2. 使用插件前必须在new Vue()启动应用之前完成，实例化之前就要配置好
3. 如果使用多次Vue.use多次注册相同的插件，那只会注册一次

### 原理
#### Vue.use()源码
```js
Vue.use = function (plugin) {   
    // 忽略已注册插件
    if (plugin.installed) {
      return
    }
    
    // 集合转数组，并去除第一个参数
    var args = toArray(arguments, 1);
    
    // 把this（即Vue）添加到数组的第一个参数中
    args.unshift(this);
    
    // 调用install方法
    if (typeof plugin.install === 'function') {
      plugin.install.apply(plugin, args);
    } else if (typeof plugin === 'function') {
      plugin.apply(null, args);
    }
    
    // 注册成功
    plugin.installed = true;
    return this;
  };
```

#### 通过mixin实现rules公共方法
1. 示例
```js
const vm = new Vue({
    data: function(){
        return {
            foo: 10
        }
    },
    rules: {
        foo: {
           validate: value => value > 1,
           message: 'foo must be greater than one'
        }
    }
})
vm.foo = 0 // 输出 foo must be greater than one
```

2. 实现步骤
   1. 实现rules插件
      ```js
       import Vue from 'vue'
       const RulesPlugin = {
           //插件应该有一个公开方法install
           //install方法第一个参数：Vue构造器， 第二个参数可选的选项对象
           install: function(Vue){
               Vue.mixin({
                   created(){
                    //获取rules对象
                    const rules = this.$options.rules;
                    if(rules){
                        Object.keys(rules).forEach((key)=>{
                            //获取所有的rules
                            const {validate, message} = rules[key];
                            
                            //监听， 键是变量， 值函数
                            this.$watch(key, newValue => {
                                const valid = validate(newValue);
                                if(valid){
                                console.log(message);
                                }
                            })
                        })
                    }                       
                   }
               })
           }
       }

       export default RulesPlugin;
      ```
   2. 验证rules插件  
     ```js
       import Vue from 'vue'
       import RulePlugin from 'RulePlugin';
       
       Vue.use(RulePlugin);

       const vm = new Vue({
            data: function(){
                return {
                    foo: 10
                }
            },
            rules: {
                foo: {
                  validate: value => value > 1,
                  message: 'foo must be greater than one'
                }
            },
            created(){
                //获取rules对象
                const rules = this.$options.rules;
                if(rules){
                    Object.keys(rules).forEach((key)=>{
                        //获取所有的rules
                        const {validate, message} = rules[key];
                        
                        //监听， 键是变量， 值函数
                        this.$watch(key, newValue => {
                            const valid = validate(newValue);
                            if(valid){
                               console.log(message);
                            }
                        })
                    })
                }
            }           
       })
     ```