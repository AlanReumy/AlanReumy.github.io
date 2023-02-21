# github 工作流

1. git clone
2. 建立本地新的 feature branch：git checkout -b my-feature
3. 修改并提交
4. git checkout main -> git pull origin main
5. git checkout my-feature -> git rebase main
6. git push -f origin my-feature
7. new pull request
8. squash and merge：合并多次提交
9. git branch -D my-feature
10. git pull origin master 拉取最新的更新
