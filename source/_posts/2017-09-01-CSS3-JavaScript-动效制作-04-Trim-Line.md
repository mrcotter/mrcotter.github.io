---
title: CSS3+JavaScript 动效制作 - 04 Trim Line
categories: 程序猿
tags:
  - CSS3
  - JavaScript
  - 动效
  - 前端
  - 裁切线
  - HTML5
abbrlink: 1f961ac4
date: 2017-09-01 21:46:54
---

`CSS3+JavaScript 动效制作`这个系列是我自己的动效学习笔记，基于动效周期表的这个项目 **[Motion Periodic Table](http://foxcodex.html.xdomain.jp/index.html)** 制作，原项目的作者来自日本的 Kazuki Akamine，所有的动效均使用 `After Effect` 制作。上一篇完成了 [Scale 缩放](http://kris2d.info/posts/4881c5ee/)效果，本期我将继续使用 **[anime.js](http://animejs.com/)** 库配合 CSS3 来制作周期表中的 [Trim Line 裁切线](http://foxcodex.html.xdomain.jp/TrimLine.html)动效。

<p align="center">
![Trim Line](https://user-images.githubusercontent.com/5259084/36769937-8c7e59cc-1c9b-11e8-8244-cde427876566.gif)
</p>

<!--more-->

-----

实现代码及效果如下：

{% codepen MrCotter oeJeVq 0 defaultTab 374 %}

### 基本样式

```css
.square-large {
  width: 200px;
  height: 300px;
  border: 2px solid #BFBFBF;
  position: relative;
}

.line {
  width: 0;
  height: 16px;
  float: left;
  border: none;
  background: #ffffff;
  position: absolute;
  top: 50%;
  left: 20px;
  right: auto;
}
```

为了还原动态效果，加入了一个长方形的外框，大小 200×300px，设置了 border 属性。另外一个 div 即为动效的主角 line，初始宽度为 0，高度 16px。剩下的 position、top、left 属性和初始位置有关，位于外框垂直距离的中间，离左侧边框 20px。最后，虽然 float 的默认值是 left，right 默认值为 auto，单独将它们放在代码中是与后面的动效变化有关，我会在后面进行说明。

### 动效实现

```javascript
var changeEl = $(".el");
var trimLine = anime.timeline({
  loop: true
});

trimLine
  .add({
    targets: ".el",
    width: [0, 160],
    duration: 800,
    easing: "easeOutQuart",
    delay: 50
  })
  .add({
    targets: ".el",
    width: [160, 160],
    easing: "linear",
    duration: 800
  })
  .add({
    targets: ".el",
    width: [160, 0],
    duration: 800,
    easing: "easeOutQuart",
    delay: 50
  })
  .add({
    targets: ".el",
    width: [0, 0],
    easing: "linear",
    duration: 800
  });

trimLine.update = function(anim) {
  if (anim.currentTime < 2500 && anim.currentTime > 850) {
    changeEl.css({ float: "right", left: "auto", right: "20px" });
  }

  if (anim.currentTime > 2500) {
    changeEl.css({ float: "left", left: "20px", right: "auto" });
  }
};
```

首先，分析原始的裁切线动效，整体可分为 `2` 步，并且可以观察到，每个阶段完成后都有**短暂的停顿**：

1. Line 从左侧**什么都没有**的状态，宽度迅速向右增加到 160px，形成一个**线条状的正方形**；
2. **线条状的正方形**的宽度迅速减少，从左向右逐渐减少到 0，再次变为一个**什么都没有**的状态。

因此，和上一期的动效一样，引入 [anime.js](http://animejs.com/) 的 [Basic Timeline](http://animejs.com/documentation/#basicTimeline)，每个动作包含在一个 `.add(...)` 中，按顺序执行。除了上述分解的2个步骤，2次停顿也需要一个动作来完成。有了上一期动效的制作经验，line 宽度的变化很简单，由 `width` 参数控制，停顿的状态宽度不变。而裁切线的动效中，`duration` 和 `delay` 的值很重要，需要记录下来参与之后的计算。

裁切线动效的难点在于 line 在后期宽度减少的过程中，方向需要反转，否则动效就仅仅是回到起点。为了实现这一点，需要在合适的时间点改变 line 元素的 `float` 属性值。我们将 `float` 设置为 `right` 后，line 的起点就从靠近左边框变成了靠近右边框，具体的 position 也需要设置对应的 `left` 和 `right` 值。

接下来，我们会用到 anime.js 的另一个功能 `Callbacks`，一共有 4 种：`begin`，`run`，`update` 以及 `complete`，具体的用法可以参考[官方文档](http://animejs.com/documentation/#allCallbacks)。这里我们会用到 `update`，`update` 对应的 function 会从动画开始执行的每一帧都会被调用，无视 `delay` 参数，这一点和 `run` 是有区别的。我们需要 line 在第 1 次停顿的阶段改变其 `float` 属性，并且直到第2阶段动作完成，之后在第 2 次停顿阶段将 `float` 属性再次改回 `left`，然后进入下一次动效循环。

`anim.currentTime` 可以返回动画执行的当前时间点，将其返回值大于或小于特定的时间点，执行相应的 function 就可以实现我们需要的功能了。那么，line 在进入第 1 次停顿状态前总共需要 `duration + delay = 800 + 50 = 850ms`，进入第 2 次停顿状态前，同理计算可得 `2500ms`。根据上述时间段设置 line 的 `float` 和 `position` 参数后，就可以还原原始的裁切线动效了。

下一期我们继续，Peace!


