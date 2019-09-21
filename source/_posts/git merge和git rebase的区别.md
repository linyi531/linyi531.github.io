---
title: git merge和git rebase的区别
date: 2019-09-21 22:53:17
tags:
  - git
categories: git
cover_img: https://tva1.sinaimg.cn/large/006y8mN6ly1g77j2diqxtj30u00u0wzn.jpg
feature_img: https://tva1.sinaimg.cn/large/006y8mN6ly1g77j2diqxtj30u00u0wzn.jpg
---

# git merge和git rebase的区别

git rebase 和 git merge 一样都是用于从一个分支获取并且合并到当前分支，但是他们采取不同的工作方式。

![img](https://tva1.sinaimg.cn/large/006y8mN6ly1g6wkw4uh2dj30xc0qtjsl.jpg)

为了将master 上新的提交合并到你的feature分支上，你有两种选择：`merging` or `rebase`

## merge

执行以下命令：

```shell
git checkout feature
git merge master
```

或

```shell
git merge master feature
```

那么此时在feature上git 自动会产生一个新的commit(merge commit)

![img](https://tva1.sinaimg.cn/large/006y8mN6ly1g6wky69tunj30w90850te.jpg)

### `git merge` 有如下特点：

- 只处理一次冲突，如果合并的时候遇到冲突，仅需要修改后重新commit
- 引入了一次合并的历史记录，合并后的所有 `commit` 会按照提交时间从旧到新排列
- 所有的过程信息更多，可能会提高之后查找问题的难度

### 优点：

记录了真实的commit情况，包括每个分支的详情

### 缺点：

因为每次merge会自动产生一个merge commit，所以在使用一些git 的GUI tools，特别是commit比较频繁时，看到分支很杂乱。

## rebase

与 `git merge` 一致，`git rebase` 的目的也是将一个分支的更改并入到另外一个分支中去。

```shell
git checkout feature
git rebase master
```

本质是**变基 变基 变基**

变基是什么? `找公共祖先`

![img](https://tva1.sinaimg.cn/large/006y8mN6ly1g6wl0azimrj30wx0823zd.jpg)

### rebase 特点：

- 改变当前分支从 `master`上拉出分支的位置
- 没有多余的合并历史的记录，会合并之前的commit历史，且合并后的 `commit`顺序不一定按照 `commit`的提交时间排列
- 可能会多次解决同一个地方的冲突（有 `squash`来解决）
- 更清爽一些，`master`分支上每个 `commit`点都是相对独立完整的功能单元

### 优点：

得到更简洁的项目历史，去掉了merge commit

### 缺点：

如果合并出现代码问题不容易定位，因为rewrite了history

### 合并时遇到冲突：

合并时如果出现冲突需要按照如下步骤解决

- 修改冲突部分
- git add
- `git rebase --continue`
- （如果第三步无效可以执行 `git rebase --skip`）

不要在git add 之后习惯性的执行 git commit命令

### git rebase 的交互模式

打开变基的交互模式只需要传入一个参数 `-i` 即可，同时还需要指定对哪些提交进行处理

```shell
git rebase -i HEAD~4
```

上述命令指定了对当前分支的最近四次提交进行操作。下面我们使用上面这行命令将 `feature` 分支的提交合并。

![img](https://tva1.sinaimg.cn/large/006y8mN6ly1g6wl7ji36xj30wi0gpq67.jpg)

中间红框内有一些命令，可以用来处理某次提交的

## 总结

- 如果你想要一个干净的，没有merge commit的线性历史树，或者当发现自己修改某个功能时，频繁进行了`git commit`提交时，发现其实过多的提交信息没有必要时，那么你应该选择git rebase
- 当需要保留详细的合并信息的时候建议使用`git merge`，特别是需要将分支合并进入`master`分支时，并且想要避免重写commit history的风险，你应该选择使用git merge