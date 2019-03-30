---
title: react-ace placeholder & 受控处理
date: 2019-03-29 23:42:03
tags:
---

我们在开发的过程中，可能会用到类似富文本编辑器这样的控件。在react体系中，react-ace是首选。这里我们来看看使用这个组件时，会遇到的问题。

## 1、placeholder

最新的官方文档里面其实是有 placeholder 这个属性配置，但是即使你安装来最新版本的react－ace，你会发现它并不生效。在这种情况下，我们能做的就是自己来实现placeholder的功能。

设置来placeholder属性后的效果如下（其实并没有生效）：

![no-placeholder](react-ace-placeholder-01.gif)


