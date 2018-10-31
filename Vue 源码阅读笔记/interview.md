- 双向绑定
- $mount、update
- virtual dom
### 生命周期
- 第一次页面加载会触发哪几个钩子？

答：会触发下面这几个beforeCreate、created、beforeMount、mounted 。

- created
初始化事件监听及初始化生命周期标识后触发 beforeCreated，依赖注入及响应式数据监听后触发created

- mounted

在执行 vm._render() 函数渲染 VNode 之前，执行了 beforeMount 钩子函数，在执行完 vm._update() 把 VNode patch 到真实 DOM 后，执行 mouted 钩子

- updated

在二次执行 vm._render() 函数渲染 VNode 之前，执行 beforeUpdated，在执行完 vm._update() 把 VNode patch 到真实 DOM 后，执行 updated 钩子

- destroyed

beforeDestroy 钩子函数的执行时机是在 $destroy 函数执行最开始的地方，接着执行了一系列的销毁动作，
包括从 parent 的 $children 中删掉自身，删除 watcher，当前渲染的 VNode 执行销毁钩子函数等，执行完毕后再调用 destroy 钩子函数

- activated

当引入keep-alive 的时候，页面第一次进入，钩子的触发顺序created-> mounted-> activated，切换不同component时的改deactivated。当再次进入（前进或者后退）时，触发activated。
退出路由时触发 destroyed

### vue路由的钩子函数
组件：beforeRouteEnter、beforeRouteUpdate、beforeRouteLeave
全局：beforeEach、beforeResolve、afterEach

router.beforeResolve 注册一个全局守卫。这和 router.beforeEach 类似，区别是在导航被确认之前，同时在所有组件内守卫和异步路由组件被解析之后，解析守卫就被调用

## 白话的 $mount 
- 配置项合并、响应式数据初始化
- 编译 template 转换成 render 函数
- 调用 render 函数返回一个 vnode
- path 调用 createElem 将vnode 转化成真实 DOM 然后插入父节点/容器
- createElem 调用 createChildren 递归以上过程

### nextTicket
- 优先使用 macroTask,第一次 nextTicket 先向 messageChannel 发送一个异步任务，后来的 nextTicket 把回调加入回调队列，在同步代码执行完毕后执行 messageChannel 的异步回调列队。

### 响应式原理
reflect.defineProperty(ES6) 返回 boolean,object.defineProperty返回一个新对象

- 递归遍历data对象,利用defineProperty监听数据变化
- 调用render时会触发数据的get方法，此时收集当前渲染watcher到sub队列
- 当数据发生变化时触发set方法，此时调用dep.notify方法遍历sub队列的watcher，调用他们的update方法异步(内部调用nextTick)更新视图

## vdom
- 核心 API: 

    - h 函数 创建 vdom
    - patch 函数，patch 分两种，初次渲染patch(container, vdom)、更新patch(oldVdom, newVdom)

- 什么是 vdom?

用 js 模拟 DOM 结构

- 为什么用 vdom?

DOM 操作是“昂贵”的，应该减少DOM 操作。
js运行效率高，将 DOM 对比操作放在 JS 层，提高性能

