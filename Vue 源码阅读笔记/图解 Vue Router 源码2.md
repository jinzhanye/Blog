## RouterLink

`RouterLink 组件` 位于 `components/link.js`，该组件其实是一个 `a` 标签， 它为该 `a` 标签绑定了点击事件。当用户点击时触发 `handler`，`handler` 阻止点击 a 标签改变 url 的行为，在后续流程中计算路径后手动调用 pushState 改变 url。
然后调用 `router.push` 最终调用 `transitionTo` 进入上一节介绍的流程进行切换路由。 

````js
const handler = e => {
      if (guardEvent(e)) {
          // .......
          router.push(location);
          // .......
      }
}

function guardEvent (e) {
  // ........
  if (e.preventDefault) {
      // 阻止点击 a 标签改变 url 的行为，在后续流程中计算路径后手动调用 pushState 改变 url
      e.preventDefault()
  }
  return true
}

// history/hash.js
push (location: RawLocation, onComplete?: Function, onAbort?: Function) {
  this.transitionTo(location, route => {
    pushHash(route.fullPath)
    // .......
  }, onAbort)
}
````

`confirmTransition` 执行 `this.cb && this.cb(route)` 调用  `app._route = route` 触发 `_route` 的 `setter` ，从而 `nofity` 视图更新。

````js
updateRoute (route: Route) {
    const prev = this.current
    this.current = route
    this.cb && this.cb(route)
    this.router.afterHooks.forEach(hook => {
      hook && hook(route, prev)
    })
}

history.listen(route => { // 设置 history.cb，在 updateRoute 的时候会执行这个 cb
  this.apps.forEach((app) => {
    app._route = route
  })
})

// history/base.js
listen (cb: Function) {
  this.cb = cb
}
````

注意，虽然 `this.current`(即`history.current`) 与 `app._route` 都是指向 `_route`，但是对 `this.current` 设值并不会触发 `setter`。因为是根 `vm` 将 `_route` 定义成响应式，`history` 对象没
没有把 `current` 定义为响应式。 

````js
// src/install.js
Vue.util.defineReactive(this, '_route', this._router.history.current)
```` 

## RouterView

上一篇提到，根 vm 在 `beforeCreate` 中执行完 `this._router.init` 后，会将 `_route` 变成响应式对象。

````js
Vue.mixin({
  beforeCreate () {
      // .....
      this._router.init(this)
      Vue.util.defineReactive(this, '_route', this._router.history.current)
      // .....
  },
})
````

`beforeCreate` 钩子执行完毕后，vue 往下执行到 `render` 函数，这个过程会渲染 `RouteView` 组件。`RouteView` 是一个函数式组件，下面是它的 `render` 函数。

````js
  render (_, { props, children, parent, data }) {
    // 这个属性非常重要，下面求 deep 值用到
    data.routerView = true
    // .......
    const h = parent.$createElement
    // 触发 _route 的 getter，install.js 中定义 Vue.util.defineReactive(this, '_route', this._router.history.current)
    const route = parent.$route
    const cache = parent._routerViewCache || (parent._routerViewCache = {})
    // .....
    
    let depth = 0
    while (parent && parent._routerRoot !== parent) {
      if (parent.$vnode && parent.$vnode.data.routerView) {
        depth++
      }
      // ......
      parent = parent.$parent
    }
    data.routerViewDepth = depth

    const matched = route.matched[depth] // create-matcher.js / createRoute 函数构造 route 时保存 matched 数组
    
    if (!matched) { // 找不到匹配的路由
      // ....
      return h() // 渲染空 Comment 节点
    }
    
    const component = cache[name] = matched.components[name]
    // .....
    return h(component, data, children)
  }
}
````

这个 `render` 函数主要做了以下事情

1. 访问 `parent.$route` 触发 `_route` 收集当前 `渲染 watcher` 到订阅队列。这是因为 `VueRouter` 在 `install` 过程中在 Vue 原型上定义 `$route` 指向 `this._routerRoot._route`。

    ````js
    Object.defineProperty(Vue.prototype, '$route', {
      get () { return this._routerRoot._route }
    })
    ````
    
2. 计算 `dep` 值，通过 `dep` 值在 `route.matched` 中匹配出相应的 `RouterRecord` 对象 
3. 调用 `parent.$createElement` 渲染 `routerRecord.component`
4. 在 `routerRecord.component` 渲染过程中如果遇到 `RouterView` 组件，以同样的步骤进行处理

## 总结 
- 根vm 在`beforeCreate`阶段，初始化路由，监听 url 变化。定义`_route` 为响应式对象。
- 根vm 在渲染阶段`RouterView` 触发`_route` 收集依赖。
- 以后每次用户点击 `RouterLink` 或者 直接改变 url 时，切换当前路由，对`_route` 设值，触发`RouterView` 更新视图。

## 参考
[Vue.js技术揭秘/Vue-Router](https://ustbhuangyi.github.io/vue-analysis/vue-router/)
