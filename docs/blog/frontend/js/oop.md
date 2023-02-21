# 面向对象

## JavaScript中的面向对象

对象是JavaScript中一个非常重要的概念，用对象来描述事物，更有利于我们将现实的事物抽象成代码中的数据结构。

早期使用创建对象的方式最多的是使用Object类，并且使用new关键字来创建一个对象：

```js
var obj = new Object()
obj.name = 'curry'
obj.age = 33
```

后来很多开发者为了方便起见，都是直接通过字面量的形式来创建对象：

```js
var obj = {
  name:'curry',
  age:33
}
```

## 对象中属性

在前面的两个例子中，我们使用了两种方式创建了对象，并且将属性直接定义或添加在对象内部，但其实我们还有另外的方式添加属性，并且为属性添加一些特性，这就是`Object.defineProperty()`。这个方法可以对属性进行添加和修改。

### Object.defineProperty()

`Object.defineProperty(obj,prop,descriptor)`方法会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性，并返回此对象。

它可以接受三个参数：

* obj：要定义属性的对象
* prop：要定义或修改的对象的属性名称或Symbol
* descriptor：要定义或修改的描述符

它的返回值是：

* 被传递给函数的对象

## 属性描述符

属性描述符分为两种：

1. 数据属性描述符
2. 存取属性描述符

|                | configurable | enumerable | value  | writable | get    | set    |
| -------------- | ------------ | ---------- | ------ | -------- | ------ | ------ |
| 数据属性描述符 | 可以         | 可以       | 可以   | 可以     | 不可以 | 不可以 |
| 存取属性描述符 | 可以         | 可以       | 不可以 | 不可以   | 可以   | 可以   |

上面这个表格表示：`get`与`set`不能与`value`和`writable`**同时出现**。

### 数据属性描述符

该描述符有如下四个特性：

#### configurable

表示属性是否可以通过`delete`删除属性，是否可以修改它的特性，或者是否可以将它修改为存取属性描述符。

* 当我们直接在一个对象上定义某个属性时，这个属性的`configurable`为`true`。

* 当我们通过属性描述符定义一个属性时，这个属性的`configurable`默认为`false`。

例子：

```js
var obj = {
  name:'curry'
}

Object.defineProperty(obj,'name',{
  configurable:false // 不可配置
})
delete obj.name

console.log(obj) // { name: 'curry' }
```

#### enumerable

表示属性是否可以通过`for-in`或者`Object.keys()`返回该属性。

* 当我们直接在一个对象上定义某个属性时，这个属性的`enumerable`为`true`。

* 当我们通过属性描述符定义一个属性时，这个属性的`enumerable`默认为`false`。

例子：

```js
var obj = {
  name:'curry'
}
console.log(Object.keys(obj)) ['name']

Object.defineProperty(obj,'name',{
  enumerable:false // 不可枚举
})
console.log(Object.keys(obj)) // []
```

#### writable

表示是否可以修改属性的值。

* 当我们直接在一个对象上定义某个属性时，这个属性的`writable`为`true`。

* 当我们通过属性描述符定义一个属性时，这个属性的`writable`默认为`false`。

例子：

```js
var obj = {
  name:'curry'
}
Object.defineProperty(obj,'name',{
  writable:false // 不可修改
})
obj.name = 'james'

console.log(obj) // { name: 'curry' }
```

#### value

表示属性的值，读取属性时会返回该值，修改属性时，会对其进行修改。

* 默认情况下这个值是`undefined`。

例子：

```js
var obj = {
  name: 'curry',
}
Object.defineProperty(obj, 'name', {
  value: 'james', // 属性的值
})
console.log(obj) // { name: 'james' }
```

### 存储属性描述符

存储属性描述符中的`configurable`和`writable`与数据属性描述符相同，这里不再说明。

#### get

获取属性时会执行的函数。默认为`undefined`

例子：

```js
var obj = {
  name: 'curry',
}
Object.defineProperty(obj, 'name', {
  get(){
    return 'abc'
  }
})

console.log(obj.name) // abc
```

#### set

修改属性时会执行的函数。默认为`undefined`

例子：

```js
var obj = {
  name: 'curry',
}
var age = 32
Object.defineProperty(obj, 'age', {
  get(){
    return age
  },
  set(newValue){
    age = newValue
  }
})
obj.age = 33
console.log(obj.age)
```

