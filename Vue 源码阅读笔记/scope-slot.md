````js
const Child = {
  template: `<div class="child">
              <slot text="Hello " :msg="msg"></slot>
              <slot name="slot2" :msg2="msg2"></slot>
            </div>`,
  data() {
    return {
      msg: 'Vue',
      msg2: 'angular',
    }
  }
}

new Vue({
  el: '#app',
  template: `<div>
              <child>
                <template slot-scope="props">
                  <p>Hello from parent</p>
                  <p>{{ props.text + props.msg}}</p>
                </template>
                <div slot="slot2" slot-scope="props2">
                  <p>{{props2.msg2}}</p>
                </div>
              </child>
             </div>`,
  components: {
    Child
  }
})
````

## 父组件

`AST` 树，与普通 `slot` 的区别在 `scope-slot` 组件内的节点不会作为 `children` 保存。 还有就是多出一个 `scopedSlots` 对象，默认 `slot` 的名称为 `defalut`

````
child = {
    children:[], // 组件内的节点不会作为children 保存
    tag:"child",
    type:1,
    scopedSlots:{
        "default":{
            attrsMap:{
               slot-scope: "props"
            }
            children:[VNode,VNode],
            slotScope :"props",
            tag:"template",
            type:1,
             //......
        }
        "slot2":{
            attrsMap:{
                slot:"slot2",
                slot-scope:"props2",
            }
            children:[VNode],
            slotTarget:""slot2"",
            slotScope :"props2",
            tag:"div",
            type:1,
        }        
        //......
    },
    // .....
}
````

`codegen` 生成代码如下，与普通 `slot` 的区别在，`scopedSlots` 对象作为 `data` 对象的一个属性传入到渲染 `child` 组件的函数。后面我们会介绍 `_u` 函数的作用。

````js
with (this) {
    return _c('div', [_c('child', {
        scopedSlots: _u([{
            key: "default",
            fn: function (props) {
                return [_c('p', [_v("Hello from parent")]), // template 节点返回 childrenList
                    _c('p', [_v(_s(props.text + props.msg))])]
            }
        }, {
            key: "slot2",
            fn: function (props2) {
                return _c('div', {}, [_c('p', [_v(_s(props2.msg2))])]) // p 作为 div 的 children
            }
        }])
    })], 1)
}
````

## 子组件

`AST` 树

````js
children =[
    {
        type: 1,
        tag: "slot",
        attrsList: [{
            name: "text",
            value: "Hello",
        }, {
            name: ":msg",
            value: "msg",
        }],
        attrsMap: {
            text: "Hello",
            ':msg': "msg",
        },
        slotName: undefined,
        attrs: [{
            name: "text",
            value: "\"Hello\"",
        }, {
            name: "msg",
            value: "msg",
        }]
        // ......
    },{
        slotName: 'slot2',
        // ......
    }
]
````

`codegen` 生成代码如下

````js
with(this){
  return _c('div',
    {staticClass:"child"},
    [_t("default",null,
      {text:"Hello ",msg:msg}
    ),_t("slot2",null,
       {msg2:msg2}
     )],
  2)}
````

## render

下面介绍渲染父组件用到的函数 `_u`，与渲染子组件用到的函数 `_t`。

`_u` 函数对的就是 resolveScopedSlots 方法，它的定义在 `src/core/instance/render-heplpers/resolve-slots.js`

````js
export function resolveScopedSlots (
  fns: ScopedSlotsData, // see flow/vnode
  res?: Object
): { [key: string]: Function } {
  res = res || {}
  for (let i = 0; i < fns.length; i++) {
    if (Array.isArray(fns[i])) {
      resolveScopedSlots(fns[i], res)
    } else {
      res[fns[i].key] = fns[i].fn
    }
  }
  return res
}
````

就是返回一个以插槽名为 `key` ，以 `fn` 为 `value` 的对象。

````js
scopedSlots = res = {
    default:function(props){/*....*/},
    slot2:function(props2){/*....*/}
}
````

在子组件执行 `vm._render` 时会将 `scopedSlots` 赋值给`$scopedSlots` 赋值，供后面 `_t` 使用。 

````js
Vue.prototype._render = function (): VNode {
    //....
     if (_parentVnode) {
      vm.$scopedSlots = _parentVnode.data.scopedSlots || emptyObject
    }
    //....
}
````

`_t` 也就是 `renderSlot` ，就是从 `$scopedSlots` 中取出 `fn` 执行生成 `VNode`。

````js
export function renderSlot (
  name: string,
  fallback: ?Array<VNode>,
  props: ?Object,
  bindObject: ?Object
): ?Array<VNode> {
  const scopedSlotFn = this.$scopedSlots[name]
  let nodes
  if (scopedSlotFn) {
    props = props || {}
    // .......
    nodes = scopedSlotFn(props) || fallback
  } else {
    // ...
  }

  // ......
  return nodes
}
````

而 `fn` 也就调用 `_c` 生成 `VNode` 的，在 `initRender` 中绑定 `vm._c`，可以看到作用域是当前组件的 `vm` 实例。
 
````js
// initRender
vm._c = function (a, b, c, d) { return createElement(vm, a, b, c, d, false); }
````

## 抄的总结
- 普通插槽是在`父组件渲染阶段`生成 `vnodes`，所以数据的作用域是父组件实例，子组件渲染的时候直接拿到这些渲染好的 `vnodes`。
- 作用域插槽，父组件在编译和渲染阶段并不会直接生成 `vnodes`，而是在父节点 `vnode` 的 `data` 中保留一个 `scopedSlots` 对象，存储着不同名称的插槽以及它们对应的渲染函数，
只有在`子组件渲染阶段`才会执行这个渲染函数生成 `vnodes`，由于是在子组件环境执行的，所以对应的数据作用域是子组件实例。

简单地说，两种插槽的目的都是让子组件 slot 占位符生成的内容由父组件来决定，但数据的作用域会根据它们 vnodes 渲染时机不同而不同。

## 参考 
[Vue.js技术揭秘/slot](https://ustbhuangyi.github.io/vue-analysis/extend/slot.html#%E4%BD%9C%E7%94%A8%E5%9F%9F%E6%8F%92%E6%A7%BD)
