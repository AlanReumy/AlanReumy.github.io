# 类数组

## 定义

- 数组是一个特殊的对象，与常规对象的区别是：

  - 当有新元素加入到数组时，length 属性自动更新
  - 设置 length 属性，会自动截断
  - 从 Array.prototype 原型对象中继承方法

- 类数组是一个拥有 length 属性的对象，并且它的属性是非负整数，它的方法是从 Object.prototype 中继承的

## 区别

类数组是普通对象，它的原型关系与数组不同

## 类数组转换为数组

1. Array.from()

```js
let args = Array.from(arguments);
console.log(args instanceof Array); // true
```

2. Array.prototype.slice.call()

```js
let args = Array.prototype.slice.call(arguments);
console.log(args instanceof Array); // true
```

3. rest 参数

```js
let args = [...arguments];
console.log(args instanceof Array); // true
```

4. 使用 apply

```js
let args = [].concat.apply([], arguments);
console.log(args instanceof Array); // true
```

## 转换须知

- 转换后的数组长度由`length`属性决定，索引不连续时转换结果是连续的，会自动补位，考虑索引仅仅考虑非负整数的索引

```js
let arrayLike = {
  length: 4,
  0: 0,
  1: 1,
  3: 3,
  4: 4,
};

console.log(Array.from(arrayLike)); // [0,1,undefined,3]
```

- 使用 slice 转换产生稀疏数组

```js
let arrayLike = {
  length: 4,
  "-1": "-1",
  0: 0,
  a: "a",
  1: 1,
  3: 3,
  4: 4,
};
console.log(Array.prototype.slice.call(arrayLike)); // [0,1,empty,3]
```
