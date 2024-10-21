# nextjs 渲染

## 几种渲染方式

### CSR

**CSR，英文全称“Client-side Rendering”，中文翻译“客户端渲染”。顾名思义，渲染工作主要在客户端执行。**

像我们传统使用 React 的方式，就是客户端渲染。浏览器会先下载一个非常小的 HTML 文件和所需的 JavaScript 文件。在 JavaScript 中执行发送请求、获取数据、更新 DOM 和渲染页面等操作。

这样做最大的问题就是不够快。（SEO 问题是其次，现在的爬虫已经普遍能够支持 CSR 渲染的页面）

在下载、解析、执行 JavaScript以及请求数据没有返回前，页面不会完全呈现。

### SSR

**SSR，英文全称“Server-side Rendering”，中文翻译“服务端渲染”。顾名思义，渲染工作主要在服务端执行。**

比如打开一篇博客文章页面，没有必要每次都让客户端请求，万一客户端网速不好呢，那干脆由服务端直接请求接口、获取数据，然后渲染成静态的 HTML 文件返回给用户。

虽然同样是发送请求，但通常服务端的环境（网络环境、设备性能）要好于客户端，所以最终的渲染速度（首屏加载时间）也会更快。

虽然总体速度是更快的，但因为 CSR 响应时只用返回一个很小的 HTML，SSR 响应还要请求接口，渲染 HTML，所以其响应时间会更长，对应到性能指标 TTFB (Time To First Byte)，SSR 更长。

### SSG

**SSG，英文全称“Static Site Generation”，中文翻译“静态站点生成”。**

SSG 会在构建阶段，就将页面编译为静态的 HTML 文件。

比如打开一篇博客文章页面，既然所有人看到的内容都是一样的，没有必要在用户请求页面的时候，服务端再请求接口。干脆先获取数据，提前编译成 HTML 文件，等用户访问的时候，直接返回 HTML 文件。这样速度会更快。再配上 CDN 缓存，速度就更快了。

所以能用 SSG 就用 SSG。“在用户访问之前是否能预渲染出来？”如果能，就用 SSG。

### ISR

**ISR，英文全称“Incremental Static Regeneration”，中文翻译“增量静态再生”。**

还是打开一篇博客文章页面，博客的主体内容也许是不变的，但像比如点赞、收藏这些数据总是在变化的吧。使用 SSG 编译成 HTML 文件后，这些数据就无法准确获取了，那你可能就退而求其次改为 SSR 或者 CSR 了。

考虑到这种情况，Next.js 提出了 ISR。当用户访问了这个页面，第一次依然是老的 HTML 内容，但是 Next.js 同时静态编译成新的 HTML 文件，当你第二次访问或者其他用户访问的时候，就会变成新的 HTML 内容了。

