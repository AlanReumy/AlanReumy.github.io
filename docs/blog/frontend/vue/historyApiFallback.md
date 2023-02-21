---
title: historyApiFallback使用
---

## 浏览器请求静态资源的过程

在输入 url 后，会经过 dns 域名解析成 ip 地址，然后去向服务器请求静态资源，此时，服务器需要作 nginx 配置，找到静态资源的文件夹。服务器将静态资源返回给浏览器。

## 例子

假设你现在已经来到了 `xxx.org` 这个网站，此时进入了一个子路径（假如是分类），那么此时的 url 就是 `xxx.org/category` 。

如果现在刷新浏览器，意味着浏览器将会以这个 url 去向服务器请求获取到静态资源，但是此时服务器的 nginx 配置一般是不会给这种子路径作配置的，因此此时会返回找不到静态资源的提示。

## 使用 webpack 中的 historyApiFallback 解决

这个时候就需要使用 `webpack` 中的 `historyApiFallback` 去解决这个问题。

当我们将这个属性配置为 `true` 时，那么在刷新返回 404 错误的时候，就会自动返回 `index.html` 中的内容。

用 `vue-cli` 创建项目的时候，在脚手架的源码里，我们可以看到这样的配置：

![](https://codertzm.oss-cn-chengdu.aliyuncs.com/QQ截图20220119173329.png)

## vue.config.js 中如何配置

```js
module.exports = {
  configureWebpack: {
    devServer: {
      historyApiFallback: true,
    },
  },
};
```
