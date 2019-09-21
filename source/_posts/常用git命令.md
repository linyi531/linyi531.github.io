---
title: 常用git命令
date: 2018-09-16 11:50:00
tags:
  - git
categories: git
cover_img: https://tva1.sinaimg.cn/large/006y8mN6ly1g77gxs21fbj30u0140b2d.jpg
feature_img: https://tva1.sinaimg.cn/large/006y8mN6ly1g77gxs21fbj30u0140b2d.jpg
---
# 常用git命令
![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6j61mtymjj31e80qu3yu.jpg)
红色代表工作区，绿色代表暂存区

## 文件操作

1. git init  
   在当前目录下新建一个 git 仓库（master 分支）
   git init [project-name]
   新建一个目录，将其初始化为 git 仓库

   <!-- more -->

2. git status 查看状态
   “.” 代表文件夹中所有文件

3. git add [file1][file2]  
   添加指定文件到暂存区

4. git commit
   在第一行写入这次修改记录。将缓存区文件放入提交区。
   git commit -m ['message']
   git commit -amend
   修改上次记录信息

5. git log
   查看提交记录
   vim ~/.zshrc 配置文件中可自定义操作
   source ~/.zshrc 修改配置文件后 source 保存生效

6. git config
   查看当前 git 配置
   git config -h
   查看帮助信息
   vim ~/.gitconfig 自定义 git 命令，修改个人名字邮箱等信息
   cat .git/config 本地配置

7. touch .gitignore 忽略一些文件

```javascript
 .vscode
 node_modules
```

写入 gitignore 后即可忽略 node_modules 文件
更多信息查看[gitignore](https://github.com/github/gitignore)

8. git diff
   现实暂存区和工作区的差异

9. git checkout --a
   放弃 a 的变更
   依照提交区恢复工作区的文件，丢弃工作区的变更

10. git reset HEAD --a
    从暂存区恢复到工作区

11. git stash
    把工作区和暂存区的文件都存入 stash 中
    git stash list
    查看 stash 中的文件
    git stash pop
    恢复 stash 中的文件到工作区（pop=apply+drop）

12. git reset HEAD^
    后退一步（几个^代表后退几步）
    git reset HEAD~[number]
    抛弃了 number 个 commit

13. git reflog
    查看近期的 log 记录
    git reset [版本号]
    回退到版本号为……的 commit

## 分支操作

1. git branch develop
   创建 develop 分支,但依然停留在当前分支
   git branch -v
   查看分支

2. git checkout develop
   切换进入 develop 分支
   git checkout -b feature
   创建并切换进入 feature 分支

3. 合并分支
   a. 先进入要合并的分支（checkout develop）
   在执行 git merge feature 即可把 feature 合并到 develop 分支上
   b. git merge feature develop
   可达到同样的效果

4. 改变基线
   git rebase -i [提交记录号]
   将 HEAD 指向记录号所在位置

## 远程仓库

1. git remote add origin [SSH 地址]
   创建远程仓库连接

2. git push -u origin develop
   上传 develop 分支到远程仓库上（远程无项目可直接 push，有项目先 merge 再 push）

3. git pull origin feature
   拉取远程 feature 分支（pull=fetch+merge）

4. git brach -d feature
   删除 feature 分支

5. git push origin :feature
   删除远程 feature 分支

6. git tag [标签号]
   git push origin [标签号]
   打标签

7. git remote remove origin
   取消本地目录下关联的远程仓库

8. git clone [url]
   下载一个项目和它的整个代码历史

## 打 tag

1. vim package.json
   (vim package-lock.json)
   可以查看 version 号

2. npm version -h
   查看这一个 tag 即将提升的版本号（大版本或者小版本）

3. npm version patch
   提升 patch 这个小版本（v0.2.2）

4. git tag --list
   查看 tag 的列表

5. git push origin master
   push 代码

6. git push origin v0.2.3
   push tag
