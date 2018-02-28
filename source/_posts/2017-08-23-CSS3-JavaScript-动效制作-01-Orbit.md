---
title: CSS3+JavaScript 动效制作 - 01 Orbit
categories: 程序猿
tags:
  - CSS3
  - JavaScript
  - 动效
  - 前端
  - 轨道旋转
  - HTML5
abbrlink: 872af450
date: 2017-08-23 14:56:41
---

`CSS3` 和 `JavaScript` 已广泛应用于网页和前端相关的动态效果开发以及 App 的原型演示工具中。CSS 作为最轻盈灵巧的前端动效实现方式，借助 Translate, Transform, Scale, Position, Easing 等可以快速实现多种动画和过渡效果。虽然有其自身的局限，比如不能操控 `Dom` 节点，监控事件等需要追踪事件的交互行为，因此在很多场景应用中需结合 Javascript 来达到完善的动效。

我接触动效开发的时间不长，目前仍然处于初期的学习阶段，而 `CSS3+JavaScript 动效制作`这个系列即作为我自己的动效学习笔记。在不久之前，我发现了一个动效周期表的网站项目 **[Motion Periodic Table](http://foxcodex.html.xdomain.jp/index.html)**，来自日本的 Kazuki Akamine，很有意思的动效元素表，作者的动效均使用 `After Effect` 制作。

![Motion Periodic Table](https://user-images.githubusercontent.com/5259084/36770132-8db123b4-1c9c-11e8-8e37-e96ec3b88c17.gif)

<!--more-->

动效制作学习笔记是从上述周期表中抽取不同的元素，使用 CSS3 和 JavaScript 实现同样的动态效果。我们从表中可以看到，动态的难度并不是顺序排列的，因此制作的动效会先从简单的入手，逐渐增加难度。同时，我会在每篇笔记中记录代码实现的过程，这也可以算作是入门的教程吧。

JS 在 Web 中的应用很广，我引入了一个较为流行的开源项目 **[anime.js](http://animejs.com/)** 用于动效的快速开发，在代码的编写上更为简洁明了。

-----

### Orbit

第一个基础动效 [Orbit 轨道旋转](http://foxcodex.html.xdomain.jp/Orbit.html)，如下图所示，一个小球绕着大的圆形轨道运动，是最常见的动效之一。

<p align="center">
![Orbit](https://user-images.githubusercontent.com/5259084/36770135-93f30102-1c9c-11e8-8c2e-da135109b201.gif)
</p>

实现代码及效果如下：

{% codepen MrCotter LjmaYQ 0 defaultTab 365 %}

### 基本样式

```css
.circle {
  background-color: #464343;
  display: block;
  border-radius: 50%;
}

.circle-1 {
  width: 200px;
  height: 200px;
  border: 2px solid #BFBFBF;
  position: relative;
}

.circle-2 {
  width: 32px;
  height: 32px;
  border: none;
  background: #ffffff;
  position: absolute;
  top: 0;
  left: 50%;
  margin: -16px 0 0 -16px;
}
```

以上是与两个球形相关的样式代码。大的球形作为运动轨道，背景色与页面背景相同，加入 2px 的 border；小球使用单纯的白色。这里当 `circle-1` 的 `position` 为 `relative`，其子元素 `circle-2` 的 `position` 设为 `absolute`，那么 `circle-2` 的位置将相对于 `circle-1` 变化，因此 `top: 0; left: 50%`，`margin` 的偏移量为其半径的负值，即可将 `circle-2` 的起始位置设为运动轨道的正上方。

### 动效实现

接下来是让小球动起来，围绕大球的中心旋转。实现方法至少应有 2 种，一种是通过计算小球在运动轨道上的运动位置，然后根据计算结果设置小球的 top、left 数值，达到移动效果，这种方法需要用到一些数学公式；另一种实现起来更加简单，将大球小球视为一个整体，整体旋转 360 度即可，我使用的是该方法。

```javascript
var orbit = anime({
  targets: '.orbit',
  rotate: 360,
  duration: 5000,
  easing: 'linear',
  loop: true
});
```

通过调用 **[anime.js](http://animejs.com/)** 的方法可以轻松实现该效果。`targets` 用于定位包含了 `circle-1` 和 `circle-2` 的 `div` 元素，即之前所说的整体。`rotate` 为旋转的动作，`duration` 用于控制动作的快慢，数值越大，完成动作的时间就越长。`easing` 需设为线性移动 `linear`，从头到尾都是同样的速度。其它预设移动如 `ease-in` 为开始的速度慢，之后逐渐变为正常速度，相关内容可以参考 [CSS3 animation-timing-function Property](https://www.w3schools.com/cssref/css3_pr_animation-timing-function.asp)。最后，`loop` 设置是否循环动作。

到此，第一个简单的轨道运动效果完成了，可以参考周期表页面的应用内容实现更复杂的效果。

下一期我们继续。


