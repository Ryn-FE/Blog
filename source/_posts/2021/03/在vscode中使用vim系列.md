--
title: 在vscode中使用vim系列
date: 2021-03-19 23:58:43
tags: vim vscode
categories: vim
--

## 前沿

在vscode中使用vim增强我的编码能力，其实一直以来都很想学习vim的编码方式，虽然我也知道这是一个艰难的过程，但是总归是要面对的，既然这个过程很痛苦并且不是很容易就能够将一些内容熟练掌握，那么就从这里给自己进行一个对应的记录，有什么看不明白或者不清楚的地方可以有一个随时翻阅的地方。顺便提一下，这个文章是看的国外一篇文章翻译过来的，对应的参考资料在文章末尾有附带上，可自行查阅。

vim很强，因为它难

## 在vscode中安装vim

1. 在扩展中搜索vim
2. 排名第一的并安装它
3. 要知道cmd+shift+p打开命令面板
4. cmd+p跳转对应的文件

## vim基础

刚开始的矩形光标，你会发现你想要输入的时候没有任何反应。当然，在你按下键盘的时候，有可能会触发一些特殊含义的命令，使得矩形光标在屏幕上飞来飞去，甚至删除了一些对应的内容。

那么所有这些都源自于vim的正常模式（也就是vim与文本的默认模式），并且在这个模式下，不会进行文本的插入。

那么什么是普通模式，模式又是什么意思？

### vim中的模式

1. 普通模式NORMAL

![vim普通模式](https://gitee.com/RenYaNan/wx-photo/raw/master/2021-3-20/1616173392387-image.png)

如何进入: esc ctrl+c ctrl+[

四处移动并编辑文本或者跳转

w: 表示跳转到下一个单词
b: 向后跳到单词的开头
e: 跳转到单词的结尾
ge: 向后跳转到单词的末尾 
j: 光标向下移动
k: 光标向上移动
h: 光标向左移动
l: 光标向右移动

移动到特定角色

f{字符}: 后面跟上特定字符，移动到行中特定出现的那个字符上
t{字符}: 移动的位置比f前置一个字符
如果执行完搜索，可以使用;转到下一个字符，,转到上一个字符

极端水平移动
0: 移动到行的第一个字符
^: 移动到行的第一个非空白字符
$: 移动到行尾
g_: 移动到行尾的非空白字符

垂直移动
}: 向下跳整段
{: 向上跳
ctrl+D: 向下移动半页
ctrl+U: 滚动半页
/{字符}: 向前搜索，一个文件中
?{字符}: 向后搜索一个文件中
enter确定你搜索的内容，n跳转到下一个匹配的内容，N上一个，可理解为重复搜索

计数可以更快的移动
2w: 向前移动2个单词
3j: 向下移动三行

gg: 跳转到文件顶部
{line}gg: 跳转到特定的某一行
G: 文件末尾

2. 插入模式INSERT

插入文本

![vim插入模式](https://gitee.com/RenYaNan/wx-photo/raw/master/2021-3-20/1616173059353-image.png)

3. 可视模式VISAUL

选择文本

w: 表示选中当前光标到下一个单词之间的内容

## 参考资料

[1] vscode中加入vim [https://www.barbarianmeetscoding.com/boost-your-coding-fu-with-vscode-and-vim/table-of-contents](https://www.barbarianmeetscoding.com/boost-your-coding-fu-with-vscode-and-vim/table-of-contents)
