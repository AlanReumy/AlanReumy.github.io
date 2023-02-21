# babel 深入解析

![20220616160008](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220616160008.png)

## 为什么需要 Babel

在开发中我们很少直接去接触 `Babel` ，但是 `Babel` 对于前端开发来说，目前是不可缺少的一部分：

- 开发中，我们想要使用 `ES6+` 的语法，想要使用 `TypeScript` ，开发 `React` 项目，它们都是离不开 `Babel` 的
- 学习 `Babel` 对于我们理解代码从编写到线上的转变过程直观重要

`Babel` 是一个工具链，主要用于旧浏览器或者缓解中将 `ECMAScript 2015+` 代码转换为向后兼容版本的，它的功能包括：语法转换、源代码转换、`Polyfill` 等

## Babel 命令行使用

`Babel` 本身可以作为一个独立的工具，不和 `webpack` 等构建工具配置来单独使用

如果我们希望在命令行尝试使用 `Babel` ，需要安装如下库：

- @babel/core：`Babel` 的核心代码，必须安装
- @babel/cli：可以让我们在命令行使用 `Babel`

```bash
npm install @babel/cli @babel/core
```

使用 `Babel` 来处理我们的源代码：

```bash
npx babel src --out-dir dist
```

其中：
- src：是源文件目录
- --out-dir：是指定要输出到的文件夹

## 插件的使用

比如我们需要转换箭头函数，那么我们就可以使用箭头函数转换相关的插件：

```bash
npm install @babel/plugin-transform-arrow-functions -D
```

```bash
npx babel src --out-dir dist --plugins=@babel/plugin-transform-arrow-functions
```

## Babel 预设

如果要转换的内容过多，一个个设置是比较麻烦的，我们可以使用预设（preset）：

- 安装@babel/preset-env预设：
  ```bash
  npm install @babel/preset-env -D
  ```
- 执行如下命令
  ```bash
  npx babel src --out-dir dist --presets=@babel/preset-env
  ```

## Babel 底层原理

`Babel` 是如何做到将我们的一段代码（`ES6、TypeScript、React`）转成另外一段代码（`ES5`）的呢

我们可以将 `Babel` 看作编译器，它的作用就是将我们的源代码转换成浏览器可以直接识别的另外一种源代码

它的工作流程如下：

- 解析（Parsing）
- 转换（Transformation）
- 生成（Code Generation）

## Babel 编译器执行原理

![20220616144934](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220616144934.png)

tokens数组：

![20220616145223](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220616145223.png)

根据 tokens数组 生成 AST抽象语法树：


![20220616150119](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220616150119.png)

每一个对象都对应一个节点

通过遍历，访问节点，再对每个节点应用 `Babel` 插件，最后生成新的 AST抽象语法树后，生成源代码

## babel-loader

在实际开发中，我们通常会在构建工具中通过配置 `Babel` 来对其进行使用的，比如在 `webpack` 中

我们需要添加配置：

```js
{
  test: /\.js$/,
  use: {
    loader: "babel-loader",
    options: {
      // 指定插件
      plugins: [
        "@babel/plugin-transform-arrow-functions",
        "@babel/plugin-transform-block-scoping"
      ]
      // 指定预设
      presets: ["@babel/preset-env"]
    }
  }
}
```

## 设置目标浏览器 browserslist

我们最终打包的JavaScript代码，是需要跑在目标浏览器上的，那么如何告知babel我们的目标浏览器呢？

- browserslist 工具
- target 属性

通过设置 target 属性：

```js
  use: {
    loader: "babel-loader",
    options: {
      presets: [
        ["@babel/preset-env", {
          targets: ["chrome 88"]
        }]
      ],
      plugins: [
        "@babel/plugin-transform-arrow-functions",
        "@babel/plugin-transform-block-scoping"
      ]
    }
  }
```

## Stage-X的preset

