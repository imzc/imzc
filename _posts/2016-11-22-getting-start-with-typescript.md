---
layout: post
title: "TypeScript 第一步"
date:   2016-11-22 22:15:45 +0800
categories: dev
---

不少小伙伴 (基友😏) 开始用 TS 了, 但现在的 JS 项目复杂度, 已经远不是 TS 刚出现时候的程度了.
各种第三方库,各种加载库(webpack, AMD, UMD, SystemJS, CommonJS), 各种运行环境 Node, Browser,
各种 JS 语言版本 es3 es5 es6 es2015 es2017. 还有 jsx 这种语言. 古人云:能用 JS 的地方就能用 TS.
那么, 怎么用呢. 今天咱们就来说一说.

<!--more-->

第一步当然还是 搭建一个简单的开发环境.

## 安装 TypeScript

可以执行 `npm install typescript -g` 来安装 TypeScript. 安装后你就能使用 `tsc` 命令了. 用来将 TS 文件编译成 JS 文件. 
我们先不管它,接着进行下面的步骤.

## TypeScript 项目结构

### tsconfig.json

TS 项目是通过 tsconfig.json 来定义的. 很多时候如果你的 IDE 没有很好的进行智能提示,一般是没有添加 tsconfig.json

在命令行中执行 `tsc --init` 可以在当前目录中快速创建一个 tsconfig.json 文件.

```json

{
    "compilerOptions": {
        "module": "commonjs",
        "target": "es5",
        "noImplicitAny": false,
        "sourceMap": false
    }
}

```
#### compilerOptions 编译选项

先说一下对 TS 项目来说最重要的两个设置

* `module` 定义生成的 js 中应该采用什么模块化组织方式. 可选的值有 `none`, `commonjs`, `AMD`, `UMD`, `System` 和 `es2015`.
在 TS 中默认使用 es2015 的模块组织形式, 通过指定 `module` 你可以将他们转化成自己熟悉的组织方式.

* `target` 定义生成的 JS 代码版本, 可选 `es3`, `es5` 和 `es2015`. 你可以根据自己的运行时 JS 版本进行选择.
    
    > 注意: 因为 ES 不同版本的公共库不一致, 所以三个 target 有些语法和库上的差异. 比如 `es3` 不支持 getter setter,
     `es5` 默认没有 `Promise`. 针对库不可用的情况, 我们可以通过添加 d.ts 的方式来使用.

