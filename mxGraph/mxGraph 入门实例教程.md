在一篇文章 [《记一次绘图框架技术选型: jsPlumb VS mxGraph》](https://segmentfault.com/a/1190000018371243) 中，提到了我为什么要去学习 mxGraph。在入门时我遇到了以下几个问题

- 官方文档偏向理论，没能较好地结合代码进行讲解。
- 虽然官方给出的例子很多，但没有说明阅读顺序，对于那里刚入门的我也不知道应该从哪开始阅读。
- 通过搜索引擎搜索 “mxGraph教程” 也没有得到太大帮助。

通过自己对着官司文档死磕了一段时间并在公司项目中进行实践后，慢慢开始掌握这个框架的使用。下面我就根据我的学习经验写一篇比较适合入门的文章。

[官方](https://jgraph.github.io/mxgraph/)列了比较多文档，其中下面这几份是比较有用的

- [mxGraph Tutorial](https://jgraph.github.io/mxgraph/docs/tutorial.html)，这份文档主要讲述整个框架的组成，在搜索引擎搜索“mxGraph教程”，一般得出的结果是这份文档的中文翻译
- [mxGraph User Manual – JavaScript Client](https://jgraph.github.io/mxgraph/docs/manual.html)，这份文档对一些重要的概念进行讲解，以及介绍一些重要的 API 
- [在线实例](https://jgraph.github.io/mxgraph/javascript/index.html)，这些实例的源码都在[这里](https://github.com/jgraph/mxgraph/tree/master/javascript/examples)有
- [API 文档](https://jgraph.github.io/mxgraph/docs/js-api/files/index-txt.html)，这是最重要的一份文档，在接下来的教程我不会对接口作详细讲述，你可以在这里对相关接口作深入了解

在看完我的文章后希望系统地学习 mxGraph 还是要去阅读这些文档的，现在可以暂时不看。因为刚开始就堆这么多理论性的东西，对入门没有好处。

这篇教程分为两部分，第一部分主要结合[官网的例子](https://github.com/jgraph/mxgraph/tree/master/javascript/examples)讲解一些基础知识。第二部分则利用第一部分讲解的知识开发一个小项目 [pokemon-diagram](https://github.com/jinzhanye/pokemon-diagram)。本教程会使用到 ES6 语法，而第二部分的项目是用 Vue 写的。阅读本教程需要你会这两项预备知识。

## 引入
### 使用 script 引入
我们来分析一下官方的 [HelloWorld](https://github.com/jgraph/mxgraph/blob/master/javascript/examples/helloworld.html) 实例是怎样通过 script 标签引入 mxGraph 的

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Hello World</title>
</head>

<body>
<div id="graphContainer"></div>
</body>

<script>
mxBasePath = '../src';
</script>

<script src="../src/js/mxClient.js"></script>
<script>
	// ......
</script>
</html>
```

首先要声名一个全局变量 `mxBasePath` 指向一个路径，然后引入 mxGraph。

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g108oivvzmj305s03e3yg.jpg)

`mxBasePath` 指向的路径作为 mxGraph 的静态资源路径。上图是 HelloWorld 项目的 `mxBasePah`，这些资源除了 js 目录 ，其他目录下的资源都是 mxGraph 运行过程中所需要的，所以要在引入 mxGraph 前先设置 `mxBasePath`。 

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g108qwr0ylj306i0dhaaj.jpg)

再来看看 javascript 目录下有两个 `mxClient.js` 版本。 一个在 `javascript/src/js/mxClient.js` ，另一个在 `javascript/mxClient.js`，后者是前者后的版本。
所以两者可以替换使用的。如果你的项目是使用 script 标签引入 mxGraph，可以参考[我这个库](https://github.com/jinzhanye/learn-mxgraph/blob/master/demo/01.helloworld.html)

### 模块化引入
模块化引入可以直接参考我的项目的这个文件 [static/mxgraph/index.js](https://github.com/jinzhanye/pokemon-diagram/blob/master/src/graph/index.js)

```js
/*** 引入 mxgraph ***/
// src/graph/index.js
import mx from 'mxgraph';

const mxgraph = mx({
  mxBasePath: '/static/mxgraph',
});

//fix BUG https://github.com/jgraph/mxgraph/issues/49
window['mxGraph'] = mxgraph.mxGraph;
window['mxGraphModel'] = mxgraph.mxGraphModel;
window['mxEditor'] = mxgraph.mxEditor;
window['mxGeometry'] = mxgraph.mxGeometry;
window['mxDefaultKeyHandler'] = mxgraph.mxDefaultKeyHandler;
window['mxDefaultPopupMenu'] = mxgraph.mxDefaultPopupMenu;
window['mxStylesheet'] = mxgraph.mxStylesheet;
window['mxDefaultToolbar'] = mxgraph.mxDefaultToolbar;

export default mxgraph;


/*** 在其他模块中使用 ***/
// src/graph/Graph.js
import mxgraph from './index';

const {
  mxGraph,
  mxVertexHandler,
  mxConstants,
  mxCellState,
  /*......*/
} = mxgraph;
```

这里有两点需要特别注意的

- `mx` 方法传入的配置项 `mxBasePath` 指向的路径一定要是一个可以通过url访问的静态资源目录。举个例子，
我项目的 static 目录是个静态资源目录，该目录下有 `mxgraph/css/common.css` 这么个资源，通过`http://localhost:7777` 可以访问我的应用，那么通过 `http://localhost:7777/static/mxgraph/css/common.css` 也应该是可以访问 `common.css` 才对

- 如果你是通过 script 标签引入 mxGraph，是不需要绑定全局变量那段代码的。模块化引入要使用这段代码是因为，mxGraph 这个框架有些代码是通过 window.mxXXX 对以上的属性进行访问的，如果不做全局绑定使用起来会有点问题。
这是官方一个未修复的 BUG，详情可以查阅上面代码注释的 issue

## 基础知识
这部分会使用到[官网的例子](https://github.com/jgraph/mxgraph/tree/master/javascript/examples)，及我自己编写的[一些例子](https://github.com/jinzhanye/learn-mxgraph)。大家可以先把代码下载下来，这些例子都是不需要使用 node 运行的，直接双击打开文件在浏览器运行即可。

### Cell
`Cell` 在 mxGraph 中可以代表`组(Group)`、`节点(Vertex)`、`边(Edge)`，[mxCell](https://jgraph.github.io/mxgraph/docs/js-api/files/model/mxCell-js.html#mxCell.mxCell) 这个类封装了 `Cell` 的操作，本教程不涉及到`组`的内容。下文若出现 `Cell` 字眼可以当作 `节点` 或 `边`。

### 事务
官方的 [HelloWorld](https://github.com/jgraph/mxgraph/blob/master/javascript/examples/helloworld.html) 的例子向我们展示了如何将节点插入到画布。比较引人注意的是 `beginUpdate` 与 `endUpdate` 这两个方法，这两个方法在官方例子中出镜频率非常高，
我们来了解一下他们是干嘛用的，嗯，真是只是了解一下就可以了，因为官方对两个方法的描述对入门者来说真的是比较晦涩难懂，而且我在实际开发中基本用不上这两个方法。
可以等掌握这个框架基本使用后再回过头来研究。下面的描述来源这个[文档](https://jgraph.github.io/mxgraph/docs/tutorial.html)，我来简单概括一下有关这两个方法的相关信息。

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g0vjeq1nasj30pe0isn1k.jpg)

- `beginUpdate、endUpdate` 用于创建一个事务，一次 `beginUpdate` 必须对应一次 `endUpdate`
- 为了保证，假如 beginUpdate 执行失败，endUpdate 永远不会被调用，`beginUpdate 一定要放到 try 块之外`
- 为了保证，假如 try 块内更新失败，endUpdate 也一定被调用，`beginUpdate一定要放到 finally 块`
- 使用 beginUpdate 与 endUpdate 可提高更新视图性能，框架内部做撤消/重做管理也需要 beginUpdate、endUpdate

你可以试着把这两个方法从代码中删掉，程序还是可以正常运行。

### insertVertex

```
mxGraph.prototype.insertVertex = function(parent, id, value,
                                          x, y, width, height, style, relative) {

  // 设置 Cell 尺寸及位置信息
  var geometry = new mxGeometry(x, y, width, height);
  geometry.relative = (relative != null) ? relative : false;

  // 创建一个 Cell
  var vertex = new mxCell(value, geometry, style);
  // ...
  // 标识这个 Cell 是一个节点
  vertex.setVertex(true);
  // ...

  // 在画布上添加这个 Cell
  return this.addCell(vertex, parent);
};
```

上面是经简化后的 `insertVertex ` 方法。 `insertVertex ` 做了三件事，先是设置几何信息，然后创建一个节点，最后将这个节点添加到画布。`insertEdge` 与 `insertVertex` 类似，中间过程会调用 `vertex.setEdge(true)` 将 `Cell` 标记为边。从这里我们也可以得知无论`节点`还是`边`在 mxGraph 中都是由 [mxCell](https://jgraph.github.io/mxgraph/docs/js-api/files/model/mxCell-js.html#mxCell.mxCell) 类表示，只是在该类内部标识这个 `Cell` 是 `节点` 还是 `边`。


### mxGeometry

```
function mxGeometry(x,y,width,height){}
```
`mxGeometry` 类表示 `Cell` 的几何信息，宽高比较好理解，只对节点有意义，对边没意义。下面通过 [08.geometry.html](https://github.com/jinzhanye/learn-mxgraph/blob/master/demo/08.geometry.html) 这个例子说明如`x、y`的作用。

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g10aagk2tmj30g107n3yl.jpg)

`mxGeometry ` 还有一个很重要的布尔属性 `relative`，

- relative 为 false 的节点，表示以画布左上角为基点进行定位，`x、y` 使用的是绝对单位

	上一小节提到 `insertVertex` 内部会创建 `mxGeometry` 类。使用 `mxGraph.insertVertex` 会创建一个 `mxGeometry.relative` 为 false 的节点，如 A 节点
	
	![](https://jgraph.github.io/mxgraph/docs/images/mx_man_non_relative_pos.png)	

- relative 为 true 的节点，表示以父节点左上角为基点进行定位，`x、y` 使用的是相对单位

	使用 `mxGraph.insertVertex` 会创建一个 relative 为 false 的节点。如果你要将一个节点添加到另一个节点中需要在该方法调用的第9个参数传入 true，将 relative 设置为 true。这时子节点使用相对坐标系，以父节点左上角作为基点，x、y 取值范围都是 \[-1,1]。如 C节点 相对 B节点定位。

	![](https://jgraph.github.io/mxgraph/docs/images/mx_man_rel_vert_pos.png)

- relative为 true 的边，`x、y` 用于定位 label

	使用 `mxGraph.insertEdge` 会创建一条 relative 为 true 的边。x、y 用于定位线条上的 label，
x 取值范围是 [-1,1]，-1 为起点，0 为中点，1 为终点。y 表示线条的正交线距离。第三个例子能帮忙大家理解这种情况。

	![](https://ws1.sinaimg.cn/large/006tKfTcgy1g0z65fc7a7j308v063dft.jpg)


	```
	const e1 = graph.insertEdge(parent, null, '30%', v1, v2);
	e1.geometry.x = -0.5; // [-1,1] 调整 label 沿连接线的位置
	e1.geometry.y = 100; // 调整label 在正交线上的距离
	```

### 设置样式
![](https://jgraph.github.io/mxgraph/docs/images/mx_man_styles.png)

查看 xx 例子，我们知道 mxGraph 提供两种设置样式的方式。

第一种是设置全局样式。[mxStylesheet](https://jgraph.github.io/mxgraph/docs/js-api/files/view/mxStylesheet-js.html#mxStylesheet.mxStylesheet) 类用于管理图形样式，通过 [graph.getStylesheet()](https://jgraph.github.io/mxgraph/docs/js-api/files/view/mxGraph-js.html#mxGraph.getStylesheet) 可以获取当前图形的 `mxStylesheet` 对象。`mxStylesheet` 对象的 `styles` 属性也是一个对象，该对象默认情况下包含两个对象`defaultVertexStyle、defaultEdgeStyle`，修改这两个对象里的样式属性对所有线条/节点都生效。

第二种是对样式进行命名，然后使用 [mxStylesheet.putCellStyle](https://jgraph.github.io/mxgraph/docs/js-api/files/view/mxStylesheet-js.html#mxStylesheet.putCellStyle) 方法为 `mxStylesheet.styles` 添加样式对象。在添加 Cell 的时候，写在参数中。形式如下

```
[stylename;|key=value;]
```

例子中设置折线有一个需要注意的地方

```
// 设置拖拽线的过程出现折线，默认为直线
    this.connectionHandler.createEdgeState = () => {
      const edge = this.createEdge();
      return new mxCellState(this.view, edge, this.getCellStyle(edge));
    };
```


虽然调用 `insertEdge` 方法时已经设置了线条为折线，但是在拖拽线条过程中依然是直线。上面这段代码重写了 `createEdgeState` 方法，将拖动中的线条样式设置成与静态时的线条样式一致。

mxGraph 所有样式在[这里](https://jgraph.github.io/mxgraph/docs/js-api/files/util/mxConstants-js.html#mxConstants.STYLE_STROKECOLOR)可以查看，打开网站后可以看到以 `STYLE_` 开头的是样式常量。但是这些样式常量不能很好展示样式效果。下面教大家一个设置的小技巧，使用 [draw.io](https://www.draw.io/) 或 [GraphEditor](https://jgraph.github.io/mxgraph/javascript/examples/grapheditor/www/index.html) (这两个应该都是使用 mxGraph 进行开发的) 的 `Edit Style` 功能可以查看当前 Cell 样式。

比如现在我想将边的样式设置成：折线、虚线、绿色、拐弯为圆角、粗3pt。在 Style 面板手动修改样式后，再点击 `Edit Style` 就可以看到对应的样式代码。
![](https://ws4.sinaimg.cn/large/006tKfTcgy1g0wstgt4a0j30ik0df3z7.jpg)

### 面向对象编程
mxGraph 框架的特点是使用面向对象的方式进行编程，所以在接下来的例子你会看到大量这种形式的方法重写(Overwrite)。

```
const oldBar =  mxFoo.prototype.bar;
mxFoo.prototype.bar = function (...args)=> {
   // .....
	oldBar.apply(this,args);
	// .....
};
```

还有就是该框架所有类带 mx 前缀。

### 节点组合

我说一下 constituent 这个例子，值得注意的几个地方

组合节点后默认情况下，父节点是可折叠的，要关闭折叠设置 `foldingEnabled` 为 `false` 即可

```js
graph.foldingEnabled = false;
```

如果希望在改变节点尺寸时，子节点与父节点等比例缩放，需要开启 `recursiveResize`

```js
graph.recursiveResize = true;
```

下面是这个例子最重要的两段代码

```js
/**
 * Redirects start drag to parent.
 */
var graphHandlerGetInitialCellForEvent = mxGraphHandler.prototype.getInitialCellForEvent;
mxGraphHandler.prototype.getInitialCellForEvent = function(me)
{
    var cell = graphHandlerGetInitialCellForEvent.apply(this, arguments);

    if (this.graph.isPart(cell))
    {
        cell = this.graph.getModel().getParent(cell)
    }

    return cell;
};

// Redirects selection to parent
graph.selectCellForEvent = function(cell)
{
    if (this.isPart(cell))
    {
        /*改变参数 arguments 会发生变化*/
        cell = this.model.getParent(cell);
    }
    mxGraph.prototype.selectCellForEvent.apply(this, arguments);
};
```

这两个方法重写(Overwrite)了原方法，思路都是判断如果该节点是子节点则替换成父节点去处理剩下的逻辑。

第一个方法 `getInitialCellForEvent` 在鼠标按下(mousedown事件，不是click事件)时触发，
如果注释掉这段代码，不使用父节点替换，当发生拖拽时子节点会被单独拖拽，不会与父节点联动。
使用父节点替换后，原本子节点应该被拖拽，现在变成了父节点被拖拽，会发生联动效果。

通过 debugger 进去 `getInitialCellForEvent` 可以得知，第二个方法 `selectCellForEvent` 其实是 `getInitialCellForEvent` 内部调用的一个方法。
这个方法的作用是将 cell 设置为 `selectCell`，设置后通过 `graph.getSelectoinCell` 可获取得该节点，与 `getInitialCellForEvent` 同理，如果不使用父节点替换，
则 `graph.getSelectoinCell` 获取到的会是子节点。

### anchor

注意以 `entry` 或 `exit` 开头的样式代表的是靶点，请参考 anchor 例子。

// TODO 修改官方 anchor.html demo 加图片说明
A->B
C->D
E->F

可以看到如果不设置靶点，当节点被拖拽后 mxGraph 会智能地更换线条的出入的方向，而设置靶点样式后线条出入口就固定了。

默认情况下，可以从图形中心拖出线条，这时线条的 `exit` 为空，mxGraph 会智能地调整线条出口方向。而从图形的靶点拖出线条，则 `exit` 为固定值，连接线条后，拖拽图形，mxGraph 不会调整线条出口方向。
入口情况同理。

## 项目实战
todo 外元素拖拽直接在代码中链接到自己写的简化版 example

### 做一个节点组合
下面我以项目这个节点为例，讲解一下如何组合节点

```js
    const nodeRootVertex = new mxCell('鼠标双击输入', new mxGeometry(0, 0, 100, 135), `node;image=${src}`);
    nodeRootVertex.vertex = true;
    
    const title = source.getAttribute('alt');
    const titleVertex = graph.insertVertex(nodeRootVertex, null, title,
      0.1, 0.65, 80, 16,
      'constituent=1;whiteSpace=wrap;strokeColor=none;fillColor=none;fontColor=#e6a23c',
      true);
    titleVertex.setConnectable(false);

    const normalTypeVertex = graph.insertVertex(nodeRootVertex, null, null,
      0.05, 0.05, 19, 14,
      `normalType;constituent=1;fillColor=none;image=/static/images/normal-type/forest.png`,
      true);
    normalTypeVertex.setConnectable(false);
```

单单 `nodeRootVertex` 就是长这个样子。通过设置自定义的 `node` 样式与 `image` 属性设置图片路径配合完成。

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g0xqbyl8ovj304t04tglm.jpg)

因为默认情况下一个节点只能有一个文本区和一个图片区，要增加额外的文本和图片就需要组合节点。
在这个基础上加上一个 `titleVertex` 文本节点，还有一个 `normalTypeVertex` 图片节点，最终达到这个效果

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g0xkrsphwyj30cv09x3yv.jpg)


有时需要为不同子节点设置不同的鼠标悬浮图标，可以参考 xxx，通过一个自定义的标识实现这个

```
const setCursor = () => {
  graph.getCursorForCell = (cell) => {
    return cell.style.includes('normalType') ? 'pointer' : 'default';
  };
};
```

#### 编辑内容
下面这段代码是编辑内容比较常用的设置

```
// 编辑时按回车键不换行，而是完成输入
this.setEnterStopsCellEditing(true);
// 编辑时按 escape 后完成输入
mxCellEditor.prototype.escapeCancelsEditing = false;
// 失焦时完成输入
mxCellEditor.prototype.blurEnabled = true;
```

默认情况下输入内容时如果按回车键内容会换行，但有些业务场景禁止换行，回车表示完成输入，通过这行代码 `graph.setEnterStopsCellEditing(true)` 设置可以满足需求。

重点说说这句 `mxCellEditor.prototype.blurEnabled`。默认情况下如果用户在输入内容时鼠标点击了画布之外的不可聚焦区域(div、section、article等)，节点内的编辑器是不会失焦的，
这导致了 `LABEL_CHANGED` 事件不会被触发。但在实际项目开发中一般我们会期望，如果用户在输入内容时鼠标点击了画布之外的地方就应该算作完成一次输入，然后通过被触发的 `LABEL_CHANGED` 事件将修改后的内容同步到服务端。
通过 `mxCellEditor.prototype.blurEnabled = true` 这行代码设置可以满足我们的需求。

#### 可换行显示的 label
````
const titleVertex = graph.insertVertex(nodeRootVertex, null, title,
      0.1, 0.65, 80, 16,
      'constituent=1;whiteSpace=wrap;strokeColor=none;fillColor=none;fontColor=#e6a23c',
      true);
````

对于非输入的文本内容，默认情况下即便文本超出容器宽度也是不会换行的。我们项目中宽度为80的 titleVertex 正是这样一个例子。

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g0xmmixmizj30lg095abj.jpg)

要设置换行需要做两件事，
第一是通过这行代码 `graph.setHtmlLabels(true)`，使用 html 渲染文本(mxGraph 默认使用 svg的text 标签渲染文本)
第二是像上面的 titleVertex 的样式设置一样，添加一句 `whiteSpace=wrap`

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g0xmcy67uaj30mj0fedj4.jpg)

### Model
现在介绍一个概念 Model，Model 代表的是当前图形的数据结构化表示。mxGraphModel 封装了 Model 的相关操作。

你可以启动项目，画一个这样的图

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g0xy1zmmqrj30f10gb0tn.jpg)

导出之后应该等到这样一份 xml

```
<mxGraphModel>
  <root>
    <mxCell id="0"/>
    <mxCell id="1" parent="0"/>
    <mxCell id="4" value="男1号" style="node;image=/static/images/ele/ele-005.png" vertex="1" parent="1">
      <mxGeometry x="350" y="50" width="100" height="135" as="geometry"/>
      <Object normalType="water.png" as="data">
        <Object id="5" icon="ele-005.png" title="智爷" as="element"/>
      </Object>
    </mxCell>
    ...........
  </root>
</mxGraphModel>
```

这份 xml 就是当前图形的 model 的结构化表示，也就是当前 model。为了方便观察，我手动格式化“男1号”这个节点 

```xml
<mxCell 
  id="4"
  value="男1号" 
  style="node;image=/static/images/ele/ele-005.png"
  vertex="1" 
  parent="1">
  <mxGeometry 
    x="400" 
    y="70" 
    width="100" 
    height="135" 
    as="geometry"/>
  <Object 
    as="data"
    normalType="">
    <Object 
      as="element"
      id="5" 
      icon="ele-005.png" 
      title="智爷" />
  </Object>
</mxCell>
```

这样一看，这份数据与下面的对象本质上是不是同一个东西

```
id: "3"
mxObjectId: "mxCell#31"
parent: mxCell
children: (2) [mxCell, mxCell]
data: {element: {…}, normalType: ""}
geometry: mxGeometry {x: 0, y: 0, width: 100, height: 135}
mxObjectId: "mxCell#30"
style: "node;image=/static/images/ele/ele-005.png"
value: "鼠标双击输入"
vertex: true
__proto__: Object
source: null
style: "normalType;constituent=1;fillColor=none;image=/static/images/normal-type/forest.png"
target: null
value: null
vertex: true
```

### 事件

本项目用到事件监听写在 AppCanvas.vue 的 _listenEvent 方法，

![](https://jgraph.github.io/mxgraph/docs/js-api/images/images/callgraph.png)

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g0xo1ow1p0j30id010gls.jpg)

#### 监听事件

`mxGraph` 类继承自 `mxEventSource` 类，使用父亲 `addListener` 方法可以将自身当作一个事件中心订阅/广播事件。

`graph.getSelectionModel()` 返回一个 `mxGraphSelectionModel` 对象，这个对象有 `mxEvent.UNDO、mxEvent.CHANGE` 两个事件，
通过监听 `mxEvent.CHANGE` 事件可以获取当前被选中的 `Cell`

#### 区别

- 添加cell的时候会触发两个事件 `ADD_CELLS`、`CELLS_ADDED`， 先触发 `CELLS_ADDED` 后触发 `ADD_CELLS`。
- `ADD_CELLS`、`CELLS_ADDED` 的区别，`ADD_CELLS` 在 `addCells` 方法中触发，而 `CELLS_ADDED` 在 `cellsAdded` 方法中触发。
而对于 `addCells` 与 `cellsAdded` 官方文档也只是两个句的说明，体现不出多大区别，对于这两个方法的本质区别我还存有疑惑。
但按经验而言后触发的事件会携带更多的信息，所以平时开发我会监听`ADD_CELLS` 事件。`MOVE_CELLS、CELLS_MOVED`、`REMOVE_CELLS、CELLS_REMOVED` 等事件与此类似。

- 判断一个节点/线被添加

从图中我们可以看到，`insertVertex`、`insertEdge` 最终都被当作 `Cell` 处理，在后续触发的事件也没有对节点/线条进行区分，而是统一当作 `Cell` 事件。
所以对于一个 `Cell` 添加事件，需要自己区别是添加了节点还是添加了条线。

```
      graph.addListener(mxEvent.CELLS_ADDED, (sender, evt) => {
        const cell = evt.properties.cells[0];
        if (graph.isPart(cell)) {
          return;
        }

        if (cell.vertex) {
          this.$message.info('添加了一个节点');
        } else if (cell.edge) {
          this.$message.info('添加了一条线');
        }
      });
```

还有就是对于子节点添加到父节点的情况(如上面提到的将 `titleVertex` 、`normalTypeVertex` 添加到 `nodeRootVertex`)也是会触发 `Cell` 添加事件的。
通常对于这些子节点不作处理，可以像 `consti.html` 那个例子一样用一个 `isPart` 判断过滤掉。

#### 自定义事件

mxGraph 提供自定义事件功能，使用 fireEvent 与 mxEventObject TODO 加文档外链。下面代码是一个最简单的例子

```
graph.addListener('自定义事件A',()=>{
  // do something .....
});
// 触发自定义事件
graph.fireEvent(new mxEventObject('自定义事件A');
```

在本项目Graph类 (`src/graph/Graph.js` ) 的 `_configCustomEvent` 方法我也实现了两个自定义了事件。
当线条开始拖动时会触发 `EDGE_START_MOVE` 事件，当节点开始拖动时会触发 `VERTEX_START_MOVE` 事件。

### 导出图片
导出图片可以使用 `mxImageExport` 这个类实现，[官方文档](https://jgraph.github.io/mxgraph/docs/js-api/files/util/mxImageExport-js.html#mxImageExport.mxImageExport)
也有一段可以使用拿出使用的代码。

```
var xmlDoc = mxUtils.createXmlDocument();
var root = xmlDoc.createElement('output');
xmlDoc.appendChild(root);

var xmlCanvas = new mxXmlCanvas2D(root);
var imgExport = new mxImageExport();
imgExport.drawState(graph.getView().getState(graph.model.root), xmlCanvas);

var bounds = graph.getGraphBounds();
var w = Math.ceil(bounds.x + bounds.width);
var h = Math.ceil(bounds.y + bounds.height);

var xml = mxUtils.getXml(root);
new mxXmlRequest('export', 'format=png&w=' + w +
     '&h=' + h + '&bg=#F9F7ED&xml=' + encodeURIComponent(xml))
     .simulate(document, '_blank');
```

但这段代码会将整块画布截图，而不是以最左上角的元素及最右下角的元素作为边界截图。如果你有以元素作为边界的需求，
则需要调用 `xmlCanvas.translate` 调整裁图边界。

```
//.....
var xmlCanvas = new mxXmlCanvas2D(root);
xmlCanvas.translate(
      Math.floor((border / scale - bounds.x) / scale),
      Math.floor((border / scale - bounds.y) / scale),
    );
//.....
```

加上上面的代码后就可以达到以最左上角的元素及最右下角的元素作为边界的截图效果。可以参考 Graph 类的 exportPicXML 方法

todo 补充java截图代码

如果图片生成失败，请使用 png 或 jpg 格式。

## 总结 
- 使用的所有 demo
- 知识点
- 注意问题 style颜色要有6位、- 左上边界搞不定
