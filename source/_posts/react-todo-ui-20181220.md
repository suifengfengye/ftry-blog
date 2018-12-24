---
title: 待办事项项目UI实现
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

既然是作为一个基础功能，我们就将项目名称命名为"todo-base"。在任何一个目录下创建"todo-base"目录，执行如下操作:

{% codeblock %}
> cd todo-base
# 初始化工程，按照步骤一步步填写
> npm init 
{% endcodeblock %}

npm init 执行完成了之后，目录下面就会有 package.json文件，该文件是什么？？
我们的项目中，会有源代码文件，也会有编译出来的代码文件供发布线上。在项目根目录下创建"src"和"dist"两个目录，分别放置源代码文件和编译出来的代码文件。

{% codeblock %}
- dist
- src
package.json
{% endcodeblock %}

接着，我们在src目录下创建我们的第一个js文件(index.js)，一个简单的npm项目架子就完成了。

{% codeblock %}
// index.js
document.write('Hello World!')
{% endcodeblock %}

### 1.1 让项目跑起来
既然说简单的项目已经成型，那怎样让"Hello World!"的字样能够在浏览器中显示呢？我们在dist目录下创建index.html文件，添加基础的html代码，然后引入"index.js"文件。

{% codeblock %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8" />
    <title>Document</title>
</head>
<body>
    <script src="../src/index.js"></script>
</body>
</html>
{% endcodeblock %}

我们用浏览器打开index.html，就可以看到"Hello World!"的字样啦。

早先的web前端开发，就是这样的方式来进行的。只不过后来逐渐有了自动化打包的工具，这些步骤都通过打包实现了。后面将会看到我们是如何使用webpack将这个过程自动化的。

## 2、+ webpack

上一节中，我们在index.html中通过script标签，引入src/index.js，引入的是源文件。源文件直接引入到html当中，有几个方面的缺点：

1. 需要自己通过script标签手动引入js文件，不太灵活;
2. 这样引入的js文件，不能使用一些新的ES6、ES7特性，因为有些浏览器并不支持这些特性；
3. script引入的js文件，还需要处理缓存的问题；
4. 想要对js文件进行混淆压缩也不方便。

一般来说，现在的前端开发都会使用像webpack、gulp等这些打包工具。本文我们使用webpack来处理js文件，并自动打包输出到dist目录中，并添加上混淆压缩的功能。

### 2.1 webpack简单配置

webpack中有两个基本的概念："输入"(entry)、"输出"(output)。webpack会根据入口的配置，逐步解析各个资源文件（包括js文件、css文件、图片等）形成一个依赖图。依赖图中的每个依赖项都会被处理，然后根据出口（output）的配置输出到对应的文件中。在我们的例子中，入口文件就是index.js，输出到"dist"目录下，所以webpack简单配置如下(在项目根目录下创建webpack.config.js文件)：

{% codeblock %}
// webpack.config.js
const path = require('path')
module.exports = {
    entry: {
        app: './src/index', // "app"key值可以随便定义，输出的时候就可以取这个key值为输出文件命名
    },
    output: {
        path: path.resolve(__dirname, './dist'),
        filename: '[name].js', // 这里[name]就对应entry中的‘app’key值
    }
}
{% endcodeblock %}

当然我们要为项目添加webpack的npm包才能使用webpack来对文件进行打包处理。在命令行中输入：

{% codeblock %}
// 确保当前处于项目根目录下再输入如下命令
> npm install -D webpack webpack-cli
{% endcodeblock %}

当依赖包安装完成之后，我们就可以使用webpack进行打包了，使用命令行工具，输入如下命令：

{% codeblock %}
// 确保当前处于项目根目录下再输入如下命令
> webpack
{% endcodeblock %}

待webpack之行完成，我们会在dist目录下看到"app.js"文件对输出。

到此我们来看看解决来本节开篇所列问题对几个？貌似一个都没有解决！我们只是使用webpack对入口文件进行了简单的打包输出。但是不要泄气，只要使用了webpack，我们再来添加一些配置，上面的问题就可以得到解决。

### 2.2 使用webpack来解决问题

1 避免手动引入js文件

在前面的例子中js文件是通过script标签引入到html中使用的。而上一小节，我们通过webpack已经能将js文件打包输出。如果html文件也能通过webpack输出，那么是不是打包出来的js文件就能自动的引入到html文件中呢（毕竟webpack的输出配置是我们手动配置的）？答案是可以的。

此时，我们需要了解webpack的另一个概念"plugin"。plugin是对webpack功能对增强，让webpack能够做更多对事情。要让webpack能够输出html文件，我们需要使用到"html-webpack-plugin"。

首先，我们来安装“html-webpack-plugin”的npm依赖包。

{% codeblock %}
> npm install -D html-webpack-plugin
{% endcodeblock %}

接着修改webpack.config.js的配置：

{% codeblock %}
// webpack.config.js
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')
module.exports = {
    entry: {
        app: './src/index', // "app"key值可以随便定义，输出的时候就可以取这个key值为输出文件命名
    },
    output: {
        path: path.resolve(__dirname, './dist'),
        filename: '[name].js', // 这里[name]就对应entry中的‘app’key值
    },
    plugins: [
        nwe HtmlWebpackPlugin()
    ]
}
{% endcodeblock %}

（2）

## 3、+ react

## 4、待办事项实现

### 4.1 添加待办事项UI

### 4.2 待办事项列表UI

### 4.3 待办事项列表过滤



