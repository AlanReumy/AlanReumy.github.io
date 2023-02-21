# Vue 响应式原理

在聊响应式之前，我们先来说一说ES6新增的 `Proxy` 与 `Reflect` 。

## 监听对象

我们在ES6之前，其实是可以通过 `Object.defineProperty()` 实现对象的监听操作。

```js
Object.keys(obj).forEach(key => {
  let value = obj[key]
  Object.defineProperty(obj,key,{
    set(newValue){
      console.log(`监听到${key}的变化`)
      value = newValue
    },
    get(){
      return value
    }
  })
})
```

**但是这样做是有缺点的**：因为 `Object.defineProperty()` 的设计初衷就不是为了监听对象来设计，其次如果我们想要监听更加丰富的操作：新增、删除等，这个方法是无法实现的。

## Proxy

在ES6中，新增了一个 `Proxy` 类，这个类从名字就可以看出来，是用于帮助我们创建一个代理的：

也就是说，如果我们希望 **监听一个对象** 的相关操作，那么我们可以先创建一个代理对象（ `Proxy` 对象）。

之后对该对象的所有操作，都通过代理对象来完成，代理对象可以监听我们想要对原对象进行的操作。

### Proxy实现监听

我们将上面的案例用 `Proxy` 来实现：

```js
const obj = {
  name: 'curry',
  age: 33,
}

// 创建代理对象
// target指的是目标对象，handler指的是包含捕获器的处理器对象
// new Proxy(target,handler)
const objProxy = new Proxy(obj, {})
```

之后我们要操作对象的话，直接操作创建出来的代理对象即可。

```js
const obj = {
  name: 'curry',
  age: 33,
}

const objProxy = new Proxy(obj, {
  get(target, key, recevicer) {
    return target[key]
  },
  set(target, key, newValue, recevicer) {
    target[key] = newValue
  },
  has(target, key, recevicer) {
    return key in target
  },
  deleteProperty(target, key) {
    delete target[key]
  },
})
```

### Proxy的捕获器

`Proxy` 的捕获器一共有13种，常用的捕获器大概有以下几种：

`handler.has()` ：`in` 操作符的捕捉器。

`handler.get()` ：属性读取操作的捕捉器。

`handler.set()` ：属性设置操作的捕捉器。

`handler.deleteProperty()` ：`delete` 操作符的捕捉器。

`handler.apply()` ：函数调用操作的捕捉器。

`handler.construct()` ：`new` 操作符的捕捉器。

## Reflect

`Reflect` 也是ES6新增的一个 `API` ，它是一个对象，字面的意思是反射。

### Reflect的作用

`Reflect` 提供了很多操作对象的方法，这些方法与 `Object` 中操作对象的方法类似。

既然有了 `Object` 中的方法可以做这些操作，**为什么需要新增 `Reflect` 对象呢？**

这是因为在早期的 `ECMA规范` 没有考虑到这种对 **对象的操作** 如何设计会更加规范，所以将这些 `API` 放在了 `Object` 上。

但是 `Object` 作为构造函数，使用这些操作并不合适。

因此才新增了 `Reflect` 对象。

### Reflect中的方法

`Reflect` 中的很多方法都与 `Object` 中的方法类似，而且它的所有方法与 `Proxy` 的13种捕获器一一对应。

### Reflect的使用

我们可以将 `Proxy` 中的捕获器中对对象的操作从 `Object` 中的方法修改为 `Reflect` 中的方法。

```js
const obj = {
  name: 'curry',
  age: 33,
}

const objProxy = new Proxy(obj, {
  get(target, key) {
    // return target[key]
    return Reflect.get(target,key)
  },
  set(target, key, newValue) {
    // target[key] = newValue
    Reflect.set(target,key,newValue)
  },
  has(target, key) {
    // return key in target
    return Reflect.has(target,key)
  },
  deleteProperty(target, key) {
    // delete target[key]
    Reflect.deleteProperty(target,key)
  },
})
```

### Proxy与Reflect中的Receiver的作用

如果我们的源对象有setter、getter的访问器属性，那么可以通过 `receiver` 来改变里面的 `this` ，将 `this` 指向了 `proxy` 对象。

```js
const obj = {
  _name: 'curry',
  age: 33,
  get name(){
    console.log(this)
    return this._name
  },
  set name(newValue){
    console.log(this)
    this._name = newValue
  }
}

const objProxy = new Proxy(obj, {
  // receiver是创建出来的代理对象
  get(target, key, receiver) {
    // return target[key]
    console.log('getter')
    // 此时 obj里面的getter中的this指向的是 objProxy对象
    return Reflect.get(target, key, receiver)
  },
  set(target, key, newValue, receiver) {
    // target[key] = newValue
    console.log('setter')
    Reflect.set(target, key, newValue, receiver)
  },
  has(target, key) {
    // return key in target
    return Reflect.has(target, key)
  },
  deleteProperty(target, key) {
    // delete target[key]
    Reflect.deleteProperty(target, key)
  },
})

objProxy.name = 'james'
```

