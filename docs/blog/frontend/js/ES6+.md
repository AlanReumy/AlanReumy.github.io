# ES6+

ES6+指的是ES6之后发布的版本，由于ES6相比ES5版本更新较大，因此一般会把ES6作为单独的版本来说，而ES6之后的版本可以统称为ES6+。

本章节只会介绍一些ES6+零碎的知识点，如果有重要知识点，会提炼出单独的章节做出总结。

## ES7

### Array Includes

在ES7之前，如果我们需要判断一个数组中是否包含某个元素，需要通过 `indexOf` 获取结果，并且判断返回值是否为 `-1` 。

```js
const arr = [1, 2, 3, 4]

// 数组是否包含2这个数字
if (arr.indexOf(2) !== -1) {
  console.log('find it')
}
```

ES7提供了 `includes` 方法来判断一个数组中是否包含一个指定的元素。如果有，返回 `true` ，否则返回 `false` 。

```js
const arr = [1, 2, 3, 4]

if (arr.includes(2)) {
  console.log('find it')
}
```

### 指数运算符

在ES7之前，计算数字的乘方需要通过 `Math.pow` 方法来完成。

```js
const res = Math.pow(2,3)
console.log(res) // 8
```

ES7增加了 `**` 运算符，可以对数字来计算乘方。

```js
const res = 2 ** 3
console.log(res) // 8
```

## ES8

### Object.values

ES8中提供了 `Object.values()` 来获取所有的value值。

```js
const obj = {
  name: 'curry',
  age: 33,
}

console.log(Object.values(obj)) // [ 'curry', 33 ]
```

### Object.entries

通过 `Object.entries` 可以获取到一个数组，数组中会存放可枚举属性的键值对数组。

```js
const obj = {
  name: 'curry',
  age: 33
}

// 如果是一个对象
console.log(Object.entries(obj)) // [ [ 'name', 'curry' ], [ 'age', 33 ] ]

// 如果是一个数组
console.log(Object.entries(['curry','james','harden'])) // [ [ '0', 'curry' ], [ '1', 'james' ], [ '2', 'harden' ] ]

// 如果是一个字符串
console.log(Object.entries('abc')) // [ [ '0', 'a' ], [ '1', 'b' ], [ '2', 'c' ] ]
```

### String Padding

某些字符串我们需要对其进行前后的填充，来实现某种格式化效果。

ES8中增加了 `padStart` 和 `padEnd` 方法，分别是对字符串的首尾进行填充的。

例子：

```js
const str = "hello world"

// 第一个参数是填充后字符的长度，第二个参数是填充的字符
console.log(str.padStart(15,'*')) // ****hello world
console.log(str.padEnd(15,'*')) // hello world****
```

应用场景（对身份证、银行卡的前面位数进行隐藏）：

```js
const cardNumber = '2131252325331231312'

const tempNumber = cardNumber.slice(-4)

const finalNumber = tempNumber.padStart(cardNumber.length, '*')

console.log(finalNumber) // ***************1312
```

### Trailing Commas

ES8中允许在函数定义和调用时多加一个逗号。

```js
function add(a, b, ) {
  return a + b
}

add(1, 1, )
```

### Object.getOwnPropertyDescriptor

通过 `Object.getOwnPropertyDescriptor(obj,prop)` 获取指定对象上一个自有属性对应的属性描述符。（自有属性指的是 **直接赋予该对象的属性** ，不需要从原型链上进行查找的属性）

```js
const obj = {
  name:'curry',
  age:33
}

// {
//   value: 'curry',
//   writable: true,
//   enumerable: true,
//   configurable: true
// }
console.log(Object.getOwnPropertyDescriptor(obj,"name")) 
```

> `Object.getOwnPropertyDescriptors(obj)` 可以获取制定对象的所有自有属性的属性描述符

## ES9

`Async iterators` ：后续迭代器章节。

`Object spread operators` ：ES6的章节中的展开语法介绍过。

`Promise finally` ：后续 `Promise` 章节。

## ES10

### Array flat

`flat()` 方法会按照一个可指定的深度递归遍历数组，并将所有元素与遍历到的子数组中的元素合并为一个新数组返回。

```js
const arr = [1,[24,[46]],29]

console.log(arr.flat()) // [ 1, 24, [ 46 ], 29 ]

// 深度为 2
console.log(arr.flat(2))

// 转换成一维数组
console.log(arr.flat(Infinity)) // [ 1, 24, 46, 29 ]
```

### Array flatMap

`flatMap()` 方法首先使用映射函数映射每个元素，然后将结果压缩成一个新数组。

**注意：**

1. `flatMap` 是先进行 `map` 操作，再做 `flat` 的操作。
2. `flatMap` 中的 `flat` 相当于深度为 1。

```js
const arr = ['hello world','my name is ','stephen curry']

// flat
const newArr1 = arr.flatMap(item => {
  return item.split(" ")
})

// 没有flat
const newArr2 = arr.map(item => {
  return item.split(" ")
})
```

### Object fromEntries

我们可以通过 `Object.entries()` 将一个对象转换成 `entries`，ES9为我们提供了 `Object fromEntries()` 将 `entries` 转换为对象。

