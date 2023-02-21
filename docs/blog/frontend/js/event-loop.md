# 事件循环

## 首先要知道的

- JavaScript 是单线程语言
- Event loop(事件循环)是 JavaScript 的执行机制
- 异步和多线程的实现是通过事件循环机制来实现的，大体有三个部分组成：调用栈，消息队列(message queue),微任务队列(microtask queue)

## JavaScript 事件循环

js 是单线程，也就意味着 js 的任务需要一个一个顺序执行。如果一个任务耗时过长，那么后一个任务也必须等着。那么问题来了，假如我们想浏览新闻，但是新闻包含的超清图片加载很慢，难道我们的网页要一直卡着直到图片完全显示出来？因此聪明的程序员将任务分为两类：

- 同步任务
- 异步任务

举个例子，当我们打开网站时，网页的渲染过程就是一大堆同步任务，比如页面骨架和页面元素的渲染。而像加载图片音乐之类占用资源大耗时久的任务，就是异步任务。

## 什么是事件循环

![](https://user-gold-cdn.xitu.io/2018/7/14/164974fb89da87c5?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

![](https://z3.ax1x.com/2021/05/29/2A7T10.gif)
分析：

- 同步和异步任务分别进入不同的执行"场所"，同步的进入主线程，异步的进入 Event Table 并注册函数
- 当指定的事情完成时，Event Table 会将这个函数移入 Event Queue。
- 主线程内的任务执行完毕为空，会去 Event Queue 读取对应的函数，进入主线程执行。
- 上述过程会不断重复，也就是常说的 Event Loop(事件循环)。

## 什么是微任务、宏任务

**微任务和宏任务皆为异步任务，它们都属于一个队列，主要区别在于他们的执行顺序，Event Loop 的走向和取值。**

![](https://user-gold-cdn.xitu.io/2018/7/14/164974fa4b42e4af?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

对于浏览器来说:

- 宏任务:整块 script、setTimeout、setInterval、ajax、DOM 监听、UI Rendering 等
- 微任务:Promise、async 中 await 后面的任务、MutationObserver 等

对于 Node 来说：

- 宏任务：setTimeout、setInterval、IO 事件、setImmediate、close 事件
- 微任务：Promise 的 then 回调、process.nextTick、queueMicrotask

原则：每次执行宏任务时，都需要清空微任务队列中的微任务。

## 一些例子

### setTimeout

```js
setTimeout(() => {
  console.log("3s");
}, 3000);
console.log(1);
```

上面这个例子中，会先输出 1 再输出 3s 因为 setTimeout 是异步任务，console.log()为同步任务<br>

我们再来修改一下代码

```js
setTimeout(() => {
  console.log("3s");
}, 3000);
sleep(1000000);
```

我们发现输出 3s 的时间远远超过 3 秒，这是为什么？首先我们来看看这段代码是如何执行的:

- `console.log("3s")`进入 Event Table 并注册,计时开始。
- 执行`sleep()`，非常慢，计时一直继续。
- 3 秒到了，计时事件 setTimeout 完成，`console.log("3s")`进入 Event Queue，但是 sleep 也太慢了吧，还没执行完，只好等着。
- sleep 终于执行完了，`console.log("3s")`终于从 Event Queue 进入了主线程执行。

**结论**：setTimeout 这个函数，是经过指定时间后，把要执行的任务(本例中为`console.log("3s")`)加入到 Event Queue 中，又因为是单线程任务要一个一个执行，如果前面的任务需要的时间太久，那么只能等着，导致真正的延迟时间远远大于 3 秒。

除此之外，我们还能看到 setTimeout(fn,0)的这种情况，是不是会立即执行呢？答案是不会的，setTimeout(fn,0)的含义是：setTimeout 里面的任务在主线程最早得到空闲的时候执行，只要主线程执行栈内的同步任务全部执行完成，栈为空就马上执行

```js
setTimeout(() => {
  console.log("1");
}, 0);
console.log("2"); // 输出结果为2 1
```

另外，关于 setTimeout 要补充的是，即便主线程为空，0 毫秒实际上也是达不到的。根据 HTML 的标准，最低是 4 毫秒。

## setInterval

它是 setTimeout 的循环的执行。对于执行顺序来说，setInterval 会每隔指定的时间将注册的函数置入 Event Queue，如果前面的任务耗时太久，那么同样需要等待。
需要注意的是，对于 setInterval(fn,ms)来说，**我们已经知道不是每过 ms 秒会执行一次 fn，而是每过 ms 秒，会有 fn 进入 Event Queue。** 也就是说，一旦 setInterval 的回调函数 fn 执行时间超过了延迟时间 ms，那么就完全看不出来有时间间隔了。

## Promise

直接看例子

```js
setTimeout(() => {
  console.log("setTimeout1");
}, 0);
let p = new Promise((resolve, reject) => {
  console.log("Promise1");
  resolve();
});
p.then(() => {
  console.log("Promise2");
});
```

上面提到过 Promise 中的参数 Promise1 是同步执行的，此外因为 Promise 是微任务，会在同步任务执行后去清空微任务，清空完微任务再去宏任务队列里面取值，因此最后输出结果是`Promise1 Promise2 setTimeout1`

```js
Promise.resolve().then(() => {
  console.log("Promise1");
  setTimeout(() => {
    console.log("setTimeout2");
  }, 0);
});

setTimeout(() => {
  console.log("setTimeout1");
  Promise.resolve().then(() => {
    console.log("Promise2");
  });
}, 0);
```

最后输出结果是`Promise1 setTimeout1 Promise2 setTimeout2`

分析：

- 一开始执行栈的同步任务执行完毕，没有输出，然后会去微任务队列中找并且清空微任务队列，输出 Promise1，同时生成异步任务 setTimeout1
- 查看宏任务队列，此时队列 setTimeout1 在 setTimeout2 之前，执行 setTimeout1
- 执行 setTimeout1 时，输出 setTimeout1，同时生成 Promise2 的一个微任务，放入微任务队列中，清空微任务队列，输出 Promise2
- 再去宏任务队列中取一个宏任务，此时输出 setTimeout2

## 自查

```js
console.log("1");

setTimeout(function () {
  console.log("2");
  process.nextTick(function () {
    console.log("3");
  });
  new Promise(function (resolve) {
    console.log("4");
    resolve();
  }).then(function () {
    console.log("5");
  });
});
process.nextTick(function () {
  console.log("6");
});
new Promise(function (resolve) {
  console.log("7");
  resolve();
}).then(function () {
  console.log("8");
});

setTimeout(function () {
  console.log("9");
  process.nextTick(function () {
    console.log("10");
  });
  new Promise(function (resolve) {
    console.log("11");
    resolve();
  }).then(function () {
    console.log("12");
  });
});
```

分析:

首先执行同步任务，输出`1 7`

**第一轮事件循环**

- 将`setTimeout1 setTimeout2`放入宏任务队列中
- 将`process1 then1`放入微任务队列
- 清空微任务队列，输出`6 8`
- 从宏任务队列中取出`setTimeout1`任务

**第二轮事件循环**

- 执行`setTimeout1`中的同步任务，输出`2 4`
- 将`process2 then2`放入微任务队列
- 清空微任务队列，输出`3 5`
- 从宏任务队列中取出`setTimeout2`任务

**第三轮事件循环**

- 执行`setTimeout2`中的同步任务，输出`9 11`
- 清空微任务队列，输出`10 12`
- 结束

因此输出结果为`1 7 6 8 2 4 3 5 9 11 10 12`
