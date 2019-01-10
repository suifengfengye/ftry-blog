---
title: redux(三)-redux搭配react
date: 2018-12-16 12:34:36
tags: redux react
---

Redux 支持 React、Vue、Angular、Ember、jQuery 甚至纯 JavaScript。Redux其实和React之间没有关系。但是在实际使用中，redux更多的时候是和react搭配使用。所以这片文章，我们先来让redux和react搭配起来使用。

本文在 {% link 待办事项项目UI实现 https://suifengfengye.github.io/2018/12/20/react-todo-ui/ %} 和 {% link 初识redux https://suifengfengye.github.io/2018/12/01/%E5%88%9D%E8%AF%86redux/ %} 这两篇文章的基础上来展开。

## 1、组合起来

在"初始redux"里面，我们完成了actions、reducers的编写，并且基于reducers创建了store。那么我们怎么样把redux和react搭配起来呢？

redux作为一个状态管理库，那么它提供出去的接口自然包括：

1. 获取应用状态信息，即state;
2. 更新应用状态：提供一个接口让外部能够触发action来更新应用状态。

而这两个接口对应的就是store实力的getState()和dispatch()方法。所以我们只要把store实例提供给react即可。简单地，我们自然可以在需要使用应用状态信息或者更新应用状态的react组件中引入store即可，但是这种方式有一个问题，就是使用起来太麻烦。而在react里面，有一种在组件之间共享类型的方式即"Context"，这种方式不必通过组件树的每个层级都通过props传递数据。基于这个原理，为了更好地把redux和react搭配起来，社区里面提供了"react-redux"这个工具把它们串联起来。

"react-redux"为应用状态能够在组件树中共享，提供了<Provider /\>这个高级组件。<Provider /\>接收redux的store作为属性来为react提供信息。在react的子组件中需要使用到应用状态的相关信息时，可以调用"react-redux"提供的connect方法。

{% codeblock %}
connect([mapStateToProps], [mapDispatchToProps], [mergeProps], [options])
mapStateToProps 
mapStateToProps是一个函数，它接收应用状态state作为第一个参数，
将传递到组件的props作为第二个参数，而它的返回值将会通过props的方式传递到组件中。
具体形如：
mapStateToProps(state, [ownProps]): stateProps
connect的四个参数都是可选参数，其他三个参数还没用到，暂时不做介绍。
{% endcodeblock %}

connect方法返回一个高阶组件。这个高阶组件接收组件作为参数返回一个包含redux相关信息的新组件。新的组件除了上面提到mapStateToProps的返回值作为props外，redux的dispatch()方法也会通过props传递进来。通过dispatch()方法我们就可以在组件中通过触发action来更改应用的状态了。

所以我们结合上两篇文章的代码，然后修改入口文件src/index.js,具体如下：

{% codeblock %}
// src/index.js
import React from 'react'
import ReactDOM from 'react-dom'

import { createStore } from 'redux'
import { Provider } from 'react-redux'
import rootReducer from './reducers.js'
const store = createStore(rootReducer)

import App from './app'

ReactDOM.render(
    <Provider store={store}>
        <App />
    </Provider>, 
    document.getElementsByTagName('BODY')[0],
)
{% endcodeblock %}

## 2、添加待办事项组合（AddTodo.js）

AddTodo.js文件的功能就是往状态管理库添加待办事项，但它并没有使用应用状态的信息。所以我们简单地使用"react-redux"提供的connect()方法来包装它，组件中就会有redux的dispatch()方法作为props传递进来。当点击添加待办事项的按钮时，我们就通过dispatch()方法触发"ADD_TODO"action即可。

{% codeblock %}
import React from 'react';
import { connect } from 'react-redux'
import { addTodo } from './actions'

const AddTodo = ({ dispatch }) => {
    let input = null
    return (
        <div>
            <input type="text"
                ref={node => {
                    input = node
                }} />
            <button type="button"
                onClick={() => {
                    dispatch(addTodo(input && input.value))
                }}
            >Add</button>
        </div>
    )
}

export default connect()(AddTodo)
{% endcodeblock %}

## 3、展示Todo列表信息

我们是在app.js中接收Todo列表的，所以使用connect将App组件和redux连接起来，然后通过connect的mapStateToProps参数来获取todos的信息。当接收到Todo列表项的点击事件的时候通过dispatch()方法更改Todo事项的状态。

{% codeblock %}
import React, { Component } from 'react';
import { connect } from 'react-redux'

import AddTodo from './AddTodo'
import TodoList from './TodoList'
import Footer from './Footer'
import { toggleTodo } from './actions'

class App extends Component {

    constructor (props) {
        super(props)
        this.onTodoClick = this.onTodoClick.bind(this)
    }

    onTodoClick (index) {
        const { dispatch } = this.props
        dispatch(toggleTodo(index))
    }

    render() {
        const { todos } = this.props
        return (
            <div>
                <AddTodo />
                <TodoList todos={todos} onTodoClick={this.onTodoClick}/>
                <Footer />
            </div>
        )
    }
}

const mapStateToProps = (state) => {
    return {
        todos: state.todos,
    }
}

export default connect(mapStateToProps)(App)
{% endcodeblock %}

## 4、过滤条件

在Footer.js文件中，我们有"ALL"、"Active"、"Completed"三个链接，它们对todos列表的过滤如下：

- ALL： 显示全部
- Active: 显示未完成的todo
- Completed：显示完成的todo

我们为三个链接添加点击事件，每当点击链接时，dispatch()触发action更改应用状态的"visibilityFilter"。

{% codeblock %}
// src/Footer.js
import React from 'react';
import { connect } from 'react-redux'

import Link from './Link'
import { setVisibilityFilter } from './actions'

const Footer = ({ dispatch, visibilityFilter }) => (
    <p>
        Show:
        <Link active={visibilityFilter === 'SHOW_ALL'}
            onClick={() => {
                dispatch(setVisibilityFilter('SHOW_ALL'))
            }}>ALL</Link>
        &nbsp;&nbsp;
        <Link active={visibilityFilter === 'SHOW_ACTIVE'}
            onClick={() => {
                dispatch(setVisibilityFilter('SHOW_ACTIVE'))
            }}>Active</Link>
        &nbsp;&nbsp;
        <Link active={visibilityFilter === 'SHOW_COMPLETED'}
            onClick={() => {
                dispatch(setVisibilityFilter('SHOW_COMPLETED'))
            }}>Completed</Link>
    </p>
)
const mapStateToProps = (state) => {
    return {
        visibilityFilter: state.visibilityFilter,
    }
}
export default connect(mapStateToProps)(Footer)
{% endcodeblock %}

visibilityFilter是用来对todos列表进行过滤的，所以在todos列表展现之前，我们要根据visibilityFilter这个条件对todos列表进行处理。我们修改app.js中的mapStateToProps方法即可。

{% codeblock %}
// src/app.js
// ...
const mapStateToProps = (state) => {
    return {
        todos: state.todos.filter((todo) => {
            if (state.visibilityFilter === 'SHOW_ACTIVE') {
                return !todo.completed
            } else if (state.visibilityFilter === 'SHOW_COMPLETED') {
                return todo.completed
            } else {
                // SHOW_ALL
                return true
            }
        }),
    }
}
// ...
{% endcodeblock %}

mapStateToProps里面，我们使用filter()函数对原始的todos列表进行过滤，如果visibilityFilter为'SHOW_ALL',我们都返回ture，那么返回的就是整个todos列表；如果visibilityFilter为'SHOW_ACTIVE'，我们就将completed属性为false的列表项返回true，filter()函数返回的结果就是所有"未完成"的待办事项；对于"SHOW_COMPLETED"也是同理。

# 5、结语

到此，redux就和react搭配了起来。基于这两个技术，我们就可以开发一些简单的react应用了。