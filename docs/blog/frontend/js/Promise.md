# Promise

在研究一个 `Promise` 之前，我们先来看看没有 `Promise` 前，存在哪些技术痛点。

## 没有Promise前遇到的问题

### 网络请求使用的问题

在没有出现 `Promise` 前，如果我们需要发送网络请求，需要封装并且调用这样的函数：

```js
function request(url, successCallback, failureCallback) {
  // 模拟网络请求
  setTimeout(() => {
    if ((url = 'curry')) {
      // 发送成功了
      successCallback()
    } else {
      // 发送失败了
      failureCallback()
    }
  }, 2000)
}
```

虽然在上面的这种解决方案中，我们确实可以获得请求后函数的结果，但是它仍然存在几个问题：

1. 我们需要自己来设计回调函数、回调函数名称、回调函数使用。
2. 对于不同的库或框架，开发出来的设计方案可能是不同的，我们只能通过看文档或者阅读文档等方式，去了解这个函数是怎么用的，这样的成本太大了。

**`Promise` 其实就是一种类似于规范的东西，规范了这种解决方案。**

### 回调地狱的问题

在发送 `ajax` 请求时，容易出现下面这样的，回调地狱的问题：

```js
$.ajax("url1", function (data1) {
        $.ajax(data1["url2"], function (data2) {
          $.ajax(data2["url3"], function (data3) {
            $.ajax(data3["url3"], function (data4) {
              console.log(data4);
            });
          });
        });
      });
```

对于上面这种情况，正常时不会有什么问题的，但是这样的代码难看且不易维护，我们更加期望的是一种更加优雅的方式来进行异步操作。

**`Promise` 可以以一种非常优雅的方式来解决这个问题。**

## Promise是什么

`Promise` 是一个类，当我们需要给予调用者一个承诺，待会我会给你回调数据时，就可以创建一个Promise的对象。

在通过 `new` 关键字创建 `Promise` 对象时，我们需要传入一个回调函数，称之为 `executor` 。

对于这个 `executor` 函数来说：

* 它会立即执行，并且会接受两个参数，这两个参数是回调函数，分别称为 `resolve` 与 `reject` 。
* 当我们调用 `resolve` 回调函数时，会执行 `Promise` 实例对象的 `then` 方法中传入的回调函数。
* 当我们调用 `reject` 回调函数时，会执行 `Promise` 实例对象的 `catch` 方法传入的回调函数。

```js
const p = new Promise((resolve, reject) => {
  // resolve('fulfilled status')
  // reject('rejected status')
})

p.then(res => {
  console.log('res: ', res)
}).catch(err => {
  console.log('err: ', err)
})

```

## Promise 的状态

`Promise` 实例对象存在三个状态：

* `pending` ： 初始状态，既没有被兑现，也没有被拒绝。（当执行 `executor` 中的代码时，处于该状态）
* `fulfilled` ：  意味着操作成功完成。（执行了 `resolve` 回调函数，处于该状态）
* `rejected` ： 意味着操作失败。（执行了 `reject` 回调函数或抛出异常，处于该状态）

> 一旦状态被确定下来，`Promise` 的状态会被**锁死**，该 `Promise` 的状态是不可更改的

## 使用Promise对请求的重构

有了 `Promise` 我们可以对之前网络请求的函数进行重构了。

```js
function request(url){
  return new Promise((resolve,reject) =>{
    setTimeout(()=>{
      if ((url = 'curry')) {
        // 发送成功了
        resolve()
      } else {
        // 发送失败了
        reject()
      }
    },2000)
  })
}
```

## resolve

我们来说明一下 `resolve` 中传入的不同参数的区别：

1. 如果传入的参数为 `Promise` 对象，则该 `Promise` 对象的结果决定了 `resolve` 的结果。

```js
new Promise((resolve, reject) => {
  resolve(new Promise((resolve, reject) => {
    resolve('fulfilled status')
  }))
}).then(res => {
  console.log('res: ', res)
})

```

