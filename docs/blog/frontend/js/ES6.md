# ES6

## class关键字

在ES6之前，我们都是使用的构造函数的形式来创建自定义类型的对象

```js
function Foo(bar) {
  this.bar = bar
}
var foo = new FOo('curry')
```

ES6为我们提供了一种新的语法，使用 `class` 关键字来创建类，但它的本质依旧是构造函数，`class` 关键字只是一种语法糖而已。

### 如何定义类

类的声明有两种方式：类声明和类表达式

```js
// 类声明
class Foo {

}

// 表达式
var Foo = class {
  
}
```

### 类的构造函数

如果我们希望在创建对象的时候给类传递一些参数，这个时候应该如何做？

在`class`中有一个叫做 `constructor`（名称固定）的构造函数，当我们通过`new`关键字创建一个类的时候，就会调用这个类的 `constructor` 方法。

> 每个类中只能有一个 `constructor` 方法，否则会抛出异常

```js
class Foo {
  constructor(bar) {
    this.bar = bar
  }
}
```

### 类的实例方法

通过`constructor`方法将对象的属性绑定在实例对象上，但是对于方法而言，我们希望将它们放在原型上。我们可以直接将方法写在类中，作为原型的方法。

```js
class Foo {
  constructor(bar) {
    this.bar = bar
  }

  running() {
    console.log(this.bar + 'running')
  }
}
```

### 类的访问器方法

类中也是有访问器方法的。

```js
class Foo {
  constructor(bar) {
    this._bar = bar
  }

  running() {
    console.log(this.bar + 'running')
  }

  set bar(newBar){
    this._bar = newBar
  }

  get bar(){
    return '123'
  }
}
```

### 类的静态方法

我们可以在类上直接定义方法，这样的方法被称为静态方法

```js
class Person {
  constructor(age) {
    this.age = age
  }

  static createPerson() {
    return new Person(Math.floor(Math.random()*100+1))
  }
}
```

### ES6中类的继承（extends）

在ES6之前，如果我们需要实现继承，需要自己创建函数，使用 **寄生组合式继承** 来实现，但ES6为我们提供了 `extends` 关键字，它也仅仅是一种语法糖而已，内部实现的方式其实就是寄生组合式继承。

```js
class Person {
  constructor(age) {
    this.age = age
  }
}

// 继承
class Coder extends Person{

}
```

### super关键字

在子（派生）类的构造函数中使用`this`或者返回默认对象之前，必须先通过 `super` 调用父类的构造函数。

```js
// 继承
class Coder extends Person{
  constructor(lang,age){
    super(age)
    this.lang =lang
  }
}
```

`super` 关键字只能用在：子类的构造函数、实例方法、静态方法。

因此，我们可以通过 `super` 关键字复用父类中的方法，在子类中来对其进行重写。

```js
class Person {
  constructor(age) {
    this.age = age
  }

  personMethod(){
    console.log(this.age)
  }

  static personStaticMethod(){
    console.log('a person')
  }
}

class Coder extends Person {
  constructor(lang,age){
    super(age)
    this.lang = lang
  }

  // 重写父类方法
  personMethod(){
    // 复用父类代码
    super.personMethod()
    console.log(this.lang)
  }

  // 重写父类静态方法
  static personStaticMethod(){
    // 复用父类代码
    super.personStaticMethod()
    console.log('a coder')
  }
}
```

### 继承内置类

我们可以继承内置类来对其进行扩展。

```js
class myArray extends Array {
  lastItem () {
    return this[this.length - 1]
  }
}

var myarr = new myArray(1,2,3)
myarr.lastItem() // 3
```

### 类的混入（mixin）

在JavaScript中只能实现单继承，如果我们需要在一个类中添加多个类的方法可以使用 **混入（mixin）** 。这种思想在`React`的高阶组件中也有体现，这里不细说。

```js
class Person {
  constructor(name) {
    this.name = name
  }
}

function mixinCoder(BaseClass) {
  return class extends BaseClass {
    coding() {
      console.log('coding')
    }
  }
}

function mixinRunner(BaseClass) {
  return class extends BaseClass {
    running() {
      console.log('running')
    }
  }
}

// 混入
class NewPerson extends mixinRunner(mixinCoder(Person)){
  constructor(name){
    super(name)
  }
}

var p = new NewPerson('curry')
p.running() // running
p.coding() // coding
```

