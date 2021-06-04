---
title: electron相关笔记总结
date: 2021-04-28 19:40:43
tags: electron
categories: electron 桌面端
---

## 介绍

基础篇，工程篇以及实战。web技术构建跨平台桌面应用，chromium+nodejs+nativeAPI跨平台

如何判断某个项目是否是electron开发。首先进入项目目录，cd Contents目录，进入Frameworks目录，可以看到Electron Framework

![识别Electron](https://gitee.com/RenYaNan/wx-photo/raw/master/2021-4-28/1619611412121-shElectron.png)

Electron的最小组成，由渲染进程，包体描述，主进程组成。

Chromium架构多进程，每个页面对应着渲染进程，Browser是主进程。node集成到chromium

NW可以兼容xp系统

## 安装

npm install electron --save-dev

npm install --arch=ia32 --platform=win32 electron

npx electron -v



