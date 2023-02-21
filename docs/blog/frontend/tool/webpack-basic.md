# webpack 入门

## 什么是 webpack

先看看 `webpack` 官方文档的解释：

> **webpack** is a *static module bundler* for modern JavaScript applications.

翻译成中文的意思就是：`webpack` 是一个为现代 `JavaScript` 应用的静态打包工具。

这里面有几个重要的名词：`static` 、`module` 、`bundler` 、`modern`

* `static`：表示静态资源，最终将它们部署到服务器上
* `module`：表示默认支持模块化开发，如：`CommonJS` 、`ES Module` 等
* `bundler`：表示这是一个打包工具
* `modern`：表示现代前端的发展存在很多问题，正是因为有这些问题，才催生了 `webpack` 的出现

### webpack 出现前存在哪些问题

如今的前端开发已经变得越来越复杂，需要考虑很多东西，如：模块化开发、使用高级特性来提高开发效率、对代码进行压缩、实施监听文件变化提高开发效率。

这个时候 `webpack` 出现了。

如今的前端三大框架 `Vue` 、`React` 、`Angular` 的创建使用都是基于脚手架（CLI），但其实，CLI都是 **基于webpack** 来帮助我们支持上述功能。

## webpack 的安装

`webpack` 的安装分为两个：`webpack`、`webpack-cli`

**它们之间的关系是这样的：**

* 在执行webpack命令的时候，会执行 `node_modules` 下的 `.bin` 目录下的 `webpack`
* `webpack` 在执行的时候，依赖 `webpack-cli` ，如果没有安装则会报错
* 而 `webpack-cli` 中的代码在执行的时候，才是真正利用了 `webpack` 的编译和打包

## webpack 的默认打包

在项目目录下，我们可以直接在终端中，执行 `webpack` 命令。

此时会生成一个 `dist` 文件夹，里面会存放 `main.js` 文件。这就是 `webpack` 的出口文件。

### 入口与出口

当我们运行 `webpack` 命令的时候，它会查找当前目录下的 `src/index.js` 文件作为入口文件。

并且会将打包生成 `dist/main.js` 文件，作为出口文件。

在这个 `main.js` 文件中，它默认会对代码进行压缩和丑化。

## 局部使用 webpack

可以通过 `npx webpack` 命令执行，它会执行 `node_modules` 下的 `.bin` 目录下的 `webpack`。

也可以通过在 `package.json` 文件中，创建 `scripts` 脚本，执行打包。

```json
  "scripts": {
    "build": "webpack"
  },
```

## webpack 的配置文件

webpack需要打包的项目是非常复杂的，并且我们需要一系列的配置来满足要求。

我们可以在根目录下创建一个 `webpack.config.js` 文件，来作为 `webpack` 的配置文件：

```js
const path = require('path')

module.exports = {
  // 配置入口文件
  entry: './src/index.js',
  // 配置出口文件
  output: {
    // 绝对路径   dirname获取当前文件所在的路径
    path: path.resolve(__dirname, './dist'),
    // 打包的文件名
    filename: 'main.js',
  },
}
```

## webpack 的依赖

`webpack` 在处理应用程序时，它会根据命令或者配置文件找到入口文件。

从入口开始，会生成一个 依赖关系图，这个依赖关系图会包含应用程序中所需的所有模块（比如.js文件、css文件、图片、字体等）。

然后遍历图结构，打包一个个模块（根据文件的不同使用不同的 `loader` 来解析）。

**`webpack` 的官方图：**

![image-20220102112311888](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20220102112311888.png)

## 什么是loader

`loader` 可以用于对模块的源代码进行转换，

我们可以将 `css` 文件也看成是一个模块，如果我们现在在一个 `js` 文件中，写如下的代码用于引入 `css` 文件：

```js
import './css/style.css'
```

此时我们进行打包的时候，是会报错的。

因为在加载这个模块的时候，`webpack` 是不知道如何对其进行加载的。我们需要使用对应的 `loader` 来完成这个加载的功能。

## loader的配置

如果需要加载 `css` 文件模块，需要使用 `css-loader`。

   在引入的文件前，加上使用的 `loader` ，并使用 ! 分割

   ```js
   import 'css-loader!./css/style.css'
   ```