### 同时定义多个属性

`Object.defineProperties()`方法直接在一个对象上定义 **多个** 新的属性或修改现有属性，并且返回该对象。

例子：

```js
var obj = {
  name: 'curry',
}
Object.defineProperties(obj, {
  name:{
    value:'james'
  },
  age:{
    enumerable:false
  }
})
console.log(Object.keys(obj)) // ['name']
```

## 创建对象的几种方案

在本文开始我们提到两种创建对象的方式，但其实还有很多种创建对象的方式，比如：**工厂模式**。我们来看看工厂模式创建对象的方法。

### 工厂模式

```js
function createPerson(name,age){
  var p = new Object()
  p.name = name
  p.age = age
  return p
}

var p1 = createPerson('curry',33)
var p2 = createPerson('james',37)
console.log(p1,p2) // { name: 'curry', age :33 } { name: "james", age:37 }
```

通过工厂模式创建对象有一个很大的**弊端**：它们创建的对象都是`Object`类型的。

因此接下来我们来说另外一种方式：构造函数的方式。

### 构造函数

来看看构造函数的方式是如何创建对象的。

```js
function Player(name,age){
  this.name = name
  this.age = age
}

var p1 = new Player('curry',33)
```

与其他编程语言有些不同，JavaScript中的构造函数和其他普普通通的函数**相同**。

但是，一个函数**被`new`关键字调用**，它就是一个构造函数。

**`new`关键字调用到底发生了什么？**为什么使用它调用后，一个普通的函数就变为了构造函数？这不禁是我们想要问的问题。

### new 关键字

如果一个函数被new关键字调用了，它就会执行如下操作：

1. 在内存中**创建一个新对象**
2. 这个新对象的内部 [[Prototype]] 特性被赋值为**构造函数的prototype属性**：

```js
p1.__proto__ = Player.prototype
```

3. **构造函数内部的this**被赋值给新的对象 也就是新对象和函数调用的this会绑定起来

```js
Player.call(p1,'curry');
```

4. 执行构造函数内部的代码 即给新对象添加属性和方法

```js
p1.name;
console.log(p1);
```

5. 如果构造函数返回非空对象，则返回该对象；否则，**返回刚创建的新对象**

这样，这个构造函数可以确保我们创建的对象是属于使用构造函数创建的那个类型的对象，而不再仅仅像工厂模式一样都是`Object`类型的对象。

## 认识对象的原型

在JavaScript中，每个对象都有一个内置的特殊属性`[[prototype]]`，这个特殊的属性指向另外一个对象。

### 原型对象

这个对象就是我们常说的**原型对象**。那么它有什么用呢？

* 当我们通过引用对象的属性key来获取一个value时，它会触发 [[Get]]的操作。
* 这个操作会首先检查该属性是否有对应的属性，如果有的话就使用它。
* 如果对象中没有改属性，那么会访问对象的[[prototype]]属性指向的那个对象上的属性。

了解了它的作用后，我们又想问：如何获取到这个对象？

获取的方式有两种：

1. 通过对象的` __proto__ `属性可以获取到，（早期浏览器添加的，存在一定兼容性问题）。
2. 通过`Object.getPrototypeOf()`方法可以获取到。

> 注意：prototype属性时函数独有的

```js
var obj = {}
console.log(obj.prototype) // undefined
```

### 理解创建对象时的内存表现

我们前面在说`new`关键字的时候说到过，其中一个步骤就是跟原型相关

> 这个新对象的内部`[[Prototype]]` 特性被赋值为**构造函数的`prototype`属性**：
>
> ```js
> p1.__proto__ = Player.prototype
> ```
>

它在内存中的表现大概是这样的（我们稍后来说`constructor`）：

![image-20211203195910947](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20211203195910947.png)

这意味着我们通过`Player`构造函数创建出来的所有对象的`[[prototype]]`属性都指向`Plyaer.prototype`。

### constructor属性

默认情况下原型上都会添加一个属性叫做`constructor`，这个`constructor`指回构造函数。

当我们需要在原型对象上新增多个属性时，一般会重写原型对象：

```js
function Player(){
}

Player.prototype = {
  name:'curry',
  eating(){
  },
    ...
}
```

此时，`Player.prototype`指向了一个普通对象，它上面将不会再有`constructor`属性了。

我们也可以手动将`constructor`**指回构造函数**：

