# git 基本使用

## 提交 （ commit ）

`git add .` 加入所有文件到暂存区

`git add <fileName>` 将指定文件加入到暂存区

`git reset <fileName>` 将指定文件从暂存区移除

## 文件状态

* Untracked 新建文件后
* Unmodified: commit 后
* Modified: 有过commit记录的文件再修改
* Staged:  git add 后

## 回退 （ reset ）

`git reset <commitID>` 将现有文件恢复到当时提交的状态

`git reflog` 查看所有的操作记录，可以查到之前的 `commitID`

如果想回到当前分支最新的commit，也可以使用 `git pull`

### reset 的模式

* --hard: 不保存所有变更
* --sort: 保留变更且变更内容处于Staged
* --mixed: 保留变更且变更内容处于Modified (不带参数，默认就是mixed)

## 分支 （ branch ）

`git checkout -b <name> <template>` 创建分支，其中name表示新分支的名字，template表示以哪个分支或者commit为模板

如果 template 不是本地的，是来自远程仓库的话，则需要在 template 前面加个 origin

`git checkout -b <name> origin <template>`

例如： `git checkout -b bc-b origin bc-a` 的意思就是创建以远程的bc-a分支为模板的bc-b分支

![image-20220202105320435](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20220202105320435.png)

模板：在一个分支中，一次次的commit都会被git记录，以这个分支为模板新切一个分支，新分支的commit记录会继承自模板分支，但是不同的分支之间没有关系

![image-20220201195048944](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20220201195048944.png)

![image-20220201195104717](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20220201195104717.png)

`git checkout <name>` 切换分支

`git branch` 查看所有分支

## 合并 （ merge ）

`git merge <name>` 合并分支，name表示分支名称

如果不同分支修改相同文件，可能会产生冲突，此时需要手动解决冲突再提交

## 远程仓库 （ remote ）

`git push --set-upstream origin <name>` 推送到远端分支并且将远端分支设置为上流分支，name表示分支名

`git fetch` 拉取远程仓库信息

`git pull` 等同于 先 `git fetch` 再 `git merge`

## 变基 （ rebase ）

使用 `git rebase <name>` 实现变基，发现产生冲突的话就解决冲突，然后将变更加入暂存区，接着使用 `git rebase --continue` 继续下一个节点的rebase

rebase 的原理就是枚举变更的 commit，依次变基

![image-20220202110517443](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20220202110517443.png)

比如这个例子：如果此时我们在bc分支上，需要以最新的 master 版本为 base 提交 commit

使用 rebase，先拿出3号commit，然后以125为基础添加3号commit，再拿出4号commit，以1253为基础添加4号commit，直到所有的新增commit都完成变基
