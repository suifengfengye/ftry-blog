---
title: reactRouter1-初识react router
date: 2019-02-25 17:43:09
tags: react-router
---

设想，我们正在使用react构建一个单页应用（SPA），我们应该怎么设置路由呢？在页面没有刷新的情况下，要更改页面的某个部分或者整个页面，这时候就是使用前端路由的时候了。配合react构建SPA的路由框架非react-router莫属。react-router更新到4.0版本，其设计思想已经和前几个版本不一样。其设计的思路是"动态路由"。何为"动态路由"?就是路由发生在app的渲染过程中！那与之相对应的就是"静态路由"，静态路由是在应用初始化前就已经定义好路由信息。所以，从定义上看是完全不一样的，那在使用的过程中也会有很大差别。

接下来我们看看react-router是如何工作的？

- (1) 把&lt;Router /&gt;组件放到应用的最顶层
- (2) 使用&lt;Link /&gt;组件定义路由
- (3) 最后，使用&lt;Route /&gt;组件展现路由对应的UI

如上，react-router会涉及到几个重要的组件，分别&lt;是Router /&gt;、&lt;Link /&gt;和&lt;Route /&gt;,翻译过来就是路由组件、导航组件和路由匹配组件。实际上，react-router也就是由这三类组件组成，所有这些组件都可以从"react-router-dom"这个npm包中导入使用。

我们来看看路由组件&lt;Router /&gt;,针对web应用，react-router提供了&lt;BrowserRouter /&gt;和&lt;HashRouter /&gt;。除了实现原理不一样外，使用上也有区别，&lt;BrowserRouter /&gt;支持服务端渲染而&lt;HashRouter /&gt;不支持，所以在使用的时候得注意这一点。

基于以上对react-router的简单认识，我们来把它使用起来，加深理解。假设我们在构建一个"Todo应用"，包含两个一面，一个待办列表页面，另一个是添加待办事项的页面。我们来一步步实现它。

# 1、搭建环境

要使用react-router，必须得把react的页面配置起来。我们简单地使用webpack来进行配置，并编写简单的页面，搭建起一个开发环境。目录信息如下：

- package.json
- webpack.config.js
- src
    - AddTodo.js (添加待办事项页面)
    - app.js (顶层App组件)
    - index.js (入口文件)
    - Todo.js (待办事项)
    - TodoList.js (待办事项列表页面)

