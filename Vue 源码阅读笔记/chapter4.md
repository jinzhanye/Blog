# update / 新旧节点相同
1. data 发生变化，触发其 set ，从而触发渲染 watcher 更新 
2. 渲染 watcher 更新过程中从 render 中获取新 vnode
3. patch 判断到旧 vnode 非真实元素(RealElement) 且 新旧 Vnode 相同，那么调用 patchVnode 处理
4. patchVnode 的功能是把新的 vnode patch 到 旧的 vnode 上， 调用 updateChildren 对 children 进行 dom diff。 updateChildren 又
会递归调用 patchVnode 比较 children vnode。
5. patchVnode 时，当更新的 vnode 是一个组件渲染 vnode 的时候，会执行 prepatch 的方法。
6. prepatch 就是将新 vnode 的属性(attrs、listeners、props)赋值给旧 vnode，触发这些属性()的 setter，然后更新子组件的 view (
注意对于新旧占位符vnode，updateChildren 是无法识别它们是否相同的，prepatch 也无法识别，每次更新它们都会执行，只能依靠 setter 对比新旧值。
)

# update / 新旧节点不相同
1. data 发生变化，触发其 set ，从而触发渲染 watcher 更新 
2. 渲染 watcher 更新过程中从 render 中获取新 vnode
3. patch 新旧 Vnode 不相同，调用 createElm 创建新真实DOM
4. 更新父占位符vnode 相关信息
5. 删除旧 DOM 和 vnode

## props update

_createElement\createComponent

````js
//data === {
//    attrs:{flag:false}
//}

// 通过 propsOptions的 props 名称 来 extract props from data
var propsData = extractPropsFromVNodeData(data, Ctor, tag);
````
