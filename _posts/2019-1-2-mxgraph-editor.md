---
layout: post
title: mxgraph 架构分析与 mxgraph-editor 编辑器开发
description: mxGraph 框架本身是一个比较老旧的框架，陈旧的开发模式对开发者造成了较大的使用成本上的困扰。基于这个痛点，我们对 mxGraph 进行二次开发，开发出可以直接低成本使用的 mxgraph-editor 组件。我们在资源引入方式、自定义图形（svg、图片、卡片）、自定义侧边栏、自定义工具栏等方面进行大幅优化，并提供出常用的一系列事件回调，比如点击、双击、删除、回退、复制、重命名等事件，使得我们可以低成本的在当前各种主流前端框架中使用 mxGraph。
categories: 
- 前端开发
- mxgraph
- mxgraph-editor

tags: 
- 前端开发
- mxgraph
- mxgraph-editor
---


## 业务背景
业务流程图在线绘制需求在业务场景中广泛存在。针对业务场景的需求，我们进行了需求调研和优先级评定。结果如下：

流程图框架需求列表

|需求名称|优先级|
|---|---|
|批量圈选(图形&连线圈选)|P0|
|拉伸|P0|
|元素形状定制|P0|
|元素样式定制（图形、连线）|P0|
|批量拖拽|P0|
|母子节点拖拽|P0|
|批量圈选添加连线|P0|
|左侧可拖拽组件在画布中展现可所见不所得|P0|
|右键功能定制|P0|
|双击功能定制|P0|
|每一个vertex一个独立id|P0|
|某条线或节点或锚点高亮|P0|
|重命名|P0|
|自动布局|P1|
|滚动时始终保持有一部分在可视范围|P1|
|快捷键批量选择|P1|
|动画（连线数据流动画）|P1|
|双击编辑|P1|
|100%自适应|P1|
|文字缩小极值（保证文字可识别）|P1|
|锚点定制|P1|
|ctrl+z 回退|P1|
|连线自动对齐|P1|

目前市面上比较流行的流程图绘图框架大致有 jointjs、jsplumb、mxgraph、bpmnjs、antv g6 等。对以上框架进行技术调研，从商业版权、易用性、功能完善性、可定制性、浏览器兼容性、社区支持方面进行评估，大致得出如下结果：

||jointjs|	jsplumb|	mxgraph|	bpmnjs|	antv g6
|---|---|---|---|---|---|
|商用版权|	+|	+|	++|	-|	++|	
|易用性|	++|	+|	+|	+|	+|	
|功能完善|	+|	+|	+++|	++|	-|	
|可定制性|	+|	+|	+++|	++|	-|	
|浏览器兼容性|	+|	+|	+++|	+|	+|	
|社区支持|	+|	+|	++|	+|	-|	

可以看出 mxgraph 在商用版权、功能完善、可定制性上表现更为突出。

结合具体需求场景，我们选择更为资深、功能更为完善的 mxgraph 作为工业大脑业务场景中业务流程图的技术方案，并且二次开发出基于 mxgraph 的编辑器，暴露常用 API ，适用于各种前端技术场景。

下面我们对 mxgraph 框架进行架构分析，并对我们的二次开发编辑器组件 mxgraph-editor 进行介绍。


## mxGraph 简介

mxGraph是一个JavaScript图表库，可以快速创建交互式图形和图表应用程序，这些应用程序可以在其供应商支持的任何主要浏览器中运行。mxGraph 提供图形绘制、图形可视化、图形交互、图形布局、图形分析等能力， 适用于工作流程图、BPMN图、网络图、UML图、循环图、组织结构图、MindMap图、机架图、甘特图、信息图、平面图等绘制。

mxGraph在2005年创建，作为商业项目一直持续到2016年，2016年创建者以Apache 2.0开源协议在GitHub上公布源码。

官网：https://www.jgraph.com/


## mxGraph 架构分析

mxGraph中有三个主要的组件：mxGraph、mxGraphModel、mxCell。mxGraph是用户直接操作的图，图的所有状态都保存在mxGraphModel中，而图中的顶点和边都是用mxCell定义。三者层次关系如图所示：

