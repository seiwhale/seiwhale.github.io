---
title: 前端应该知道的Wweb Components
date: 2019-8-6 14:57:32
tags:
  - javascript
  - component
categories:
  - 技术随笔
keywords: 'Html,component,javascript'
description: Web Components是允许我们创建可重用的定制元素（即组件）的一种技术，可以在你喜欢的任何地方重用，不必担心代码冲突，本质就是组件化。由于web components是由w3c组织去推动的，因此它很有可能在不久的将来成为浏览器的一个标配。
cover: https://cdn.jsdelivr.net/gh/seiwhale/assets/img/web_components.jpg
---

看到这个标题可能有的小伙伴想到的是`React` 、`Vue` 或者 `Angular` 的自定义组件，**no no no！**

`Web Components` 是一套不同的技术，允许您创建可重用的定制元素（它们的功能封装在您的代码之外）并且在您的web应用中使用它们。

## 流行组件库现状

前端目前比较主流的框架有 `react`，`vuejs`，`angular` 等。

我们通常去搭建组件库的时候都是基于某一种框架去搭建， 比如 `ant-design` 是基于 `react` 搭建的 `UI` 组件库，而 `elementUI` 则是基于 `vuejs` 搭建的组件库。虽然目前社区有相关工具，提供框架之间的转化服务，比如讲 `vuejs` 组件转化为 `react` 组件。 但是毕竟是不同的框架，有不同的标准。因此框架api发生变动，那么你就需要重写转化逻辑， 显然是不灵活的。

那么有没有更好的方案，一次编写，到处使用呢？

答案就是 `Web Components`，一起去看看是什么吧！

## 什么是web-component

`Web Components` 是允许我们创建可重用的定制元素（即组件）的一种技术，可以在你喜欢的任何地方重用，不必担心代码冲突，本质就是组件化。由于web components是由w3c组织去推动的，因此它很有可能在不久的将来成为浏览器的一个标配。

`Web Components` 主要有下面四部分组成：

- **Custom elements（自定义元素）**：一组JavaScript API，允许您定义custom elements及其行为，然后可以在您的用户界面中按照需要使用它们。
 -  **Shadow DOM（影子DOM）**：一组JavaScript API，用于将封装的“影子”DOM树附加到元素（与主文档DOM分开呈现）并控制其关联的功能。通过这种方式，您可以保持元素的功能私有，这样它们就可以被脚本化和样式化，而不用担心与文档的其他部分发生冲突。
