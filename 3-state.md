## setState 用法

React 项目中 UI 的改变来源于 state 改变，类组件中 setState 是更新组件，渲染视图的主要方式。

```js
setState(obj,callback)
第一个参数：当 obj 为一个对象，则为即将合并的 state ；如果 obj 是一个函数，那么当前组件的 state 和 props 将作为参数，返回值用于合并新的 state。

第二个参数 callback ：callback 为一个函数，函数执行上下文中可以获取当前 setState 更新后的最新 state 的值，可以作为依赖 state 变化的副作用函数，可以用来做一些基于 DOM 的操作。

/_ 第一个参数为 function 类型 _/
this.setState((state,props)=>{
return { number:1 }
})
/_ 第一个参数为 object 类型 _/
this.setState({ number:1 },()=>{
console.log(this.state.number) //获取最新的 number
})
```

批量更新：

```js
export default class index extends React.Component {
  state = { number: 0 };
  handleClick = () => {
    this.setState({ number: this.state.number + 1 }, () => {
      console.log("callback1", this.state.number);
    });

    console.log(this.state.number);

    this.setState({ number: this.state.number + 1 }, () => {
      console.log("callback2", this.state.number);
    });

    console.log(this.state.number);

    this.setState({ number: this.state.number + 1 }, () => {
      console.log("callback3", this.state.number);
    });

    console.log(this.state.number);
  };
  render() {
    return (
      <div>
        {this.state.number}
        <button onClick={this.handleClick}>number++</button>
      </div>
    );
  }
}
```

点击打印：**0, 0, 0, callback1 1 ,callback2 1 ,callback3 1**
批量更新下，先合并 state


为什么异步操作里面的批量更新规则会被打破呢？比如用 promise 或者 setTimeout 在 handleClick 中这么写：

```js
setTimeout(() => {
  this.setState({ number: this.state.number + 1 }, () => {
    console.log("callback1", this.state.number);
  });

  console.log(this.state.number);

  this.setState({ number: this.state.number + 1 }, () => {
    console.log("callback2", this.state.number);
  });

  console.log(this.state.number);

  this.setState({ number: this.state.number + 1 }, () => {
    console.log("callback3", this.state.number);
  });

  console.log(this.state.number);
});
```

打印：**callback1 1 , 1, callback2 2 , 2,callback3 3 , 3**

那么，如何在如上异步环境下，继续开启批量更新模式呢？

React-Dom 中提供了批量更新方法 `unstable_batchedUpdates`，可以去手动批量更新，可以将上述 setTimeout 里面的内容做如下修改:

```js
import ReactDOM from "react-dom";
const { unstable_batchedUpdates } = ReactDOM;

setTimeout(() => {
  unstable_batchedUpdates(() => {
    this.setState({ number: this.state.number + 1 });
    console.log(this.state.number);
    this.setState({ number: this.state.number + 1 });
    console.log(this.state.number);
    this.setState({ number: this.state.number + 1 });
    console.log(this.state.number);
  });
});
```

打印：**0 , 0 , 0 , callback1 1 , callback2 1 ,callback3 1**

那么如何提升更新优先级呢？

React-dom 提供了 flushSync ，flushSync 可以将回调函数中的更新任务，放在一个较高的优先级中。React 设定了很多不同优先级的更新任务。如果一次更新任务在 flushSync 回调函数内部，那么将获得一个较高优先级的更新。

接下来，将上述 handleClick 改版如下样子：

```js
handerClick=()=>{
  setTimeout(()=>{
    this.setState({ number: 1 })
    })
    this.setState({ number: 2 })
    ReactDOM.flushSync(()=>{
    this.setState({ number: 3 })
    })
    this.setState({ number: 4 })
  }
render(){
  console.log(this.state.number)
  return ...
}
```

打印 **3 4 1**

**flushSync 补充说明**：flushSync 在同步条件下，会合并之前的 setState | useState，可以理解成，如果发现了 flushSync ，就会先执行更新，如果之前有未更新的 setState ｜ useState ，就会一起合并了，所以就解释了如上，2 和 3 被批量更新到 3 ，所以 3 先被打印。

综上所述， React 同一级别更新优先级关系是:

**flushSync 中的 setState > 正常执行上下文中 setState > setTimeout ，Promise 中的 setState。**


## 函数组件中的state

