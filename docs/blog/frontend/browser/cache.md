# 浏览器缓存

什么是缓存呢？

当我们第一次访问网站的时候，比如 juejin.cn，电脑会把网站上的图片和数据下载到电脑上，当我们再次访问该网站的时候，网站就会从电脑中直接加载出来，这就是缓存。

缓存有哪些好处？

1. 缓解服务器压力，不用每次都去请求某些数据了。

2. 提升性能，打开本地资源肯定会比请求服务器来的快。

3. 减少带宽消耗，当我们使用缓存时，只会产生很小的网络消耗，至于为什么打开本地资源也会产生网络消耗，下面会有说明。

**浏览器缓存过程： 强缓存，协商缓存**。

浏览器缓存位置一般分为四类： Service Worker-->Memory Cache-->Disk Cache-->Push Cache

## 强缓存

强缓存是当我们访问 URL 的时候，不会向服务器发送请求，直接从缓存中读取资源，但是会返回 200 的状态码。

### 如何设置强缓存

我们第一次进入页面，请求服务器，然后服务器进行应答，浏览器会根据 response Header 来判断是否对资源进行缓存，如果响应头中 expires、pragma 或者 cache-control 字段，代表这是强缓存，浏览器就会把资源缓存在 memory cache 或 disk cache 中。

![image](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220306145630.png)

### 控制缓存的字段

主要有以下几个：expires、Cache-control、pragma

#### expires

是 HTTP1.0 控制网页缓存的字段，值为一个时间戳，准确来讲是格林尼治时间，服务器返回该请求结果缓存的到期时间，意思是，再次发送请求时，如果未超过过期时间，直接使用该缓存，如果过期了则重新请求。
有个缺点，就是它判断是否过期是用本地时间来判断的，本地时间是可以自己修改的。

#### Cache-Control

是 HTTP1.1 中控制网页缓存的字段，当 Cache-Control 都存在时，Cache-Control 优先级更高，主要取值为：

- public：资源客户端和服务器都可以缓存。
- privite：资源只有客户端可以缓存。
- no-cache：客户端缓存资源，但是是否缓存需要经过协商缓存来验证。
- no-store：不使用缓存。
- max-age：缓存保质期。

**Cache-Control 使用了 max-age 相对时间，解决了 expires 的问题。**

#### pragma

这个是 HTTP1.0 中禁用网页缓存的字段，其取值为 no-cache，和 Cache-Control 的 no-cache 效果一样。

## 缓存位置

上面我们说，强缓存我们会把资源房放到 memory cache 和 disk cache 中，那什么资源放在 memory cache，什么资源放在 disk cache 中？

**存存储图像和网页**等资源主要缓存在 disk cache，**操作系统缓存文件**等资源大部分都会缓存在 memory cache 中。具体操作浏览器自动分配，看谁的资源利用率不高就分给谁。

可以看到 memory cache 请求时间都是 0ms，这个是不是太神奇了，这方面我来梳理下。

查找浏览器缓存时会按顺序查找: Service Worker-->Memory Cache-->Disk Cache-->Push Cache。

### Service Worker

是运行在浏览器背后的独立线程，一般可以用来实现缓存功能。使用 Service Worker 的话，传输协议必须为 HTTPS。因为 Service Worker 中涉及到请求拦截，所以必须使用 HTTPS 协议来保障安全。Service Worker 的缓存与浏览器其他内建的缓存机制不同，它可以让我们自由控制缓存哪些文件、如何匹配缓存、如何读取缓存，并且缓存是持续性的。

### Memory Cache

内存中的缓存，主要包含的是当前中页面中已经抓取到的资源，例如页面上已经下载的样式、脚本、图片等。读取内存中的数据肯定比磁盘快，内存缓存虽然读取高效，可是缓存持续性很短，会随着进程的释放而释放。一旦我们关闭 Tab 页面，内存中的缓存也就被释放了。

### Disk Cache

存储在硬盘中的缓存，读取速度慢点，但是什么都能存储到磁盘中，**比之 Memory Cache 胜在容量和存储时效性上**。

在所有浏览器缓存中，Disk Cache 覆盖面基本是最大的。它会根据 HTTP Herder 中的字段判断哪些资源需要缓存，哪些资源可以不请求直接使用，哪些资源已经过期需要重新请求。并且即使在跨站点的情况下，相同地址的资源一旦被硬盘缓存下来，就不会再次去请求数据。**绝大部分的缓存都来自 Disk Cache**。

