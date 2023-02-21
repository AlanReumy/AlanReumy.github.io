# 项目管理代码规范

本文主要介绍前端项目搭建的时候配置代码规范化的问题

## editorconfig

在多人合作的项目中，每个人的开发习惯是不同的。以缩进来说，有的人习惯使用 space 键来进行缩进，有的人喜欢用 tab 键，有的人喜欢设置缩进为 4 个空格，有的人喜欢设置为 2 个空格。这样产生的后果就是每个人修改后的代码在格式上总是不统一的，那么提交到 git 上就会代码风格不一致，变得丑陋无比。

使用 `editorconfig` 可以对不同文件进行代码风格的控制

`.editorconfig`：

```.editorconfig
# http://editorconfig.org
root = true

[*]
#缩进风格：空格
indent_style = space
#缩进大小2
indent_size = 2
#换行符lf
end_of_line = lf
#字符集utf-8
charset = utf-8
#是否删除行尾的空格
trim_trailing_whitespace = true
#是否在文件的最后插入一个空行
insert_final_newline = true

[*.md]
trim_trailing_whitespace = false

[Makefile]
indent_style = tab
```

使用 `vscode` 可以搭配插件 `EditorConfig for VS Code` 使用

## prettier

`prettier` 是一个 Opinionated 的代码格式化工具

1. 安装

   ```bash
   npm intall prettier -D
   ```

2. 配置

   创建 `.prettierrc` 文件进行配置

   ```bash
   "useTabs": false,
   "printWidth": 80,
   "tabWidth": 2,
   "singleQuote": true,
   "trailingComma": "none",
   "semi": false
   ```

   - `useTabs` 表示是否使用 tab 缩进
   - `printWidth` 表示单行宽度
   - `tabWidth` 表示 tab 宽度
   - `singleQuote` 表示是否单引号
   - `trailingComma` 表示是否有行尾逗号
   - `semi` 表示是否使用分号

   创建 `.prettierignore` 表示需要忽略格式化的文件

   ```.prettierignore
   /dist/*
   .local
   .output.js
   /node_modules/**

   **/*.svg
   **/*.sh

   /public/*
   ```

3. 使用

   使用命令 `prettier --write`可以对代码进行格式化

   `vscode` 可以搭配插件 `prettier` 使用

## eslint

`eslint` 是 `JavaScript` 代码检测工具

1. 安装

   在使用 `vue-cli` 创建项目的时候选择 `eslint+prettier`

2. 配置

   使用 `.eslintrc.js` 进行配置

   ```js
   module.exports = {
     root: true,
     env: {
       node: true,
     },
     extends: [
       "plugin:vue/vue3-essential",
       "eslint:recommended",
       "@vue/typescript/recommended",
       "@vue/prettier",
       "@vue/prettier/@typescript-eslint",
       // 此插件是为了解决eslint与prettier冲突的问题
       "plugin:prettier/recommended",
     ],
     parserOptions: {
       ecmaVersion: 2020,
     },
     rules: {
       "no-console": process.env.NODE_ENV === "production" ? "warn" : "off",
       "no-debugger": process.env.NODE_ENV === "production" ? "warn" : "off",
       "@typescript-eslint/no-var-requires": "off",
     },
   };
   ```

## git 规范

### husky

在使用 `git hooks` 的时候，我们一般会使用 `husky` 工具，它可以使本地 `git hooks` 变的更加的简单

1. 安装

   ```bash
   npm install husky --save-dev
   ```

2. 修改 git hooks 默认目录

   ```bash
   npx husky install
   ```

3. 创建 hooks，语法：`husky add <file> [cmd]`

   ```bash
   npx husky add .husky/pre-commit "npm run lint"
   ```

此时，在进行 `git commit` 的时候，就会自动进行代码格式化

### commitizen

`commitizen` 是一个格式化 commit message 的工具

1. 安装

   ```
   npx commitizen init cz-conventional-changelog --save-dev --save-exact
   ```

2. 使用

   使用命令可以进入交互式命令行：`npx cz`

### commitlint

`commitlint` 是为了防止别人依旧使用 `git commit` 按照不规范的格式提交

1. 安装

   ```
   npm install @commitlint/config-conventional @commitlint/cli -D
   ```

2. 配置

   创建 `commitlint.config.js`

   ```
   module.exports = {
     extends: ['@commitlint/config-conventional']
   }
   ```

3. 使用 `husky` 生成 `commit-msg` 文件，验证提交信息

   ```
   npx husky add .husky/commit-msg "npx --no-install commitlint --edit $1"
   ```
