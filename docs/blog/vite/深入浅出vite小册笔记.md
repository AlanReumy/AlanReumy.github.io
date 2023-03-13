---
title: 深入浅出vite小册笔记
---

## 前端工程化的痛点

- **模块化需求** -> 提供模块加载方案，兼容不同模块规范
- **兼容浏览器，编译高级语法** -> 高级语法转译，资源加载
- **线上代码的质量** -> 产物压缩、代码混淆、Tree Shaking、语法降级
- **开发效率** -> HMR、构建提速

## Vite 为什么快

- 模块化方面，Vite 基于浏览器原生 ESM 的支持实现模块加载，并且无论是开发环境还是生产环境，都可以将其他格式的产物(如 CommonJS)转换为 ESM。
- 语法转译方面，Vite 内置了对 TypeScript、JSX、Sass 等高级语法的支持，也能够加载各种各样的静态资源，如图片、Worker 等等
- 产物质量方面，Vite 基于成熟的打包工具 Rollup 实现生产环境打包，同时可以配合`Terser`、`Babel`等工具链，可以极大程度保证构建产物的质量。

## 静态资源处理

### 生产环境

1. 自定义部署域名：配置 base 参数
2. 单文件或内联：通过 `build.assetsInlineLimit` 来配置
3. 图片压缩：基于 `imagemin` 的 vite 插件 -> `vite-plugin-imagemin`

## 预构建

Vite 是一个提倡  `no-bundle`  的构建工具，相比于传统的 Webpack，能做到开发时的模块按需编译，而不用先打包完再加载
这里所说的模块代码其实分为两部分，一部分是源代码，也就是业务代码，另一部分是第三方依赖的代码，即`node_modules`中的代码。所谓的`no-bundle`**只是对于源代码而言**，对于第三方依赖而言，Vite 还是选择 bundle(打包)，并且使用速度极快的打包器 Esbuild 来完成这一过程，达到秒级的依赖编译速度。

### 为什么需要预构建

1. 将其他格式(如 UMD 和 CommonJS)的产物转换为 ESM 格式，使其在浏览器通过  `<script type="module"><script>`的方式正常加载
2. 打包第三方库的代码，将各个第三方库分散的文件合并到一起，减少 HTTP 请求数量，避免页面加载性能劣化。

这两件事情全部是由 `esbuild` 完成的。

### 如何开启预构建

分为自动开启和手动开启。

#### 自动开启

项目启动后的 `node_modules/.vite` 文件夹下存放的就是预构建的产物。

#### 手动开启

少数场景下我们不希望用本地的缓存文件，比如需要调试某个包的预构建结果，我推荐使用下面任意一种方法清除缓存：

1. 删除`node_modules/.vite`目录。
2. 在 Vite 配置文件中，将`server.force`设为`true`。(注意，Vite 3.0 中配置项有所更新，你需要将  `optimizeDeps.force`  设为`true`)
3. 命令行执行`npx vite --force`或者`npx vite optimize`。

### 预构建的自定义配置

#### 入口文件

`optimizeDeps.entries`，通过这个参数你可以自定义预构建的入口文件。

#### 添加依赖

`optimizeDeps.include`，决定了强制开启预构建的依赖项

使用场景：

- 动态 import
- 某些包被手动 exclude

## 分析 Esbuild 和 Rollup 在 vite 中扮演的角色

### Esbuild

#### 一、依赖预构建——作为 Bundle 工具

Esbuild 性能很强，但是仍然具有以下缺点：

- 不支持降级到  `ES5`  的代码。这意味着在低端浏览器代码会跑不起来。
- 不支持  `const enum`  等语法。这意味着单独使用这些语法在 esbuild 中会直接抛错。
- 不提供操作打包产物的接口，像 Rollup 中灵活处理打包产物的能力(如`renderChunk`钩子)在 Esbuild 当中完全没有。
- 不支持自定义 Code Splitting 策略。传统的 Webpack 和 Rollup 都提供了自定义拆包策略的 API，而 Esbuild 并未提供，从而降级了拆包优化的灵活性。