- 我们需要先了解一下TC39的组织：
  - TC39 是指技术委员会（Technical Committee）第 39 号
  它是 ECMA 的一部分，ECMA 是 `ECMAScript` 规范下的 `JavaScript` 语言标准化的机构
  - `ECMAScript` 规范定义了 `JavaScript` 如何一步一步的进化、发展

- TC39 遵循的原则是：分阶段加入不同的语言特性，新流程涉及四个不同的 Stage
  - **Stage 0**：strawman（稻草人），任何尚未提交作为正式提案的讨论、想法变更或者补充都被认为是第 0 阶段的"稻草人"
  - **Stage 1**：proposal（提议），提案已经被正式化，并期望解决此问题，还需要观察与其他提案的相互影响；
  - **Stage 2**：draft（草稿），Stage 2 的提案应提供规范初稿、草稿。此时，语言的实现者开始观察 runtime 的具体实现是否合理
  - **Stage 3**：candidate（候补），Stage 3 提案是建议的候选提案。在这个高级阶段，规范的编辑人员和评审人员必须在最终规范上签字。Stage 3 的提案不会有太大的改变，在对外发布之前只是修正一些问题
  - **Stage 4**：finished（完成），进入 Stage 4 的提案将包含在 ECMAScript 的下一个修订版中

## Babel Stage-X设置

在babel7之前（比如babel6中），我们会经常看到这种设置方式：

- 它表达的含义是使用对应的 `babel-preset-stage-x` 预设
- 但是从 `Babel7` 开始，已经不建议使用了，建议使用 `preset-env` 来设置

  ```js
  module.exports = {
    "preset": ["stage-0"]
  }
  ```

## Babel 配置文件

- 像之前一样，我们可以将 `Babel` 的配置信息放到一个独立的文件中，`Babel` 给我们提供了两种配置文件的编写
  - babel.config.json（或者.js,.cjs,.mjs）文件
  - .babelrc.json（或者.babelrc,.js,.cjs,.mjs）文件

- 它们两个有什么区别呢？目前很多的项目都采用了多包管理的方式（Babel本身、element-plus、umi等）
  - .babelrc.json：早期使用较多的，但是对于 Monorepos 项目配置比较麻烦
  - babel.config.json：可以直接作用于 Monorepos 项目的子包，更加推荐

```js
module.exports = {
  presets: [
    ["@babel/preset-env"],
  ],
}
```

## 认识 polyfill

- 比如我们使用了一些语法特性，例如：Promise、Generator、Symbol等
- 但是某些浏览器压根不认识这些特性
- 我们可以使用 `polyfill` 来打补丁

## 如何使用 polyfill

安装入 `core-js` 和 `regenerator-runtime`

```bash
npm install core-js regenerator-runtime --save
```

我们在配置的时候，建议排除 `node-modules` 下的文件

原因是：有些依赖包里可能已经自己做了 `polyfill`，如果不排除的话，可能会引起 `polyfill` 的冲突

```js
{
  test: '/\.m?js$/,
  exclude: /node-modules/,
  use: 'babel-loader',
}
```

我们需要在 `babel.config.js` 文件中，给 `preset-env` 配置一些属性：

- useBuiltIns：设置以什么样的方式来使用 `polyfill`
- corejs：设置 `corejs` 的版本，目前使用较多的是 3.x 的版本
  - 另外 `corejs` 可以设置是否对提议阶段的特性进行支持
  - 设置 `proposals` 属性为 `true` 即可

### useBuiltIns属性设置

- false: 
  - 打包后的文件不使用 `polyfill` 来适配
  - 这个时候不需要设置 `corejs` 属性
- usage: 
  - 会根据源代码中出现的语言特性，自动检测所需要的 `polyfill`
  - 这样可以确保最终包里的 `polyfill` 数量的最小化，打包的包相对会小一些
  - 可以设置 `corejs` 属性来确定使用的 `corejs` 的版本
- entry:
  - 如果我们依赖的某一个库本身使用了某些 `polyfill` 的特性，但是因为我们使用的是usage，所以之后用户浏览器可能会报错
  - 所以，如果你担心出现这种情况，可以使用 entry
  - 并且需要在入口文件引入：`core-js` 和 `regenerator-runtime`
  - 这样做会根据 browserslist 目标导入所有的 `polyfill`，但是对应的包也会变大

