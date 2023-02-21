# 函数重载
现在有这样有一个场景：这里有一个 `greet` 的函数

```ts
function greet(name: string) {
	return `Hello ${name}`
}
```

假如我们又想传入多个名字，我们可以使用联合类型：

```ts
function greet(name: string | string[]) {
	return `Hello ${name}`
}
```

其实我们还有另外一种方式：**函数重载**

## 使用

使用函数重载，需要定义 **重载签名** 和 **实现签名**

重载签名定义了函数中每个参数的类型和返回值类型，但不包含函数体，一个函数可以有多个重载签名：

```ts
function greet(name: string): string

function greet(name: string[]): string[]
```

实现签名的函数参数和返回值类型都需要是更通用的类型，并且包含函数体，一个函数只能有一个实现签名：

```ts
function greet(name: unknown): unknown {
	if (typeof name === 'string') {
		return `Hello ${name}`
	} else if (Array.isArray(name)) {
		return name.map(item => `Hello ${item}`)
	}
	throw new Error('can not to greet')
}
```

在使用函数重载的时候，`TS` 编译器会帮我们一个个匹配函数参数的类型，如果匹配，就立即返回
