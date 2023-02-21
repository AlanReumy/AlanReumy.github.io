# declare 作用

在某些时候，我们想将项目从 `JS` 迁移 至 `TS`，这时如果我们直接修改文件后缀名，修改后可能会提示模块找不到的情况，这是什么原因呢？

答案是，`TS` 不认识某个全局变量， 如果我们想要解决这个问题，就需要 `declare` 关键字来声明变量

## 使用

我们可以使用 `declare` 关键字来声明一个变量，他的使用方法如下：

```ts
declare var koa: any;
// |     |   |    |
//声明  变量 变量名 类型
```

在 `TS` 内部，已经帮我们完成了常用变量和模块的声明操作，比如 `Math` 、`JSON`、`Object` 等

此外，`declare` 关键字除了可以声明全局变量，还可以声明全局函数，全局类，全局枚举等，甚至可以声明文件模块

比如，在 `vite` 的 `client.d.ts` 中，我们可以看到如下的代码：

```ts
declare module "*.css*" {
  const css: string;
  export default css;
}
```

这是为了在文件中引入 `css` 文件并且不报错，否则，`TS` 会提示模块找不到的错误

在日常使用中，我们不需要自己编写第三方库的 `.d.ts` 文件，直接去 [官网](https://www.typescriptlang.org/zh/dt/search?search=) 搜索安装即可

## 扩展已有模块中定义的类型

使用 `declare module` 语法来扩展已有模块中定义的类型

比如，给 `Vue` 组件实例增加 `$axios` 属性：

```ts
declare module "" {
  interface ComponentCustomProperties {
    $axios: AxiosInstance;
  }
}
```

然后我们通过全局挂载属性后，就可以使用了：

```ts
const { proxy } = getCurrentInstance as ComponentInternalInstance;
proxy!.$axios.get("xxx").then((res) => {});
```
