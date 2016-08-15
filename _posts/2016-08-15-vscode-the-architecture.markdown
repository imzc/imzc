---
layout: post
title:  "Visual Studio Code / Egret Wing 技术架构：基础"
date:   2016-08-15 22:42:35 +0800
categories: dotnet
---

## 系列目录
- [核心架构](2016-08-15-vscode-the-architecture.markdown)
- [插件系统](2016-08-16-vscode-the-extensions.markdown)
- [Wing 对 VS Code 的扩展](2016-08-17-wing-vs-vscode.markdown)

## VS Code 简单介绍

Visual Studio Code (下面简称VSC) 是由微软公司开发的开源、免费、跨平台的代码编辑器，微软希望它在保持核心轻量化文本编辑器的基础上，为编辑器添加项目支持、智能感知和编译调试。

VSC Team 由著名工程师 Erich Gamma 领导，Erich 是《设计模式》作者之一，Eclipse 之父，拥有多年的 IDE 开发经验。 

![Erich Gamma](images/erich.jpg)

VSC 的前身是微软基于云端的编辑器项目：Monaco 编辑器，它作为微软云服务的一部分，提供在线编辑源代码的能力。

![Monaco 编辑器](images/monaco.png)

由于云端编辑器的种种限制，和微软近年来对Windows外平台的态度转变，微软决定由它扩展开发为一个全平台通用的代码编辑器。

## VSC 技术路线

### Electron

为了保护本地文件的安全性，浏览器都没有提供直接的本地文件访问权限。为了实现本地文件系统的访问，VSC 采用了 Github 的开源项目 Electron。

Electron 原名 Atom-Shell，是 Github 为 Atom 编辑器编写的一个开源框架。它将 Chromium 和 Node.js 完美融合，让开发者能用使用 HTML 来实现 App UI，用 Node.js API 来访问文件系统。

### TypeScript

VSC 的主要代码都是用 TypeScript 编写，目前 VSC 的核心有 1100 多个 TS 文件，TypeScript 的语言优势为多次重构提供了保障。

### 纯 DOM 操作

为了保证 UI 响应速度，VSC 没有采用现有的 UI 库，大部分 UI 采用绝对尺寸，简单粗暴的避免大面积 UI 的联动刷新。

## VSC 实例的进程结构

VSC 采用多进程架构, VSC 可以拆分成几大块：

- 后台进程
- 编辑器窗口 - 由后台进程启动，也是多进程架构
    - 编辑器 UI 进程
        - HTML 编写的 UI
            - Activitybar
            - Viewlets
            - Panels
            - Editors
            - Statusbar
        - Nodejs 异步 IO
            - FileService
            - ConfigurationService
    - 插件宿主进程
    - Debug 进程
    - Search 进程

### 后台进程

后台进程是 VSC 的入口，主要负责管理编辑器生命周期，进程间通信，自动更新，菜单管理等。

我们启动 VSC 的时候，后台进程会首先启动，读取各种配置信息和历史记录，然后将这些信息和主窗口 UI 的 HTML 主文件路径整合成一个 URL，启动一个浏览器窗口来显示编辑器的 UI。后台进程会一直关注 UI 进程的状态，当所有 UI 进程被关闭的时候，整个编辑器退出。

此外后台进程还会开启一个本地的 Socket，当有新的 VSC 进程启动的时候，会尝试连接这个 Socket，并将启动的参数信息传递给它，由已经存在的 VSC 来执行相关的动作，这样能够保证 VSC 的唯一性，避免出现多开文件夹带来的问题。

### 编辑器窗口


VSC team 做了很多工作来提高 VSC 代码编辑器的性能。代码显示的虚拟化，让编辑器只渲染可见的部分，减小大文件编辑对浏览器的压力。