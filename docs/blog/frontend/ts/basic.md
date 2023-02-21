# TypeScript 入门

## 为什么会有 TypeScript

随着近几年前端领域的快速发展，让 JavaScript 迅速被普及和受广大开发者的喜爱，借助于 JavaScript 本身的强大，也让使用 JavaScript 开发的人员越来越多。

但是 JavaScript 一个非常大的缺点：**没有类型检测**

## 类型缺失带来的问题

编程开发中我们有一个共识：**错误出现的越早越好**

JavaScript 并不能在代码编译阶段发现代码的错误，如果此时我们给 JavaScript 加上限制，在开发阶段就能避免很多问题。

## JavaScript 添加类型约束

为了弥补 JavaScript 类型约束上的缺陷，增加类型约束，很多公司推出了自己的方案：

- 2014 年，Facebook 推出了 flow 来对 JavaScript 进行类型检查
- 同年，Microsoft 微软也推出了 TypeScript1.0 版本

而现在，无疑 TypeScript 已经完全胜出：

## 什么是 TypeScript

将官方文档的说法翻译过来是这样的：

> TypeScript 是拥有类型的 JavaScript 超集，它可以编译成普通、干净、完整的 JavaScript 代码

我们可以将 TypeScript 理解成加强版的 JavaScript：

- JavaScript 所拥有的特性，TypeScript 全部都是支持的，并且它紧随 ECMAScript 的标准，所以 ES6、ES7、ES8 等新语法标准，它都是支持的；

- 在语言层面上，不仅仅增加了类型约束，而且包括一些语法的扩展，比如枚举类型（Enum）、元组类型（Tuple）等

- TypeScript 最终会被编译成 JavaScript 代码，所以你并不需要担心它的兼容性问题，在编译时也不需要借助于 Babel 这样的工具

## TypeScript 的编译环境

TypeScript 最终会被编译成 JavaScript 来运行，因此我们需要搭建对应的环境。

通过安装 TypeScript 可以通过 TypeScript 的 Compiler 将其编译为 JavaScript

> npm install typescript

## TypeScript 的运行环境

如果每次修改 ts 代码后都需要经过 tsc 编译，然后通过 node 或者浏览器来运行，未免显得有些繁琐。

有以下两种解决方法：

1. 通过 webpack，配置本地的 TypeScript 编译环境和本地服务，直接运行在浏览器上

   ```js
   // webpack.config.js
   const path = require("path");
   const HtmlWebpackPlugin = require("html-webpack-plugin");
   module.exports = {
     mode: "development",
     entry: "./src/main.ts",
     output: {
       path: path.resolve(__dirname, "./dist"),
       filename: "bundle.js",
     },
     resolve: {
       extensions: [".ts", ".js"],
     },
     module: {
       rules: [
         {
           test: /\.ts$/,
           loader: "ts-loader",
         },
       ],
     },
     plugins: [
       new HtmlWebpackPlugin({
         template: "./index.html",
       }),
     ],
   };
   ```

2. 通过 ts-node 库，为 TypeScript 的运行提供执行环境
   安装：

   > npm install ts-node -g
   > npm install tslib @types/node -g

   通过 ts-node 运行：

   > ts-node ts 文件

## 变量

### 变量的声明

在 TypeScript 中定义变量需要指定标识符的类型

声明了类型后 TypeScript 就会进行类型检测，声明的类型可以称之为类型注解

```ts
var/let/const 标识符: 数据类型 = 赋值
```

例子：

```ts
const username: string = "123";
const message: String = "123";
```

注意：

> string: 表示 ts 中的字符串类型
> String: 表示 js 中的字符串包装类 严格来说 ts 中不应该这样写类型注解

### 变量的类型推断（推导）

在开发中，有时候为了方便起见我们并不会在声明每一个变量时都写上对应的数据类型，我们更希望可以通过 TypeScript 本身的特性帮助我们推断出对应的变量类型

```ts
let message = "hello";
message = 1; // error
```

## 数据类型

### number 类型

和 JavaScript 的 number 类型几乎相同

```ts
let num1: number = 10;
// 二级制
let num2: number = 0b1;
// 八进制
let num3: number = 0o7;
// 十六进制
let num4: number = 0x15;
// 小数
let num5: number = 10.0;
```

### boolean 类型

```ts
let flag: boolean = true;
flag = false;
```

### string 类型

```ts
const message: string = "hello";
```

### 数组类型

数组的定义方式有两种:

```ts
const names: string[] = [];
const message: Array<string> = []; // 不推荐 react jsx 中有冲突
```

### object 类型

object 对象类型可以用于描述一个对象

但是我们不能从里面获取数据，也不能设置数据

