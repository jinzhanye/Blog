## 编译

parse 阶段，addHandler 做了 3 件事情

- 首先根据 modifier 修饰符对事件名 name 做处理，
- 接着根据 modifier.native 判断是一个纯原生事件还是普通事件，分别对应 el.nativeEvents 和 el.events，
- 最后按照 name 对事件做归类，并把回调函数的字符串保留到对应的事件中

````js
el.events = { // 组件事件
  select: {
    value: 'selectHandler'
  }
}

el.nativeEvents = { // 原生事件
  click: {
    value: 'clickHandler',
    modifiers: {
      prevent: true
    }
  }
}
````

codegen 阶段，会在 genData 函数中根据 AST 元素节点上的 events 和 nativeEvents 生成如下 data。会在 vnode 中用到 data

````js
{
  on: {"select": selectHandler},
  nativeOn: {"click": function($event) {
      $event.preventDefault();
      return clickHandler($event)
    }
  }
}
````

`on` 为组件事件，`nativeOn` 为原生事件，官方文档[深入 data 对象](https://cn.vuejs.org/v2/guide/render-function.html#%E6%B7%B1%E5%85%A5-data-%E5%AF%B9%E8%B1%A1)也有提及。

## render 阶段

```js
let target: any
function updateDOMListeners (oldVnode: VNodeWithData, vnode: VNodeWithData) {
  if (isUndef(oldVnode.data.on) && isUndef(vnode.data.on)) {
    return
  }
  const on = vnode.data.on || {}
  const oldOn = oldVnode.data.on || {}
  target = vnode.elm
  normalizeEvents(on)
  updateListeners(on, oldOn, add, remove, vnode.context)
  target = undefined
}
````

`vnode.data.on` 就是 codegen 阶段生成的 `data.on` 。`updateListeners` 遍历 on 去添加事件监听，遍历 oldOn 去移除事件监听

````js
export function updateListeners (
  on: Object,
  oldOn: Object,
  add: Function,
  remove: Function,
  vm: Component
) {
  let name, def, cur, old, event
  for (name in on) {
    def = cur = on[name]
    old = oldOn[name]
    event = normalizeEvent(name)
    // ....
    if (isUndef(cur)) {
       //  警告提示 ...
    } else if (isUndef(old)) {
      // 第一次
      if (isUndef(cur.fns)) {
        cur = on[name] = createFnInvoker(cur)
      }
      add(event.name, cur, event.once, event.capture, event.passive, event.params)
    } else if (cur !== old) {
      // 以后，
      old.fns = cur
      on[name] = old
    }
  }
  // 添加新事件
  for (name in oldOn) {
    if (isUndef(on[name])) {
      event = normalizeEvent(name)
      remove(event.name, oldOn[name], event.capture)
    }
  }
}
````

下次进入时 `cur !== old` ，直接替换 `fns` ，因为 `createFnInvoker` 返回的 cur 里就有 `fns`

`event = normalizeEvent(name)` 这里的 `normalizeEvent` 就是对事件修饰符作处理，然后返回一个事件对象 

````js
const normalizeEvent = cached((name: string): {
  name: string,
  once: boolean,
  capture: boolean,
  passive: boolean,
  handler?: Function,
  params?: Array<any>
} => {
  const passive = name.charAt(0) === '&'
  name = passive ? name.slice(1) : name
  const once = name.charAt(0) === '~' // Prefixed last, checked first
  name = once ? name.slice(1) : name
  const capture = name.charAt(0) === '!'
  name = capture ? name.slice(1) : name
  return {
    name,
    once,
    capture,
    passive
  }
})
````

添加事件的 `add` 方法对 `handler` 调用 `withMacroTask` 进行异步包裹。

````js
function add (
  event: string,
  handler: Function,
  once: boolean,
  capture: boolean,
  passive: boolean
) {
  handler = withMacroTask(handler)
  if (once) handler = createOnceHandler(handler, event, capture)
  target.addEventListener(
    event,
    handler,
    supportsPassive
      ? { capture, passive }
      : capture
  )
}


export function nextTick (cb?: Function, ctx?: Object) {
  // ......
  if (!pending) {
    pending = true
    if (useMacroTask) {
      macroTimerFunc()
    } else {
      microTimerFunc()
    }
  }
  // ......
}
````

## 自定义事件
                  
                  
