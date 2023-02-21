# source-map 配置

## 认识 source-map

我们的代码通常运行在浏览器上时，是通过打包压缩的：

- 也就是真实跑在浏览器上的代码，和我们编写的代码其实是有差异的
- 比如 `ES6` 的代码可能被转换成 `ES5`
- 比如对应的代码行号、列号在经过编译后肯定会不一致
- 比如代码进行丑化压缩时，会将编码名称
- 比如我们使用了 `TypeScript` 等方式编写的代码，最终转换成 `JavaScript`

但是，当代码报错需要调试时（debug），调试 **转换后的代码** 是很困难的

如何可以调试这种转换后不一致的代码呢？答案就是 **source-map**

source-map：是从已转换的代码，映射到 **原始的源文件**，使得浏览器可以重构原始源并在调试器中显示重建的原始源

## 如何使用 source-map

两个步骤：

- 第一步：根据源文件，生成source-map文件，webpack在打包时，可以通过配置生成source-map
- 第二步：在转换后的代码，最后添加一个注释，它指向sourcemap
  `//# sourceMappingURL=common.bundle.js.map`

在浏览器中，我们可以按照如下的方式打开 `source-map`：

![20220615192632](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220615192632.png)

![20220615192747](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220615192747.png)


## 分析 source-map

最初 `source-map` 生成的文件带下是原始文件的10倍，第二版减少了约50%，第三版又减少了50%，所以目前一个133kb的文件，最终的 `source-map` 的大小大概在300kb

`source-map` 的内容大概如下：

- version：当前使用的版本，也就是最新的第三版
- sources：从哪些文件转换过来的 `source-map` 和打包的代码（最初始的文件）
- names：转换前的变量和属性名称（因为我目前使用的是 `development` 模式，所以不需要保留转换前的名称）
- mappings： `source-map` 用来和源文件映射的信息（比如位置信息等），一串 `base64 VLQ` （veriable length quantity 可变长度值）编码
- file：打包后的文件（浏览器加载的文件）
- sourceContent：转换前的具体代码信息（和sources是对应的关系）
- sourceRoot：：所有的sources相对的根目录

`source-map` 文件：

![20220615193441](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220615193441.png)

## 生成 source-map

`webpack` 为我们提供了非常多的选项（目前是26个），来处理 `source-map`

选择不同的值，生成的 `source-map` 会稍微有差异，打包的过程也会有**性能的差异**，可以根据不同的情况进行选择

下面几个值不会生成source-map：

- false：不使用 `source-map`，也就是没有任何和 `source-map` 相关的内容。
- none（缺省的 `devtool`）：production模式下的默认值，不生成 `source-map`
- eval：
  - `development` 模式下的默认值，不生成 `source-map`，但是它会在eval执行的代码中，添加 `//# sourceURL=`
  - 它会被浏览器在执行时解析，并且在调试面板中生成对应的一些文件目录，方便我们调试代码

    ![20220615194446](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220615194446.png)

    ![20220615194511](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220615194511.png)

## source-map 的值

- source-map
  - 生成一个独立的 `source-map` 文件，并且在 `bundle` 文件中有一个注释，指向 `source-map` 文件
  - bundle 文件有如下注释，开发工具可以根据这个注释找到 `source-map` 文件，并且解析：
  `//# sourceMappingURL=bundle.js.map`
  ![20220615194821](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220615194821.png)
- eval-source-map
  - 会生成 `source-map` 文件，但是 `source-map` 是以 `DataUrl` 添加到 `eval` 函数的后面
  ![20220615195043](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220615195043.png)
- inline-source-map
  - 会生成 `source-map` 文件，但是 `source-map` 是以 `DataUrl` 添加到 `bundle` 文件的后面
  ![20220615195214](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220615195214.png)
- cheap-source-map
  - 会生成 `source-map` 文件，但是会更加高效一些（cheap表示低开销），因为它没有生成列映射（Column Mapping）,只能定位到行
  ![20220615195407](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220615195407.png)
- cheap-module-source-map
  - 会生成 `source-map` 文件，类似于 `cheap-source-map` ，但是对**源自loader的sourcemap**处理会更好，也就意味者 `loader` 的 `source-map` 也会定位到行
  - 如果我们对 `js` 文件加入 `babel-loader`，可以看到如下效果：

    cheap-source-map:
    ![20220615212441](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220615212441.png)
    cheap-module-source-map:
    ![20220615212529](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220615212529.png)
- hidden-source-map
  - 会生成 `source-map` 文件，但是不会对 `source-map` 文件进行引用
  - 相当于删除了打包文件中对 `source-map` 的引用注释
- nosources-source-map
  - 会生成 `source-map` 文件，但是生成的 `source-map` 只有错误信息的提示，不会生成源代码文件
  ![20220615212342](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220615212342.png)
  ![20220615212324](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220615212324.png)

## 多个值的组合

webpack提供给我们的**26个值**，是可以进行多组合的

组合的规则如下：

- inline- | hidden- | eval 三个值时三选一
- nosources：可选值
- cheap 可选值，并且可以跟随 module 的值

> 组合：[inline-|hidden-|eval-][nosources-][cheap-[module-]]source-map

在开发中，最佳的实践是什么呢？

- 开发阶段：推荐使用 `source-map` 或者 `cheap-module-source-map`
- 测试阶段：推荐使用 `source-map` 或者 `cheap-module-source-map`
- 发布阶段：false、缺省值