```ts
const info: object = {
  name: "curry",
  age: 34,
};
console.log(info.age); // error
```

### symbol 类型

可以通过 symbol 来定义相同的名称，Symbol 函数返回的是不同的值，这样可以实现在对象中添加相同属性名称的操作。

```ts
const title1 = Symbol("title");
const title2 = Symbol("title");
const info = {
  [title1]: "coder",
  [title2]: "teacher",
};
```

### null 和 undefined

在 TypeScript 中，null 和 undefined 各自的类型也是 undefined 和 null，也就意味着它们既是实际的值，也是自己的类型

```ts
let n: null = null;
let u: undefined = undefined;
```

### any 类型

在某些情况下，我们确实无法确定一个变量的类型，并且可能它会发生一些变化，这个时候我们可以使用 any 类型

- 可以对 any 类型的变量进行任何的操作，包括获取不存在的属性、方法；

- 可以给一个 any 类型的变量赋值任何的值

```ts
let msg: any = "hello world";
msg = 123;
msg = false;
msg = {};
```

### unknown 类型

unknown 类型用于描述类型不确定的变量

unknown 与 any 的不同：

- any 类型可以赋值给任意类型
- unknown 类型只能赋值给 any 类型和 unknown 类型

```ts
function foo() {
  return 123;
}

function bar() {
  return "hello";
}

let result: unknown;
let flag: boolean = true;
if (flag) {
  result = foo();
} else {
  result = bar();
}
```

### void 类型

void 通常用来指定一个函数是没有返回值的，那么它的返回值就是 void 类型

- 可以将 null 和 undefined 赋值给 void 类型，也就是函数可以返回 null 或者 undefined

  ```ts
  function sum(a: number, b: number) {
    console.log(a + b);
  }
  ```

- 如果一个函数没有写任何类型，那么它默认返回值的类型就是 void 的，也可以显示的来指定返回值是 void：

  ```ts
  function sum(a: number, b: number): void {
    console.log(a + b);
  }
  ```

### never 类型

never 表示永远不会发生值的类型

应用场景：

- 一个函数中是一个死循环或者抛出一个异常，此时的函数返回值就不应该是 void 类型，而应该是 never 类型

  ```ts
  // 死循环
  function foo(): never {
    while (true) {}
  }

  // 抛出异常
  function bar(): never {
    throw new Error();
  }
  ```

- 假定 message 只能使用 number 和 string 另一个人想将布尔值传进来 他仅仅在形参中加入了联合类型，此时通过一个 never 类型的变量 check 则会出错 因此这样提高了代码的可维护性

  ```ts
  function handleMessage(message: number | string | boolean) {
    switch (typeof message) {
      case "string":
        console.log("789");
        break;
      case "number":
        console.log("456");
        break;
      case "boolean":
        console.log("123");
        break;
      default:
        const check: never = message;
    }
  }

  handleMessage("123");
  ```

### tuple 类型

tuple 元组类型表示多种元素的组合

tuple 和 数组的区别：

- 数组中通常建议存放相同类型的元素，不同类型的元素是不推荐放在数组中。（可以放在对象或者元组中）

- 元组中每个元素都有自己特性的类型，根据索引值获取到的值可以确定对应的类型

```ts
// 数组的弊端
// const info:any[] = ['curry',33,'1.92']

// 元组
const info: [string, number, string] = ["curry", 33, "1.92"];
```

函数将元组作为返回值，在使用的时候会更加方便

简单实现 react 的 useState ：

```ts
function useState<T>(state: T) {
  let currentState = state;
  const changeState = (newState: T) => {
    currentState = newState;
  };

  const tuple: [T, (newState: T) => void] = [currentState, changeState];
  return tuple;
}
```

### 联合类型

联合类型是由两个或者多个其他类型组成的类型，表示可以是这些类型中的任何一个值

联合类型中的每一个类型被称之为联合成员（union's members）

```ts
// number | string 联合类型
function printId(id: number | string) {
  if (typeof id === "string") {
    console.log(id.toUpperCase());
  } else {
    console.log(id);
  }
}

printId(1);
printId("1");
```

### 类型别名

通过 type 关键字，可以给对象类型起一个别名

```ts
// type 用于定义类型别名
type idType = string | number | boolean;
type pointType = {
  x: number;
  y: number;
  z?: number;
};

function printId(id: idType) {
  console.log(id);
}

function printPoint(point: pointType) {
  console.log(point);
}

printId("123");
printId(456);

printPoint({ x: 1, y: 2 });
printPoint({ x: 1, y: 2, z: 3 });
```

### 类型断言

有时候 TypeScript 无法获取具体的类型信息，这个我们需要使用类型断言（Type Assertions）

