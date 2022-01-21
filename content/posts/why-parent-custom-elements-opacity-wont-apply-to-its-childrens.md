---
title: 为什么父自定义元素的不透明度不影响子元素
description: 一个迷惑但简单的问题
date: 2021-02-11 10:44:58
isCJKlanguage: true
categories:
  - '浏览器'
tags:
  - 'Web Components'
  - '自定义元素'
  - '跨浏览器'
keywords: 
  - '透明度'
  - '子元素'
  - '父元素'
ShowToc: true
draft: false
---
## 背景

我尝试利用 Web Components 中的自定义元素创建文章列表元素和文章元素，这样可以将逻辑隐藏到元素的类中，从而降低心智负担。

应用有搜索文章功能，在搜素无结果的情况下，我想通过设置文章列表元素的透明度，隐藏文章列表。

## 问题

然而，这个元素在 Firefox 中与其他基于 Webkit 的浏览器表现不一致。

在 Firefox 里，改变这个元素的透明度会影响到其子元素，而在其他浏览器上不会，[这里有个例子](/misc/different-behaviour-across-browsers.html "不同浏览器表现不一")，可以用 Firefox 和 Chrome 对照着看。

例子中简单对比了`<div><div></div></div>`和`<host-element><div></div></host-element>`这两种情况。可以看到，在 Chrome 中，隐藏外层元素后，前者的子元素也同时隐藏了，而后者的子元素没有；而在 Firefox 中，两者的子元素都隐藏了。到底是什么问题呢？

## 寻找原因

我一开始以为是 Chrome 对自定义元素的支持不佳，于是去 Can I Use 查找兼容性表格[^compact-table]，发现早在 Chrome v67[^chrome-v67-release-date] 时，Chrome 就已经完整支持自定义元素了，所以这并不是 Chrome 中的自定义元素有问题。

[^chrome-v67-release-date]: 于 2018 年 4 月发布。

