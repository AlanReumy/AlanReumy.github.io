# postcss

针对不同浏览器的特性，css语法、js语法，它们的兼容性是不一定的。

在很多脚手架中，我们都能看到这样的信息：

```
> 1%
last 2 versions
not dead
```

我们可以从 `caniuse.com` 这个网站中的usage-table找到大部分浏览器的市场占有率

## 认识browserlist

如何在前端各种工具中，共享配置的浏览器兼容性条件，这个时候就需要 `browserlist` 这个工具

browserlist: 是一个在不同的前端工具之间，共享目标浏览器和Node.js版本的配置。比如，`Babel`、`EsLint`、`PostCSS`、`AutoPrefixer` 等工具都需要根据 `browserlist` 工具来进行配置。

### 编写规则

编写规则一：

- defaults: Browserlist的默认浏览器：( > 0.5%, last 2 versions, Firefox ESR, not dead)
- 5%：市场占有率大于5%的浏览器
- dead：24个月内没有官方支持或更新的浏览器
- last 2 version：每个浏览器的最后两个版本

编写规则二：

- node10：选择最新的node.js10.x.x版本
- iOS7：直接使用iOS浏览器版本7
- supports es6-module：支持特定功能的浏览器，这里是es6-module


> 注意：编写的多个条件之间的关系是并行的，只要有一个满足即可

## PostCss

- `PostCSS` 是一个通过JavaScript来转换样式的工具，可以帮助我们进行一些CSS的转换和适配，比如自动添加浏览器前缀，css样式的重置。
- 但是实现这些工具，我们需要借助于 `PostCSS` 对应的插件


如何使用：

- 查找 `PostCSS` 在构建工具中对应的扩展，比如 webpack 中的 `postcss-loader`
- 选择可以添加你需要的 `PostCSS` 相关的插件

### 命令行使用postcss

- 安装：
  ```
  npm install postcss postcss-cli -D
  ```
- 执行：
  ```
  npx postcss -o result.css ./src/css/test.css
  ```

### 插件autoprefixer

- 添加浏览器前缀：安装 `autoprefixer`：
  ```
  npm install autoprefixer -D
  ```
- 直接使用使用postcss工具，并且制定使用autoprefixer：
  ```
  npx postcss --use autoprefixer -o result.css ./src/css/style.css
  ```

### postcss-loader

在真实开发中我们必然不可能使用命令行工具来对 `css` 进行处理，而是可以借助构建工具：

webpack中使用 `postcss` 就是通过 `postcss-loader` 来处理

- 安装：
  ```
  npm install postcss-loader -D
  ```
- 添加配置：
  ```js
  {
    test: /\.css$/,
    use: [
      { loader: 'style-loader' },
      { loader: 'css-loader' },
      { 
        loader: 'postcss-loader',
        options: {
          postcssOptions: {
            plugins: [
              require('autoprefixer')
            ]
          }
        }
      },
    ]
  }
  ```

### 单独配置文件

当然，我们也可以将这些配置信息放到一个单独的文件中进行管理：

```js
module.exports = {
  plugins: [
    require('autoprefixer')
  ]
}
```

### 插件postcss-preset-env

事实上，在配置 `postcss-loader` 时，我们配置插件并不需要使用 `autoprefixer`

我们可以使用另外一个插件：`postcss-preset-env`

`postcss-preset-env` 也是一个 `postcss` 的插件：

- 它可以帮助我们将一些现代的CSS特性，转成大多数浏览器认识的CSS，并且会根据目标浏览器或者运行时环境添加所需的polyfill
- 也包括会自动帮助我们添加 `autoprefixer`（所以相当于已经内置了 `autoprefixer`）

使用：

- 安装：
  ```
  npm install postcss-preset-env -D
  ```
- 修改配置：
  ```js
  {
    test: /\.css$/,
    use: [
      { loader: 'style-loader' },
      { loader: 'css-loader' },
      { 
        loader: 'postcss-loader',
        options: {
          postcssOptions: {
            plugins: [
              // 在使用某些postcss插件时，也可以直接传入字符串
              // require('postcss-preset-env') 等同于
              'postcss-preset-env'
            ]
          }
        }
      },
    ]
  }
  ```

### 注意事项

在 `css` 文件中，如果使用 `@import '.xxx'` 来引入其他 `css` 文件，默认情况下是不会被 `postcss-loader` 处理的，我们需要做单独的配置来解决这个问题：

```js
{
  test: /\.css$/,
  use: [
    { loader: 'style-loader' },
    { 
      loader: 'css-loader',
      options: {
        // 此配置表示加载新的css文件时，再会被postcss-loader处理一次
        importLoaders: 1
      }
    },
    { 
      loader: 'postcss-loader',
      options: {
        postcssOptions: {
          plugins: [
            'postcss-preset-env'
          ]
        }
      }
    },
  ]
}
```