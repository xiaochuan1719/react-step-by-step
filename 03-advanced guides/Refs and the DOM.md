# Refs and the DOM

> Refs 提供了一种方式，允许我们访问 DOM 节点或在 render 方法中创建的 React 元素。

在典型的 React 数据流中，props 是父组件与子组件交互的唯一方式，要修改一个子组件，你需要使用新的 props 来重新渲染它。但是，在某些情况下，你需要在典型数据流之外强制修改子组件。被修改的子组件可能是一个 React 组件的实例，也可能是一个 DOM 元素。对于这两种情况，React 都提供了解决方案。

## 何时使用 Refs

- 管理焦点、文本选择或媒体播放

- 触发强制动画

- 集成第三方 DOM 库

**避免使用 refs 来做任何可以通过声明式实现来完成的事情。**

## 创建 Refs

Refs 是使用 `React.createRef()` 创建的，并通过 `ref` 属性附加到 React 元素。在构造组件时，通常将 Refs 分配给实例属性，以便可以在整个组件中引用它们。

**`React.createRef()`是在 React 16.3 版本引入的新的 API 。如果在使用一个较早版本的 React，推荐使用 `回调形式的 refs`。**

```javascript
class MyComponent extends React.Component {
  constructor(props) {
    super(props)
    this.myRef = React.createRef()
  }

  render() {
    return <div ref={this.myRef} />
  }
}
```

## 访问 Refs

当 ref 被传递给 `render` 中的元素时，对该节点的引用可以在 ref 的 `current` 属性中被访问。

```javascript
const node = this.myRef.current
```

ref 的值根据节点的类型而有所不同：

- 当 `ref` 属性用于 HTML 元素时，构造函数中使用 `React.createRef()` 创建的 `ref` 接收底层 DOM 元素作为其 `current` 属性

- 当 `ref` 属性用于自定义 class 组件时，`ref` 对象接收组件的挂载实例作为其 `current` 属性

- **不能在函数组件上使用 ref 属性**，因为它们没有实例

例一：为 DOM 元素添加 ref

```javascript
// 使用 ref 去存储 DOM 节点的引用
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props)
    // 创建一个 ref 来存储 textInput 的 DOM 元素
    this.textInput = React.createRef()
    this.focusTextInput = this.focusTextInput.bind(this)
  }

  focusTextInput() {
    // 通过 "current" 属性访问当前 DOM 节点
    // 直接使用原生 API 使 text 输入框获得焦点
    this.textInput.current.focus()
  }

  render() {
    // 告诉 React 我们想把 <input> ref 关联到构造器里创建的 `textInput` 上
    return (
      <div>
        <input type="text" ref={this.textInput} />
        <input type="button" value="Focus the text input" onClick={this.focusTextInput} />
      </div>
    )
  }
}

ReactDOM.render(<CustomTextInput />, document.querySelector("#app"))
```

> React 会在组件挂载时给 `current` 属性传入 DOM 元素，并在组件卸载时传入 `null` 值。**`ref` 会在 `componentDidMount` 或 `componentDidUpdate` 生命周期钩子触发前更新。**

例二：为 class 组件添加 Ref

```javascript
/// 新版本的 React 使用 `React.createRef()` API

/// 为 class 组件添加 ref

/// 例二：包装 CustomTextInput，模拟它挂载之后立即被点击的操作
/// 使用 ref 来获取这个自定义的 input 组件并手动调用它的 focusTextInput 方法

class CustomTextInput extends React.Component {
  constructor(props) {
    super(props)
    // 创建一个 ref 来存储 textInput 的 DOM 元素
    this.textInput = React.createRef()
    this.focusTextInput = this.focusTextInput.bind(this)
  }

  focusTextInput() {
    // 通过 "current" 属性访问当前 DOM 节点
    // 直接使用原生 API 使 text 输入框获得焦点
    this.textInput.current.focus()
  }

  render() {
    // 告诉 React 我们想把 <input> ref 关联到构造器里创建的 `textInput` 上
    return (
      <div>
        <input type="text" ref={this.textInput} />
        <input type="button" value="Focus the text input" onClick={this.focusTextInput} />
      </div>
    )
  }
}

// 模拟它挂载之后立即被点击的操作
class AutoFocusTextInput extends React.Component {
  constructor(props) {
    super(props)
    this.textInput = React.createRef()
  }

  componentDidMount() {
    // 调用 CustomTextInput 实例的 focusTextInput 方法
    this.textInput.current.focusTextInput()
  }

  render() {
    return <CustomTextInput ref={this.textInput} />
  }
}

ReactDOM.render(<AutoFocusTextInput />, document.querySelector("#app"))
```

```javascript
/// 16.3之前版本的 React 使用回调函数形式的 refs

/// 为 class 组件添加 ref

/// 例二：包装 CustomTextInput，模拟它挂载之后立即被点击的操作
/// 使用 ref 来获取这个自定义的 input 组件并手动调用它的 focusTextInput 方法

class CustomTextInput extends React.Component {
  constructor(props) {
    super(props)
    this.focusTextInput = this.focusTextInput.bind(this)
  }

  focusTextInput() {
    this.textInput.focus()
  }

  render() {
    // Use the `ref` callback to store a reference to the text input DOM
    // element in an instance field (for example, this.textInput).
    return (
      <div>
        <input type="text" ref={input => (this.textInput = input)} />
        <input type="button" value="Focus the text input" onClick={this.focusTextInput} />
      </div>
    )
  }
}

// 模拟它挂载之后立即被点击的操作
class AutoFocusTextInput extends React.Component {
  constructor(props) {
    super(props)
  }

  componentDidMount() {
    console.log("node", this.textInput)
    // 调用 CustomTextInput 实例的 focusTextInput 方法
    // CustomTextInput 必须存在 focusTextInput 方法才行，否则报错：this.textInput.focusTextInput is not a function...
    this.textInput.focusTextInput()
  }

  render() {
    // 该 ref 回调函数会接收到一个挂载的组件实例作为参数
    return <CustomTextInput ref={input => (this.textInput = input)} />
  }
}

ReactDOM.render(<AutoFocusTextInput />, document.querySelector("#app"))
```

