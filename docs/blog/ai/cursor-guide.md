# Cursor 不完全上手指北

介绍 Cursor AI IDE 的最新上手指南，包括下载安装、基础操作、高级用法和技巧

## 下载并安装 Cursor

前往 Cursor 官网 下载。

## 熟悉界面

> 如果熟悉 VS Code，那么使用 Cursor 会让你感到得心应手。

Cursor 可以选择导入 VS Code 已有的配置。左侧显示文件结构，你可以搜索、暂存文件，并访问类似 VS Code 布局的扩展。一个主要的区别在于设置选项卡，你可以在这里启用自动补全、代码索引和文件排除等功能，可以根据你的需求自定义 Cursor

## 关键功能

- 自动补全：启用该功能后，Cursor 会在你输入代码时提供代码补全建议
- 代码索引：该功能会为你的代码库建立索引，以提供上下文相关的建议和查询。它对于理解大型项目非常有用，尤其是那些难以全面把握的代码库
- 文件排除：可以将敏感或无关文件排除在 AI 索引之外，以保护隐私并减少不必要的建议干扰
- composer：用于代码创作和编辑的功能模块，它可以帮助开发者更高效地编写和改进代码
- chat：像是对话助手，用户可以与之交流代码的问题、请教实现细节、或者了解某些编程概念

## 常用快捷键

### Edit

- Ctrl/⌘ + K：允许在编辑器窗口中生成新代码或编辑现有代码

![](https://codertzm.oss-cn-chengdu.aliyuncs.com/image.png)

![](<https://codertzm.oss-cn-chengdu.aliyuncs.com/image%20(1).png>)

### chat

- Ctrl/⌘ + L： 打开聊天
- Ctrl/⌘ + Alt/Option + L： 打开聊天历史（需要在打开聊天面板的情况下使用）

![](<https://codertzm.oss-cn-chengdu.aliyuncs.com/image%20(2).png>)

### composer

- Ctrl/⌘ + I，打开 composer 面板
- composer 是 Cursor 的特色功能，它的功能是在一个对话窗口里同时对多个文件进行修改

### Fix

- Ctrl/⌘ + Shift + E：对 linter 错误进行 AI 修复

![](<https://codertzm.oss-cn-chengdu.aliyuncs.com/image%20(3).png>)

## 实用技巧

### 1. 解析当前项目，优化程序

提示词：请分析项目、整理问题、并给出改进建议

### 2. 设置 Rules for AI

![](<https://codertzm.oss-cn-chengdu.aliyuncs.com/image%20(4).png>)

1. 配置 rules 让 cursor 更加智能
2. 给编辑器配置 rules：
   - 在项目根目录下，创建.cursorrules 文件
   - 具体 rules 规则可参考：[https://github.com/PatrickJS/awesome-cursorrules](https://github.com/PatrickJS/awesome-cursorrules)

### 3. 同步代码索引

为了得到准确的回答，记得开启这两项

![](<https://codertzm.oss-cn-chengdu.aliyuncs.com/image%20(5).png>)

其中：

codebase indexing 是将本地代码库分割成小块的语言数据，然后将每个小块发送给 cursor 服务器，使用 open AI 的 embeding API 进行嵌入，并且将文件的相对路径一起存储到远程的矢量数据库中，既保证了代码的安全性，又能利用索引提高回答质量

如果你不希望核心代码或者与项目无关的代码（日志文件、第三方工具、环境变量、打包后的文件）被索引，也可以设置忽略文件或目录（已经存在的 ignore 文件记得备份，不然会被覆盖）：

![](<https://codertzm.oss-cn-chengdu.aliyuncs.com/image%20(6).png>)

### 4. chat vs composer

在 cursor 中，有两处是可以用来对话的：

![](<https://codertzm.oss-cn-chengdu.aliyuncs.com/image%20(7).png>)

问题：什么时候用 chat 或 composer？

回答：chat 用于思考层面，composer 用于快速编辑

- chat：

  - 当你对需要实现的功能不太确定，所涉及的逻辑又十分复杂，并且这个逻辑又涉及到多个文件的时候，不妨在 chat 里询问清楚
  - 把自己想到的创意的点子说出来，让 chat 为你提供一些有参考价值的建议

- composer：

  - 当思路被理清楚，分解成指令
  - 你认为这个任务很简单
  - 你任务要实现的功能非常明确

### 5. 乱改代码的问题

给 cursor 提出问题前，为了防止他出现非常离谱的回答

- 可以让他先复述一遍你的需求
- 添加一句：不要影响其他功能和界面布局样式

### 6. 日志输出

给项目加上日志，提示词：在 【功能】的执行过程中，输出关键信息

### 7. README 文档

让 cursor 对 项目 README 文档进行更新和维护，这样重新回到项目的时候，也可以让 cursor 读取这个 README 文档

### 8. 增加注释

让 cursor 生成代码的时候增加注释，帮助自己理解实现的逻辑

### 9. 调试时 save all

使用 composer 时，当处于在项目测试和改 bug 阶段，先选择 save all，此时代码处于运行的状态，如果满意，选择 accept all，如果不满意，选择 reject all

### 10. 多用 @codebase

使用 composer 和 chat 功能时，多用 @codebase，否则 cursor 经常不知道项目内容是什么

参考：

- [Cursor - Build Software Faster](https://docs.cursor.com/get-started)
- [Cursor 十大使用技巧：免费无限量使用 Cursor Pro 会员指南\_哔哩哔哩\_bilibili](https://www.bilibili.com/video/BV1YAtReqEkH/?spm_id_from=333.337.search-card.all.click&vd_source=ec61df0d63e147463c27541f414a804d)
- [实用的 Cursor 使用技巧分享\_哔哩哔哩\_bilibili](https://www.bilibili.com/video/BV1RBm2YJEb9/?spm_id_from=333.337.search-card.all.click&vd_source=ec61df0d63e147463c27541f414a804d)
