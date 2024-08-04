# 百度实习记录

## 百度的 git 工作流

1. 提交代码，发起 code review: git push origin `branchName`:refs/for/`branchName`
2. 在 feature 分支上开发完毕后，合并到测试分支：
   - `git checkout qa_test`
   - `git merge branchName`
3. 如果测试出现 bug，则新建一共`branchName-fix`分支，进行 bug 修复的代码提交
4. 修改后，合并到 `qa_test` 分支
5. 测试通过后，合入到 `master` 分支: `git merge branchName`
6. 进入灰度
