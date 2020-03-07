# Render Props

> The term “render prop” refers to a technique for sharing code between React components using a prop whose value is a function.

具有 `render prop` 的组件接受一个函数，该函数返回一个 React 元素并调用它而不是实现自己的渲染逻辑。

```javascript
<DataProvider render={data => <h1>Hello, {data.target}</h1>} />
```

**使用 render prop 的库有`React Router`、`Downshift` 以及 `Formik`。**

## Use Render Props for Cross-Cutting Concerns

使用 Render Props 来解决横切关注点（Cross-Cutting Concerns）。

组件是 React 代码复用的主要单元，但如何分享一个组件封装到其他需要相同 state 组件的状态或行为并不总是很容易。

例如：以下组件跟踪 web 应用程序中的鼠标位置：

```javascript
// 定义一个组件，跟踪 WEB 应用程序中的鼠标位置
class MouseTracker extends React.Component {
  constructor(props) {
    super(props)
    this.state = {
      x: 0,
      y: 0
    }
    this.handleMouseMove = this.handleMouseMove.bind(this)
  }

  handleMouseMove(e) {
    this.setState({
      x: e.clientX,
      y: e.clientY
    })
  }

  render() {
    return (
      <div style={{ height: "100%" }} onMouseMove={this.handleMouseMove}>
        <h1>移动鼠标</h1>
        <p>
          当前的鼠标位置是（{this.state.x}，{this.state.y}）
        </p>
      </div>
    )
  }
}

ReactDOM.render(<MouseTracker />, document.querySelector("#app"))
```

render props：提供一个带有函数 prop 的 `<Mouse>` 组件，它能够动态决定什么需要渲染的，而不是将 `<Cat>` 硬编码到 `<Mouse>` 组件里，并有效地改变它的渲染结果。

```javascript
class Cat extends React.Component {
  render() {
    const mouse = this.props.mouse
    return <img className="catImage" src="./cat.jpg" style={{ position: "absolute", left: mouse.x, top: mouse.y }} />
  }
}

class Mouse extends React.Component {
  constructor(props) {
    super(props)
    this.state = {
      x: 0,
      y: 0
    }
    this.handleMouseMove = this.handleMouseMove.bind(this)
  }

  handleMouseMove(e) {
    // +30 的目的是为了避免类似鼠标拖动图片的效果
    this.setState({
      x: e.clientX + 30,
      y: e.clientY + 30
    })
  }

  render() {
    return (
      <div style={{ height: "100%" }} onMouseMove={this.handleMouseMove}>
        {/* 
            Instead of providing a static representation of what <Mouse> renders,
            use the `render` prop to dynamically determine what to render. 
        */}
        {this.props.render(this.state)}
      </div>
    )
  }
}

class MouseTracker extends React.Component {
  render() {
    return (
      <div>
        <h1>Move the mouse around</h1>
        <Mouse render={mouse => <Cat mouse={mouse} />} />
      </div>
    )
  }
}

ReactDOM.render(<MouseTracker />, document.querySelector("#app"))
```

更具体地说，**render prop是一个用于告知组件需要渲染什么内容的函数 prop。**

这项技术使我们共享行为非常容易。要获得这个行为，只要渲染一个带有 `render` prop 的 `<Mouse>` 组件就能够告诉它当前鼠标坐标（x, y）要渲染什么。

关于 render prop 一个有趣的事情：可以使用带有 `render` prop 的常规组件来实现大多数**高阶组件（HOC）**。

```javascript
// 如果你出于某种原因真的想要 HOC，那么你可以轻松实现使用具有 render prop 的普通组件创建一个
const withMouse = (Component) => {
    return class extends React.Component {
        render() {
            return (
                <Mouse render={mouse => (
                    <Component {...this.props} mouse={mouse} />
                )} />
            )
        }
    }
}
```

因此，可以将任一模式与 render prop 一起使用。

## 使用 Props 而非 render

render prop 是因为模式才被称为 `render prop`，所以不一定要用名为 `render` 的 prop 来使用这种模式。事实上，**任何被用于告知组件需要渲染什么内容的函数 prop 在技术上都可以被称为“render prop”。**

之前使用了 `render`，这里同样可以简单地使用 `children` prop。

```javascript
<Mouse children={ mouse => (
    <p>鼠标的位置是 {mouse.x}, {mouse.y}</p>
)} />
```

Important: `children` prop 并不真正需要添加到 JSX 元素的 “attributes” 列表中。相反，可以直接放置到元素的内部！

```javascript
<Mouse>
    { mouse => (<p>鼠标的位置是 {mouse.x}, {mouse.y}</p>)}
</Mouse>
```

[react-motion](https://github.com/chenglou/react-motion) 的 API 中应用了此技术。