-  **HTML templates（HTML模板）：**[`<template>`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/template "HTML内容模板（<template>）元素是一种用于保存客户端内容机制，该内容在加载页面时不会呈现，但随后可以在运行时使用JavaScript实例化。") 和 [`<slot>`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/slot "HTML <slot> element—是 Web Components 技术的一部分，是自定义web组件的占位符，目的是分离创建DOM树和DOM树的表现（）。") 元素使您可以编写不在呈现页面中显示的标记模板。然后它们可以作为自定义元素结构的基础被多次重用。
-   **[HTML Imports（HTML导入）](https://developer.mozilla.org/zh-CN/docs/Web/Web_Components/HTML%E5%AF%BC%E5%85%A5)：**一旦定义了自定义组件，最简单的重用它的方法就是使其定义细节保存在一个单独的文件中，然后使用导入机制将其导入到想要实际使用它的页面中。HTML 导入就是这样一种机制，尽管存在争议 — Mozilla 根本不同意这种方法，并打算在将来实现更合适的。

## 优缺点

使用 `web components` 搭建组件库能够带来什么好处呢？前面说了，`web components` 是 `w3c` 推动的一系列的规范，它是一个标准。如果我们使用 `web components` 的 `api` 开发一个组件，这个组件是脱离框架存在的，也就是说你可以在任何框架中使用它，当然也可以直接在原生 `js` 中使用。

- **互操作性** — 组件超越框架而存在，可以在不同的技术栈中使用
- **寿命** — 因为组件的互操作性，它们将有更长的寿命，基本不需要为了适应新的技术而重写
- **移植性** — 组件可以在任何地方使用，因为很少甚至没有依赖，组件的使用障碍要明显低于依赖库或者框架的组件

但同时由于 `web components` 的标准暂时还是不稳定的，并且缺少文档、范例、教程，想要入门是一件相当困难的事情，就好像你直接用原生 `canvas api` 去开发游戏一样，尤其是如果要支持旧式浏览器。

## 使用方法
声明自定义标签:
```
window.customElements.define(name, constructor, options?);
```
其中 `name` 为自定义标签的标签名。
`constructor` 定义自定义标签的功能。
`options` 为 `{ extends: htmlTagName }` ，定义自定义标签继承于哪种组件。例如：

``` javascript
class AppDrawer extends HTMLElement {
  constructor() {
    super(); // always call super() first in the constructor.
    
    // 开启 shadow dom 根结点
    const shadowRoot = this.attachShadow({mode: 'open'});
    shadowRoot.innerHTML = `
      <!-- style中的样式只在组件内部生效 -->
      <style>#tabs { ... }</style>
      <!-- id的作用域只在组件内部 -->
      <div id="tabs">...</div>
      <div id="panels">...</div>
    `;
  }
  
  // 标签被添加到dom中时触发
  connectedCallback() {
    ...
  }
  
  // 标签被移除时触发
  disconnectedCallback() {
    ...
  }
  
  // 属性变化时触发
  attributeChangedCallback(attrName, oldVal, newVal) {
    ...
  }
}
```

定义后即可在html中直接使用：

``` html
<app-drawer></app-drawer> 
```

#### 插槽

在定义组件内部模版时可使用 `<slot>` 标记声明此处未来可能会被外部元素代替，以增强组件的扩展性。如果有 `Vue` 开发经验应该对 `slot` 并不陌生。

使用：
``` html
<better-button>
  <!-- 具名插槽 -->
  <img src="gear.svg" slot="icon"> 
  <span>Settings</span>
</better-button>
```
定义：
``` html
#shadow-root
  <style>...</style>
  <!-- 具名插槽 -->
  <slot name="icon"></slot>
  <span id="wrapper">
  <!-- 默认插槽 -->
    <slot>Button</slot>
  </span>
```

#### 样式
`Shawdom Dom` 的开启会完全隔绝组件内外，所以外部的样式是无法作用的组件内部的（插槽除外）。如果需要在外部修改内部样式只能使用 `css var()` 函数。

外部样式：
``` css
app-drawer {
  --text-color: red;
}
```
内部样式：
``` css
#tabs {
  color: var(--text-color, green);
}
```

## Web Component 库

如何更简便的使用这个技术呢？现阶段流行的库有 [Polymer](https://www.polymer-project.org/1.0/)、[stencil]([https://stenciljs.com/](https://stenciljs.com/)
/) 和 [Bosonic](http://bosonic.github.io/documentation/getting-started/getting-the-code.html)。它们提供了一系列能够被使用和拓展的组件。正如所有流行的项目一样，它们有健全的文档、范例和稳定的社区。

但是，这些库会让组件拥有一个重要的依赖，使用这些组件的用户必须去确保这个依赖的存在。下面我简单列一个 `stencil` 的例子：

``` js
import { Component, Prop, State } from '@stencil/core';

@Component({
  tag: 'my-component',
  styleUrl: 'my-component.scss'
})
export class MyComponent {
  // Indicate that name should be a property on our new component
  @Prop() first: string;

  @Prop() last: string;

  @State() isVisible: boolean = true;

  render() {
    return (
      <p>
        Hello, my name is {this.first} {this.last}
      </p>
    );
  }
}
```

还挺简单的~

## 总结

`Web components` 的出现真正意义上原生实现了样式私有化，DOM作用域及扩展原生组件，让我们应用的构建方式可以迅速地适应变化，不用再担心多个组件库不能共用组件的问题。`web components` 是基于最新的 web 标准，所以它的的互操作性，可以让我们的代码拥有更长的寿命。最终，因为更少的依赖和前置条件，我们的组件可以满足更多人的需求。

想要深入了解有关 `Web Components` 的更多信息？大家可以查看 `MDN` 官网或者持续关注本人，我会不时更新技术文档~



