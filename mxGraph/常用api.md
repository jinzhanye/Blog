## TODO
- ~~禁止从中间拉出线条~~
- ~~线条设置成不可调节~~
- ~~点击的节点层级设最高~~
- ~~线条层级设最低~~
- 序列化

## 常用 API
- graph.getCellStyle(cell) 获取样式字典
- graph.isMultigraph 判断是否多个图(即是不存在游离节点) 
- mxConnectionHandler.prototype.validateConnection 验证是否可连接
// Enables return key to stop editing (use shift-enter for newlines)
			graph.setEnterStopsCellEditing(true);
- // 禁止拖动线条
        mxGraph.prototype.isCellMovable = cell => !cell.edge;			

      graph.addListener(mxEvent.CLICK, function (sender, evt) {
        const cell = evt.getProperty('cell');

        const isLabelEvent = mxCellRenderer.prototype.isLabelEvent;
        mxCellRenderer.prototype.isLabelEvent = function (state, evt) {
          const res = isLabelEvent.apply(this, arguments);
          if (res) {
            console.log(res);
            debugger;
          }
          return res;
        };
      });

      const createLabel = mxCellRenderer.prototype.createLabel;

      const mxCellEditorInit = mxCellEditor.prototype.init;
      mxCellEditor.prototype.init = function (...args) {
        console.log(graph.getSelectionCell());
        mxCellEditorInit.apply(this, args);
        mxEvent.addListener(this.textarea, 'input', function (evt) {
          console.log(evt.target.innerText);
        });
      };


验证连接
````js
const isValidConnection = mxGraph.prototype.isValidConnection;
mxGraph.prototype.isValidConnection = function(){
  console.log('1212121212');
  console.log(graph.getEdgesBetween(arguments[0], arguments[1]));
  // return isValidConnection.apply(this, arguments);
  return true;
}
````

高亮相关
````js
 // mxConstraintHandler.prototype.highlightColor = '#333';
    mxConstants.HIGHLIGHT_OPACITY = 30;
    mxConstants.HIGHLIGHT_SIZE = 5;

    const setHighlightColor = mxCellHighlight.prototype.setHighlightColor;
    mxCellHighlight.prototype.setHighlightColor = function (color) {
      color = '#008aff';
      return setHighlightColor.call(this, color);
    }
    mxConstants.VERTEX_SELECTION_DASHED = true;
    mxConstants.EDGE_SELECTION_DASHED = true;
    mxConstants.EDGE_SELECTION_COLOR = '#008aff';
    mxConstants.VERTEX_SELECTION_COLOR = '#008aff';
    mxConstants.EDGE_SELECTION_STROKEWIDTH = 1;
    mxConstants.VERTEX_SELECTION_STROKEWIDTH = 1;
````

mxCellHighlight.prototype.keepOnTop = true;

核心类 mxGraphView、mxGraphSelection、mxStyleSheet、mxCellRender、mxGraphModel