[^compact-table]: [Custom Elements (V1)](https://caniuse.com/custom-elementsv1)

排除了上述可能，剩下就只有样式了。因为元素在 Firefox 中的表现更符合我的期望，所以我认为这可能是 Chromium 的问题，于是我在 Chromium 的[问题追踪平台](https://bugs.chromium.org)中搜索是否已经有人发现这一现象。

因为不确定具体是哪方面出了问题，我只能直接以`opacity`作为关键词搜索，一条条排除。还好，搜索结果一共就 200 多条，找起来不是很费劲。

很快，我就看到两个贴子，似乎提到了遇到的问题：

- [Issue 764011: Block elements in inline elements with opacity:0 are displaying](https://bugs.chromium.org/p/chromium/issues/detail?id=764011)

- [Issue 836244: Text inside the block part of an IB split isn't affected by parent opacity.](https://bugs.chromium.org/p/chromium/issues/detail?id=836244)

注意到这两个帖子都提到了内联元素：

> ... in **inline** elements...

以及

> **Text** inside...

### 分析

> 难道这个问题与内联元素有关？但我没有用到内联元素啊。

我真的没有用到内联元素吗？再看[前面的例子](/misc/different-behaviour-across-browsers.html)。

`head`中没有可疑部分，`style`中只有一个 class：

```html
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Document</title>
<style>.hidden { opacity: 0; transition: opacity 0.3s ease-in; }</style>
```

`body`元素内的前几行：

```html
<div id="a">
  <span style="padding: 0.5rem; background-color: lightgrey;">foo</span>
</div>
```

显然都是块级元素，没问题。

`script`标签中的代码呢？

```javascript
class HostElement extends HTMLElement {
  constructor() {
    super();
    this.attachShadow( { mode: 'open' } );
  }
}
customElements.define('host-element', HostElement);
```

自定义元素的类声明，以及标准要求的定义元素操作，看起来没啥问题。

```javascript
const div = document.createElement('div');
div.style = 'padding: 0.5rem; background-color: lightgrey;';
div.textContent = 'bar';

const host = new HostElement();
host.shadowRoot.append(div);
```

创建`div`元素，设置其样式和内容；创建自定义元素的实例；把`div`加到自定义元素中的影子 DOM 中。看不出问题。

```javascript
const btn = document.createElement('button');
btn.textContent = 'Toggle hidden';
btn.addEventListener('click', e => {
    host.classList.toggle('hidden');
    document.getElementById('a').classList.toggle('hidden');
});

document.body.append(host, btn);
```

创建`button`元素，设置其内容，绑定点击事件，在点击时切换自定义元素和前面HTML中`div`的 class，最后把自定义元素和按钮都添加到`body`中。还是看不出问题。

这个内联元素究竟在哪儿？

---

打开 Chrome 浏览器开发人员工具，检查自定义元素，查看其样式计算值：

![自定义元素的样式计算值](/image/custom-elements-computed-style.png)

自定义元素的`display`的值居然是`inline`，我之前理所当然地认为其`display`的值是`block`！

标准是否规定了自定义元素`display`属性的默认值？打开 HTML 标准，找到自定义元素部分，这是标准对自定义元素的定义[^custom-elements-definition]：

[^custom-elements-definition]: [4.13.3 Core concepts](https://html.spec.whatwg.org/multipage/custom-elements.html#custom-elements-core-concepts "自定义元素的定义")

![自定义元素的定义](/image/spec-definition-of-custom-elements.png)

标准表明自定义元素属于 Phrasing Content，或短语元素。

实际上，HTML5 标准没有定义块级元素和内联元素，很多原本叫内联元素的元素，都属于短语元素一类。通过查阅 MDN[^phrasing-content-elements]可以发现，短语元素包含了许多熟知的内联元素：

[^phrasing-content-elements]: [短语元素（Phrasing content）](https://developer.mozilla.org/zh-CN/docs/Web/Guide/HTML/Content_categories#%E7%9F%AD%E8%AF%AD%E5%85%83%E7%B4%A0%EF%BC%88phrasing_content%EF%BC%89 "属于短语元素的元素")

![属于短语元素的元素](/image/phrasing-content-on-mdn.png)

我在标准中，确实没找到自定义元素或短语元素`display`的具体取值，似乎是标准[^spec-of-rendering]选择留给浏览器用户代理实现，但在 Google Developers 上有篇说明 Web Components 的文章[^article-mentioning-web-components]，其中提到了自定义元素的`display`默认值为`inline`：

[^spec-of-rendering]: [15 Rendering](https://html.spec.whatwg.org/multipage/rendering.html#rendering "标准中有关渲染的部分")

[^article-mentioning-web-components]: [Custom Element Best Practices](https://developers.google.com/web/fundamentals/web-components/best-practices#shadow-dom)

> Set a `:host` display style (e.g. `block`,`inline-block`, `flex`) unless you prefer the default of `inline`.
>
> - Why?
>
>   - Custom elements are `display: inline` by default...

我就把这篇文章当作事实标准吧。

### 为什么 Chromium 会出问题

明白了这个内联元素在哪儿，回过头看看前面提到的两个 issue。

- [Issue 764011: Block elements in inline elements with opacity:0 are displaying](https://bugs.chromium.org/p/chromium/issues/detail?id=764011)

- [Issue 836244: Text inside the block part of an IB split isn't affected by parent opacity.](https://bugs.chromium.org/p/chromium/issues/detail?id=836244)

两者均是内联元素[^no-content-categories]中存在块级元素后发生的问题，违反了内联元素中不能出现块级元素这一规则。

[^no-content-categories]: 为了避免引入过多新概念，后文仍然使用“内联元素”。要了解 HTML5 标准的内容分类，参见 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/Guide/HTML/Content_categories)

虽然说在内联元素中出现块级元素违反了规则，但也不至于子元素直接无视父元素的透明度。点开两个 issue，其中第二个帖子中提到了相关的 CSS 规范[^opacity-css-spec]，规范说：

[^opacity-css-spec]: [11. Transparency: the opacity property](https://drafts.csswg.org/css-color/#transparency)

> Opacity can be thought of as a postprocessing operation. Conceptually, after the element (including its descendants) is rendered into an RGBA offscreen image, the opacity setting specifies how to blend the offscreen rendering into the current composite rendering.

提到了按元素，且包括其子元素，应用透明度，没有提到内联元素或块级元素。由此可以推知，Chromium 的行为并不符合规范。

第二个帖子中还有回复提到，Chromium 没有正确构造[Layout Tree](https://developer.mozilla.org/en-US/docs/Web/Performance/How_browsers_work#layout)[^wrongly-constructed-layout-tree]，导致本应在内联元素内部的块级元素，出现在内联元素之外，所以透明度没有作用到块级元素上。

[^wrongly-constructed-layout-tree]: [I've attached the box trees for Firefox and Chromium for the following example: `data:text/html,<span id=target style="opacity:0.2">inline text<div>block text</div></span>`](https://bugs.chromium.org/p/chromium/issues/detail?id=836244#c16)

## 解决方案

明白了 Chromium 还没有修复这种情况下的渲染问题，那么避开就好了：将父元素的`display`设置成非`inline`即可。[符合规范的例子](/misc/different-behaviour-across-browsers-a-more-spec-compliant-example.html)。毕竟，内联元素中本就不该出现块级元素。
