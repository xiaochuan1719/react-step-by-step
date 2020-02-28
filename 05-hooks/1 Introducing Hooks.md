# Hook 简介

> Hook 是 React 16.8 版本开始新增的特性。它可以让你在不编写 class 的情况下使用 state 以及其他的 React 特性。

```javascript
import React, { useState } from 'react'
const Exampe = () => {
    // 声明一个新的叫做 ”count“ 的 state 变量
    const [count, setCount] = useState(0)
    
    return (
        <div>
            <p>You clicked { count } times</p>
            <button onClick={ () => setCount(count + 1) }>
                Click Me
            </button>
        </div>
    )
}
```