---
layout: post
title:  "Visual Studio Code / Egret Wing 技术架构：Wing 对 VS Code 的扩展"
date:   2016-08-17 22:42:35 +0800
categories: ide,vscode
---

## 系列目录
- [核心架构](/ide,vscode/2016/08/15/vscode-the-architecture/)
- [插件系统](/ide,vscode/2016/08/16/vscode-the-extensions/)
- [Wing 对 VS Code 的扩展](/ide,vscode/2016/08/17/wing-vs-vscode/)

Wing 的主要使命是进行可视化的游戏开发，但 VSC 现有的插件系统并不能满足可视化开发的需求，所以 Wing 对 VSC 进行了一系列扩展。

# Wing 扩展 VSC 遵循的原则

- 优先使用插件完成新功能
- 不改动 VSC 的核心架构
- 遵循 VSC 内部组件的注册和管理方式
- 保证 Wing 跟新版本 VSC 的兼容性

# UI 改动

VSC 的 UI 是由一个个 `Part` 组成的，每个 `Part` 由 `Workbanch` 统一管理，由 `WorkbanchLayout` 负责布局。每个 `Part` 功能独立，改变布局并不会对功能产生影响。添加新的 `Part` 可以为 Wing 添加新的功能比如右边栏和工具栏。

## 取消 ActivityBar

为了使左边栏，下边栏（输出、错误窗口）和右边栏风格统一，我们在 Wing 3 的第一个版本中取消了 VSC 的ActivityBar 将它挪到了左边栏的上方。同时为下边栏添加了切换按钮，方便切换不同的面板。

## 添加内置控制台

早期版本的 VSC 中没有内置的控制台，这让很多开发者感到不便。所以我们在 Wing 3.0 的第二个版本中添加了内置的控制台。内置控制台派生自 VSC 的 Panel，对 VSC 本身改动非常小。只需要简单添加引用然后注册即可。在今年中 VSC 也是添加了内置控制台的功能，经过几个版本的更新，现在已经支持中文和复制粘贴，Wing 3.1 目前已经将控制台还原到 VSC 内置的版本。

## 添加右边栏

右边栏在可视化编辑界面中非常重要，所以我们添加了右边栏这个 `Part`, 并将接口通过插件 API 开放给了开发者。此次插件大赛中多款产品都使用了右边栏的功能。

## Windows 下使用无边框窗口

VSC 在 Windows 系统下，标题栏和菜单栏占据了很大的空间，而且在深色主题下，整体风格不太统一。我们重新设计了顶部区域，将菜单栏和标题栏合二为一，整体风格更加统一了。 

## 添加工具栏

工具栏集成了常用的操作按钮，这一点也是向传统 IDE 的回归，降低入手门槛。

# Wing 对 插件 API 的扩展

## UI 扩展 API

VSC 的插件体系中没有 UI 相关的 API，让用户交互变得非常困难，为此 Wing 对插件 API 做的主要扩展就是添加 UI 相关的 API。现在 Wing 主要有两种 UI 展现方式。

### 数据驱动的 UI

你可以使用 `wing.window.showPopup<O>(type: PopupType, store?: Store, options?: O): Thenable<Store>` 来显示一个弹出窗体。`store` 为一个类似 JSON Schema 的 object，通过这个 object Wing 会自动创建一个表单 UI，展现给用户，用户完成输入后，将结果返回给插件。这种方式的交互，插件开发者可以不关心 Wing 的主题样式，无需编写 UI 即可完成简单的交互操作，缺点就是对 UI 的控制性比较弱。

![](http://edn.egret.com/cn/data/upload/asset/20160308/56de4ff73daba.png)

### 使用 WebView 编写 UI

在 WebView 中用户可以展示自己的 HTML page，可以完成非常复杂的 UI 交互。目前在 Wing 中，插件开发者可以在弹出窗口，代码编辑器区域和右边栏中使用 WebView 设计自己的界面。

由于 WebView 是单独的进程，WebView 进程中的代码可以使用 Wing 提供的 IPC 通信与插件主进程进行通信，将 UI 跟用户的编辑器操作连接起来。

Wing 内置的 HTTP Server 就是采用了 WebView 弹窗作为 UI 的插件，由插件进程启动 HTTP Server，当用户点击插件在状态栏中的 icon 时，插件进程启动一个WebView 来展示 Server 的状态。

![](http://edn.egret.com/cn/data/upload/asset/20160621/57693c4dce59f.gif)

## 模板扩展 API

项目模板也是非常常用的一个功能。Wing 为 VSC 添加了新建项目的向导。通过插件可以为这个向导添加项目模板。目前 Wing 内置的十几种项目模板都是由插件实现的。其中也包含了 Egret 这样具有外部依赖，比较复杂的项目模板。

此外 Wing 还增加了文件创建模板，方便扩展新建文件的菜单，比如 Wing 3.1 内置的创建 EUI 组件的功能就是依靠这个 API 实现的，它能够同时创建两个相互关联的 EXML 和 TS 文件。

## 集成常用插件

Wing 集成了 Egret 开发必备的 VSC 开源插件：Chrome Debugger，并对它进行了扩展和增强。




