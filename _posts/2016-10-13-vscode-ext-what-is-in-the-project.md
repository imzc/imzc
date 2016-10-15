---
layout: post
title: "VS Code 插件项目结构与配置文件"
date:   2016-10-13 20:12:45 +0800
categories: tools
---


* [VS Code 插件创建与发布](/tools/2016/10/13/getting-started-with-vscode-ext/)
* [VS Code 插件项目结构与配置文件](/tools/2016/10/13/vscode-ext-what-is-in-the-project/)
* [VS Code 插件示例，一个 TypeScript 即时预览插件](/tools/2016/10/13/vscode-ext-typescript-live-preview/)

# VS Code Extension 项目结构

打开 yo code 创建的项目，我们能看到下面的目录结构。
![yo code output](http://imzc.me/public/images/openchina2016/project.jpg)

展开之后得到的是下面的层级结构（我省略了大部分用于测试的 node_modules）。

```
├─.vscode
│      launch.json
│      settings.json
│      tasks.json
│          
├─src
│      extension.ts
│      
├─test
│       extension.test.ts
│       index.ts
│      
├─node_modules
│  ├─@types
│  │  ├─mocha
│  │  │      index.d.ts
│  │  │      
│  │  └─node
│  │          index.d.ts
│  │          
│  ├─typescript
│  │                  
│  └─vscode
│     │  thenable.d.ts
│     │  tslint.json
│     └─ vscode.d.ts     
│          
├─out
│  └─src
│     └─  extension.js
│  
│  .gitignore
│  .vscodeignore
│  package.json
│  README.md
│  tsconfig.json
└─ vsc-extension-quickstart.md
```

* `package.json` 是整个 extension 的配置文件。它除了描述 extension 的基本信息，如名称，作者版本等外，
更重要的是描述插件对 vscode 所做的扩展，注册用户可以执行的命令，标记该 extension 应该何时启动。
* `src/extension.ts` 是插件的入口，这个文件 export 了 `activate` 方法，在插件被激活时，vscode 将调用这个方法，执行用户代码。
在这个例子中我们在这里用 `vscode.commands.registerCommand` 注册了 `extension.sayHello` 这个 command 的具体实现。显示一个消息：`Hello World`。
* `.vscode` 这里是 vscode 的项目文件。其中 
    * `launch.json` 定义了当前项目的调试方式。
    * `tasks.json` 定义了编译动作。
    * `settings.json` 定义了针对当前项目的编辑器设定。例如排除不想要显示的文件。
* `node_modules` 中绝大多数 package 为 test 的 devDependencies。与插件开发相关的是 `typescript`、 `vscode` 和 `@types`。
    * 这里引入一个 `typescript` 的原因是为了更好的限定开发中使用的 TS 版本 和 JS 的运行环境。比如，在插件中是不能使用 `window` 对象的，通过在 `tsconfig.json` 中指定 `lib` 即可。
    * `vscode` 主要是为了引入 vscode package 的 TypeScript 定义。让开发体验更加友好。
    * `@types` 是 TS 2.0 引入的第三方 js 库的 TypeScript 定义文件安装方式，这里引入了 nodejs 和 mocha 的定义文件。
* `.vscodeignore` 指定在发布插件时需要排除的文件。

大体了解了整个项目的结构，让我们来看一下实际运行的效果。在 VS Code 中直接按 F5 即可启动调试。VS Code 会
启动一个新的 VS Code 窗口，在这个窗口中会额外加载我们的插件项目以便我们进行测试。

![debug host](http://imzc.me/public/images/openchina2016/1run.png)

我们按 `F1` 或者 `ctrl+shift+p` 输入 hello 就能发现我们刚刚注册的 command。

![debug host](http://imzc.me/public/images/openchina2016/1command.png)

执行这个 command 可以看到此时，我们项目调试控制台输出 
`Congratulations, your extension "vscode-extension" is now active!` 插件的 `activate` 方法被调用。
VS Code 显示一个 Hello World 信息。

![debug host](http://imzc.me/public/images/openchina2016/1runcommand.png)

你可能会问，为什么我们的代码在我们真正执行了 command 的时候才会被激活，这是 VS Code 的一个优化。
vscode 可以直接根据 package.json 来展示插件定义的 command，而不必执行插件的代码，
保证 vscode 的启动速度。在这个插件模板中，我们注册了一个 title 为 Hello World 的 command。
`activationEvents` 为 `onCommand:extension.sayHello`, 仅当我们执行这个 command 的时候，插件才被激活。
VS Code 才会真正开始执行我们的代码。

## package.json


## 字段定义

名称   |   必需   |   类型   |   描述
---- |:--------:| ---- | ------
`name` | 是 | `string` | 插件扩展的名称, 全部是小写字母，并且不能有空格.
`version` | 是 | `string` | 参考[Semver](http://semver.org/lang/zh-CN/). 例如: `1.0.0`
`publisher` | 是 | `string` | 发布者名称
`engines` | 是 | `object` | 包含`vscode`字段的对象， 定义所需环境的版本。例如 `^1.5.0` 表示需要1.5.0或者更高版本.
`displayName` | | `string`| 插件显示的名称.
`description` | | `string` | 插件的描述文本.
`icon` | | `string` | 插件使用的图标，大小为 128*128.
`main` | | `string` | 插件入口的相对路径，当插件启动时将调用入口文件的`activate`方法.
`categories` | | `string[]` | 插件的分类，常用分类有: `[Languages, Snippets, Linters, Themes, Debuggers, Other]`.
[`contributes`](https://code.visualstudio.com/docs/extensionAPI/extension-points) | | `object` | 插件扩展点的描述对象，详见 [contributions](https://code.visualstudio.com/docs/extensionAPI/extension-points).
[`activationEvents`](https://code.visualstudio.com/docs/extensionAPI/activation-events) | | `string[]` | 插件激活事件的描述数组，详见 [activation events](https://code.visualstudio.com/docs/extensionAPI/activation-events).
`keywords` | | `string[]` | 插件的关键词，准确的关键词有助于更好的找到插件.
`dependencies` | | `object` | 插件依赖的 `Node.js` 库. 参见 [npm dependencies](https://docs.npmjs.com/files/package.json#dependencies).
`devDependencies` | | `object` | 插件开发环境下依赖的 `Node.js` 库. 例如，如果插件使用TypeScript编写，则开发环境需要依赖 `typescript`. 参见 [npm devDependencies](https://docs.npmjs.com/files/package.json#devdependencies).
`extensionDependencies` | | `string[]` | 插件依赖的其他插件的id. 插件id为 `${publisher}.${name}`. 例如: `egret.text-tools`.
`scripts` | | `object` | 详见 [npm scripts](https://docs.npmjs.com/misc/scripts)

### contributes（扩展点）

扩展点允许开发者在 vscode 中自定义一些扩展功能。

## contributes.commands

自定义命令 `command` 。定义的命令可以在命令面板(Command Palette)中找到并执行。

>**Note:** 当命令通过快捷键或者命令面板被执行时, 将派发 `activationEvent` `onCommand:${command}`.

可使用下列字段：

名称    | 必需   | 类型   | 描述
---- |:--------:| ---- | ------
`command` | 是 | `string` | 表示命令的唯一id.
`title` | 是 | `string` | 表示命令的名称，此属性将显示在命令面板中.


### 例子

```
...
"contributes": {
	"commands": [{
		"command": "extension.sayHello",
		"title": "Hello World"
	}]
}
...
```


## contributes.keybindings

自定义快捷键 `keybinding`。可以自定义命令执行的按键绑定。

>**Note:** 可以定义不同操作系统平台的快捷键。

>**Note:** 当命令通过快捷键或者命令面板被执行时, 将派发 `activationEvent` `onCommand:${command}`.

可使用下列字段：

名称 | 必需 | 类型 | 描述
---- |:--------:| ---- | ------
`command` | 是 | `string` | 快捷键绑定的命令的id
`key` | 是 | `string` | 快捷键的按键组合，可以的按键的组合字符串参见 附录1 .
`win` |   | `string` | 在windows平台下的按键组合，如果用户在windows下，此属性将覆盖`key`字段指定的按键组合.
`mac` |   | `string` | 在mac平台下的按键组合，如果用户在mac下，此属性将覆盖`key`字段指定的按键组合.
`when` |   | `string` | 快捷键何时被触发的表达式，可用的属性值参见 附录2 .


### 附录1 `key`字段可用的按键组合字符串规则

按键规则一般由组合键 + 其他按键组成。 

组合按键表示如下：

操作系统 | 组合键
---- | ---------
MACOS X | `ctrl+`, `shift+`, `alt+`, `cmd+`
Windows | `ctrl+`, `shift+`, `alt+`, `win+`

>**Note:** 特殊的可以使用 `ctrlcmd` 表示windows下的`ctrl`, mac下的`cmd`


其他按键表示如下：

* `f1-f15`, `a-z`, `0-9`
* `` ` ``, `-`, `=`, `[`, `]`, `\`, `;`, `'`, `,`, `.`, `/`
* `left`, `up`, `right`, `down`, `pageup`, `pagedown`, `end`, `home`
* `tab`, `enter`, `escape`, `space`, `backspace`, `delete`
* `pausebreak`, `capslock`, `insert`


使用空格分割的按键序列，例如(ctrl+k ctrl+c)，表示先按ctrl+k，再按ctrl+c 才能触发该快捷键。

如果要使某一命令既可以按ctrl+k触发，又可以按ctrl+c触发，请定义两组keybinding，command相同，但是key字段不同。


### 附录2 `when`字段可用的字符串

`when`字段可以使用 `!`, `&&`, `==`, `!=` 等表达式限定条件。如

`!inDebugMode`  , `editorTextFocus && editorLangId == 'ts'` 都是可用的表达式。



* `editorFocus` 编辑器获得焦点时
* `editorTextFocus` 文本编辑器获得焦点时
* `editorLangId` 表示编辑器的语言id，`editorLangId` 通常对应编辑器对应文档的扩展名
* `inDebugMode` 表示启动调试时


### 例子

定义命令 `"extension.sayHello"` 的windows下快捷键为 `Ctrl+F1`，mac下快捷键为 `Cmd+Shift+F1` :

```
...
"contributes": {
	"keybindings": [{
		"command": "extension.sayHello",
		"key": "ctrl+f1",
		"mac": "cmd+shift+f1",
		"when": "editorTextFocus"
	}]
}
...
```

## contributes.configuration

提供可设置的选项，用户可以在全局或者工作空间的设置中修改这些设置信息。

插件的开发者需要提供一套 JSON schema 来描述这些可以配置的选项，这样用户在修改时，能够得到编辑器的智能提示。

你可以通过下面的方法来读取用户自定义的配置信息
`vscode.workspace.getConfiguration('myExtension')`.

### 例子

```
...
"contributes": {
	"configuration": {
		"type": "object",
		"title": "TypeScript configuration",
		"properties": {
			"typescript.useCodeSnippetsOnMethodSuggest": {
				"type": "boolean",
				"default": false,
				"description": "Complete functions with their parameter signature."
			},
			"typescript.tsdk": {
				"type": "string",
				"default": null,
				"description": "Specifies the folder path containing the tsserver and lib*.d.ts files to use."
			}
		}
	}
}
```

## contributes.languages

提供一种新语言的信息，以便 VS Code 能够支持它的开发.

这这一节中, language 一般是指一个跟文件名相关的id (See `TextDocument.getLanguageId()`).

VS Code 有三种方式确定一个文件使用的开发语言，这三种方式可以独立使用
1. 文件的扩展名 (下面说的 `extensions`)
2. 文件名 ( 下面说的 `filenames`)
3. 文件的第一行中的文本 (下面说的 `firstLine`)

最后一项 VS Code 需要的信息就是 `aliases`：语言的别名，这个属性值列表的第一个会作为语言的显示名称，显示在编辑器的右下角和语言选择列表中。

当用户打开了一个文件，VS Code会用这三个规则匹配文件，来确认文件所使用的语言，同时，VS Code会抛出一个 activationEvent `onLanguage:${language}`(例如下面例子的 `onLanguage:python`),来激活对应的插件。


### 例子

```
...
"contributes": {
	"languages": [{
		"id": "python",
		"extensions": [ ".py" ],
		"aliases": [ "Python", "py" ],
		"filenames": [ ... ]
		"firstLine": "^#!/.*\\bpython[0-9.-]*\\b"
	}]
}
```
## contributes.debuggers

为 VS Code 提供一个 debug 适配器，帮助 VS Code 连接到一个外部的 debug 服务，扩展 VS Code 的调试功能。

debug 适配器运行在一个单独的进程中，与 VS Code 之间采用特定的协议来通信。你需要提供至少一个可执行的脚本来实现这个 debug 适配器。


### 例子

```
...
"contributes": {
	"debuggers": [{
        	"type": "node",
        	"program": "./debugger/out/node/nodeDebug.js",
        	"runtime": "node",
        	"enableBreakpointsFor": { "languageIds": ["javascript", "typescript", "coffeescript"] }
        }]
}
...
```

## contributes.grammars

提供一套 TextMate grammar 来为 VS Code 添加语言支持。你需要提供，这组语法需要适配的语言 `language`，语法的 TextMate `scopeName` 值，和 语法定义文件的路径

>**注意:** 语法定义文件的格式可以是 JSON (扩展名 .json) 或 XML plist 格式 (如果扩展名不是 json 就认为是这种格式).

### 例子

```
...
"contributes": {
	"grammars": [{
		"language": "shellscript",
		"scopeName": "source.shell",
		"path": "./syntaxes/Shell-Unix-Bash.tmLanguage"
	}]
}
...
```

## contributes.themes

为 VS Code 提供一个 TextMate theme 代码配色主题。你需要指定一个 `label` 表明这个主题是暗色主题还是浅色主题，还有主题文件的路径（XML plist 格式）


### 例子

```
"contributes": {
	"themes": [{
		"label": "Monokai",
		"uiTheme": "vs-dark",
		"path": "./themes/Monokai.tmTheme"
	}]
}
```

## contributes.snippets

```
"contributes": {
	"snippets": [{
			"language": "go",
			"path": "./snippets/go.json"
	}]
}
```

## contributes.jsonValidation

提供特定 json 文件的 JSON schema， `url`可以是插件中内置的文件，或者一个远程的url，例如 [json schema store](http://schemastore.org/json)

```
"contributes": {
    "jsonValidation": [{ 
 		"fileMatch": ".jshintrc",
 		"url": "http://json.schemastore.org/jshintrc"
	}]
} 
```


更多信息可以参考 [Extension Manifest File - package.json](https://code.visualstudio.com/docs/extensionAPI/extension-manifest)

下面，让我们来写一个 即时预览 TypeScript 执行的插件。[VS Code 插件示例，一个 TypeScript 即时预览插件](/tools/2016/10/13/vscode-ext-typescript-live-preview/)

