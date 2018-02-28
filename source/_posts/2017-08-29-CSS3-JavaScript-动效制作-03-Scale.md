---
title: CSS3+JavaScript 动效制作 - 03 Scale
categories: 程序猿
tags:
  - CSS3
  - JavaScript
  - 动效
  - 前端
  - 缩放
  - HTML5
abbrlink: 4881c5ee
date: 2017-08-29 22:35:07
---

`CSS3+JavaScript 动效制作`这个系列是我自己的动效学习笔记，基于动效周期表的这个项目 **[Motion Periodic Table](http://foxcodex.html.xdomain.jp/index.html)** 制作，原项目的作者来自日本的 Kazuki Akamine，所有的动效均使用 `After Effect` 制作。上一篇完成了 [Move 运动](http://kris2d.info/posts/3246e3bb/)效果，本期我将继续使用 **[anime.js](http://animejs.com/)** 库配合 CSS3 来制作周期表中的 [Scale 缩放](http://foxcodex.html.xdomain.jp/Scale.html)动效。

<p align="center">
![Scale](https://user-images.githubusercontent.com/5259084/36770002-e62d1b70-1c9b-11e8-9d47-49c5249a9d54.gif)
</p>

<!--more-->

-----

实现代码及效果如下：

{% codepen MrCotter GvwRxy 0 defaultTab 265 %}

### 基本样式

```css
.square {
  width: 0;
  height: 0;
  border: none;
  background: #ffffff;
}
```

一个简单的 `div` 元素，初始宽度和高度均为 0px，无 `border`，背景色设为白色。

### 动效实现

```javascript
var scaleSquare = anime.timeline({
  loop: true
});

scaleSquare
.add({
  targets: '.el',
  width: [0, 50],
  height: [0, 50],
  duration: 700,
  easing: 'easeInOutSine',
  delay: 250
})
.add({
  targets: '.el',
  width: [50, 200],
  easing: 'easeInOutSine',
  duration: 700,
  delay: 250
})
.add({
  targets: '.el',
  height: [50, 200],
  easing: 'easeInOutSine',
  duration: 700,
  delay: 250
})
.add({
  targets: '.el',
  width: [200, 0],
  height: [200, 0],
  easing: 'easeInOutSine',
  duration: 700,
  delay: 250
})
.add({
  targets: '.el',
  width: 0,
  height: 0,
  duration: 100
})
```

从原始的缩放动态效果来看，整体分为 `4` 步，并且我们可以观察到，每个阶段之间都有**短暂的停顿**：

1. 从**什么都没有**到一个**小正方形**；
2. **小正方形**的宽度迅速增加，变为一个**长方形**；
3. **长方形**的高度逐渐增加到和宽度一致，变为一个**大正方形**；
4. 最后，**大正方形**迅速缩小到**什么都没有**，动效结束。

因此，使用代码实现动效也需要经过不同的阶段，这里需要引入 [anime.js](http://animejs.com/) 的 [Basic Timeline](http://animejs.com/documentation/#basicTimeline)，在一个动作完成后开始下一个动作，一个动作包含在一个 `.add(...)` 中，按顺序执行。每个阶段的缩放涉及到 div 元素 `width` 和 `height` 的大小变化，例如 width: [0, 50]，表示宽度从 0px 增加至 50px，官方文档见 [Specific Initial Value](http://animejs.com/documentation/#specificInitialValue)。其它参数的作用在之前的两篇文章中均有提及，`duration` 表示该动作完成的时间，`easing` 这里设为 `easeInOutSine`，符合原动效的速度变化方式，而非一成不变的速度。每个动作开始前也都设有 delay 延时。

在最后，我增加了一个时间极短的动作，保持 div 元素的宽高为 0。如果没有这一条，在开启循环的时候，上一个动作结束后会有大小为 200×200px 的正方形一闪而过，我不太确定这是否是 anime.js 的 bug，希望有高手能解答。因此，我在最后正方形缩小到没有之后，增加了上述动作，才得以还原原始的缩放动效。

到此，缩放效果就完成了，可以参考周期表页面的应用内容尝试做更复杂的动效。

下一期我们继续。Peace!



 


