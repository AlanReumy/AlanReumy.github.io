# 从 the super tiny compiler 读到编译原理

无意间看见了 `the super tiny compiler` 这个仓库，注释清晰，代码简洁，很适合通过这个仓库学习简单的编译原理

## 实现一个简单的编译器

在项目中，将 lisp 代码编译成 c 语言代码

```txt
                   LISP                      C

    2 + 2          (add 2 2)                 add(2, 2)
    4 - 2          (subtract 4 2)            subtract(4, 2)
    2 + (4 - 2)    (add 2 (subtract 4 2))    add(2, subtract(4, 2))
```

一个编译器在编译代码的过程中大概会经过如下过程：

1. 解析：包括词法分析和语法分析，词法分析会生成 tokens 数组，接着通过语法分析生成 ast 抽象语法树
2. 转换：将 ast 进行转换，进行 dfs 深度优先遍历，通过 visitor 对象，生成新的 ast
3. 代码生成：通过新的 ast 生成最终的代码

### 1. 解析(Parsing)

#### 1.1 词法分析(Lexical Analysis)

词法分析通过获取源代码，并将他们分割成 tokens 数组，tokens 数组的每一个元素是一个对象，用来描述一小段语法

转换后的 tokens 数组如下

```js
[
  { type: "paren", value: "(" },
  { type: "name", value: "add" },
  { type: "number", value: "2" },
  { type: "paren", value: "(" },
  { type: "name", value: "subtract" },
  { type: "number", value: "4" },
  { type: "number", value: "2" },
  { type: "paren", value: ")" },
  { type: "paren", value: ")" },
];
```

#### 1.2 语法分析(Syntactic Analysis)

语法分析阶段获取 tokens 数组，并将他们进行重新格式化，他们被用来描述语法的每个部分及其彼此的关系

转换后的 ast 抽象语法树如下

```js
{
  type: 'Program',
  body: [{
    type: 'CallExpression',
    name: 'add',
    params: [{
      type: 'NumberLiteral',
      value: '2',
    }, {
      type: 'CallExpression',
      name: 'subtract',
      params: [{
        type: 'NumberLiteral',
        value: '4',
      }, {
        type: 'NumberLiteral',
        value: '2',
      }]
    }]
  }]
}
```
