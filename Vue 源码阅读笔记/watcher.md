- vm._watcher 保存着该实例的渲染 watcher，在 `$forceUpdate` 用到 

````js
export function updateChildComponent (){
 //.....   
 if (hasChildren) {
     vm.$slots = resolveSlots(renderChildren, parentVnode.context)
     vm.$forceUpdate()
   }
 //.....      
}


// lifecycle.js
 Vue.prototype.$forceUpdate = function () {
    const vm: Component = this
    if (vm._watcher) {
      vm._watcher.update()
    }
  }
````


vm._watchers 保存着该实例所有 watcher
vm._computedWatchers 保存着所有 computed watcher 实例

- 无论是函数形式的 user watcher ，还是对象形式的 user watcher，最终都会转换成调用 Vue.prototype.$watch，之后$watch 会调用 new Watcher
- Watcher 起到两个作用
    1. 一个是初始化的时候会执行 this.getter
    1. 另一个是当 vm 实例中的监测的数据发生变化的时候执行 this.getter
    1. this.cb 什么时候触发 run -> getAndInvoke
    
````js
  this.deps = [];
  this.newDeps = [];
  this.depIds = new _Set();
  this.newDepIds = new _Set();
  
  this.dep
````    


`````js
// Watcher 构造函数
 if (this.computed) {
      this.value = undefined
      this.dep = new Dep()
    }
    
depend () {
  if (this.dep && Dep.target) {
    this.dep.depend()
  }
}
`````
触发 computed watcher 时， Dep.target 是渲染 watcher，所以 this.dep.depend() 相当于渲染 watcher 订阅了这个 computed watcher 的变化。

user watcher的user字段为true


## Watcher
- computed watcher computed 字段为 true Vue.extend 会 initComputed 初始化 computed watcher
- render watcher
- user watcher

expOrFn: computed、render watcher 时 expOrFn 为函数 ，user 时为字符串
````js
if (typeof expOrFn === 'function') {
    this.getter = expOrFn;
  } else {
    // xxxxx
  }
````

只有 user watcher 的 user 字段为true ，其他两个 watcher 的 user 字段为false

三种 watcher 的 getter 方法

- user watcher

````js
export function parsePath (path: string): any {
  if (bailRE.test(path)) {
    return
  }
  const segments = path.split('.')
  return function (obj) {// obj 为 vm
    for (let i = 0; i < segments.length; i++) {
      if (!obj) return
      obj = obj[segments[i]] // Dep.target 是当前userWatcher，所以当访问vm.data里的数据时会将当前userWatcher收集为依赖
    }
    return obj
  }
}
````

- computed watcher

- render watcher
updateComponent

dep 字段是 computed 字段专有的，其他两个 watcher 只有 deps、newDeps 数组

nextTick 会执行 flushSchedulerQueue，遍历所有 watcher ,执行它们的 run 方法
````js
Watcher.prototype.run = function run () {
  if (this.active) {
    this.getAndInvoke(this.cb);
  }
};
````

get 方法很关键

````js
/**
 * Evaluate the getter, and re-collect dependencies.
 */
Watcher.prototype.get = function get () {
  pushTarget(this);
  var value;
  var vm = this.vm;
  try {
    value = this.getter.call(vm, vm);
  } catch (e) {
    if (this.user) {
      handleError(e, vm, ("getter for watcher \"" + (this.expression) + "\""));
    } else {
      throw e
    }
  } finally {
    // "touch" every property so they are all tracked as
    // dependencies for deep watching
    if (this.deep) {
      traverse(value);
    }
    popTarget();
    this.cleanupDeps();
  }
  return value
};
````


watcher options
````
  this.deep = !!options.deep
  this.user = !!options.user
  this.computed = !!options.computed
  this.sync = !!options.sync
````
