---
title: 在游戏中学会git相关操作
date: 2021-04-05 11:27:17
tags: Git
categories: Git操作
---

## 网站链接

https://learngitbranching.js.org/
https://www.liaoxuefeng.com/wiki/896043488029600/897013573512192

## 1. 初始化

- git init 将某个目录变成git可管理的仓库，隐藏的目录可以通过ls -ah看到隐藏目录
- git add readme1.md readme2.md 将某个文件添加到仓库，可以写多个文件，空格分隔
- git commit -m 'some thing' 把文件提交到仓库，-m后面跟随提交内容的说明
- git status 当前仓库的状态
- git diff 修改的内容比较
- git log 历史记录，参数--pretty=oneline可以缩减输出信息