```js
Player.prototype = {
  constructor:Player
  ...
}
```

这种方式虽然可行，但是会造成`constructor`变为可枚举属性。（默认情况下，`constructor`属性不可枚举）。

这个时候，就可以使用我们之前提到过的`Object.defineProperty()`

```js
Object.defineProperty(Player,prototype,'constructor',{
  enumerable:false,
  value:Player
})
```

### 构造函数和原型组合创建对象

有了原型和构造函数后，我们可以将一些重复出现的方法放在原型对象上供所有实例对象上使用。

```js
function Player (name) {
  this.name = name
}

Player.prototype.running = function() {
  console.log(this.name + ' running')
}

Player.prototype.eating = function() {
  console.log(this.name + ' eating')
}

var p1 = new Player('curry')
var p2 = new Player('james')
p1.running() // curry running
p2.eating() // james eating
```

## 继承

面向对象有三大特性：封装、继承、多态。我们着重聊聊继承。

**继承的作用：可以将重复的代码逻辑抽取到父类中使用，子类可以通过继承父类来使用这些代码**。

在聊继承前，我们需要明白JavaScript中的**原型链**。

### 原型链

当我们获取对象的属性的时候，如果没有从当前的属性中查找到属性，就会从它的原型对象上查找，直到找到`Object.prototype`。

为什么会找到`Object.prototype`就不往上找了呢？这是因为`Object.prototype`中的`__proto__`指向`null`，这就代表它是**顶层对象**了。

在内存中表现具体如下：

![image-20211203205004143](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20211203205004143.png)

```js
p1.toString()
```

如果我们需要调用`p1`的`toString()`方法，就会从当前对象`p1`查找这个方法，然后通过`__proto__`属性去它的原型对象上找，直到找到该属性，没有找到则会找到`Object.prototype`这个对象结束。

**这就是原型链！**

### 原型链实现继承

了解了原型链后，我们可以来说一说JavaScript中的继承了，我们先来说第一种继承方式：**通过原型链实现继承**。

```js
function Person() {
  this.friends = []
}

Person.prototype.running = function () {
  console.log(this.name + ' running')
}

function Student(name, age) {
  this.name = name
  this.age = age
}

var p = new Person()
Student.prototype = p

var s1 = new Student('curry',33)
var s2 = new Student('james',37)

s1.friends.push('kobe')
s2.friends.push('green')

s1.running() // curry running
console.log(s1) // { name: 'curry', age: 33 }
console.log(s1.friends) // [ 'kobe', 'green' ]
console.log(s2.friends) // [ 'kobe', 'green' ]
```

它的内存表现如下：

![image-20211203210808461](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20211203210808461.png)

我们从上面这个例子可以看出原型链继承存在几个弊端：

1. 我们通过直接打印对象是看不到原型对象的属性。
2. 属性会被多个对象共享，如果这个对象是一个引用类型，那么就会造成多个属性操作同一个引用。

### 借用构造函数继承

为了解决原型链继承中存在的问题，开发人员提供了一种新的技术: **借用构造函数（constructor stealing）**，也称之为组合继承。

这种继承方式很简单：在子类型构造函数中的内部**调用了父类的构造函数**。

```js
function Person(friends) {
  this.friends = friends
}

Person.prototype.running = function () {
  console.log(this.name + ' running')
}

function Student(name, age,friends) {
  Person.call(this,friends)
  this.name = name
  this.age = age
}

// 该对象上会多出来friends属性，并且它的值是undefined
var p = new Person()
Student.prototype = p

var s1 = new Student('curry',33,['ad','wade'])
var s2 = new Student('james',37,['bosh','howard'])
s1.friends.push('kobe')
s2.friends.push('green')

s1.running() // curry running
console.log(s1) // { friends: [ 'ad', 'wade', 'kobe' ], name: 'curry', age: 33 }
console.log(s1.friends) // [ 'ad', 'wade', 'kobe' ]
console.log(s2.friends) // [ 'bosh', 'howard', 'green' ]
```

它的内存表现如下：

![image-20211203212529228](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20211203212529228.png)

借用构造函数仍然存在弊端：

1. `Person`函数被调用了两次。
2. `s1`的原型对象会多出来一些属性（比如`friends`属性），但是这些属性没有存在的必要。

### 寄生式继承

