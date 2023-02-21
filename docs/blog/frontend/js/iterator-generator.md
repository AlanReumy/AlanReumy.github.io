---
title: 迭代器与生成器
---

## 什么是迭代器

**迭代器** 是指确使用户可在容器对象（container，例如链表或数组）上遍访的对象，使用该接口无需关心对象的内部实现细节。

在很多编程语言里，都有自己的迭代器。JavaScript也不例外。在JavaScript中，迭代器是一个对象，它需要符合一种协议，这种协议叫做 **迭代协议**。

这个对象中需要有一个叫做 `next` 的函数，这个函数 **必须有以下要求** ：

1. 它的参数是一个或者没有。
2. 它的返回值是一个包含 `done` 属性 和 `value` 属性的对象。
3. `done` 属性是一个布尔值，如果迭代器可以产生序列中的下一个值，则为 `false` 。如果迭代器已将序列迭代完毕，则为 `true` 。这种情况下，`value` 是可选的，如果它依然存在，即为迭代结束之后的默认返回值。
4. `value` 属性表示迭代器返回的值。

## 编写一个迭代器

下面这个例子，我们来编写一个自己的迭代器。

```js
const names = ['curry', 'james', 'harden']

function createNamesIterator(names){
  let index = 0
  return {
    next:() => {
      if (index < names.length) return { done: false, value: names[index++] }
      else return { done: true, value: undefined }
    }
  }
}

const namesIterator = createNamesIterator(names)

console.log(namesIterator.next())
console.log(namesIterator.next())
console.log(namesIterator.next())
console.log(namesIterator.next())
```

## 可迭代对象

**可迭代对象和迭代器是不同的概念**：

- 迭代器是一个对象，它实现的是迭代器协议(iterator protocol)，要实现一个next方法
- 可迭代对象是一个对象，它实现的是可迭代协议(iterable protocol)，要实现一个 [Symbol.iterator] 的方法，该方法返回一个迭代器

当一个对象实现了可迭代协议的时候，那么它就是一个可迭代对象。

可迭代对象要求必须实现 `@@iterator` 方法，这意味着对象（或者它原型链上的某个对象）必须有一个键为 `@@iterator` 的属性，可通过常量 `Symbol.iterator` 访问该属性。

> 当一个对象变成可迭代对象的时候，可以进行某些迭代操作。比如 `for of` 操作，就是调用 `@@iterator` 方法。

```js
const iterableObj = {
  names: ['curry', 'james', 'harden'],
  [Symbol.iterator]: function () {
    let index = 0
    return {
      next: () => {
        if (index < this.names.length)
          return { done: false, value: this.names[index++] }
        else return { done: true, value: undefined }
      },
    }
  },
}

for (const item of iterableObj) {
    console.log(item)
}
```

## 内置的可迭代对象

JavaScript的很多内置对象都实现了可迭代协议，会生成一个迭代器对象：

包括： `String` 、`Array` 、`Map` 、`Set` 、`arguments对象` 、`NodeList集合` 。

## 可迭代对象的应用场景

可迭代对象在很多地方都被使用：

1. `for of`

```js
// 上一个例子
for (const item of iterableObj) {
    console.log(item)
}
```

2. 展开语法

```js
const arr1 = [1, 2, 3]
const arr2 = [4, 5, 6]
const arr3 = [...arr1, ...arr2]
```

> 注意：对象的展开运算符并不是使用的迭代器，而是ES9新增的特性。

3. 解构语法

```js
const [name1, name2] = names
```

> 注意：对象的解构并不是使用的迭代器，而是ES9新增的特性。

4. 创建其他对象

例如 `Map` 、 `Set` 、`Promise.all` 中，我们传入的参数必须是一个可迭代对象。

```js
const set1 = new Set(iterableObj)
const set2 = new Set(names)

const arr1 = Array.from(iterableObj)

// 5.Promise.all
Promise.all(iterableObj).then(res => {
  console.log(res)
})
```

## 自定义类的迭代

如果我们要设计一个类，并且默认这个类创建处理的是可迭代对象，那么我们在原型上需要加上 `@@iterator` 方法。

```js
class Team {
  constructor(name, players) {
    this.name = name
    this.players = players
  }

  [Symbol.iterator]() {
    let index = 0
    return {
      next: () => {
        if (index < this.players.length) {
          return {
            done: false,
            value: this.players[index++],
          }
        } else {
          return {
            done: true,
            value: undefined,
          }
        }
      },
    }
  }
}

const team1 = new Team('gsw', ['curry', 'green', 'klay'])

const team2 = new Team('lakers', ['ad', 'james', 'kobe'])

for (const player of team1) {
  console.log(player)
}

for (const player of team2) {
  console.log(player)
}
```

