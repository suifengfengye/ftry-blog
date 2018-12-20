---
title: redux搭配react
date: 2018-12-16 12:34:36
tags:
---

Redux 支持 React、Vue、Angular、Ember、jQuery 甚至纯 JavaScript。Redux其实和React之间没有关系。但是在实际使用中，redux更多的时候是和react搭配使用。所以这片文章，我们先来让redux和react搭配起来使用。使用如下"待办事项"的例子来展开，具体需求如下：

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
{% endblockquote %}

在使用redux之前，我们先来搭建一个react的简单应用，然后再搭配使用redux。

## 1、react应用

{% codeblock %}
// index.js
import React from 'react'
import ReactDOM from 'react-dom'
import App from './app'

ReactDom.render(
    <App />,
    document.getElementById('root'),
)
{% endcodeblock %}

{% codeblock %}
// app.js
import React, { Component } from 'react'
import ReactDOM from 'react-dom'

class App extend Component {
    render () {
        return (
            <div>
                {'react app'}
            </div>
        )
    }
}

export default App

{% endcodeblock %}

