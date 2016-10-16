---
layout: post
title: "3) VS Code 插件示例，一个 TypeScript 即时预览插件"
date:   2016-10-15 22:15:45 +0800
categories: tools
---


* [1) VS Code 插件创建与发布](/tools/2016/10/13/getting-started-with-vscode-ext/)
* [2) VS Code 插件项目结构与配置文件](/tools/2016/10/14/vscode-ext-what-is-in-the-project/)
* 3) VS Code 插件示例，一个 TypeScript 即时预览插件

> 源代码：https://github.com/imzc/vscode-live-code
## 插件目标

这个插件最终的效果，希望是能够在编辑 TypeScript 时，能够即时在 HTML 中运行，看到运行效果。

为了实现这个目标我们需要解决几个问题

1. 插件需要在编辑 TypeScript 时激活
2. 需要在用户输入的时候即时编译 TypeScript
3. 需要注册 command 在 VS Code 中打开一个 WebView
4. 在 WebView 中加载编译后的 JavaScript

<!--more-->

## 创建项目

参考 [VS Code 插件创建与发布](/tools/2016/10/13/getting-started-with-vscode-ext/) 创建一个
TypeScript 项目。

## 添加 command，修改 activationEvents

如图添加一个 command 并添加一个 activationEvent，当文件语言为 TypeScript 的时候激活插件。

![](http://imzc.me/public/images/openchina2016/addtscommands.jpg)

同时我们还添加了一个快捷键 `ctrl+k, ctrl+t` 来快速调用这个 command。

在 scr/extensions.ts 中进行相应的修改，并添加一个 outputChannel 来往输出面板中打印消息，
方便我们输出信息和调试。

![](http://imzc.me/public/images/openchina2016/2regcommands.jpg)

调试并打开一个 TypeScript 文件，可以看到插件激活，输出面板显示出来。

![](http://imzc.me/public/images/openchina2016/2run.png)

## 处理用户输入

VS Code 提供了 `vscode.workspace.onDidChangeTextDocument` 方法来处理文件变化。
这个方法跟 File Watcher 的区别是，用户没有执行 save 的时候我们也能收到消息，更加符合我们的需求。

![](http://imzc.me/public/images/openchina2016/3handlechanges.jpg)

在回调函数中我们判断一下当前文件的语言，排除不需要处理的文件。然后打印一个消息。

![](http://imzc.me/public/images/openchina2016/3output.jpg)

## 处理 TypeScript 编译

TypeScript 提供了丰富的 API 让我们能够用编程的方式编译 TypeScript。

首先我们引入 `typescript` 库。

> 注意将 package.json 中的 typescript 库从 devDeps 转移到 deps 中。

今天我们用到的是 ts 提供的最简化的一个 API `ts.transpileModule`, 它能够将 TS 源码直接转化为 JS。
我们将转化的输出暂时输出到输出面板中。

![](http://imzc.me/public/images/openchina2016/4compile.jpg)

运行后，我们修改一个 ts 文件，可以看到即时生成的 JavaScript。

![](http://imzc.me/public/images/openchina2016/4output.jpg)

## 打开一个 WebView

处理完了上面的流程，终于来到最关键的预览环节。这里用到 vscode 的另外一个 API

### vscode.previewHtml
`vscode.commands.executeCommand(cmd:string, ...params:object[])` 用来执行 vscode 中
支持的 command，包括内置的和其他插件扩展的，都可以。 

这里我们使用的是 `vscode.previewHtml`,这个 command 支持打开一个 html url 在 vscode 进行展示。
但处于安全原因，vscode 对 url 进行了限制，只能支持 file 和 插件注册的 url schema。在这里我们使用
`livecode`作为我们预览的 url schema，最终交给 vscode 的 url大概是这样的：`livecode:/c:/work/demo/test.ts?file:///c:/work/demo/test.ts`
然后我们会注册这个 schema 并编写代码根据 这个 url来生成对应的 html 代码。

我们先改一下 preview command 对应的代码，让它打开一个 webview

![](http://imzc.me/public/images/openchina2016/5openhtml.jpg)

### 构建 html
我们使用 `TextDocumengContentProvider` 来给 webview 生成 html 代码。我们需要实现 
`provideTextDocumentContent(uri: Uri): Thenable<string>` 根据 url 生成 html 代码。

![](http://imzc.me/public/images/openchina2016/6provider.jpg)

我们添加一个文件 LiveCodeDocumentContentProvider.ts 来实现

![](http://imzc.me/public/images/openchina2016/6livecodeprovide6r.jpg)

`provideTextDocumentContent` 中我们简单的构建了一个 html 文档，其中 body 部分我们会添加一个 script tag。
用来承载编译后的 JavaScript 代码，我们会用一个 function 包裹 TS 编译的逻辑，将它传递给 provider，
也就是这里看到的 `this._render`

回到 extensions.ts 我们来注册 livecode 这个schema。首先引入 LiveCodeDocumentContentProvider，
实例化，将TS compile function 传给它，然后注册。

![](http://imzc.me/public/images/openchina2016/6reg.jpg)


![](http://imzc.me/public/images/openchina2016/6compile.jpg)

最后，当TS文件变化时，通知 provider。
![](http://imzc.me/public/images/openchina2016/6update.jpg)

我们来运行一下。可以看到 TypeScript 运行的效果，我们将它拖到右侧，修改代码，可以看到实时的效果

![](http://imzc.me/public/images/openchina2016/6preview.jpg)

### 在右侧编辑区域预览
我们稍微修改一下 previewHtml 的参数，让它在右侧的编辑区域打开，并在 title 中显示 “Preview”

![](http://imzc.me/public/images/openchina2016/7columntitle.jpg)

![](http://imzc.me/public/images/openchina2016/7preview.jpg)

另外，当我们代码中出现问题的时候，预览窗口中没有任何输出。我们稍微修改一下 compileTypeScript 在 JS 外层
包裹一层异常捕获代码。

![](http://imzc.me/public/images/openchina2016/7trycatch.jpg)


![](http://imzc.me/public/images/openchina2016/7error.jpg)

# Next
至此，我们完成了一个 TypeScript 插件的基本功能。但是其实还有很多问题值得进一步深入。

* 项目级别的预览，TS 之间相互引入
* 第三方库引入
* Node.js 环境下的实时执行
* 等等