2. 配置方式**（推荐）**

   `module.rules`  中允许配置多个 `loader`

   `rules` 对应的值是一个数组：数组中是一个个的对象，对象中可以设置多个属性：

   * `test` 属性：用于对 resource（资源）进行匹配的，通常会设置成正则表达式
   * `use` 属性：对应的是一个数组，数组中的每个元素是对象，对象的属性包括
     * `loader`：必须有一个 loader属性，对应的值是一个字符串
     * `options`：可选的属性，值是一个字符串或者对象，值会被传入到loader中
   * `loader`属性： `Rule.use: [ { loader } ]` 的简写

   ```js
   module：{
     rules:[
       {
         {
           // 正则表达式
           test: /\.css$/,
           // 第一种写法 语法糖
           // loader: 'css-loader',
   
           // 第二种写法 完整写法
           use: [
             {
               loader: 'css-loader',
             },
           ],
         },
       }
     ]
   }
   ```

配置完成，重新打包后，我们会发现样式并没有生效，这是因为 `css-loader` 只是**负责将.css文件进行解析**，并不会将解析之后的css插入到页面中。

如果我们需要完成插入`style` 的操作，需要使用 `style-loader` 。

## loader 的配置顺序

`loader` 的配置是需要考虑顺序的，比如 `style-loader` 的使用是需要建立在有 `css-loader`  的基础上的。

因为loader的执行顺序是从后到前，所以我们需要将 `style-loader` 写到 `css-loader` 的前面；

```js
use: [
  'style-loader',
  'css-loader'
],
```

## less 的处理

在实际开发中，我们可能会使用 `less`、`sass` 这样的预处理器，（这里以 `less` 为例）因此我们需要一个将`less` 转换为 `css` 文件的编译工具。

我们可以使用 `less` 工具完成编译：

安装：

`npm install less -D`

执行如下命令，进行编译转换：

`npx lessc 'less文件' 'css文件'`

### less-loader

但是，在实际开发中，我们需要编写很多的 `less` 文件，如果全部进行手动编译，是不现实的。因此，我们需要 `less-loader` 来帮助我们完成自动编译。

```js
{
  test:/\.less$/,
  use: [
    'less-loader',
    'style-loader',
    'css-loader'
  ],
}
```

## PostCSS  配置

`PostCSS` 是一个通过 `JavaScript` 来转换样式的工具，可以帮助我们进行一些CSS的转换和适配，比如自动添加浏览器前缀、css样式的重置。

但是实现这些功能，我们需要借助于PostCSS对应的插件。

在 `webpack` 中，可以通过 `postcss-loader` 来使用，然后选择需要的 `PostCSS` 相关的插件。

### PostCSS 的命令行使用

在命令行中也是可以使用 `PostCSS` 的。

首先安装：

`npm install postcss postcss-cli -D`

假如我们现在需要为 `css` 属性添加浏览器前缀，那么我们需要安装 `autoprefixer` 插件：

`npm install autoprefixer -D`

执行命令：

`npx postcss --use autoprefixer -o end.css ./src/css/style.css`

### postcss-loader

在实际开发中，肯定是不会用命令行来对 `css` 进行处理，在 `webpack` 中，使用 `postcss-loader` 可以达到这样的效果。

安装：

`npm install postcss-loader -D`

配置：

![image-20220107203826906](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20220107203826906.png)

也可以将这些配置信息单独放在一个配置文件 ( postcss.config.js ) 中管理：

![image-20220107203948463](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20220107203948463.png)

## 静态资源的打包

如果我们需要在项目中使用静态资源（ 如：图片，字体等 ），也需要对它进行打包的处理。

在 `webpack5` 之前，加载这些资源我们需要使用一些 `loader`，比如 `raw-loader` 、`url-loader`、`file-loader` 。

从 `webpack5` 开始，我们可以直接使用资源模块（ asset module type ），来代替上面的这些 `loader` 。

资源模块类型( asset module type )，通过添加 4 种新的模块类型，来替换所有这些 `loader`：

* **asset/resource** 发送一个单独的文件并导出 URL。之前通过使用 `file-loader` 实现；

* **asset/inline** 导出一个资源的 data URI。之前通过使用 `url-loader` 实现；

* **asset/source** 导出资源的源代码。之前通过使用 `raw-loader` 实现；

* **asset** 在导出一个 data URI 和发送一个单独的文件之间自动选择。之前通过使用 url-loader，并且配置资源体积限制实现；

