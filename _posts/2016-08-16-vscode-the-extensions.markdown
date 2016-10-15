---
layout: post
title:  "Visual Studio Code / Egret Wing 技术架构：插件系统"
date:   2016-08-16 22:42:35 +0800
categories: dev
---

## 系列目录
- [核心架构](/dev/2016/08/15/vscode-the-architecture/)
- [插件系统](/dev/2016/08/16/vscode-the-extensions/)
- [Wing 对 VS Code 的扩展](/dev/2016/08/17/wing-vs-vscode/)

在上一篇我介绍了 VSC 的基础架构。这篇我们来说一下对 VSC 来讲至关重要的插件系统。

# 为什么插件系统对 VSC 如此重要

在早期的版本中 VSC 并没有插件系统，只支持 TypeScript、JavaScript和C#的智能感知。还有其余40中语言的代码着色。所以 VSC 只是出现在微软技术的社区中。15 年 12 月份，VSC 发布了第一个支持扩展的版本。不久之后就出现了许多其他语言的支持，比如 Go 语言、C++、Java、Python、Ruby。

所以说有了核心编辑器的极速体验，加上良好的扩展能力才成就了 VSC。

<!--more-->

# VSC 支持哪些扩展

## 语言支持

VSC 制订了一套完善的语言支持体系，方便支持新的编程语言。

### 一个代码编辑器需要哪些功能来支持一种新语言

- 代码显示
    - 代码着色
- 智能感知
    - 代码提示
    - 代码跳转
    - 鼠标触碰提示
    - 查找引用
    - 错误提示
- 代码修改
    - 自动补全
    - 重构功能

![](/public/images/lang-impls.png)

### 兼容 TextMate 的代码着色分析
可以简单的将 TextMate 的语言着色配置文件拷贝到插件中，并在 package.json 中指定即可。

### 语言支持通用协议
VSC 约定了一种通用的通信协议来支持多种语言

![](/public/images/lang-service.png)

由于 VSC 采用多进程的架构，语言的开发者可以使用自己熟悉的语言编写这门语言的语言服务，VSC 将采用 JSON-RPC 通信的方式跟语言服务沟通，执行用户命令，获取结果。

![](/public/images/protocols.png)

## Debugger

同语言服务类似，VSC 开放了一组通用协议来满足不同语言不通平台的调试需求。

## 主题/配色方案

VSC 采用了跟 TextMate 相同的配色方案文件格式。

## 编辑器辅助

VSC 提供了编辑器操作 API，你能够实时获取用户输入点、当前文件代码。从而可以根据用户当前文档确定可以提供的快捷操作。比如自动添加不存在的方法等。

## 扩展命令

开发者可以在插件中定义自己的命令，这些命令会出现在“命令面板” 中，开发者可以通过 “ctrl/cmd + shift + p” 或 “F1” 来调用这些命令，完成复杂的操作。

插件可以使用所有的 NodeJS API，配合各种 NodeJS 库，能够完成非常有想象力的功能。

## 扩展菜单

VSC 提供了文件管理器菜单，编辑器菜单，文件标题菜单扩展点。方便开发者针对不同的上下文进行操作。

## 快捷键

开发者可以为各种自定义操作指定快捷键。

[VSC 插件开发文档](https://code.visualstudio.com/docs/extensions/overview)

[Wing 插件开发文档](http://developer.egret.com/cn/github/egret-docs/Wing/plugin/introduction/index.html)