* 其他编译选项可以参考: 
[TS 编译参数(官方英文)](https://www.typescriptlang.org/docs/handbook/compiler-options.html)
或者 [TS 编译参数(社区中文)](http://tslang.cn/docs/handbook/compiler-options.html)


#### 其他配置项

默认情况下, TS 项目会包含 该目录下所有的 `ts`, `tsx`, `.d.ts` 文件. 我们可以通过
`files`, `include` 和 `exclude` 来指定哪些文件应该在项目中.

```json
{
    "compilerOptions": {
    },

    "files":[
        "a.ts",
        "b.ts"
    ],

    "include": [
        "src/**/*"
    ],

    "exclude": [
        "node_modules",
        "**/*.spec.ts"
    ]
}
```

现在你已经大致了解 TS 项目是什么样子的了. 我们接下来按照 JS 发展的历史, 从简单到复杂来介绍
用 TS 如何更爽的开发 web 应用.

## 传统浏览器运行环境

在传统的浏览器中, JS 加载过程是由浏览器控制的. 说是控制, 但其实非常简单粗暴. 按照先后顺序
阻塞加载. 造成的结果一般是浏览器会空白一段时间, `head` 里所有的 JS 加载完成后, `body` 才会
开始渲染. 但噩梦还没结束, "经典" 的 web 开发, JS 是无处不在的, 两个 `div` 之间加个 `script` 也
很常见. 然后页面再卡一段时间😩. 

因为没有模块化的组织方式,页面要加载哪些 JS 文件都是手动写在 html 中的, 而且顺序还不能错 😩. 
这就是 tsconfig 最先支持 `files` 的原因 (是的, 一开始是没有 include 和 exclude 的, 苦啊). 

### 一个最基本的 TS 项目

我们将 tsconfig.json 最简化

```json
{
    "compilerOptions": {
        "target": "es5"
    }
}
```
添加一个 index.ts 我们就可以写代码了.

```typescript
var body = document.body;
var div = document.createElement('div');
div.textContent = 'Hi TS';
body.appendChild(div);
```
添加一段文字到 body 中. 这段代码根本不像 TS 啊, 是的, 毕竟 TS 是对 JS 的扩展, 写 JS 当然是
没有任何问题的. 但是在 TS 中, 借助强大的类型推倒(推导)功能, 编译器其实知道 body 是一个 HTMLElement,
div 是一个 HTMLDivElement 的. 至于为什么, 我们**后面介绍 d.ts 的时候再说**.

我们在命令行中执行 `tsc` 编译这个 TS 项目. 当 `tsc` 命令没有指定任何参数时, 它将会在当前目录
寻找 tsconfig.json 并按照里面的配置编译项目中的 TS 文件.

在这个 TS 文件中我们没有写任何的 `export` 那么 TSC 会认为它是一个传统的 JS 文件. 
`body` 和 `div` 都是全局变量. 当我们再添加一个 TS 文件时, 我们在 IDE 中是能够收到 `body` 和 `div`
的智能提示的.

我们再添加一个文件 `date-picker.ts`.

```typescript
var h1 = document.createElement('h1');
div.appendChild(h1);
```
编译, 没有任何问题. 

每次执行 `tsc` 命令是一件挺麻烦的事情. 我们可以给 `tsc` 命令加一个参数
`-w`, `tsc` 将会自动监听文件变化, 自动进行编译.

> 经常遇到有开发者问, 我怎么在页面的 script 中使用 TS 定义的变量啊? 答案是, 直接用就行了. 
至于如何在 TS 中使用在 script 标签中定义的变量. 我们还是**等到介绍 d.ts 的时候说**.

### 命名空间 namespace

即便在远古 JS 开发时代, 大家也是不希望有变量名污染的, 不然项目打(大)起来就是灾难了.

TS 提供了 `namespace` 关键字来更好的管理 class 变量名等.

在 date-picker.ts 中我们定义下面的代码

```typescript
namespace components {
    // 公开变量
    export var ready = false;
    // 局部变量
    var count = 0;
    export class DatePicker { 
        constructor(private target: HTMLElement) { 
            count++;
        }

        popup() { }
    }
}
```
在 index.ts 中可以这样用

```typescript
var componentReady = components.ready;
var componentCount = components.count; // 错误
var datePicker = new components.DatePicker(div);
```

`namespace` 中的局部变量外界是无法访问的, 可以配合 `export` 关键字公开其访问级别.

至于 `namespace` 是怎么实现的, 我们看一下生成的代码就再清楚不过了.

```javascript
var components;
(function (components) {
    components.ready = false;
    var count = 0;
    var DatePicker = (function () {
        function DatePicker(target) {
            this.target = target;
            count++;
        }
        DatePicker.prototype.popup = function () { };
        return DatePicker;
    }());
    components.DatePicker = DatePicker;
})(components || (components = {}));
```

从这段 JS 我们也能看出另外一个特性, 就是同名的 `namespace` 是可以分布在不同的文件中的.
所有 `export` 的属性都会合并到这个 `namespace` 中.

## NodeJS 以及其他模块化组织的项目

近几年 NodeJS 异军突起, 凭借良好的群众基础, 迅速占领了大片乡村, 有要与 PHP Java 大军决一死战的意思. 
也圆了很多小伙伴的全栈梦. 确实, 谁让浏览器现在还只认 JS 呢.

NodeJS 所在的环境跟浏览器有非常大的不同. 所有的 JS 都在运行时本地. 代码加载起来没有任何障碍.
所以 NodeJS 采用了 CommonJS 组织代码. 每个 JS 文件都是一个 module, 通过 `exports.xxx = xxx` 
来公开属性, 用 require 加载其他 JS.

让我们把 tsconfig.json 稍加修改,添加 `"module": "commonjs"`.

date-picker.ts 改为

```typescript
export var ready = false;

var count = 0;
export class DatePicker { 
    constructor(private target: HTMLElement) { 
        count++;
    }

    popup() { }
}
```
区别于普通 CommonJS 的导出方式, TS 直接在需要导出的变量前加 `export` 即可.

在 index.ts 中引用 `DatePicker` 的写法有很多, 先写一个经典的

```typescript
import components = require('./date-picker');
var datePicker = new components.DatePicker(null);
```
`import` + `require` 是 TS 最早支持的包引用方法, 跟 CommonJS 非常像. 随着 ES2015 的
确定, TS 中越来越多的语法开始跟 ES2015 取齐. 所以你还可以这么写

```typescript
import * as components from './date-picker';
var datePicker = new components.DatePicker(null);
```
或者

```typescript
import { DatePicker } from './date-picker';
var datePicker = new DatePicker(null);
```
因为我们指定了 JS module 为 `CommonJS` 生成的代码就是 CommonJS 风格的

```javascript
"use strict";
var date_picker_1 = require("./date-picker");
var datePicker = new date_picker_1.DatePicker(null);
```
此时我们将 tsconfig 的 `module` 改成其他值测试一下

AMD:

```javascript
define(["require", "exports", "./date-picker"], function (require, exports, date_picker_1) {
    "use strict";
    var datePicker = new date_picker_1.DatePicker(null);
});
```

ES2015:

```javascript
import { DatePicker } from './date-picker';
var datePicker = new DatePicker(null);
```

完美!

至此, 你已经了解了 tsconfig.json, 知道如何用 TS 编写传统的浏览器项目
和模块化组织的 TS 项目了. 接下来我会介绍入门 TS 的最大障碍: 如何使用
现有的 JS 库.