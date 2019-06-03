---
title: vue插件原理及实现
date: 2019-05-26 12:25:55
tags:
    - vue
    - vue插件
---

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