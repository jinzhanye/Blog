- 核心方法
    1. createElement
    2. patch(container,vnode) 将 vnode 渲染成真实 dom
    3. patch(vnode, newVnode) 更新 vnode
        
        ````js
        //  Vue.prototype._update
        if (!prevVnode) {
          vm.$el = vm.__patch__(vm.$el, vnode, hydrating, false /* removeOnly */)
        } else {
          vm.$el = vm.__patch__(prevVnode, vnode)
        }
        ````

- 渲染 vnode 与 占位符 vnode 的区别
    1. tag 为原生标签的vnode ，叫做渲染 vnode，tag 为组件标签的叫占位符 vnode 
    1. 渲染 vnode 没有 componentInstance 属性，而占位符 vnode 有
    1. 渲染 vnode 的 tag 为原生 tag，而占位符 vnode 的 tag 为 'vue-component'-cid-componentName
    1. 渲染 vnode 有 children 属性， 占位符 vnode 的 children 属性为 undefined。具体见 create-component.js
    
        ````js
        const vnode = new VNode(
        `vue-component-${Ctor.cid}${name ? `-${name}` : ''}`,
        data, undefined, undefined, undefined, context,
        { Ctor, propsData, listeners, tag, children },
        asyncFactory
        )
        ````

- vnode 与 vm 的相互引用 
    - 组件vnode, vnode.componentInstance 指向vm，定义在 create-component.js componentVNodeHooks的init方法
    - vm.$vnode 指向该组件 vnode，定义在 Vue.prototype._render `vm.$vnode = _parentVnode`
    - vm._node 指向 render 返回的 vnode， _update 方法 `vm._vnode = vnode`

- vm.$el 的来源
    - patch 返回创建后的真实DOM , `vm.$el = vm.__patch__(prevVnode, vnode)` 

- vnode.componentInstance 与 vnode.context 的区别
vnode.context 指向 vnode 所对应的 vm 实例。而 vnode.componentInstance 指向该占位符 vnode 对应的渲染 vnode 的 vm 实例
   
- vnode.elm 的来源
    - 无论是渲染vnode，还是占位符vnode, vnode.elm 属性必须是创建这个vnode对象的真实DOM后才定义的
    - 渲染vnode，`vnode.elm = nodeOps.createElement(tag, vnode)`，见 patch.js/createElm
    - 占位符vnode，`vnode.elm = vnode.componentInstance.$el` ，见 patch.js/createElm/createComponent/initComponent
    - vnode.elm 与 vm.$el 是同一个东西。`vnode.componentInstance` 指向当前组件的渲染vnode 对应的 vm 实例
