- 初始化过程 $mount
 - 非组件 $mount
 - 组件 $mount
 
 
1. Vue  只能通过 new 关键字初始化，然后构造函数内部调用 this._init 方法
1. _init 方法主要就干了几件事件，合并配置、初始化生命周期、初始化事件、初始化渲染、初始化 data、props、computed、watcher 等等，最后调用 vm.$mount(vm.$options.el)
1. runtime with compiler 版本的 $mount 方法将 template 转换成 render 函数(在 Vue 2.0 版本中，所有 Vue 的组件的渲染最终都需要 render 方法，无论我们是用单文件 .vue 方式开发组件，还是写了 el 或者 template 属性，最终都会转换成 render 方法，那么这个过程是 Vue 的一个“在线编译”的过程)，
然后将这个 render 函数赋值给 options.render。最后调用原来的 Vue.prototype.$mount，mount.call(this, el, hydrating)
1. Vue.prototype.$mount 根据 el 查询得到原生 DOM 对象， 然后调用 mountComponent(this, elDOM, hydrating)
1. mountComponent 主要做了两件事
    1. 定义了 updateComponent 函数，该函数作用调用 `vm._render` 方法生成 vnode，然后将 vnode 作为参数调用 `vm._update`
    1. 实例化 render watcher，在它的回调函数当中调用 updateComponent。最终调用 `vm._update` 更新 DOM
 
1. _render 设置 `vm.$vnode = _parentVnode` ,调用 `$options.render.call(vm._renderProxy, vm.$createElement)` (render 是经过编译template 生成的，或者用户自己手写的)，生成 vnode，然后将其返回。
1. render 调用 createElement，createElement 方法实际上是对 _createElement 方法的封装，它允许传入的参数更加灵活，在处理这些参数后，调用真正创建 VNode 的函数 _createElement
1. _createElement 主要做了两件事。
    1. 规范化 children
    1. 创建 vnode。如果判断 tag 为 原生tag ，就调用 new VNode 创建 vnode。如果 tag 为 组件名tag / options / constructor 则调用 createComponent(这个函数名起得非常不好，如果改为 createComponentVNode 会更加直观) 创建组件类型 vnode (注意，这个 createComponent 不是 patch 的 那个 createComponent，并没有递归的过程，只是简单成构造一个 vnode 实例)。
    
1. 回到 `vm._update`，主要是调用patch方法，并将结果保存到 `vm.$el`， `vm.$el = vm.__patch__(vm.$el, vnode, hydrating, false /* removeOnly */)`
1. patch。顶级节点 patch 时，oldVNode 不为 undefined，执行以下步骤
    1. 调用 emptyNodeAt 方法把 $el 转换成 vnode 实例，也就是 oldVnode
    1. 调用 createEle ，将 vnode 转化成真实 DOM，将并插入到它的父节点中。
    1. 从父节点中删除 oldVnode
    1. 调用 `invokeInsertHook(vnode, insertedVnodeQueue, isInitialPatch)` 遍历 insertedVnodeQueue，执行每个组件的 mounted 方法 
    1. 返回 vnode.elm
   
   组件第一次 pacth时，oldVNode 为 undefined，执行以下步骤
   
   1. 调用 createEle ，将 vnode 转化成真实 DOM。
   1. 调用 `invokeInsertHook(vnode, insertedVnodeQueue, isInitialPatch)` 遍历 insertedVnodeQueue，执行每个组件的 mounted 方法 
   1. 返回 vnode.elm
   
   最后回溯到 pacth.js/createComponent，才将这个真实DOM 插入到父节点。
    
1. createEle
    1. 调用 createComponent 递归创建组件，如果返回 true，则直接 return。否则继续往下走
    1. 将 vnode 转化成真实 DOM
    1. 将当前 vnode 作为父节点，调用 createChildren 为孩子节点递归调用 createEle
    1. 将真实 DOM 插入到父节点   

patch.js/createComponent 
1. 调用 vnode.data.hook.init 方法以初始化 VueComponent
1. 调用 initComponent 获取 vnode.elm
1. 将 vnode.elm 插入到父节点
   
create-component.js/createComponent 做三件事
1. 调用 `Vue.extend` ，构造 Vue 子类构造函数，也就是 VueComponent
1. 调用 `installComponentHooks(data)`，安装组件钩子函数
1. 实例化 vnode 并返回。vnode.data.hook 为组件钩子，vnode.componentOptions.Ctor 即 VueComponent 构造函数
