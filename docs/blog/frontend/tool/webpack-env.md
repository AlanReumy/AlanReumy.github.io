# webpack 区分环境与代码分离

## 如何区分开发环境

如果我们将所有代码放到 `webpack.config.js` 这一个文件中：

- 当配置越来越多的时候，这个文件会越来越难维护
- 并且难以区分开发环境和生产环境

区分的方案：

1. 编写两个不同的配置文件，开发和生产不同的环境分别加载不同的配置文件
2. 使用相同的入口配置文件，通过设置参数来区分它们：
   ```json
   {
     "scripts": {
       "build": "webpack --config ./config/webpack.common.js --env production",
       "serve": "webpack serve --config ./config/webpack.common.js --env development"
     }
   }
   ```

### 入口文件解析

如果我们将配置文件放到 `config` 文件夹下，这个时候如果将入口文件根据路径进行修改，会报错，因为入口文件和 `context` 属性有关

`context` 属性的作用是用于解析入口文件

![20220619124032](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220619124032.png)

### 配置文件的分离

我们创建三个文件：

- `webpack.common.js`
- `webpack.dev.js`
- `webpack.prod.js`

目前我们的目录是这样的：

![20220619124334](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220619124334.png)

然后通过 `webpack-merge` 这个库，来合并配置

```js
// webpack.common.js
const resolveApp = require("./paths");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const VueLoaderPlugin = require("vue-loader/lib/plugin");

const { merge } = require("webpack-merge");

const prodConfig = require("./webpack.prod");
const devConfig = require("./webpack.dev");

const commonConfig = {
  entry: "./src/index.js",
  output: {
    filename: "bundle.js",
    path: resolveApp("./build"),
  },
  resolve: {
    extensions: [".wasm", ".mjs", ".js", ".json", ".jsx", ".ts", ".vue"],
    alias: {
      "@": resolveApp("./src"),
      pages: resolveApp("./src/pages"),
    },
  },
  module: {
    rules: [
      {
        test: /\.jsx?$/i,
        use: "babel-loader",
      },
      {
        test: /\.vue$/i,
        use: "vue-loader",
      },
      {
        test: /\.css/i,
        use: ["style-loader", "css-loader"],
      },
    ],
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: "./index.html",
    }),
    new VueLoaderPlugin(),
  ],
};

// 返回一个函数
module.exports = function (env) {
  // env对象中根据不同的环境，production 和 development 有不同的布尔值
  const isProduction = env.production;
  process.env.NODE_ENV = isProduction ? "production" : "development";

  const config = isProduction ? prodConfig : devConfig;
  const mergeConfig = merge(commonConfig, config);

  return mergeConfig;
};
```

这里面 `resolveApp` 函数如下：

```js
const path = require("path");

const appDir = process.cwd();
const resolveApp = (relativePath) => path.resolve(appDir, relativePath);

module.exports = resolveApp;
```

这个函数的目的是读取到进程目录，作目录的合并。可以防止修改配置文件目录后，要去手动修改配置文件中的配置的目录

## 认识代码分离

代码分离 `（Code Splitting）` 是 `webpack` 一个非常重要的特性：

- 它主要的目的是将代码分离到不同的 `bundle` 中，之后我们可以按需加载，或者并行加载这些文件
- 比如默认情况下，所有的 `JavaScript` 代码（业务代码、第三方依赖、暂时没有用到的模块）在首页全部都加载就会影响首页的加载速度
- 代码分离可以分出出更小的 `bundle` ，以提供代码的加载性能

常用的代码分离方案有三种：

1. 入口起点：使用 `entry` 配置手动分离代码；
2. 防止重复：使用 `Entry Dependencies` 或者 `SplitChunksPlugin` 去重和分离代码
3. 动态导入：通过模块的内联函数调用来分离代码

### 多入口起点

入口起点的含义非常简单，就是配置多入口：

- 比如配置一个 `index.js` 和 `main.js` 的入口
- 他们分别有自己的代码逻辑

![20220619125959](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220619125959.png)

