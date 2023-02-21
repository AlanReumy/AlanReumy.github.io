# 节流

## 什么是节流

节流的核心是：触发事件，执行任务并设置时间间隔，如果时间间隔内有触发行为就取消任务，如果时间间隔后有触发行为，就再次执行任务并设置时间间隔

## 定时器版本

在持续触发事件的过程中，函数不会立即执行，并且会等待delay秒执行一次

```js
function throttle(fn,delay){
  let timeout;
  return function(){
    let context = this;
    let args = arguments;
    if(!timeout){
      timeout = setTimeout(function(){
        timeout = null;
        fn.apply(context,args);
      },delay)
    }
  }
}
```

## 时间戳版本

在持续触发事件的过程中，函数会立即执行，并且会每delay秒执行一次

```js
function throttle(fn,delay){
  let pre = 0;
  return function(){
    let context = this;
    let args = arguments;
    let now = +new Date();
    if(now - pre > delay){
      fn.apply(context,args);
      pre = now;
    }
  }
}
```

## 防抖与节流的区别

如果事件触发频繁，防抖中的计时会不断重置，从而会不断延迟了函数的执行，而节流中的计时器不会因为事件的触发而重置，仅仅是取消事件的触发