2. 如果传入的参数为 非`Promise` 类型的对象 则返回的结果为成功的 `promise` 对象。

```js
new Promise((resolve, reject) => {
  resolve('fulfilled status')
}).then(res => {
  console.log('res: ', res)
})
```

3. 如果传入的参数为一个对象，并且这个对象有实现 `then` 方法（可以称为 `thenable` ），那么会执行该 `then` 方法，并且根据 `then` 方法的结果来决定 `Promise` 的状态。

```js
new Promise((resolve, reject) => {
  resolve({
    then:function(resolve,reject){
      resolve("fulfilled status")
    }
  })
}).then(res => {
  console.log('res: ', res)
})
```

> 使用 `Promise` 静态方法中的 `resolve` 传参的使用相同

## reject

我们来说明一下 `reject` 中传入的不同参数的区别：

对于 `reject` 来说，无论传入什么值，返回的结果都是失败的 `Promise` 对象

```js
const p = Promise.reject(new Promise(() => {}))

p.then(res => {
  console.log("res:", res)
}).catch(err => {
  console.log("err:", err) // err: Promise { <pending> }
})
```

## then

### then接收的参数

`then` 方法可以接受两个参数，用来处理成功的回调和失败的回调。

```js
const p = new Promise((resovle, reject) => {
  // resovle()
  // reject()
})

p.then(
  res => {},
  err => {}
)

// 等价于
p.then(res => {

}).catch(err => {

})
```

### then可以多次被调用

一个 `Promise` 实例对象可以多次调用 `then` 方法，当我们的 `resolve` 方法被回调时, 所有的 `then` 方法传入的回调函数都会被调用：

```js
promise.then(res => {
  console.log("res1:", res)
})

promise.then(res => {
  console.log("res2:", res)
})

promise.then(res => {
  console.log("res3:", res)
})
```

### then的返回值

`then` 方法返回一个 `Promise` 对象，所以我们可以进行链式调用：

```js
p.then(res => {

}).then(res => {

}).then(res => {

}).catch(err => {
  
})
```

那么返回的 `Promise` 对象处于什么状态呢？有以下几种情况：

1. 当返回值为 `fulfilled` 状态的 `Promise` 、`fulfilled` 状态的 `thenable` 、普通的值时，它会处于 `fulfilled` 状态。

2. 当返回值为 `rejected` 状态的 `Promise` 、 `then` 方法抛出异常、`rejected` 状态的 `thenable`，它会处于 `rejected` 状态。

## catch

### catch可以多次被调用

一个 `Promise` 实例对象的 `catch` 方法和 `then` 方法一样，也是可以多次调用的，每次调用我们都可以传入 `reject` 的回调函数，当 `Promise` 实例对象状态变为 `rejected` 的时候，这些函数都会被调用。

```js
promise.catch(err => {
  console.log("err1:", err)
})

promise.catch(err => {
  console.log("err2:", err)
})

promise.catch(err => {
  console.log("err3:", err)
})
```

### catch的返回值

`catch` 方法和 `then` 方法同样，也会返回一个 `Promise` 对象，返回值的状态判断也和 `then` 方法一样。

如果我们需要后续继续执行 `catch` 方法中的回调函数，就需要返回一个 `rejected` 状态的 `Promise` 对象。

## finally

`finally` 方法表示：无论 `Promise` 对象无论变成 `fulfilled` 还是 `rejected` 状态，最终都会被执行的代码。

它不接收参数，只要之前的 `Promise` 对象处于非 `pending` 状态，那么它都会执行。

```js
const p = new Promise((resolve,reject) => {
  resolve('fulfilled status')
})

p.then(res => {
  return new Promise((resolve,reject) => {
    reject('rejected status')
  })
},err => {

}).then(res => {

},err => {
  console.log("err: ",err)
}).finally(()=>{
  console.log("finally")
})
```

## all

