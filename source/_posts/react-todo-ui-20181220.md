---
title: "待办事项"项目UI实现
date: 2018-12-20 13:32:47
tags:
---

在前端学习的过程当中，很多文档里面都会以"待办事项"作为例子展开说明。在学习redux的过程中，我们也使用"待办事项"作为例子。在介绍"redux搭配react"之前，我们先来实现"待办事项"需求的UI界面。待办事项的具体需求如下：

{% blockquote %}
1. 一个待办事项的列表(用todos数组存储)
   todos数组中的元素为待办事项内容，包含两个字段:
   {% codeblock %}
   {
       text: '待办事项1', // string
       completed: false,// boolean 是否完成的状态
   }
   {% endcodeblock %}
2. 一个过滤条件filter，filter是一个字符串，标明目前需要将哪些待办事项展现出来
   {% codeblock %}
    SHOW_ALL: 展示全部待办事项
    SHOW_COMPLETED: 展示已完成的待办事项
    SHOW_ACTIVE: 展示未完成的待办事项
   {% endcodeblock %}
3. 添加待办事项
{% endblockquote %}


原本想把本文的相关内容和"redux搭配react"融在一起。但是在梳理章节的时候，发现需要比较多的前置知识，融合在一起，"redux搭配react"文章就太长了，不方便阅读。所以我们把"'待办事项'项目UI实现"独立出来，方便后面的文章举例说明。

## 1、工程初始化


## 2、+ webpack

## 3、+ react

## 4、待办事项实现

### 4.1 添加待办事项UI

### 4.2 待办事项列表UI

### 4.3 待办事项列表过滤