```js
const obj = {
  name: 'curry',
  age: 33,
}

const entries = Object.entries(obj)

console.log(entries)
console.log(Object.fromEntries(entries)) // { name: 'curry', age: 33 }
```

### trimStart trimEnd

我们可以通过 `trim()` 方法去除一个字符串首尾的空格。

ES10中给我们提供了 `trimStart` 和 `trimEnd` 单独去除首部或者尾部的空格。

```js
const str = '  hello world   '

console.log(str.trimStart()); // hello world   
console.log(str.trimEnd()) //  hello world
```

### ES10其它知识点

`Symbol description` ：在ES6的章节介绍过。

`Optional catch binding` ：后续介绍异常捕获的章节中介绍。

## ES11

### BigInt

在早期的JavaScript中，我们不能正确的表示过大的数字。

```js
const maxInt = Number.MAX_SAFE_INTEGER
console.log(maxInt) // 9007199254740991
```

超过 `Number.MAX_SAFE_INTEGER` 的数值表示可能是不正确的。

在ES11中引入了新的数据类型 `BigInt` 用来表示大的整数

```js
const bigInt = 9007199254740991n
console.log(bigInt) // 9007199254740991n

// BigInt类型的数字只能与同类型的数字运算
console.log(bigInt + BigInt(10))
```

### Nullish Coalescing Operator（空值合并运算符）

在ES11前，我们使用判断默认值的时候，如果碰到 空字符串 和 数字0 的时候，`逻辑或` 运算符会将它们隐式类型转换为 **布尔类型** 的 `false` 。

ES11增加了 `Nullish Coalescing Operator（空值合并运算符）` ，0与空字符串就不会出现之前的问题了。

```js
const foo = 0

const res1 = foo || 'defalut value'
const res2 = foo ?? 'defalut value'

console.log(res1) // defalut value
console.log(res2) // 0
```

### Optional Chaining（可选链）

ES11新增的 `Optional Chaining（可选链）` ，让我们的代码在进行 `null` 和 `undefined` 判断时更加清晰和简洁。

```js
const obj = {
  name:'curry',
  teamate:{
    // name:'green'
  }
}

console.log(obj?.teamate?.name) // undefined
```

### globalThis

在ES11中，我们获取JavaScript中的全局对象的方式是不同的：

* 浏览器中可以通过 `this` 、 `window` 对象来获取。
* Node中可以通过 `global` 来获取。

ES11对获取全局对象作出了统一规范：使用 `globalThis` 关键字。

```js
console.log(global) // Node
console.log(this) // 浏览器
console.log(window) // 浏览器
console.log(globalThis) // ES11 统一
```

### ES11其它知识点

`Dynamic Import` ：后续模块化章节中介绍。
`Promise.allSettled` ：后续 `Promise` 章节介绍。
`import meta` ：后续模块化中章节中介绍。

## ES12

### FinalizationRegistry

`FinalizationRegistry` 对象可以让你在对象被垃圾回收时请求一个回调。

* `FinalizationRegistry` 提供了这样的一种方法：当一个在注册表中注册的对象被回收时，请求在某个时间点上调用一个清理回调。（清理回调有时被称为 `finalizer` ）。
* 你可以通过调用 `register` 方法，注册任何你想要清理回调的对象，传入该对象和所含的值。

```js
let obj = {
  name: 'curry',
  age: 33
}

const registry = new FinalizationRegistry(value => {
  console.log("对象被销毁了",value)
})

registry.register(obj,"obj")

obj = null
```

### WeakRef

如果我们希望一个对象赋值给另外一个引用是弱引用，可以使用 `WeakRef` 。

```js
const finalRegistry = new FinalizationRegistry((value) => {
  console.log("注册在finalRegistry的对象, 某一个被销毁", value)
})

let obj = { name: "why" }
let info = new WeakRef(obj)

finalRegistry.register(obj, "obj")

obj = null

// WeakRef.prototype.deref:
setTimeout(() => {
  console.log(info.deref()?.name)
  console.log(info.deref() && info.deref().name)
}, 10000)
```

可以使用 `WeakRef.prototype.deref()` 获取到原来的对象：

* 如果原来的对象没有销毁，那么可以获取到原来的对象。
* 如果原来的对象已经销毁了，那么获取到的是 `undefined` 。

### logical assignment operators

#### 逻辑或赋值运算

```js
let str = ""

str ||= "default value"
// 相当于
str = str || "default value"
console.log(str) // default value
```

#### 逻辑空运算符

```js
let str = ""

str ??= "default value"
// 相当于
str = str ?? "default value"
console.log(str) // ""
```

#### 逻辑与运算符

```js
let obj = {
  name: 'curry',
  age: 33
}

obj &&= obj.name
console.log(obj) // curry
```

### ES12它知识点

`Numeric Separator` ：长数字的分隔符，ES6章节中介绍过。

`String.replaceAll(pattern, replacement)` ：新字符串所有满足 `pattern` 的部分都已被 `replacement` 替换。

## 总结

本章主要介绍了ES6+零碎的知识，并对这些知识进行了简单的demo演示，如果需要更加深入了解，请阅读 `ECMA规范` 或 `MDN文档` 。