`all` 方法接收一个包含 `Promise` 的数组。只要有一个 `Promise`对象的结果为 `rejected` 状态，那么返回的结果就为 `rejected` 状态的 `Promise`，否则它返回的结果是 `fulfilled` 状态的 `Promise` 对象 。

```js
let p1 = new Promise((resolve, reject) => {
  resolve("OK")
})
let p2 = Promise.resolve("success")
let p3 = Promise.resolve("oh yeah")
const res = Promise.all([p1, p2, p3])
console.log(res)
```

## allSettled

`allSettled` 方法接收一个包含 `Promise` 的数组，等待所有 `Promise` 对象的状态确定，再根据顺序返回这个包含 `Promise` 对象的数组。

```js
const p1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve(11111)
  }, 1000);
})

const p2 = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject(22222)
  }, 2000);
})

const p3 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve(33333)
  }, 3000);
})

Promise.allSettled([p1, p2, p3]).then(res => {
  console.log(res)
}).catch(err => {
  console.log(err)
})
```

## race

`race` 方法接收一个包含 `Promise` 的数组，返回一个新的 `Promise` 对象。其中，第一个确定状态的 `Promise` 对象的结果状态就是最终的结果状态

```js
let p1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("OK")
  }, 1000);
})
let p2 = Promise.resolve("success")
const res = Promise.race([p1, p2, p3])
console.log(res)
```

## 改变状态与指定回调的顺序执行问题

1. 当执行器里面的任务为同步任务时，先该变 `Promise` 对应的状态 再指定回调。
2. 当执行器里面的任务为异步任务时，先指定回调 再改变状态。
3. 指定不是执行。

```js
 let p = new Promise((resolve, reject) => {
   setTimeout(() => {
     resolve("ok")
   }, 1000);
 })
 p.then(value => {
   console.log(value)
 }, reason => {
   console.log(reason)
 })
```

`then` 是微任务，定时器是宏任务。所以宏任务执行完后先去执行 `then`，也就是指定了`promise`改变状态时候的回调函数，然后再执行定时器，执行了`resolve("ok")`，触发`then`里面的回调。

## Promise如何串联多个任务

由于 `Promise` 中的 `then` 会返回 `Promise` 对象，因此使用链式调用将多个任务串联起来。

```js
 let p = new Promise((resolve, reject) => {
   setTimeout(() => {
     resolve("ok")
   }, 1000);
 })

 p.then(value => {
   return new Promise((resolve, reject) => {
     resolve("success")
   })
 }).then(value => {
   console.log(value)
 }).then(value => {
   console.log(value)
 })
```

## Promise中的异常传透

当使用 `Promise` 的 `then` 链式调用时，可以在最后指定失败的回调前面任何操作出了异常，都会传到最后失败的回调中处理。

```js
 let p = new Promise((resolve, reject) => {
   setTimeout(() => {
     resolve("ok")
     // reject("error")
   }, 1000);
 })
 p.then(value => {
   // console.log(111)
   throw "error"
 }).then(value => {
   console.log(222)
 }).then(value => {
   console.log(333)
 }).catch(reason => {
   console.log(reason)
 })
```

## 中断promise链

有且只有一种方式，返回一个 `padding` 状态的 `Promise`

```js
 p.then(value => {
   console.log(111)
   return new Promise(() => { })
 }).then(value => {
   console.log(222)
 }).then(value => {
   console.log(333)
 }).catch(reason => {
   console.log(reason)
 })
```

## aysnc与await

### async函数

1. 函数的返回值是一个promise对象
2. promise对象的结果由async函数执行的返回值决定

```js
 const fn = async function () {
   return new Promise((resolve, reject) => {
     resolve()
   })
 }
 console.log(fn())
```

### await表达式

1. await右侧的表达式一般为promise对象，但也可以是其它的值
2. 如果表达式是promise对象，await返回的是promise成功的值
3. 如果await表达式返回的promise是失败的状态，则需要使用```try catch```捕获失败
4. 如果表达式是其它值，直接将此值作为await的返回值

