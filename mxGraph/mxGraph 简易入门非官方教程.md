## 引入
### 使用 script 引入
[Hello World](https://github.com/jgraph/mxgraph/blob/master/javascript/examples/helloworld.html)

我们先来分析一下官方是怎样通过 script 标签引入 mxGraph 的

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
</html>
```

首先要声名一个全局变量 `mxBasePath` 指向一个路径，然后引入 mxGraph。

`mxBasePaht` 指向的路径作为 mxGraph 的静态资源路径，这些资源除了 `js目录` ，其他目录下的资源都是 mxGraph 运行过程中所需要的，所以要在引入 mxGraph 前先设置 `mxBasePaht`。 

再来看看 javascript 目录下有两个 `mxClient.js` 版本。 一个在 `javascript/src/js/mxClient.js` ，另一个在 `javascript/mxClient.js`，后者是前者后的版本。
所以两者可以替换使用的，如果你的项目是使用script标签引入 mxGraph，可以参考我的引入方式。

### 模块化引入
模块化引入可以直接参考我的项目，static/mxgraph

```js
// 引入 mxgraph
// src/graph/index.js
import mx from 'mxgraph';

const mxgraph = mx({
  mxBasePath: '/static/mxgraph',
});

// decode bug https://github.com/jgraph/mxgraph/issues/49
window['mxGraph'] = mxgraph.mxGraph;
window['mxGraphModel'] = mxgraph.mxGraphModel;
window['mxEditor'] = mxgraph.mxEditor;
window['mxGeometry'] = mxgraph.mxGeometry;
window['mxDefaultKeyHandler'] = mxgraph.mxDefaultKeyHandler;
window['mxDefaultPopupMenu'] = mxgraph.mxDefaultPopupMenu;
window['mxStylesheet'] = mxgraph.mxStylesheet;
window['mxDefaultToolbar'] = mxgraph.mxDefaultToolbar;

export default mxgraph;

// 在其他模块中使用
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

有两点需要特别注意的

`mxBasePath` 指向的路径一定要是一个可以通过url访问的静态资源目录。
比如说我项目的 static 目录下有 `mxgraph/css/common.css` 这么个资源，我的项目部署在 `http://localhost:7777`，那么 `http://localhost:7777/static/mxgraph/css/common.css` 应该是可以访问才对。

还有就是有一段绑定全局变量的代码，如果你在通过 script标签引入 mxGraph，这段代码是不需要的。
之所有要这样做是因为，mxGraph 有些源码是通过 window.xxx 对以上的属性进行访问的，如果不做全局绑定使用起来会有点问题。
这是官方还没修复的一个 BUG，详情可以查看上面注释的 issue。

## 小Demo 讲解
### 插入节点

### 事务
这个 Hello World 的例子并不难。

```js
const parent = graph.getDefaultParent();
graph.insertVertex(parent, null, 'Hello,', 20, 20, 80, 30);
```

这段代码可以理解为将节点插入到画布，当然还可以将节点插入到另一个节点中，后面会讲述。

还有就是 `beginUpdate` 与 `endUpdate` 这两个方法，这两个方法在官方例子中出镜频率非常高，
我们来了解一下他们是干嘛用的，嗯，真是只是了解一下就可以了，因为官方对两个方法的描述对入门者来说真的是比较晦涩难懂，而且我在实际开发中基本用不上这两个方法。
可以等掌握这个框架基本使用后再回过头来研究

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g0vjeq1nasj30pe0isn1k.jpg)

- beginUpdate、endUpdate 用于创建一个事务，一次 beginUpdate 必须对应一次 endUpdate
- 为了保证，假如 beginUpdate 执行失败，endUpdate 不会被调用，`beginUpdate 一定要放到 try 块之外`
- 为了保证，假如try块内更新失败，endUpdate 也一定会执行，`beginUpdate一定要放到 finally 块`
- 使用 beginUpdate 与 endUpdate 可提高更新视图性能，框架内部做撤消/重做管理也需要 beginUpdate、endUpdate

你可以试着把这两个方法从代码中删掉，程序还是可以正常运行。

### style 的两种方式，小技巧

`mxStylesheet` 默认情况下有两个属性 defaultEdgeStyle 、defaultVertexStyle。
修改这两个属性对所有线条/节点都生效。

你还可以使用 `mxStylesheet.putCellStyle` 为 mxStylesheet 添加样式对象。然后在添加 Cell 的时候，写在参数中。 

```js
this.getStylesheet().putCellStyle('normalType', normalTypeStyle);
```


### 节点组合、相对坐标

width、height 等以像素为单位

坐标系 左上 0,0

https://jgraph.github.io/mxgraph/docs/images/mx_man_rel_vert_pos.png

https://jgraph.github.io/mxgraph/docs/images/mx_man_non_relative_pos.png

默认情况下创建一个 relative 为 false 的节点，所以组合节点时需要将 relative 为 true

默认情况下创建一条 relative 为 true 的线条

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

这两个方法对原方法做了继承，思路都是判断如果该节点是子节点则替换成父节点去处理剩下的逻辑。

第一个方法 `getInitialCellForEvent` 在鼠标按下(mousedown事件，不是click事件)时触发，
如果注释掉这段代码，不使用父节点替换，当发生拖拽时子节点会被单独拖拽，不会与父节点联动。
使用父节点替换后，原本子节点应该被拖拽，现在变成了父节点被拖拽，会发生联动效果。

通过 debugger 进去 `getInitialCellForEvent` 可以得知，第二个方法 `selectCellForEvent` 其实是 `getInitialCellForEvent` 内部调用的一个方法。
这个方法的作用是将 cell 设置为 `selectCell`，设置后通过 `graph.getSelectoinCell` 可获取得该节点，与 `getInitialCellForEvent` 同理，如果不使用父节点替换，
则 `graph.getSelectoinCell` 获取到的会是子节点。

## 添加 Cell
### 在画布上添加Cell

```js
    const parent = graph.getDefaultParent();
    graph.getModel().beginUpdate();
    try {
        const v1 = graph.insertVertex(parent, null, 'Hello,', 20, 20, 80, 30);
        const v2 = graph.insertVertex(parent, null, 'World!', 200, 150, 80, 30);
        const e1 = graph.insertEdge(parent, null, '30%', v1, v2);
    } finally {
        graph.getModel().endUpdate();
    }
```

### 在 Cell 里面再添加 Cell

分析官方 constiuent.html demo

```js
    const nodeRootVertex = new mxCell('Name', new mxGeometry(0, 0, 100, 135), `node;image=${src}`);
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

```js
// Redirects selection to parent
graph.selectCellForEvent = function(cell)
{
    if (this.isPart(cell))
    {
        cell = this.model.getParent(cell);
    }
    
    mxGraph.prototype.selectCellForEvent.apply(this, arguments);
};
```

重点说说这段代码
