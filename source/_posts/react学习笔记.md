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