---
title: git常用命令及作用
date: 2021-04-05 11:27:17
tags: Git
categories: Git操作
---

## 网站链接

https://www.liaoxuefeng.com/wiki/896043488029600/897013573512192

## 1. 初始化

- git init 将某个目录变成git可管理的仓库，隐藏的目录可以通过ls -ah看到隐藏目录
- git add readme1.md readme2.md 将某个文件添加到仓库，可以写多个文件，空格分隔
- git commit -m 'some thing' 把文件提交到仓库，-m后面跟随提交内容的说明
- git status 当前仓库的状态
- git diff 修改的内容比较

## 2. 穿梭

Git中HEAD表示当前版本，上个版本就是HEAD^，上上一个就是HEAD^^，向上一百个写成HEAD～100

- git log 历史记录，参数--pretty=oneline可以缩减输出信息
- git reset --hard HEAD^ 当前版本回退到上一个版本，当然HEAD^也可以被替换成commit id
- git reflog 记录每一次的命令
- git diff HEAD -- readme.txt 查看工作区和版本库里面最新版本的区别
- git checkout -- readme.txt 把某个文件在工作区的修改撤销，没有--表示切换到另一个分支
- git reset HEAD readme.txt 把暂存区的修改撤销重新放回工作区
- git rm 删除版本库中的文件

## 3. 远程仓库

github为例，本地git和github之间是通过ssh加密的，所以需要一点对应的设置

创建ssh key，首先看用户主目录有没有.ssh目录，然后再看这个目录下有没有id_rsa和id_rsa.pub这两个文件，有的话跳转到下一步，没有的话进行对应的创建。id_rsa是私钥，id_rsa.pub是公钥

```ssh
ssh-keygen -t rsa -C "youremail@example.com"
```

- git remote add origin git地址 把本地仓库关联到远端某个仓库地址
- git push -u origin master 把本地仓库所有的内容推到远程，-u表示将本地的master分支和远程的master分支关联，以后推送可以简化
- git remote -v 查看远程库信息
- git remote rm origin 根据名字删除origin
- git clone git地址 克隆远端仓库

## 4. 分支管理

- git checkout -b dev 创建并切换到dev分支，也可以使用git switch -c dev
- git branch dev 创建dev分支
- git checkout dev 切换到dev分支，也可使用 git switch dev
- git branch 查看当前分支
- git merge dev 当前分支合并某个分支
- git branch -d dev 删除dev分支
- git merge --no-ff -m 'merge with no-ff' dev 禁用fast forward，-m创建一个新的commit
- git stash 把当前工作现场储藏起来
- git stash list 查看stash存储的内容
- git stash apply stash@{0} 恢复，恢复之后stash内容并不删除
- git stash drop 删除stash存储的内容
- git stash pop 恢复的同时也删除
- git cherry-pick 4c805e2 复制一个特定的提交到当前分支
- git branch -D name 强行删除某一个未合并分支
- git remote -v 查看远程仓库信息
- git branch --set-upstream-to=origin/dev dev 指定本地分支与远程origin/dev分支的链接
- git rebase 把本地未push分叉的提交历史整理成一条直线

## 5. 标签管理

标签作为一个版本库的快照

- git tag name 打一个标签，使用git tag查看所有标签，默认标签是打在当前的commit上面的
- git tag v0.9 f52c633 给对应的commit打对应的标签
- git tag -d v0.9 删除某个标签
- git push origin tagname 推送某个标签到远程
- git push origin --tags 推送所有标签
- git push origin :refs/tags/v0.9 先删除本地的内容，再删除远端的内容

## 6. 配置管理

- git config --global color.ui true 显示颜色
