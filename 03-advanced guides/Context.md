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
                <Toolbar /> 
            <ThemedContext.Povider>
        )
    }
}
```

### # 使用 Context 之前的考虑

Context 主要应用场景在于很多不同层级的组件需要访问同样一些的数据。需谨慎使用，因为这会使得组件的复用性变差。

**如果只是想避免层层传递一些属性，组件组合（`component composition`）可能相比 context 是一个更好的解决方案。**

```javascript
<Page user={user} avatarSize={avatarSize} />
// ... 渲染出 ...
<PageLayout user={user} avatarSize={avatarSize} />
// ... 渲染出 ...
<NavigationBar user={user} avatarSize={avatarSize} />
// ... 渲染出 ...
<Link href={user.permalink}>
    <Avatar user={user} size={avatarSize} />
</Link>
```

如果在最后只有 `Avatar` 组件真的需要 `user` 和 `avatarSize`，那么层层传递这两个 props 就显得非常冗余。而且一旦 `Avatar` 组件需要更多从来自顶层组件的 props，你还得在中间层级一个一个加上去，这将会变得非常麻烦。

一种**无需 context**的解决方案就是将 `Avatar` 组件自身传递下去，因而中间组件无需知道 `user` 或 `avatarSize` 等 props：

```javascript
const Page = (props) => {
    const user = props.user
    const userLink = (
        <Link href={user.permalink}>
            <Avatar user={user} size={props.avatarSize} />
        </Link>
    )
    return <PageLayout userLink={userLink} />
}

// 现在，我们有这样的组件
<Page user={user} avatarSize={avatarSize} />
// ... 渲染出 ...
<PageLayout userLink={...} />
// ... 渲染出 ...
<NavigationBar userLink={...} />
// ... 渲染出 ...
{props.userLink}
```

这种变化下，只有最顶部的 Page 组件需要知道 `Link` 和 `Avatar` 组件是如何使用 `user` 和 `avatarSize` 的。

这种对组件的**控制反转**减少了在你的应用中要传递的 props 数量，这在很多场景下会使得你的代码更加干净，使你对根组件有更多的把控。但是，这并不适用于每一个场景：这种将逻辑提升到组件树的更高层次来处理，会使得这些高层组件变得更复杂，并且会强行将低层组件适应这样的形式，这可能不是我们想要的。

### # Context API

#### # React.createContext 

```javascript
const MyContext = React.createContext(defaultValue)
```

创建一个 Context 对象。当 React 渲染一个订阅了这个 Context 对象的组件，这个组件会从组件树中离自身最近的那个匹配的 `Provider` 中读取到当前的 context 值。

**只有**当组件所处的树中没有匹配 Provider 时，其 `defaultValue` 参数才会生效。这有助于在不使用 Provider 包装组件的情况下对组件进行测试。**注意：将 `undefined` 传递给 Provider 的 value 时，消费组件的 `defaultValue` 不会生效**。

#### # Context.Provider 

```javascript
<MyContext.Provider value={ /* 某个值 */ } />
```

每个 Context 对象都会返回一个 Provider React 组件，它允许消费组件订阅 context 的变化。

Provider 接收一个 `value` 属性，传递给消费组件。一个 Provider 可以和多个消费组件有对应关系。多个 Provider 也可以嵌套使用，里层的会覆盖外层的数据。

当 Provider 的 `value` 值发生变化时，它内部的所有消费组件都会重新渲染。Provider 及其内部 consumer 组件都不受制于 `shouldComponentUpdate` 函数，因此当 consumer 组件在其祖先组件退出更新的情况下也能更新。

通过新旧值检测来确定变化，使用了与 `object.is` 相同的算法。

> **注意：**当传递对象给 value 时，检测变化的方式会导致一些问题：见注意事项

#### # Class.contextType

```javascript
class MyClass extends React.Component {
    componentDidMount() {
        let value = this.context
        // 在组件挂载完后，使用 MyContext 组件的值来执行一些有副作用的操作
    }

    componentDidUpdate() {
        let value = this.context
        // ...
    }

    componentWillUnmount() {
        let value = this.context
        // ...
    }

    render() {
        let value = this.context
        // 基于 MyContext 组件的值进行渲染
    }
}
MyClass.contextType = MyContext
```

挂载在 class 上的 `contextType` 属性会被重新赋值为一个由 `React.createContext()` 创建的 Context 对象。通过使用 `this.context` 来消费最近 Context 上的那个值。可以在任何生命周期中访问到它，包括 render 函数中。

> **注意：**通过该 API 订阅单一 context。
> 
> 如果在使用实验性的 [public class fields 语法](https://babeljs.io/docs/plugins/transform-class-properties/)，你可以使用 `static` 这个类属性来初始化 contextType.

```javascript
class MyClass extends React.Component {
    static contextType = MyContext
    render() {
        let value = this.context
        // 基于这个值进行渲染工作
    }
}
```

#### # Context.Consumer

```javascript
<MyContext.Consumer>
    { value => /* render something based on the context value */ }
</MyContext.Consumer>
```

React 组件也可以订阅到 context 变更。这能让我们在**函数式组件**中完成订阅 context。

这需要**函数作为子元素（function as a child）**这种做法。这个函数接收当前的 context 值，返回一个 React 节点。传递给函数的 `value` 值等同于往上组件树离这个 context 最近的 Provider 提供的 `value` 值。如果没有对应的 Provider，`value` 参数等同于传递给 `createContext()` 的 `defaultValue`。

#### # Context.displayName

context 对象接受一个名为 `displayName` 的 property，类型为字符串。React DevTools 使用该字符串来确定 context 要显示的内容。

```javascript
const MyContext = React.createContext(/* some value */)
MyContext.displayName = 'MyDisplayName'

<MyContext.Provider />  // "MyDisplayName.Provider" 在 DevTools 中
<MyContext.Consuer />   // "MyDisplayName.Consumer" 在 DevTools 中
```

### # 示例

#### # 动态 Context

使用动态值（dynamic values）后更复杂的用法：

```javascript
// theme-context.js
export const themes = {
    light: {
        foreground: '#000',
        background: '#eee'
    },
    dark: {
        foreground: '#fff',
        background: '#222'
    }
}

export const ThemeContext = React.createContext(themes.dark)  // 默认值为 themes.dark
```

```javascript
// themed-button.js
import { ThemeContext } from './theme-context'
class ThemedButton extends React.Component {
    render() {
        let props = this.props
        let theme = this.context
        return (
            <button {...props} style={{backgroundColor: theme.background}} />
        )
    }
}
ThemedButton.contextType = ThemeContext
export default ThemedButton
```

```javascript
import { ThemeContext, themes } from './theme-context'
import ThemedButton from './themed-button'

// 一个使用 ThemedButton 的中间组件
const Toolbar = props => {
    return (
        <ThemedButton onClick={ props.changeTheme }>
            Change Theme
        </ThemedButton>
    )
}

class App extends React.Component {
    constructor(props) {
        super(props)
        this.state = {
            theme: themes.light
        }

        this.toggleTheme = () => {
            this.setState(state => ({
                theme: state.theme === themes.dark ? themes.light : themes.dark
            }))
        }
    }

    render() {
        // 在 ThemeProvider 内部的 ThemedButton 按钮组件使用 state 中的 theme 值，
        // 而外部的组件使用默认的 theme 值
        return (
            <Page>
                <ThemeContext.Provider value={ this.state.theme }>
                    <Toolbar changeTheme={ this.toggleTheme } />
                </ThemeContext.Provider>
            </Page>
        )
    }
}

ReactDOM.render(<App />, document.querySelector('#app'))
```