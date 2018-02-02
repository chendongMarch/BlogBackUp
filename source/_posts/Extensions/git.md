---
layout: post
title: Git 命令
categories:
  - Git
tags:
  - Extensions
  - Git
abbrlink: b51d9f8e
date: 2017-08-10 16:50:00
location: 杭州
photos: 
---

Git 日常命令收集。

对平常使用的 git 命令及版本管理中遇到的问题进行梳理。

<!--more-->


更改 `.gitignore` 后无法生效的问题，原因是 `.gitignore` 只能忽略那些原来没有被 `track` 的文件，如果某些文件已经被纳入了版本管理中，则修改 `.gitignore` 是无效的。那么解决方法就是先把本地缓存删除，改变成未 `track` 状态，然后再提交

```bash
git rm -r --cached .
git add .
git commit -m 'update .gitignore'
```

解决 `fix fatal: cannot do a partial commit during a merge.` 的问题，这个问题是出现在合并代码时出现问题，分支一直处于 `merging` 状态无法提交的问题。

```bash
git commit -i * -m "message"
```

解决 `Unable to create '.git/index.lock': File exists` 问题，删除 `lock` 文件

```bash
rm -f ./.git/index.lock
```

删除远程文件夹

```bash
git rm -r --cached dirname
git commit -m 'say something'
git push origin master
```

仓库清理

```bash
git gc
git gc --aggressive
```

初始化仓库

```bash
git init
git remote add origin git@github.com:chendongMarch/xxxx.git
git add fileName
git add -A
git commit -m "first commit"
git push -u origin master
```

用户配置查看及更改

```bash
git config user.name
git config email

git config --global user.name = 'test'
git config --global user.email = 'test@qq.com'
```

git remote

```bash
// 查看远程仓库名
git remote -v


// 添加 origin 仓库
git remote add origin [url]

// 删除 orgin 仓库
git remote rm origin

// 重置 orign 指向的仓库
git remote set-url origin [url]
```