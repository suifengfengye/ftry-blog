---
title: react-ace placeholder & 受控处理
date: 2019-03-29 23:42:03
tags: react ace placeholder 内容清空 光标 错位
---

我们在开发的过程中，可能会用到类似富文本编辑器这样的控件。在react体系中，react-ace是首选。这里我们来看看使用这个组件时，会遇到的问题。

## 1、placeholder

最新的官方文档里面其实是有 placeholder 这个属性配置，但是即使你安装了最新版本的react－ace，你会发现它并不生效。在这种情况下，我们能做的就是自己来实现placeholder的功能。

设置了placeholder属性后的效果如下（其实并没有生效）：

![no-placeholder](react-ace-placeholder-01.gif)

下面这段代码用来添加placeholder功能。

{% codeblock %}
import React, { Component } from 'react'
import AceEditor from 'react-ace'

let editorRef = null

class App extends Component {

    handleChange = () => {
        this.update()
    }

    update = () => {
        if (!editorRef) {
            return
        }
        const editor = editorRef.editor
        let shouldShow = !editor.session.getValue().length
        let node = editor.renderer.emptyMessageNode
        if (!shouldShow && node) {
            editor.renderer.scroller.removeChild(editor.renderer.emptyMessageNode)
            editor.renderer.emptyMessageNode = null
        } else if (shouldShow && !node) {
            node = document.createElement('div')
            editor.renderer.emptyMessageNode = node
            node.innerHTML = '我是placeholder~~~~'
            node.className = 'ace_invisible ace_emptyMessage'
            node.style.padding = '0 9px'
            node.style.position = 'absolute'
            node.style.zIndex = 5
            editor.renderer.scroller.appendChild(node)
        }
    }

    render() {
        return (
            <div>
                <AceEditor
                    onChange={this.handleChange}
                    ref={(r) => {
                        editorRef = r
                        setTimeout(() => {
                            this.update()
                        })
                    }}
                    placeholder="placeholder ace editor"
                />
            </div>
        )
    }
}

export default App
{% endcodeblock %}

实现效果如下：
![react-ace-placeholder](react-ace-02.gif)

demo地址大家可以查看 {% link 这里 https://github.com/Two-Ftry/react-essay/tree/master/react-ace-00 %} 。

我们来分析一下这段代码的实现。首先我们先获得了react-ace的引用（editorRef），然后调用update()方法。editorRef中，保存有ace-editor的实例（即editorRef.editor）。拿到了ace-editor实例，我们就要去看看{% link ace-editor的文档 https://ace.c9.io/#nav=api&api=editor %}，看看都有些什么API。这段代码里面，用到了session、renderer这两个实例。

- session：是EditorSession的实例，它存储了编辑器的所有状态数据；通过getValue()方法可以获得当前文档中输入的字符串。
- renderer：是VirtualRenderer的实例，负责将所有东西渲染到屏幕上。
  
而整体渲染在浏览器中的dom结构如下：

- ace_editor
    - textarea
    - gutter(ace_gutter)
    - ace_scroller
        - ace_content (输入内容的显示区)

我们要控制placeholder，那么可能得控制ace_content部分。而在renderer实例里面，有一个scroller的实例，存储的是ace_scroller对应的dom对象（它是ace_content的父元素），这就是我们要放置placeholder的dom了。

有了对这几个实例的了解。我们就不难分析这段代码了。首先，用getValue()方法，获得输入内容。如果内容为空，那么就需要显示placeholder。所以使用document.createElement('div')来创建了一个dom，并加上placeholder的内容以及style样式。然后把这个dom存放在renderer.emptyMessageNode属性中，并把dom通过appendChild()方法添加到scroller中（即ace_scroller对应的区域）渲染出来。此时，placeholder的效果就显示出来了。

而如果用户输入内容，我们使用onChange事件监听用户的输入，在onChange事件中调用update()方法。如果用户输入了内容，getValue()返回不为空，这时我们就从scroller对象中移除对应的dom，并将renderer.emptyMessageNode设置为null。这样，placeholder的效果就实现了。

PS：这段代码是基于stackoverflow上 {% link 这个回答 https://stackoverflow.com/questions/26695708/how-can-i-add-placeholder-text-when-the-editor-is-empty %} 的基础上实现的，大家也可去这里查看实现。


## 2、 AceEditor受控组件

在开发过程中，可能会遇到一个问题。那就是你将AceEditor做成受控组件，然后在输入英文的过程中，切换输入中文，发现前面的输入会清空。这个问题的原因是，AceEditor组件在输入的过程中被更新了，导致之前输入的内容都消失。这时，手动控制shouldComponentUpdate即可。

{% codeblock %}
 shouldComponentUpdate (nextProps) {
    const { content } = this.props
    if (content === nextProps.content) {
      return false
    }
    return true
  }
{% endcodeblock %}

PS:在开发中遇到这个问题，但是在写demo的时候，一直无法重现，先记录在此。

## 3、光标错位

当页面内容嵌套比较深的时候，极有可能出现光标错位的问题，起因就是上下文的字体样式影响到了ace-editor，导致它计算位置错误。这时我们只要重置ace-editor本身的样式即可。

问题如下：
![react-ace-cursor](react-ace-cursor.png)

修复方案：
{% codeblock %}
.ace_editor {
  * {
    font-family: inherit;
  }
}
{% endcodeblock %}