Next.js v9.5 就发布了稳定的 ISR 功能，这是当时提供的 [demo](https://link.juejin.cn/?target=https%3A%2F%2Freactions-demo.vercel.app%2F "https://reactions-demo.vercel.app/") 效果：

![reactions-demo.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/26964dd0d6c14517abe5aa90fca2bba6~tplv-k3u1fbpfcp-jj-mark:2495:0:0:0:q75.awebp#?w=1434&h=1464&s=1406805&e=gif&f=226&b=fcfcfc)


## SSR 和 RSC

### React Server Component

React Server Component 把数据请求的部分放在服务端，由服务端直接给客户端返回带数据的组件。

最终的目标是：在原始只有 Client Components 的情况下，一个 React 树的结构如下：

![image.png](https://codertzm.oss-cn-chengdu.aliyuncs.com/20241021160734.png)

在使用 React Server Component 后，React 树会变成：

![image.png](https://codertzm.oss-cn-chengdu.aliyuncs.com/20241021160746.png)

其中黄色节点表示 React Server Component。在服务端，React 会将其渲染会一个包含基础 HTML 标签和 **客户端组件占位** 的树。它的结构类似于：

![image.png](https://codertzm.oss-cn-chengdu.aliyuncs.com/20241021160833.png)

因为客户端组件的数据和结构在客户端渲染的时候才知道，所以客户端组件此时在树中使用特殊的占位进行替代。

当然这个树不可能直接就发给客户端，React 会做序列化处理，客户端收到后会在客户端根据这个数据重构 React 树，然后用真正的客户端组件填充占位，渲染最终的结果。

![image.png](https://codertzm.oss-cn-chengdu.aliyuncs.com/20241021160855.png)


使用 React Server Component，因为服务端组件的代码不会打包到客户端代码中，它可以减小包（bundle）的大小。且在 React Server Component 中，可以直接访问后端资源。当然因为在服务端运行，对应也有一些限制，比如不能使用 useEffect 和客户端事件等。

### RSC 和 SSR 的比较

了解了 RSC 和 SSR 这两个基本概念，现在让我们来回顾下。表面上看，RSC 和 SSR 非常相似，都发生在服务端，都涉及到渲染，目的都是更快的呈现内容。但实际上，这两个技术概念是相互独立的。RSC 和 SSR 既可以各自单独使用，又可以搭配在一起使用（搭配在一起使用的时候是互补的）。

正如它们的名字所表明的那样，Server-side Rendering 的重点在于 **Rendering**，React Server Components 的重点在于 **Components**。

简单来说，RSC 提供了更细粒度的组件渲染方式，可以在组件中直接获取数据，而非像 Next.js v12 中的 SSR 顶层获取数据。RSC 在服务端进行渲染，组件依赖的代码不会打包到 bundle 中，而 SSR 需要将组件的所有依赖都打包到 bundle 中。

当然两者最大的区别是：

SSR 是在服务端将组件渲染成 HTML 发送给客户端，而 RSC 是将组件渲染成一种特殊的格式，我们称之为 RSC Payload。这个 RSC Payload 的渲染是在服务端，但不会一开始就返回给客户端，而是在客户端请求相关组件的时候才返回给客户端，RSC Payload 会包含组件渲染后的数据和样式，客户端收到 RSC Payload 后会重建 React 树，修改页面 DOM。

## Suspense 和 Streaming 

使用 SSR，需要经过一系列的步骤，用户才能查看页面、与之交互。具体这些步骤是：

1. 服务端获取所有数据
2. 服务端渲染 HTML
3. 将页面的 HTML、CSS、JavaScript 发送到客户端
4. 使用 HTML 和 CSS 生成不可交互的用户界面（non-interactive UI）
5. React 对用户界面进行水合（hydrate），使其可交互（interactive UI）

SSR 的缺点：

1. SSR 的数据获取必须在组件渲染之前
2. 组件的 JavaScript 必须先加载到客户端，才能开始水合
3. 所有组件必须先水合，然后才能跟其中任意一个组件交互

### Suspense

为了解决这些问题，React 18 引入了 Suspense 组件。它可以允许你推迟渲染某些内容，直到满足某些条件（例如数据加载完毕）。

你可以将动态组件包装在 Suspense 中，然后向其传递一个 fallback UI，以便在动态组件加载时显示。如果数据请求缓慢，使用 Suspense 流式渲染该组件，不会影响页面其他部分的渲染，更不会阻塞整个页面。

### Streaming

Suspense 背后的这种技术称之为 Streaming。将页面的 HTML 拆分成多个 chunks，然后逐步将这些块从服务端发送到客户端。

![image.png](https://codertzm.oss-cn-chengdu.aliyuncs.com/20241021161906.png)

这样就可以更快的展现出页面的某些内容，而无需在渲染 UI 之前等待加载所有数据。提前发送的组件可以提前开始水合，这样当其他部分还在加载的时候，用户可以和已完成水合的组件进行交互，有效改善用户体验。

![image.png](https://codertzm.oss-cn-chengdu.aliyuncs.com/20241021161920.png)