### 入口依赖

假如我们的 `index.js` 和 `main.js` 都依赖两个库：`lodash`、`dayjs`

通过 `share` 属性可以对依赖进行共享：

![20220619130343](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220619130343.png)

### SplitChunks

另外一种分包的模式是 `splitChunk` ，它是使用 `SplitChunksPlugin` 来实现的：

- 因为该插件 `webpack` 已经默认安装和集成，所以我们并不需要单独安装和直接使用该插件
- 只需要提供 `SplitChunksPlugin` 相关的配置信息即可

`Webpack` 提供了 `SplitChunksPlugin` 默认的配置，我们也可以手动来修改它的配置

在默认配置中, `chunks` 仅仅设置为 `async` 异步请求，我们可以设置为 `initial` 或者 `all`

![20220619131207](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220619131207.png)

#### SplitChunks 自定义配置解析

- Chunks：
  - 默认值是 `async`
  - 另外一个值是 `initial`，表示对同步的代码进行处理
  - `all` 表示对同步和异步的代码都进行处理
- minSize：
  - 拆分包的大小, 至少为 `minSize`
  - 如果一个包拆分出来达不到 `minSize` ,那么这个包就不会拆分
- maxSize：
  - 将大于 `maxSize` 的包，拆分为不小于 `minSize` 的包
- minChunks：
  - 至少被引入的次数，默认是 1
  - 如果我们写一个 2，但是引入了一次，那么不会被单独拆分
- name：
  - 设置拆包的名称
  - 可以设置一个名称，也可以设置为 `false`
  - 设置为 `false` 后，需要在 cacheGroups 中设置名称
- cacheGroups：
  - 用于对拆分的包就行分组，比如一个 `lodash` 在拆分之后，并不会立即打包，而是会等到有没有其他符合规则的包一起来打包
  - `test` 属性：匹配符合规则的包
  - `name` 属性：拆分包的 name 属性
  - `filename` 属性：拆分包的名称，可以自己使用占位属性

### 动态导入(dynamic import)

另外一个代码拆分的方式是动态导入时，`webpack` 提供了两种实现动态导入的方式：

1. 使用 `ECMAScript` 中的 `import()` 语法来完成，也是目前推荐的方式
2. 使用 `webpack` 遗留的 `require.ensure`，目前已经不推荐使用

比如在下面这个例子中，我们想要在需要加载的时候再加载 `foo.js` 文件，我们可以用这种方式动态导入：

```js
import("./foo").then((res) => {
  console.log(res);
});
```

通过 `chunkFilename` 我们可以设置文件名：

![20220619133358](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220619133358.png)

默认情况下，id 和 name 是一样的，我们可以使用 `magic comment` 魔法注释，来进行设置

```js
import(/* webpackChunkName: "foo" */ "./foo").then((res) => {
  console.log(res);
});
```

#### 代码懒加载

最常见的动态导入场景就是代码懒加载：

![20220619133651](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220619133651.png)

### optimization.chunkIds 设置

`optimization.chunkIds` 配置用于告知 `webpack` 模块的 `id` 采用什么算法生成

有一下三种方式：

- natural：使用自然数
- named：使用包所在目录作为文件名，（开发环境下推荐）
- deterministic：生成 `id` ，针对相同文件生成的 `id` 是不变的

最佳实践：

- 开发中使用 `name`
- 生产中使用 `deterministic`

### optimization.runtimeChunk 配置

- 配置 `runtime` 相关的代码是否抽取到一个单独的 `chunk` 中
- 抽离出来后，有利于浏览器缓存策略
- 设置的值：
  - true/multiple：针对每个入口打包一个 `runtime` 文件
  - single：打包一个 `runtime` 文件
  - 对象：name 属性决定 `runtimeChunk` 的名称

### prefetch 和 preload

`webpack` 在 4.6.0 以上的版本增加了对预获取和预加载的支持

- 在声明 `import` 时，使用这些内置指令，来告知浏览器：

  - prefetch（预获取）：将来某些导航下可能用到的资源
  - preload（预加载）：当前导航下需要的资源

