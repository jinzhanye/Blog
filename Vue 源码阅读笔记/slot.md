## 普通插槽

例子

````js
let AppLayout = {
  template: `<div class="container">
                <header><slot name="header"></slot></header>
                <main><slot>默认内容</slot></main>
                <footer><slot name="footer"></slot></footer>
              </div>`
};

let vm = new Vue({
  el: '#app',
  template: `<div>
                <app-layout>
                    <h1 slot="header">{{title}}</h1>
                    <p>{{msg}}</p>
                    <p slot="footer">{{desc}}</p>
                </app-layout>
              </div>`,
  data() {
    return {
      title: '我是标题',
      msg: '我是内容',
      desc: '其它信息'
    }
  },
  components: {
    AppLayout
  }
});
````

### 父组件

父组件模块 `parse` 之后，有 `slot` 属性的标签的 `AST节点` 会多出一个 `slotTarget`。例如 `h1` 标签的 `AST节点` 如下

````js
{
    type:1
    tag:"h1"
    attrsList:[]
    attrsMap:{...}
    parent:{...}
    children:[...]
    plain:false
    slotTarget:"\"header\""
    staticRoot:false
}
````

AST 经 `codegen` 阶段生成的代码如下：

````js
with(this){
  return _c('div',
    [_c('app-layout',
      [_c('h1',{attrs:{"slot":"header"},slot:"header"},
         [_v(_s(title))]), // 空白字符
       _c('p',[_v(_s(msg))]),
       _c('p',{attrs:{"slot":"footer"},slot:"footer"},
         [_v(_s(desc))]
         )
       ])
     ],
   1)}
````

`slot:"header"` 等，就是从 AST 的 `slotTarget:"\"header\""` 中取值的。

### 子组件

子组件的 `slot` 的 AST节点如下：

````js
{
    type:1
    tag:"slot"
    attrsList:[]
    attrsMap:{...}
    parent:{...}
    children:[]
    plain:false
    slotName:"\"header\""
    static:false
    staticRoot:false
}
````

`slotName` 对应 `<slot name="header">` 中 `name` 的值，如果无 `name` 属性，则 `slotName` 值为 `undefined`。

`slotName` 值为 `undefined`，`codegen` 阶段用 `default` 代替。最终生成代码如下：

````js
// _t = renderSlot
// _v = createTextVNode
with(this) {
  return _c('div',{
    staticClass:"container"
    },[
      _c('header',[_t("header")],2),
      _c('main',[_t("default",[_v("默认内容")])],2),
      _c('footer',[_t("footer")],2)
      ]
   )
}
````
 
## render

`_t` 对应是的 `renderSlot` 函数，从下面抽出主干流程的 `renderSlot` 可以看出，程序用插槽名作为`key` 从 `$slots` 取出对应的 `VNode` 数组，然后返回 

````js
export function renderSlot (
  name: string,
  fallback: ?Array<VNode>,
  props: ?Object,
  bindObject: ?Object
): ?Array<VNode> {
  let nodes
  // ....... 
  const slotNodes = this.$slots[name]
  // .......  
  nodes = slotNodes || fallback
  // .......
  return nodes
}
````

在子组件 `AppLayout` 在 `init` 阶段会获取 `vm.$slot` 属性， 调用链为 `init -> initRender -> resolveSlots`

````js
export function initRender (vm: Component) {
    // ....
    vm.$slots = resolveSlots(options._renderChildren, renderContext)
    // ....
}

export function resolveSlots (
  children: ?Array<VNode>,
  context: ?Component
): { [key: string]: Array<VNode> } {
  const slots = {}
  if (!children) {
    return slots
  }
  for (let i = 0, l = children.length; i < l; i++) {
    const child = children[i]
    const data = child.data
    // remove slot attribute if the node is resolved as a Vue slot node
    if (data && data.attrs && data.attrs.slot) {
      delete data.attrs.slot
    }
    // named slots should only be respected if the vnode was rendered in the
    // same context.
    if ((child.context === context || child.fnContext === context) &&
      data && data.slot != null
    ) {
      // 对应父组件  _c('h1',{attrs:{"slot":"header"},slot:"header"} ....
      // slot 属性就是 name 
      const name = data.slot
      const slot = (slots[name] || (slots[name] = []))
      if (child.tag === 'template') {
        slot.push.apply(slot, child.children || [])
      } else {
        slot.push(child)
      }
    } else {
      (slots.default || (slots.default = [])).push(child)
    }
  }
  // ....
  return slots
}
````

`resolveSlots` 最终获取按插槽名为 `key` 的对象。而每个属性对应的是 `VNode` 数组。

````js
vm.$slots = {
 default:[VNode,VNode,VNode], // 其中两个是空白字符 VNode
 footer:[VNode],
 header:[VNode]  
}
````

在组件更新阶段因为不走 `init`，所以会在 `updateChildComponent` 更新 `$slot`。之所以后面还需要调用 `$forceUpdate`，是要让子组件在下次 `patch` 中 `diff` 更新。

````js
// src/core/instance/lifecycle.js
export function updateChildComponent (
  vm: Component,
  propsData: ?Object,
  listeners: ?Object,
  parentVnode: MountedComponentVNode,
  renderChildren: ?Array<VNode>
) {
  const hasChildren = !!(
    renderChildren ||          
    vm.$options._renderChildren ||
    parentVnode.data.scopedSlots || 
    vm.$scopedSlots !== emptyObject 
  )

  // ...
  if (hasChildren) {
    vm.$slots = resolveSlots(renderChildren, parentVnode.context)
    vm.$forceUpdate()
  }
}
````

## _renderChildren

那么 `initRender` 的 `options._renderChildren` 是从哪里来的？

````js
with(this){
  return _c('div',
    [_c('app-layout',
      [_c('h1',{attrs:{"slot":"header"},slot:"header"},
         [_v(_s(title))]),
       _c('p',[_v(_s(msg))]),
       _c('p',{attrs:{"slot":"footer"},slot:"footer"},
         [_v(_s(desc))]
         )
       ])
     ],
   1)}
````

其实是调用 `_c('app-layout',/*.....*/)` 时获取的，`_c` 就是 `createElement(/*....*/,children,/*....*/)`，`app-layout` 下的 `子节点VNode` 数组，作为 `children` 参数。`createElement` 过程还会调用 `createComponent`，
`createComponent` 有这么一段代码，就是将 `children` 作为参数构造 `VNode`。

````js
const vnode = new VNode(
  `vue-component-${Ctor.cid}${name ? `-${name}` : ''}`,
  data, undefined, undefined, undefined, context,
  { Ctor, propsData, listeners, tag, children },
  asyncFactory
)
````

此时 `vnode.componentOptions.children` 就是上面提到的 `children`。再执行到子组件 `init` 阶段选项合并，就会将 `children` 赋值给 `opts._renderChildren`。也就是我们在 `initRender` 中看到的 `options._renderChildren`

````js
export function initInternalComponent (vm: Component, options: InternalComponentOptions) {
  const opts = vm.$options = Object.create(vm.constructor.options)
  const parentVnode = options._parentVnode
  // .......  
  const vnodeComponentOptions = parentVnode.componentOptions
  opts._renderChildren = vnodeComponentOptions.children
  // .......  
}
````