## 字面量的增强

ES6中对 **对象字面量** 进行了增强，称之为 Enhanced object literals（增强对象字面量）

字面量的增强主要包括下面几部分：

属性的简写：**Property Shorthand**

方法的简写：**Method Shorthand**

计算属性名：**Computed Property Names**

### 属性的简写

```js
var name = 'curry'
var age = 18
var obj = {
  // ES6之前
  name: name,
  age: age,
  
  // ES6
  name,
  age,
}
```

### 方法的简写

```js
var obj = {
  // ES6之前
  foo:function(){},
  
  // ES6
  foo(){}
}
```

### 计算属性名

```js
var foo = 'curry'

var obj = {
  // 计算属性名
  [foo + '123']() {
    console.log('bar')
  },
}

obj[foo + '123']()
```

## 解构

ES6中新增了一个从数组或对象中方便获取数据的方法，称之为 **解构Destructuring** 。

解构分为两种：数组的解构和对象的解构。

### 数组的解构

1. 基本解构过程

   ```js
   var names = ['curry', 'james', 'harden']
   // ES6之前
   var item1 = names[0]
   var item2 = names[1]
   var item3 = names[2]
   
   // ES6
   // 数组解构
   var [item1, item2, item3] = names
   
   var [itemx, ...newNames] = names
   console.log(newNames)
   
   var [itema, itemb, itemc, itemd = 'kobe'] = names
   ```

2. 顺序解构

   ```js
   // 解构后面的元素
   var [, , itemz] = names
   console.log(itemz) // 'harden'
   ```

3. 解构出数组

   ```js
   // 解构出一个元素,后面的元素放到一个新数组中
   var [itemx, ...newNames] = names
   console.log(newNames) // ['james','harden']
   ```

4. 默认值

   ```js
   // 解构的默认值
   var [itema, itemb, itemc, itemd = 'kobe'] = names
   console.log(itemd) // kobe
   ```

### 对象的解构

1. 基本解构过程

   ```js
   var obj = {
     name: 'curry',
     age: 33,
     height: 1.93
   }
   
   var { name, age, height } = obj
   console.log(name, age, height) // 'curry',33,1.93
   ```

2. 任意顺序

   ```js
   var { age } = obj
   console.log(age) // 33
   ```

3. 重命名

   ```js
   var { name: newName } = obj
   console.log(newName) // 'curry'
   ```

4. 默认值

   ```js
   var { name: newName = "james" } = obj
   console.log(newName) // 'curry' 没有默认值是'james'
   ```

## let与const

在ES6之前，我们只能使用 `var` 关键字来声明变量，从从ES6开始新增了两个关键字可以声明变量：`let` 、`const` 。

### let关键字

从直观的角度来说，let和var是没有太大的区别的，都是用于声明一个变量。

### const关键字

`const` 声明的变量一旦被赋值，就不能被修改，但是如果赋值的是 **引用类型** ，那么可以通过引用找到对应的对象，修改对象的内容。

### let与const的作用域提升

在 [运行原理与作用域](frontend/JavaScript/scope.md)中，我们知道 `var` 关键字是有变量提升的，但是如果我们使用 `let` 声明的变量，在声明之前访问会报错，因此 `let` 与 `const` 是没有作用域提升的。

```js
console.log(foo) // Cannot access 'foo' before initialization
let foo = 1;
```

这并不意味着 `let` 声明的变量只有在代码执行阶段才会创建，根据 `ECMA262`规范的描述 **这些变量会被创建在包含他们的词法环境被实例化时，但是是不可以访问它们的，直到词法绑定被求值** 。

### window对象添加属性

我们知道使用 `var` 关键字声明的变量会在 `window` 对象中添加这个属性。

但是 `let` 和 `const` 不会给 `window` 对象添加属性。

那么这个变量保存在哪里呢？

在 [运行原理与作用域](frontend/JavaScript/scope.md)中，我们最后提到了最新的 `ECMA标准` 中对执行上下文的表述的变化：

> 每一个执行上下文会关联到一个变量环境（**variable Environment**），在执行代码中变量和函数的声明会作为**环境记录**（Environment Record）添加到变量环境中。

这就意味着我们声明的变量和环境记录是被添加到变量环境中的，这跟JS引擎的内部实现是有关系的。