#### 二、单文件编译——作为 TS 和 JSX 编译工具

在 TS(X)/JS(X) 单文件编译上面，Vite 也使用 Esbuild 进行语法转译，也就是将 Esbuild 作为 Transformer 来用。

虽然 Esbuild Transformer 能带来巨大的性能提升，但其自身也有局限性，最大的局限性就在于 TS 中的类型检查问题。这是因为 Esbuild 并没有实现 TS 的类型系统。

#### 三、代码压缩——作为压缩工具

在生产环境中 Esbuild 压缩器通过插件的形式融入到了 Rollup 的打包流程中。

传统的方式都是使用 Terser 这种 JS 开发的压缩器来实现，在 Webpack 或者 Rollup 中作为一个 Plugin 来完成代码打包后的压缩混淆的工作。但 Terser 其实很慢，主要有 2 个原因。

1. 压缩这项工作涉及大量 AST 操作，并且在传统的构建流程中，AST 在各个工具之间无法共享，比如 Terser 就无法与 Babel 共享同一个 AST，造成了很多重复解析的过程。
2. JS 本身属于解释性 + JIT（即时编译） 的语言，对于压缩这种 CPU 密集型的工作，其性能远远比不上 Golang 这种原生语言。

因此，Esbuild 这种从头到尾**共享 AST**  以及**原生语言编写**的 Minifier 在性能上能够甩开传统工具的好几十倍。

### Rollup

#### 生产环境 Bundle

1. CSS 代码分割。如果某个异步模块中引入了一些 CSS 代码，Vite 就会自动将这些 CSS 抽取出来生成单独的文件，提高线上产物的`缓存复用率`
2. 自动预加载。Vite 会自动为入口 chunk 的依赖自动生成预加载标签`<link rel="moduelpreload">`
3. 异步 Chunk 加载优化。

#### 插件兼容机制

vite 自己实现了一个 `plugin container` 用来模拟 Rollup 调度各个 Vite 插件的执行逻辑，而 Vite 的插件写法完全兼容 Rollup，因此在生产环境中将所有的 Vite 插件传入 Rollup 也没有问题。反过来说，Rollup 插件却不一定能完全兼容 Vite。

## Esbuild 的使用

### 为什么 `Esbuild` 性能高

1. **使用 Golang 开发**，构建逻辑代码直接被编译为原生机器码，而不用像 JS 一样先代码解析为字节码，然后转换为机器码，大大节省了程序运行时间。
2. **多核并行**。内部打包算法充分利用多核 CPU 优势，所有的步骤尽可能并行，这也是得益于 Go 当中多线程共享内存的优势。
3. **从零造轮子**。 几乎没有使用任何第三方库，所有逻辑自己编写，大到 AST 解析，小到字符串的操作，保证极致的代码性能。
4. **高效的内存利用**。Esbuild 中从头到尾尽可能地复用一份 AST 节点数据，而不用像 JS 打包工具中频繁地解析和传递 AST 数据（如 string -> TS -> JS -> string)，造成内存的大量浪费。

### 使用

#### 项目打包——build API

```js
const { build, buildSync, serve } = require("esbuild");

async function runBuild() {
  // 异步方法，返回一个 Promise
  const result = await build({
    // ----  如下是一些常见的配置  ---
    // 当前项目根目录
    absWorkingDir: process.cwd(),
    // 入口文件列表，为一个数组
    entryPoints: ["./src/index.jsx"],
    // 打包产物目录
    outdir: "dist",
    // 是否需要打包，一般设为 true
    bundle: true,
    // 模块格式，包括`esm`、`commonjs`和`iife`
    format: "esm",
    // 需要排除打包的依赖列表
    external: [],
    // 是否开启自动拆包
    splitting: true,
    // 是否生成 SourceMap 文件
    sourcemap: true,
    // 是否生成打包的元信息文件
    metafile: true,
    // 是否进行代码压缩
    minify: false,
    // 是否开启 watch 模式，在 watch 模式下代码变动则会触发重新打包
    watch: false,
    // 是否将产物写入磁盘
    write: true,
    // Esbuild 内置了一系列的 loader，包括 base64、binary、css、dataurl、file、js(x)、ts(x)、text、json
    // 针对一些特殊的文件，调用不同的 loader 进行加载
    loader: {
      ".png": "base64",
    },
  });
  console.log(result);
}

runBuild();
```

