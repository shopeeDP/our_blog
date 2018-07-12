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

## 基本
#### config
```
查看git配置
git config --list

编辑git配置
git config -e [--global]

设置用户信息
git config [--global] user.name "[name]"
git config [--global] user.name "[emali]"
```

#### status
```
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
git diff --shortstat "@{0 day ago}
```

## 本地
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
git branch --track [branch] [remote-branch]
git branch --set-upstream [branch] [remote-branch]
```

#### 

## 远程
