---
layout: post
title: "TypeScript 进阶：给第三方库编写声明文件"
date:   2016-11-30 22:15:45 +0800
categories: dev
---


知道了 [TS 项目的结构](/dev/2016/11/22/getting-start-with-typescript/)信息之后，编写代码是非常容易的。没那么容易的应该是怎么样使用第三方或者是项目原有的 JS。
今天我们来说一说。本文会涉及到大量原来我们在 JS 中经常使用的“奇技淫巧”在 TS 中的实现，值得一读。

## 第三方库使用的关键：d.ts 文件

TS 目前支持三种文件格式：.ts .tsx 和 .d.ts，分别是普通的 ts 文件，主要用于 React 支持的 tsx 文件和
声明（declaration）文件。dts 文件会参与编译，但不会输出任何代码，它的主要作用是为其他 JS 文件提供类型声明。
比较像 C C++ 的 头文件。

## TSC 内置的 d.ts

默认情况下我们每个 TS 项目都包含了至少一个 dts 文件。为什么这么说呢，我们可以测试一下。

在 index.ts 中输入下面的代码

```typescript
var array = [1,2,3];
array.forEach(i=>console.log(i));
```
在 `forEach` 上右击，选择“转到定义”。你用的是记事本，没有右键菜单？那就想象吧。IDE 会打开一个 "lib.d.ts" 文件。
你会看到一些这样的代码：

```typescript
interface Array<T> {

    // 一些其他代码

    /**
      * Performs the specified action for each element in an array.
      * @param callbackfn  A function that accepts up to three arguments. forEach calls the callbackfn function one time for each element in the array.
      * @param thisArg  An object to which the this keyword can refer in the callbackfn function. If thisArg is omitted, undefined is used as the this value.
      */
    forEach(callbackfn: (value: T, index: number, array: T[]) => void, thisArg?: any): void;
}
```

在 lib.d.ts 中定义了 `Array` 这种类型拥有的属性和方法。lib.d.ts 中定义了数组有一个 `forEach` 方法。
它需要至少一个参数：`callbackfn`，第二个参数 `thisArg` 是可选的，类型是任意的 Object。 

lib.d.ts 是被 TSC 默认包含的，它里面定义了 JavaScript 的公共库，如 Array、Math、RegExp 等。
除了 JS 公共库，它里面还包含了 HTML DOM 的类型定义，比如 `window`、`document`、`DOMParser` 等等。这样的默认设定
可以满足大部分 TS 开发的需求。

### 指定项目需要的公共库

你可能会问，如果我开发的是一个 NodeJS 应用，我是不希望有 DOM 的 API 的，因为运行时肯定会报错。TSC 有对应的解决方案。
你可以在 tsconfig.json 中通过 `lib` 指定需要包含的 libs。没有指定时，它的值相当于 `target` 语言版本 + `dom`。

```json

{
    "compilerOptions": {
        "target": "es5",
        "module": "es2015",
        "lib": [
            "es5",
            "dom"
        ]
    }
}

```
我们可以通过指定 `lib` 为 `es5` 或者 `es2015` 表示 NodeJS 这种没有 DOM 的开发环境。`lib` 的可选值非常多，TS team
针对当下的 JS 开发环境，对 `es2015` 的库进行了细分，让开发者可以方便的使用部分 `es2015` 中的库，比如可以通过添加
`es2015.promise` 来增加 `Promise` 支持。

## 第三方库的 d.ts

知道了 JS 公共库的定义方式，其他第三方库也是一样的道理。社区中已经有大量常用库的 dts 文件，我们可以根据需要下载使用。
但是社区中的 dts 五花八门，良莠不齐。为了更好的让他们为我所用。我们还是需要了解一些编写 dts 的技巧，这样遇到不符合我们
需求的 dts 文件，我们可以对他们稍加修改。对于没那么热门的库，自己编写 dts 就尤为重要了。

## 为 jQuery 编写 d.ts

jQuery 作为瑞士军刀一样的库，大家已经非常熟悉了。我们来尝试为它写一个 dts。选它的原因当然是因为它够复杂！

### dts 中可以只包含自己需要的功能

如果我们只是希望在我们的项目中使用某个库，我们可以只写我们需要的部分。比如我们只希望用 jQuery 在 DOM Ready 的时候执行我们的
函数，比如在 index.ts 中加入下面的代码：

```typescript
$(function(){
  console.log("DOM Ready!");
});
```

TS 需要知道 `$` 是一个全局函数，接收一个函数作为参数。我们在项目根目录新建一个 jQuery.d.ts 文件。

```typescript

declare function $(callback:Function):void;

```

这里声明了一个 function `$`，接收一个类型为 `Function` 的参数，没有返回值。编译项目，没有问题，下班回家！

