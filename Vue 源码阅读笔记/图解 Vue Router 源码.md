## install
`Vue.use(Router)` 内部调用插件的 `install` 方法。 VueRouter 的 `install` 方法位于 `src/install.js`，`install` 主要做了以下事件

1. 标识 `install.installed = true`， 确保 Vue Router 只安装一次
1. 调用 `Vue.mixin` 把 `beforeCreate` 与 `destoryed` 钩子函数混入每个组件中。
1. 在 Vue 原型上定义 `$router` 指向 `this._routerRoot._router`，定义 `$route` 指向 `this._routerRoot._route`
1. 注册 `RouterView` 与 `RouterLink` 为全局组件
1. 定义路由中的钩子函数的合并策略，和普通的钩子函数一样

## new VueRouter
VueRouter 的构造函数位于 `src/index.js`。主要做了以下事件

1. 初始化全局钩子函数数组
1. `this.matcher = createMatcher(options.routes || [], this)` 生成 `matcher`，用于匹配路径
1. 根据 `mode` 参数生成相应 `history`。`history` 构造过程中在根 url 后补上 `/#/`，`history.router` 指向 VueRouter 实例
1. 定义 `init` 方法
1. 定义 `go`、`back`、`forward`、`push`、`replace`、`beforeEach`、`afterEach` 等方法

## createMatcher
`createMatcher` 是一个工厂函数，是构造 VueRouter 实际过程中重要的一步，`createMatcher` 定义在 `src/create-matcher.js`。主要做了以下事件

1. 调用 `createRouteMap(routes)`，获取 `pathList, pathMap, nameMap`
1. 定义 `_createRoute` 等函数
1. 返回 `Matcher` 对象，利用 `Matcher` 私有访问 `pathList, pathMap, nameMap`，调用 `_createRoute` 等

`Matcher` 对象定义如下

````js
export type Matcher = {
  match: (raw: RawLocation, current?: Route, redirectedFrom?: Location) => Route;
  addRoutes: (routes: Array<RouteConfig>) => void;
};
```` 

## createRouteMap
`createRouteMap` 定义在 `src/create-route-map`， `createMatcher` 调用 `createRouteMap` 时传入的 `routes` 参数就是开发者编写的路由配置。长这个样子

````js
const routes = [{
  path: '/foo/:id',
  component: Foo,
  name:'Foo',
}, {
  path: '/bar',
  component: Bar,
  name:'Bar',
  children: [{
    path: 'baz',
    component: Baz,
    name:'Baz'
  }]
}];
````

`createRouteMap` 遍历 `routes`，执行 `addRouteRecord` ，将 `route` 转换成 `routerRecord` 加入到 `pathList, pathMap, nameMap`。 `record` 长这个样子

````js
const record: RouteRecord = {
  path: normalizedPath,
  regex: compileRouteRegex(normalizedPath, pathToRegexpOptions),
  components: route.components || { default: route.component },
  instances: {}, // 对应组件的 vm 实例，组件 beforeCreate 阶段调用 view.js/registerRouteInstance 中获取
  name,
  parent,
  matchAs,
  redirect: route.redirect,
  beforeEnter: route.beforeEnter,
  meta: route.meta || {},
  props: route.props == null
    ? {}
    : route.components
      ? route.props
      : { default: route.props }
}
````

同时 `addRouteRecord` 执行过程中会检测到子路由会递归调用自身，将子路由也加入 `pathList, pathMap, nameMap`。`createRouteMap` 执行后得到这样的 `pathList, pathMap, nameMap`。

````js
pathList = ["/foo/:id","/bar/baz","/bar"];

nameMap = { // 以 name 字段作为 key
 Baz:{
     beforeEnter:undefined,
     components:{default: Foo},
     instances:{},
     matchAs:undefined,
     meta:{},
     name:"Baz",
     parent:BarRecordInstance,
     path:"/bar/baz",
     props:{},
     redirect:undefined,
     regex:/^\/foo\/((?:[^\/]+?))(?:\/(?=$))?$/i
 },
 Foo:{},// 与 Baz 结构相同
 Bar:{},
}

pathMap = {} // 与 nameMap 结构一样，不同的是以 path 字段作为 key
````

## beforeCreate
以上，VueRouter 实例构造完毕。接下来 `new Vue`，进入根 vm 实例的 `beforeCreate` 方法。`beforeCreate` 在 VueRouter 的 `install` 过程混入

````js
beforeCreate () {
    if (isDef(this.$options.router)) {// 根 vm 实例
      this._routerRoot = this // _routerRoot 也就是根 vm 实例
      this._router = this.$options.router // VueRouter 的实例 router
      this._router.init(this) // 初始化 router
      // 将 _route 变成响应式对象
      Vue.util.defineReactive(this, '_route', this._router.history.current)
    } else { // 非根 vm
      this._routerRoot = (this.$parent && this.$parent._routerRoot) || this // 指向根 vm 实例
    }
    registerInstance(this, this) // 将当前 vm 实例绑定到当前 match 到的 `routeRecord` 的 `instance` 属性
  },
```` 

## init
`_route.init` 过程比较复杂, `init` 方法主要做了如下事情



## transitionTo
`history.transitionTo`，这个方法的作用是切换当前路由，也就是切换 `history.current`。定义如下

````js
transitionTo (location: RawLocation, onComplete?: Function, onAbort?: Function){/*....*/}
````


1. 首先调用 `this.router.match` (在上面 `new VueRouter` 的小章有提到)。`match` 方法的作用是，根据新路由的 `path` 属性从 `pathMap` 中匹配出对应的 `routeRecord`，
然后调用 `_createRoute`

````js
for (let i = 0; i < pathList.length; i++) {
  const path = pathList[i]
  const record = pathMap[path]
  if (matchRoute(record.regex, location.path, location.params)) {
    return _createRoute(record, location, redirectedFrom)
  }
}
````

`_createRoute`  最终调用 `createRoute`，`createRoute` 是一个工厂函数，返回 `route` 对象。其中 `matched` 属性非常重要，这个属性保存从当前路由到根路由的所有 `routeRecord`。在
`RouterView` 渲染组件时会用到这个 `matched` 属性。

````js
// createRoute(record,.....)
const route: Route = {
  name: location.name || (record && record.name),
  meta: (record && record.meta) || {},
  path: location.path || '/',
  hash: location.hash || '',
  query,
  params: location.params || {},
  fullPath: getFullPath(location, stringifyQuery),
  matched: record ? formatMatch(record) : [] // 向上遍历parent，获取到根路径的所有 record
}

function formatMatch (record: ?RouteRecord): Array<RouteRecord> {
  const res = []
  while (record) {
    res.unshift(record)
    record = record.parent
  }
  return res
}
````

结合上面例子，`Baz` 路由的 `matced` 是这样的
 
````js
// routeRecord 数组
matced = [{path:'/bar',/*..其他属性..*/},{path:'/bar/baz',/*..其他属性..*/}];
````

注意，里面的对象的顺序是非常重要的，顺序代表的是路由的深度。`RouterView` 就是利用深度匹配出 `routeRecord`。
