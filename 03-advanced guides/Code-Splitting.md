## # 代码分割

大多数 React 应用都会使用 `webpack`、`Rollup`、`Browserify` 这类的构建工具来打包文件。打包是一个将文件引入并合并到一个单独文件的过程，最终形成一个 "bundle" 。接着在页面上引入该 bundle，整个应用即可一次性加载。

### # `import()` 

在应用中引入代码分割的最佳方式是通过动态 `import()` 语法。

**使用之前：**

```javascript
import { add } from './math'
console.log(add(16, 26))
```

**使用之后：**

```javascript
import('./math').then(math => {
    console.log(math.add(16, 26))
})
```

当 Webpack 解析到该语法时，会自动进行代码分割。如果你使用 Create React App，该功能已开箱即用，你可以立刻使用该特性。

当使用 `Babel` 时，你要确保 Babel 能够解析动态 import 语法而不是将其进行转换。对于这一要求则需要 `babel-plugin-syntax-dynamic-import` 插件。

### # `React.lazy()` 

> **注意：**`React.lazy` 和 `Suspense` 技术还不支持服务端渲染。如果你想要在使用服务端渲染的应用中使用，推荐 `Loadable Components` 这个库。

`React.lazy` 函数能让你像渲染常规组件一样处理动态引入（的组件）。

**使用之前：**

```javascript
import OtherComponent from './OtherComponent'
```

**使用之后：**

```javascript
// 代码会在组件首次渲染时，自动导入包含 `OtherComponent` 组件的包
const OtherComponent = React.lazy(() => import('./OtherComponent'))
```

`React.lazy` 接受一个函数，这个函数需要动态调用 `import()`。它必须返回一个 `Promise`，该 Promise 需要 resolve 一个 `default export` 的 React 组件。

```javascript
// 在 `Suspense` 组件中渲染 lazy 组件，如此使得我们可以使用在等待加载 lazy 组件时做
// 优雅降级（如 loading 指示器等）
const OtherComponent = React.lazy(() => import('./OtherComponent'))
const MyComponent = () => {
    return (
        <div>
            <Suspense fallback={ <div>Loading...</div> }>
                <OtherComponent />
            </Suspense>
        </div>
    )
}
```

`fallback` 属性接受任何在组件加载过程中你想展示的 React 元素。
`Suspense` 组件置于懒加载组件之上的任何位置，甚至可以用一个 `Suspense` 组件包裹多个懒加载组件。

```javascript
const OtherComponent = React.lazy(() => import('./OtherComponent'))
const AnotherComponent = React.lazy(() => import('./AnotherComponent'))
const MyComponent = () => {
    return (
        <div>
            <Suspense fallback={ <div>Loading...</div> }>
                <section>
                    <OtherComponent />
                    <AnotherComponent />
                </section>
            </Suspense>
        </div>
    )
}
```

### # 异常捕获边界 Error boundaries

如果模块加载失败（如网络问题），它会触发一个错误，你可以通过异常捕获边界（`Error boundaries`）技术来处理这些情况，以显示良好的用户体验并管理恢复事宜。

```javascript
import MyErrorBoundary from './MyErrorBoundary'
const OtherComponent = React.lazy(() => import('./OtherComponent'))
const AnotherComponent = React.lazy(() => import('./AnotherComponent'))
// const MyComponent = () => {
//     return (
//         // ...
//     )
// }

// 等效于
const MyComponent = () => (
    <div>
        <MyErrorBoundary>
            <Suspense fallback={ <div>Loading...</div> }>
                <section>
                    <OtherComponent />
                    <AnotherComponent />
                </section>
            </Suspense>
        </MyErrorBoundary>
    </div>
)
```

### # 基于路由的代码分割

> 决定在哪引入代码分割需要一些技巧。你需要确保选择的位置能够均匀地分割代码包而不会影响用户体验。

一个不错的选择是从路由开始。大多数网络用户习惯于页面之间能有个加载切换过程。

```javascript
// 展示如何在应用中使用 `React.lazy` 和 `React Router` 这类的第三方库，来配置基于路由的代码分割。
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom'
import React, { Suspense, lazy } from 'React'

const Home => lazy(() => import('./routes/Home'))
const Abount => lazy(() => import('./routes/About'))

// exact (boolean) 为 true 时，则要求路径与 location.pathname 必须完全匹配
const App = () => {
    <Router>
        <Suspense fallback={ <div>Loading...</div> }>
            <Switch>
                <Route exact path="/" component={ Home } />
                <Route path="/about" component={ About } />
            </Switch>
        </Suspense>
    </Router>
}
```

### # 命名导出 Named Exports

`React.lazy` 目前只支持默认导出（default exports）。如果你想被引入的模块使用命名导出（named exports），你可以创建一个中间模块，来重新导出为默认模块。这能保证 tree shaking 不会出错，并且不必引入不需要的组件。

```javascript
// ManyComponents.js
export const MyComponent = /* ... */
export const MyUnusedComponent = /* ... */
```

```javascript
// MyComponent.js
export { MyComponent as default } from './ManyComponents.js'
```

```javascript
// MyApp.js
import React, { lazy } from 'react'
const MyComponent = lazy(() => import('./MyComponent.js'))
```

