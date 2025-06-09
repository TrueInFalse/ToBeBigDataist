
## Git分支管理标准操作流程

### 一、在dev分支开发

1. 确保在dev分支
```shell
git branch        # 查看当前分支
git checkout dev  # 如果不在dev分支，切换到dev
```

2. 开发前先拉取最新代码

```shell
git pull origin dev
```

3. 开发完成后，查看更改
```shell
git status
git diff        # 查看具体更改内容
```
4. 提交更改到 dev

```shell
git add .
git commit -m "feat: 更新内容描述"
git push origin dev
```
### 二、合并到main分支

1. 切换到 main 分支
```shell
git checkout main
```



2. 拉取 main 分支最新代码
```shell
git pull origin main
```

3. 合并 dev 分支
```shell
git merge dev
```


4. 如果有冲突


```shell
# 1. 打开冲突文件
# 2. 解决冲突（选择保留哪些代码）
# 3. 标记解决
git add .
# 4. 提交合并
git commit -m "merge: dev into main"
```


5. 推送到远程

```shell
git push origin main
```

### 三、常见错误处理

1. 如果合并出错想重来

```shell
git merge --abort
```
2. 如果提交出错想撤销

```shell
git reset --soft HEAD^  # 撤销上一次提交，保留更改
```


3. 如果想放弃所有更改

```shell
git reset --hard HEAD  # 警告：会丢失所有未提交的更改
```

### 四、建议

- 每次切换分支前先 git status 检查是否有未提交的更改
- 合并前确保两个分支都是最新的
- 养成写清晰的提交信息的习惯
- 遇到复杂冲突可以使用 git mergetool 工具