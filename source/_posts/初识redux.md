---
title: 初识redux
date: 2018-12-01 14:29:09
categories: 
- redux
tags: 
- redux
---

redux是一个状态管理库。核心概念，包括：action、reducer和store。

1. action： 是把数据从应用传到store的有效载荷。
2. reducer: 指定了应用状态的变化如何相应action，并发送到store。
3. store: 把action和reducer联系到一起的对象。

下面，我们一步一步来深入学习这三个概念。

## 1 Action

Action 是把数据从应用传到store的有效载荷。Action本质上是一个普通的javascript对象。Action有一个约定，这个javascript对象必须包含 **"type"** 属性。

{% codeblock %}
{
    type: 'ADD_TODO',
    data: 'something',
}
{% endcodeblock %}

在实际应用中，我们会编写生成Action的方法，即 *Action* 创建函数，方便将传递Action。

{% codeblock %}
// actions.js
export function addTodo(text) {
  return { type: 'ADD_TODO', text }
}

export function toggleTodo(index) {
  return { type: 'TOGGLE_TODO', index }
}
{% endcodeblock %}

在actions.js中，定义了两个Action创建函数，addTodo将待办事项添加到应用状态中，toggleTodo函数会将index对应的事项更改状态（未完成／已完成）。

## 2 Reducer

reducer会响应接收到的action，然后改变对应的应用状态。reducer实际上是一个function，这个function包含两个参数，第一个参数为state，第二个参数为action。

{% codeblock %}
// reducers.js
function todos(state = [], action) {
    switch (action.type) {
        case 'ADD_TODO':
            return [action.data]
        break
        // ...
        default:
            return state
    }
}
{% endcodeblock %}

在这里，参数state=[]的写法为es6语法，表明将state初始化为一个空数组。reducer的返回值组成了应用状态。每一次调用reducer，参数state的值为对应reducer对应状态的上一个值，该reducer的返回值又将更新到应用状态中。如果是第一次调用reducer，state则为初始值。参数action则是一个带有type属性的javascript对象。

reducer的触发时机，就是当使用redux提供的dispatch方法派发一个action时触发。但是dispatch方法不是redux的全局方法，它是store实例的方法，接下来我们就介绍store。

## 3 Store

store是把action和reducer联系到一起的对象。这里我们要注意和state区别，state由reducer的返回值组成，是整个应用的状态。store则是redux提供的一个实例，它由redux提供的createStore方法创建。

{% codeblock %}
// index.js
import { createStore } from 'redux'
import todoApp from './reducers'
const store = createStore(todoApp)
{% endcodeblock %}