## 迭代器的中断

在某些情况下，迭代器会在没有完全迭代完的时候被中断掉。

如果我们需要监听中断，可以实现 `return` 方法。

```js
const iterableObj = {
  names: ['curry', 'james', 'harden'],
  [Symbol.iterator]() {
    let index = 0
    return {
      next: () => {
        if (index < this.names.length)
          return { done: false, value: this.names[index++] }
        else return { done: true, value: undefined }
      },
      return() {
        console.log('被中断了')
        return { done: true }
      },
    }
  },
}

for (const item of iterableObj) {
  console.log(item)
  if (item === 'james') break
}
```

## 什么是生成器

**生成器** 是ES6中新增的一种函数控制、使用的方案，它可以让我们更加灵活的控制函数什么时候继续执行、暂停执

行等操作。

生成器也是一个函数，但是和普通函数存在一些区别：

1. 生成器函数在声明的时候需要在 `function` 后面加一个符号：*。
2. 生成器函数可以通过 `yield` 关键字来控制函数的执行流程。
3. 生成器函数的返回值是一个生成器。

> 注意：生成器本质上是一种特殊的迭代器

## 生成器函数的执行过程与传参

通过调用生成器函数，返回一个生成器对象， 然后可以调用生成器对象的 `next`  方法，执行生成器函数的代码。

在这过程中，可以通过 `yield` 来返回结果。

```js
function* foo(m) {
  console.log('开始执行')

  // 参数m是创建生成器对象的时候传入的参数
  const value1 = 100 * m
  console.log(value1)
  const n = yield value1

  // 参数n是第二次调用next函数时传入的参数
  const value2 = 200 * n
  console.log(value2)
  yield value2
  
  console.log('结束执行')
}

const generator = foo(10)
console.log(generator.next())
console.log(generator.next(2))
console.log(generator.next())
```

## 生成器提前结束

生成器可以调用 `return` 函数，之后调用 `next` 函数就不会继续生成值了，返回的生成器的 `value` 值始终是 `undefined` 。

```js
function* foo(m) {
  console.log('开始执行')

  const value1 = 100 * m
  console.log(value1)
  const n = yield value1

  const value2 = 200 * n
  console.log(value2)
  yield value2
  
  console.log('结束执行')
}

const generator = foo(10)
console.log(generator.next())
console.log(generator.return(10))
console.log(generator.next())
console.log(generator.next())
```

## 生成器抛出异常

可以通过生成器的 `throw` 函数给生成器函数内部抛出异常。

```js
function* foo() {
  console.log("开始执行")

  const value1 = 100
  try {
    yield value1
  } catch (error) {
    console.log("捕获到异常情况:", error)

    yield "abc"
  }

  const value2 = 200
  yield value2

  console.log("执行结束")
}

const generator = foo()

const result = generator.next()
generator.throw("error")
```

## 使用生成器代替迭代器

我们之前提到过，生成器是一种特殊的迭代器，因此在一些情况下，我们可以使用生成器来代替迭代器。

有以下三种写法：

```js
// 生成器来替代迭代器
function* createArrayIterator(arr) {

  // 3.第三种写法 yield* 这种写法是 for of 的语法糖形式
  // yield* arr

  // 2.第二种写法
  for (const item of arr) {
    yield item
  }
  // 1.第一种写法
  // yield "abc" // { done: false, value: "abc" }
  // yield "cba" // { done: false, value: "abc" }
  // yield "nba" // { done: false, value: "abc" }
}

const names = ["abc", "cba", "nba"]
const namesIterator = createArrayIterator(names)

console.log(namesIterator.next())
console.log(namesIterator.next())
console.log(namesIterator.next())
console.log(namesIterator.next())
```

## 使用生成器实现自定义类的迭代

```js
class Team {
  constructor(name, players) {
    this.name = name
    this.players = players
  }

  *[Symbol.iterator]() {
    yield* this.players
  }
}

const team1 = new Team('gsw', ['curry', 'green', 'klay'])

const team2 = new Team('lakers', ['ad', 'james', 'kobe'])

for (const player of team1) {
  console.log(player)
}

for (const player of team2) {
  console.log(player)
}
```
