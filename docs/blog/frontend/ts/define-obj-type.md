# 定义对象类型

在 `TS` 中如何定义对象中属性的类型呢？

其中一个方式就是使用 **索引签名**，当我们知道索引和值的类型时，我们就可以使用索引签名来定义该对象的类型

## 索引签名

使用方法：

```ts
interface typeObject {
  [key: keyType]: ValueType;
}
```

其中，keyType 的类型只能是：`string`、`number`、`symbol` 和 模板字面量类型

例子：

```ts
interface Option {
  [k in string]: string | boolean | number;
  timeout: number;
}

const option: Option = {
  message: "success",
  timeout: 5000,
  isFetching: false,
};
```

这时候有人会想，为什么在访问对象的属性时，可以用字符串或者数字来访问？

事实上：这是因为，在 `JS` 读取属性的时候，会将访问的键，**隐式的转换** 为字符串，`TS` 中也有这样的转换

```ts
interface Named {
  [k in string]: string;
}

const named: Named = {
  "1": "1",
  "2": "2",
};

named[1]; // '1'
named["1"]; // '1'

type K = keyof Named; // string | number
```

## Record 类型

除了索引签名外，我们也可以使用 `TS` 工具库中的 `Record` 类型来定义对象类型

```ts
// 内部实现
type Record<K extends keyof any, T> = {
  [P in K]: T;
};

// 使用
type Named = Record<string, string>;
```

### 索引签名 和 Record 类型 区别

它们之间的最大区别就是 **键的类型**：

- 索引签名中，键的类型只能是 `string`、`number`、`symbol` 和 模板字面量类型
- Record 类型中，键的类型可以是 字面量类型 或 字面量类型组成的联合类型