```js
 const fn = async () => {
   try {
     let n = await 123
     console.log(n)//123
     let p = await new Promise((resolve, reject) => {
       // resolve("ok")
       reject("error")
     })
   } catch (error) {
     console.log(error)
   }
 }
 fn()
```

## 手写Promise

```js
// 声明构造函数
function Promise(executor) {
  // 状态
  this.PromiseState = "pending"
  // 结果值
  this.PromiseResult = null
  // 保存this值
  const self = this
  this.callbacks = []
  // resolve函数
  function resolve(data) {
    // promise的状态只能被修改一次
    if (self.PromiseState !== "pending")
      return
    self.PromiseState = "fulfilled"
    self.PromiseResult = data
    // 调用成功的回调
    setTimeout(() => {
      self.callbacks.forEach(item => {
        item.onResolved(data)
      })
    });
  }

  // reject函数
  function reject(data) {
    // promise的状态只能被修改一次
    if (self.PromiseState !== "pending")
      return
    self.PromiseState = "rejected"
    self.PromiseResult = data
    // 调用成功的回调
    setTimeout(() => {
      self.callbacks.forEach(item => {
        item.onRejected(data)
      })
    })
  }

  try {
    // 同步调用执行器函数
    executor(resolve, reject)
  } catch (error) {
    // promise的状态只能被修改一次
    if (this.PromiseState !== "pending")
      return
    this.PromiseState = "rejected"
    this.PromiseResult = error
  }
}

// 添加then方法
Promise.prototype.then = function (onResolved, onRejected) {
  const self = this
  // 判断回调函数的参数
  // 保证异常传透
  if (typeof onRejected !== "function") {
    onRejected = reason => {
      throw reason
    }
  }
  // 保证值传递
  if (typeof onResolved !== "function") {
    onResolved = value => value
  }
  return new Promise((resolve, reject) => {
    // 封装函数
    function callback(type) {
      try {
        let result = type(self.PromiseResult)
        if (result instanceof Promise) {
          result.then(v => {
            resolve(v)
          }, r => {
            reject(r)
          })
        } else {
          resolve(result)
        }
      } catch (error) {
        reject(error)
      }
    }

    // promise实例对象状态为fulfilled
    if (this.PromiseState === "fulfilled") {
      setTimeout(() => {
        callback(onResolved)
      });
    }
    // promise实例对象状态为rejected
    if (this.PromiseState === "rejected") {
      setTimeout(() => {
        callback(onRejected)
      });
    }
    // promise实例对象状态为pending
    if (this.PromiseState === "pending") {
      // 保存then中传入的回调函数
      this.callbacks.push({
        onResolved: function () {
          callback(onResolved)
        },
        onRejected: function () {
          callback(onRejected)
        }
      })
    }
  }
  )
}

// 添加catch方法
Promise.prototype.catch = function (onRejected) {
  return this.then(undefined, onRejected)
}

// 添加Promise.resolve方法
Promise.resolve = function (value) {
  return new Promise((resolve, reject) => {
    if (value instanceof Promise) {
      value.then(v => {
        resolve(v)
      }, r => {
        reject(r)
      })
    } else {
      resolve(value)
    }
  })
}

// 添加Promise.reject方法
Promise.reject = function (reason) {
  return new Promise((resolve, reject) => {
    reject(reason)
  })
}

// 添加Promise.all方法
Promise.all = function (promises) {
  return new Promise((resolve, reject) => {
    let count = 0
    let arr = []
    promises.forEach((item, index) => {
      item.then(v => {
        count++
        arr[index] = v
        if (count === promises.length) {
          resolve(arr)
        }
      }, r => {
        reject(r)
      })
    })
  })
}

// 添加Promise.race方法
Promise.race = function (promises) {
  return new Promise((resolve, reject) => {
    promises.forEach((item) => {
      item.then(v => {
        resolve(v)
      }, r => {
        reject(r)
      })
    })
  })
}

```