```js
module.exports = {
  presets: [
    ["@babel/preset-env", {
      // false: 不用任何的polyfill相关的代码
      // usage: 代码中需要哪些polyfill, 就引用相关的api
      // entry: 手动在入口文件中导入 core-js/regenerator-runtime, 根据目标浏览器引入所有对应的polyfill
      useBuiltIns: "usage",
      corejs: 3
    }],
    ["@babel/preset-react"]
  ],
}
```

### 了解 Plugin-transform-runtime

- 在前面使用的 `polyfill` ，默认情况是添加的所有特性都是全局的
  - 如果我们正在编写一个工具库，这个工具库需要使用 `polyfill`
  - 别人在使用我们工具时，工具库通过 `polyfill` 添加的特性，可能会污染它们的代码
  - 所以，当编写工具时，`Babel` 更推荐我们使用一个插件： `@babel/plugin-transform-runtime` 来完成 `polyfill` 的功能

### 使用 Plugin-transform-runtime

- 安装 `@babel/plugin-transform-runtime`
  ```bash
  npm install @babel/plugin-transform-runtime -D
  ```
- 使用 `plugins` 来配置 `babel.config.js` ：
  ```js
  module.exports = {
    presets: [
      ["@babel/preset-env"],
    ],
    plugins: [
       ["@babel/plugin-transform-runtime", {
        corejs: 3
       }]
     ]
  }
  ```

## jsx 支持

- 在编写 `React` 代码时，`React` 使用的语法是 `jsx`，`jsx` 是可以直接使用 `Babel` 来转换的
- 对 `jsx` 代码进行处理需要如下的插件：
  - @babel/plugin-syntax-jsx
  - @babel/plugin-transform-react-jsx
- 但是我们不需要一个个安装插件，只需要安装预设即可：
  ```bash
  npm install @babel/preset-react -D
  ```
## TypeScript 编译

在项目开发中，我们会使用 `TypeScript` 来开发，那么 `TypeScript` 代码是需要转换成 `JavaScript` 代码

### 使用 ts-loader

- 如果我们希望在 `webpack` 中使用 `TypeScript` ，那么我们可以使用 `ts-loader` 来处理ts文件
  ```bash
  npm install ts-loader -D
  ```
- 配置 `ts-loader`：
  ```js
  {
    test: /\.ts$/,
    exclude: /node_modules/,
    // 本质上是依赖于typescript(typescript compiler)
    use: "babel-loader"
  }
  ```
- 以后，我们直接打包即可

### 使用 babel-loader

- 除了可以使用 `TypeScript Compiler` 来编译 `TypeScript` 之外，我们也可以使用 `Babel`：
  - `Babel` 是有对 `TypeScript` 进行支持
  - 我们可以使用插件： @babel/tranform-typescript；
  - 但是更推荐直接使用preset：@babel/preset-typescript；
  ```js
  module.exports = {
    presets: [
      ["@babel/preset-env", {
        useBuiltIns: "usage",
        corejs: 3
      }],
      ["@babel/preset-react"],
      ["@babel/preset-typescript"]
    ],
  }
  ```

### ts-loader vs babel-loader

- ts-loader：
  - 直接编译 `TypeScript` ，那么只能将ts转换成js
  - 如果我们还希望在这个过程中添加对应的 `polyfill`，那么 `ts-loader` 是无能为力的
- babel-loader：
  - 直接编译 `TypeScript` ，也可以将ts转换成js，并且可以实现 `polyfill` 的功能
  - 但是 `babel-loader` 在编译的过程中，不会对类型错误进行检测
  
### 编译 TypeScript 最佳实践

官方文档给了我们答案：我们使用 `Babel` 来完成代码的转换，使用 `tsc` 来进行类型的检查

![20220616162438](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220616162438.png)

![20220616162535](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220616162535.png)

