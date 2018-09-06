https://cn.vuejs.org/v2/api/#keep-alive
````
<keep-alive> 是一个抽象组件：它自身不会渲染一个 DOM 元素，也不会出现在父组件链中
当组件在 <keep-alive> 内被切换，它的 activated 和 deactivated 这两个生命周期钩子函数将会被对应执行。
````

````js
export default {
  name: 'keep-alive',
  abstract: true,
  
  // .......

  created () {
    this.cache = Object.create(null)
    this.keys = []
  },

  // .......

  render () {
    const slot = this.$slots.default
    const vnode: VNode = getFirstComponentChild(slot)
    const componentOptions: ?VNodeComponentOptions = vnode && vnode.componentOptions
    if (componentOptions) {
       // .......
      const { cache, keys } = this
      const key: ?string = vnode.key == null
        // same constructor may get registered as different local components
        // so cid alone is not enough (#3269)
        ? componentOptions.Ctor.cid + (componentOptions.tag ? `::${componentOptions.tag}` : '')
        : vnode.key
      if (cache[key]) {
        vnode.componentInstance = cache[key].componentInstance
        // make current key freshest
        remove(keys, key)
        keys.push(key)
      } else {
        cache[key] = vnode
        keys.push(key)
      }

      vnode.data.keepAlive = true
    }
    return vnode || (slot && slot[0])
  }
}
````
`<keep-alive>` 在 `created` 钩子里定义了 `this.cache` 和 `this.keys`，本质上它就是去缓存已经创建过的 vnode。

````js
const slot = this.$slots.default
const vnode: VNode = getFirstComponentChild(slot)
````

由于我们也是在 `<keep-alive>` 标签内部写 DOM，所以可以先获取到它的默认插槽，然后再获取到它的`第一个子节点`。`<keep-alive>` 只处理第一个子元素，所以一般和它搭配使用的有 `component` 动态组件或者是 `router-view`。
接下来是 `cache`，逻辑很简单，如果命中缓存，则直接从缓存中拿 `vnode` 的组件实例，并且重新调整了 `key` 的顺序放在了最后一个；否则把 vnode 设置进缓存。

## 渲染
对于首次渲染而言，除了在 `<keep-alive>` 中建立缓存，并设置 `vnode.data.keepAlive = true`，其他的和普通组件渲染没什么区别。下面介绍更新过程

当数据发送变化，在 patch 的过程中会执行 patchVnode 的逻辑，它会对比新旧 vnode 节点，甚至对比它们的子节点去做更新逻辑，但是对于组件 vnode 而言，是没有 children 的，是通过 `patch -> patchVnode -> prepatch -> updateChildComponent` 这条路线更新。

````js
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

检测到组件内嵌有子节点通过 `resolveSlots` 更新 `$slots`，之所以后面还需要调用 `$forceUpdate`，是要让子组件在下次 `patch` 中 `diff` 更新。
下次执行 `<keep-alive>` 的 `render` 方法，这个时候如果它包裹的第一个组件 `vnode` 命中缓存，则直接返回缓存中的 vnode.componentInstance。接着又会执行 patch 过程，再次执行到 createComponent 方法

````js
function createComponent (vnode, insertedVnodeQueue, parentElm, refElm) {
  let i = vnode.data
  if (isDef(i)) {
    const isReactivated = isDef(vnode.componentInstance) && i.keepAlive
    if (isDef(i = i.hook) && isDef(i = i.init)) {
      i(vnode, false /* hydrating */)
    }
    // after calling the init hook, if the vnode is a child component
    // it should've created a child instance and mounted it. the child
    // component also has set the placeholder vnode's elm.
    // in that case we can just return the element and be done.
    if (isDef(vnode.componentInstance)) {
      initComponent(vnode, insertedVnodeQueue)
      insert(parentElm, vnode.elm, refElm)
      if (isTrue(isReactivated)) {
        reactivateComponent(vnode, insertedVnodeQueue, parentElm, refElm)
      }
      return true
    }
  }
}
````

再次调用 init 钩子函数的时候因为 `vnode.data.keepAlive` 为true， 所以不会再执行组件的 mount 过程了

`````js
const componentVNodeHooks = {
  init (vnode: VNodeWithData, hydrating: boolean): ?boolean {
    if (
      vnode.componentInstance &&
      !vnode.componentInstance._isDestroyed &&
      vnode.data.keepAlive
    ) {
      // kept-alive components, treat as a patch
      const mountedNode: any = vnode // work around flow
      componentVNodeHooks.prepatch(mountedNode, mountedNode)
    } else {
      const child = vnode.componentInstance = createComponentInstanceForVnode(
        vnode,
        activeInstance
      )
      child.$mount(hydrating ? vnode.elm : undefined, hydrating)
    }
  },
  // ...
}
`````

这个时候 `isReactivated` 为 true，`reactivateComponent` 是解决对 reactived 组件 transition 动画不触发的问题。最后通过执行 insert(parentElm, vnode.elm, refElm) 就把缓存的 DOM 对象直接插入到目标元素中，这样就完成了在数据更新的情况下的渲染过程。

## 生命周期
在渲染的最后一步，会执行 `invokeInsertHook(vnode, insertedVnodeQueue, isInitialPatch)` 函数执行 `vnode` 的 `insert` 钩子函数，它的定义在 `src/core/vdom/create-component.js`

!_isMounted(初始渲染) : `componentVNodeHooks.insert` -> `activateChildComponent(vm)` -> `activateChildComponent(vm.$children[i])` -> `callHook(vm, 'activated')`
_isMounted(更新) :  `componentVNodeHooks.insert` -> `queueActivatedComponent`  -> `back to flushSchedulerQueue` -> `callActivatedHooks(activatedQueue)` -> `callHook(vm, 'activated')`


