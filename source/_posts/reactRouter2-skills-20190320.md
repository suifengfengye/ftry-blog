---
title: reactRouter2-skills
date: 2019-03-20 17:18:16
tags: react router
---

react-router由路由组件、导航组件和路由匹配组件这三类组件构成。整体使用上不复杂。但是在使用上也有一些技巧。本文，我们就来看看在实际开发中会使用到的react-router的技巧吧。

## 1、处理当前选中菜单

效果如下：
<div style="border: 1px solid #ccc;">
![routerskill1](router-children.gif)
</div>

在上图的效果里面，我们看到当选中当前菜单后，对应的菜单不再可以点击。我们分两步来处理这个问题，首先我们要有路由，使用&lt;Link /&gt;组件就可以实现。

{% codeblock %}
<ul>
    <li>
        <Link to="/">列表页</Link>
    </li>
    <li>
        <Link to="/addTodo">添加页</Link>
    </li>
</ul>
{% endcodeblock %}

&lt;Link /&gt;组件默认会为我们生成一个a标签，点击这个标签就可以进行跳转。那我们如何让当前选中的菜单不可被点击呢？这就需要用到&lt;Route /&gt;组件的一个children属性。children是一个方法，Route会显示children方法中返回的内容；无论&lt;Route /&gt;组件的path是否匹配当前路由，都会显示出来。如果匹配，传入的match参数会包含路由信息，否则match参数为空。利用这个特性，我们就可以实现想要的效果啦。

{% codeblock %}
<ul>
    <li>
        <Route exact path="/">
            {
                ({ match }) => {
                    if (match) {
                        return (<span>列表页</span>)
                    }
                    return (<Link to="/">列表页</Link>)
                }
            }
        </Route>  
    </li>
    <li>
        <Route exact path="/addTodo">
            {
                ({match}) => {
                    if (match) {
                        return (<span>添加页</span>)
                    }
                    return (<Link to="/addTodo">添加页</Link>)
                }
            }
        </Route>   
    </li>
</ul>
{% endcodeblock %}

## 2、404

react-router中有一个&lt;Switch /&gt;组件，它的作用是只显示第一个匹配到的路由对应的组件。利用这个特性，可以做404页面的展示。

在&lt;Switch /&gt;的子组件中，把支持的路由写在前面，最后加一个没有path属性的路由，并将路由组件设置为404的提示，就能实现404的功能。

{% codeblock %}
<Switch>
    <Route exact path="/" render={ () => {
        return (<TodoList todos={data} />)
    }} />
    <Route path="/addTodo" component={AddTodo} />
    <Route component={Page404} />
</Switch>
{% endcodeblock %}

效果如下：

<div style="border: 1px solid #ccc;">
![routerskill2](router-404.gif)
</div>

## 3、demo地址

上面两个小技巧的demo地址在 {% link 这里 https://github.com/Two-Ftry/react-essay/tree/master/react-router-02 %} 。