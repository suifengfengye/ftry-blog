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

但是单单了解这三个概念和它们的内在联系还是不足够的。我们还需要了解state的由来，以及如何获取state、更新state；state和store又是什么区别。

下面，我们一步一步来深入学习这三个概念。

{% blockquote %}
在文章中，我们会以"待办事项"为例子展开讲解redux。待办事项的需求如下：
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

## 1、 Action

Action 是把数据从应用传到store的有效载荷。Action本质上是一个普通的javascript对象。Action有一个约定，它必须包含 **"type"** 属性。

{% codeblock %}
{
    type: 'ADD_TODO',
    text: 'something',
}
{% endcodeblock %}

在实际应用中，我们会编写生成Action的方法，即 *Action* 创建函数，方便传递Action。

{% codeblock %}
// actions.js
// 添加待办事项
export function addTodo(text) {
  return { type: 'ADD_TODO', text }
}
// 对index对应待办事项的状态进行取反操作
export function toggleTodo(index) {
  return { type: 'TOGGLE_TODO', index }
}
// 更改过滤条件
export function setVisibilityFilter(filter) {
  return { type: 'SET_VISIBILITY_FILTER', filter }
}
{% endcodeblock %}

在actions.js中，定义了三个Action创建函数，addTodo将待办事项添加到应用状态中，toggleTodo函数会更改index对应的事项的状态（未完成／已完成），setVisibilityFilter设置过滤条件。

## 2、 Reducer

reducer会响应接收到的action，然后改变对应的应用状态。reducer实际上是一个函数，这个函数包含两个参数，第一个参数为state，第二个参数为action。

{% codeblock %}
// reducers.js
// 待办事项的过滤条件
function visibilityFilter(state = 'SHOW_ALL', action) {
  switch (action.type) {
    case 'SET_VISIBILITY_FILTER':
      return action.filter
    default:
      return state
  }
}
{% endcodeblock %}

在这里，参数state=[]的写法为es6语法，表明将state初始化为一个空数组。reducer的返回值组成了应用的状态（即"总state",下面写到"应用的状态"的时候指的就是"总state"）。每一次调用reducer，参数state(即"子state"，"子state"构成了"总state")的值为对应reducer对应状态的上一个值，该reducer的返回值又将更新到应用的状态中。如果是第一次调用reducer，参数state则为初始值。参数action则是上面介绍的一个带有type属性的普通javascript对象。

如果仅有一个reducer，那么这个reducer的返回值就是应用的状态。不过在一般的应用中，为了便于阅读和维护，都会有多个reducer。然后再通过redux提供的全局方法 combineReducers 将reducer组合起来，构成最终的应用状态。combineReducers函数接收一个Object对象参数，该对象的key值可以设置任意多个，key值对应的value值为一个reducer。

{% codeblock %}
// reducers.js
import { combineReducers } from 'redux'
// 待办事项的过滤条件
function visibilityFilter(state = 'SHOW_ALL', action) {
    switch (action.type) {
      case 'SET_VISIBILITY_FILTER':
        return action.filter
      default:
        return state
    }
  }
  // 待办事项列表reducer
  function todos(state = [], action) {
      switch (action.type) {
          // 添加待办事项
          case 'ADD_TODO':
              return [...state, {
                  text: action.text,
                  completed: false,
              }]
          break
          // 对index对应待办事项的状态进行取反操作
          case 'TOGGLE_TODO':
              const data = state.map((todo, index) => {
                  return {
                      ...todo,
                      completed: index === action.index ? !todo.completed : todo.completed
                  }
              })
              return data
          break
          default:
              return state
      }
  }
  
  const rootReducer = combineReducers({
      visibilityFilter,
      todos
  })
  
  export default rootReducer
{% endcodeblock %}

reducers.js中，最终导出的是一个合并的reducer(rootReducer)。rootReducer最终的结构(即应用状态的结构)如下：

{% codeblock %}
{
    visibilityFilter: 'SHOW_ALL',
    todos: [],
}
{% endcodeblock %}

应用状态由reducers的返回值构成，但是在代码中我们不会直观地看到上面这个结构。这个结构需要我们结合各个reducer，根据它的第一个参数state或者返回值分析出来。在我们学习更高级框架(如dva)的时候，会看到如果应用的状态能够直接配置，那么我们对应用状态的维护将会变得更加简便和直观。

reducer的触发时机，就是当使用redux提供的dispatch方法派发一个action时触发。但是dispatch方法不是redux的全局方法，它是store实例的方法，接下来我们就介绍store。

## 3、 Store

store是把action和reducer联系到一起的对象。这里我们要注意和state区别开来，state由reducer的返回值组成，是整个应用的状态。store则是由redux提供的createStore方法创建的一个实例，createStore的参数就是reducers。

{% codeblock %}
// index.js
import { createStore } from 'redux'
import rootReducer from './reducers.js'
const store = createStore(rootReducer)
{% endcodeblock %}

store创建之后，获取应用状态信息、更新应用状态等操作都是通过store实例的方法来实现。

1. 获取应用状态：提供 getState() 方法获取 state；
2. 更新应用状态：提供 dispatch(action) 方法更新 state；
3. 订阅：通过 subscribe(listener) 注册监听器。

以上，便介绍完了redux的几个重要概念。接下来，我们来验证一下。redux是一个状态管理库，它本身是不涉及UI，所以验证工作我们就通过console.log方法打印出state来验证。这可以通过subscribe方法来实现，当state方法变化时，subscribe方法的回调函数参数就会被调用。

在index.js中补充验证代码，如下：

{% codeblock %}
// index.js
import { createStore } from 'redux'
import { addTodo, toggleTodo, setVisibilityFilter } from './actions.js'
import rootReducer from './reducers.js'
const store = createStore(rootReducer)

// 验证
store.subscribe(() => {
    console.log(store.getState())
})
// 打印初始状态
console.log('初始状态', store.getState())

store.dispatch(addTodo('Learn Action'))
store.dispatch(addTodo('Learn reducer'))
store.dispatch(addTodo('Learn state'))
store.dispatch(addTodo('Learn store'))
store.dispatch(toggleTodo(0))
store.dispatch(toggleTodo(1))
store.dispatch(setVisibilityFilter('SHOW_COMPLETED'))

{% endcodeblock %}

{% asset_img ./result-20181213.png %}