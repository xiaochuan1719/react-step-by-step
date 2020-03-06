# Render Props

> The term “render prop” refers to a technique for sharing code between React components using a prop whose value is a function.

具有 `render prop` 的组件接受一个函数，该函数返回一个 React 元素并调用它而不是实现自己的渲染逻辑。

```javascript
<DataProvider render={ data => (<h1>Hello, { data.target }</h1>) } />
```

**使用 render prop 的库有`React Router`、`Downshift` 以及 `Formik`。**

## Use Render Props for Cross-Cutting Concerns

使用 Render Props 来解决横切关注点（Cross-Cutting Concerns）。

组件是 React 代码复用的主要单元，但如何分享一个组件封装到其他需要相同 state 组件的状态或行为并不总是很容易。

例如：以下组件跟踪 web 应用程序中的鼠标位置：

