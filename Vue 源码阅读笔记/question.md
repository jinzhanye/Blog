- 为什么 patch.js/ patch 中用 oldVnode.nodeType 判断 isRealElement？ 
- 为此时渲染 vnode 的 nodeType 为 undefined ？
- vnode.nodeType 是在什么时候赋值的 ？

````js
var isRealElement = isDef(oldVnode.nodeType);
````

所有问题的答案在于，当 oldVnode 为一个真实DOM 时才会有 nodeType 属性。 nodeType 是 Document 对象的属性。 Vnode 对象是没有这个属性的。
所以可以用这个属性来区分真实 DOM 与 Vnode。