![](https://gw.alicdn.com/tfs/TB1A7VtzSzqK1RjSZFHXXb3CpXa-1068-586.png_800x800.jpg)

当用户对mxGraph进行操作时，所有操作都映射到对mxGraphModel中保存的状态进行修改，而mxGraphModel中保存的状态也就是mxCell的状态。

围绕着这三个组件，mxGraph定义了很多属性，比如图的功能、mxGraphModel的持久化、mxCell的外观等等。此外mxGraph还具有非常强大的事务管理机制和事件监听器。

除此之外，mxGraph 还有 mxClient、组结构、复杂管理等部分组成，并提供 editor、io、handler、shape、view、util、layout 等 API。具体框架结构如下。
![](https://img.alicdn.com/tfs/TB1tjvxwCzqK1RjSZFHXXb3CpXa-1510-601.png)
其中 API 提供如下能力：
|包	|描述|
|---|---|
|editor	|提供了实现图编辑器所需的类，主要的类是mxEditor|
|handler, layout, shape|	分别包含事件监听器，布局算法和形状。图形事件监听器包括用于框线选择的mxRubberband，用于工具提示的mxTooltipHandler和用于基本单元修改的mxGraphHandler。mxCompactTreeLayout实现树形布局算法，shape包提供各种形状，这些形状是mxShape的子类。|
|view, model|	view和model实现了图形组件，由mxGraph表示。它指的是包含了mxCells并缓存mxGraphView中单元格的状态的mxGraphModel。根据mxStylesheet中定义的外观，使用mxCellRenderer绘制单元格。撤消历史记录在mxUndoManager中实现。要在图表上显示图标，可以使用mxCellOverlay。验证规则使用mxMultiplicity定义。|
|util|	提供了实用程序类，包括用于复制粘贴的mxClipboard，用于键和样式表值的mxConstants，用于跨浏览器事件处理和通用功能的mxEvent和mxUtils，用于国际化的mxResource和mxLog用于控制台输出。|
|io	|实现了一个通用的mxObjectCodec，用于将JavaScript对象转换为XML。主要类是mxCodec。mxCodecRegistry是自定义编解码器的全局注册表。|

下面是部分核心组件的介绍。

### 1. mxClient
mxClient.js是客户端的引导机制，此文件include了运行mxGraph所需的所有源文件，并加载了其依赖的资源文件，以及配置了客户端的语言。

mxClient.js的作用如下：

* 设置加载相关文件的全局变量
* 设置相关路径
* 设置客户端语言
* 加载css文件和js文件

### 2. mxGraph

mxGraph继承自mxEventSource以实现基于Web的图形组件的功能性方面。要激活平移和连接，使用setPanning和setConnectable，对于框线选择，必须创建一个新的mxRubberband实例。默认情况下，以下监听器添加到mouseListeners：

* tooltipHandler：显示工具提示的mxTooltipHandler
* panningHandler：用于平移和弹出菜单的mxPanningHandler
* connectionHandler：用于创建连接的mxConnectionHandler
* graphHandler：用于移动和克隆cell的mxGraphHandler
如果启用了这些监听器，则将按上述顺序调用它们。

mxGraph的功能依赖关系如图所示：

![](https://img-blog.csdn.net/20181013164207627?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1doaXRlX0lkaW90/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


### 3. mxGraphModel

mxGraphModel 是描述了图形结构的核心的模型，被称为mxGraphModel，可以在model包中发现。 另外，对图形结构的添加，更改和清除是通过图模型API来完成的。该模型还提供了方法来确定图形 的结构，以及提供方法来设置，如能见度、分组和样式的视觉状态。mxGraphModel是基本的对象，它存储着图形的数据结构。

mxGraph使用事务处理系统来更新模型。在HelloWorld的例子里，我们看到如下代码：

```
// Adds cells to the model in a single step
graph.getModel().beginUpdate();
try
{
   var v1 = graph.insertVertex(parent, null, 'Hello,', 20, 20, 80, 30);
   var v2 = graph.insertVertex(parent, null, 'World!', 200, 150, 80, 30);
   var e1 = graph.insertEdge(parent, null, '', v1, v2);
}
finally
{
   // Updates the display
   graph.getModel().endUpdate();
}
```

执行2个节点和1条连线的插入。对于模型的每一个变化，请调用beginUpdate()，作出适当的调用更改模型，然后调用endUpate()方法来完成的变化，通知发送变化的事件出去。

**关键API方法：**

- **mxGraphModel.beginUpdate()** - 启动一个事务或子事务处理。
- **mxGraphModel.endUpdate()** - 完成一个事务或子事务处理。
- **mxGraph.addVertex()** - 添加一个新节点到指定的父单元。
- **mxGraph.addEdge()** - 添加一个连线到指定的父单元。

#### 3.1 模型改变方法

以下是更改图形模型并应直接或间接放置更新范围的方法的列表：

```
add(parent, child, index)
remove(cell)
setCollapsed(cell, collapsed)
setGeometry(cell, geometry)
setRoot(root)
setStyle(cell, style)
setTerminal(cell, terminal, isSource)
setTerminals(edge,source,target)
setValue(cell, value)
setVisible(cell, visible)
```

最初，我们只关心添加和删除，以及几何和样式编辑方法。请注意，这些方法不是核心API方法，像往常一样，这些方法在mxGraph类中是适用的，他们执行的是被封装的更新。

**插入图元**

你可以使用add()方法添加节点和连线到模型中。但是，考虑到日常使用库的场景，请学习mxGrap.insertVertex()和mxGraph.insertEdge()这两个添加图元的核心公共API。

模型功能的要求，被添加的图元必须已经创建，而mxGraph.insertVertex()为你创建了图元。

**核心API方法：**

**mxGraph.insertVertex(parent, id, value, x, y, width, height, style)** –在调用开始/结束更新中，创建并插入一个新的节点到模型中。

**mxGraph.insertEdge(parent, id, value, source, target, style)** –在调用开始/结束更新中，创建并插入一条新的连线到模型中。

mxGraph.insertVertex() 会创建一个mxCell对象并返回。方法的参数为：

**parent** – 组结构中此图元的直接父图元。我们会很快谈论到组结构，但现在我们直接使用graph.getDefaultParent();作为默认的父图元，就像在 HelloWorld 这个例子一样。

**id** – 描述此单元的全局唯一身份号码，总是一个字符串。主要用于外部对这单元的引用。如果你不想自己维护这些号码，只需要传入一个空参数并确保mxGraphModel.isCreateIds()返回真即可。这样，模型就会管理这些号码，并保证它们的唯一性。

**value** – 此单元的用户对象。用户对象只是一些对象，可以让您把应用程序的商务逻辑与mxGraph的可视化呈现相关联。在手册的后面有详细地描述，这里我们就只用字符 串就好，并把它们显示成节点和边的标签。

**x, y, width, height** – 就像名字提到的，这是节点的左上角的 x 和 y 的位置以及它的宽度和高度。

**style** – 将被应用到节点的样式描述。关于样式，很快会有更详细的描述，简单来讲，就是一个特定格式的字符串。这个字符串有零个或多个样式名字和一些键/值配对，用来覆盖全局设置或者创立新的样式。除非我们要创建自己的样式，我们可以直接使用这些现有的设置。

添加连线的方法和添加节点的方法使用了同样的参数。source和target参数定义了节点要连接的节点。注意，源节点 和目标节点需要已经被加入到模型中。

### 4. mxCell

mxCell是节点和连线的图元对象。mxCell从模型那里复制了许多的方法。它们的主要差别在于，使用模型的方法会创建相关的事件通知以及撤销方法，使用图元的方法可以发生改变但不记录它们。这对于临时改变视觉效果，如动画或鼠标效果，是非常有用的。作为通用规则，一般使用模型的编辑API，除非您遇到什么特殊问题。

当创建新的图元时，构造函数需要三部分参数，值（用户数据），几何参数以及样式。我们在回到讨论图元之前，先了解一下这三个概念。

#### 4.1 样式

样式和样式表的概念挺像CSS的样式表，尽管你已经注意到mxGraph中事实上使用的就是CSS，但CSS只是影响HTML的DOM的全局样式。当你使用编辑器打开 util.mxConstants.js 文件并查询开头皮匹配“STYLE_”,如果你向下滚动鼠标，将会看到很多带有这个前缀的各种样式定义字符串。一些样式应用到节点上，一些适用于节点，一些适用于连线，一些都适用。正如你所见，这些视觉属性定义在它们操作的元素上。

mxStylesheet包含一个对象，样式，则以一个散列表映射样式名称到一个样式数组。

![image](https://gw.alicdn.com/tfs/TB1jJJzzSzqK1RjSZFjXXblCFXa-1084-652.png)

*样式集合中的样式列表*

上面蓝框里的图显示了在mxStyleSheet里面的样式表。字符串“defaultVertex”是真实样式串/值配对列表中的键值。请注意，mxGraph创建两组默认样式，一个给节点，一个给连线。如果你看回helloworld这个例子，作为可选参数，没有任何样式值传入insertVertex或insertEdge。这种情况下，默认的样式会被使用。

**设置图元的样式**

如果你希望给一个图元指定非默认的样式，你必须在创建时传入（mxGraph的insertVertex和insertEdge都有为这个用途的可选参数），或者对这个图元用model.setStyle()设定样式。

同样，mxGraph在核心API中提供了工具方法来访问和修改图元的样式：

**核心API方法：**


 **mxGraph.setCellStyle(style, cells)** – 封装在开始/结束的更新中，指定一组图元的样式。
 
 **mxGraph.getCellStyle(cell)** – 返回指定单元的样式，融合了这种单元类型，任何本地的和默认全局的样式。
 
**创建新的全局样式**

要创建上述ROUNDED全局样式，你可以按照这个模板来创建一个样式，并将其注册到mxStyleSheet上：

```
var style = new Object();
style[mxConstants.STYLE_SHAPE] = mxConstants.SHAPE_RECTANGLE;
style[mxConstants.STYLE_OPACITY] = 50;
style[mxConstants.STYLE_FONTCOLOR]= '#774400';
graph.getStylesheet().putCellStyle('ROUNDED',style);
```

#### 4.2 几何

在JavaScript的坐标系统中，x是向右为正，y是向下为正，而对图形而言，是相对mxGraph放置的容器的绝对位置。

有专门独立的mxGeometry而不是简单地用mxRectangle保存这个信息的原因，是因为连线也有相应的几何信息。

**相对位置**

默认情况下，一个节点的x和y位置是图元本身的边界矩形的左上角点与父图元的边界矩形的左上角点的偏移量。父图元和组的概念将在本章的后面进行讨论，这里就不深入细节，如果一个图元没有父图元， 那么为了定位，图形容器就是它的父图元。

![image](https://gw.alicdn.com/tfs/TB1ZrVDzFzqK1RjSZFoXXbfcXXa-996-926.png_500x500.jpg)

**偏移**

在mxGeometry中，偏移量字段是应用到图元标签的绝对x，y偏移量。在连线标签的情况下，该偏移量总是施加在边标签已根据上面部分相对标志计算好之后。

**核心API方法：**

- **mxGraph.resizeCell(cell, bounds)** - 在开始/结束的更新调用之间，改变指定图元的大小到指定的边框。
- **mxGraph.resizeCells(cells, bounds)** - 在开始/结束的更新调用之间，改变指定队列中所有图元的大小到指定的边框。

#### 4.3 用户对象

用户对象给了mxGraph一个环境，可以把业务逻辑与可视图元相关联。在HelloWorld的例子里，用户对象只是一个字符串，只是简单地用来呈现单元的标签。在更加复杂的应用里，用户对象可以是一个复杂的对象。这个对象的一些属性可以用来作为图元的显示使用，而对象的其余部分可以用来描述应用领域的逻辑关系。

用一个简单的工作流或过程应用程序来做例子，假定我们有以下的图形（[这个例子已在线](https://jgraph.github.io/mxgraph/javascript/examples/editors/workfloweditor.html)，在任务窗口选择泳道Swimlanes的例子）：

![image](https://jgraph.github.io/mxgraph/docs/images/mx_man_simple_workflow.png)

*一个简单工作流*

通常情况下，该工作流程将存在于某些应用服务器和/或数据库上。浏览器端用户连接到该服务器，或者一些前端服务器连接到应用程序服务器和用户的Web应用程序请求“订购”的工作流。网站服务器取得该工作流中的数据并将其发送到客户端。

mxGraph支持服务器端填充模型、发送到客户端，然后再返回的过程。请参阅后面的章节关于“I/O和服务器之间的通信”。

数据的发送在视觉模型（图中）和以及业务逻辑（主要是包含在用户对象）都会发生。客户端将首先显示如上图。如果用户有权限编辑工作流程，他们通常会做两件事情：1）编辑图表，添加和删除节点，以及改变的连接，2）编辑图元的用户对象（节点和/或连线）。

#### 4.4 图元(cell)类型

cell 分为两个类型：节点和连线。可通过 mxGraphModel 的 isVertex(cell) 和 isEdge(cell) 方法来检测当前 cell 是节点还是连线。

### 5. 组结构

在mxGraph中，分组的概念就是逻辑上将一个图元与另一个进行关联。在图形模型的数据结构中，分组就是把一个或更多的连线或节点变成一个父节点或连线的子图元。分组使得mxGraph拥有了更多的有用的特性：

- 子图 一个逻辑上独立的图形的概念，在较高的层次图中显示为每个子图的图元。
- 展开和折叠 折叠是这样的一种能力，它使用父图元在视觉上来替换一组子图元。展开则是与其相反的逻辑。通过单击在线[工作流程示例](http://jgraph.github.io/mxgraph/javascript/examples/editors/workfloweditor.html)中泳道示例的组单元格左上角的小“ - ”可以看到此行为。这会在下面的 _复杂性管理_ 章节中进行讲解。
- 分层 层次的概念是指在图形的显示中，让图元以特定的顺序显示。
- 下钻，递升 这两个概念是让子图可以当作完整的图形一样可视化和编辑。在 _用户对象_ 章节我们看到 “check inventory” （检查库存）节点作为一个简单图元。举个例子来说，一个开发人员正在描述流程中每一个节点作为软件流程中执行的任务。应用程序中可能会有一个选项去下钻到检查库存的节点。这将导致一个新的图形出现，详细地描述了检查库存的细节。图中可能会有一个标题为“检查库存”的节点用以表示它是一个子图，以及一个选项去递升回到上一个级别。


在分组中，图元都分配了一个父图元。最简单的情况是，所有的图元都有一个默认的父图元作为他们的父图元。这个父图元是一个不可见的图元，有着和整张图相同的界限。一个节点的x,y位置是相对它的父节点的，因此在默认分组的情况下（所有的图元共享默认父图元），图元的定位也是图形组件上的绝对坐标。分组概念用一个简单图示意如下。

![image](https://gw.alicdn.com/tfs/TB1DVRDzFzqK1RjSZFvXXcB7VXa-1304-728.png)

插入图元的到组结构是通过使用mxGraph类的insertVertex和insertEdge的parent参数方法。这些函数相应地将父图元格设置在子图元上，而且重点是，会通知父图元新子图元的加入。

通过mxGraph.groupCells（）和mxGraph.ungroupCells（）函数来更改组结构。

**核心API方法**

- mxGraph.groupCells(group, border, cells) – 添加指定的图元到指定的组，在一个begin/end更新结构中。
- mxGraph.ungroupCells(cells) – 从指定图元的父图元中移除它们，并将其加入到它们的父图元的父图元中。任何空的组在这个操作后将被删除。这个操作需要在一个begin/end更新结构中


### 6. 复杂度管理

任何时候控制显示图元的个数主要有两个原因。首先是性能，绘制越来越多的图元在任何平台上都将会在某个时间点达到性能的瓶颈。第二个原因是易用性，一个人只能理解一定量的信息。上面列出的与组相关的所有概念，能够用来为用户降低屏幕上信息的复杂度。

#### 6.1 可折叠

可折叠是我们用来描述组展开和折叠一个集合名词。我们可以将令一个图元内的子图元不可见称为折叠。有许多与此特性相关的方法：

**核心API方法**

- **mxGraph.foldCells(collapse, recurse, cells)** – 在一个begin/end更新结构中，指定图元的折叠状态。


**可折叠相关方法：**

- **mxGraph.isCellFoldable(cell, collapse)** – 对于有子图元的图元而言，默认为真。

- **mxGraph.isCellCollapsed(cell)** – 返回图元的折叠状态

当一个组图元折叠时，默认发生三件事：

- 图元的子图变为不可见
- 组图元的边界被使用。在mxgeometry中有一个alternativebounds字段，并且在组图元中，默认地分别存储了展开和折叠状态的边界。这些实例之间的切换由mxgraph.swapbounds（）调用，实际上这些是foldcells（）调用过程中为你处理的。这允许折叠组能够被改变大小，同时当再次展开看起来和折叠前的尺寸一样。
- 默认情况下会发生连线提升。连线提升意味着去展现这样的连线，它们连着子图元通过折叠组图元，组图元同样连着外面的折叠组图元，通过使得这些子图元消失而转去连接折叠的父图元。

![image](https://jgraph.github.io/mxgraph/docs/images/mx_man_expand_swim.png)

_展开泳道_

![image](https://jgraph.github.io/mxgraph/docs/images/mx_man_collapse_swim.png)

_折叠泳道_

以上两张图片展示了这三个概念。在扩展状态下，上面的组图元在左上角显示一个小框，里面有一个“ - ”字符。这表示单击此框会折叠组图元。点击后我们将获得下面的图片，图中组图元使用着它的折叠尺寸。没有离开子节点和边的图元都是不可见的。最后，离开组图元的连线被提升，表现为连接到折叠的组图元。点击框中现在出现的“+”字符展开组图元，并将其返回到上面的那个张图的原始状态。

通过调用mxGraph.foldCells()方法，你可以通过编程获得与单击展开/折叠符号相同的结果。一个常见的用法是这样的，当应用缩放到指定的数值，图元群被分组并且组图元折叠（往往不带“-”框，因为应用程序在控制折叠）。通过这种方式，用户可以看到更少，更大的图元，逻辑上每一个图元都代表其子图元。接着你可以提供一种机制去放大一个组，以在该过程中扩展它。你也可以提供下钻/递升，接下来就解释一下。

#### 6.2 子图, 下钻 / 递升

有时候，作为一种展开/折叠方案的另一个选择，或者可能与其结合使用，你的图会由一组图组成，嵌套到层次结构中，下面我们看一个简单的例子：

![image](https://jgraph.github.io/mxgraph/docs/images/mx_man_drill_down.png)

_一个顶级工作流的例子_


下钻后看到如下效果：

![image](https://jgraph.github.io/mxgraph/docs/images/mx_man_top_level.png)

退出组用 _shape->exit group_ 选项，它会调用mxGraph.exitGroup，带你回到原始的三个节点的最高一级的图。

**核心API方法：**

- **mxGraph.enterGroup(cell)** – 使指定的图元成为显示区域的根图元。
- **mxGraph.exitGroup()** - 使当前图元的父图元（如果有）成为新的根图元。
- **mxGraph.home()** - 离开所有组，使默认父图元成根图元

#### 6.3 分层和过滤

在mxGraph里, 像其它图形化应用一样，它也有z顺序的概念。也就是说，你看向屏幕的方向就是对象的顺序。对象可以位于其他对象的后面或前方，如果它们重叠且不透明，则最后面的对象将部分或完全遮盖。

![image](https://jgraph.github.io/mxgraph/docs/images/mx_man_overlap.png)

-重叠的节点-

如上图，可以看到World节点位于hello节点的前面。这是因为World节点比Hello节点有一个更高的子顺序，它们分别在根图元的有序子图元集合中的位置1和0位置上。

改变顺序我们可以用mxGraph.orderCells方法。

**核心API方法：**

- **mxGraph.orderCells(back, cells)** – 在开始/结束的更新调用之间，根据标志位，移动一组图元到它们的兄弟图元的前面或者后面。

mxGraph中的兄弟图元是指任何拥有相同父图元的图元。因此，对Hello节点执行此方法，会使其覆盖World节点。

排序和分组可以扩展成逻辑分层组。图元通过深度优先搜索被绘制。

## mxgraph-editor 编辑器组件

mxGraph 框架本身是一个比较老旧的框架，陈旧的开发模式对开发者造成了较大的使用成本上的困扰。基于这个痛点，我们对 mxGraph 进行二次开发，开发出可以直接低成本使用的 [mxgraph-editor](https://www.npmjs.com/package/mxgraph-editor) 组件。我们在资源引入方式、自定义图形（svg、图片、卡片）、自定义侧边栏、自定义工具栏等方面进行大幅优化，并提供出常用的一系列事件回调，比如点击、双击、删除、回退、复制、重命名等事件，使得我们可以低成本的在当前各种主流前端框架中使用 mxGraph。

下面是对 mxgraph-editor 组件的主要优化点和使用方法的介绍。

### 优化点
#### 资源引入方式
mxgraph 默认引入资源的方式是异步引入，你在 html 中通过 script 标签的方式引入 mxClient 资源，mxClient 会通过 script 标签的方式异步引入 200 多个资源。这 200 多个资源就是 200 多个额外的请求，对于页面加载性能来说，无疑是一个比较大的负担。

mxgraph-editor 放弃了这种笨拙的资源引入方式，改成了目前前端通用 commonjs 方式，实现了引入一次资源就可全量将 mxgraph 依赖的所有附加资源引入成功，避免了不必要资源请求对页面加载性能的影响。

#### 自定义图形
目前支持 svg、图片、卡片三种形式的图形，其中 svg 支持框架自带和自定义两种形式，卡片和图片支持自定义。

* 默认 svg 图形
默认 svg 图形支持 Rectangle、Rounded Rectangle、Text、Eclipse、Square、Circle、Process、Diamond、Parallelogram、Triangle 等图形。

![](![IMAGE](quiver-image-url/57AF4B2EBEE54CB19A7A2A090AF3DE18.jpg =634x255))

* 自定义 svg 图形
mxgraph 只能解析约定格式的 xml 结构来渲染 svg 图形，因此当想在 mxgraph 中自定义 svg 图形时，必须将 svg 图形转换成约定格式的 xml 。这里推荐一款不错的转换工 https://github.com/jgraph/svg2xml 。使用方法如下：
  * 编译
  ```
  mkdir classes
  javac -d classes -classpath lib/mxgraph-core.jar -sourcepath src src/com/mxgraph/svg2xml/Svg2XmlGui.java
  ```
  * 运行
  ```
  java -classpath lib/mxgraph-core.jar:classes com.mxgraph.svg2xml.Svg2XmlGui
  ```
  
![](https://gw.alicdn.com/tfs/TB1eXVXzMHqK1RjSZFkXXX.WFXa-1260-364.png)
  
* 自定义图片
mxgraph-editor 支持配置自定义图片的图形，可定义图形的名称、key、logo和宽高，格式如下：

```
[
  {
    name: '工业',
    key: 'gongye1',
    logo: 'https://img.alicdn.com/tfs/TB1NzGpt9rqK1RjSZK9XXXyypXa-80-80.svg',
    width: 80,
    height: 80
  },
  {
    name: '油桶',
    key: 'youtong',
    logo: 'https://img.alicdn.com/tfs/TB188qwt3HqK1RjSZFgXXa7JXXa-80-80.svg',
    width: 80,
    height: 80
  }
]
```

![](https://gw.alicdn.com/tfs/TB1mKZ5zpYqK1RjSZLeXXbXppXa-1130-340.png)

* 自定义卡片

mxgraph-editor 同时也提供一种卡片形式的图形，效果如下：

![](https://gw.alicdn.com/tfs/TB1SJI1zBLoK1RjSZFuXXXn0XXa-1108-306.png)

配置数据结构如下：

```
[{
  name: '主设备',
  key: 'zhushebei',
  logo: 'https://img.alicdn.com/tfs/TB1eD9LdgHqK1RjSZJnXXbNLpXa-144-128.png',
  width: 100,
  height: 80
}, {
  name: '附加设备',
  key: 'fujiashebei',
  logo: 'https://img.alicdn.com/tfs/TB1ejUeiAPoK1RjSZKbXXX1IXXa-36-32.png',
  width: 100,
  height: 80
}, {
  name: '产出物',
  key: 'chanchuwu',
  logo: 'https://img.alicdn.com/tfs/TB1ht.aisbpK1RjSZFyXXX_qFXa-32-32.png',
  width: 100,
  height: 80
}]
```

#### 自定义锚点
mxgraph-editor 图形在鼠标 hover 状态下，会显示可连线的锚点，该锚点图形支持自定义，自定义方法如下：
```
editor.initCustomPort('https://gw.alicdn.com/tfs/TB1PqwZzzDpK1RjSZFrXXa78VXa-200-200.png');
```
效果如下：
![](https://gw.alicdn.com/tfs/TB1AJNazQzoK1RjSZFlXXai4VXa-352-294.png)

#### command/ctrl+z 回退
mxgraph-editor 默认支持 command/ctrl+z 回退功能

#### command/ctrl+c/v 复制黏贴
mxgraph-editor 默认支持 command/ctrl+c/v 复制黏贴功能

#### 常用方法
* 功能
  * 节点自定义
    * svg
    * 图片
  * 锚点自定义
  * command/ctrl+z回退
  * command/ctrl+c 复制，command/ctrl+v 黏贴
  * delete 删除
  * 连线
  * 自动对齐
  * 自动保存
  * 重命名
  * 设置样式
  * 组合
  * 获取已选节点
  * 获取图表 xml
  * 根据 xml 渲染图表
  * 创建节点
  * 创建连线
  * 获取节点 id
  * 清除全局事件监听
* 事件
  * 单击
  * 双击
  * hover
  * 删除
  * 自动保存


### mxgraph-editor 的使用

#### 安装方法
```
npm install --save mxgraph-editor
```

#### 示例

```
import Sidebar from './sidebar';
import Toolbar from './toolbar';

import IMAGE_SHAPES from './shape-config/image-shape';
import CARD_SHAPES from './shape-config/card-shape';
import SVG_SHAPES from './shape-config/svg-shape.xml';

import Editor from 'mxgraph-editor';

const editor = new Editor({
    container: '.graph-content',
    clickFunc: this.clickFunc,
    doubleClickFunc: this.doubleClickFunc,
    autoSaveFunc: this.autoSaveFunc,
    cellCreatedFunc: this.cellCreatedFunc,
    deleteFunc: this.deleteFunc,
    valueChangeFunc: this.valueChangeFunc,
    IMAGE_SHAPES,
    CARD_SHAPES,
    SVG_SHAPES
  });

 render() {
    return (
      <div className="editor-container">
        <Layout>
          <Sider width="235" theme="light">
            <Sidebar key="sidebar" graph={this.editor} />
          </Sider>
          <Content>
            <div className="graph-inner-container">
              <Toolbar
                graph={this.editor}
                updateDiagramData={this.updateDiagramData}
              />
              <div className="graph-content" key="graphcontent" />
            </div>
          </Content>
        </Layout>
      </div>
    );
  }


```

#### API

|属性|	说明|	类型|	默认值|
|---|---|---|---|
|SVG_SHAPES|SVG形状配置|Array|[]|
|CARD_SHAPES|卡片形状配置|Array|[]|
|IMAGE_SHAPES|图片形状配置|Array|[]|
|container|画布容器选择器|selector|
|clickFunc|单击事件回调函数|function(cell)|
|doubleClickFunc|双击事件回调函数|function(cell)||
|autoSaveFunc|自动保存回调函数|function(xml)|
|cellCreatedFunc|节点创建回调函数|function(cell)||
|deleteFunc|节点删除回调函数|function(e)||
|valueChangeFunc|节点 value 变更回调函数|function(value)||
|initSidebar|初始化侧边栏|function(elements)||
|initCustomPort|自定义锚点, 10x10px|function((picture))||
|zoom|缩放|function(type)，入参：in（放大）、out（缩小）、actual（还原）||
|updateStyle|更新样式|function(cell, key, value)，入参：cell 节点，key 新样式的 key，value 新样式的 value||
|groupCells|组合|function(groupId, labelName)，入参：groupId 分组id，name 标签名称||
|ungroupCells|取消分组|function(cells)|
|getCellsSelected|获取当前选中的所有cell|function()||
|getGraphXml|获取当前 graph 的 xml|function()||
|createVertex|创建节点|function(shapeLabel, x, y, width, height, shapeStyle)|
|insertEdge|插入连线|function(vertex1, vertex2)||
|getCellById|通过id获取cell|function(id)||
|getAllCells|获取当前画布所有 cell|function()|
|refresh|重新渲染画布|function()||
|clearSelection|清除当前选择|function()||
|startPanning|开始平移画布|function()||
|stopPanning|停止平移画布|function()||



## 参考资料
[mxgraph官方文档](https://jgraph.github.io/mxgraph/)









