# 编译小结

## parse
对模板做解析，生成 AST，它是一种抽象语法树

````html
<ul :class="bindCls" class="list" v-if="isShow">
    <li v-for="(item,index) in data" @click="clickItem(index)">{{item}}:{{index}}</li>
</ul>
````

经过 parse 过程后，生成的 AST 如下：

````js
ast = {
  'type': 1,
  'tag': 'ul',
  'attrsList': [],
  'attrsMap': {
    ':class': 'bindCls',
    'class': 'list',
    'v-if': 'isShow'
  },
  'if': 'isShow',
  'ifConditions': [{
    'exp': 'isShow',
    'block': // ul ast element
  }],
  'parent': undefined,
  'plain': false,
  'staticClass': 'list',
  'classBinding': 'bindCls',
  'children': [{
    'type': 1,
    'tag': 'li',
    'attrsList': [{
      'name': '@click',
      'value': 'clickItem(index)'
    }],
    'attrsMap': {
      '@click': 'clickItem(index)',
      'v-for': '(item,index) in data'
     },
    'parent': // ul ast element
    'plain': false,
    'events': {
      'click': {
        'value': 'clickItem(index)'
      }
    },
    'hasBindings': true,
    'for': 'data',
    'alias': 'item',
    'iterator1': 'index',
    'children': [
      'type': 2,
      'expression': '_s(item)+":"+_s(index)'
      'text': '{{item}}:{{index}}',
      'tokens': [
        {'@binding':'item'},
        ':',
        {'@binding':'index'}
      ]
    ]
  }]
}
````

这是一个在线 [Vue AST 预览工具](https://ktsn.github.io/vue-ast-explorer/)

## optimize

`optimize` 对每个 `AST 节点` 标记 `static`、`staticRoot`。这是经过 `optimize` 后的 AST,多了 `static`、`staticRoot` 两个属性。 

````js
ast = {
      type:1
      tag:"div"
      attrsList:[]
      attrsMap:{
         class:"counter"
      }
      parent:undefined
      children:[...]
      plain:false
      staticClass:"\"counter\""
      static:false
      staticRoot:false
 }
````

## codegen

`codegen` 把优化后的 AST 树转换成可执行的代码

````js
with(this){
  return (isShow) ?
    _c('ul', {
        staticClass: "list",
        class: bindCls
      },
      _l((data), function(item, index) {
        return _c('li', {
          on: {
            "click": function($event) {
              clickItem(index)
            }
          }
        },
        [_v(_s(item) + ":" + _s(index))])
      })
    ) : _e()
}
````

这里的 _c 函数定义在 `src/core/instance/render.js` 中，作用是执行 createElement 去创建 VNode。

````js
vm._c = (a, b, c, d) => createElement(vm, a, b, c, d, false)
````

而 `_l`、`_v` 定义在 `src/core/instance/render-helpers/index.js`

````js
export function installRenderHelpers (target: any) {
  target._o = markOnce
  target._n = toNumber
  target._s = toString
  target._l = renderList
  target._t = renderSlot
  target._q = looseEqual
  target._i = looseIndexOf
  target._m = renderStatic
  target._f = resolveFilter
  target._k = checkKeyCodes
  target._b = bindObjectProps
  target._v = createTextVNode
  target._e = createEmptyVNode
  target._u = resolveScopedSlots
  target._g = bindObjectListeners
}
````

## 参考 
[Vue.js 技术揭秘/编译](https://ustbhuangyi.github.io/vue-analysis/compile/)