```json
{
  test: /\.(jpe?g|png|gif|svg)$/,
  type: "asset",
  generator: {
    // 设置生成的文件名和在哪个文件夹下面
    // 其中img/为文件夹名称 
    // [name] 原文件名
    // [hash:xxx] 设置的hash值
    // [ext] 文件扩展名
    filename: "img/[name]_[hash:6][ext]"
  },
  parser: {
    dataUrlCondition: {
      // 设置需要转换base64的文件大小最大值
      maxSize: 100 * 1024
    }
  }
},
```

## Plugin

`webpack` 的另一个核心概念是 `Plugin` 。

`Loader` 仅仅是用于特定的模块类型进行转换。

而 `Plugin` 可以用于执行更加广泛的任务，如打包优化，资源管理等。

### CleanWebpackPlugin

此插件可以帮助我们打包的时候，自动删除原打包文件夹。

#### 使用

首先需要使用 `npm` 安装：

`npm install clean-webpack-plugin -D`

接着引入，并在 `webpack.config.js` 文件中使用：

```js
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
plugins:[
  new CleanWebpackPlugin()
]
```

### **HtmlWebpackPlugin**

此插件是对 `index.html` 文件进行打包处理。

```js
new HtmlWebpackPlugin()
```

#### 自定义HTML模板

如果我们想要在自己的模块中加入一些特别的内容，这时候我们需要一个自己的模板

```js
new HtmlWebpackPlugin({
  template: "./public/index.html",
  title: "html文件的标题名"
})
```

### DefinePlugin

此插件可以在编译 `template` 中的常量，对常量进行解析。

```js
new DefinePlugin({
  // 配置全局常量 BASE_URL
  BASE_URL: "'./'"
}),
```

### **CopyWebpackPlugin**

在打包过程中，如果我们想要将一个目录下的文件被复制到 `dist` 文件夹中，我们可以使用 `CopyWebpackPlugin` 来完成。

```js
new CopyWebpackPlugin({
  // 复制的规则在patterns中设置
  patterns: [
    {
      // from是原目录
      from: "public",
      // to是复制到的目录 默认值是打包的目录
      to: "./",
      // globOptions用于设置一些额外的选项，其中可以编写需要忽略的文件：
      globOptions: {
        ignore: [
          "**/index.html"
        ]
      }
    }
  ]
})
```

## mode配置

mode配置选项，可以告知 `webpack` 使用响应模式的内置优化。

默认值是  `production` ，可选值有 `none` | `development` | `production`

其中在开发阶段，我们一般设置为 `development` ，生产阶段设置为 `production` 。

> 注意：mode的配置一旦配置，webpack其实帮我们配置了其他很多东西，如果出现与你的配置重复，则会选择合并，不会覆盖。

## babel 的配置

`babel` 是一个工具链，主要用于旧浏览器或者环境中将 `ES5` 之后的代码转换为向后兼容版本的 `js` 。这其中包括：语法转换、源代码转换等。

### babel 的命令行使用

在命令行中使用 `babel` ，需要安装 `@babel/cli` 与 `@babel/core`

命令行：`npx babel 源文件目录 --out-dir 输出的目录`

### babel 插件

如果我们想要转换箭头函数，我们需要使用额外的插件。

箭头函数转换：

`npm install @babel/plugin-transform-arrow-functions -D`

`npx babel src --out-dir dist --plugins=@babel/plugin-transform-arrow-functions`

 将 `const` 转换为 `var` 也需要特定的插件来实现：

`npm install @babel/plugin-transform-block-scoping -D`

`npx babel src --out-dir dist --plugins=@babel/plugin-transform-block-scoping,@babel/plugin-transform-arrow-functions`

### babel 预设

如果需要转换的内容过多，一个个设置会非常麻烦，我们可以使用预设 ( preset ) 。

安装预设：

`npm install @babel/preset-env -D`

执行命令：

`npx babel src --out-dir dist --presets=@babel/preset-env`

### babel 的底层原理

`babel` 可以被看做是一个编译器，将我们的源代码，转换成浏览器可以直接识别的另外一段源代码。

`babel` 也有编译器的工作原理:

1. 解析阶段 ( Parsing )
2. 转换阶段 ( Transformation )
3. 生成阶段 ( Code Generation )

`babel` 执行原理：

![image-20220107182632060](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20220107182632060.png)

### babel-loader

