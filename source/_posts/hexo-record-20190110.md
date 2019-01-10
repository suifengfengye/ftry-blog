---
title: hexo-record
date: 2018-01-10 11:01:19
tags: hexo 
---

在使用hexo的过程中，会不断地遇到一些需要解决的问题或者添加一些新特性，发现这是一个持续渐进的过程。所以一一记录在这里，方便查阅。

# 1、图片资源

在官方文档中，要添加资源，给出的使用方式是,先修改_config.yml配置文件post_asset_folder属性，然后使用 asset_img 标签。

{% codeblock %}
_config.yml
post_asset_folder: true
{% endcodeblock %}

&#123;&#37; asset_img slug [title] &#37;&#125;

但是一直没有成功。解决方式是，添加一个hexo-asset-image插件，然后使用markdown的\!\[alt\]\(path\)语法来添加。同样，post_asset_folder属性也需要设置为true。

## 1.1 安装插件
在命令行工具中，进入到hexo项目的根目录，执行如下命令:
{% codeblock %}
> npm install -S hexo-asset-image
{% endcodeblock %}

## 1.2 使用注意
加入我们使用hexo new "blog-name"添加了一篇名字为"blog-name"的博客，那么也会生成一个blog-name命名的文件夹，在这个文件夹中添加资源文件即可。在使用\!\[alt\]\(path\)引用时，path直接写资源文件名称即可。
    假如有如下的目录结构：
{% codeblock %}
|- _posts
 |- blog-name
  |- hexo-asset.png
 |- blog-name.md
{% endcodeblock %}
那在blog-name.md中引用hexo-asset.png时，直接写名字即可。
{% codeblock %}
![alt](hexo-asset.png)
{% endcodeblock %}

# 2、流程图

使用hexo写博客的时候，会遇到需要添加流程图的地方，简单粗暴的方式就是使用其他的画图软件画好后截图然后通过图片的方式引入。不过，在hexo里面，我们也可以通过编码的方式实现流程图。这就需要用到"hexo-filter-mermaid-diagrams"插件。

## 2.1 安装插件
在命令行工具中，进入到hexo项目的根目录，执行如下命令:
{% codeblock %}
> npm install -S hexo-filter-mermaid-diagrams
{% endcodeblock %}

## 2.2 添加配置
在_config.yml文件中添加如下配置：
{% codeblock %}
# mermaid chart
mermaid: ## mermaid url https://github.com/knsv/mermaid
  enable: true  # default true
  version: "7.1.2" # default v7.1.2
  options:  # find more api options from https://github.com/knsv/mermaid/blob/master/src/mermaidAPI.js
    #startOnload: true  // default true
{% endcodeblock %}
这段配置就告诉hexo启用hexo-filter-mermaid-diagrams插件。但是这还不行，我们还需要在主题文件中添加加载css和js的代码。以在themes/landscape/layout/_partial/after-footer.ejs中添加为例。
{% codeblock %}
// themes/landscape/layout/_partial/after-footer.ejs
<% if (theme.mermaid.enable) { %>
    <script src='https://cdn.bootcss.com/mermaid/<%= theme.mermaid.version %>/mermaid.min.js'></script>
    <script>
      if (window.mermaid) {
        mermaid.initialize({theme: 'forest'});
      }
    </script>
  <% } %>
{% endcodeblock %}
    hexo模板引擎在解析这段代码的时候，会判断theme.mermaid.enable是否为true（即我们第一步的配置）。如果启用，会将<%= theme.mermaid.version %>替换为配置中的版本号。
    
## 2.3 使用流程图

\`\`\`mermaid
graph TD;
    A-->B;
    A-->C;
    B-->D;
    C-->D;
\`\`\` 

结果如下：
```mermaid
graph TD;
    A-->B;
    A-->C;
    B-->D;
    C-->D;
``` 
# 3、代码高亮

hexo本身是有代码高亮功能的，如果不喜欢，可以添加其他的高亮方式。添加其他的高亮方式。我们这里以添加highlight.min.js为例。
首先在_config.yml中将hexo本身的代码高亮关闭。

{% codeblock %}
// _config.yml
highlight:
  enable: false
{% endcodeblock %}

然后在主题中添加如下代码：

{% codeblock %}
// themes/landscape/layout/_partial/after-footer.ejs
<link rel="stylesheet" href="//cdn.bootcss.com/highlight.js/9.2.0/styles/github.min.css">
<script src="//cdn.bootcss.com/highlight.js/9.2.0/highlight.min.js"></script>
<script>
(function(){
  hljs.initHighlightingOnLoad();
}())
</script>
{% endcodeblock %}