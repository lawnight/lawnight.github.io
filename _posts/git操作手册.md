---
title: git操作手册
categories: server
date: 3-2
---
列举一些git常用，又老是忘记的操作。
## 命令行
```bash
git branch -a               #列举分支，包括远端
git branch -vv              #当前分支以及定义的远端分支

git diff > patch
git diff --no-index  "c++.md" "c2.md" #比较本地两个文件
```