在 `webpack` 中，我们可以通过配置来使用 `babel` ，这需要我们安装 `@babel/core` 与 `babel-loader` 。

在 `webpack.config.js` 中，配置 `babel-loader` ：

![image-20220107183032354](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20220107183032354.png)

通过 `options` 属性中的 `plugins` 属性，设置 `babel` 要使用的插件。

同样，我们也可以配置预设 ( 使用 `preset` 属性 )：

![image-20220107183149372](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20220107183149372.png)

### babel 的配置文件

我们可以将 `babel` 的配置信息放在一个独立的文件中，新建：`babel.config.js` 文件 或者 `.babelrc.json` 文件。

它们的区别如下：

`.babelrc.json` ：早期使用较多的配置方式，但是对于配置Monorepos项目是比较麻烦的。

`babel.config.json（babel7）`：可以直接作用于Monorepos项目的子包，更加推荐。

## 对 vue 文件的配置

和其他文件一样，`vue` 文件也有自己的 `loader` —— `vue-loader` ，来处理文件。

在 `webpack.config.js` 中配置：

![image-20220107183702178](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20220107183702178.png)

如果我们配置完成后，直接打包会报错，因为我们必须添加 `@vue/compiler-sfc` 来对 `template` 进行解析

`npm install @vue/compiler-sfc -D`

同时配置对应的插件：

![image-20220107184455918](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20220107184455918.png)

![image-20220107184546229](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20220107184546229.png)

### vue 打包后不同版本解析

* **vue(.runtime).global(.prod).js：**
  * 通过浏览器中的 `<script src="...">` 直接使用；
  * 我们之前通过CDN引入和下载的Vue版本就是这个版本；
  * 会暴露一个全局的 `Vue` 来使用；
* **vue(.runtime).esm-browser(.prod).js：**
* 用于通过原生 ES 模块导入使用 (在浏览器中通过 `<script type="module">` 来使用)。
* **vue(.runtime).esm-bundler.js**：
  * 用于 `webpack`，`rollup` 和 `parcel` 等构建工具；
  * 构建工具中默认是 `vue.runtime.esm-bundler.js` ；
  * 如果我们需要解析模板 `template` ，那么需要手动指定 `vue.esm-bundler.js` ；
* **vue.cjs(.prod).js：**
  * 服务器端渲染使用；
  * 通过 `require()` 在 `Node.js` 中使用；

### vue 中 运行时 + 编译器 vs 仅运行时

在 `vue` 开发中，有三种方式可以编写 `DOM` 元素：

1. `template` 模板
2. `render` 函数
3. 通过 `.vue` 文件中的 `template`

它们的模板分别是如何处理的：

1. 需要通过源代码中一部分代码进行解析
2. 通过 `vue-loader` 进行解析
3. 通过 `h` 函数返回一个虚拟节点

`vue` 在让我们选择版本的时候分为 **运行时 + 编译器** 与 **仅运行时**：

* **运行时 + 编译器** 包含了对 `template` 模板的编译代码，更加完整，但是也更大一些
* **仅运行时** 没有包含对 `template` 版本的编译代码，相对更小一些

## devServer 的配置

我们可以通过配置 **本地服务器** 做到：文件发生变化的时候，可以自动完成编译与展示

有如下几种可选方式：

* `webapck watch mode`
* `webpack-dev-server`（ 常用 ）
* `webpack-dev-middleware`

### webpack watch

`webpack` 为我们提供了 `watch` 模式，该模式下，`webpack` 依赖图 中的所有文件，只要有一个发生了变化，那么代码将被重新编译。

开启 `watch` 模式的两种方式：

1. 在配置文件中，加入 `watch: true`
2. 在执行 `webapck` 的命令中，添加 `--watch` 标识

![image-20220107191156568](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20220107191156568.png)

在这种情况下，可以监听到文件的变化，但不会自动刷新浏览器。

我们可以通过安装 `webpack-dev-server` 来实现：

`npm install webpack-dev-server -D`

修改 `webpack.config.js` 文件：

```js
module.exports = {
  // 设置为 什么东西 打包 node / web 
  target: 'web',
  // 配置devServer
  devServer: {
    // 如果有些资源没有从webpack加载到，就会从contentBase中加载
    contentBase: "./public",
  },
}
```

### HMR（ 模块热替换 ）

**模块热替换** 是指在 应用程序运行过程中，替换、添加、删除模块，而无需重新刷新整个页面。

