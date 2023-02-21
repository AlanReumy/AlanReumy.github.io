# 防抖

## 什么是函数防抖(debounce)

>在事件被触发n秒后再执行回调，如果在这n秒内又被触发，则重新计时。

举个例子，一个按钮点击后将会发送网络请求，但是如果用户连续5次，会发送5次网络请求吗？答案是不会的，因为在每次请求后，都会等几秒再执行回调，如果再次被点击，将会重新计时，这就是函数防抖

## 使用场景

* 按钮提交场景：防止多次提交按钮，只执行最后提交的一次
* 搜索框联想场景：防止联想发送请求，只发送最后一次输入

## 实现函数防抖

现在我们有一个按钮，在点击后它将输出```hello```

```js
const btn = document.querySelector("button");
function say() {
  console.log("hello");
}
btn.addEventListener("click", say);
```

接下来我们来一步步实现函数防抖

### 创建防抖函数

```js
const btn = document.querySelector("button");
function say() {
  console.log("hello");
}
// 防抖函数
function debounce(fn){
  fn();
}
btn.addEventListener("click", debounce(say));
```

如果你实现这段代码的话你会发现，在没有点击的时候，```say()```函数就已经执行了，这是因为定义监听函数的时候就已经执行了函数，因此我们需要使用高阶函数，也就是在函数里面返回函数，**这也是防抖的第一个难点**

```js
const btn = document.querySelector("button");
function say() {
  console.log("hello");
}
// 防抖函数
function debounce(fn){
  return function(){
    fn();
  }
}
btn.addEventListener("click", debounce(say));
```

### 添加延时

```js
const btn = document.querySelector("button");
function say() {
  console.log("hello");
}
// 防抖函数
function debounce(fn,delay){
  return function(){
    setTimeout(()=>{
      fn();
    },delay)
  }
}
btn.addEventListener("click", debounce(say,1000));
```

在上述代码中，我们加入了定时器，并且可以让使用者自定义延时时间，接着我们需要实现在每次点击前清除延时的功能

### 清除延时

```js
const btn = document.querySelector("button");
function say() {
  console.log("hello");
}
// 防抖函数
function debounce(fn,delay){
  return function(){
    // 未声明就使用，报错
    clearTimeout(timer);
    let timer = setTimeout(()=>{
      fn();
    },delay)
  }
}
btn.addEventListener("click", debounce(say,1000));
```

很明显上述代码存在问题，```timer```在未声明就使用，因此我们需要在前面提前声明```timer```变量

```js
function debounce(fn,delay){
  return function(){
    let timer;
    clearTimeout(timer);
    let timer = setTimeout(()=>{
      fn();
    },delay)
  }
}
btn.addEventListener("click", debounce(say,1000));
```

实现上述代码后，我们执行程序的话可以发现并未达到我们想要的效果，这是因为每次执行的函数内部都是独立的```timer```变量，因此我们需要通过闭包延长作用域链，**这是防抖的第二个难点**

```js
function debounce(fn,delay){
  let timer;
  return function(){
    clearTimeout(timer);
    let timer = setTimeout(()=>{
      fn();
    },delay)
  }
}
btn.addEventListener("click", debounce(say,1000));
```

执行程序后发现，我们已经实现了防抖功能了，但是还没完，让我们回到```say()```方法里输出一下```this```

```js
function say() {
  console.log("hello");
  console.log(this);
}

function debounce(fn,delay){
  let timer;
  return function(){
    clearTimeout(timer);
    let timer = setTimeout(()=>{
      fn();
    },delay)
  }
}
//btn.addEventListener("click",say); // this指向button
btn.addEventListener("click", debounce(say,1000)); // this指向Window
```

很明显，在按钮绑定say方法的时候，使用隐式绑定，```this```指向```button```，而在按钮绑定防抖函数的时候，出现了回调函数中隐式绑定丢失的问题，因此```this```指向```Window```。我们可以通过使用```apply、call、bind```方法实现**硬绑定**来解决这个问题

```js
function debounce(fn,delay){
  let timer;
  return function(){
    let context = this;
    clearTimeout(timer);
    let timer = setTimeout(()=>{
      fn.apply(context)
    },delay)
  }
}
btn.addEventListener("click", debounce(say,1000)); // this指向button
```

**防抖函数就正式完成了**
