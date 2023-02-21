# node 包管理

通过包管理工具我们可以更好的管理自己的工具包，其他人也可以更好的使用我们的工具包。

## npm 包管理工具

Node Package Manager，也就是 Node 包管理器，但是目前已经不仅仅是 Node 包管理器了，在前端项目中我们也在使用它来管理依赖的包，比如 express、koa、react、react-dom、axios、babel、webpack 等等

npm 管理的包存放在哪里呢：

- 我们发布自己的包其实是发布到**registry**上面的
- 当我们安装一个包时其实是从**registry**上面下载的包

### 项目配置文件

常见属性：

- 必填属性：name、version
  - name 是项目名称
  - version 是当前项目的版本号
  - description 是项目描述
  - author 是作者相关信息（发布时用到）
  - license 是开源协议（发布时用到）
- private 属性：
  - 记录当前的项目是否是私有的
  - 当值为 true 时，npm 是不能发布它的，这是防止私有项目或模块发布出去的方式
- main 属性：
  - 设置程序的主入口
  - 这个入口和 webpack 打包的入口并不冲突，它是在你发布一个模块的时候会用到的
- scripts 属性：
  - 用于配置一些脚本命令，以键值对的形式存在
  - 配置后我们可以通过 npm run 命令的 key 来执行这个命令
  - npm start 和 npm run start 的区别是什么：
    - 它们是等价的
    - 对于常用的 start、 test、stop、restart 可以省略掉 run 直接通过 npm start 等方式运行
- dependencies 属性：
  - dependencies 属性是指定无论开发环境还是生成环境都需要依赖的包
  - 通常是我们项目实际开发用到的一些库模块
- devDependencies 属性
  - 一些包在生成环境是不需要的，比如 webpack、babel 等
  - 这个时候我们会通过 npm install webpack --save-dev，将它安装到 devDependencies 属性中
- engines 属性：
  - engines 属性用于指定 Node 和 NPM 的版本号
  - 在安装的过程中，会先检查对应的引擎版本，如果不符合就会报错
  - 也可以指定所在的操作系统 "os" : [ "darwin", "linux" ]，只是很少用到
- browserslist 属性：
  - 用于配置打包后的 JavaScript 浏览器的兼容情况，否则我们需要手动的添加 polyfills 来让支持某些语法
  - 也就是说它是为 webpack 等打包工具服务的一个属性

### 版本管理问题

npm 的包通常需要遵从 semver 版本规范：

- semver：<https://semver.org/lang/zh-CN/>
- npm semver：<https://docs.npmjs.com/misc/semver>

semver 版本规范是 X.Y.Z：

- X 主版本号（major）：当你做了不兼容的 API 修改（可能不兼容之前的版本）
- Y 次版本号（minor）：当你做了向下兼容的功能性新增（新功能增加，但是兼容之前的版本）
- Z 修订号（patch）：当你做了向下兼容的问题修正（没有新功能，修复了之前版本的 bug）

  **^和~的区别：**

- ^x.y.z：表示 x 是保持不变的，y 和 z 永远安装最新的版本
- ~x.y.z：表示 x 和 y 保持不变的，z 永远安装最新的版本

### npm install

安装 npm 包分两种情况：

- 全局安装（global install）： npm install yarn -g
- 项目（局部）安装（local install）： npm install

#### 全局安装

全局安装是直接将某个包安装到全局，通常使用 npm 全局安装的包都是一些工具包：yarn、webpack，并不是类似于 axios、express、koa 等库文件，所以全局安装了之后并不能让我们在所有的项目中使用 axios 等库。

#### 项目安装

项目安装会在当前目录下生产一个 node_modules 文件夹

局部安装分为开发时依赖和生产时依赖：

安装开发时依赖：

```shell
npm install webpack --save-dev
npm install webpack -D
npm i webpack -D
```

安装生产时依赖：

```shell
npm install axios
npm i axios
```

根据 package.json 中的依赖包

```shell
npm install
```

#### npm install 原理

原理图：

![image-20220312201625182](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20220312201625182.png)

npm install 会检测是否有 package-lock.json 文件：

- 没有 lock 文件:
  - 分析依赖关系，这是因为我们可能包会依赖其他的包，并且多个包之间会产生相同依赖的情况
  - 从 registry 仓库中下载压缩包（如果我们设置了镜像，那么会从镜像服务器下载压缩包）
  - 获取到压缩包后会对压缩包进行缓存（从 npm5 开始有的）
  - 将压缩包解压到项目的 node_modules 文件夹中
- 有 lock 文件:
  - 检测 lock 中包的版本是否和 package.json 中一致（会按照 semver 版本规范检测）
    - 不一致，那么会重新构建依赖关系，直接会走顶层的流程
  - 一致的情况下，会去优先查找缓存
    - 没有找到，会从 registry 仓库下载，直接走顶层流程
  - 查找到，会获取缓存中的压缩文件，并且将压缩文件解压到 node_modules 文件夹中

### package-lock.json

属性：

- name：项目的名称
- version：项目的版本
- lockfileVersion：lock 文件的版本
- requires：使用 requires 来跟着模块的依赖关系
- dependencies：项目的依赖
  - version 表示实际安装依赖的版本
  - resolved 用来记录下载的地址，registry 仓库 中的位置
  - requires 记录当前模块的依赖
  - integrity 用来从缓存中获取索引，再通过索引 去获取压缩包文件

### npm 其他命令

卸载某个依赖包：

```shell
npm uninstall package
npm uninstall package --save-dev
npm uninstall package -D
```

强制重新 build：

```shell
npm rebuild
```

清除缓存：

```shell
npm cache clean
```

## Yarn 工具

yarn 是由 Facebook、Google、Exponent 和 Tilde 联合推出了一个新的 JS 包管理工具，是为了弥补 npm 的一些缺陷而出现的

早期的 npm 存在很多的缺陷，比如安装依赖速度很慢、版本依赖混乱等等一系列的问题

虽然从 npm5 版本开始，进行了很多的升级和改进，但是依然很多人喜欢使用 yarn

## npx 工具

npx 是 npm5.2 之后自带的一个命令。

npx 的作用非常多，但是比较常见的是使用它来调用项目中的某个模块的指令
