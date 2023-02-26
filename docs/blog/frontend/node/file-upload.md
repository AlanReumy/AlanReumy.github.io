# node 文件上传

本文介绍从基础文件读写到node的原生文件上传以及express与koa的文件上传

## node中的fs

fs是File System的缩写，表示文件系统。

- 对于任何一个为服务器端服务的语言或者框架通常都会有自己的文件系统：
  - 因为服务器需要将各种数据、文件等放置到不同的地方；
  - 比如用户数据可能大多数是放到数据库中的；
  - 比如某些配置文件或者用户资源（图片、音视频）都是以文件的形式存在于操作系统上的；
- Node也有自己的文件系统操作模块，就是fs：
  - 借助于Node帮我们封装的文件系统，我们可以在任何的操作系统（window、Mac OS、Linux）上面直接去操作文件；
  - 这也是Node可以开发服务器的一大原因，也是它可以成为前端自动化脚本等热门工具的原因；

### fs中的API

fs中的大多数API都提供三种操作方式：

- 方式一：同步操作文件：代码会被阻塞，不会继续执行；

    ```js
    const fs = require('fs');
    const filepath = "./abc.txt";

    // 1.方式一: 同步操作
    const info = fs.statSync(filepath);
    console.log("后续需要执行的代码");
    console.log(info);
    ```

- 方式二：异步回调函数操作文件：代码不会被阻塞，需要传入回调函数，当获取到结果时，回调函数被执行；

  ```js
  // 2.方式二: 异步操作
  fs.stat(filepath, (err, info) => {
    if (err) {
      console.log(err);
      return;
    }
    console.log(info);
    console.log(info.isFile());
    console.log(info.isDirectory());
  });
  console.log("后续需要执行的代码");
  ```

- 方式三：异步Promise操作文件：代码不会被阻塞，通过 fs.promises 调用方法操作，会返回一个Promise，可以通过then、catch进行处理；

  ```js
  // 3.方式三: Promise
  fs.promises.stat(filepath).then(info => {
    console.log(info);
  }).catch(err => {
    console.log(err);
  });

  console.log("后续需要执行的代码");
  ```

### 文件的读写

如果我们希望对文件的内容进行操作，这个时候可以使用文件的读写：

- `fs.readFile(path[, options], callback)`：读取文件的内容；
- `fs.writeFile(file, data[, options], callback)`：在文件中写入内容；

其中：`options`的选项有两个比较常用：flag(写入方式)、encoding(编码方式)

#### flag选项

