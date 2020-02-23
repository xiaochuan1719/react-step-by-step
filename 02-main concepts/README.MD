# MAIN CONCEPTS

## Hello World

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <style type="text/css">
        #app {
            color: #61dafb;
        }
    </style>
</head>
<body>
    <div id="app"></div>

    <!-- 
    <script crossorigin src="https://unpkg.com/react@16.12.0/umd/react.development.js"></script>
    <script crossorigin src="https://unpkg.com/react-dom@16.12.0/umd/react-dom.development.js"></script> 
    <script crossorigin src="https://unpkg.com/babel-standalone@7.0.0-beta.3/babel.min.js"></script>    
    -->
    <script src="https://cdn.jsdelivr.net/npm/react@16.12.0/umd/react.development.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/react-dom@16.12.0/umd/react-dom.development.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/@babel/standalone@7.8.4/babel.min.js"></script>
    <script type="text/babel">
        // 1. 虚拟 DOM 元素对象，不是字符串；
        // 重要：没有报错的原因在于该脚本 type 为 “text/babel”，
        // "text/babel" 声明，告知 babel.js 解析里面的代码，前提使需要引入 babel.js 文件
        const vDom = <h1>Hello, world!</h1>;
        // 2. 将 虚拟 DOM 元素对象渲染到页面真实 DOM 容器中
        ReactDOM.render(vDom, document.querySelector('#app'))
    </script>
</body>
</html>
```

## Introducing JSX

```js
// The tag syntax is neither a string nor html
const element = <h1>Hello, world!</h1>
```

> JSX, that is a syntax extension to JavaScript. 

JSX 可以很好的描述 UI 应该呈现出它应有交互的本质形式。

### Why JSX

React 认为渲染逻辑本质上与其他 UI 逻辑内在耦合，比如，在 UI 中需要绑定处理事件、在某些时刻状态发生变化时需要通知到 UI，以及需要在 UI 中展示准备好的数据。

### Embedding Expressions in JSX

```javascript
const name = 'Nicholas'
const element = <h1>Hello, {name}</h1>

ReactDOM.render(element, document.querySelector('#app'))
```

在 JSX 语法中，你可以在大括号内放置任何有效的 JavaScript 表达式。

```javascript
// 在 JSX 中嵌入表达式
function formatName(user) {
    return user.firstName + ' ' + user.lastName
}

const user = {
    firstName: 'Nicholas',
    lastName: 'Eee'
}

const vDomEle = (
    <h1>
        Hello, {formatName(user)}
    </h1>
)

ReactDOM.render(vDomEle, document.querySelector('#app'))
```

### JSX is an Expression Too

> After compilation, JSX expressions become regular JavaScript function calls and evaluate to JavaScript objects.

```javascript
// 可以在 if 语句和 for 语句循环的代码块中使用 JSX
function getGreeting(user) {
    if (user) {
        return <h1>Hello, {formatName(user)}!</h1>
    } else {
        return <h1>Hello, Nicholas.</h1>
    }
}
```

### Specifying Attributes with JSX

```javascript
// You can use quotes to specify string literals as attributes
const element = <div tabIndex="0"></div>
```

```javascript
// You may also use curly braces to embed a JavaScript expression in an attribute
const element = <img src={user.avatarUrl} />
```

> Don’t put quotes around curly braces when embedding a JavaScript expression in an attribute. You should either use quotes (for string values) or curly braces (for expressions), but not both in the same attribute.

**重要** 由于 JSX 的语法更加接近 JavaScript 而不是 HTML，所以 ReactDOM 使用 `camelCase`（小驼峰命名）来定义属性的名称，而不使用 HTML 属性名称的命名约定。

比如：`class` -> `className`、`tabindex` -> `tabIndex`

### Specifying Children with JSX

> If a tag is empty, you may close it immediately with `/>`，like XML:

```javascript
const element = <img src={user.avatarUrl}" />
```

> JSX tags may contain chilren:

```javascript
<div>
    <h1>Hello, {formatName(user)}</h1>
    <img src={user.avatarUrl} alt="avatar" />
</div>
```

> React 中的一个常见模式是一个组件返回多个元素。Fragments 允许你将子列表分组，而无需向 DOM 添加额外节点。

> `<></>` 是 `<React.Fragment></React.Fragment>` 的 短语法；但不支持 key 或 属性

```javascript
<Fragment>
    <h1>Hello, {formatName(user)}</h1>
    <img src={user.avatarUrl} alt="avatar" />
</Fragment>

// 等效于

<>
    <h1>Hello, {formatName(user)}</h1>
    <img src={user.avatarUrl} alt="avatar" />
</>
```

### JSX Prevents Injection Attacks

可以安全地在 JSX 当中插入用户输入的内容：

```javascript
const title = response.potentiallyMaliciousInput;
// 直接使用是安全的，放心大胆使用吧 ~
const element = <h1>{title}</h1>
```

ReactDOM 在渲染所有输入内容之前，默认会进行转义。它可以确保在你的应用中，永远不会注入那些并非自己明确编写的内容。所有的内容在渲染之前都被转义成了字符串；可有效防止 `XSS (cross-site-scripting 跨站脚本攻击)` 攻击

### JSX Represents Objects 

> Babel compiles JSX down to `React.createElement()` calls.

Babel 会把 JSX 转译成一个名为 `React.createElement()` 函数调用。

也就是说：

```javascript
const element = (
    <h1 className="greeting">
        Hello, world
    </h1>
)

// 等效于

const element = React.createElement('h1', {className: 'greeting'}, 'Hello, world')

// 实际上会创建类似这样的一个对象：
// 注意：这里只是简单地描述这个 JSX 对象的结构
const element = {
    type: 'h1',
    props: {
        className: 'greeting',
        children: 'Hello, world'
    }
}
```