### Reflect中的construct

利用 `Reflect.construct(target, argumentsList, newTarget)` 方法，可以实现通过一个函数作为构造函数，创建出另外一个类型的操作。

```js
function Person(name,age){
  this.name = name
  this.age = age
}

function Coder(){

}

const obj = Reflect.construct(Person,['curry',33], Coder)
console.log(obj.__proto__ == Coder.prototype) // true
```

## 响应式

说完 `Proxy` 与 `Reflect` ，我们就可以来看一看响应式，首先来了解，**什么是响应式** 。

### 什么是响应式

来看下面这段代码：

```js
let m = 10

// m变化时，需要重复执行的代码
console.log(m)
console.log(m + 2)

m = 40
```

在这个例子中，我们想要实现当变量 `m` 变化时，输出的那两句代码可以重复执行。像这样的一种可以自动响应数据变量的代码机制，我们称之为是 **响应式** 。

### 响应式函数设计

在我们的开发中，会封装很多函数，那么我们需要区分一个函数是否需要响应式。

```js
// 需要响应式
function foo(){
  const value = obj.name
  console.log(value)
}

// 不需要响应式
function bar(){
  let res = 1 + 1
  console.log(res)
}
```

很明显，上面这段代码中的两个函数：

对于 `foo` 来说，需要在 `obj.name` 发生变化的时候，重新执行，做出响应。

对于 `bar` 来说，它是一个完全独立的函数，不需要执行任何的响应式操作。

### 响应式函数实现

我们需要封装一个函数，来区别与普通函数，告诉开发者，只要是通过这传入这个函数的函数，都是响应式的函数。其他默认都不是响应式的函数。

```js
const reactiveFns = []

function watchFn(fn) {
  reactiveFns.push(fn)
  fn()
}

watchFn(function(){
  const value = obj.name
  console.log(value)
})
```

### 响应式依赖的收集

如果我们仅仅用一个 `reactiveFns` 的数组来保存所有的响应式函数，是不现实的。因为，在实际开发中，需要很多创建对象，而对象的每一个属性都需要对应自己的一组响应式函数。

因此，我们需要一个类，用来专门收集一个对象的某一个函数的依赖。（替代了原来的 `reactiveFns` 数组）

`Depend` 类

```js
class Depend{
  constructor(){
    this.reactiveFns = []
  }

  // 添加依赖
  addDepend(fn){
    this.reactiveFns.push(fn)
  }

  // 做出响应
  notify(){
    this.reactiveFns.forEach(fn => {
      fn()
    })
  }
}
```

`watchFn` 函数

```js
const depend = new Depend()

function watchFn(fn){
  depend.addDepend(fn)
  fn()
}
```

### 监听对象的变化

接下来我们就可以使用 `Proxy` 与 `Reflect` 监听对象的变化

```js
const objProxy = new Proxy(obj, {
  get(target, key, receiver) {
    return Reflect.get(target, key, receiver)
  },
  set(target, key, newValue, receiver) {
    Reflect.set(target, key, newValue, receiver)
  },
})
```

### 对象的依赖管理

我们目前创建的 `depend` 对象，只能用来收集对于 `obj` 对象的 某一个属性的变化需要监听的响应函数。

但是在开发中，我们需要对不同的对象的不同的属性进行不同的响应。

这个时候，我们需要借助ES6中新增的数据结构：`WeakMap` 。它对应的数据结构如下图所示：

