---
title: git常用操作命令
date: 2022-07-01 11:39:10
tags:
 - git
---

## 工作区
### a、删除未跟踪的文件 untracked files
```sh
# 查看需要删除的文件
git clean -nfd

# 确认删除文件
git clean -fd

# 连gitignore的untrack 文件/目录也一起删掉 
#（慎用，一般这个是用来删掉编译出来或者node_modules的之类的文件用的）
git clean -xfd
```

### b、删除已跟踪的文件
```sh
git checkout .
```

## 暂存区
### a、取消暂存区已缓存的内容，修改前内容还在工作区
```sh
git reset HEAD 
```

### b、撤销丢弃所有文件修改，内容不在工作区
```sh
git reset --hard HEAD
```

## 本地仓库
### a、回退暂存区相关文件到上一个版本，内容还在工作区
```sh
git reset HEAD^ #回退所有内容到上一个版本
git reset HEAD^ hello.php  # 回退hello.php文件的版本到上一个版本
git reset commitID hello.php # 回退hello.php文件的版本到某一个版本
```

### b、回退暂存区相关文件到上一个版本，内容不在工作区
```sh
git reset HEAD^ --hard
```
## 远程仓库
>git reset 是把HEAD向后移动了一下，而git revert是HEAD继续前进

### a、撤销revert
```sh
# git 日志会新增一条记录
git revert commitID (撤销指定的版本) 
git push 
```

### b、回退reset
```sh
# git日志不会新增一条记录，
# 反而会把commitID之后的日志给删掉
git reset commitID (HEAD指向版本)
git push -f #此时如果用“git push”会报错，因为我们本地库HEAD指向的版本比远程库的要旧
```

## 分支
### 创建新的分支，且关联远程
```sh
git checkout -b branch_name origin/branch_name
```

### 本地没有分支，远程有
```sh
#自动跟踪远程的同名分支 branch_name
git checkout --track origin/branch_name
```

### 本地有分支，远程没有
```sh
git push --set-upstream origin branch_name
```

### 删除本地分支
```sh
git branch -d branch_name
```

### 删除远程分支
```sh
git branch -r -d origin/branch-name  
git push origin :branch-name 
```

### dev分支合并到prod分支
```sh
# 切换到prod分支
git checkout prod

# 拉取prod最新代码
git pull origin prod

# dev分支合并到prod
git merge dev

# 查看状态
git status
```