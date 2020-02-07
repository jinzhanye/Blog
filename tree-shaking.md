# tree-shaking 前生今世
## 简介
- 是一种死代码删除技术(dead code elimination, DCE)
- 不是删除永远不能执行的代码，而是从入口点开始摇树，并且只包含保证执行的代码。 它被简洁地描述为“活代码包含”(live code inclusion)
- 历史：动态语言中的死码删除比静态语言中的难得多，1990 就被提出，思想是入口点建立调用树，未调用的代码可以被删除
- 为什么只有在前端火起来，我想是因为浏览器资源加载速度影响用户，如何资源体积变小是一件重要的事情。但 jar 包在服务器运行就不会有这样的困扰。android、iOS 也需要做这个处理
- 在 JavaScript 中 tree-shaking 的流行是基于这样一个事实: 与 CommonJS 模块不同，ECMAScript 6模块加载是静态的，因此通过静态解析语法树可以推导出整个依赖树。 因此，摇树变成了一个简单的问题。但是，tree-shaking 不仅适用于 import/export 语句级别，它还可以在语句级别工作，具体取决于实现。那问题来了，虽然 CommonJS 是动态加载模块的，但是上面不是说了“它还可以在语句级别工作，具体取决于实现”吗？
- 答案在 es6 module 可以进行编译时静态分析，详情见[ECMAScript 6 入门 - Module 的语法
](http://es6.ruanyifeng.com/#docs/module#%E6%A6%82%E8%BF%B0)
- 动态引入的模块是不会进行 tree-shaking 的
- 应用: dart2js、rollup、webpack、uglyfyjs

## tree-shaking
- 代码不会被执行
	- 未引用的，见 case1 的 baz 方法
	- 引用了却不使用，case1 的 qux 方法
	- 使用了但语句里有不可达代码，见 case2 的 foo 方法

- 代码执行的结果不会被用到，见 case2 的 baz 方法

## 副作用
进行 tree-shaking 时，有副作用的代码不会被删除，见 case2 的 baz 方法

## 使用第三方库开启 tree-shaking 注意单项
- sideEffects 设置成 false， 来告知 webpack，它可以安全地删除未用到的 export。详见 webpack 官方指南
- package.json 的 module 字段, jsnext:main 是早期的称呼，因为不够直观被 module 替代了
- sideEffects、module、jsnext都不是 package.json 的官方配置字段


- todo 添加 lodash-es 截图
- 添加 roullup pkg 截图

https://shuheikagawa.com/blog/2017/01/05/main-jsnext-main-and-module/

https://github.com/rollup/rollup/wiki/pkg.module/_history


- 出现了一些插件将 import 语法进行转换，但后来 rollup 的 jsnext 出现了，详见[你的Tree-Shaking并没什么卵用](https://segmentfault.com/a/1190000012794598)

## Scope Hoisting
这是 webpack 3 新增的功能，用于减少体积提升代码执行速度。这个功能的原理很简单：将所有模块的代码按照引用顺序放在一个函数作用域里，然后适当的重命名一些变量以防止变量名冲突。

```
//App.js
import { getEntry } from './utils'
console.log(getEntry());

//utils.js
import entry1 from './entry.js'
export function getEntry() {
  return entry1();
}

//entry.js
export default function entry1() {
  return 'entry1'
}
复制代码result: 简化后的代码如下
//摘录核心代码
function(e, t, r) {
  "use strict";
  r.r(t), console.log("entry1")
}
```

## 如何使用
- 开启 production 即可
- 还有就是上面提到的第三方库要按需引入

## 参考 
- [Tree shaking wiki](https://en.wikipedia.org/wiki/Tree_shaking)
- [Tree Shaking webpack](https://webpack.js.org/guides/tree-shaking/#src/components/Sidebar/Sidebar.jsx)
- [ECMAScript 6 入门 - Module 的语法
](http://es6.ruanyifeng.com/#docs/module#%E6%A6%82%E8%BF%B0)
- [你的Tree-Shaking并没什么卵用](https://segmentfault.com/a/1190000012794598)