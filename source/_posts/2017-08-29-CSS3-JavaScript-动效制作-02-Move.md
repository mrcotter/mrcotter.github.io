---
title: CSS3+JavaScript 动效制作 - 02 Move
categories: 程序猿
tags:
  - CSS3
  - JavaScript
  - 动效
  - 前端
  - 物体移动
  - HTML5
abbrlink: 3246e3bb
date: 2017-08-29 00:48:52
---

`CSS3+JavaScript 动效制作`这个系列是我自己的动效学习笔记，基于动效周期表的这个项目 **[Motion Periodic Table](http://foxcodex.html.xdomain.jp/index.html)** 制作，原项目的作者来自日本的 Kazuki Akamine，所有的动效均使用 `After Effect` 制作。上一篇完成了 [Orbit 轨道旋转](http://kris2d.info/posts/872af450/)这一基础动效，今天我使用代码来制作周期表中的 [Move 运动](http://foxcodex.html.xdomain.jp/Move.html)效果。

<p align="center">
![Move](https://user-images.githubusercontent.com/5259084/36770007-e96f4592-1c9b-11e8-8331-a5b86aabf63a.gif)
</p>

<!--more-->

-----

实现代码及效果如下：

{% codepen MrCotter MvPbQq 0 defaultTab 388 %}

### 基本样式

```css
.square-large {
  width: 192px;
  height: 300px;
  border: 2px solid #BFBFBF;
  outline: 64px solid #464343;
  position: relative;
  z-index: 100;
}

.square {
  width: 32px;
  height: 32px;
  border: none;
  background: #ffffff;
  position: absolute;
  top: 50%;
  left: 198px;
  z-index: 90;
}
```

以上是两个方形的样式代码，当运动的方形移动到边缘后，超出的部分需要隐藏。这里我使用了一个简单的方法，为外框元素增加 `outline` 属性，颜色与背景相同。同时为两个方形都增加 `z-index`，外框赋值大于运动方块，这样就能遮盖超出的部位，之后只要大致计算方形的运动距离即可。

### 动效实现

```javascript
var moveSquare = anime({
  targets: '.square',
  translateX: -240,
  duration: 1500,
  easing: 'linear',
  loop: true,
  delay: 600
});
```

通过调用 **[anime.js](http://animejs.com/)** 的方法可以轻松实现该效果。`targets` 定位到 `square` 的 `div` 元素。`translateX` 代表向左横向移动 240px，`duration` 用于控制动作的快慢。`easing` 需设为线性移动 `linear`，从头到尾都是同样的速度。`loop` 设置是否循环动作。最后，原动效中方块移动至左边缘后并没有立即出现在右边，因此加入了 `delay`，使循环的动作有一定的延迟。

到此，物体平移的运动效果就完成了，可以参考周期表页面的应用内容尝试做更复杂的动效。

下一期我们继续。Peace!


