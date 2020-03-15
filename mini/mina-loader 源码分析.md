# mina-loader 源码分析

团队最近需要对小程序进行工程化升级，在网上看了[《小程序工程化实践》](https://juejin.im/post/5d00aa5e5188255a57151c8a)一文收获满满，刚好文件提到的 [mina-webpack](https://github.com/tinajs/mina-webpack) 也是我们团队正在使用的工具。mina-webpack 是 [tinaJS](https://tina.js.org/#/?id=main) 配套的工程化工具。（有关 tinaJS源码在我的另一篇[文章](https://segmentfault.com/a/1190000021949561)有讲述）`mina-webpack` 包括好几个包，其中 `mina-runtime-webpack-plugin`、`mina-entry-webpack-plugin`、`mina-loader` 最为重要。《小程序工程化实践》已经讲解了 `mina-runtime-webpack-plugin`、`mina-entry-webpack-plugin` 的原理，本文主要讲解 `mina-loader` 原理。

## 分析
mina-loader 的作用是将 `.mina` 文件分离成 `.json`、`.wxml`、`.js`、`wxss` 四种类型文件。


```
// app.mina
<config>
{
  "usingComponents": {
    "foo": "/components/foo.mina"
  }
}
</config>

<template>
  <view>
    Hello World
  </view>
</template>

<script>
Page({
  onLoad () {},
})
</script>

<style>
view {
  background: blue;
}
</style>
```

对于 mina-loader 内部做了什么我们不从源码入手，
而是从 webpack 输出的信息进行的领导。进入 example 目录，执行 `npm run dev`，可以看到如下信息

```
     Asset       Size  Chunks             Chunk Names
./app.json   18 bytes          [emitted]  
./app.wxml   24 bytes          [emitted]  
./app.wxss   28 bytes          [emitted]  
    app.js  314 bytes       0  [emitted]  app.js
 common.js    5.76 kB       1  [emitted]  common.js


[0] ./app.mina 95 bytes {0} [built]
[1] ../node_modules/@tinajs/mina-loader/lib/loaders/parser.js!./app.mina 390 bytes [built] 
[2] ../node_modules/@tinajs/mina-loader/lib/loaders/selector.js?type=script!./app.mina 66 bytes {0} [built]
[3] ../node_modules/@tinajs/mina-loader/node_modules/file-loader/dist/cjs.js?name=./[name].json!../node_modules/@tinajs/mina-loader/lib/loaders/mina-json-file.js?{"publicPath":"/"}!../node_modules/@tinajs/mina-loader/lib/loaders/selector.js?type=config!./app.mina 56 bytes [built]
[4] ../node_modules/@tinajs/mina-loader/node_modules/file-loader/dist/cjs.js?name=./[name].wxml!../node_modules/wxml-loader/lib?{"publicPath":"/"}!../node_modules/@tinajs/mina-loader/lib/loaders/selector.js?type=template!./app.mina 56 bytes [built]
[5] ../node_modules/@tinajs/mina-loader/node_modules/file-loader/dist/cjs.js?name=./[name].wxss!../node_modules/extract-loader/lib/extractLoader.js?{"publicPath":"/"}!../node_modules/css-loader!../node_modules/@tinajs/mina-loader/lib/loaders/selector.js?type=style!./app.mina 56 bytes [built]
```

可以看到输出了 6 个 module，我们主要看后面 3 个，为什么是!？？。·以 `!` 号为分割格式化一下可以看到一些规律

```
[3]
../node_modules/@tinajs/mina-loader/node_modules/file-loader/dist/cjs.js?name=./[name].json!
../node_modules/@tinajs/mina-loader/lib/loaders/mina-json-file.js?{"publicPath":"/"}!
../node_modules/@tinajs/mina-loader/lib/loaders/selector.js?type=config!
./app.mina

[4]
../node_modules/@tinajs/mina-loader/node_modules/file-loader/dist/cjs.js?name=./[name].wxml!
../node_modules/wxml-loader/lib?{"publicPath":"/"}!
../node_modules/@tinajs/mina-loader/lib/loaders/selector.js?type=template!
./app.mina

[5]
../node_modules/@tinajs/mina-loader/node_modules/file-loader/dist/cjs.js?name=./[name].wxss!
../node_modules/extract-loader/lib/extractLoader.js?{"publicPath":"/"}!
../node_modules/css-loader!../node_modules/@tinajs/mina-loader/lib/loaders/selector.js?type=style!
./app.mina
```

按照 webpack 解析过程，上面信息的意思对于每个 module，webpack 读取 `app.mina` 文件然后从下往上依次用多个 loader 处理 `app.mina`。从上面三个 module 可以看出，第一个处理的 `loader`
都是 `selector-loader` ，最后一个 loader 都是 `file-loader`，也就是最后肯定有文件输出，输出的是 `[name].json`、`[name].wxml`、`[name].wxss` 这几个文件。再看看 `selecor-loader` 有个 `type`
参数，`type=style` 输出的是 `[name].wxss` 文件、`type=template` 输出的是 `[name].wxml` 文件、`type=style` 输出的是 `[name].wxss` 文件，由此可以推断出 `selector-loader` 的作用是按照 `type` 参数
对 `app.mina` 提取相关类型内容。最后就是第二个 loader，不同类型的内容由特定的 loader 进行处理。

到此我们大概了解 `mina-loader` 的工作流程，就是分别按类型提取 `app.mina` 的内容，然后将用特定的 loader 处理特定的内容，最后输出不同类型的文件。那么接下来我们瞅一下 `mina-loader` 的源码，看具体是怎么做的。

## 源码分析
先来看看整体的流程图，跟上面说的流程差不多

```
// todo 加图
```

```js
// mina-loader/lib/index
module.exports = require('./loaders/mina')

// mina-loader/lib/loaders/mina.js
module.exports = function () {
  const done = this.async()

  const options = {}

  const url = loaderUtils.getRemainingRequest(this)
  const parsedUrl = `!!${parserLoaderPath}!${url}`

  const loadModule = helpers.loadModule.bind(this)

  const getLoaderOf = (type, options) => {}

  loadModule(parsedUrl).then(/*...*/)
```

配合流程图看源码的话理解起来不困难。前文一直没提到 loadModule 是哪里来的，其实[loadModule](https://webpack.docschina.org/api/loaders/#this-loadmodule) 是 webpack loader 暴露给开发者使用的一个 api，用于在执行 loader 时也能去加载一个模块。然后再看看 selector-loader

## selector-loader

前文提到 `selector-loader` 是用来提取某一类型的文件内容

```js
module.exports = function (rawSource) {
  this.cacheable()
  const cb = this.async()
  const { type } = loaderUtils.getOptions(this) || {}
  // url = '!!parser.js!app.mina'
  const url = `!!${parserLoaderPath}!${loaderUtils.getRemainingRequest(this)}`
  this.loadModule(url, (err, source) => {
    if (err) {
      return cb(err)
    }
    const parts = this.exec(source, url)
    cb(null, parts[type].content)
  })
}
```

操作跟 `mina-loader` 很像，利用 `parser-loader` 将 `.mina` 文件转换成如下对象

```
parts = {
  "style": {
    "type": "style",
    "content": /*...*/,
  },
  "config": {
    "type": "config",
    "content": /*...*/,
  },
  "script": {
    "type": "script",
    "content": /*...*/,
  },
  "template": {
    "type": "template",
    "content": /*...*/,
  }
}

```

然后根据 `type` 返回对应的内容即可。但是这里有一个问题，我们看看下面代码

```
// mina-loader/lib/loaders/mina.js

// ...
const parsedUrl = `!!${parserLoaderPath}!${url}`
// ...
loadModule(parsedUrl)


// mina-loader/lib/loaders/parser.js
const url = `!!${parserLoaderPath}!${loaderUtils.getRemainingRequest(this)}`
this.loadModule(url,/*....*/)
```

可以看到 `loaderModule(parseredUrl)` 也就是 `loadModule('!!parser.js!app.mina')` 这个模块被重复加载了多次，这样的话会不会带来性能的损耗呢？答案是不会的，每次 `loadModule` 后 webpack 会将加载完的 module 以请求路径为 key 保存在 `Compilation` 对象。参考源码如下

```
// webpack/lib/Compilation.js
addModule(module, cacheGroup) {
	const identifier = module.identifier();
	// identifier 是 request 路径，在这个案例就是 'parser.js!app.mina'
	if(this._modules[identifier]) {
		return false;
	}
	// 缓存模块
	this._modules[identifier] = module;
	if(this.cache) {
	   this.cache[cacheName] = module;
	}
	this.modules.push(module);
	return true;
}
```