memory cache 要比 disk cache 快的多。举个例子：从远程 web 服务器直接提取访问文件可能需要 500 毫秒(半秒)，那么磁盘访问可能需要 10-20 毫秒，而内存访问只需要 100 纳秒，更高级的还有 L1 缓存访问(最快和最小的 CPU 缓存)只需要 0.5 纳秒。

### prefetch cache(预取缓存)

link 标签上带了 prefetch，再次加载会出现。

prefetch 是预加载的一种方式，被标记为 prefetch 的资源，**将会被浏览器在空闲时间加载**。

### Push Cache

Push Cache（推送缓存）是 HTTP/2 中的内容，**当以上三种缓存都没有命中时，它才会被使用**。它只在会话（Session）中存在，一旦会话结束就被释放，并且缓存时间也很短暂，在 Chrome 浏览器中只有 5 分钟左右，同时它也并非严格执行 HTTP 头中的缓存指令。

## 协商缓存

协商缓存就是强缓存失效后，浏览器携带缓存标识向服务器发送请求，由服务器根据缓存标识来决定是否使用缓存的过程。

![image](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220306145714.png)

主要有以下两种情况：

1. 协商缓存生效，返回 304

![image-20220216161804242](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20220216161804242.png)

2. 协商缓存失效，返回 200 和请求结果

![image-20220216161818821](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20220216161818821.png)

### 如何设置协商缓存

#### Last-Modified / If-Modified-Since

![last-modified](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220306145815.png)

Last-Modified 是服务器响应请求时，返回该资源文件在服务器最后被修改的时间。

If-Modified-Since 则是客户端再次发起该请求时，携带上次请求返回的 Last-Modified 值，通过此字段值告诉服务器该资源上次请求返回的最后被修改时间。服务器收到该请求，发现请求头含有 If-Modified-Since 字段，则会根据 If-Modified-Since 的字段值与该资源在服务器的最后被修改时间做对比，若服务器的资源最后被修改时间大于 If-Modified-Since 的字段值，则重新返回资源，状态码为 200；否则则返回 304，代表资源无更新，可继续使用缓存文件。

#### Etag / If-None-Match

![etag](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220306145848.png)

Etag 是服务器响应请求时，返回当前资源文件的一个唯一标识(由服务器生成)。

If-None-Match 是客户端再次发起该请求时，携带上次请求返回的唯一标识 Etag 值，通过此字段值告诉服务器该资源上次请求返回的唯一标识值。服务器收到该请求后，发现该请求头中含有 If-None-Match，则会根据 If-None-Match 的字段值与该资源在服务器的 Etag 值做对比，一致则返回 304，代表资源无更新，继续使用缓存文件；不一致则重新返回资源文件，状态码为 200。

> Etag / If-None-Match 优先级高于 Last-Modified / If-Modified-Since，同时存在则只有 Etag / If-None-Match 生效。

如果资源被重复生成，而内容不变，则 Etag 更精确

## 强缓存与协商缓存的区别

1. 强缓存不发请求到服务器，所以有时候资源更新了浏览器还不知道，但是协商缓存会发请求到服务器，所以资源是否更新，浏览器肯定知道。

2. 大部分 web 服务器都默认开启协商缓存。

## 缓存方案

目前的项目大多使用这种缓存方案的：

- HTML: 协商缓存；

- css、js、图片：强缓存，文件名带上 hash。

## 刷新对于强缓存和协商缓存的影响

1. 当 ctrl+f5 强制刷新网页时，直接从服务器加载，跳过强缓存和协商缓存。

2. 当 f5 刷新网页时，跳过强缓存，但是会检查协商缓存。

3. 浏览器地址栏中写入 URL，回车 浏览器发现缓存中有这个文件了，不用继续请求了，直接去缓存拿。强缓存有效，协商缓存有效。

## 总结

![image-20220216161428643](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20220216161428643.png)

首先通过 `Cache-Control` 验证强缓存是否可用

- 如果强缓存可用，直接使用
- 否则进入协商缓存，即发送 HTTP 请求，服务器通过请求头中的\`If-Modified-Since`或者`If-None-Match`这些**条件请求**字段检查资源是否更新
  - 若资源更新，返回资源和 200 状态码
  - 否则，返回 304，告诉浏览器直接从缓存获取资源
