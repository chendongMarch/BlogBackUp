---
layout: post
title: Git 命令
categories:
  - bash
tags:
  - Git
abbrlink: b51d9f8e
date: 2017-08-10 16:50:00
location: 杭州
photos: http://olx4t2q6z.bkt.clouddn.com/18-2-2/60301604.jpg
---

Git 日常命令收集，对平常使用的 `git` 命令及版本管理中遇到的问题进行梳理。

文中 `{ }` 包含的部分表示变量参数，如 `{git-url}` 指的是远程 `git` 地址，`[ ]` 包含的部分指的是这部分命令是可以省略的。

<!--more-->

## git config 用户配置查看及更改

```bash
# 查看
git config user.name
git config email

# 配置
git config --global user.name = 'test'
git config --global user.email = 'test@qq.com'
```

## git init 

```bash
git init
git remote add origin git@github.com:chendongMarch/xxxx.git
git add fileName
git add -A
git commit -m "first commit"
git push -u origin master
```


## git clone

```bash
git clone [-b {branch-name}] {git-url} [{dest-dir}] [--depth 1]

# 只 clone 最近一次 commit，防止提交太多 clone 很慢
git clone {git-url} --depth 1
```

## git remote 查看和更改远程

```bash
# 查看远程仓库名
git remote -v

# 添加 origin 仓库
git remote add origin {git-url}

# 删除 orgin 仓库
git remote rm origin

# 重置 orign 指向的仓库
git remote set-url origin {git-url}
```

## git branch 分支

```bash
# 删除分支
git branch -d {branch-name}

# 新建分支
git branch {branch-name}

# 切换分支
git checkout {branch-name}
```

## git stash 暂存

暂存已经 `add` 的修改，可以选择在其他合适的时机，将暂存提取应用到当前分支

```bash
git add -A # 首先应该执行 add 命令，才能执行 stash

git stash # 暂存当前的修改

git stash list # 查看已经存在的暂存 
# 展示结果如下
# stash@{1}: WIP on master: 8d983f6 ...

git stash apply stash@\{0\} # 应用暂存
```

## git log 查看日志

```bash
git reflog # 查看命令历史

git log test.file # 查看单个文件的日志
```

## git reset 重置

```bash
git reset --hard HEAD^ # 重置到上一次提交

git reset --hard HEAD~10 # 重置到之前的10次提交

git reset --hard {commitId} # 重置到这次 commit
```

## git rm

删除远程文件夹

```bash
git rm -r --cached {dirname}
git commit -m {msg}
git push origin master
```

更改 `.gitignore` 后无法生效的问题，原因是 `.gitignore` 只能忽略那些原来没有被 `track` 的文件，如果某些文件已经被纳入了版本管理中，则修改 `.gitignore` 是无效的。那么解决方法就是先把本地缓存删除，改变成未 `track` 状态，然后再提交

```bash
git rm -r --cached .
git add .
git commit -m 'update .gitignore'
```

## git gc 仓库清理

```bash
git gc
git gc --aggressive
```


## Fix


解决 `fix fatal: cannot do a partial commit during a merge.` 的问题，这个问题是出现在合并代码时出现问题，分支一直处于 `merging` 状态无法提交的问题。

```bash
git commit -i * -m "message"
```

解决 `Unable to create '.git/index.lock': File exists` 问题，删除 `lock` 文件

```bash
rm -f ./.git/index.lock
```
