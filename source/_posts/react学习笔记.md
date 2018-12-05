---
title: react学习笔记
date: 2018-10-24 14:28:05
tags:
---
### 事件处理

#### 1，注意事项
* React事件绑定属性的命名采用驼峰式写法，而不是小写。
* 如果是用jsx的语法需要穿一个函数作为事件处理函数，而不是一个字符串
```js
<button onClick={activateLasers}>
  Activate Lasers
</button>
```
* 在 React 中另一个不同是你不能使用返回 false 的方式阻止默认行为。你必须明确的使用 preventDefault。
```js
function ActionLink() {
  function handleClick(e) {
    e.preventDefault();
    console.log('The link was clicked.');
  }
  return (
    <a href="#" onClick={handleClick}>
      Click me
    </a>
  );
}
```
* ES6 class语法定义组件，事件处理器会成为类的一个方法。
```js
class Toggle extends React.Component {
  constructor(props) {
    super(props);
    this.state = {isToggleOn: true};
    // This binding is necessary to make `this` work in the callback
    this.handleClick = this.handleClick.bind(this);
  }
  handleClick() {
    this.setState(prevState => ({
      isToggleOn: !prevState.isToggleOn
    }));
  }
  render() {
    return (
      <button onClick={this.handleClick}>
        {this.state.isToggleOn ? 'ON' : 'OFF'}
      </button>
    );
  }
}
```


#### 2，向事件处理程序传递参数
* 下面两种方式是等价的，分别通过 arrow functions 和 Function.prototype.bind 来为事件处理函数传递参数。
```js
<button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>
<button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
```
* 箭头函数必须显示传递e, bind方式要隐示传递
* 通过bind 方式向监听函数传参，在类组件中定义的监听函数，事件对象 e 要排在所传递参数的后面
```js
class Popper extends React.Component{
    constructor(){
        super();
        this.state = {name:'Hello world!'};
    }
    preventPop(name, e){    //事件对象e要放在最后
        e.preventDefault();
        alert(name);
    }   
    render(){
        return (
            <div>
                <p>hello</p>
                {/* Pass params via bind() method. */}
                <a href="https://reactjs.org" onClick={this.preventPop.bind(this,this.state.name)}>Click</a>
            </div>
        );
    }
}
```

### Redux

####  1, 介绍
* Store
* State
* Action
* Action Creator
* store.dispatch()
* Reducer
* store.subscribe()