默认情况下，`webpack-dev-server` 已经支持HMR，我们只需要开启即可。

![image-20220107191831993](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20220107191831993.png)

开启后，我们需要指定哪些模块发生更新时，进行 `HMR`。

```js
if(module.hot) {
  module.hot.accept("./util.js", () => {
    console.log("util更新了")
  })
}
```

可能有人就会说，那是不是在平时开发中，每一个模块都需要自己手动写开启 `HMR` 的代码呢？

答案肯定是否定的，社区已经针对这些有很成熟的解决方案了：

* 如 `vue` 开发中，`vue-loader` 支持 `HMR` ，提供了开箱即用的体验
* 如 `react` 开发中， `react-refresh` 支持 `HMR` ，实时调整 `react` 组件

### HMR 的原理

`webpack-dev-server` 会创建两个服务：提供静态资源的服务（ express ）和 `Socket` 服务（ net.Socket ）

* `express server` 负责直接提供静态资源的服务（打包后的资源直接被浏览器请求和解析）
* `HMR Socket Server` 是一个 `socket` 的长连接：
  * 长连接有一个最好的好处是建立连接后双方可以通信（服务器可以直接发送文件到客户端）
  * 当服务器监听到对应的模块发生变化时，会生成两个文件 `.json（manifest文件）` 和 `.js文件（update chunk）`
  * 通过长连接，可以直接将这两个文件主动发送给客户端（浏览器）
  * 浏览器拿到两个新的文件后，通过 `HMR runtime` 机制，加载这两个文件，并且针对修改的模块进行更新

`HMR` 的原理图：

![image-20220107193450326](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20220107193450326.png)

### devServer中的其他配置

* `host` 配置主机地址
* `port` 配置监听端口
* `open` 配置是否打开浏览器
* `compress` 配置是否为静态文件开启 `gzip compression`

### proxy

`proxy` 是我们开发中非常常用的一个配置选项，它的目的设置代理来解决跨域访问的问题。

我们可以在 `devServer` 中，配置 `proxy` 属性进行设置：

![image-20220107194648139](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20220107194648139.png)

具体属性的含义：

* `target` ：表示的是代理到的目标地址，比如 /api/moment会被代理到 `http://localhost:10086/api/moment`

* `pathRewrite` ：默认情况下，我们的 `/api` 也会被写入到URL中，如果希望删除，可以使用 `pathRewrite`

* `secure` ：默认情况下不接收转发到https的服务器上，如果希望支持，可以设置为false

* `changeOrigin` ：它表示是否更新代理后请求的 `headers` 中 `host` 地址

## resolve 模块解析

`resolve` 用于设置模块如何被解析。

`webpack` 能解析三种文件路径：

1. 绝对路径

2. 相对路径

3. 模块路径

   在 `resolve.modules` 中指定的所有目录检索模块，默认值是 `['node_modules']`

   也可以设置别名。

### 解析过程

如果没有后缀 先去 该名称的文件夹下查找是否存在 `index.js` 文件，再去找符合 `extensions`  ( `extension` 是 `resolve` 中的一个属性 ) 中后缀名的文件。

### resolve 中的 extensions 与 alias 配置

`extensions` 是解析到文件时自动添加扩展名，默认值是：`['.wasm', '.mjs', '.js', '.json']`，如果我们代码中想要添加加载 `.vue` 或者 `jsx` 或者 `ts` 等文件时，我们必须自己写上扩展名。

`alias` 用来配置路径的别名

![image-20220107201912032](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20220107201912032.png)

## webpack 配置文件的分离

在实际开发中，会在不同的环境中使用不同的 `webpack` 配置，因此我们可以建立如下的文件夹：

![image-20220107202339104](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20220107202339104.png)

其中，`webpack.comm.config.js` 用来存放通用配置，

然后我们将不同环境的配置抽离出来，放在对应环境的配置文件中。

最后，我们需要安装 `webpack-merge` 将不同的环境的配置文件，与通用配置文件合并：

`webpack.dev.config.js`：

![image-20220107202626573](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20220107202626573.png)

## 总结

`webpack` 对于现在的前端来说，相当重要。这篇文章几乎涵盖了 `webpack` 的大部分配置，曾经的我学习 `webpack` 也认为它配置很多，学起来很杂，但其实疏通了整个结构就很清晰了。`webpack` 配置并不难，复杂的是它内部的实现原理。