具体实现参考 {% link 这里 https://github.com/Two-Ftry/react-essay/tree/master/react-router-00 %} 。

# 2、添加react-router

react-router 4.0+的三类组件（即路由组件、导航组件和路由匹配组件），都在"react-router-dom"这个npm包里面。我们先在项目根目录下安装依赖。

{% codeblock %}
> npm install -D react-router-dom
{% endcodeblock %}

依赖安装完成之后。我们来实现路由。按照上面提到的react-router工作的三个步骤，我们来一步步实现。

## 2.1 添加&lt;Router /&gt;组件

在app的顶层添加router组件，这里我们的index.js入口文件，就是整个web app的顶部，所以我们在index.js文件中添加&lt;Router /&gt;。

{% codeblock %}
import React from 'react'
import ReactDOM from 'react-dom'
import { HashRouter as Router } from 'react-router-dom'

import App from './app'

ReactDOM.render(
    <Router>
        <App />
    </Router>,
    document.getElementsByTagName('BODY')[0],
)
{% endcodeblock %}

这里我们使用HashRouter，因为是前端的简单实现，使用HashRouter和BrowserRouter都可。然后把&lt;Router /&gt;把  &lt; App /&gt;包裹起来，就完成任务了。

但是Router到底做了些什么？为什么要加Router在顶层呢？

## 2.2 使用&lt;Link /&gt;组件定义路由

todo应用的操作逻辑，进来之后首先展现列表页；从列表页，可以进入“添加页面”；而从“添加页面”又可以返回列表页，形成一个闭环。

列表页(TodoList.js)添加 &lt;Link /&gt; 的代码如下：
<table style="margin-left: auto; margin-right: auto;margin-top:-150px;">
        <tr>
            <td>
    {% codeblock %}
    // TodoList.js(添加Link前)
    import React from 'react'
    import Todo from './Todo'

    const TodoList = ({todos}) => (
        <div>
            <h1>待办列表</h1>
            <ul>
                {todos.map((todo, index) => (
                    <Todo key={index} {...todo} />
                ))}
            </ul>
        </div>
    )

    export default TodoList
    {% endcodeblock %}                
            </td>
            <td>
    {% codeblock %}
    // TodoList.js(添加Link后)
    import React from 'react'
    import { Link } from 'react-router-dom'
    import Todo from './Todo'
    const TodoList = ({todos}) => (
        <div>
            <h1>待办列表</h1>
            <Link to="addTodo">添加</Link>
            <ul>
                {todos.map((todo, index) => (
                    <Todo key={index} {...todo} />
                ))}
            </ul>
        </div>
    )
    export default TodoList
    {% endcodeblock %}
            </td>
        </tr>
    </table>
    
添加页(AddTodo.js)添加 &lt;Link /&gt; 的代码如下：

<table style="margin-left: auto; margin-right: auto;margin-top:-150px;">
        <tr>
            <td>
    {% codeblock %}
    // AddTodo.js(添加Link前)
    import React from 'react';

    const AddTodo = () => {
        return (
            <div>
                <h1>添加待办</h1>
                <br />
                <input type="text" />
                <button type="button">Add</button>
            </div>
        )
    }

    export default AddTodo
    {% endcodeblock %}                
            </td>
            <td>
    {% codeblock %}
    // AddTodo.js(添加Link后)
    import React from 'react';
    import { Link } from 'react-router-dom'
    const AddTodo = () => {
        return (
            <div>
                <h1>添加待办</h1>
                <Link to="/">返回</Link>
                <br />
                <input type="text" />
                <button type="button">Add</button>
            </div>
        )
    }
    export default AddTodo
    {% endcodeblock %}
            </td>
        </tr>
    </table>

## 2.3 使用&lt;Route /&gt;组件展现路由对应的UI

上一个小节，我们列表页的路由为"/",AddTodo页面的路由为"/addTodo"。所以我们在使用&lt;Route /&gt;就使用这两个路由定义。&lt;Route /&gt;有三种方式渲染路由组件，

1. component：component属性，直接配置需要展示的组件即可；
2. render: render是一个方法，可以添加函数定义；而要展示的组件，通过return返回即可展示；
3. children：children和render有一点类似，都是一个方法，不过children中的组件，无论路由是否匹配都会展示。

PS: 当看到children属性的时候，我们可能会想到在react的组件树，父组件可以通过children组件获取到子组件。没错，这里的children和react中定义的是一样的。

我们在app.js文件中定义&lt;Route /&gt;，代码如下(只展示render部分的代码)：
<table style="margin-left: auto; margin-right: auto;margin-top:-150px;">
        <tr>
            <td style="vertical-align: top;">
    {% codeblock %}
    // app.js(添加Route前)
    <div>
        <TodoList todos={data} />
        <AddTodo />
    </div>
    {% endcodeblock %}                
            </td>
            <td style="vertical-align: top;">
    {% codeblock %}
    // app.js(添加Route后)
    <div>
        <Route exact path="/" render={() => {
            return (<TodoList todos={data} />)
        }}/>
        <Route path="/addTodo" render={() => {
            return (<AddTodo />)
        }}/>
    </div>
    {% endcodeblock %}
            </td>
        </tr>
    </table>

# 3、结语

以上就是，一个简单的使用了react-router的react SPA应用。详细代码请 {% link 戳这里 https://github.com/Two-Ftry/react-essay/tree/master/react-router-01 %}