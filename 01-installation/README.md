# Chapter01 Introduction

> React is a JavaScript library for building user interfaces.

## 开发环境配置

- **在 HTML 页面中引入 `React` 和 `ReactDOM` 的 UMD 版本，创建一个简单的 react 应用**

```javascript
<script crossorigin src="https://unpkg.com/react@16/umd/react.development.js"></script>
<script crossorigin src="https://unpkg.com/react-dom@16/umd/react-dom.development.js"></script>
<script src="https://unpkg.com/browse/babel-standalone@7/babel.min.js"></script>
// <script src="https://cdn.jsdelivr.net/npm/react@16.12.0/umd/react.development.js"></script>
// <script src="https://cdn.jsdelivr.net/npm/react-dom@16.12.0/umd/react-dom.development.js"></script>
// <script src="https://cdn.jsdelivr.net/npm/@babel/standalone@7.8.4/babel.min.js"></script>
```

```javascript
<script crossorigin src="https://unpkg.com/react@16/umd/react.production.min.js"></script>
<script crossorigin src="https://unpkg.com/react-dom@16/umd/react-dom.production.min.js"></script>
<script src="https://unpkg.com/babel-standalone@7/babel.min.js"></script>
```

当然,也可以不通过 CDN 的方式引入；也可以将相关 js 库下载下来，直接通过路径的方式引入到 html 页面中亦可。

- **通过官方提供的 `create-react-app` 集成工具创建一个 react 应用**

```shell
npx create-react-app my-app
cd my-app
npm start
```

```shell
# npm 6+  npm init <initializer>
npm init react-app my-app
```

```shell
# Yarn 0.25+  yarn create <starter-kit-package>
yarn create react-app my-app
```

- **通过 webpack/Parcel/rollup 及相关工具自己搭建一个 React 开发环境**

推荐 webpack 打包工具。