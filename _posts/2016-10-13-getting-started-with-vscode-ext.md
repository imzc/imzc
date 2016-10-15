---
layout: post
title: "VS Code 插件向导"
date:   2016-10-13 22:42:35 +0800
categories: tools
---

我会通过几篇文章介绍一下 VS Code 的插件开发。

* [VS Code 插件创建与发布](/tools/2016/10/13/getting-started-with-vscode-ext/)
* [VS Code 插件项目结构与配置文件](/tools/2016/10/13/vscode-ext-what-is-in-the-project/)
* [VS Code 插件示例，一个 TypeScript 即时预览插件](/tools/2016/10/13/vscode-ext-typescript-live-preview/)

VS Code 使用 Node.js 来开发插件，插件运行在单独的 Node.js 进程中。
你可以看这里了解更多 VS Code 插件系统的信息：
* [VS Code 核心架构](/dev/2016/08/15/vscode-the-architecture/)
* [VS Code 插件系统的能力](/dev/2016/08/16/vscode-the-extensions/)

# VS Code 插件创建与发布


## 创建项目

vscode team 开发了 [Yeoman generator](https://github.com/Microsoft/vscode-generator-code) 帮助大家快速开始创建一个 vscode extension.


### 安装生成器

```bash
npm install -g yo generator-code
```

### 启动生成器

vscode 的 Yeoman 生成器会引导你创建一个 vscode extension 项目

在控制台执行:

```bash
yo code
```

![yo code output](http://imzc.me/public/images/openchina2016/yocode.jpg)

生成器提供了空项目和若干其他项目类型的模板，可以按需选择。生成器的更多详细信息可以参考 [vscode 的官方文档](https://code.visualstudio.com/docs/tools/yocode)。
我们这里使用 TypeScript 创建一个新的 vscode extension。

## 开发测试

我们稍后主要讲这一块

## 发布 extension

### 安装 vscode extension 发布工具 vsce

vsce 是 vscode team 开发的命令行工具，辅助开发者打包和发布 extension。

```bash
npm install -g vsce
```

### 打包插件

```bash
vsce package
```

`package` 命令可以将 extension 打包为一个 `.vsix` 文件，你可以用 vscode 打开它本机安装，或者将它分享给他人。

### 发布到 vscode Marketplace

```bash
vsce publish
```

你可以使用 `publish` 来将插件发布到 vscode 的官方市场，与全球数百万开发者分享。
需要注意的是，在执行命令之前你需要注册一个 [Visual Studio Team Services](https://www.visualstudio.com/en-us/get-started/setup/sign-up-for-visual-studio-online) 账号，
并对 vsce 进行一些设置。具体请参考 [vscode vsce 官方文档](https://code.visualstudio.com/docs/tools/vscecli)。

我们继续看一下 [VS Code 插件项目的结构与配置文件](/tools/2016/10/13/vscode-ext-what-is-in-the-project/)







