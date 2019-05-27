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