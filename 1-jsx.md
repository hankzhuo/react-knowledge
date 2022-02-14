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


| jsx元素类型	| react.createElement 转换后 | type 属性 |
| ----------- | ----------- | ----------- |
| elment元素类型 |	react element类型	 | 标签字符串，例如 div ｜
|fragment类型| 	react element类型|	symbol react.fragment类型 |
| 文本类型| 	直接字符串| 	无 |
| 数组类型| 	返回数组结构，里面元素被react.createElement转换| 	无 |
| 组件类型| 	react element类型| 	组件类或者组件函数本身 |
| 三元运算 / 表达式| 	先执行三元运算，然后按照上述规则处理| 	看三元运算返回结果 | 
| 函数执行| 	先执行函数，然后按照上述规则处理| 	看函数执行返回结果 |


### 问: React.createElement 和 React.cloneElement 到底有什么区别呢?

答: 可以完全理解为，一个是用来创建 element 。另一个是用来修改 element，并返回一个新的 React.element 对象。