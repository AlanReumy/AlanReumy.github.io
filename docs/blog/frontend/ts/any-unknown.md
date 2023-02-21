# any 与 unknown

在 `TS` 中，有时候我们可能会出现不得不使用 `any` 类型来解决的情况

因为 `any` 类型是通用父类型，可以将任意类型赋值给它，但是如果多处使用了 `any` 类型，将失去使用 `TS` 的类型系统的意义。

`TS` 在 3.0 的版本中新增了 `unknown` 类型，来更加安全的处理不知道的类型。

我们可以将 `unknown` 类型理解为：不知道的类型，将 `any` 类型理解为：任意类型

## 应用场景

假如我们有这样一个函数：

```ts
const fn = (cb: any) => {
  try {
    cb();
  } catch (err) {
    throw new Error(err);
  }
};

fn(1);
```

这样的代码运行时肯定会报错，但是由于我们使用了 `any` 类型，因此 `TS` 在静态类型检查时并不会帮我们报错。

如果我们改为 `unknown` 类型

```ts
const fn = (cb: unknown) => {
  try {
    cb(); // x
  } catch (err) {
    throw new Error(err);
  }
};

fn(1);
```

此时，`TS` 会帮助我们报错，提示：变量 `cb` 的类型为 `unknown`，这就是 `unknown` 类型的意义

那么如何解决这种错误呢？我们可以使用类型守卫来缩窄类型，比如：

```ts
const fn = (cb: unknown) => {
  try {
    if (typeof cb === "function") {
      cb();
    }
  } catch (err) {
    throw new Error(err);
  }
};

fn(1);
```

## 运算符结果

我们来看看与不同类型进行联合类型或交叉类型计算后，`unknown` 进行运算后的结果有什么区别

```ts
type A1 = unknown & string; // string
type A2 = unknown & number; // number
type A3 = unknown & null; // null
type A4 = unknown & undefined; // null
type A5 = unknown & null & undefined; // never
type A6 = unknown & string[]; // string[]
type A7 = unknown & any; // any

type B1 = unknown | string; // unknown
type B2 = unknown | number; // unknown
type B3 = unknown | null; // unknown
type B4 = unknown | undefined; // unknown
type B5 = unknown | (null & undefined); // unknown
type B6 = unknown | string[]; // unknown
type B7 = unknown | any; // any
```

## 总结

1. 可以将 `any` 类型赋给给任意类型，并对该变量执行任何操作
2. 可以将 `unknown` 类型赋给任意类型，但是必须进行类型检查和类型断言才可以对变量进行操作
