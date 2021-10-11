1. yarn init -y 初始化项目
1. 创建测试目录 写下第一个测试
1. 解决 jest 在 ts 下的环境
1. `npx tsc --init` (前提是安装了 typescript)
1. `yarn add jest @types/jest --dev`​
1. 在 tsconfig.json type 字段中加上 jest
1. 在 ts 文件中导出一个函数，在 tsconfig 中 noImplicitAny 设置为 false 可以忽略类型报错
1. 配置规范 babel 因为 jest 是运行在 nodejs 不支持 esm
1. 使用 Babel<br />如果需要使用 Babel，可以通过 yarn 来安装所需的依赖。

```js
yarn add --dev babel-jest @babel/core @babel/preset-env
```

<br />可以在工程的根目录下创建一个 babel.config.js 文件用于配置与你当前 Node 版本兼容的 Babel：<br />

```js
// babel.config.js

module.exports = {
  presets: [['@babel/preset-env', { targets: { node: 'current' } }]]
}
```

10. 使用 TypeScript

Jest 可以通过 Babel 支持 TypeScript。 首先，在项目中正确的使用 Babel。 其次，使用 Yarn 或者 Npm 安装 @babel/preset-typescript

```js
yarn add --dev @babel/preset-typescript
```

```js
// babel.config.js

module.exports = {
  presets: [['@babel/preset-env', { targets: { node: 'current' } }], '@babel/preset-typescript']
}
```

<br />11. `yarn test` 完成项目搭建
