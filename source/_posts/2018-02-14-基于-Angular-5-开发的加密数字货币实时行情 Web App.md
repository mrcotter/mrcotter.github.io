---
title: 基于 Angular 5 开发的加密数字货币实时行情 Web App
categories: 程序猿
tags:
  - Angular
  - Cryptocurrency
  - Bitcoin
  - 数字货币
  - 加密货币
  - 网站开发
abbrlink: c4863de1
date: 2018-02-14 21:05:50
---

[Angular](https://angular.io/) 是基于 [TypeScript](https://www.typescriptlang.org/) 的 Javascript 框架，由 Google 进行开发和维护，并且需要说明的是，它与 AngularJS（旧版本）不兼容，而这次开发的碰到的许多问题，搜索 Angular 相关的内容，绝大多数都是基于 AngularJS 这一旧版本的，可见当时受欢迎的程度。随着这几年前端框架的流行和发展，逐渐形成了目前的三分天下之势，`Angular`、`Vue` 和 `React`，后两者分别由 Facebook 和 Google 前员工 Evan You 开发和维护。我自己出于学习一种前端框架的目的，选择 Angular 这一成熟的框架，开发了一个展示`数字加密货币（Cryptocurrency）`的 Web App，欢迎浏览 [https://crypto.kris2d.info](https://crypto.kris2d.info/) 进行查看。

<!--more-->

------

### 简单介绍

Cryptocurrency Market 是基于 Angular 5 框架开发的 Progressive Web App，用于显示热门加密数字货币的实时行情和历史价格走势。目前支持 60 种货币，可以获取并更新实时价格信息，数据来源 [CryptoCompare](https://www.cryptocompare.com/api/)。点击每一种货币可以进一步浏览最近 24h、7 天、1 个月以及 1 年的价格走势，后续会增加更多的币种和排序功能。

### 平台环境

* [Angular 5.2.9](https://angular.io/)
* [Angular-Cli 1.7.4](https://github.com/angular/angular-cli)
* [NG-ZORRO 0.6.15](https://ng.ant.design/#/docs/angular/introduce)
* [Chart.js 2.7.2](http://www.chartjs.org/)
* [Nginx 1.12.2](https://nginx.org/en/)

### 特性

* 纯前端开发，无后端代码；
* 服务器采用 Nginx 作 Reverse Proxy，http-server 和 PM2 用于设置不间断运行的静态服务器；
* 应用了 `Ant Design` 简化了部分视觉代码，视觉风格上保持统一；
* 每 15 秒获取最新的价格数据，如果价格产生变化，根据涨跌情况渲染不同颜色的背景色作为提示（具体的效果个人感觉没有很流畅，后续进行完善）；
* 价格走势图使用 `Chart.js` 绘制，根据需求进行了许多自定义修改，例如增加了伴随鼠标移动的纵向线，增加了图标数据的可读性，手机端纵向线也会随着手指的触摸左右移动，美观上还有待改善；
* 针对移动端做了显示优化，包括最新的 iPhone X;
* 支持 Google 的 Progressive Web Apps 特性，可以添加到 iOS 和 Andorid 手机桌面或者 Chrome 浏览器的 Apps 页面中。

![Cryptocurrency Market](https://user-images.githubusercontent.com/5259084/37133234-2f08308c-22e6-11e8-8c31-ea321f825ae6.png)

![Cryptocurrency Market](https://user-images.githubusercontent.com/5259084/37133243-383f6760-22e6-11e8-850c-863b2e912cd7.png)

![Cryptocurrency Market](https://user-images.githubusercontent.com/5259084/37133247-3e5bab72-22e6-11e8-8df3-ec6d9a82257b.jpg)

### 使用

打开你的浏览器，访问 [https://crypto.kris2d.info](https://crypto.kris2d.info/)

### To-do

* 增加更多数字货币，可能的话，按照最新的市值动态获取数据；
* 增加首页表格的排序功能；
* 获取行情相关新闻并增加到首页内容中；
* favicon 在上线后没有正常显示，本地测试没有问题，之后需要解决；
* 服务器 nginx 配置和性能优化（原打算将 nginx 配合另一个 http server 作为 reverse proxy 来使用，无奈总是遇到 403 forbidden error，后续再调整）；
* 其它样式和功能完善，之后尝试制作一个微信小程序版本。

### 24/02/2018 小更新

* 显示 Top 20 数字货币，之后如果再增加考虑对表格分页；
* 增加了按货币名称排序的功能，为此重新调整了部分数据结构，以适应排序事件触发的算法；

### 07/03/2018 再次更新

* 全站启用 HTTPS（支持 HTTP/2），使用 Let's Encrypt 的免费证书；
* 增加数字货币总量到 60，以后有需要再增加，常用的基本都有了；
* 增加了查找和排序功能，可以切换每页显示的货币数量，默认 20/page；
* 为了偷懒，集成了 Travis CI 进行自动化 build 和 deployment。

### 05/04/2018 再次更新

* 增加 Progressive Web Apps 特性，可以将 App 添加至 Android 手机桌面直接运行（需配合使用 Chrome，浏览页面时会有提示信息）；
* 完善多平台多系统的图标及配置信息，例如支持 iPhone、Android 及 Windows 等操作系统，也支持 TouchBar 和 Windows Tile 图标显示。

### 后话

这次开发前前后后还是遇到了不少坑，有关技术填坑的内容，以后找机会聊，Peace！