#### 静态资源——serve API

1. 开启 serve 模式后，将在指定的端口和目录上搭建一个`静态文件服务`，这个服务器用原生 Go 语言实现，性能比 Nodejs 更高
2. 类似 webpack-dev-server，所有的产物文件都默认不会写到磁盘，而是放在内存中，通过请求服务来访问
3. **每次请求**到来时，都会进行重新构建(`rebuild`)，永远返回新的产物。

```js
const { build, buildSync, serve } = require("esbuild");

function runBuild() {
  serve(
    {
      port: 8000,
      // 静态资源目录
      servedir: "./dist",
    },
    {
      absWorkingDir: process.cwd(),
      entryPoints: ["./src/index.jsx"],
      bundle: true,
      format: "esm",
      splitting: true,
      sourcemap: true,
      ignoreAnnotations: true,
      metafile: true,
    }
  ).then((server) => {
    console.log("HTTP Server starts at port", server.port);
  });
}

runBuild();
```

#### 单文件转译——transform API

```js
const { transform, transformSync } = require("esbuild");

async function runTransform() {
  // 第一个参数是代码字符串，第二个参数为编译配置
  const content = await transform(
    "const isNull = (str: string): boolean => str.length > 0;",
    {
      sourcemap: true,
      loader: "tsx",
    }
  );
  console.log(content);
}

runTransform();
```

### 插件开发

有四个钩子：OnResolve、 OnLoad、 OnStart、 OnEnd

## Rollup 插件机制

Rollup 内部主要经历了  `Build`  和  `Output`  两大阶段：

- 首先，Build 阶段主要负责创建模块依赖图，初始化各个模块的 AST 以及模块之间的依赖关系。其实也就是调用 rollup 方法
- 真正进行打包的过程会在  `Output`  阶段进行，即在`bundle`对象的  `generate`或者`write`方法中进行。

### 拆解插件工作流

插件的各种 Hook 可以根据这两个构建阶段分为两类: `Build Hook`  与  `Output Hook`。

- `Build Hook`即在`Build`阶段执行的钩子函数，在这个阶段主要进行模块代码的转换、AST 解析以及模块依赖的解析，那么这个阶段的 Hook 对于代码的操作粒度一般为`模块`级别，也就是单文件级别。
- `Ouput Hook`(官方称为`Output Generation Hook`)，则主要进行代码的打包，对于代码而言，操作粒度一般为  `chunk`级别(一个 chunk 通常指很多文件打包到一起的产物)。

根据不同的 Hook 执行方式也会有不同的分类，主要包括`Async`、`Sync`、`Parallel`、`Squential`、`First`这五种。

#### Build 阶段工作流