比如V8引擎中是通过VariableMap的一个hashmap来实现它们的存储的。

而对于`window` 对象是早期的 `GO` 对象，在最新的实现中其实是浏览器添加的全局对象，并且一直保持了 `window` 和 `var` 之间值的相等性。

### 块级作用域

在ES6之前，JavaScript只会形成两个作用域：全局作用域和函数作用域。

在ES6中新增了块级作用域，并且通过 `let`、`const`、`function`、`class` 声明的标识符是具备块级作用域的限制的：

```js
{
  let foo = '123'
  console.log(foo)
  function bar() {
    console.log('bar')
  }
}
bar() // bar

console.log(foo) // foo is not defined
```

我们发现函数拥有块级作用域，但是外面依然是可以访问的：

这是因为引擎会对函数的声明进行特殊的处理，允许像 `var` 那样进行提升。

### 暂时性死区

在ES6中，我们还有一个概念称之为 **暂时性死区** ：

它表达的意思是在一个代码中，使用 `let` 、`const` 声明的变量，在声明之前，变量都是不可以访问的；

我们将这种现象称之为 **temporal dead zone（暂时性死区，TDZ）**。

```js
var foo = "123"

if (true) {
  console.log(foo) // Cannot access 'foo' before initialization
  let foo = "456" 
}
```

## 模板字符串

在ES6之前，如果我们想要将字符串和一些动态的变量（标识符）拼接到一起，是非常麻烦的。

在ES6中，我们可以使用模板字符串来嵌入 **JS的表达式** 来进行字符串的拼接：

```js
console.log(`my name is ${obj.name}, age is ${obj.age}`)
```

### 标签模板字符串

模板字符串还有另外一种用法：标签模板字符串（Tagged Template Literals）。

```js
function foo(...args){
  console.log(args)
}
const name = 'curry'
const age = 33

foo`hello ${name} world ${age}` // [ [ 'hello ', ' world ', '' ], 'curry', 33 ]
```

上面这个例子中，我们使用标签模板字符串，并且在调用的时候插入其他的变量，我们发现：

* 模板字符串被拆分了。

* 第一个元素是数组，是被模块字符串拆分的字符串组合。

* 后面的元素是一个个模块字符串传入的内容。

## 函数相关

### 默认参数

在ES6之前，我们编写的函数参数是没有默认值的，所以我们在编写函数时，如果有下面的需求：

* 传入了参数，那么使用传入的参数；

* 没有传入参数，那么使用一个默认值；

我们需要写出如下的代码：

```js
function foo() {
  var x = arguments.length > 0 && arguments[0] !== undefined ? arguments[0] : 10;
  var y = arguments.length > 1 && arguments[1] !== undefined ? arguments[1] : 20;
  console.log(x, y);
}

foo();
```

而在ES6中，我们可以给函数的参数以 **默认值** ：

```js
function foo(x=10,y=20){
 console.log(x,y)
}

foo()
```

默认值也可以和解构一起使用：

```js
function foo({ name, age } = {name:'curry',age:33}) {
  console.log(name, age)
}

foo() // curry 33
```

也可以这样写：

```js
function foo({ name = 'curry', age = 33 } = {}) {
  console.log(name, age)
}

foo()
```

### 剩余参数

ES6中引用了 **rest parameter（剩余参数）**，可以将不定数量的参数放入到一个数组中。

> 剩余参数必须放到最后一个位置，否则会报错。

#### 剩余参数和arguments的区别

1. 剩余参数只包含那些没有对应形参的实参，而 `arguments`对象包含了传给函数的所有实参。

2. `arguments` 对象不是一个真正的数组，而 `rest` 参数是一个真正的数组，可以进行数组的所有操作。

3. `arguments` 是早期的 `ECMAScript` 中为了方便去获取所有的参数提供的一个数据结构，而 `rest` 参数是ES6中提供

   并且希望以此来替代 `arguments` 的。

### 箭头函数

ES6提供了一种根据方便的编写函数的写法，叫做 **箭头函数** 。

```js
const foo = () => {}
```

它与普通函数有如下差别：

* 在[this指向](frontend/JavaScript/this.md)中，我们提到过，**箭头函数没有自己的 `this`**，它只会向上层作用域查找 `this` 。

* 箭头函数没有显示原型，因此它也不能通过 `new` 关键字来调用。

  ```js
  foo.prototype // undefined
  ```

