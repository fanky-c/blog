---
title: Vue内部运行机制
date: 2019-07-15 21:56:08
tags:
    - vue
    - vue运行机制
---
[本文来源参考](https://juejin.im/book/5a36661851882538e2259c0f/section/5a37bbb35188257d167a4d64)

### new Vue() 内部流程图

```js
 new Vue() ---init----> $mount ----> compile() ----> 
```