---
title: git常用命令
date: 2018-07-11 16:09:16
tags:
- git
categories:
- 组内分享
description: 一些常用的git命令
---

# 常用git命令

## 一、基本
#### config
```
查看git配置
git config --list

编辑git配置
git config -e [--global]

设置用户信息
git config [--global] user.name "[name]"
git config [--global] user.email "[emali]"
```

#### status
```
显示工作目录和暂存区的状态
git status
```

#### log
```
git log
git log --stat
git log -S [keyword]
git log -p [file]
git shortlog -sn
```

#### diff
```
git diff
git diff [branch 1] [branch 2]
git diff --shortstat "@{0 day ago}"
```

## 二、本地
#### add
```
git add [file...]
git add [path]
git add .
git add -u
git add -A
```

#### commit
```
git commit -m "[your message]"
git commit --amend
git commit -a
git commit -v
```

#### branch
```
git branch
git branch -a
git branch -r
git branch [branch-name]
git branch -m [old branch] [new branch]
git branch -f [branch old place] [new place]
git branch --track [branch] [remote-branch]
git branch --set-upstream [branch] [remote-branch]
git branch -dr [remote/branch]
```

#### checkout
```
git checkout [branch]
git checkout -b [new branch]
git checkout -
```

#### cherry-pick
```
git cherry-pick [commit]
```

#### rebase
```
git rebase [upstream] [branch]
git rebase -i
```

## 三、远程
#### clone
```
git clone
```

#### remote
```
git remote -v
git remote show [remote]
git remote add [new name] [url]
```

#### fetch
```
git fetch
```

#### pull
```
git pull
git pull [remote] [source]:[destination]
git pull --rebase
```

#### push
```
git push
git push 
git push [remote] [source]:[destination]
git push origin --delete [branch-name]
git push [remote] --force
git push [remote] --all
```

## 四、撤销
#### checkout
```
git checkout .
```

#### commit
```
git commit --amend
```

#### reset
```
git reset [file]
git reset --hard
git reset [commit]
git reset --hard [commit]
git reset --keep [commit]
```

#### revert
```
git revert [commit]
```

#### stash
```
git stash
git stash pop
```

| 作者：[Dk](https://github.com/Darkindom)