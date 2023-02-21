# 映射类型

映射类型是一种泛型类型，可用于原有的对象类型，映射成新的对象类型

## 语法

```ts
{ [P in K]: T }
```

其中 `[p in K]` 类似于 `JS` 中的 `for in` 遍历，而 `TS` 变量用于表示任意类型

此外，你还可以使用 `readonly` 和 `?` 这两个额外的修饰符

```ts
{ readonly [P in K]?: T }
```

## 例子

```ts
type Item = { a: string };
type T1 = { [P in "x" | "y"]: P }; // { x: 'x',y: 'y' }
type T2 = { [P in "x" | "y"]: number }; // { x: number, y: number }
type T3 = { [P in "a"]: Item[P] }; // { a: string }
```

在 `TS` 的工具类型中，大量使用了 `映射类型` ，比如 `Partial` ，它可以将一个类型所有的键都变为可选的

自己实现 `Partial`：

```ts
type MyPartial<T> = { [P in keyof T]?: T[P] };

type User = {
  id: number;
  name: string;
  age: string;
};

type PartialUser = MyPartial<User>;

// 等价于

type PartialUser = {
  id?: number | undefined;
  name?: string | undefined;
  age?: string | undefined;
};
```

## as 对 key 进行再次映射

TODO
