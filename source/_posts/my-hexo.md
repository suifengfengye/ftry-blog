---
title: d3 树图、封闭图
---

# 1、树图

## 1.1、简介
能够展现不同数据元素之间的依赖。树图中，两个节点之间只有一条路径相互连接彼此。

## 1.2 构建树图

- 数据

树图的数据，仍然使用d3.hierarchy()或者d3.stratify()生成的数据。（见9-1）

- 布局

树图的布局，使用d3.tree()这个方法。d3.tree()是一个工厂方法，根据设置生成一个可以用于布局的方法（假设为 *_tree* ）。 

*_tree()* 方法接收一个参数，即d3.hierarchy()或者d3.stratify()生成的数据座位参数。方法执行后，就往参数对象中回填x/y坐标数据。

```
var _tree = d3.tree()
        .size([w, h - 80]);
// root为d3.hierarchy()或者d3.stratify()方法生成的树状结构数据
_tree(root);
```


- 画圆圈、文本

由于树状结构数据不是一个数组，而我们在画圆圈、或者现实文本的时候需要一个数组，所以需要将root转化为数组。使用 *root.descendants()* 方法转化。

画圆圈、添加文本按正常的d3添加circle、text元素即可。

```
const nodes = root.descendants();
```

- 画连线

画竖线使用 *d3.linkVertical()* 方法，画横线使用 *d3.linkHorizonal()* 方法。这两个方法的返回值，都有x、y两个属性方法，把上一步中的树状结构节点中生存的x、y值传入即可。（x、y值已经使用d3.tree()(root)计算出来）。

而连线的数据需要调用 *_tree(root).links()* 方法获取。

具体代码如下：

```
// 画线
const linkFn = isVertical ? d3.linkVertical()
        .x(function (d) { return d.x; })
        .y(function (d) { return d.y; })
        : d3.linkHorizontal()
            .x(function (d) { return d.y; })
            .y(function (d) { return d.x; })

_bodyG.selectAll('.link')
        .data(_tree(root).links())
        .enter()
        .append('path')
        .attr('class', 'link')
        .attr('d', linkFn);
```

2、封闭图

封闭图跟创建树图的步骤基本一致。下面只总结一下封闭图的不同知识点。

- 布局

布局使用 *d3.pack()* 方法，这是一个工厂方法。根据设置可以生成一个可以用于布局封闭图的方法（假设生成的方法为 *_pack* ）.

```
var _pack = d3.pack()
    .size([w, h])
    .padding(3);
```

调用 *_pack()* 方法，可以为树状结构数据计算出坐标值（包括x／y/r这些属性值）。

```
// root为树状结构数据，由d3.hierarchy()或者d3.stratify()方法生成。
_pack(root);
```

- 背景色

```
var _colors = d3.scaleSequential(d3.interpolateMagma)
    .domain([-4, 4]);
```

- 剪切

由于圆圈内的text文本可能超出范围，为避免这种情况出现，就可以使用svg的剪切功能。

```
circle-id
clipPath
    - use('xlink:href', '#circle-id')
    - id: pack-clip-id
text
    - attr('clip-path', 'url(#pack-clip-id)')
```

- hover

d3支持svg的事件。使用 *mouseover* 和 *mouseout* 来模拟hover。

```
packEnters.each(function (d) {
        d.node = this;
    }).on('mouseover', hovered(true))
        .on('mouseout', hovered(false));
    function hovered (hover) {
        return (d) => {
            d3.selectAll(d.ancestors().map(function (d) {
                return d.node;
            })).classed('pack-node-hover', hover);
        }
    };
```