## React 各阶段生命周期能做些什么

### constructor

在 constructor 做一些初始化的工作。

```js
constructor(props){
    super(props)        // 执行 super ，别忘了传递props,才能在接下来的上下文中，获取到props。
    this.state={       //① 可以用来初始化state，比如可以用来获取路由中的
        name:'alien'
    }
    this.handleClick = this.handleClick.bind(this) /* ② 绑定 this */
    this.handleInputChange = debounce(this.handleInputChange , 500) /* ③ 绑定防抖函数，防抖 500 毫秒 */
    const _render = this.render
    this.render = function(){
        return _render.bind(this)  /* ④ 劫持修改类组件上的一些生命周期 */
    }
}
/* 点击事件 */
handleClick(){ /* ... */ }
/* 表单输入 */
handleInputChange(){ /* ... */ }

```

### getDerivedStateFromProps

> getDerivedStateFromProps(nextProps,prevState)

两个参数：

- nextProps 父组件新传递的 props ;
- prevState 组件在此次更新前的 state 。

getDerivedStateFromProps 方法作为类的静态属性方法执行，内部是访问不到 this 的，它更趋向于纯函数。

只要组件更新，就会执行 getDerivedStateFromProps，不管是 props 改变，还是 setState ，或是 forceUpdate 。

```js
static getDerivedStateFromProps(newProps){
    const { type } = newProps
    switch(type){
        case 'fruit' :
        return { list:['苹果','香蕉','葡萄' ] } /* ① 接受 props 变化 ， 返回值将作为新的 state ，用于 渲染 或 传递给 shouldComponentUpdate */
        case 'vegetables':
        return { list:['菠菜','西红柿','土豆']}
    }
}
render(){
    return <div>{ this.state.list.map((item)=><li key={item} >{ item  }</li>) }</div>
}
```

getDerivedStateFromProps 作用：

- 代替 componentWillMount 和 componentWillReceiveProps
- 组件初始化或者更新时，将 props 映射到 state。
- 返回值与 state 合并完，可以作为 shouldComponentUpdate 第二个参数 newState ，可以判断是否渲染组件。(请不要把 getDerivedStateFromProps 和 shouldComponentUpdate 强行关联到一起，两者没有必然联系)

### getSnapshotBeforeUpdate

> getSnapshotBeforeUpdate(prevProps,preState){}

两个参数：

- prevProps 更新前的 props ；
- preState 更新前的 state；

把 getSnapshotBeforeUpdate 用英文解释一下 ， get | snap shot | before | update ， 中文翻译为 获取更新前的快照，可以进一步理解为 获取更新前 DOM 的状态。见名知意，上面说过该生命周期是在 commit 阶段的 before Mutation ( DOM 修改前)，此时 DOM 还没有更新，但是在接下来的 Mutation 阶段会被替换成真实 DOM 。此时是获取 DOM 信息的最佳时期，getSnapshotBeforeUpdate 将返回一个值作为一个 snapShot(快照)，传递给 componentDidUpdate 作为第三个参数。

```js
getSnapshotBeforeUpdate(prevProps,preState){
    const style = getComputedStyle(this.node)
    return { /* 传递更新前的元素位置 */
        cx:style.cx,
        cy:style.cy
    }
}
componentDidUpdate(prevProps, prevState, snapshot){
    /* 获取元素绘制之前的位置 */
    console.log(snapshot)
}
```

getSnapshotBeforeUpdate 这个生命周期意义就是配合 componentDidUpdate 一起使用，计算形成一个 snapShot 传递给 componentDidUpdate 。保存一次更新前的信息。

### componentDidUpdate

```js
componentDidUpdate(prevProps, prevState, snapshot){
    const style = getComputedStyle(this.node)
    const newPosition = { /* 获取元素最新位置信息 */
        cx:style.cx,
        cy:style.cy
    }
}
```

三个参数：

- prevProps 更新之前的 props ；
- prevState 更新之前的 state ；
- snapshot 为 getSnapshotBeforeUpdate 返回的快照，可以是更新前的 DOM 信息。

作用

- componentDidUpdate 生命周期执行，此时 DOM 已经更新，可以直接获取 DOM 最新状态。这个函数里面如果想要使用 setState ，一定要加以限制，否则会引起无限循环。
- 接受 getSnapshotBeforeUpdate 保存的快照信息。

