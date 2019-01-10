---
title: redux(四)-异步Action
date: 2018-12-30 01:48:02
tags: redux mockjs
---

使用js做前端开发，异步是我们经常会遇到的场景。最常见的，就是我们在调取后台数据的时候发送的异步请求。我们使用原生js来温习一下。

{% codeblock %}
const xhr = new XMLHttpRequest()

xhr.onreadystatechange = function () {
    if (xhr.readyState === 4) {
        if (xhr.status >= 200 && xhr.status <= 300 || xhr.status === 304) {
            console.log(xhr.responseText)
        } else {
            console.log(`Request was unsuccessful: ${xhr.responseText}`)
        }
    }
}

xhr.open('get', 'url', true)
xhr.send(null)

{% endcodeblock %}

在这一小段代码里面，我们先创建一个XMLHttpRequest对象，然后使用onreadystatechange属性的回调函数来监听状态变化。接着使用open()函数建立链接，最后使用send()函数发送数据到url对应的地址。这是一段异步代码，当send()函数执行之后，代码会继续往下执行，等到url对应地址的响应数据返回之后，才会执行onreadystatechange回调函数。

针对这个场景，有三个关键的时刻：发起请求、接收到响应（成功或者失败，也可能会超时）。在不考虑太多的情况下，我们只需要在接收到成功响应的情况下，把返回的数据通过dispatch()传递到redux的state中，这样就把数据保存了起来。我们把ajax请求单独写在services.js文件中，所以代码看起来会是这样的：

{% codeblock %}
// services.js
export const getTodos = () => {
    return new Promise((resolve, reject) => {
        const xhr = new XMLHttpRequest()
        xhr.onreadystatechange = function () {
            if (xhr.readyState === 4) {
                if (xhr.status >= 200 && xhr.status <= 300 || xhr.status === 304) {
                    console.log(xhr.responseText)
                    resolve(JSON.parse(xhr.responseText))
                } else {
                    console.log(`Request was unsuccessful: ${xhr.responseText}`)
                    reject(xhr.responseText)
                }
            }
        }
        xhr.open('get', '/getTodos', true)
        xhr.send(null)
    })
}
{% endcodeblock %}
再在请求回来后dispatch到redux中。
{% codeblock %}
// 使用示意
const { getTodos } from './services.js'
getTodos().then((data) => {
    dispatch(initTodos(data.list))
}, (err) => {
    console.log('@@@fail', err)
})
{% endcodeblock %}

我们的案例是以"待办事项"为例子，并没有后台接口与之对应，所以在继续往下学习之前，我们需要考虑如何通过接口来拿到数据。正常的项目开发中，会有后台的小伙伴给我们提供接口。现在我们没有后台的小伙伴那该怎么办？这个场景，就要考虑到mockjs了。

# 1、 mockjs

