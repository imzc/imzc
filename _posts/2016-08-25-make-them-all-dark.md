---
layout: post
title: "让世界变沉静 - 让 Web 开发环境全面黑化"
date:   2016-08-25 22:42:35 +0800
categories: dev
---


话不多说，这里介绍怎样将开发环境尽可能变成暗色，防止眼睛疲劳。

## IDE

目前主流的 IDE 都提供了主题选择，这里便不再多说。

## Chrome Developer Tools

Chrome DevTools 是 Web 开发者必不可少的工具，借助一款 Extension 我们能够很容易将它改造成深色。

[DevTools Theme: Zero Dark Matrix](https://chrome.google.com/webstore/detail/devtools-theme-zero-dark/bomhdjeadceaggdgfoefmpeafkjhegbo)

![](/public/images/devtools.png)

### 安装步骤

- 安装该 Extension 
- 打开 [chrome://flags/#enable-devtools-experiments](chrome://flags/#enable-devtools-experiments) 启用 `开发者工具实验性功能`，重启 Chrome。
- 打开 Chrome DevTools, 选择 `Experiments` 标签, 选中 'Allow custom UI themes'.

![](/public/images/devtools-setting.png)

![](/public/images/devtools-allow-theme.png)

- 重新打开 DevTools.


## Windows

Windows 提供了主题机制，但是，只限于官方主题，大部分就是改改背景和 Title bar 配色，我们可以借助第三方软件实现更加定制化的主题。

比如这个仿照 Gnome Arc Theme 来做的 Windows 主题

[A Theme for Windows 10 - Arc](http://neiio.deviantart.com/art/Arc-618235768)

![](/public/images/arc.png)

几个前置依赖：

[How to Install Custom Themes](http://neiio.deviantart.com/art/How-to-Install-Custom-Themes-262833454) - 解除 Windows 对第三方主题的限制

[OldNewExplorer](http://www.msfn.org/board/topic/170375-oldnewexplorer-118/) - （可选） 还原 Win7 的 资源管理器，因为 Win 8 / 10 的 Ribbon 菜单还不支持主题。


## Mac OS X

Mac OS 目前支持让 Dock 和 Menu 变反色，知道让 Finder 变深色方法的话请告诉我。


> Have fun!





