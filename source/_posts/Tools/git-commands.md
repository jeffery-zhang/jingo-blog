---
title: Git 常用命令备忘
description: Git 常用命令备忘
date: 2022-10-13 14:43:24
categories:
  - Tools
tags:
  - Git
  - 命令行
  - 常用命令
---

# Git 常用命令备忘

Git 常用命令备忘

## Git 配置

### 用户信息

```sh
git config --global user.name "your name"
git config --global user.email "your@email.com"
```

### 查看配置信息

```sh
git config --list
git config -l
```

### Git 本地代理

```sh
git config --global https.proxy http://127.0.0.1:1080

git config --global https.proxy http://127.0.0.1:1080

# unset
git config --global --unset http.proxy

git config --global --unset https.proxy
```

## 常见状态

1. untracked: 未被 git 跟踪的文件
2. unmodified: 未被修改的文件
3. modified: 已修改的文件
4. staged: 修改已被暂存的文件

## 仓库管理

### Git 仓库初始化

```sh
git init
```

### 克隆远程仓库

```sh
git clone https://github.com/your-name/your-repo

# 克隆特定分支
git clone -b dev https://github.com/your-name/your-repo
```

### 检查文件状态

```sh
git status
```

### 暂存文件

暂存的功能:

1. 把已跟踪的文件放入暂存区
2. 开始跟踪新文件
3. 合并时把有冲突的文件标记为已解决状态

```sh
git add <file>
```

### 查看差异

```sh
git diff # 对比工作目录中当前文件和暂存区中文件差异
git diff --staged # 查看已暂存文件
```

### 提交更新

将暂存区快照进行提交

```sh
git commit # 直接输入会默认打开 vim 编辑器输入提交描述
git commit -a # -a 参数会自动将已跟踪文件暂存并提交
git commit -m 'bugfix' # -m 参数为提交加上描述
```

### 提交历史

```sh
git log
```

## 撤销操作

### 修改最后一次 commit

```sh
git commit -m bugfix
git add <file>
git --amend # 将 commit 之后 add 的文件加入到上次提交中
```

使用 --amend 参数修改 commit 不会创建一个新的 commit

### 取消暂存

```sh
git reset <file>
```

### 撤销修改

```sh
git checkout <file> # 撤销到 staged 状态
```

### 销毁暂存

```sh
git clean -f # 删除未暂存文件
git clean -fd # 删除未暂存文件和目录
git clean -fxd # 删除未暂存文件和目录, 以及 gitignore 中标记的未跟踪文件/目录
```

```sh
git checkout -f # 丢弃暂存区中所有文件, 慎用!!!
```

### 版本回退

```sh
git reset --hard HEAD^
git reset --hard [commit_id]
```

### 版本回退保留更改

```sh
git reset --soft HEAD^
git reset --soft [commit_id]
```

## 远程仓库

### 查看远程仓库

```sh
git remote # origin 是默认名称
git remote -v # 显示 url
git remote show [remote-name] # 显示远程仓库详细信息
```

### 添加远程仓库

```sh
git remote add [remote-name] <url> # 添加一个叫 remote-name 的远程仓库, 地址为 url
```

### 拉取

```sh
git fetch [remote-name] [branch]
```

通过 fetch 拉取远程仓库会在本地拥有该仓库所有分支或指定分支的引用, 可以查看或合并, 但不会自动合并或修改当前工作

```sh
git pull
git branch --set-upstream-to=[remote-name]/[remote-branch] [local-branch] # 为还未设置远程分支的当前分支设置对应远程分支并拉取
```

拉取远程分支到当前分支, 并进行自动合并

### 推送

```sh
git push
git push -u [remote-name] [branch] # 为还未设置远程分支的当前分支设置对应远程分支并推送
```

### 远程仓库的移除和重命名

```sh
git remote rename [old-remote-name] [new-remote-name] # 重命名
git rm remote [remote-name] # 移除
```

## 标签

### 轻量标签

```sh
git tag <tag name> # 仅包含标签名信息
```

### 附注标签

```sh
git tag -a <tag name> # 包含打标签者名字, 邮箱, 日期等信息
```

### 推送标签

```sh
git push --tags # --tags 参数会将所有未推送的标签推送到远程仓库
```

## 分支管理

### 创建分支

```sh
git branch [branch-name]
git checkout -b [branch-name]
```

### 查看分支

```sh
git branch
```

### 切换分支

```sh
git checkout [branch-name]
```

### 删除分支

```sh
git branch -d [branch-name] # 删除本地分支
git push [remote-name] -d [branch-name] # 删除远程仓库分支
```

### 修改本地分支名称

```sh
git branch -m [old-branch-name] [new-branch-name]
```

### 合并分支

```sh
git merge [branch-name]
```

合并时整合提交记录, 避免过多分叉

```sh
git rebase [branch-name]
```

## Gitignore

使用 npm 开源包 [ggig](https://github.com/jeffery-zhang/ggig) 可以方便的自动生成各种不同语言的 gitignore 文件

### 安装 ggig

```sh
npm install -g ggig
```

### 查看支持的语言

```sh
ggig ls

# AL, Actionscript, Ada, Agda, Android, AppEngine, AppceleratorTitanium, ArchLinuxPackages, Autotools, Ballerina, C, C++, CFWheels, CMake, CUDA...
```

### 创建指定语言的 gitignore 文件

```sh
ggig gen --template Java
# or
ggig gen -t Java
```
