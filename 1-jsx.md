jsx 终将变成什么。

### babel 处理后的样子

JSX 元素节点会被编译成 React Element 形式。

```js
React.createElement(type, [props], [...children]);
```

`createElement` 参数：

1. 第一个参数：如果是组件类型，会传入组件对应的类或函数；如果是 dom 元素类型，传入 div 或者 span 之类的字符串。
2. 第二个参数：一个对象，在 dom 类型中为标签属性，在组件类型中为 props 。
3. 其他参数：依次为 children，根据顺序排列。

例子

```js
<div>
  <TextComponent />
  <div>hello,world</div>
  let us learn React!
</div>
```

上面的代码会被 babel 先编译成：

```js
React.createElement(
  "div",
  null,
  React.createElement(TextComponent, null),
  React.createElement("div", null, "hello,world"),
  "let us learn React!"
);
```


### createElement 处理后的样子