- 区别：
  - `preload chunk` 会在父 chunk 加载时，以并行方式开始加载。`prefetch chunk` 会在父 chunk 加载结束后开始加载
  - `preload chunk` 具有中等优先级，并立即下载。`prefetch chunk` 在浏览器闲置时下载
  - `preload chunk `会在父 chunk 中立即请求，用于当下时刻。`prefetch chunk` 会用于未来的某个时刻

### CDN

#### 什么是 CDN

**CDN** 称之为内容分发网络（Content Delivery Network 或 Content Distribution Network，缩写：`CDN`）

- 它是指通过相互连接的网络系统，利用最靠近每个用户的服务器
- 更快、更可靠地将音乐、图片、视频、应用程序及其他文件发送给用户,来提供高性能、可扩展性及低成本的网络内容传递给用户

使用 CDN 的方式：

- 打包的所有静态资源，放到 CDN 服务器，用户所有资源都是通过 CDN 服务器加载的
- 一些第三方资源放到 CDN 服务器上

#### webpack 中关于 CDN 的配置

我们可以直接将打包后的代码放到 CDN 上，然后通过修改 `output` 中的 `publicPath` 在打包时添加自己的 CDN 地址：

![20220624092441](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220624092441.png)

![20220624092457](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220624092457.png)

#### 使用第三方的 CDN

1. 通过 `webpack` 的配置，来排除库的打包
2. 在 `html` 模块中，加入对应的 CDN 服务器地址

![20220624092714](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220624092714.png)

![20220624092735](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220624092735.png)

### shimming

- shamming 是对某一类功能的统称：
  - 它的翻译是垫片，也就是给代码填充一些垫片来处理问题
  - 比如我们现在依赖一个第三方库，而这个第三方库本身依赖于另一个第三方库，但是默认没有对另一个第三方库进行导入，我们就可以通过 `ProvidePlugin` 来实现 `shimming` 的效果

> `webpack` 并不推荐使用 `shimming`，因为 `webpack` 的理念是使得前端开发变得更加模块化，也就意味着需要编写具有封闭性，不存在隐式依赖的彼此隔离的模块

#### 应用场景

假如我们在一个文件中使用了 `axios`，但是没有对它进行引入，那么下面这段代码会报错：

```js
import axios, { get } from "axios";

axios.get("http://zimtang.cn/users").then((res) => {
  console.log(res);
});

get("http://zimtang.cn/data").then((res) => {
  console.log(res);
});
```

但是我们可以通过使用 `ProvidePlugin` 来实现 `shimming` 效果：

- `ProvidePlugin` 能帮助我们在每个模块中，通过一个变量来获取 `package`
- 如果`webpack` 看到这个模块，它将会在最终的 `bundle` 中引入这个模块
- `ProvidePlugin` 是 `webpack` 默认的一个插件，所以不需要专门导入

```js
new webpack.ProvidePlugin({
  axios: "axios",
  get: ["axios", "get"],
});
```

### MiniCssExtractPlugin

`MiniCssExtractPlugin` 可以帮助我们将 css 提取到一个独立的 css 文件中，该插件需要在 `webpack4+` 才可以使用。

![20220624094056](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220624094056.png)

![20220624094156](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220624094156.png)

### hash、contenthash、chunkhash

- hash 值的生成和整个项目有关系：
  - 比如我们现在有两个入口 `index.js` 和 `main.js`
  - 它们分别会输出到不同的 `bundle` 文件中，并且在文件名称中我们有使用 `hash`
  - 这个时候，如果修改了 `index.js` 文件中的内容，那么 hash 会发生变化
  - 那就意味着两个文件的名称都会发生变化
- chunkhash 可以有效的解决上面的问题，它会根据不同的入口进行借来解析来生成 hash 值：
  - 比如我们修改了 `index.js` ，那么 `main.js` 的 `chunkhash` 是不会发生改变的
- contenthash 表示生成的文件 `hash` 名称，只和内容有关系
