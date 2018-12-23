---
title: Event Bus实现原理
date: 2018-12-23 17:14:51
tags:
  - vue/react通信
  - Event Bus
---
### vue、react不同组件怎么通信

#### Vue
1. 父子组件：props、$refs、$children
2. 子父组件：$parent、$emit自定义事件
3. 如果项目够复杂,可能需要Vuex等全局状态管理库通信
4. 非父子组件用Event Bus通信（$emit、$on）
```
  //main.js
  var eventBus = {
    install(Vue,options) {
        Vue.prototype.$bus = vue
    }
  };
  Vue.use(eventBus);

  //children.js
  this.$bus.$emit('dosth', params);

  //children.js
  this.$bus.$on('dosth , (params)=>{}); 
  beforeDestory(){
    this.$bus.$off('dosth');
  } 
```
5. $dispatch(vue2已经废除)和$broadcast(vue2已经废除)

#### React
1. 父子组件：props，如果父组件与子组件之间不止一个层级，可通过...运算符（Object 剩余和展开属性）。
```
class Child_1 extends Component{
  render() {
    return <div>
      <p>{this.props.msg}</p>
      <Child_1_1 {...this.props}/>
    </div>
  }
}
class Child_1_1 extends Component{
  render() {
    return <p>{this.props.msg}</p>
  }
}
```
2. 子父组件：props回调方法
3. 项目复杂的话用Redux、Mobx等全局状态管理管库
4. 非父子组件,用发布订阅模式的Event模块
5. 用新的Context Api
   1. 父组件首先声明自己可以支持 context，定义 childContextTypes 方法。
   2. 父组件需要定义方法，返回 context 对象，定义 getChildContext 方法。
   3. 子组间声明自己需要使用 context，定义 contextTypes 方法。

#### Event Bus
1. 它实际上是发布订阅模式。
2. 实现流程
   1. 实现类class
   ```
     class EventEmeitter{
         constructor(){
            this._evt = this._evt || new Map();
            this._maxListener = this._maxListener || 10;
         }
     }
   ```
   2. 监听和触发
   ```
     class EventEmeitter{
       emit(type, ...args){
            let handler;
            handler = this._evt.get(type); //new Map()方法
            if (args.length > 0) {
                handler.apply(this, args);
            } else {
                handler.call(this);
            }
            return true;
       }
       on(type, fn){
            if (!this._evt.get(type)) {
                this._evt.set(type, fn);
            }
     }
   ```