### componentDidMount

componentDidMount 生命周期执行时机和 componentDidUpdate 一样，一个是在初始化，一个是组件更新。此时 DOM 已经创建完，既然 DOM 已经创建挂载，就可以做一些基于 DOM 操作，DOM 事件监听器。

```js
async componentDidMount(){
    this.node.addEventListener('click',()=>{
        /* 事件监听 */
    })
    const data = await this.getData() /* 数据请求 */
}
```

作用：

- 可以做一些关于 DOM 操作，比如基于 DOM 的事件监听器。
- 对于初始化向服务器请求数据，渲染视图，这个生命周期也是蛮合适的。

### shouldComponentUpdate

> shouldComponentUpdate(newProps,newState,nextContext){}

shouldComponentUpdate 三个参数，第一个参数新的 props ，第二个参数新的 state ，第三个参数新的 context 。

```js
shouldComponentUpdate(newProps,newState){
    if(newProps.a !== this.props.a ){ /* props中a属性发生变化 渲染组件 */
        return true
    }else if(newState.b !== this.props.b ){ /* state 中b属性发生变化 渲染组件 */
        return true
    }else{ /* 否则组件不渲染 */
        return false
    }
}
```

这个生命周期，一般用于性能优化，shouldComponentUpdate 返回值决定是否重新渲染的类组件。需要重点关注的是第二个参数 newState ，如果有 getDerivedStateFromProps 生命周期 ，它的返回值将合并到 newState ，供 shouldComponentUpdate 使用。

### componentWillUnmount

componentWillUnmount 是组件销毁阶段唯一执行的生命周期，主要做一些收尾工作，比如清除一些可能造成内存泄漏的定时器，延时器，或者是一些事件监听器。

```js
componentWillUnmount(){
    clearTimeout(this.timer)  /* 清除延时器 */
    this.node.removeEventListener('click',this.handerClick) /* 卸载事件监听器 */
}
```

作用

- 清除延时器，定时器。
- 一些基于 DOM 的操作，比如事件监听器。

## 函数组件生命周期替代方案

### useEffect

```js
useEffect(() => {
  return destory;
}, dep);
```

useEffect 第一个参数 callback, 返回的 destory ， destory 作为下一次 callback 执行之前调用，用于清除上一次 callback 产生的副作用。

第二个参数作为依赖项，是一个数组，可以有多个依赖项，依赖项改变，执行上一次 callback 返回的 destory ，和执行新的 effect 第一个参数 callback 。

对于 useEffect 执行， React 处理逻辑是采用异步调用 ，对于每一个 effect 的 callback， React 会向 setTimeout 回调函数一样，放入任务队列，等到主线程任务完成，DOM 更新，js 执行完成，视图绘制完毕，才执行。所以 effect 回调函数不会阻塞浏览器绘制视图。

### useLayoutEffect

useLayoutEffect 和 useEffect 不同的地方是采用了同步执行，那么和 useEffect 有什么区别呢？

- 首先 useLayoutEffect 是在 DOM 绘制之前，这样可以方便修改 DOM ，这样浏览器只会绘制一次，如果修改 DOM 布局放在 useEffect ，那 useEffect 执行是在浏览器绘制视图之后，接下来又改 DOM ，就可能会导致浏览器再次回流和重绘。而且由于两次绘制，视图上可能会造成闪现突兀的效果。
- useLayoutEffect callback 中代码执行会阻塞浏览器绘制。

一句话概括如何选择 useEffect 和 useLayoutEffect ：修改 DOM ，改变布局就用 useLayoutEffect ，其他情况就用 useEffect 。

### componentDidMount 替代方案

```js
React.useEffect(()=>{
/_ 请求数据 ， 事件监听 ， 操纵 dom _/
},[]) /_ 切记 dep = [] _/
```

这里要记住 dep = [] ，这样当前 effect 没有任何依赖项，也就只有初始化执行一次。

### componentWillUnmount 替代方案

```js
React.useEffect(()=>{
/_ 请求数据 ， 事件监听 ， 操纵 dom ， 增加定时器，延时器 _/
return function componentWillUnmount(){
/_ 解除事件监听器 ，清除定时器，延时器 _/
}
},[])/_ 切记 dep = [] _/
```

在 componentDidMount 的前提下，useEffect 第一个函数的返回函数，可以作为 componentWillUnmount 使用。