* 箭头函数只能通过表达式来声明

### 展开语法

```js
const names = ["curry", "james", "harden"]
const name = "messi"
const obj = {name: "cr7", age: 37}

// 1.函数调用时
function foo(x, y, z) {
  console.log(x, y, z)
}

foo(...names)
foo(...name)

// 2.构造数组时
const newNames = [...names, ...name]
console.log(newNames)

// 3.构建对象字面量时ES2018(ES9)
const obj = { ...info, address: "广州市", ...names }
console.log(obj)
```

使用展开语法：

* 可以在函数调用/数组构造时，将数组表达式或者字符串在语法层面展开；

* 还可以在构造字面量对象时, 将对象表达式按key-value的方式展开；

展开语法的场景：

* 在函数调用时使用；

  ```js
  const names = ["curry", "james", "harden"]
  const name = "messi"
  const info = { name: "curry", age: 33 }
  
  function foo(x, y, z) {
    console.log(x, y, z)
  }
  
  foo(...names) // curry james harden
  foo(...name) // m e s
  ```

* 在数组构造时使用；

  ```js
  const newNames = [...names, ...name]
  console.log(newNames) // ['curry','james','harden'],'m','e','s','s','i']
  ```

* 在构建对象字面量时，也可以使用展开运算符，这个是在ES2018（ES9）中添加的新特性；

  ```js
  const obj = { ...info, team: "gsw", ...names }
  
  // {
  //   '0': 'curry',
  //   '1': 'james',
  //   '2': 'harden',
  //   name: 'curry',
  //   age: 33,
  //   team: 'gsw'
  // }
  console.log(obj)
  ```

展开语法只是一种浅拷贝，下面这段代码中在内存表现大致如下：

```js
const foo = {
  name: 'curry',
  friend:{
    name:'kobe'
  }
}

const bar = {...foo}
foo.friend.name = 'james'
console.log(bar.friend.name) // james
```

![image-20211208172052795](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20211208172052795.png)

可以看到：`foo` 和 `bar` 中的 `friend` 属性都指向同一个对象，这也证明了展开语法是一种浅拷贝的说法。

## 数值的表示

ES6规范了二进制和八进制的写法：

```js
const num2 = 0b100 // 二进制
const num3 = 0o100 // 八进制
```

另外ES2021新增了如下特性：数字过长时，可以使用_作为连接符：

```js
const num = 10_000_000_000_000_000
```

## Symbol

`Symbol` 是ES6中新增的一个基本数据类型，翻译为符号。

为什么需要 `Symbol` ？

* 在ES6之前，对象的属性名其实都是**字符串形式**，那么很容易造成属性名的冲突。

* 如果原来有一个对象，我们希望在其中添加一个新的属性和值，但是我们在不确定它原来内部有什么内容的情况下，很容易造成冲突，从而**覆盖掉它内部的某个属性**。

* 开发中我们如果使用混入，那么混入中出现了**同名的属性**，必然有一个会被覆盖掉。

我们可以发现：`Sybmol` 其实就是用来 **生成一个独一无二的值** 。

Symbol函数执行后每次创建出来的值都是独一无二的，**我们也可以在创建Symbol值的时候传入一个描述description**：

```js
const s1 = Symbol()
const s2 = Symbol()
const s3 = Symbol('abc')
console.log(s1 === s2)
```

我们可以使用 `Symbol` 来作为属性名，有三种方式：

```js
// 1.定义字面量直接使用
const obj = {
  [s3]:'james'
}

// 2.属性名赋值
obj[s3] = 'james'

// 3.Object.defineProperty
Object.defineProperty(obj,s3,{
  value:'james'
})
```

如果我们想通过 `Symbol` 创建相同的值，可以使用 `Symbol.for()`这个函数。

```js
const s4 = Symbol.for('abc')
const s5 = Symbol.for('abc')
console.log(s4 === s5) // true
```

也可以使用 `Symbol.keyFor()` 这个函数来得到 `Symbol` 的 描述。

```js
const s4 = Symbol.for('abc')
const s5 = Symbol.for('abc')
const key = Symbol.keyFor(s5) // 'abc'
const s6 = Symbol.for(key)
console.log(s4 === s5) // true
console.log(s4 === s6) // true
```

## Set

