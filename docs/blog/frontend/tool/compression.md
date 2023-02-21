# 代码压缩

## JavaScript压缩

### Terser

`Terser` 是一个JavaScript的解释（Parser）、Mangler（绞肉机）/Compressor（压缩机）的工具集

也就是说，`Terser` 可以帮助我们压缩、丑化我们的代码。让我们的 `bundle` 变得更小

#### 安装与使用

安装：

```bash
npm install terser
```

命令行使用：

```bash
npx terser [input.files] [options]
```

例子：

```
npx terser js/file1.js -o foo.min.js -c -m
```

#### 常用的配置

- Compress option：
  - arrows：`class` 或者 `object` 中的函数，转换成箭头函数
  - arguments：将函数中使用 `arguments[index]` 转成对应的形参名称
  - dead_code：移除不可达的代码 `（tree shaking）`
- Mangle option
  - toplevel：默认值是 `false`，顶层作用域中的变量名称，进行丑化（转换）
  - keep_classnames：默认值是 `false`，是否保持依赖的类名称
  - keep_fnames：默认值是 `false`，是否保持原来的函数名称

> 其他属性可以查看文档

#### Terser 在 webpack 中配置

- 真实开发中，一般不需要手动通过 `terser` 来处理代码，我们可以直接通过 `webpack` 来处理
  - `webpack` 中的 `minimizer` 属性，在 `production` 模式下，默认就是使用 `TerserPlugin` 来处理我们的代码的
  - 如果我们对默认的配置不满意，也可以自己来创建 `TerserPlugin` 的实例，并且覆盖相关的配置
- 首先，我们需要打开 `minimize` ，让其对我们的代码进行压缩（默认production模式下已经打开了）
- 其次，我们可以在 `minimizer` 创建一个 `TerserPlugin`：
  - extractComments：默认值为 `true`，表示会将注释抽取到一个单独的文件中；
    - 在开发中，我们不希望保留这个注释时，可以设置为false；
  - parallel：使用多进程并发运行提高构建的速度，默认值是true，并发运行的默认数量： os.cpus().length - 1
    - 我们也可以设置自己的个数，但是使用默认值即可
  - terserOptions：设置我们的terser相关的配置
    - compress：设置压缩相关的选项
    - mangle：设置丑化相关的选项，可以直接设置为true
    - toplevel：底层变量是否进行转换
    - keep_classnames：保留类的名称
    - keep_fnames：保留函数的名称

## css压缩

CSS压缩通常是去除无用的空格等，因为很难去修改选择器、属性的名称、值等；

我们可以直接使用 `css-minimizer-webpack-plugin`，它是使用 `cssnano` 工具来优化、压缩 `css`

使用：

1. 安装：
  ```bash
  npm install css-minimizer-webpack-plugin -D
  ```
2. 添加配置：
  ```js
  new MiniCssExtractPlugin({
    filename: "css/[name].[contenthash:6].css"
  }),
  ```

### Scope Hoisting

`Scope Hoisting` 功能是对作用域进行提升，并且让 `webpack` 打包后的代码更小、运行更快

默认情况下 `webpack` 打包会有很多的函数作用域，包括一些（比如最外层的）`IIFE`：

而 `Scope Hoisting` 可以将函数合并到一个模块中来运行；

使用 `Scope Hoisting` 非常的简单，`webpack` 已经内置了对应的模块：
- 在 `production` 模式下，默认这个模块就会启用；
- 在 `development` 模式下，我们需要自己来打开该模块：
  ```js
  new webpack.optimize.ModuleConcatenationPlugin()
  ```