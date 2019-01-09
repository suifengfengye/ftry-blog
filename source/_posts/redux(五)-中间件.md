---
title: redux(五)-中间件
tags: redux
date: 2019-01-08 18:01:33
---

redux的生态中，提供了中间件（即middleware）的方式来增强redux。middleware是指可以被嵌入在框架接收请求到产生响应过程之中的代码。middleware的方式能够很好的对代码进行解耦，如此middleware就能够独立成一个个的功能模块，方便维护和升级。那在redux中是如何提供对middleware的支持呢？redux提供了applyMiddleware()方法来支持middleware。applyMiddleware()方法的实现思路，就是将这些middleware形成一个链表，当dispatch()一个action的时候，就逐个调用调用链表里的middlware。它看起来就像下面这样。



实现的代码像下面这样：

{% codeblock %}
function applyMiddleware(store, middlewares) {
  middlewares = middlewares.slice()
  middlewares.reverse()

  let dispatch = store.dispatch
  middlewares.forEach(middleware =>
    dispatch = middleware(store)(dispatch)
  )

  return Object.assign({}, store, { dispatch })
}
{% endcodeblock %}

这段代码，首先将middlewares逆序排列，然后遍历这个逆序的middlewares，获取每个middleware的返回函数赋给dispatch，遍历完成后dispatch变量就是第一个middleware的返回函数，最后把store中的dispatch()方法替换成了第一个middleware的返回函数。所以如果我们添加了中间件，在执行store.dispatch()方法时，这个方法已经不是redux本身的dispatch()方法，而是第一个middleware的返回函数，redux本身的dispatch()方法会在所有middlewares的返回函数都执行后被调用。

同时，我们也注意到在遍历middlewares，我们对middleware进行了两次调用，分别传入了store和dispatch变量，即middleware(store)(dispatch)，store变量是一个固定的值，但是dispatch在每一次遍历后都重新指向了前一个middleware的返回值，在官方文档中这个dispatch变量命名为next，其实是更合理的。所以我们在编写middleware的时候，需要遵循如下的写法：

{% codeblock %}
const middleware = (store) => (next) => (action) => {
    // ... code
    const result = next(action)
    // ... code
    return result
}
{% endcodeblock %}

store的传入只是方便在middleware中获取应用的状态，而next的传入是为了形成一个链表，指向下一个middleware，所以在自己编写middleware的时候，需要在middleware内部手动调用next(action)以便能调用下一个middleware。

# 1、logger中间件

logger中间件的目的，是在dispatch执行前做一次记录，dispatch完成后，打印出dispatch之后的state。按照middleware的方式，我们来完成这个组件。

{% codeblock %}
// middlewares/logger.js
const logger = (store) => (next) => (action) => {
    console.log('dispatching', action)
    next(action)
    console.log('next state', store.getState())
}

export default logger
{% endcodeblock %}

logger中间件，在执行next之前打印出当前的action，然后执行next（即往后继续执行其他的middleware或者执行redux的dispatch()）,当dispatch完成后，打印出state信息。

# 2、thunk middleware

我们都知道，如果想要使用redux dispatch一个function类型的action，需要添加一个redux-thunk的middleware。那它是怎样实现的呢？我们来尝试实现它。

{% codeblock %}
// middlewares/thunk.js
const thunk = (store) => (next) => (action) => {
    if (typeof action === 'function') {
        action(store.dispatch, store.getState)
    } else {
        next(action)
    }
}

export default thunk
{% endcodeblock %}

上面这段代码，仍然采用了redux middleware的实现模式来包装（即(store) => (next) => (action)）。在实现的内部，我们判断传递进来的action，如果不是一个function，那么就调用下一个middleware；如果是function，我们就执行这个action，并且把redux的dispatch()和getState()方法参数传入到这个action方法，到此调用链结束。在acion方法的内部，如果使用了dispatch()方法重新发起action，则会沿着middlewares的调用链表重新执行。一般dispatch一个function action，会像下面这样：

{% codeblock %}
// 使用示意
export function dispatchInitTodos() {
  return (dispatch) => {
    getTodos().then((data) => {
      dispatch({
        type: 'INIT_TODOS',
        data,
      })
    })
  }
}
{% endcodeblock %}

# 3、promise middleware

dispatch一个Promise，和dispatch一个function会不会是一样的方式呢？我们先来写一个使用promise作为action的代码。

{% codeblock %}
function dispatchPromise(){
  return  new Promise((resolve, reject) => {
    // ...
  })
}
{% endcodeblock %}

如上述代码，我们dispatch一个Promise，这个Promise对象在创建出来的时候，就已经开始执行内部的逻辑了。这和function还是不一样的，function定义出来之后，可以延后执行，所以我们可以在middleware的实现中调用function。

所以在promise middleware的时候，思路是将Promise的resolved（或者rejected）的值作为一个action，重新使用store.dispatch()方法来发起这个action。

{% codeblock %}
// middlewares/promise.js
const promise = (store) => (next) => (action) => {
    // 抽取一个dispatch的方法
    // 支持派发多个actions
    const dispatchInnerAction = (data) => {
        if (Array.isArray(data)) {
            data.forEach(item => {
                store.dispatch(item)
            })
        } else {
            store.dispatch(data)
        }
    }
    if (typeof action.then === 'function') {
        action.then((data) => {
            dispatchInnerAction(data)
        }, (err) => {
            dispatchInnerAction(err)
        })
    } else {
        next(action)
    }
}

export default promise
{% endcodeblock %}

上述这段代码，当action.then为一个function的时候，我们就断定这个action为一个Promise对象。我们调用action.then()方法来获取这个Promises对象的resolved值（或者rejected值）。当然，这里resolved值（或者rejected值）要求必须为一个action。为了支持多个action，对resolved值（或者rejected值）进行来判断，如果是一个数组，逐个遍历并dispatch，否则直接dispatch。

Promise action的生成函数大体如下：

{% codeblock %}
// actions.js
import { getTodos } from './services.js'

// 初始化todos
export function initTodos(list) {
  return {
    type: 'INIT_TODOS',
    list,
  }
}

export function dispatchInitTodosPromise(){
  return  getTodos().then((data) => {
    return initTodos(data.list)
  })
}
{% endcodeblock %}

上述代码，getTodos()方法本身返回了一个Promise，我们调用getTodos().then()后传递给middleware的依然是一个Promise，并且使用 return initTodos(...)的形式resolved了一个action。在promise middleware中，这个resolved了的action会使用store.dispatch()方法发起。

＃ 4、结语

本文我们学习了redux middleware的实现机制，同时举了几个例子来编写自定义的middleware。如果不明白，可以到redux的官方文档中，查看 {% link middlewares https://redux.js.org/advanced/middleware %} 这篇教程，里面很详尽地道出了middleware的演进思路。不过这篇文章中举的两个例子，logger和crashReporter，虽然简单易懂，但是代表性不是特别强，做好自己动手实现一个简单的redux-thunk和redux-promise，对于redux middleware的理解将会更加深入。

本章的实现代码可以查看{% link redux-06 https://github.com/Two-Ftry/react-essay/tree/master/redux-06 %}。