在说寄生式继承前，我们先来看看原型式继承函数。

这种模式要从道格拉斯·克罗克福德（Douglas Crockford，著名的前端大师，JSON的创立者）在2006年写的一篇文章说起: Prototypal Inheritance in JavaScript(在JS中使用原型式继承)。

```js
// 原型式继承函数
function createObject(o){
  function F(){}
  // 将它的原型对象指向改变
  F.prototype = o
  var newObj = new F()
  // 返回的对象(newObj)可以通过__proto__属性访问到对象o
  return newObj
}
```

> ES6提供了`Object.create()`方法来实现原型式继承函数的作用

寄生式继承就是将原型式继承函数和工厂模式结合起来的一种继承方式。

它的原理是：创建一个封装继承过程的函数, 该函数在内部以某种方式来增强对象，最后再将这个对象返回。

```js
function createStudent(person){
  var stu = Object.create(person)
  stu.studying = function(){
    console.log("studying")
  }
}
```

因为它使用了工厂模式，所以仍然是存在弊端工厂模式的弊端的。

### 最终解决方案-寄生组合式继承

其实从名字就可以看出，这是将寄生式继承和组合式继承（借用构造函数）结合起来的一种继承方式。

我们提到过：组合式继承（借用构造函数）存在两个问题：1. 父类构造函数被调用两次；2. 实例化的子类的原型对象是会多出来一些无用的属性。

然而通过这种最终解决方案，这些问题都能被解决。直接看代码：

```js
// 继承的工具函数 subType-子类，SuperType-父类
function inheritPrototype(SubType, SuperType) {
  SubType.prototype = Object.create(SuperType.prototype)
  // contructor指回原来的构造函数
  Object.defineProperty(SubType.prototype, "constructor", {
    enumerable: false,
    configurable: true,
    writable: true,
    value: SubType
  })
}

// 父类Person
function Person(name, age, friends) {
  this.name = name
  this.age = age
  this.friends = friends
}

// 给原型上添加方法
Person.prototype.running = function() {
  console.log("running~")
}

// 给原型上添加方法
Person.prototype.eating = function() {
  console.log("eating~")
}

// 子类Student
function Student(name, age, friends, sno, score) {
  // 借用父类的构造函数，但是只调用了一次父类的构造函数，解决了组合继承的问题
  Person.call(this, name, age, friends)
  this.sno = sno
  this.score = score
}

// 给原型上添加方法
Student.prototype.studying = function() {
  console.log("studying~")
}

// Student继承Person
inheritPrototype(Student, Person)

// 实例化子类
var stu = new Student("why", 18, ["kobe"], 111, 100)

stu.studying() // studying~
stu.running() // running~
stu.eating() // eating~

console.log(stu.constructor.name) // Student
```

## **proto**、prototype、constructor

说完继承后，我们最后来完全弄懂这三个属性。

![](https://codertzm.oss-cn-chengdu.aliyuncs.com/20190311194017886.png)

直接看这张图很容易晕，我们一步步来分解。

### **proto**

我们来看看`__ proto __`属性，先只看上图红色的那条线，并且需要牢记两点：

1. `__proto__`和`constructor`属性是**对象**所独有的。
2. `prototype`属性是**函数**所独有的。但是由于JS中函数也是一种对象，所以函数也拥有`__proto__`和`constructor`属性。

### prototype

接着我们来看看`prototype`属性（绿色的那条线），它是**函数所独有的**，它是从**一个函数指向一个对象**。它的含义是**函数的原型对象**，也就是这个函数（其实所有函数都可以作为构造函数）所创建的实例的原型对象。**任何函数在创建的时候，其实会默认同时创建该函数的prototype对象。**

### constructor

最后我们再来看`constructor`属性，它也是**对象才拥有的**，它是从**一个对象指向一个函数**，含义就是**指向该对象的构造函数**。

可以看出**Function**这个对象比较特殊，**它的构造函数就是它自己**（因为Function可以看成是一个函数，函数也是一个对象），所有函数和对象最终都是由Function构造函数得来，所以`constructor`属性的终点就是**Function**这个函数。

## 总结

本文涉及的知识点较多，关于ES6中的`class`关键字我们这里不提及是因为它不过是一种语法糖而已。

同时只有真正掌握了JavaScript中的原型和面向对象的概念才能真正理解，JavaScript中的继承是如何来实现的。
