# typeof 操作符

在 `JS` 中，我们可以使用 `typeof` 关键字得到一个变量的类型

`TS` 也有 `typeof` 关键字

## 应用场景

1. 有时候我们想定义一个对象的类型，但是此对象的类型嵌套太多层级，很难定义，我们可以使用 `typeof` 关键字，减少类型的重复定义

```ts
const user = {
	name: 'curry'
	age: 34,
	address: {
		country: 'US',
		city: 'golden state'
	}
}

type Address = typeof user[address]
type User = typeof user
```

2. 获取 函数 或者 `class` 类型，再利用内置的工具类型做事情

函数类型的处理：

```ts
function add(x: number, y: number) {
  return x + y;
}

type AddFnType = ReturnType<typeof add>;
type AddFnTParamype = Parameters<typeof add>;
```

`class` 的处理：

```ts
class Point {
  x: number;
  y: number;
  constructor(x: number, y: number) {
    this.x = x;
    this.y = y;
  }
}

function createPoint(Contructor: typeof Point, x: number, y: number) {
  return new Contructor(x, y);
}
```

3. 借助 `const` 断言，获取更加精准的类型

```ts
let requestMethod1 = "Get";
let requestMethod2 = "Get" as const;
type R0 = typeof requestMethod1; // string
type R1 = typeof requestMethod2; // 'Get'

let user1 = {
  id: 30,
  name: "curry",
};

let user2 = {
  id: 30,
  name: "curry",
} as const;

type U1 = typeof user1;
type U2 = typeof user2;
```