例三：Refs 与函数组件

默认情况下，**不能再函数组件上使用 ref 属性**，因为它们没有实例：

```javascript
/// Refs 与 函数组件
/// 默认情况下，不能再函数组件上使用 ref 属性
const MyFunctionComponent = props => {
  return <input type="text" />
}

class Parent extends React.Component {
  constructor(props) {
    super(props)
    this.textInput = React.createRef()
  }

  render() {
    // not work!
    return <MyFunctionComponent ref={this.textInput} />
  }
}

ReactDOM.render(<Parent />, document.querySelector("#app"))

// Warning: Function components cannot be given refs.
// Attempts to access this ref will fail. Did you mean to use React.forwardRef()?
```

如果要在函数组件中使用 `ref`，你可以使用 `forwardRef`（可与 `useImperativeHandle` 结合使用），或者可以将该组件转化为 class 组件。

不管怎样，仍然可以**在函数组件内部使用 ref 属性**，只要它指向一个 DOM 元素或者 class 组件：

```javascript
/// Refs 与 函数组件
/// 可以在函数组件内部使用 ref 属性，只要它指向一个 DOM 元素或者 class 组件
const CustomTextInput = props => {
  // 这里必须声明 textInput，这样 ref 才可以引用它
  let textInput = React.createRef()

  const handleClick = () => {
    textInput.current.focus()
  }

  return (
    <div>
      <input type="text" ref={textInput} />
      <input type="button" value="Focus the text input" onClick={handleClick} />
    </div>
  )
}

ReactDOM.render(<CustomTextInput />, document.querySelector("#app"))
```

## 将 DOM Refs 暴露给父组件

极少数情况下，希望在父组件中引用子节点的 DOM 节点。通常不建议这样做，因为它会打破组件的封装，但它偶尔可用于触发焦点或测量子 DOM 节点的大小或位置。

**可以向子组件添加 ref，但这不是一个理想的解决方案，因为只能获取组件实例而不是 DOM 节点。并且，它还在函数组件上无效。**

如果使用 16.3 或 更高版本的 React，这种情况下推荐使用 `ref 转发`。**Ref 转发使组件可以像暴露自己的 ref 一样暴露子组件的 ref。**

如果使用 16.2 或更低版本的 React，或者需要比 ref 转发更高的灵活性，可以使用 **[替代方案]()** 将 ref 作为特殊名字的 prop 直接传递。

不建议暴露 DOM 节点，但有时它会成为救命稻草。注意这个方案需要你在子组件中增加一些代码。如果你对子组件的实现没有控制权的话，你剩下的选择是使用 `findDOMNode()`，但在严格模式 下已被废弃且不推荐使用。

## 回调 Refs

在 16.3 或 更高版本的 React 中，通过 `this.myRef = React.createRef()` 设置 refs；而在 16.2 或 更低版本的 React 中是通过 `回调 refs` 来设置 refs 的，在高版本中依旧可用，它能更精细地控制何时 refs 被设置和解除。

不同于传递 `createRef()` 创建的 `ref` 属性，你会传递一个函数。这个函数中接收 React 组件实例或 HTML DOM 元素作为参数，以使它们能在其他地方被存储和访问。

```javascript
/// Callback Refs

/// 在 RefsAndDomExample04.html 示例中已展示 Callback Refs 的使用

class CustomTextInput extends React.Component {
  constructor(props) {
    super(props)

    this.textInput = null

    this.setTextInputRef = element => {
      this.textInput = element
    }

    this.focusTextInput = () => {
      // Focus the text input using the raw DOM API
      if (this.textInput) this.textInput.focus()
    }
  }

  componentDidMount() {
    // autofocus the input on mount
    this.focusTextInput()
  }

  render() {
    // Use the `ref` callback to store a reference to the text input DOM
    // element in an instance field (for example, this.textInput).
    return (
      <div>
        <input type="text" ref={this.setTextInputRef} />
        <input type="button" value="Focus the text input" onClick={this.focusTextInput} />
      </div>
    )
  }
}

ReactDOM.render(<CustomTextInput />, document.querySelector("#app"))
```

React 将在组件挂载时，会调用 `ref` 回调函数并传入 DOM 元素，当卸载时调用它并传入 `null`。在 `componentDidMount` 或 `componentDidUpdate` 触发前，React 会保证 refs 一定是最新的。

**同样，在组件间传递回调形式的 refs，就像传递通过 `React.createRef()` 创建的对象 refs 一样。**

```javascript
const CustomTextInput = props => {
  return (
    <div>
      <input ref={props.inputRef} />
    </div>
  )
}

class Parent extends React.Component {
  render() {
    return <CustomTextInput inputRef={el => (this.inputElement = el)} />
  }
}
```

## 关于回调 refs 的说明

如果 `ref` 回调函数是以内联函数的方式定义的，在更新过程中它会被执行两次，第一次传入参数 `null`，然后第二次会传入参数 DOM 元素。这是因为在每次渲染时会创建一个新的函数实例，所以 React 清空旧的 ref 并且设置新的。

通过将 ref 的回调函数定义成 class 的绑定函数的方式可以避免上述问题，但是大多数情况下它是无关紧要的。