在ES6之前，我们存储数据的结构主要有两种：数组和对象，

ES6新增了两种数据结构：`Set` 、 `Map` 以及它们的另外的形式 `WeakSet` 、 `WeakMap` 。

`Set` 用来保存数据，类似于数组，但是和数组的区别是 **元素不能重复** 。

### 创建Set

```js
// 创建Set
const set = new Set()
set.add(1)
set.add(2)
set.add(1)
console.log(set) // Set(2) { 1, 2 }
```

### 数组去重

在没有 `Set` 之前，如果需要进行数组去重，我们需要写出这样的代码：

```js
const arr = [1, 2, 1, 4]

const newArr = []
for (var i = 0; i < arr.length; i++) {
  if (newArr.indexOf(arr[i]) === -1) {
    newArr.push(arr[i])
  }
}
console.log(newArr)
```

在有了 `Set` 之后，我们可以这样实现：

```js
const arr = [1, 2, 1, 4]
const newArr = [...(new Set(arr))]

console.log(newArr) // [ 1, 2, 4 ]
```

看起来简洁了很多。

### Set常见的属性和方法

#### 常见属性

* `size`：返回 `Set` 中元素的个数。

#### 常见方法

* `add(value)`：添加某个元素，返回 `Set` 对象本身。
* `delete(value)`：从 `Set` 中删除和这个值相等的元素，返回 `boolean` 类型。
* `has(value)`：判断 `set` 中是否存在某个元素，返回 `boolean`  类型。
* `clear()`：清空 `Set` 中所有的元素，没有返回值。
* `forEach(callback, [, thisArg])`：通过 `forEach` 遍历 `Set` 。
* Set也支持 `for of` 遍历。

## WeakSet

和 `Set` 类似的另外一个数据结构称之为 `WeakSet` ，也是内部元素不能重复的数据结构。

### WeakSet和Set的区别

1. `WeakSet` 中只能存放对象类型，不能存放基本数据类型。
2. `WeakSet` **对元素的引用是弱引用**（后面会说什么是弱引用），如果没有其他引用对某个对象进行引用，那么 `GC（垃圾回收器）` 可以对该对象进行回收。

常见方法：

1. `add(value)`：添加某个元素，返回 `WeakSet` 对象本身。

2. `delete(value)`：从 `WeakSet` 中删除和这个值相等的元素，返回 `boolean` 类型。

3. `has(value)`：判断 `WeakSet` 中是否存在某个元素，返回 `boolean` 类型。

### WeakSet中对元素的引用是弱引用

我们上面提到，`WeakSet` 对元素的引用是弱引用，那么什么是弱引用呢？通过一个例子我们来认识一下：

如果我们使用 `Set` 来存放对象类型，那么它在内存表现是这样的**（红色的线代表强引用）**：

```js
const obj = {
  name: 'curry',
  age: 33
}

const set = new Set()
set.add(obj)
```

![image-20211208230549399](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20211208230549399.png)

而使用 `WeakSet` 来存放对象类型的数据，它的内存表现是这样的**（红色的线代表强引用，黄色的线代表弱引用）**：

```js
const obj = {
  name: 'curry',
  age: 33
}

const wset = new WeakSet()
wset.add(obj)
```

![image-20211208230722876](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20211208230722876.png)

我们上面介绍了：

> `WeakSet` **对元素的引用是弱引用**，如果没有其他引用对某个对象进行引用，那么 `GC（垃圾回收器）` 可以对该对象进行回收。

**这也就意味着：**

如果我们使用 `Set` 存放对象类型，当我们将 `obj` 赋值为 `null` ，虽然 `obj` 不再指向 `0xa00` 这个内存空间，但是 `set变量` 中的一个元素仍然保存着 `0xa00` 的引用（由于它是强引用），因此，`0xa00` 这块内存空间仍然不会被 `GC` 回收。

如果我们使用 `WeakSet` 存放对象类型，当我们将 `obj` 赋值为 `null` ， `obj` 不再指向 `0xa00` 这个内存空间。并且，由于 `WeakSet` 保存的元素的引用是弱引用，因此，`WeakSet` 不再保存对这块内存空间的引用， `0xa00` 这块内存空间将会被 `GC` 回收。

### WeakSet的应用场景

事实上，`WeakSet` 的应用场景并不多。

