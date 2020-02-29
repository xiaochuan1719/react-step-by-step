## # Context

> Context 提供了一个无需为每层组件手动添加 props，就能在组件树间进行数据传递的方法

在一个典型的 React 应用中，数据是通过 props 属性自上而下（由父及子）进行传递的，但这种做法对于某些类型的属性而言是及其繁琐的（如：地区偏好、UI主题等），这些属性是应用程序中许多组件都需要的。`Context` 提供了一种在组件之间共享此类值的方式，而不必显式地通过组件树的逐层传递 `props`。

### # 何时使用 `Context`

`Context` 设计目的是为了共享那些对于一个组件树而言是 “全局” 的数据，如当前认证的用户、主题或首选语言。

```javascript
// 举个栗子：通过一个 “theme” 属性手动调整一个按钮组件的样式
const Toolbar = (props) => {
    // Toolbar 组件接受一个额外的 “theme” 属性，然后传递给 ThemedButton 组件。
    // 如果应用中每一个单独的按钮都需要知道 theme 的值，这会是件很麻烦的事，
    // 因为必须将这个值层层传递所有组件
    return (
        <div>
            <ThemedButton theme={ props.theme } />
        </div>
    )
}

class ThemedButton extends React.Component {
    render() {
        return <Button theme={ this.props.theme } />
    }
}

class App extends React.Component {
    render() {
        return <Toolbar theme="dark" />
    }
}
```

props 层层传递给子组件；而使用 `context` 可以避免通过中间元素传递 props:

```javascript
// Context 可以让我们无须明确地传遍每一个组件，就能将值深入传递进组件树。
// 为当前的 theme 创建一个 context ("light" 为默认值)
const ThemedContext = React.createContext("light")

// 中间的组件再也不必指明往下传递 theme 了
const Toolbar = (props) => {
    return (
        <div>
            <ThemedButton />
        </div>
    )
}

class ThemedButton extends React.Component {
    // 指定 contextType 读取当前的 theme context
    // React 会往上找到最近的 theme Provider，然后使用它的值
    // 在这个例子中，当前的 theme 值为 “dark”
    static contextType = ThemeContext
    render() {
        return <Button theme={ this.context } />
    }
}

class App extends React.Component {
    render() {
        // 使用一个 Provider 来将当前的 theme 传递给以下的组件树
        // 无论多深，任何组件都能读取这个值
        // 在这个例子中，我们将 “dark” 作为当前的值传递下去
        return (
            <ThemedContext.Provider value="dark">
                <Toolbar>
            <ThemedContext.Povider>
        )
    }
}
```

### # 使用 Context 之前的考虑

Context 主要应用场景在于很多不同层级的组件需要访问同样一些的数据。需谨慎使用，因为这会使得组件的复用性变差。

**如果只是想避免层层传递一些属性，组件组合（`component composition`）可能相比 context 是一个更好的解决方案。**

