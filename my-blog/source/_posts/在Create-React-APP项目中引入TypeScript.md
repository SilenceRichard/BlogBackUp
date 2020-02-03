---
title: 在Create-React-APP项目中引入TypeScript
date: 2020-02-03 19:19:16
tags:
  - 工具链
---

[参考博文](https://www.cnblogs.com/vvjiang/p/11944912.html)

> 把大象放入冰箱需要三步

# 打开冰箱门

```bash
  tsc --init # 生成tsconfig.json
```

以下是 tsconfig.json 的配置

```json
{
  "compilerOptions": {
    "declaration": true,
    "baseUrl": ".",
    "paths": {
      "utils/*": ["include/utils/*"],
      "stylesheets": ["stylesheets/"],
      "examples": ["examples/"]
    },
    "rootDirs": ["include"],
    "outDir": "dist",
    "module": "esnext",
    "target": "es5",
    "lib": ["es6", "dom"],
    "sourceMap": true,
    "jsx": "react",
    "moduleResolution": "node",
    "rootDir": ".",
    "forceConsistentCasingInFileNames": true,
    "noImplicitReturns": true,
    "noImplicitThis": true,
    "noImplicitAny": true,
    "importHelpers": true,
    "strictNullChecks": true,
    "esModuleInterop": true,
    "noUnusedLocals": true
  },
  "include": ["src/**/*"],
  "exclude": [
    "node_modules",
    "build",
    "dist",
    "scripts",
    "acceptance-tests",
    "webpack",
    "jest",
    "src/setupTests.ts",
    "*.js"
  ]
}
```

```bash
# 安装ts相关依赖
yarn add @types/react @types/react-dom typescript awesome-typescript-loader
```

# 放入大象

配置 webpack.config.js

```js
// 关键配置， 新增loader
{
  test: /\.tsx?$/,
  loader: 'awesome-typescript-loader',
},
```

# 关冰箱门

在项目中创建.tsx/.ts文件，使用typescript开发

# 装饰冰箱

## 在eslint中配置ts
[参考文章](https://juejin.im/entry/5a156adaf265da43231aa032)
```bash
# 安装依赖
yarn add @typescript-eslint/parser @typescript-eslint/eslint-plugin
```
配置.eslintrc.js
```js
  parser: '@typescript-eslint/parser',
  plugins: ['@typescript-eslint'],
  rules: {
    // 禁止使用 var
    'no-var': "error",
    // 优先使用 interface 而不是 type
    '@typescript-eslint/consistent-type-definitions': [
        "error",
        "interface"
    ]
  }
```
over~