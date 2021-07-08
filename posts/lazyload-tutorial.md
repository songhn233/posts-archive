---
title: "Lazyload 实践指北"
date: 2020-09-10T20:30:27+08:00
description: Lazyload 懒加载作为一项前端优化技术能够有效加快首屏加载、提升用户使用体验等，本文是基本的 lazyload 和在图片、JavaScript插件等方面的实践指北
summary: Lazyload 懒加载作为一项前端优化技术能够有效加快首屏加载、提升用户使用体验等，本文是基本的 lazyload 和在图片、JavaScript插件等方面的实践指北
featured_image: https://img.songhn.com/img/lazyload-tutorial-banner.png?imageslim
categories:
- 前端
tags:
- 性能优化
---

## Lazyload 是什么

Lazyload 是一种在加载页面时延迟加载非关键资源的策略，作为一项前端优化技术有着非常多的使用场景与设计技巧，在网页中几乎各类元素都能通过 lazyload 来进行优化，像是图片视频等各类静态资源、非必要的 CSS 与 JS 文件、DOM操作等。

## 图片资源上的 lazyload 实践

### Native Lazyload

首先值得一提的是 `<img>` 标签本身的 `loading attribute` 就支持懒加载 images 与 iframes，通过 [计算与资源相差的距离](https://css-tricks.com/native-lazy-loading/) 来动态加载。不过浏览器支持度很感人，以下是 2020 年 9 月的浏览器支持度

![](https://img.songhn.com/img/lazyload-tutorial-loading-use.png?imageslim)

并且上面的方式与我们常见的 lazyload 方案有一些不同，会根据距离、网速等因素来延迟资源的加载，这样当然是十分优秀的，但是做不到资源在窗口时再进行加载与模糊效果等自定义行为，并且手动实现 lazyload 能实现更快的 `DOMContentLoaded` 与更好的使用体验

### 实现 Lazyload

图片的 lazyload 原理比较简单，在 `<img>` 标签上添加`data-src` 或类似的 `attribute` 存放图片链接，`src`则不设置或者显示缩略图，通过 `IntersectionObserver` 或 `scroll` 等 API 检测到当前视窗滚动或可能滚动到图片位置时把 `data-src` 的链接替换给 `src attribute` 以完成动态加载。需要注意的是尽量避免使用空的 `src` 标签，这样也会增加一个请求影响性能。

```html
<img data-src="image.jpg" src="thumbnail.jpg" >
```

不过上面的基础写法会产生许多潜在的问题：

#### 搜索引擎爬虫与 RSS

在 2020 年的今天，搜索引擎无法识别不带 `src` 的 `<img>` 标签的问题已经基本不用考虑了，`Google` 等现代搜索引擎已经支持了 lazyload 图片的爬取（当然古代搜索引擎下文也提到了解决方案），不过对于 `RSS` 来说目前还没有这种技术，好在我们有一个全新的 `attribute` —— `srcset`，`srcset` 和 `src` 的作用基本一致并且优先级高于 `src`，这就使得我们可以采用下方写法

```html
<img
  src="image.jpg"
  srcset="thumbnail.jpg"
  data-src="image.jpg"
>
```

浏览器会优先加载 `srcset`，当需要加载时 `data-src` 再替换 `srcset`，当前环境不支持`srcset`时会自动回退到 `src`，这个特性对于古代搜索引擎以及 RSS 等无法处理无 `src` 的场景来说是非常有效的，同时主流浏览器都已经支持该 `attribute` 。

![](https://img.songhn.com/img/lazyload-tutorial-srcset-use.png?imageslim)

#### 图片闪烁与自适应布局

lazyload 或者说图片本身会带来一个对用户很不友好的问题 —— 布局闪烁，一开始图片还未加载，加载完成后突然出现使得整个布局发生变化，这是很不美观的。对于这种问题有一些常见的解决方案：

首先是写死图片的宽高，不过这对于响应式图片不太友好；首先使用图片的缩略图，但即使是压缩以后的体积已经很小了，依然有可能在某一些网络中出现加载时的布局浮动；最后相对更合适的方法就是长宽比容器 ( Aspect Ratio Boxes )。

这种概念的原理就是首先通过包裹元素的 `padding` 进行占位，之后子元素也就是图片加载完成后利用绝对布局填上留存空间，有几种不同的写法，首先最简单就是利用 `background-image`（以下例子均使用 `56.25%` 即 16 : 9 的比例）：

```html
<div class="ratio-box">There is an image.</div>
```

```css
.ratio-box {
  height: 0;
  overflow: hidden;
  padding-top: 56.25%;
  background-image: url('./image.jpg');
  background-size: cover;
}
```

这样随着窗口大小的变化图片大小也会变化但比例始终保持 16 : 9，接下来简单介绍一下这段的原理。在大部分情况下设置 `height` 百分比是没有效果的（只有当父元素有绝对高度或者使用绝对定位脱离文档流时才会生效），而 `padding` 则不同，它是根据父元素的 `width` 算出来的，而 `width` 一般情况继承 `100%`，并且这也恰好符合宽高比固定的要求。当然假如需要使用 `<img>` 的话也有一种套一层 `wrapper` 的写法：

```html
<div class="box-wrapper">
  <img class="box" src="image.jpg" >
</div>
```

```css
.box-wrapper {
  height: 100%;
  padding-top: 56.25%;
  overflow: hidden;
  position: relative;
}

.box {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
}
```

有了之前的解释现在使用 `wrapper` 的代码就很好理解了，还有一种伪类的写法可以参考 [CSS-TRICKS](https://css-tricks.com/aspect-ratio-boxes/)。

当然在实际应用中我们的所有图片肯定不是一样的比例，有一种解决方案是计算所有存在的图片平均比例。我们可以把这种方法得到的比例当作默认值，同时加入对 `style` 的 `padding-top` 覆写来实现一定的客制化。 

### 图片总结

虽然本身 lazyload 实现简单但是在实践也会遇到各种实现问题，本身的 `CSS` 和 `HTML` 也有许多值得关注的地方，至于也很关键的 `JS` 将在下文优化非关键脚本加载介绍，原理一致。

## 非关键脚本的 Lazyload

在一个站点上是有许多的脚本是不需要在首屏加载的。例如博客，评论系统以及其他插件不会对阅读产生关键影响，又会拖慢加载性能，我们可以利用 lazyload 来处理，当然也可以学习 Medium 的做法点击加载按钮后再加载评论（也可以和预加载结合）。

### IntersectionObserver

谈到 lazyload 似乎最适合的 API 就是 `IntersectionObserver` 了，它在有良好性能的同时也能有效的监听元素与祖先的交叉状态，而通过 `scroll` 进行监听则会消耗过多的性能，除非 IE 用户在兼容性考虑之中，否则采用上者会比较合适。

![](https://img.songhn.com/img/lazyload-tutorial-intersectionobserver-use.png?imageslim)

简单介绍一下 `IntersectionObserver`，它是一种异步观察观测元素和祖先或视窗交叉情况的方法，我们可以在观测对象发现待加载元素进入视窗后动态加载，基本使用很简单：

```js
const intersectionObserver = new IntersectionObserver(callback[, options]);
```

- `callback` : 回调函数，用于检测到交叉状态变化时进行操作。
- `options` :  配置参数对象。

回调函数会接受一个数组表示多个被检测的元素，数组每个元素都是 `IntersectionObserverEntry` 对象，具体定义可参考 [w3c文档](https://w3c.github.io/IntersectionObserver/#intersection-observer-interface)。不过在简单的 lazyload 中并不需要用到对象中复杂的参数，只需要 `IntersectionObserverEntry.isIntersecting` 用于判断是否是进入视窗而不是离开视窗，`IntersectionObserverEntry.target` 返回检测元素即可。我们先实现一下之前提到的图片懒加载：

```html
<div>
    <img class="lazy" data-src="image-1.jpg" >
    <img class="lazy" data-src="image-2.jpg" >
    ...
</div>
```

```js
const intersectionObserver = new IntersectionObserver(entries => {
  entries.forEach(item => {
    if (item.isIntersecting) {
      const image = item.target;
      image.src = img.dataset.src;
      intersectionObserver.unobserve(image);
    };
  });
});

const imagesList = Array.from(querySelectorAll('img.lazy')).forEach(item => {
  intersectionObserver.observe(item);
});
```

而在加载脚本时有一些需要注意的地方：假如是单文件可以直接插入，但假如有先后关系或者 `css` 文件的配合并且有 `async` 需求则可能需要使用 `onload` 等方法来控制顺序。同时也要考虑爬虫和脚本等情况的加载，下面是一个例子：

```js
const runningOnBrowser = typeof window !== "undefined";
const isBot = runningOnBrowser && !("onscroll" in window) || typeof navigator !== "undefined" && /(gle|ing|ro|msn)bot|crawl|spider|yand|duckgo/i.test(navigator.userAgent);
const supportsIntersectionObserver = runningOnBrowser && "IntersectionObserver" in window;

setTimeout(() => {
  if (!isBot && supportsIntersectionObserver) {
      const observer = new IntersectionObserver(entries => {
        if(entries[0].isIntersecting) {
          loadScript();
          observer.disconnent();
        }
     });
     const container = querySelector('.container');
     observer.observe(container);
  } else {
    loadScript();
  }
}, 0);
```

这就是简单的通过 `JS` 实现 lazyload 的过程。

> 上面代码通过 `setTimeout` 包裹则是一个小 trick，这样该段代码会进入事件循环，延迟加载不会影响 `DOMContentLoaded`。

## 上文未列出的参考资料

- [Intersection Observer - Web API 接口参考](https://developer.mozilla.org/zh-CN/docs/Web/API/IntersectionObserver)
- [HTMLImageElement.loading - Web API 接口参考](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLImageElement/loading)
- [图片 lazyload 的学问和在 Hexo 上的最佳实践](https://blog.skk.moe/post/img-lazyload-hexo/)
- [使 Disqus 不再拖累性能和页面加载](https://blog.skk.moe/post/prevent-disqus-from-slowing-your-site/)
- [结合响应式图片实现更好的图片懒加载方案](https://www.h404bi.com/blog/2019/07/better-lazyload-implementation-with-responsive-images/#fnref1)
- [A Native Lazy Load for the Web](https://css-tricks.com/a-native-lazy-load-for-the-web-platform/)
- [verlok/vanilla-lazyload : LazyLoad is a lightweight (2.4 kB) and flexible script](https://github.com/verlok/vanilla-lazyload)