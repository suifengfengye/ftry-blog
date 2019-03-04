---
title: reactRouter1-初识react router
date: 2019-02-25 17:43:09
tags: react-router
---

设想，我们正在使用react构建一个单页应用（SPA），我们应该怎么设置路由呢？在页面没有刷新的情况下，要更改页面的某个部分或者整个页面，这时候就是使用前端路由的时候了。配合react构建SPA的路由框架非react-router莫属。react-router更新到4.0版本，其设计思想已经和前几个版本不一样。其设计的思路是"动态路由"。何为"动态路由"?就是路由发生在app的渲染过程中！那与之相对应的就是"静态路由"，静态路由是在应用初始化前就已经定义好路由信息。所以，从定义上看是完全不一样的，那在使用的过程中也会有很大差别。

接下来我们看看react-router是如何工作的？

- (1) 把&lt;Router /&gt;组件放到应用的最顶层
- (2) 使用&lt;Link /&gt;组件定义路由
- (3) 最后，使用&lt;Router /&gt;组件展现路由对应的UI

如上，react-router会涉及到几个重要的组件，分别&lt;是Router /&gt;、&lt;Link /&gt;和&lt;Route /&gt;,翻译过来就是路由组件、导航组件和路由匹配组件。实际上，react-router也就是由这三类组件组成，所有这些组件都可以从"react-router-dom"这个npm包中导入使用。

我们来看看路由组件&lt;Router /&gt;,针对web应用，react-router提供了&lt;BrowserRouter /&gt;和&lt;HashRouter /&gt;。除了实现原理不一样外，使用上也有区别，&lt;BrowserRouter /&gt;支持服务端渲染而&lt;HashRouter /&gt;不支持，所以在使用的时候得注意这一点。

基于以上对react-router的简单认识，我们来把它使用起来，加深理解。假设我们在构建一个"Todo应用"，包含两个一面，一个待办列表页面，另一个是添加待办事项的页面。我们来一步步实现它。