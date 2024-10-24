---
title: nginx简单配置
---

## 前置

你需要的：

- 一个云服务器
- linux 基本命令
- 相关工具
- 本文基于 ubuntu20.04

## 安装 nginx

```shell
sudo apt-get install nginx
```

查看 nginx 是否安装成功

```shell
nginx -v
```

启动 nginx

```shell
service nginx start
```

## nginx 相关文件路径

```txt
/etc/nginx/ // 配置文件
/usr/share/nginx/ // 默认前端代码存放处
```

默认配置在/etc/nginx/nginx.conf,nginx 根据默认配置，监听 80 端口，80 端口指定了/usr/share/nginx/html 这个文件夹，于是你访问公网 ip -> 公网 ip:80 -> /usr/share/nginx/html。

## 在 7777 端口上搭建一个站点

我们的代码除了部署在默认 80 端口上面，还可以其他端口上，例如 8888，7777，等等

​ 接下来我们就在一个自定义的端口上面部署一个站点

这里我们需要注意 nginx.conf 中的一行配置

根据配置文件的提示我们到/etc/nginx/conf.d/下面，建立 test.conf，nginx.conf 根据通配符匹配 conf.d/下的配置文件

通过 vim 写入内容

```txt
server {
    listen       7777;
    server_name  _;

    root /usr/share/nginx/test;
}
```

在上面配置文件中：

- location 用来匹配 url
- root 将 URL 请求的路径附加到 root 指定的路径后。

再去文件夹/usr/share/nginx/test 下建立一个 index.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    <div>我是7777端口</div>
  </body>
</html>
```

重启 nginx

```shell
nginx -s reload
```

在阿里云防火墙中添加安全组规则

之后就能正常访问了

## nginx 常用命令

```bash
nginx -s quit/reload/reopen/start // nginx取消/重新加载配置/重新启动/开启
nginx -t // 定位nginx配置位置
```

## nginx 配置

nginx 配置可以分为三个块，分别是

- 全局块：全局配置，比如 workder 进程数量
- events 块：服务器和客户端之间网络连接的配置，比如指定每个 worker 进程可以同时接收多少个网络连接
- http 块：虚拟主机（server）、反向代理、负载均衡都是在这里配置

## 反向代理

在 `nginx.conf` 中进行配置

比如，现在我们通过 `node` 开启了三个服务，分别是 8000、8001、8002 端口，我们需要在 nginx.conf 中进行配置：

```txt
upstream backend {
  server 127.0.0.1:8000;
  server 127.0.0.1:8001;
  server 127.0.0.1:8002;
}
```

> 这里的 upstream 后面的 backend 是命名，可以换别的名字

然后在 `server` 块中添加 `location` 配置

```txt
server {
  ...

  location /app {
    proxy_pass http://backend;
  }

  ...
}
```

上面这段配置：让所有以 `app` 开始的请求都被代理到刚刚配置的 `upstream` 中

## 负载均衡

服务器性能配置可能会有所不同，通过调整负载均衡的策略来实现负载均衡

### weight 权重

可以使用权重 `weight` 来调整负载均衡的策略，权重越大，被分配到的请求次数也就越多。

修改一下上面的配置：

```txt
upstream backend {
  server 127.0.0.1:8000 weight=3;
  server 127.0.0.1:8001;
  server 127.0.0.1:8002;
}
```

### ip_hash

这个策略可以根据客户端的 ip 地址来进行哈希，同一个客户端的请求就会被分配到同一个服务器上，能够解决 session 相关的问题

```txt
upstream backend {
  ip_hash;
  server 127.0.0.1:8000 weight=3;
  server 127.0.0.1:8001;
  server 127.0.0.1:8002;
}
```

## https 配置

https 协议是安全版本的 http 协议，它的默认端口是 443，同时需要使用到 SSL 证书。

证书申请：

- 云平台：可以在主流的云平台中申请 SSL 证书，证书申请到之后，会得到密钥文件和证书文件
- 没有云平台：通过 openssl 来生成证书

  ![](https://codertzm.oss-cn-chengdu.aliyuncs.com/20241024173738.png)

然后在 nginx 配置中进行配置：

![](https://codertzm.oss-cn-chengdu.aliyuncs.com/20241024174219.png)

配置 http 重定向到 https：

![](https://codertzm.oss-cn-chengdu.aliyuncs.com/20241024174123.png)