mockjs的{% link 官网 http://mockjs.com %}的介绍很简单——"生成随机数据，拦截Ajax请求"。但是这个中文介绍的前后顺序貌似有点问题，正常的顺序应该是"拦截Ajax请求，生产随机数据"。也就是说，使用mockjs可以拦截我们写的ajax请求，然后生成随机的数据返回给ajax。

在mockjs中，可以使用Mock.mock()这个方法来设置拦截哪个url，以及设置返回什么数据。

当然我们先要使用命令行来安装mockjs依赖包。
{% codeblock %}
> npm install -D mockjs
{% endcodeblock %}

接着添加如下代码：

{% codeblock %}
// mock.js
import Mock from 'mockjs'

Mock.mock('/getTodos', {
    'list|1-5': [{
        'text': 'todo-item',
        'completed|1': true,
    }]
})
{% endcodeblock %}

上述代码，Mock.mock()方法会拦截"/getTodos"请求，然后返回一个object对象，object对象包含一个list属性，list数组包含5个元素，每个元素的text属性为“todo-item”，completed属性为随机的boolean值。

{% codeblock %}
{
    list: [
        {
            text: 'todo-item',
            completed: true,
        },
        // ...
    ]
}
{% endcodeblock %}

最后别忘了在入口文件(index.js)中引入mock.js

{% codeblock %}
// src/index.js
// ...
import './mock'
// ...
{% endcodeblock %}

# 2、添加完整代码

我们要处理一个不一样的action，那么有两个步骤需要完成：

1. 添加action
2. 添加reducer

在actions.js文件中添加type为'INIT_TODOS'的action。

{% codeblock %}
// actions.js
// 初始化todos
export function initTodos (list) {
  return {
    type: 'INIT_TODOS',
    list,
  }
}
{% endcodeblock %}

然后在reducers.js文件中处理这个action，对应用状态的todos进行初始化。

{% codeblock %}
// 待办事项列表reducer
function todos(state = [], action) {
    switch (action.type) {
        // ...
        // 初始化 todos
        case 'INIT_TODOS':
            return [...state, ...action.list]
        break
        // ...
    }
}
{% endcodeblock %}

到此我们就处理了异步加载数据并将数据添加到redux的场景。这和普通的action没有任何区别。

# 3、加一个loading

假如要在请求发送到请求返回的这个时间段，添加一个loading效果。常规的做法，在执行getTodos()之前，先dispatch一个类型为'LOADING'的action到redux中（如设置loading为true），到请求返回后，处理将返回数据添加到redux，同时也需要将loading的标记设置为false。

{% codeblock %}
// 示意代码
dispatch({
    type: 'LOADING',
    loading: true,
})
getTodos().then((data) => {
    dispatch(initTodos(data.list))
    dispatch({
        type: 'LOADING',
        loading: false,
    })
}, (err) => {
    console.log('@@@fail', err)
    dispatch({
        type: 'LOADING',
        loading: false,
    })
})
{% endcodeblock %}

这是完全可行的。但是这段属于redux的代码写到业务组件里面，代码就耦合了。解耦的方式，可以这段代码都整合到actions.js中来进行。简单的，我们进行代码抽取就可以实现。我们抽取一个dispatchInitTodos()方法来实现。

{% codeblock %}
// actions.js
import { getTodos } from './services.js'
// ...
export function setLoading(loading) {
  return {
    type: 'SET_LOADING',
    loading,
  }
}

// 初始化todos
export function initTodos(list) {
  return {
    type: 'INIT_TODOS',
    list,
  }
}

export function dispatchInitTodos(dispatch) {
  dispatch(setLoading(true))
  getTodos().then((data) => {
    dispatch(initTodos(data.list))
    dispatch(setLoading(false))
  }, (err) => {
    console.log('@@@fail', err)
    dispatch(setLoading(false))
  })
}
{% endcodeblock %}

然后app.js中引入dispatchInitTodos()来执行。

{% codeblock %}
import { toggleTodo, initTodos, dispatchInitTodos } from './actions'

class App extends Component {
    // ...
    componentDidMount () {
        const { dispatch } = this.props
        dispatchInitTodos(dispatch)
    }
    // ...
}
{% endcodeblock %}

对此，我们还需要在reducers里面添加一个loading属性。

{% codeblock %}
// reducers.js
// ...
function loading (state=false, action) {
    switch(action.type) {
        case 'SET_LOADING':
            return !!action.loading
        break
        default:
            return state
    }
}

const rootReducer = combineReducers({
    visibilityFilter,
    todos,
    loading,
})
// ...
{% endcodeblock %}

上面只是一些代码的片段，并不完整。完整代码请查看 {% link redux-04 https://github.com/Two-Ftry/react-essay/tree/master/redux-04 %}。

针对异步的场景，我们上面做了简单的封装。实际上，redux针对异步的场景有很多中间件(即middleware)可以使用，不需要我们手动来封装。

# 4、redux-thunk中间件

redux的dispatch()方法自身的功能极其简单，它只能dispatch一个Object对象。但是redux提供的中间件方式，可以极大的扩展dispatch的功能。redux-chunk就是一个可以dispatch()一个function的中间件。redux提供了applyMiddleware(...middlewares)方法来添加中间件。

使用命令行，先添加redux-thunk依赖包。
{% codeblock %}
> npm install -D redux-thunk
{% endcodeblock %}

然后在createStore的时候将中间件redux-thunk加入到redux中。
{% codeblock %}
// index.js
// ...
import { createStore, applyMiddleware } from 'redux'
import reduxThunk from 'redux-thunk'
import rootReducer from './reducers.js'
const store = createStore(rootReducer, applyMiddleware(reduxThunk))

// ...
{% endcodeblock %}

使用redux-thunk可以dispatch一个function，这个function接收一个dispatch参数(由redux-thunk传入)。所以修改action如下。

{% codeblock %}
// actions.js
// ...
export function dispatchInitTodos() {
  return (dispatch) => {
    dispatch(setLoading(true))
    getTodos().then((data) => {
      dispatch(initTodos(data.list))
      dispatch(setLoading(false))
    }, (err) => {
      console.log('@@@fail', err)
      dispatch(setLoading(false))
    })
  }
}
{% endcodeblock %}

在app.js中使用。

{% codeblock %}
// app.js
// ...
import { toggleTodo, initTodos, dispatchInitTodos } from './actions'

class App extends Component {
    // ...
    componentDidMount () {
        const { dispatch } = this.props
        dispatch(dispatchInitTodos())
    }
    // ...
}
// ...
{% endcodeblock %}

如此，redux-thunk就使用起来了。

除了redux-thunk，还有其他的处理异步场景的中间件。

1. redux-promise:  dispatch Promise。
2. redux-saga：创建更加复杂的异步 action。
3. ...

这些中间件要不要学习？感觉一口气学完会蛮吃力的，是不是我会redux-thunk就可以了？现在搭建出来的项目一般不会单纯地使用redux-thunk这种简单的中间件，所以是要掌握的。不过redux-promise可以放一放，因为这个中间件是FSA兼容redux的一个中间件。redux-saga得花些时间去掌握。本章篇幅有限，先到此打住了。

本章完整的代码请查看 {% link redux-05 https://github.com/Two-Ftry/react-essay/tree/master/redux-05 %} 。