这一切是怎么工作的呢， tsconfig.json 中我们没有指定 `files` 项目目录中的 jQuery.d.ts 就被包括在了项目中。
它告诉编译器在运行环境中 JS 会定义一个名为 `$` 的全局函数。TSC 会根据这个声明来验证其他 TS 中 `$` 的调用是否满足定义。
比如 `$(123)` 将会报错，因为 `123` 不是一个函数。

### 重载的声明方式

我自己都看不下去了，这根本不是 jQuery，连 CSS Selector 都不支持。通常 jQuery 的使用场景是这样的：

```typescript
var button = $("#id");
var color = button.css("color");
button
    .css("height","20px")
    .click(e=>{
        console.log("Click!");
    });
```
编译会有大量报错信息，因为我们之前定义的 `$` 只能接受一个 `Function` 而且没有返回值。为了支持 CSS Selector 我们
让 `$` 支持多种调用方式，一般语言中叫做 “重载” 。

```typescript
declare function $(callback:Function):void;
declare function $(selector:string):any;
```
这里我们重复声明了 `$`，TSC 会根据 `$` 的参数类型自动匹配函数的返回值。这里我们给 `$` 增加了接收一个 `string` 类型
参数的重载，它返回一个 `any` 类型的结果。但在真实的 JS 中，它会返回一个 `JQueryElement` 有 `css`、`click` 等方法。
所以我们继续完善这个 dts。 

### Interface 声明

我们先定义一个 interface 叫做 `JQueryElement`，并添加对应的方法。

```typescript
interface JQueryElement {
    click(Function): JQueryElement;

    css(name: string): string;
    css(name: string, value: any): JQueryElement;
}

declare function $(selector: string): JQueryElement;
```
针对可以链式访问的方法返回类型同样设置成 `JQueryElement`，值得注意的是 `css` 这个方法也应用了刚刚说的重载，
根据参数的数量返回不同的类型。最终我们把 `$(selector: string)` 的返回值改成 `JQueryElement` 回去看 index.ts
中的代码，已经没有报错信息了。


### 复杂类型的变量声明

jQuery 远没有这么简单，比如使用频度跟 Selector 差不多的还有它对 XHR 的封装。

```typescript
$.get("http://domain.com/data", data => { 

});

$.get("http://domain.com/data", { param: 1 }, data => { 
    
});
```
此时 `$` 其实已经不是一个简单的 Function 了，它同时还是一个包含属性的 Object。于是我们需要给它单独定义一个 interface
来满足它的各种特性。我们先把刚刚写的 function 形式转化成 interface 形式

```typescript
interface JQueryVariable {
    (callback: Function): void;
    (selector: string): JQueryElement;
}

declare var $:JQueryVariable;
```
我们可以简单的通过定义一个没有名字的方法来告诉 TSC，这个类型的变量是一个 Function。
然后我们生命一个类型是 `JQueryVariable` 的变量 `$`。

此时，为 `$` 添加 `get` 方法就很容易了，完成后的效果如下

```typescript
interface JQueryVariable {
    (callback: Function): void;
    (selector: string): JQueryElement;

    get(url:string, success:Function);
    get(url:string, data:any, success:Function);
}
```

### 回调函数的处理

在上面的 dts 中所有的回调函数我都用使用了 Function 类型，但在实际项目中，应该尽量避免。它起不到任何约束或者智能提示
的作用。

稍微总结一下，在 TS 中，函数的类型一般有下面几种表示方式：

```typescript

// 直接声明
declare function name():void;

// interface 形式
interface Callback {
    ():void;
}
declare var name:Callback;

```
当然还有最常用的

```typescript
// lambda 表达式方式
declare var name:() => void;
```

给回调函数指定类型时，最常用的是 lambda 表达式方式，毕竟写起来比较简单。但是当一个回掉函数类型需要被重用，
或者回掉函数需要比较丰富的注释时，还是更加推荐 interface 的写法。

```typescript
interface JQueryAjaxCallback {
    /**
     * @param data 服务器返回的数据
     * @param xhr 原始的 XMLHttpRequeset 对象
     */
    (data:any,xhr:XMLHttpRequest):void;
}

interface JQueryVariable {
    (callback: Function): void;
    (selector: string): JQueryElement;

    get(url:string, success:JQueryAjaxCallback);
    get(url:string, data:any, success:JQueryAjaxCallback);
}
```

待续...

本文是我在微软 Ignite 2016 的一个演讲的文字版。这里附上 PPT 和部分实例代码，当然主要代码已经包括在了文章中。

[Ignite China 2016 TypeScript Session PPT](/public/files/ts-session.pdf)

[Ignite China 2016 TypeScript Session Demo](/public/files/ignite2016-demos.zip)