```ts
const el = document.getElementById("app") as HTMLImageElement;

el.src = "http://twitzz.cn";

class Person {}

class Student extends Person {
  studying() {}
}

function sayHello(p: Person) {
  (p as Student).studying();
}
const s = new Student();
sayHello(s);
```

### 非空类型断言

非空断言使用的是 ! ，表示可以确定某个标识符是有值的，跳过 ts 在编译阶段对它的检测

```ts
function printMessage(message?: string) {
  // 非空类型断言
  // 忽略null和undefined的情况
  console.log(message!.length);
}
```

### 可选链

可选链并不是 TypeScript 独有的特性，它是 ES11（ES2020）中增加的特性

它的作用是当对象的属性不存在时，会短路，直接返回 undefined，如果存在，那么才会继续执行

```ts
type Person = {
  name: string;
  friend?: {
    name: string;
    age?: number;
  };
};

const info: Person = {
  name: "curry",
  friend: {
    name: "green",
  },
};

// 可选链
console.log(info.friend?.age);
```

### !!运算符

!!运算符的作用是隐式类型转换为 boolean 类型

```ts
const message = "hello";
const flag = !!message;
```

### ??运算符

??运算符又叫空值合并操作符，当操作符的左侧是 null 或者 undefined 时，返回其右侧操作数，否则返回左侧操作数

```ts
// let message: string | null = null;
let message: string | null = "world";

const content = message ?? "hello";
// 类似于
// const content = message ? message : "hello";
```

### 字面量类型

```ts
// 类型是123
const num = 123;
```

字面量类型的意义就是结合联合类型来使用

```ts
type Alignment = "left" | "right" | "center";
let align: Alignment;
align = "center";
```

### 字面量推理

使用字面量推理意味着：

- 该表达式中的字面类型不应被扩展（例如：不能从“hello”转换为字符串）

- 对象字面量获取只读属性

- 数组字面量成为只读元组

```ts
type Method = "GET" | "POST";
function request(url: string, method: Method) {}

type Request = {
  url: string;
  method: Method;
};

// 字面量推理
const option = {
  url: "www.baidu.com",
  method: "POST",
} as const;

const option = {
  url: "www.baidu.com",
  method: "POST",
}; // error option.method 的类型是 string类型

request(option.url, option.method);
```

### 类型保护（type guard）

常见的类型保护有以下几种：

- typeof

  ```ts
  type IDType = number | string;
  function printId(id: IDType) {
    console.log(id);
    // typeof
    if (typeof id === "string") {
      console.log(id.toLowerCase());
    } else {
      console.log(id);
    }
  }
  ```

- if switch

  ```ts
  type Direction = "left" | "center" | "right";

  function printDircetion(direction: Direction) {
    console.log(direction);
    // if
    if (direction === "center") {
      console.log(direction);
    }

    // switch
    switch (direction) {
      case "center":
        break;

      default:
        break;
    }
  }
  ```

- instanceof

  ```ts
  function pritnTime(time: string | Date) {
    if (time instanceof Date) {
      console.log(time.getTime());
    } else {
      console.log(time);
    }
  }
  ```

- in

  ```ts
  type Fish = {
    swimming: () => void;
  };

  type Dog = {
    running: () => void;
  };

  function walk(animal: Fish | Dog) {
    // 判断是否有该属性
    if ("swimming" in animal) {
      animal.swimming();
    }
  }
  ```

## 函数相关

### 函数的参数和返回值

TypeScript 允许我们指定函数的参数和返回值的类型

```ts
// 在开发中，可以不写返回值类型，它会有类型推断
function sum(num1: number, num2: number): number {
  return num1 + num2;
}
```

### 匿名函数的参数

当一个函数出现在 TypeScript 可以确定该函数会被如何调用的地方时，该函数的参数会自动指定类型

```ts
const names = ["abc", "cba"];

// item的类型是根据上下文环境推导出来的，这个时候可以不添加类型注解
// 上下文的函数可以不添加类型注解
names.forEach((item) => {});
```

### 接受的参数为对象类型

如果希望限定一个函数接受的参数是一个对象，可以使用一个对象来作为类型

```ts
// point:x,y => 对象类型
function printPoint(point: { x: number; y: number }) {}

printPoint({ x: 123, y: 456 });
```

### 可选类型

对象类型也可以指定哪些属性是可选的，可以在属性的后面添加一个?

```ts
// point:x,y,z => 对象类型
// 问号?表示可选类型
function printPoint(point: { x: number; y: number; z?: number }) {}

printPoint({ x: 123, y: 456 });
printPoint({ x: 123, y: 456, z: 190 });
```
