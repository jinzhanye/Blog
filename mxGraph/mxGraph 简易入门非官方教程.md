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
