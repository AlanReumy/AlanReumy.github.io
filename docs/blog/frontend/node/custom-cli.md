# 实现一个 CLI 工具

## CLI 概念

Command Line Interface，是一种通过命令行来在运行一些代码，交互的方式来实现某些功能的工具或者应用，如我们前端开发的过程中使用到的，vue-cli，vite、create-react-app 等等，使用他们可以减少开发中的一些低级重复劳动，或者规范开发工作流，提高开发效率。

## 功能目标

- 创建模板创建一个 `Koa` 项目
- 根据命令来指定中间件以及配置文件
- 指定包管理工具

## 实现

要实现上述功能，我们大致需要一下几个库：

- inquirer - 以问答形式指定配置
- commander - 向用户提示可传递的参数
- ejs - 创建模板
- execa - 更加人性化的开启子进程
- chalk - 打印信息的时候可以增加颜色等功能

### 入口文件

我们首先在 `bin` 目录下创建 `index.js` 入口文件：

```js
import { Command } from "commander";
import { create } from "./command/create.js";

const program = new Command();

program.name("koa-inventor").description("a cli for koa").version("1.0.4");

program
  .command("create")
  .description("create a koa project")
  .action(() => {
    // @todo
    create();
  });

program.parse();
```

通过创建 `Command` 实例对象，我们可以指定一些信息，用来给用户提示，运行 `index.js`

接着我们可以实现一下 上面没有实现的 `create()` 函数

`create()`函数大概做了如下功能：

1. 创建文件夹
2. 创建入口文件
3. 创建 `package.json` 文件
4. 安装项目依赖

`create()`函数实现如下：

```js
export async function create() {
  // 根据 inquirer 创建配置对象
  const config = await createConfig();
  const rootPath = `./${config.projectName}`;
  config.rootPath = rootPath;

  // 1.创建文件夹
  fs.mkdirSync(rootPath);

  createEditorConfig(config);
  createMiddleWareFile(config);

  console.log(chalk.blue("create projectFolder successfully"));

  // 2.创建入口文件
  fs.writeFileSync(`./${rootPath}/index.js`, createBootstrapTemplate(config));
  console.log(chalk.blue("create index.js successfully"));

  // 3.创建package.json
  fs.writeFileSync(
    `./${rootPath}/package.json`,
    createPackageJsonTemplate(config)
  );
  console.log(chalk.blue("create package.json successfully"));

  // 4.安装依赖
  console.log(chalk.blue("installing dependencies..."));

  await installDependencies(config);

  console.log(chalk.blue("install dependencies successful"));
  console.log(chalk.blue("happy coding~~"));
}
```

### 配置信息

但我们运行 `node index.js create` 的时候，此时需要根据用户自定义的信息来创建文件，这个时候我们就需要使用 `inquirer` 这个库，具体使用请参考官方文档

`createConfig()`函数 :

```js
async function createConfig() {
  return await inquirer.prompt([
    projectNameConfig(),
    middlewareConfig(),
    portConfig(),
    packageManagerConfig(),
  ]);
}
```

`projectNameConfig()`函数：

```js
function projectNameConfig() {
  return {
    type: "input",
    name: "projectName",
    validate(projectName: string) {
      if (projectName) return true;
      return "please enter your project name!";
    },
  };
}
```

这里以这两个函数来作为参考。这时候我们重新运行该项目

这样我们就可以通过问答的方式，让用户创建项目

### 创建模板及文件

我们可以通过 `ejs` 这个模板引擎库来创建模板，具体使用方式参考官方文档，主要是调用了 `ejs` 的 `render` 函数来进行创建：

```js
export function createBootstrapTemplate(config: Config) {
  const __dirname = fileURLToPath(import.meta.url);
  const template = fs.readFileSync(
    path.resolve(__dirname, "../../../templates/index.ejs")
  );
  const code = ejs.render(template.toString(), config);
  return code;
}
```

> 这里之所以通过 `fileURLToPath` 函数获取到 `__dirname` 是因为，我使用的是 es module 的模块化规范，如果想要在 `node` 环境下，使用该规范，可以在 `package.json` 文件中添加 `type: module` 这个字段

`index.ejs` 文件：

```bash
import Koa from 'koa'

<% if (middleware.includes('koa-static')) { %>
import serve from 'koa-static'
<% } %>
<% if (middleware.includes('koa-bodyparser')) { %>
import bodyParser from 'koa-bodyparser'
<% } %>
<% if (middleware.includes('koa-router')) { %>
import Router from 'koa-router'
import useRoutes from './router/index.js'
<% } %>
const app = new Koa()
<% if (middleware.includes('koa-static')) { %>
app.use(serve('./static'));
<% } %>
<% if (middleware.includes('koa-bodyparser')) { %>
app.use(bodyParser())
<% } %>
<% if (middleware.includes('koa-router')) { %>
app.useRoutes = useRoutes
app.useRoutes()
<% } %>
app.listen(<%= (port) %>,() => {
  console.log('server is running on <%= (port) %> ');
})
```

接着我们可以以相同的方式创建模板以生成 `package.json` 文件

### 安装依赖

在根据用户配置，创建完所有配置文件后，我们需要根据 `package.json` 来安装项目依赖。

通过 `execa` 这个库，我们可以开启子进程，并指定运行的目录。

```js
import { execa } from "execa";
import { Config } from "../type/config";

export async function installDependencies(config: Config) {
  const { packageManager, rootPath } = config;
  const installStatement = packageManager.includes("npm")
    ? `${packageManager} install`
    : "yarn";
  await execa(installStatement, {
    cwd: rootPath,
  });
}
```

### 测试

在 `package.json` 文件中，指定 `bin` 字段作为入口文件，接着使用 `npm link` 将该项目链接到本地，进行测试。

## 发布

开发完成后，我们需要发布到 `npm` 上供别人使用。

- `npm login` 登录
- `npm publish` 发布

## 结语

本项目的起因主要是自己，使用 `koa` 开发的时候发现每次都要重新安装很多中间件以及配置，使用该 `cli` 就可以快速开发项目，减少配置的时间。

`koa-inventor` 项目地址： https://github.com/b1t010/koa-inventor

如果本文或本项目帮助到你的话，可以点赞或点个 star⭐ 支持一下我，非常感谢。
