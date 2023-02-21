---
title: nginx简单配置
---

## 前置

你需要的：

- 一个云服务器
- linux基本命令
- 相关工具
- 本文基于ubuntu20.04

## 安装nginx

```shell
sudo apt-get install nginx
```

查看nginx是否安装成功

```shell
nginx -v
```

启动nginx

```shell
service nginx start
```

## nginx相关文件路径

```
/etc/nginx/ // 配置文件
/usr/share/nginx/ // 默认前端代码存放处
```

默认配置在/etc/nginx/nginx.conf,nginx根据默认配置，监听80端口，80端口指定了/usr/share/nginx/html这个文件夹，于是你访问公网ip -> 公网ip:80 -> /usr/share/nginx/html。

## 在7777端口上搭建一个站点

我们的代码除了部署在默认80端口上面，还可以其他端口上，例如8888，7777，等等

​接下来我们就在一个自定义的端口上面部署一个站点

这里我们需要注意nginx.conf中的一行配置

根据配置文件的提示我们到/etc/nginx/conf.d/下面，建立test.conf，nginx.conf根据通配符匹配conf.d/下的配置文件

通过vim写入内容

```
server {
    listen       7777;
    server_name  _;

    root /usr/share/nginx/test;
}
```

再去文件夹/usr/share/nginx/test下建立一个index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <div>我是7777端口</div>
</body>
</html>

```

重启nginx

```shell
nginx -s reload
```

在阿里云防火墙中添加安全组规则

之后就能正常访问了
