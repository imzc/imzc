---
layout: post
title: "Angular 2 two-way binding property"
date:   2016-10-27 22:15:45 +0800
categories: dev
---

希望有一个 Pager 组件，能够绑定 `total` 和 `current` 效果如下面的代码。

```html
<mx-pager [total]="totalPage" [current]="currentPage"></mx-pager>
```

对应的 `Pager` 代码应该是这样的

```typescript
export class PagerComponent {
	
    @Input()
    total: number;

    @Input()
    current: number;
}
```
但是，我们的 Pager 不应该只是一个展示数据的组件，当用户点击新 page 的时候，
我们需要在父组件中执行相应的动作。也就是说，`current` 属性需要双向绑定。

<!--more-->

## Twe-way binding property

在 ng2 中可以通过 `Output` 装饰器来实现双向绑定属性。

首先添加一个 `currentChange` property。
类型为 `EventEmitter<number>` 我们将通过这个 emitter 向外部派发 `current` 属性改变的事件。

> `EventEmitter<number>` 有一个泛型参数，需要跟 `current` 属性的类型一致。
表明 emitter 在 call 事件监听方法时传递的参数类型。

```typescript
export class PagerComponent {
	
    @Input()
    total: number;

    @Input()
    current: number;

    currentChange = new EventEmitter<number>();
}
```

然后跟 `@Input()` 类似，为 `currentChange` 添加 `@Output()` 装饰器

```typescript
@Output
currentChange = new EventEmitter<number>();
```

当 `current` 变化的时候, 我们只需要执行 `currentChange` 的 `emit` 方法即可。

```typescript
this.currentChange.emit(someValue);
```

一开始的 template 代码可以改为双向绑定了：

```html
<mx-pager [total]="totalPage" [(current)]="currentPage"></mx-pager>
```

## 使用 Accessor 提高一致性

为了保持一致性，我习惯将 `emit` 的调用跟 `current` 的赋值过程结合起来。
将 `current` 改成 `accessor`。

```typescript
private _currnet:number;

@Input()
get current(){
    return this._currnet;
}
set current(value:number){
    if(value === this._currnet)
        return;

    this._currnet = value;
    this.currentChange.emit(value);
}

```

这样在组件内部，我们只需要关心 `current` 即可，在赋值过程中 `currentChange` 事件会自动触发。

> Have fun!