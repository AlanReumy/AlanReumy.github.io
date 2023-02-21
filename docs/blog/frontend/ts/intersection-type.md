# 交叉类型

在 `TS` 中，我们可以使用交叉运算符 & 表示交叉类型，例如：

```ts
type A = string & number;
type B = "A" & "B";
```

## 交叉运算符的特性

交叉运算符具有如下特性：

1. 唯一性：A & A => A
2. 交换律：A & B => B
3. 结合律：A & (B & C) => (A & B) & C
4. 父类型收敛：若 B 是 A 的父类型，则 A & B => A

例子如下：

```ts
type A1 = 1 & number; // 1
type A2 = "A" & string; // 'A'
type A3 = true & boolean; // true

type B1 = any & string; // any
type B2 = any & number; // any
type B3 = any & never; // never
```

## 几个应用场景

### 接口对象属性不同

如果在一个 `interface` 中，包含相同的属性，但是属性类型却不同时，结果会是怎样的呢？

```ts
interface Student {
  name: string;
  age: string;
}

interface Teacher {
  name: string;
  age: number;
}

type A = Student & Teacher; // => { name: string; age: never }
```

上面代码最终结果的原因是，`age` 属性经过交叉运算符运算后，得到的是 `string & number`，而这样的类型是不存在的，因此返回 `never` 类型

### 字面量类型或字面量类型组成的联合类型

如果存在字面量类型或字面量类型组成的联合类型，经过交叉运算符计算后，返回的将会是 `never` 类型

例子如下：

```ts
interface A {
  name: "curry";
}

interface B {
  name: "james";
}

type AB = A & B; // => { name: never }
```

### 函数类型的交叉运算

除了对象类型可以交叉运算外，函数类型也可以进行交叉运算

例子如下：

```ts
type F1 = (a: number, b: number) => void;
type F2 = (a: string, b: string) => void;

const fn: F1 & F2 = (a: number | string, b: number | string) => {};
fn(1, 2); // √
fn("A", "B"); // √
fn("c", 1); // x
```

`TS` 会利用函数重载的特性来实现不同函数类型的交叉运算

要解决上述错误，我们可以新增一个类型：

```ts
type F3 = (a: string, b: number) => void;
const fn: F1 & F2 & F3 = (a: number | string, b: number | string) => {};
fn("c", 1); // √
```