如果我们需要判断一个对象是否是通过自己的实例来调用该对象内部的方法，我们可以使用 `WeakSet` ：

```js
const pwset = new WeakSet()
class Person {
  constructor() {
    pwset.add(this)
  }

  running() {
    if (!pwset.has(this)) {
      throw new Error('不能通过其他对象调用running方法')
    }
    console.log('running')
  }
}

const p = new Person()

p.running() // running
p.running.call({ name: 'curry' }) // error
```

## Map

另外一个新增的数据结构是 `Map` ，用于存储映射关系。

### Map与普通对象的区别

对于对象来说：

* 存储映射关系只能用字符串（ES6新增了Symbol）作为属性名（key）。
* 某些情况下我们可能希望通过其他类型作为key，比如对象，这个时候会自动将对象转成字符串来作为key。

遇到这种情况，我们可以使用 `Map` 。

### Map的常见属性和方法

#### 常见属性

* `size` ： 返回 `Map` 中元素的个数。

#### 常见方法

* `set(key,value)` ：在 `Map` 中添加 `key` 、 `value` ，并且返回整个 `Map` 对象。
* `get(key)` ：根据 `key` 获取 `Map` 对象中的 `value` 。
* `has(key)` ：判断是否包括某一个 `key` ，返回 `boolean` 类型。
* `delete(key)` ：根据 `key` 删除一个键值对，返回 `boolean` 类型。
* `clear()` ：清空所有元素
* `forEach(callback,[,thisArg])` ：通过 `forEach` 遍历 `Map` 。
* `Map` 也支持通过 `for of` 遍历。

## WeakMap

和 `Map` 类型相似的另外一个数据结构称之为 `WeakMap` ，也是以键值对的形式存在的。

### WeakMap和Map的区别

1. `WeakMap` 的 `key` 只能使用对象，不接受其他的类型作为 `key` 。
2. `WeakMap` 的 `key` 对对象的引用是弱引用，如果没有其他引用引用这个对象，那么 `GC` 可以回收该对象。
3. `WeakMap` 不能遍历，因为没有 `forEach` 方法，也不支持通过 `for of` 的方式进行遍历。

### WeakMap的常见方法

* `set(key, value)` ：在 `Map` 中添加 `key` 、`value` ，并且返回整个 `Map` 对象。
* `get(key)` ：根据 `key` 获取 `Map` 中的 `value` 。
* `has(key)` ：判断是否包括某一个 `key` ，返回 `boolean` 类型。
* `delete(key)` ：根据 `key` 删除一个键值对，返回 `boolean` 类型。

### WeakMap的应用场景

在 `Vue` 的响应式原理中，其实就使用了 `WeakMap`。

看下面这个例子：

```js
const obj1 = {
  name: 'curry',
  age: 30,
}

function objNameFn1() {
  console.log('objNameFn1被执行了')
}

function objNameFn2() {
  console.log('objNameFn2被执行了')
}

function obj1AgeFn1() {
  console.log('objAgeFn1被执行了')
}

function obj1AgeFn2() {
  console.log('objAgeFn2被执行了')
}

const weakMap = new WeakMap()

const obj1Map = new Map()
obj1Map.set('name', [objNameFn1, objNameFn2])
obj1Map.set('age', [obj1AgeFn1, obj1AgeFn2])

weakMap.set(obj,obj1Map)

// obj1.name 发生了改变
// vue 通过 proxy 和 Object.defineProperty 监听变化 

// 通过 WeakMap 实现响应式变化

// 获取到要变化的那个对象（obj1）中的所有属性
const targetMap = weakMap.get(obj1)

// 根据要变化的那个属性 获取到所有改变这个属性值的方法
const nameFns = targetMap.get("name") 

// 执行方法对属性进行响应式更改
nameFns.forEach(item => item())
```

在上面这个例子中，`WeakMap` 做的事情就是保存 `obj1` 这个对象的引用，当通过 `proxy` 和 `Object.defineProperty` 监听到属性的变化，就将该对象中的属性响应式的方法获取并对其调用。

## 总结

以上就是ES6中的一些知识点，其中还有不少知识点，例如：`Promise` 、 `迭代器` 等知识我们将会在另外的章节对其进行总结。

ES6是JavaScript的一个更新相当大的版本，知识点较多，ES6之后的版本相对来说会少很多东西。
