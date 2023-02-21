# webpack 的devServer与HMR

## 为什么要搭建本地服务

- 提高开发效率：
  我们肯定不希望每次修改源代码，就自己手动编译打包一次，搭建本地服务器可以省略自己编译打包的过程
- 为了完成自动编译，`webpack` 给了我们以下几种方式：
  - webpack watch
  - webpack-dev-server
  - webpack-dev-middleware

## wepback watch

`webpack` 为用户提供了 `watch` 模式：

- 该模式下，只要 `webpack` 依赖图里的文件，有一个发生了更新，那么代码就会重新编译
- 不需要再手动运行 `npm run build`

如何开启：

- 方式一：在配置文件中将 watch属性设置为 `true`
- 方式二：在执行的 `script` 中，改为：`webpack --watch`

## webpack-dev-server

上面这种方式监听到文件的变化，但它本身是没有自动刷新浏览器的功能的

我们可以使用 `webpack-dev-server` ：

- 安装：
  ```bash
  npm install webpack-dev-server -D
  ```
- 添加 `script`：
  ```bash
  webpack serve
  ```

>  webpack-dev-server 在编译之后不会写入到任何输出文件。而是将 `bundle.js` 文件保留在内存中，它的底层使用了 `memfs` 这个库

## webpack-dev-middleware

默认情况下，`webpack-dev-server` 帮我们配置好了一切，但是如果需要更高的自由度的话，可以使用 `webpack-dev-middleware`：

什么是 `webpack-dev-middleware`：

- `webpack-dev-middleware` 是一个封装器，可以把 `webpack` 处理过的文件发送到一个 `server`
- `webpack-dev-server` 在内部使用了它，然而它也可以作为一个单独的包来使用，以便根据需求进行

如何使用：

- 安装：
  ```bash
  npm install webpack-dev-middleware -D
  ```
- 编写 `server.js` 文件，这里使用了 `express` 来搭建，也可以使用 `koa`：
  ```js
  const express = require('express');
  const webpack = require('webpack');
  const webpackDevMiddleware = require('webpack-dev-middleware');

  const app = express();

  const config = require("./webpack.config");

  // 传入配置信息, webpack根据配置信息进行编译
  const compiler =  webpack(config);

  const middleware = webpackDevMiddleware(compiler);
  app.use(middleware);

  app.listen(3000, () => {
    console.log("服务已经开启在3000端口上~");
  });
  ```
- 命令行中 `node server.js` 运行

## 模块热替换(HMR)

### 什么是 HMR：

`HMR` 的全称是：Hot Module Replacement，模块热替换

它的作用是在应用程序的运行过程中，替换、添加、删除模块后，无需重新刷新整个页面，只更新需要更新的内容

修改了 `css`，`js` 文件，会立即在浏览器更新，相当于在浏览器的 `devtool` 中修改代码

### 开启 HMR

- 修改配置文件：
  ```js
  devServer: {
    hot: true
  }
  ```
- 我们重新运行 `npm run serve` 后，会在浏览器看到如下效果：
  ![20220618144541](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220618144541.png)
- 但是你会发现，当我们修改了某一个模块的代码时，依然是刷新的整个页面：
  这是因为，我们需要指定哪些模块需要在更新时，使用 `HMR`

### 框架的 HMR

在使用框架开发时，如 `vue`，`react` 开发时，是否需要经常写 `module.hot.accpet` 来支持 `HMR`呢？

肯定是不可能的，在实际开发中，对应的框架都有对应的 `loader` 或者 `plugin` 来提供开箱即用的 `HMR` 体验

- `Vue` 中使用的是 `vue-loader` 来实现
- `React` 中使用的是 `react-refresh` 来实现

### HMR 原理

`webpack-dev-server` 会创建两个服务：提供静态资源的服务（ express ）和 `Socket` 服务（ net.Socket ）

* `express server` 负责直接提供静态资源的服务（打包后的资源直接被浏览器请求和解析）
* `HMR Socket Server` 是一个 `socket` 的长连接：
  * 长连接有一个最好的好处是建立连接后双方可以通信（服务器可以直接发送文件到客户端）
  * 当服务器监听到对应的模块发生变化时，会生成两个文件 `.json（manifest文件）` 和 `.js文件（update chunk）`
  * 通过长连接，可以直接将这两个文件主动发送给客户端（浏览器）
  * 浏览器拿到两个新的文件后，通过 `HMR runtime` 机制，加载这两个文件，并且针对修改的模块进行更新

`HMR` 的原理图：

![image-20220107193450326](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20220107193450326.png)

### proxy 配置

`proxy` 是我们开发中非常常用的一个配置选项，它的目的设置代理来解决跨域访问的问题。

我们可以在 `devServer` 中，配置 `proxy` 属性进行设置：

![image-20220107194648139](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20220107194648139.png)

具体属性的含义：

* `target` ：表示的是代理到的目标地址，比如 /api/moment会被代理到 `http://localhost:10086/api/moment`

* `pathRewrite` ：默认情况下，我们的 `/api` 也会被写入到URL中，如果希望删除，可以使用 `pathRewrite`

* `secure` ：默认情况下不接收转发到https的服务器上，如果希望支持，可以设置为false

* `changeOrigin` ：它表示是否更新代理后请求的 `headers` 中 `host` 地址

### devServer 其他配置

* `host` 配置主机地址
* `port` 配置监听端口
* `open` 配置是否打开浏览器
* `compress` 配置是否为静态文件开启 `gzip compression`

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