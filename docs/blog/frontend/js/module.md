# 模块化

## CommonJS

CommonJS 是一个规范，最初提出来是在浏览器以外的地方使用，并且当时被命名为 ServerJS，后来为了体现它的广泛性，修改为 CommonJS，平时我们也会简称为 CJS

- Node 是 CommonJS 在服务器端一个具有代表性的实现；
- Browserify 是 CommonJS 在浏览器中的一种实现；
- webpack 打包工具具备对 CommonJS 的支持和转换；

Node 中对 CommonJS 进行了支持和实现，让我们在开发 node 的过程中可以方便的进行模块化开发：

- 在 Node 中每一个 js 文件都是一个单独的模块；
- 这个模块中包括 CommonJS 规范的核心变量：exports、module.exports、require；
- exports 和 module.exports 可以负责对模块中的内容进行导出
- require 函数可以帮助我们导入其他模块（自定义模块、系统模块、第三方库模块）中的内容

### exports 导出

exports 是一个对象，我们可以在这个对象中添加很多个属性，添加的属性会导出

```js
exports.name = name;
exports.age = age;
```

另一个文件导入

```js
const bar = require("bar");
```

- 意味着 main 中的 bar 变量等于 exports 对象；
- 也就是 require 通过各种查找方式，最终找到了 exports 这个对象；
- 并且将这个 exports 对象赋值给了 bar 变量；
- bar 变量就是 exports 对象了；**它们之间的关系是一种引用赋值**

### module.exports 导出

- CommonJS 中是没有 module.exports 的概念的；
- 但是为了实现模块的导出，Node 中使用的是 Module 的类，每一个模块都是 Module 的一个实例，也就是 module；
- 所以在 Node 中真正用于导出的其实根本不是 exports，而是 module.exports； 只是为了满足 CommonJS 的规范，创建了 exports 对象
- 因为 module 才是导出的真正实现者；

**为什么 exports 也可以导出呢？**

- 这是因为 module 对象的 exports 属性是 exports 对象的一个引用；
- 也就是说 module.exports = exports = main 中的 bar；

### require 细节

require 是一个函数，可以帮助我们引入一个文件（模块）中导入的对象

#### 查找规则

```js
require(X);
```

- 情况一：X 是一个核心模块，比如 path、http
  - 直接返回核心模块，并且停止查找
- 情况二：X 是以 ./ 或 ../ 或 /（根目录）开头的
  - 第一步：将 X 当做一个文件在对应的目录下查找；
    - 1.如果有后缀名，按照后缀名的格式查找对应的文件
    - 2.如果没有后缀名，会按照如下顺序：
      - 1 > 直接查找文件 X
      - 2 > 查找 X.js 文件
      - 3 > 查找 X.json 文件
      - 4 > 查找 X.node 文件
  - 第二步：没有找到对应的文件，将 X 作为一个目录
    - 查找目录下面的 index 文件
      - 1 > 查找 X/index.js 文件
      - 2 > 查找 X/index.json 文件
      - 3 > 查找 X/index.node 文件
  - 如果没有找到，则报错
- 直接是一个 X（没有路径），并且 X 不是一个核心模块：那么会从当前目录查找 node_module 下是否有该文件，一级一级向上找，直到找到根目录，没有找到则报错

## ES Module

ES Module 和 CommonJS 的模块化有一些不同之处：

- 一方面它使用了 import 和 export 关键字；
- 另一方面它采用编译期的静态分析，并且也加入了动态引用的方式；

ES Module 模块采用 export 和 import 关键字来实现模块化：

- export 负责将模块内的内容导出；
- import 负责从其他模块导入内容；

采用 ES Module 将自动采用严格模式：use strict

### export 关键字

- 方式一：在语句声明的前面直接加上 export 关键字 n

  ```js
  export n
  ```

- 方式二：将所有需要导出的标识符，放到 export 后面的 {}中 p

  > **注意：这里的 {}里面不是 ES6 的对象字面量的增强写法，{}也不是表示一个对象；**

  ```js
  export { name, age };
  ```

  所以： `export {name: name}`，是错误的写法；

- 方式三：导出时给标识符起一个别名

  ```js
  export { name as barName, age as barAge };
  ```

### export 和 import 结合使用

```js
export { sum as barSum } from "./bar.js";
```

为什么要这样做呢？

- 在开发和封装一个功能库时，通常我们希望将暴露的所有接口放到一个文件中；
- 这样方便指定统一的接口规范，也方便阅读；
- 这个时候，我们就可以使用 export 和 import 结合使用；

### default 用法

```js
export default demo;
```

默认导出（default export）：

- 默认导出 export 时可以不需要指定名字；
- 在导入时不需要使用 {}，并且可以自己来指定名字；
- 它也方便我们和现有的 CommonJS 等规范相互操作；

> 注意：在一个模块中，只能有一个默认导出（default export）

### import 函数

通过 import 加载一个模块，是不可以在其放到逻辑代码中的：

- 这是因为 ES Module 在被 JS 引擎解析时，就必须知道它的依赖关系；

- 由于这个时候 js 代码没有任何的运行，所以无法在进行类似于 if 判断中根据代码的执行情况；

- 甚至下面的这种写法也是错误的：因为我们在运行的时候才能确定 path 的值；

  ```js
  if (true) {
    import sub from "./modules/foo.js";
  }
  ```

- 但是某些情况下，我们确确实实希望动态的来加载某一个模块：这个时候我们需要使用 import() 函数来动态加载；

  ```js
  if (flag) {
    import("./modules/aaa.js").then((aaa) => {
      aaa.aaa();
    });
  }
  ```

## CommonJS 的加载过程

CommonJS 模块加载 js 文件的过程是运行时加载的，并且是同步的：

- 运行时加载意味着是 js 引擎在执行 js 代码的过程中加载 模块；

- 同步的就意味着一个文件没有加载结束之前，后面的代码都不会执行；

  ```js
  if (flag) {
    const foo = require("./foo");
  }
  ```

CommonJS 通过 module.exports 导出的是一个对象：

- 导出的是一个对象意味着可以将这个对象的引用在其他模块中赋值给其他变量；
- 但是最终他们指向的都是同一个对象，那么一个变量修改了对象的属性，所有的地方都会被修改；

## ES Module 加载过程

ES Module 加载 js 文件的过程是编译（解析）时加载的，并且是异步的：

- 编译时（解析）时加载，意味着 import 不能和运行时相关的内容放在一起使用：

- 比如 from 后面的路径需要动态获取；

- 比如不能将 import 放到 if 等语句的代码块中；

- 所以我们有时候也称 ES Module 是静态解析的，而不是动态或者运行时解析的；

异步的意味着：JS 引擎在遇到 import 时会去获取这个 js 文件，但是这个获取的过程是异步的，并不会阻塞主线程继续执行：

- 也就是说设置了 type=module 的代码，相当于在 script 标签上也加上了 async 属性；
- 如果我们后面有普通的 script 标签以及对应的代码，那么 ES Module 对应的 js 文件和代码不会阻塞它们的执行；

ES Module 通过 export 导出的是变量本身的引用：

- export 在导出一个变量时，js 引擎会解析这个语法，并且创建模块环境记录（module environment record）；
- 模块环境记录会和变量进行 绑定（binding），并且这个绑定是实时的；
- 而在导入的地方，我们是可以实时的获取到绑定的最新值的；

> 注意：在导入的地方不可以修改变量，因为它只是被绑定到了这个变量上（其实是一个常量）

![image-20220311202437372](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20220311202437372.png)