![对应的数据结构](https://codertzm.oss-cn-chengdu.aliyuncs.com/20211214134324.png)

### 对象的依赖管理的实现

我们可以封装一个 `getDepend` 函数，用来管理这种依赖关系

```js
function getDepend(target, key) {
  let map = targetWeakMap.get(target)
  if (!map) {
    map = new Map()
    targetWeakMap.set(target, map)
  }

  let depend = map.get(key)
  if(!depend){
    depend = new Depend()
    map.set(key,depend)
  }

  return depend
}
```

目前存在的问题是：

`watchFn` 函数这种收集方式根本不知道是哪一个 `key` 的哪一个 `depend` 需要收集依赖。

并且只能针对一个单独的 `depend` 对象来添加你的依赖对象。

所以我们需要做出修改：如果一个函数中使用了某个对象的 `key` ，那么它就应该被收集

```js
// watch 函数的修改
function watchFn(fn) {
  activeReactiveFn = fn
  fn()
  activeReactiveFn = null
}

// proxy对象捕获器的修改
const objProxy = new Proxy(obj, {
  get(target, key, receiver) {
    // 获取依赖
    const depend = getDepend(target,key)
    // 添加依赖
    depend.addDepend(activeReactiveFn)
    return Reflect.get(target, key, receiver)
  },
  set(target, key, newValue, receiver) {
    Reflect.set(target, key, newValue, receiver)
    const depend = getDepend(target,key)
    depend.notify()
  },
})
```

### Depend对象的重构

目前还存在两个问题：

1. 如果一个函数用到了多次同一个 `key` ，那么这个函数会被收集两次。

2. 我们并不希望将 `activeReactiveFn` 放到 `get` 捕获器中，它是属于 `Depend` 对象的行为。

解决问题一：使用 `Set` 对象，而非数组。

解决问题二：修改原有 `addDepend` 方法。

```js
class Depend {
  constructor() {
    // 使用Set对象
    this.reactiveFns = new Set()
  }

  // 添加依赖
  addDepend() {
    if (activeReactiveFn) {
      this.reactiveFns.add(activeReactiveFn)
    }
  }

  // 做出响应
  notify() {
    this.reactiveFns.forEach(fn => {
      fn()
    })
  }
}
```

### 创建响应式对象

目前我们的响应式仅仅针对于 `obj` ，我们可以将响应式提取出一个函数，用于将所有对象都变成响应式对象。

```js
function reactive(obj){
  return new Proxy(obj,{
    get(target, key, receiver) {
      const depend = getDepend(target,key)
      depend.addDepend()
      return Reflect.get(target, key, receiver)
    },
    set(target, key, newValue, receiver) {
      Reflect.set(target, key, newValue, receiver)
      const depend = getDepend(target,key)
      depend.notify()
    },
  })
}
```

至此，我们的响应式实现完毕，`Vue3` 就是基于 `Proxy` 与 `Reflect` 来实现的这种响应式功能。

完整代码如下：

```js
let activeReactiveFn = null

const targetMap = new WeakMap()

// 普通
const obj = {
  name: 'curry',
  age: 33,
}

class Depend {
  constructor() {
    // this.reactiveFns = []
    this.reactiveFns = new Set()
  }

  addDepend() {
    if (activeReactiveFn) {
      this.reactiveFns.add(activeReactiveFn)
    }
  }

  notify() {
    this.reactiveFns.forEach(fn => {
      fn()
    })
  }
}

// 监视
function watchFn(fn) {
  activeReactiveFn = fn
  fn()
  activeReactiveFn = null
}

// 获取依赖集
function getDepend(target, key) {
  let map = targetMap.get(target)
  if (!map) {
    map = new Map()
    targetMap.set(target, map)
  }

  let depend = map.get(key)
  if (!depend) {
    depend = new Depend()
    map.set(key, depend)
  }

  return depend
}

// 实现对象的响应式
function reactive(obj) {
  return new Proxy(obj, {
    get(target, key, receiver) {
      const depend = getDepend(target, key)
      depend.addDepend()
      return Reflect.get(target, key, receiver)
    },
    set(target, key, newValue, receiver) {
      Reflect.set(target, key, newValue, receiver)
      const depend = getDepend(target, key)
      depend.notify()
    },
  })
}

const objProxy = reactive(obj)

watchFn(function () {
  console.log(objProxy.name, 'name------')
  console.log(objProxy.name, 'name------')
})

watchFn(function () {
  console.log(objProxy.age, 'age-----')
})

const infoProxy = reactive({
  address:'beijing'
})

watchFn(function(){
  console.log(infoProxy.address, 'address-----')
})

objProxy.name = 'hello'
objProxy.age = 30

infoProxy.address = 'chongqing'
```

### Vue2 的响应式

`Vue2` 是通过 `Object.defineProperty()` 实现的响应式。

我们可以将 `reactive()` 函数进行重构：

在传入对象时，我们可以遍历所有的 `key` ，并且通过属性存储描述符来监听属性的获取和修改。

```js
// 实现对象的响应式
function reactive(obj) {
  Object.keys(obj).forEach(key => {
    let value = obj[key]
    Object.defineProperty(obj, key, {
      get() {
        const depend = getDepend(obj,key)
        depend.addDepend()
        return value
      },
      set(newValue) {
        value = newValue
        const depend = getDepend(obj,key)
        depend.notify()
      },
    })
  })
  return obj
}
```

## 总结

在本章节，我们介绍了 `Proxy` 、`Reflect` 以及响应式原理。一步步从零开始，将ES6中新增的一些特性结合起来，实现了对对象的响应式。同时回顾了 `Vue2` 和 `Vue3` 响应式的不同实现。
