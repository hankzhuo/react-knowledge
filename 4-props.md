## 3 监听 props 改变

### 类组件中

① componentWillReceiveProps 可以作为监听 props 的生命周期，但是 React 已经不推荐使用 componentWillReceiveProps ，未来版本可能会被废弃，因为这个生命周期超越了 React 的可控制的范围内，可能引起多次执行等情况发生。于是出现了这个生命周期的替代方案 getDerivedStateFromProps ，在下一章节，会详细介绍 React 生命周期。

### 函数组件中

② 函数组件中同理可以用 useEffect 来作为 props 改变后的监听函数。(不过有一点值得注意, useEffect 初始化会默认执行一次)

```js
React.useEffect(()=>{
// props 中 number 改变，执行这个副作用。
console.log('props 改变：' ，props.number )
},[ props.number ])
```

## 抽象 props

### 混入 props

```js
function Son(props) {
  console.log(props);
  return <div> hello,world </div>;
}
function Father(props) {
  const fatherProps = {
    mes: "let us learn React !",
  };
  return <Son {...props} {...fatherProps} />;
}
function Index() {
  const indexProps = {
    name: "alien",
    age: "28",
  };
  return <Father {...indexProps} />;
}
```

### 抽离 props

有的时候想要做的恰恰和上面相反，比如想要从父组件 props 中抽离某个属性，再传递给子组件，那么应该怎么做呢？

```js
function Son(props) {
  console.log(props);
  return <div> hello,world </div>;
}

function Father(props) {
  const { age, ...fatherProps } = props;
  return <Son {...fatherProps} />;
}
function Index() {
  const indexProps = {
    name: "alien",
    age: "28",
    mes: "let us learn React !",
  };
  return <Father {...indexProps} />;
}
```

### 注入 props

#### 显式注入 props

显式注入 props ，就是能够直观看见标签中绑定的 props 。

```js
function Son(props) {
  console.log(props); // {name: "alien", age: "28"}
  return <div> hello,world </div>;
}
function Father(prop) {
  return prop.children;
}
function Index() {
  return (
    <Father>
      <Son name="alien" age="28" />
    </Father>
  );
}
```

如上向 Son 组件绑定的 name 和 age 是能直观被看见的。

#### 隐式注入 props

这种方式，一般通过 React.cloneElement 对 props.chidren 克隆再混入新的 props 。

```js
function Son(props) {
  console.log(props); // {name: "alien", age: "28", mes: "let us learn React !"}
  return <div> hello,world </div>;
}
function Father(prop) {
  return React.cloneElement(prop.children, { mes: "let us learn React !" });
}
function Index() {
  return (
    <Father>
      <Son name="alien" age="28" />
    </Father>
  );
}
```

如上所示，将 mes 属性，隐式混入到了 Son 的 props 中。
