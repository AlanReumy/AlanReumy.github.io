# keyof 操作符

如果你使用过 `TS` 内置的工具类型，就会发现它们的内部实现都使用了一个叫 `keyof` 的操作符

在 `JS` 中，我们可以使用 `Object.keys()` 方法来返回一个对象的所有类型

而在 `TS` 中，我们可以使用 `keyof` 关键字，来返回一个对象类型对应所有键的类型的联合类型：

```ts
const user = {
  id: 30,
  name: "curry",
};

type UserKeys = keyof typeof user; // 'id' | 'name'

type U = typeof user[keyof typeof user]; // string | number
```

## 应用场景

假如我们要实现一个 `getProperty` 函数，在 `JS` 中，它的实现如下：

```js
function getProperty(obj, key) {
  return obj[key];
}
```

如果改成 `TS` 来写，直接写入到 `TS` 文件中会报错，提示 `obj` 和 `key` 是 `any` 类型，而我们使用 泛型 + `keyof` 操作符 修改后：

```ts
function getProperty<T extends object, K extends keyof T>(obj: T, key: K) {
  return obj[key];
}
```

这样我们就可以获取对象的指定属性的值：

```ts
const userId = getProperty(user, "id");
```

`keyof` 操作符 不仅仅可以应用于对象类型，还可以应用于类， `any` 类型，基本数据类型，枚举类型，映射类型

```ts
type K = keyof boolean;

class Person {
  name: string = "curry";
}
type P = keyof Person;

enum HttpMethod {
  GET,
  POST,
}
type Method = keyof typeof HttpMethod;
```