flag的值有很多：[flag](https://nodejs.org/dist/latest-v14.x/docs/api/fs.html#fs_file_system_flags)

- w 打开文件写入，默认值；
- w+ 打开文件进行读写，如果不存在则创建文件；
- r+ 打开文件进行读写，如果不存在那么抛出异常；
- r 打开文件读取，读取时的默认值；
- a 打开要写入的文件，将流放在文件末尾。如果不存在则创建文件；
- a+ 打开文件以进行读写，将流放在文件末尾。如果不存在则创建文件

#### encoding选项

如果不填写encoding，返回的结果是Buffer

### 文件夹的操作

- 新建文件夹

  ```js
  // 创建文件夹
  const dirname = './zim';
  if (!fs.existsSync(dirname)) {
    fs.mkdir(dirname, err => {
      console.log(err);
    });
  }
  ```

- 获取文件夹内容

  ```js
  // 读取文件夹下的所有文件
  function getFiles(dirname) {
    fs.readdir(dirname, { withFileTypes: true }, (err, files) => {
      for (let file of files) {
        if (file.isDirectory()) {
          const filepath = path.resolve(dirname, file.name);
          getFiles(filepath);
        } else {
          console.log(file.name);
        }
      }
    });
  }

  getFiles(dirname);
  ```

## node中的events

- Node中的核心API都是基于异步事件驱动的：
  - 在这个体系中，某些对象（发射器（Emitters））发出某一个事件；
  - 我们可以监听这个事件（监听器 Listeners），并且传入的回调函数，这个回调函数会在监听到事件时调用；
- 发出事件和监听事件都是通过EventEmitter类来完成的，它们都属于events对象。
  - emitter.on(eventName, listener)：监听事件，也可以使用addListener；
  - emitter.off(eventName, listener)：移除事件监听，也可以使用removeListener；
  - emitter.emit(eventName[, ...args])：发出事件，可以携带一些参数；

```js
const EventEmitter = require("events");

// 1.创建发射器
const emitter = new EventEmitter();

// 2.监听某一个事件
// addListener是on的alias简写
emitter.on('click', (args) => {
  console.log("监听1到click事件", args);
})

const listener2 = (args) => {
  console.log("监听2到click事件", args);
}
emitter.on('click', listener2)

// 3.发出一个事件
setTimeout(() => {
  emitter.emit("click", "coderwhy", "james", "kobe");
  // 取消事件监听
  emitter.off("click", listener2);
  emitter.emit("click", "coderwhy", "james", "kobe");
}, 2000);
```

### EventEmitter的实例有一些属性，可以记录一些信息

- `emitter.eventNames()`：返回当前 EventEmitter对象注册的事件字符串数组；
- `emitter.getMaxListeners()`：返回当前 EventEmitter对象的最大监听器数量，可以通过setMaxListeners()来修改，默认是10；
- `emitter.listenerCount(事件名称)`：返回当前 EventEmitter对象某一个事件名称，监听器的个数；
- `emitter.listeners(事件名称)`：返回当前 EventEmitter对象某个事件监听器上所有的监听器数组；

### 其他方法

- `emitter.once(eventName, listener)`：事件监听一次
- `emitter.prependListener()`：将监听事件添加到最前面
- `emitter.prependOnceListener()`：将监听事件添加到最前面，但是只监听一次
- `emitter.removeAllListeners([eventName])`：移除所有的监听器

## node中的Stream

Stream：

- 是连续字节的一种表现形式和抽象概念；
- 流应该是可读的，也是可写的；

我们可以直接通过 `fs.readFile` 或者 `fs.writeFile` 方式读写文件，为什么还需要流呢？

- 直接读写文件的方式，虽然简单，但是无法控制一些细节的操作；
- 比如从什么位置开始读、读到什么位置、一次性读取多少个字节；
- 读到某个位置后，暂停读取，某个时刻恢复读取等等；
- 或者这个文件非常大，比如一个视频文件，一次性全部读取并不合适；

文件读写的Stream

Node中很多对象是基于流实现的：

- http模块的Request和Response对象；
- process.stdout对象；

> 所有的流都是EventEmitter的实例

Node.js中有四种基本流类型：

- Writable：可以向其写入数据的流（例如 `fs.createWriteStream()`）。
- Readable：可以从中读取数据的流（例如 `fs.createReadStream()`）。
- Duplex：同时为Readable和的流Writable（例如 `net.Socket`）。
- Transform：Duplex可以在写入和读取数据时修改或转换数据的流（例如 `zlib.createDeflate()`）。

### Readable

之前我们读取一个文件：

```js
fs.readFile('./foo.txt', (err, data) => {
  console.log(data)
})
```

这种方式是一次性将一个文件中所有的内容都读取到程序（内存）中，但是这种读取方式就会出现我们之前提到的很多问题：

- 文件过大、读取的位置、结束的位置、一次读取的大小；
- 这个时候，我们可以使用 `createReadStream`，我们来看几个参数，更多参数可以参考官网：
  - start：文件读取开始的位置；
  - end：文件读取结束的位置；
  - highWaterMark：一次性读取字节的长度，默认是64kb；

### Readable的使用

```js
// 流的方式读取
const reader = fs.createReadStream("./foo.txt", {
  start: 3,
  end: 10,
  highWaterMark: 2,
});

// 数据读取的过程
reader.on("data", (data) => {
  console.log(data);

  // 暂停读取
  reader.pause();

  setTimeout(() => {
    // 恢复读取
    reader.resume();
  }, 1000);
});

// 文件被打开事件监听
reader.on("open", () => {
  console.log("文件被打开");
});

// 文件被关闭事件监听
reader.on('close', () => {
  console.log("文件被关闭了");
})
```

### Writable

之前我们写入一个文件的方式是这样的：

```js
fs.writeFile('./foo.txt', (err) => {

})
```

- 这种方式相当于一次性将所有的内容写入到文件中，但是这种方式也有很多问题：
  - 比如我们希望一点点写入内容，精确每次写入的位置等；
- 这个时候，我们可以使用 `createWriteStream`，我们来看几个参数，更多参数可以参考官网：
  - flags：默认是 w，如果我们希望是追加写入，可以使用 a 或者 a+；
  - start：写入的位置；

### Writable的使用

```js
// Stream的写入方式
const writer = fs.createWriteStream('./bar.txt', {
  flags: "r+",
  start: 4
});

writer.write("你好啊", (err) => {
  if (err) {
    console.log(err);
    return;
  }
  console.log("写入成功");
});

writer.write("李银河", (err) => {
  console.log("第二次写入");
})

// 我们会发现，我们并不能监听到 close 事件：
// 这是因为写入流在打开后是不会自动关闭的；
// 我们必须手动关闭，来告诉Node已经写入结束了；
// 并且会发出一个 finish 事件的；

writer.on('finish', () => {
  console.log("文件写入结束");
})

writer.on('close', () => {
  console.log("文件被关闭");
})

writer.close()


// end方法相当于做了两步操作： write传入的数据和调用close方法；
writer.end("Hello World");
```

### pipe方法

pipe函数实际就是对可读流的一种封装

```js
// 传统的写法
fs.readFile('./foo.txt', (err, data) => {
  fs.writeFile('./foz.txt', data, (err) => {
    console.log(err);
  })
})

const reader = fs.createReadStream("./foo.txt");
const writer = fs.createWriteStream('./foz.txt');

reader.pipe(writer);
```

## node中的http

### 创建服务器

有两种方式：

```js
const server1 = http.createServer((req, res) => {
  res.end("Server1");
});

const server2 = new http.Server((req, res) => {
  res.end("Server2");
});
```

> - req：request请求对象，包含请求相关的信息
> - res：response响应对象，包含我们要发送给客户端的信息

### 监听主机和端口号

Server通过listen方法来开启服务器，并且在某一个主机和端口上监听网络请求：

- 也就是当我们通过 ip:port的方式发送到我们监听的Web服务器上时；
- 我们就可以对其进行相关的处理；
  - listen函数有三个参数：
- 端口port: 可以不传, 系统会默认分配端, 后续项目中我们会写入到环境变量中；
- 主机host: 通常可以传入localhost、ip地址127.0.0.1、或者ip地址0.0.0.0，默认是0.0.0.0；
  - localhost：本质上是一个域名，通常情况下会被解析成127.0.0.1；
  - 127.0.0.1：回环地址（Loop Back Address），表达的意思其实是我们主机自己发出去的包，直接被自己接收；
    - 正常的数据库包经常 应用层 - 传输层 - 网络层 - 数据链路层 - 物理层 ；
    - 而回环地址，是在网络层直接就被获取到了，是不会经常数据链路层和物理层的；
    - 比如我们监听 127.0.0.1时，在同一个网段下的主机中，通过ip地址是不能访问的；
  - 0.0.0.0：
    - 监听IPV4上所有的地址，再根据端口找到不同的应用程序；
    - 比如我们监听 0.0.0.0时，在同一个网段下的主机中，通过ip地址是可以访问的；
- 回调函数：服务器启动成功时的回调函数；

```js
server1.listen(8000, () => {
  console.log("server1启动成功~");
});

// 启动服务器,并且制定端口号和主机
server2.listen(8888, '0.0.0.0', () => {
  console.log("server2启动成功~");
});
```

### request对象

request对象包含很多信息，如：url、请求方式、headers等

```js
const server = http.createServer((req, res) => {
  // request对象中封装了客户端给我们服务器传递过来的所有信息
  console.log(req.url);
  console.log(req.method);
  console.log(req.headers);

  res.end("Hello Server");
});
```

### URL的解析

如果用户发送的地址中还携带一些额外的参数，如：<http://localhost:8000/login?name=why&password=123>;

可以使用内置模块url、以及qs模块
> 最新node版本已经被弃用

```js
const { pathname, query } = url.parse(req.url);
if (pathname === '/login') {
  console.log(query);
  console.log(qs.parse(query));
  const { username, password } = qs.parse(query);
  console.log(username, password);
  res.end("请求结果~");
}
```

### headers属性

在request对象的header中也包含很多有用的信息，客户端会默认传递过来一些信息

- content-type 是这次请求携带的数据的类型：
  - application/json 表示是一个json类型；
  - text/plain 表示是文本类型；
  - application/xml 表示是xml类型；
  - multipart/form-data 表示是上传文件；
- content-length：文件的大小和长度
- keep-alive：
  - http是基于TCP协议的，但是通常在进行一次请求和响应结束后会立刻中断；
  - 在http1.0中，如果想要继续保持连接：
    - 浏览器需要在请求头中添加 connection: keep-alive；
    - 服务器需要在响应头中添加 connection:keey-alive；
    - 当客户端再次放请求时，就会使用同一个连接，直接一方中断连接；
  - 在http1.1中，所有连接默认是 connection: keep-alive的；
    - 不同的Web服务器会有不同的保持 keep-alive的时间；
    - Node中默认是5s中；
- accept-encoding：告知服务器，客户端支持的文件压缩格式，比如js文件可以使用gzip编码，对应 .gz文件；
- accept：告知服务器，客户端可接受文件的格式类型；
- user-agent：客户端相关的信息；

### 返回响应结果

如果我们希望给客户端响应的结果数据，可以通过两种方式：

- write方法：这种方式是直接写出数据，但是并没有关闭流
- end方法：这种方式是写出最后的数据，并且写出后会关闭流

```js
// 响应结果
res.write("响应结果一");
res.end("Hello World");
```

> 如果我们没有调用 end 和 close，客户端将会一直等待结果，所以客户端在发送网络请求时，都会设置超时时间

### 返回状态码

设置状态码常见有两种方式：

```js
res.statusCode = 400;
res.writeHead(200);
```

### 返回响应头

返回头部信息，主要有两种方式：

- res.setHeader：一次写入一个头部信息；
- res.writeHead：同时写入header和status

```js
res.setHeader("Content-Type", "text/plain;charset=utf8");
res.writeHead(200, {
  "Content-Type": "text/html;charset=utf8"
});
```

- Header设置 Content-Type有什么作用呢？

  - 默认客户端接收到的是字符串，客户端会按照自己默认的方式进行处理；

### http请求

```js
// http发送post请求
const req = http.request({
  method: 'POST',
  hostname: 'localhost',
  port: 8888
}, (res) => {
  res.on('data', (data) => {
    console.log(data.toString());
  });

  res.on('end', () => {
    console.log("获取到了所有的结果");
  })
});
```

## 文件上传

### 错误方式

下面这种上传方式是错误的，其原因在于我们写入的字节流中包含多余的数据，会导致写入的文件无法正确解码与读取

```js
const server = http.createServer((req, res) => {
  if (req.url === '/upload') {
    if (req.method === 'POST') {
      const fileWriter = fs.createWriteStream('./foo.png', {flags: 'a+'});

      req.on('data', (data) => {
        console.log(data);
        fileWriter.write(data);
      });

      req.on('end', () => {
        console.log("文件上传成功~");
        res.end("文件上传成功~");
      })
    }
  }
});
```

### 正确方式

```js
const server = http.createServer((req, res) => {
  if (req.url === '/upload') {
    if (req.method === 'POST') {
      req.setEncoding('binary');

      let body = '';
      const totalBoundary = req.headers['content-type'].split(';')[1];
      const boundary = totalBoundary.split('=')[1];

      req.on('data', (data) => {
        body += data;
      });

      req.on('end', () => {
        console.log(body);
        // 处理body
        // 1.获取image/png的位置
        const payload = qs.parse(body, "\r\n", ": ");
        const type = payload["Content-Type"];

        // 2.开始在image/png的位置进行截取
        const typeIndex = body.indexOf(type);
        const typeLength = type.length;
        let imageData = body.substring(typeIndex + typeLength);

        // 3.将中间的两个空格去掉
        imageData = imageData.replace(/^\s\s*/, '');

        // 4.将最后的boundary去掉
        imageData = imageData.substring(0, imageData.indexOf(`--${boundary}--`));

        fs.writeFile('./foo.png', imageData, 'binary', (err) => {
          res.end("文件上传成功~");
        })
      })
    }
  }
});
```

### express中的文件上传

使用express官方的multer库

```js
const path = require('path');

const express = require('express');
const multer = require('multer');

const app = express();

app.use(express.json());
app.use(express.urlencoded({ extended: true }));

const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, './uploads/');
  },
  filename: (req, file, cb) => {
    cb(null, Date.now() + path.extname(file.originalname));
  }
})

const upload = multer({
  storage
});


app.post('/upload', upload.array('file'), (req, res, next) => {
  console.log(req.files);
  res.end("文件上传成功~");
});

app.listen(8000, () => {
  console.log("form-data解析服务器启动成功~")
});
```

### koa中的文件上传

```js
const Koa = require('koa');
const Router = require('koa-router');
const multer = require('koa-multer');

const app = new Koa();
const uploadRouter = new Router({prefix: '/upload'});

const upload = multer({
  dest: './uploads/'
});

uploadRouter.post('/avatar', upload.single('avatar'), (ctx, next) => {
  console.log(ctx.req.file);
  ctx.response.body = "上传头像成功~";
});

app.use(uploadRouter.routes());

app.listen(8000, () => {
  console.log("koa初体验服务器启动成功~");
});
```
