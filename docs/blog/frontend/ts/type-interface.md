# type 与 interface

`type` 被称为类型别名，可以为一个类型取个新名字

`TS` 中的工具类型，例如：`Partial`、`Required`、`Pick` 等类型，都是通过 `type` 关键字来定义的

`interface` 被称为接口，只能用于定义对象类型

## 相同点

1. 类型别名和接口都可以用来描述对象或函数

```ts
type Point = {
  x: number;
  y: number;
};

type SetPoint = (x: number, y: number) => void;
```

```ts
interface Point {
	x: number;
	y: number;
}

interface SetPoint {
	(x: number, y: number) => void;
}
```

2. 类型别名和接口都支持扩展

类型别名可以用 **交叉类型** 来扩展

```ts
type Animal = {
  name: string;
};

type Cat = Animal & {
  miao: boolean;
};
```

接口可以用 `extends` 关键字来扩展

```ts
interface Animal {
  name: string;
}

interface Cat extends Animal {
  miao: boolean;
}
```

> 此外，接口也可以通过 `extends` 来扩展类型别名，类型别名也可以通过 **交叉类型** 来扩展接口

## 不同点

1. 类型别名可以为基本类型、联合类型或元组类型定义别名，而接口不会

```ts
type MyString = string;
type StringOrNumber = string | number;
type Point = [number, string];
```

2. 同名接口会自动合并，而类型别名不会

```ts
interface User {
  name: string;
}

interface User {
  age: number;
}
```

## 应用场景

- 使用类型别名的场景：
  - 定义基本类型的别名
  - 定义元组类型
  - 定义函数类型
  - 定义联合类型
  - 定义映射类型
- 使用接口的场景：
  - 需要利用接口的自动合并特性
  - 定义对象类型而无需使用类型别名的时候
