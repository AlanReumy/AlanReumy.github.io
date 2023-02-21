---
title: React中的setState
---

## 为什么使用setState

开发中我们并不能直接通过修改state的值来让界面发生更新：

- 因为我们修改了state之后，希望React根据最新的State来重新渲染界面，但是这种方式的修改React并不知道数据发生了改变;
- React并没有实现类似于Vue2中的Object.defineProperty或者Vue3中的Proxy的方式来监听数据的变化;
- 我们必须通过setState来告知React数据已经发生了变化;

## setState异步更新

```js
changeText() {
  this.setState({
    message: "Hello world"
  })
  console.log(this.state.message)
}
```

- 最终打印结果是Hello World
- 可见setState是异步的操作,我们并不能在执行完setState之后立马拿到最新的state的结果

### 为什么setState设计为异步呢?

- setState设计为异步,可以显著的提升性能;
  - 如果每次调用 setState都进行一次更新,那么意味着render函数会被频繁调用,界面重新渲染,这样效率是很低的;
  - 最好的办法应该是获取到多个更新,之后进行批量更新;
- 如果同步更新了state,但是还没有执行render函数,那么state和props不能保持同步;
  - state和props不能保持一致性,会在开发中产生很多的问题;

### 如何获取异步的结果?

- 方式一：setState的回调
  - setState接受两个参数:第二个参数是一个回调函数,这个回调函数会在更新后会执行;

  ```js
  changeText() {
    this.setState({
      message: "Hello world"
    }, () => {
      console.log(this.state.message)
    }) 
  }
  ```

### setState不一定是异步的

- 在setTimeout中的更新
- 原生dom事件

也就意味着：

1. 在组件生命周期或React合成事件中，setState是异步的
1. 在setTimeout或原生dom事件中，setState是同步的
