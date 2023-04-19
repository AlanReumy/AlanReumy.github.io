# 浏览器安全

## 什么是同源策略

如果两个 URL 的协议、域名和端口都相同，我们就称这两个 URL 同源。浏览器默认两个相同的源之间是可以相互访问资源和操作 DOM 的。两个不同的源之间若想要相互访问资源或者操作 DOM，那么会有一套基础的安全策略的制约，我们把这称为同源策略。

同源策略主要表现在以下三个方面：

1. DOM: 同源策略限制了来自不同源的 JavaScript 脚本对当前 DOM 对象读和写的操作
2. 数据: 同源策略限制了不同源的站点读取当前站点的 Cookie、IndexDB、LocalStorage 等数据
3. 网络: 限制了通过 XMLHttpRequest 等方式将站点的数据发送给不同源的站点

## XSS 攻击

XSS 攻击全名：Cross Site Scripting,跨站脚本攻击,为了与“CSS”区分开来，故简称 XSS

页面可能会被恶意注入 js 脚本，但是浏览器无法分辨哪些是恶意注入的脚本，哪些是正常内容。所以恶意注入的脚本也有了正常 js 的权限。

恶意脚本能做的事情：

1. 窃取 cookie 信息
2. 监听用户行为
3. 通过修改 DOM，伪造假的登录窗口，窃取用户名和密码
4. 在页面内生成浮窗广告，影响用户体验

### XSS 攻击分类

#### 1. 存储型攻击

所谓存储型攻击就是将恶意代码上传存储到有漏洞的服务器中，它大概会经过如下步骤：

1. 首先黑客利用站点漏洞将一段恶意 js 代码提交到网站的数据库中；
2. 然后用户向网站请求包含了恶意 js 脚本的页面；
3. 当用户浏览该页面的时候，恶意脚本就会将用户的 cookie 信息等数据上传到服务器。

#### 2. 反射型攻击

在一个反射型 XSS 攻击过程中，恶意 JavaScript 脚本属于用户发送给网站请求中的一部分，随后网站又把恶意 JavaScript 脚本返回给用户。当恶意 JavaScript 脚本在用户页面中被执行时，黑客就可以利用该脚本做一些恶意操作。

下面是一个用 expres 实现的服务器:

```js
var express = require("express");
var router = express.Router();

/* GET home page. */
router.get("/", function (req, res, next) {
  res.render("index", { title: "Express", xss: req.query.xss });
});

module.exports = router;
```

这是视图代码：

```html
<!DOCTYPE html>
<html>
  <head>
    <title><%= title %></title>
    <link rel="stylesheet" href="/stylesheets/style.css" />
  </head>
  <body>
    <h1><%= title %></h1>
    <p>Welcome to <%= title %></p>
    <div><%- xss %></div>
  </body>
</html>
```

如果用户通过这个链接：`http://localhost:3000/?xss=<script>alert('你被xss攻击了')</script>` 进行访问，就被 XSS 反射型攻击了

在现实生活中，黑客经常会通过 QQ 群或者邮件等渠道诱导用户去点击这些恶意链接

#### 3. 基于 DOM 的 XSS 攻击

基于 DOM 的 XSS 攻击是不牵涉到页面 Web 服务器的。具体来讲，黑客通过各种手段将恶意脚本注入用户的页面中，比如通过网络劫持在页面传输过程中修改 HTML 页面的内容

### 如何防止 XSS 攻击

#### 1. 服务器对输入脚本进行过滤或转义

比如下面这段：

```txt
code:<script>alert('你被 xss 攻击了')</script>
```

在经过过滤后只留下了 code，其余的都被过滤掉了，其次它也可以通过被转义来达到相同的效果

```txt
code:&lt;script&gt;alert(&#39; 你被 xss 攻击了 &#39;)&lt;/script&gt;
```

#### 2. 充分利用 CSP

CSP 即内容安全策略，它的功能如下：

1. 限制加载其他域下的资源文件
2. 禁止向第三方域提交数据，这样用户数据也不会外泄
3. 禁止执行内联脚本和未授权的脚本
4. 还提供了上报机制，这样可以帮助我们尽快发现有哪些 XSS 攻击，以便尽快修复问题

#### 3. 使用 HttpOnly 属性

通常服务器可以将某些 Cookie 设置为 HttpOnly 标志，HttpOnly 是服务器通过 HTTP 响应头来设置的

```txt
set-cookie: NID=189=M8q2FtWbsR8RlcldPVt7qkrqR38LmFY9jUxkKo3-4Bi6Qu_ocNOat7nkYZUTzolHjFnwBw0izgsATSI7TZyiiiaV94qGh-BzEYsNVa7TZmjAYTxYTOM9L_-0CN9ipL6cXi8l6-z41asXtm2uEwcOC5oh9djkffOMhWqQrlnCtOI; expires=Sat, 18-Apr-2020 06:52:22 GMT; path=/; domain=.google.com; HttpOnly
```

在响应头的 `set-cookie` 中我们可以看到 `HttpOnly`，这会让 js 无法读取到这段 cookie

## CSRF 攻击

CSRF 英文全称是 Cross-site request forgery，所以又称为“跨站请求伪造”，是指黑客引诱用户打开黑客的网站，在黑客的网站中，利用用户的登录状态发起的跨站请求。简单来讲，CSRF 攻击就是黑客利用了用户的登录状态，并通过第三方的站点来做一些坏事

### 攻击分类

#### 1.自动 Get 请求

#### 2. 自动 POST 请求

#### 3. 引诱用户点击链接

发起 CSRF 攻击的条件：

1. 目标站点存在 CSRF 漏洞
2. 用户登录过目标站点
3. 需要用户打开第三方站点

### 如何避免 CSRF 攻击

1. 充分利用好 Cookie 的 SameSite 属性
2. 验证请求的来源站点
3. CSRF Token