![20230112103519](https://codertzm.oss-cn-chengdu.aliyuncs.com/20230112103519.png)

1. 首先经历 options 钩子进行配置的转换，得到处理后的配置对象。

2. 随之 Rollup 会调用 buildStart 钩子，正式开始构建流程。

3. Rollup 先进入到 resolveId 钩子中解析文件路径。(从 input 配置指定的入口文件开始)。

4. Rollup 通过调用 load 钩子加载模块内容。

5. 紧接着 Rollup 执行所有的 transform 钩子来对模块内容进行进行自定义的转换，比如 babel 转译。

6. 现在 Rollup 拿到最后的模块内容，进行 AST 分析，得到所有的 import 内容，调用 moduleParsed 钩子:
   1. 如果是普通的 import，则执行 resolveId 钩子，继续回到步骤 3。
   2. 如果是动态 import，则执行 resolveDynamicImport 钩子解析路径，如果解析成功，则回到步骤 4 加载模块，否则回到步骤 3 通过 resolveId 解析路径。

直到所有的 import 都解析完毕，Rollup 执行 buildEnd 钩子，Build 阶段结束。

当然，在 Rollup 解析路径的时候，即执行 resolveId 或者 resolveDynamicImport 的时候，有些路径可能会被标记为 external(翻译为排除)，也就是说不参加 Rollup 打包过程，这个时候就不会进行 load、transform 等等后续的处理了。

#### Output 工作流

![20230112103913](https://codertzm.oss-cn-chengdu.aliyuncs.com/20230112103913.png)

1. 执行所有插件的 outputOptions 钩子函数，对 output 配置进行转换。
2. 执行 renderStart，并发执行 renderStart 钩子，正式开始打包。
3. 从入口模块开始扫描，针对动态 import 语句执行 renderDynamicImport 钩子，来自定义动态 import 的内容。
4. 如果没有遇到 import.meta 语句，则进入下一步，否则:
   1. 对于 import.meta.url 语句调用 resolveFileUrl 来自定义 url 解析逻辑
   2. 对于其他 import.meta 属性，则调用 resolveImportMeta 来进行自定义的解析。
5. 串行执行所有插件的 banner、footer、intro、outro 钩子(底层用 Promise.all 包裹所有的这四种钩子函数)，这四个钩子功能很简单，就是往打包产物的固定位置(比如头部和尾部)插入一些自定义的内容，比如协议声明内容、项目介绍等等。
6. 接着 Rollup 会生成所有 chunk 的内容，针对每个 chunk 会依次调用插件的 renderChunk 方法进行自定义操作，也就是说，在这里时候你可以直接操作打包产物了。
7. 对每个即将生成的 chunk，执行 augmentChunkHash 钩子，来决定是否更改 chunk 的哈希值，在 watch 模式下即可能会多次打包的场景下，这个钩子会比较适用。
8. 随后会调用 generateBundle 钩子，这个钩子的入参里面会包含所有的打包产物信息，包括 chunk (打包后的代码)、asset(最终的静态资源文件)。你可以在这里删除一些 chunk 或者 asset，最终这些内容将不会作为产物输出。
9. 前面提到了 rollup.rollup 方法会返回一个 bundle 对象，这个对象是包含 generate 和 write 两个方法，两个方法唯一的区别在于后者会将代码写入到磁盘中，同时会触发 writeBundle 钩子，传入所有的打包产物信息，包括 chunk 和 asset，和 generateBundle 钩子非常相似。不过值得注意的是，这个钩子执行的时候，产物已经输出了，而 generateBundle 执行的时候产物还并没有输出。
10. 当上述的 bundle 的 close 方法被调用时，会触发 closeBundle 钩子，到这里 Output 阶段正式结束。

### 常用 Hook 实战

常用 Hook 主要有：

- resolveId: 一般用来解析模块路径
- load: 通过 resolveId 解析后的路径来加载模块内容
- transform: 对加载后的模块内容进行自定义的转换
- renderChunk: Chunk 级代码修改
- generateBundle: 产物生成最后一步

TODO

---

Rollup 的插件体系非常灵活，主要体现在：

1. 插件逻辑集中管理
2. 插件 API 简洁，符合直觉
3. 插件间的互相调用

## 开发一个 vite 插件

- config: 用来进一步修改配置。
- configResolved: 用来记录最终的配置信息。
- configureServer: 用来获取 Vite Dev Server 实例，添加中间件。
- transformIndexHtml: 用来转换 HTML 的内容。
- handleHotUpdate: 用来进行热更新模块的过滤，或者进行自定义的热更新处理。

钩子顺序：

- 服务启动阶段: config、configResolved、options、configureServer、buildStart
- 请求响应阶段: 如果是 html 文件，仅执行 transformIndexHtml 钩子；对于非 HTML 文件，则依次执行 resolveId、load 和 transform 钩子。
- 热更新阶段: 执行 handleHotUpdate 钩子。
- 服务关闭阶段: 依次执行 buildEnd 和 closeBundle 钩子。

### 插件应用位置

Vite 会依次执行如下的插件:

- Alias (路径别名)相关的插件。
- ⭐️ 带有 enforce: 'pre' 的用户插件。
- Vite 核心插件。
- ⭐️ 没有 enforce 值的用户插件，也叫普通插件。
- Vite 生产环境构建用的插件。
- ⭐️ 带有 enforce: 'post' 的用户插件。
- Vite 后置构建插件(如压缩插件)。

## HMR API 及 原理

HMR 的全称叫做 `Hot Module Replacement`，即模块热替换或者模块热更新。

Vite 作为一个完整的构建工具，本身实现了一套 HMR 系统，值得注意的是，这套 HMR 系统基于原生的 ESM 模块规范来实现，在文件发生改变时 Vite 会侦测到相应 ES 模块的变化，从而触发相应的 API，实现局部的更新。

Vite 的 HMR API 设计也并非空穴来风，它基于一套完整的 [ESM HMR](https://github.com/FredKSchott/esm-hmr) 规范来实现，这个规范由同时期的 no-bundle 构建工具 Snowpack、WMR 与 Vite 一起制定，是一个比较通用的规范。

### 模块更新时逻辑-accept

在 import.meta.hot 对象上有一个非常关键的方法 accept，因为它决定了 Vite 进行热更新的边界

- 接受自身模块的更新
- 接受某个子模块的更新
- 接受多个子模块的更新

### 模块销毁时逻辑-dispose

这个方法相较而言就好理解多了，代表在模块更新、旧模块需要销毁时需要做的一些事情

### 数据共享-data

这个属性用来在不同的模块实例间共享一些数据。

## 代码分割(code splitting)

### 想要解决的问题

在传统的单 chunk 打包模式下，当项目代码越来越庞大，最后会导致浏览器下载一个巨大的文件，从页面加载性能的角度来说，主要会导致两个问题:

- 无法做到按需加载，即使是当前页面不需要的代码也会进行加载。
- 线上缓存复用率极低，改动一行代码即可导致整个 bundle 产物缓存失效。

### vite 默认拆包策略

- Vite 实现了自动 CSS 代码分割的能力，即实现一个 chunk 对应一个 css 文件
- Vite 基于 Rollup 的 manualChunksAPI 实现了应用拆包的策略:
  - 业务代码和第三方包代码分别打包为单独的 chunk,在 Vite 2.9 及以后的版本，默认打包策略更加简单粗暴，将所有的 js 代码全部打包到 index.js 中。
  - 对于 Async Chunk 而言 ，动态 import 的代码会被拆分成单独的 chunk，如上述的 Dynamic 组件。

### 自定义拆包策略

针对更细粒度的拆包，Vite 的底层打包引擎 Rollup 提供了 manualChunks，让我们能自定义拆包策略，但需注意解决循环引用

也可以使用插件 `vite-plugin-chunk-split`

## 语法降级和 Polyfill

这两类问题本质上是通过前端的编译工具链(如 Babel)及 JS 的基础 Polyfill 库(如 corejs)来解决的，不会跟具体的构建工具所绑定。

### 底层工具链

- 编译时工具。代表工具有@babel/preset-env 和@babel/plugin-transform-runtime。
- 运行时基础库。代表库包括 core-js 和 regenerator-runtime。

编译时工具的作用是在代码编译阶段进行语法降级及添加 polyfill 代码的引用语句

而运行时基础库是根据 ESMAScript 官方语言规范提供各种 Polyfill 实现代码，主要包括 core-js 和 regenerator-runtime 两个基础库

实际可以使用插件: `@vitejs/plugin-legacy`

## 模块联邦(Module Federation)

### 痛点

复用：

1. 发布 npm 包

   1. 开发效率问题，每次改动都需要发版，并所有相关的应用安装新依赖，流程比较复杂
   2. 项目构建问题，引入了公共库之后，公共库的代码都需要打包到项目最后的产物后，导致产物体积偏大，构建速度相对较慢。

2. Git Submodule
   1. 缺点和上个方式几乎一样
3. 依赖外部化(external)+ CDN 引入
   1. 兼容性问题
   2. 依赖顺序问题
   3. 产物体积问题
4. Monorepo
   1. 所有的应用代码必须放到同一个仓库。
   2. Monorepo 本身也存在一些天然的局限性，如项目数量多起来之后依赖安装时间会很久、项目整体构建时间会变长等等，我们也需要去解决这些局限性所带来的的开发效率问题。
   3. 项目构建问题。跟 发 npm 包的方案一样，所有的公共代码都需要进入项目的构建流程中，产物体积还是会偏大。

### 概念

模块联邦中主要有两种模块: 本地模块和远程模块。

本地模块即为普通模块，是当前构建流程中的一部分，而远程模块不属于当前构建流程，在本地模块的运行时进行导入，同时本地模块和远程模块可以共享某些依赖的代码，

![20230313155243](https://codertzm.oss-cn-chengdu.aliyuncs.com/20230313155243.png)

值得强调的是，在模块联邦中，每个模块既可以是本地模块，导入其它的远程模块，又可以作为远程模块，被其他的模块导入。

![20230313155322](https://codertzm.oss-cn-chengdu.aliyuncs.com/20230313155322.png)

优势：

1. 实现任意粒度的模块共享
2. 优化构建产物体积
3. 运行时按需加载
4. 第三方依赖共享

可以使用 `vite-plugin-federation` 插件来实现

## 性能优化

### 1. 网络优化

1. http2:
   1. 多路复用。将数据分为多个二进制帧，多个请求和响应的数据帧在同一个 TCP 通道进行传输，解决了之前的队头阻塞问题。而与此同时，在 HTTP2 协议下，浏览器不再有同域名的并发请求数量限制，因此请求排队问题也得到了解决。
   2. Server Push，即服务端推送能力。可以让某些资源能够提前到达浏览器，比如对于一个 html 的请求，通过 HTTP 2 我们可以同时将相应的 js 和 css 资源推送到浏览器，省去了后续请求的开销。
2. DNS 预解析:
   浏览器在向跨域的服务器发送请求时，首先会进行 DNS 解析，将服务器域名解析为对应的 IP 地址。我们通过 dns-prefetch 技术将这一过程提前，降低 DNS 解析的延迟时间

   ```html
   <link rel="preconnect" href="https://fonts.gstatic.com/" crossorigin />
   <link rel="dns-prefetch" href="https://fonts.gstatic.com/" />
   ```

3. Preload/Prefetch

对于一些比较重要的资源，我们可以通过 Preload 方式进行预加载，即在资源使用之前就进行加载，而不是在用到的时候才进行加载，这样可以使资源更早地到达浏览器。

```html
<link rel="preload" href="style.css" as="style" />
<link rel="preload" href="main.js" as="script" />
```

与普通 script 标签不同的是，对于原生 ESM 模块，浏览器提供了 modulepreload 来进行预加载:

```html
<link rel="modulepreload" href="/src/app.js" />
```

### 2. 资源优化

1. 产物分析报告

通过 `rollup-plugin-visualizer` 插件生成产物分析报告

2. 资源压缩

在生产环境中，为了极致的代码体积，我们一般会通过构建工具来对产物进行压缩。具体来说，有这样几类资源可以被压缩处理: JavaScript 代码、CSS 代码和图片文件。

### 3. 预渲染优化

预渲染是当今比较主流的优化手段，主要包括服务端渲染(SSR)和静态站点生成(SSG)这两种技术。

在 SSR 的场景下，服务端生成好完整的 HTML 内容，直接返回给浏览器，浏览器能够根据 HTML 渲染出完整的首屏内容，而不需要依赖 JS 的加载，从而降低浏览器的渲染压力；而另一方面，由于服务端的网络环境更优，可以更快地获取到页面所需的数据，也能节省浏览器请求数据的时间。

而 SSG 可以在构建阶段生成完整的 HTML 内容，它与 SSR 最大的不同在于 HTML 的生成在构建阶段完成，而不是在服务器的运行时。SSG 同样可以给浏览器完整的 HTML 内容，不依赖于 JS 的加载，可以有效提高页面加载性能。不过相比 SSR，SSG 的内容往往动态性不够，适合比较静态的站点，比如文档、博客等场景。
