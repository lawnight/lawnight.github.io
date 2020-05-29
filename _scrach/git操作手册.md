---
title: git操作手册
categories: server
date: 3-2
---
列举一些git常用，又老是忘记的操作。

## 存储原理

git是分布式，本地.git存储有所有版本信息。

主要版本信息存储在`.git/objects`中。分为四种对象模型：分别是 Blob，Tree， Commit， Tag 他们都用 SHA-1 进行命名。
blob:单个文件内容，不包括元数据。
Tree:目标快照，文件名和blob的sha-1 对应起来。
commit：提交日志，当前commit对应的tree

文件提交时生成sha-1编码，并进行zlib压缩生成blob文件。（压缩率测试大概有60%）

可以用`git cat-file`看文件里的内容

### 压缩

git提供了pack机制，将所有object下的对象文件，计算deleta，压缩到单个文件中。减少了上传，下载的网络消耗，也能节省磁盘空间。

pack后会生成一个pack文件和一个idx文件。idx文件有对应object在pack中的偏移，加快信息检索。

所以在`git pull`和`git push`的时候，会自动触发`git gc`，也可以通过`git gc`来手动触发pack。

git提供了丰富的配置来控制这个行为

- `git config --global gc.auto 0` 关闭delta压缩
- `core.bigFileThreshold` 大于这个size的文件，不会尝试用delta压缩，默认为500m。





## 命令行
```bash
GIT_TRACE=1 git gc                                  # 显示命令执行的详细信息
git branch -a                                       #列举分支，包括远端
git branch -vv                                      #当前分支以及定义的远端分支

git diff > patch
git diff --no-index  "c++.md" "c2.md"               #比较本地两个文件

git merge -X theirs develop -m 'merge from xxx'     #用develop的版本解决冲突的版本
git cat-file -p master^{tree}                       #看master指针对应的tree内容
git verify-pack -v .git/objects/pack/pack-978e03944f5c581011e6998cd0e9e30000905586.idx #看pack内容
```