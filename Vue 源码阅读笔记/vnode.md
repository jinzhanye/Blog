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

- 普通元素节点 vnode 与 组件节点 vnode 的区别
    1. 普通元素节点 vnode 有 children 属性， 而组件节点 vnode 的 children 属性为 undefined。具体见 create-component.js
    
        ````js
        const vnode = new VNode(
        `vue-component-${Ctor.cid}${name ? `-${name}` : ''}`,
        data, undefined, undefined, undefined, context,
        { Ctor, propsData, listeners, tag, children },
        asyncFactory
        )
        ````
    1. 普通元素节点 vnode._isComponent 为 false，而组件节点 vnode._isComponent 为 true。 见 createComponentInstanceForVnode 方法

    
- vnode 与 vm 的相互引用 
    - 组件vnode, vnode.componentInstance 指向vm，定义在 create-component.js componentVNodeHooks的init方法
    - vm.$vnode 指向该组件 vnode，定义在 Vue.prototype._render `vm.$vnode = _parentVnode`
    - vm._node 指向 render 返回的 vnode， _update 方法 `vm._vnode = vnode`

无论是渲染vnode，还是占位符vnode, vnode.elm 属性必须是创建这个vnode对象的真实DOM后才定义的
- 渲染vnode:patch.js/createElm `vnode.elm = nodeOps.createElement(tag, vnode)`
- 占位符vnode，`vnode.elm = vnode.componentInstance.$el` ，见 patch.js/createElm/createComponent/initComponent
- patch 返回创建后的真实DOM , `vm.$el = vm.__patch__(prevVnode, vnode)` 
- vm.$el 与 vnode.elm 是同一个东西。上面提到的vnode.componentInstance就是 vm