####  2, 工作流程
* 首先，用户发出 Action
```js
store.dispatch(action);
```
* 然后，Store 自动调用 Reducer，并且传入两个参数：当前 State 和收到的 Action。 Reducer 会返回新的 State 。
```js
let nextState = todoApp(previousState, action);
```
* State 一旦有变化，Store 就会调用监听函数。
```js
store.subscribe(listener)
```
* 可以通过store.getState()得到当前状态。如果使用的是 React，这时可以触发重新渲染 View。
```js
  let newState = store.getState();
  component.setState(newState);
```
参考[http://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_one_basic_usages.html]

#### 3,组件拆分和react-redux、Provider、connect(连接器)
* 在组件开发中，按职责我们可以划分：容器类组件、展示类组件，前者负责从state获取属性，后者负责渲染界面和自身的状态控制。
* react-redux 为 React 组件和 Redux 提供的 state 提供了连接。当然可以直接在 React 中使用 Redux：在最外层容器组件中初始化 store，然后将 state 上的属性作为 props 层层传递下去。但是这样并是我们推荐的状态管理方式。
```
class App extends Component{
  componentWillMount(){
    store.subscribe((state)=>this.setState(state))
  }
  render(){
    return <Comp state={this.state}
                 onIncrease={()=>store.dispatch(actions.increase())}
                 onDecrease={()=>store.dispatch(actions.decrease())}/>
  }
}
```
* 我们推荐的方式是react-redux：1，内容组件最外层包裹Provider，将之前创建的store作为prop传给Provider。Provider接收redux的createStore()的结果，并且放到context里，让子组件可以通过context属性直接获取到这个createStore的结果。且返回一个与store连接后的容器组件。2，Provider内的任何一个组件，如果需要使用state中的数据，就必须是「被 connect 过的」组件——使用connect方法对当前内容组件包装后的产物，connect会返回一个与store连接后的新组件。3，connect会返回一个与store连接后的新组件。会接收到mapStateToProps，会在内部subscribe全局state的改变，来判断props是否更改，如果需要更新，才触发更新。

```
import React, { Component } from 'react';
import PropTypes from 'prop-types';
import { connect } from 'react-redux';
import { bindActionCreators } from 'redux';
import * as dictActions from '../../actions/dict';

class DictPage extends Component {
  constructor(props){
    super(props);

  }
  componentWillMount(){
     const { getDictsResult } = this.props;
     getDictsResult({
       q: 'good',
       lang: 'en'
     })
  }
  componentDidMount(){
    const { dict } = this.props;
  }
  render() {
    return (<div>xxxxx</div>);
  }
}
//创建输入逻辑mapStateToProps,是store、state映射
function mapStateToProps(state) {
  return {
    dict: state.dict.data.data
  };
}

//输出逻辑mapDispatchToProps，是dispatch映射
function mapDispatchToProps(dispatch) {
  return bindActionCreators(dictActions, dispatch);
}

//把容器组件和ui组件结合导出
export default connect(mapStateToProps, mapDispatchToProps)(DictPage);
```

### React-router(v4版本)

#### 1， 包的选择
* 浏览器用的router在react-router-dom里。所以浏览器里使用的时候只需要import react-router-dom就可以。react-router-native供React Native应用使用。

#### 2，路由不在集中存放一起，可以和Ui组件放在一起，成为组件的一部分。

#### 3，包含式路由与exact
* 如匹配 path="/users" 的路由会匹配 path="/"的路由，在页面中这两个模块会同时进行渲染，所以就多了exact。

#### 4，独立路由Switch
* 采用Switch 只有一个路由会被渲染，并且总是渲染第一个匹配到的组件。

#### 5，link
* to（string / object）：要跳转的路径或地址；
* replace ：为 true 时，点击链接后将使用新地址替换掉访问历史记录里面的原地址。反之不会替换记录，默认为false。
* NavLink ：<NavLink>是<Link>的一个特定版本，会在匹配上当前URL的时候会给已经渲染的元素添加样式参数。

#### 6，异步加载路由和模块
* [参考](https://www.jianshu.com/p/ba3c295be412)

#### 7, 手动控制路由的跳转
* 使用 withRouter高阶组件，提供了history让你使用
```
import React from "react";
import {withRouter} from "react-router-dom";

class MyComponent extends React.Component {
  ...
  myFunction() {
    this.props.history.push("/some/Path");
  }
  ...
}
const MyComponent=withRouter(connect(mapStateToProps,mapDispatchToProps)(MyComponent))
export default MyComponent;
```
* 使用Context,在Router组件中通过Contex暴露了一个router对象。context增加了耦合难度尽量少用或者用在全局登录状态、颜色等等。
```
import React from "react";
import PropTypes from "prop-types";

class MyComponent extends React.Component {
  static contextTypes = {
    router: PropTypes.object
  }
  ...
  myFunction() {
    this.context.router.history.push("/some/Path");
  }
  ...
}
```
* 引入Redirect，重定向。

* 引入react-router-redux库，操作redux，进行时间旅行调试。
```
import React from "react";
import { push } from 'react-router-redux'
import PropTypes from "prop-types";

class MyComponent extends React.Component {
  static contextTypes = {
    store: PropTypes.object
  }
  ...
  myFunction() {
    this.context.store.dispatch(push('path'))
  }
  ...
}
```

### constructor(super(props))
#### constructor( )——构造方法
* ES6对类的默认方法，new命令生成实例自动调用，如果没有显示定义会默认添加空的constructor方法。
* es5没有继承写法，通过prototype来达到目的。
```
//构造函数People
   function People (name,age){
        this.name = name;
        this.age = age
    }
    People.prototype.sayName = function(){
        return '我的名字是：'+this.name;
    }
```
* ES6中，可以通过class来实现
```
class People{
        //构造方法constructor就等于上面的构造函数People
        constructor(name,age){
            this.name = name;
            this.age = age;
        }

        sayName(){
            return '我的名字是：'+this.name;
        }
    }
```
#### super() -- 继承。
* 子类是没有自己的this对象的，它只能继承自父类的this对象，然后对其进行加工，而super( )就是将父类中的this对象继承给子类的。没有super，子类就得不到this对象
```
class People{
        constructor(name,age){
            this.name = name;
            this.age = age;
        }
        sayName(){
            return '我的名字是：'+this.name;
        }
    }

    class har extends People{
        constructor(name,age,sex){
            super(name,age);//调用父类的constructor(name,age)
            this.sex = sex;
        }
        haha(){
            return this.sex + ' ' + super.sayName();//调用父类的sayName() 
        }
    }
``` 
####  es5 new
* 1.生成一个空的对象并将其作为 this；
* 2.将空对象的 __proto__ 指向构造函数的 prototype；
* 3.运行该构造函数；
* 4.如果构造函数没有 return 或者 return 一个返回 this 值是基本类型，则返回this；如果 return 一个引用类型，则返回这个引用类型。

##### es5和es6继承机制区别
* es5先创建子类的实例对象this，然后再将父类的方法添加到this上（ Parent.apply(this) ） 
* ES6采用的是先创建父类的实例this（故要先调用 super( )方法），完后再用子类的构造函数修改this

### 学习文章参考
* https://juejin.im/post/5b4de4496fb9a04fc226a7af