## Contents

* [无状态函数](#无状态函数)
* [JSX扩展属性](#JSX扩展属性)
* [解构参数](#解构参数)
* [条件渲染](#条件渲染)
* [子元素类型](#子元素类型)
* [数组做为子元素](#数组做为子元素)
* [函数作为子元素](#函数作为子元素)
* [渲染回调](#渲染回调)
* [子元素传递](#子元素传递)
* [代理组件](#代理组件)
* [样式组件](#样式组件)
* [事件转换](#事件转换)
* [布局组件](#布局组件)
* [容器组件](#容器组件)
* [高阶组件](#高阶组件)
* [状态提升](#状态提升)
* [可控的input](#可控的input)

## 无状态函数

[无状态函数](https://facebook.github.io/react/docs/components-and-props.html) 是定义高度可复用组件的绝妙方法。它们不保存状态，它们仅仅是函数。

```js
const Greeting = () => <div>Hi there!</div>
```

它们得到传入的`props`和`context`.

```js
const Greeting = (props, context) =>
  <div style={{color: context.color}}>Hi {props.name}!</div>
```

它们可以定义函数块内的局部变量。

```js
const Greeting = (props, context) => {
  const style = {
    fontWeight: "bold",
    color: context.color,
  }

  return <div style={style}>{props.name}</div>
}
```

但使用其他函数可以得到相同的结果。

```js
const getStyle = context => ({
  fontWeight: "bold",
  color: context.color,
})

const Greeting = (props, context) =>
  <div style={getStyle(context)}>{props.name}</div>
```

它们也可以定义`defaultProps`, `propTypes`和`contextTypes`.

```js
Greeting.propTypes = {
  name: PropTypes.string.isRequired
}
Greeting.defaultProps = {
  name: "Guest"
}
Greeting.contextTypes = {
  color: PropTypes.string
}
```


## JSX扩展属性

扩展属性是一个JSX特性。这是用于将所有对象的属性作为JSX属性传递的语法糖。

这两个例子是等价的。

```js
// 写作属性的props
<main className="main" role="main">{children}</main>

// 从对象中扩展的props
<main {...{className: "main", role: "main", children}} />
```

使用这种方式传送`props`给下层组件

```js
const FancyDiv = props =>
  <div className="fancy" {...props} />
```

现在，我可以期待FancyDiv添加它所关注还不属于它的属性。

```js
<FancyDiv data-id="my-fancy-div">So Fancy</FancyDiv>

// output: <div className="fancy" data-id="my-fancy-div">So Fancy</div>
```

要记得顺序很重要。如果`props.className`被定义，它将会破坏`FancyDiv`定义的`className`

```js
<FancyDiv className="my-fancy-div" />

// output: <div className="my-fancy-div"></div>
```

我们可以将`FancyDiv`的`className`放置在扩展`props`（`{...props}`）之后，使其始终“获胜”。

```js
// my `className` clobbers your `className`
const FancyDiv = props =>
  <div {...props} className="fancy" />
```

你应该优雅地处理这些类型的`props`。在这种情况下，我将合并作者的`props.className`和样式组件所需的`className`。

```js
const FancyDiv = ({ className, ...props }) =>
  <div
    className={["fancy", className].join(' ')}
    {...props}
  />
```


## 解构参数

[解构赋值](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)是一个ES2015特性。它与无状态函数中的`props`配对得很好。

这些例子是等价的。
```js
const Greeting = props => <div>Hi {props.name}!</div>

const Greeting = ({ name }) => <div>Hi {name}!</div>
```

[剩余参数语法](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters) (`...`)允许你收集新对象中的所有其余属性。

```js
const Greeting = ({ name, ...props }) =>
  <div>Hi {name}!</div>
```

反过来，该对象可以使用[JSX扩展属性](#JSX扩展属性)将`props`转发给组合组件。

```js
const Greeting = ({ name, ...props }) =>
  <div {...props}>Hi {name}!</div>
```

避免将非DOM `props`转发给组合组件。解构使得这很容易，因为你可以创建一个新的`props`对象，而不需要特定的组件`props`。


## 条件渲染

你不能在组件定义中使用常规的if/else条件。[条件（三元）运算符](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Conditional_Operator)是你的朋友。

`if`

```js
{condition && <span>Rendered when `truthy`</span> }
```

`unless`

```js
{condition || <span>Rendered when `falsey`</span> }
```

`if-else` (整洁的一行)

```js
{condition
  ? <span>Rendered when `truthy`</span>
  : <span>Rendered when `falsey`</span>
}
```

`if-else` (大块儿)

```js
{condition ? (
  <span>
    Rendered when `truthy`
  </span>
) : (
  <span>
    Rendered when `falsey`
  </span>
)}
```


## 子元素类型

React可以渲染多种类型的子元素。在大多数情况下，它是一个'array'或一个'string'。

`string`

```js
<div>
  Hello World!
</div>
```

`array`

```js
<div>
  {["Hello ", <span>World</span>, "!"]}
</div>
```

函数可能被用作子元素。但是，它需要[与父组件协调](#渲染回调)才有用。

`function`

```js
<div>
  {(() => { return "hello world!"})()}
</div>
```


## 数组做为子元素

提供一个数组作为子元素是非常普遍的。在React中列表就是这么绘制的。

我们使用`map()`创建一个数组，该数组中的每个值对应一个React Elements。

```js
<ul>
  {["first", "second"].map((item) => (
    <li>{item}</li>
  ))}
</ul>
```

这相当于提供一个直译的数组。

```js
<ul>
  {[
    <li>first</li>,
    <li>second</li>,
  ]}
</ul>
```

这个模式可以与解构，JSX扩展属性和其他组件结合使用，以获得一些严谨的简洁。

```js
<ul>
  {arrayOfMessageObjects.map(({ id, ...message }) =>
    <Message key={id} {...message} />
  )}
</ul>
```


## 函数作为子元素

用函数作为子元素不是天生就有用的。

```js
<div>{(() => { return "hello world!"})()}</div>
```

但是，它能被用在一些严肃有力的组件创造上。这种技术通常被称为`渲染回调`。

这种强大的技巧被很多库使用像[ReactMotion](https://github.com/chenglou/react-motion)。在应用时，渲染逻辑可以保存在所有者组件中，而不是被委派。

更多详细信息，请参阅[渲染回调](#渲染回调)。

## 渲染回调

这是一个使用渲染回调的组件。它没有用，但是一个用来开始的简单例子。

```js
const Width = ({ children }) => children(500)
```

该组件把子组件当做函数调用，传入一些数字参数。在这里是数字500。

为了使用这个组件，我们给它一个[函数作为子元素](#函数作为子元素)。

```js
<Width>
  {width => <div>window is {width}</div>}
</Width>
```

我们得到这个输出。

```js
<div>window is 500</div>
```

通过这个设置，我们可以使用这个宽度来作出渲染决定。

```js
<Width>
  {width =>
    width > 600
      ? <div>min-width requirement met!</div>
      : null
  }
</Width>
```

如果我们打算使用这个条件，我们可以定义另一个组件来封装重用的逻辑。

```js
const MinWidth = ({ width: minWidth, children }) =>
  <Width>
    {width =>
      width > minWidth
        ? children
        : null
    }
  </Width>
```

很显然，一个静态的`width`组件并不是很有用，但是它可以监视浏览器窗口。这是一个示例实现。

```js
class WindowWidth extends React.Component {
  constructor() {
    super()
    this.state = { width: 0 }
  }

  componentDidMount() {
    this.setState(
      {width: window.innerWidth},
      window.addEventListener(
        "resize",
        ({ target }) =>
          this.setState({width: target.innerWidth})
      )
    )
  }

  render() {
    return this.props.children(this.state.width)
  }
}
```

许多开发人员喜欢这种类型的函数性的高阶组件。这是一个偏好的问题。


## 子元素传递

你可以创建一个旨在应用上下文（context）并呈现其子级的组件。

```js
class SomeContextProvider extends React.Component {
  getChildContext() {
    return {some: "context"}
  }

  render() {
    // how best do we return `children`?
  }
}
```

你面临着一个决定。将子元素包裹在无关`<div />`中或直接返回子元素。第一个选项会增加额外的标记（这可能破坏一些样式表）。第二个将导致无益的错误（译注：我用15.4+版本测试的这里不会报错，不知道是作者表述有误还是我理解有误，[demo](https://codepen.io/yueshuiniao/pen/BmmdzY)）。

```js
// option 1: 额外的div
return <div>{children}</div>

// option 2: 无益的错误
return children
```

最好把子元素当成一种不透明的数据类型。React提供React.Children来适当地处理子元素。

```js
return React.Children.only(this.props.children)
```


## 代理组件

*(我不确定这个名字是否有意义)*

网络应用程序中的按钮无处不在。而且它们每个都必须将类型属性设置为"button"。

```js
<button type="button">
```

写这个属性几百次是容易出错的。我们可以编写更高级别的组件来代理props到更低级别的按钮组件。

```js
const Button = props =>
  <button type="button" {...props}>
```

我们可以使用`Button`来代替`button`，并确保类型属性始终适用于任何地方。

```js
<Button />
// <button type="button"><button>

<Button className="CTA">Send Money</Button>
// <button type="button" class="CTA">Send Money</button>
```


## 样式组件

这是一个应用于样式实践的[代理组件](#代理组件).

假设我们有一个按钮。它使用class "primary"作为按钮样式。

```js
<button type="button" className="btn btn-primary">
```

我们可以使用几个单一用途的组件来生成这个输出。

```js
import classnames from 'classnames'

const PrimaryBtn = props =>
  <Btn {...props} primary />

const Btn = ({ className, primary, ...props }) =>
  <button
    type="button"
    className={classnames(
      "btn",
      primary && "btn-primary",
      className
    )}
    {...props}
  />
```

更形象化的展示。

```js
PrimaryBtn()
  ↳ Btn({primary: true})
    ↳ Button({className: "btn btn-primary"}, type: "button"})
      ↳ '<button type="button" class="btn btn-primary"></button>'
```

使用这些组件，所有这些都会产生相同的输出。

```js
<PrimaryBtn />
<Btn primary />
<button type="button" className="btn btn-primary" />
```

这对样式的维护很有好处。它将样式的所有关注点分离到单个组件。


## 事件转换

在编写事件处理程序时，通常采用 `handle{eventName}` 命名约定。

```js
handleClick(e) { /* do something */ }
```

对于处理多个事件类型的组件，这些函数名称可能会重复。名称本身可能不会提供太多的意义，因为它们只是代理其他的actions/functions。

```js
handleClick() { require("./actions/doStuff")(/* action stuff */) }
handleMouseEnter() { this.setState({ hovered: true }) }
handleMouseLeave() { this.setState({ hovered: false }) }
```

考虑为你的组件编写一个单独的事件处理程序，并开启 `event.type`。

```js
handleEvent({type}) {
  switch(type) {
    case "click":
      return require("./actions/doStuff")(/* action dates */)
    case "mouseenter":
      return this.setState({ hovered: true })
    case "mouseleave":
      return this.setState({ hovered: false })
    default:
      return console.warn(`No case for event type "${type}"`)
  }
}
```

另外，对于简单的组件，可以使用箭头函数直接从组件中调用导入的actions/functions。

```js
<div onClick={() => someImportedAction({ action: "DO_STUFF" })}
```

在遇到问题之前，不要担心性能优化问题。真的不要。


## 布局组件

布局组件会导致某种形式的静态DOM元素。如果有的话，它可能不需要经常更新。

考虑一个能够渲染两个并排子元素的组件。

```js
<HorizontalSplit
  leftSide={<SomeSmartComponent />}
  rightSide={<AnotherSmartComponent />}
/>
```

我们可以积极地优化这个组件。

虽然 `HorizontalSplit` 将成为这两个组件的父组件，但它永远不会是他们的所有者。我们可以告诉它永远不更新，而不会中断组件的生命周期。

```js
class HorizontalSplit extends React.Component {
  shouldComponentUpdate() {
    return false
  }

  render() {
    <FlexContainer>
      <div>{this.props.leftSide}</div>
      <div>{this.props.rightSide}</div>
    </FlexContainer>
  }
}
```


## 容器组件

“一个容器获取数据，然后渲染其相应的子组件，就是这样。”&mdash;[Jason Bonta](https://twitter.com/jasonbonta)

给定这个可复用的CommentList组件。

```js
const CommentList = ({ comments }) =>
  <ul>
    {comments.map(comment =>
      <li>{comment.body}-{comment.author}</li>
    )}
  </ul>
```

我们可以创建一个新的组件，负责获取数据并渲染无状态的CommentList组件。

```js
class CommentListContainer extends React.Component {
  constructor() {
    super()
    this.state = { comments: [] }
  }

  componentDidMount() {
    $.ajax({
      url: "/my-comments.json",
      dataType: 'json',
      success: comments =>
        this.setState({comments: comments});
    })
  }

  render() {
    return <CommentList comments={this.state.comments} />
  }
}
```

我们可以为不同的应用上下文编写不同的容器。


## 高阶组件

[高阶函数](https://en.wikipedia.org/wiki/Higher-order_function)是一个函数，它接受和/或返回一个函数。这并不比这更复杂。那么，什么是高阶组件？

如果你已经在使用[容器组件](#容器组件), 这些只是封装在函数中的通用容器。

让我们从我们的无状态 `Greeting` 组件开始吧。

```js
const Greeting = ({ name }) => {
  if (!name) { return <div>Connecting...</div> }

  return <div>Hi {name}!</div>
}
```

如果它得到的 `props.name`，它会渲染该数据。否则，会显示"Connecting..."。然后是高阶部分。

```js
const Connect = ComposedComponent =>
  class extends React.Component {
    constructor() {
      super()
      this.state = { name: "" }
    }

    componentDidMount() {
      // this would fetch or connect to a store
      this.setState({ name: "Michael" })
    }

    render() {
      return (
        <ComposedComponent
          {...this.props}
          name={this.state.name}
        />
      )
    }
  }
```

这只是一个返回组件的函数，该函数将渲染作为参数传递的组件。

最后一步，我们需要将我们的 `Greeting` 组件包装在 `Connect` 中。

```js
const ConnectedMyComponent = Connect(Greeting)
```

这是为任何数量的[无状态函数组件](#无状态函数)提供获取和绑定数据的强大模式。

## 状态提升
[无状态函数](#无状态函数)不保存状态（顾名思义）。

事件是状态的变化。
数据要传给有状态的父[容器组件](#容器组件)。

这就是所谓的"状态提升"。它是通过从容器组件传递一个回调到一个子组件来完成的。

```js
class NameContainer extends React.Component {
  render() {
    return <Name onChange={newName => alert(newName)} />
  }
}

const Name = ({ onChange }) =>
  <input onChange={e => onChange(e.target.value)} />
```

`Name`接收来自`NameContainer`的`onChange`回调并调用事件。
上面的`alert`是一个简洁的演示，但它不会改变状态。我们来改变NameContainer的内部状态。

```js
class NameContainer extends React.Component {
  constructor() {
    super()
    this.state = {name: ""}
  }

  render() {
    return <Name onChange={newName => this.setState({name: newName})} />
  }
}
```

通过提供的回调，状态被提升到容器，用于更新本地状态。这设置了一个很好的清晰边界，并最大化了无状态函数的可复用性。

这种模式不限于无状态函数。因为无状态函数没有生命周期事件，所以你也可以在类组件中使用这个模式。

*[受控的input](#受控的input)是了解状态提升使用的重要模式。

*(最好在有状态组件上处理事件对象)*


## 受控的input
摘要中很难谈到受控的input。让我们从一个不受控制的（正常）输入开始，然后从那里开始。

```js
<input type="text" />
```

当你在浏览器中捣鼓这个输入时，你会看到你的改变。这个是正常的。

受控输入不允许使这成为可能的DOM突变。你可以在组件区域中设置输入的值，并且在DOM区域中不会更改

```js
<input type="text" value="This won't change. Try it." />
```

显然，静态输入对用户来说不是很有用。所以，我们从`state`获得`value`。

```js
class ControlledNameInput extends React.Component {
  constructor() {
    super()
    this.state = {name: ""}
  }

  render() {
    return <input type="text" value={this.state.name} />
  }
}
```

然后，改变输入是一个改变组件状态的问题。

```js
    return (
      <input
        value={this.state.name}
        onChange={e => this.setState({ name: e.target.value })}
      />
    )
```

这是一个受控的输入。它只在组件状态发生变化时更新DOM。创建一致的用户界面时，这是非常宝贵的。

*如果你正在使用[无状态函数](#无状态函数)作表单元素，请阅读使用[状态提升](#状态提升)将新状态移动到组件树上。*
