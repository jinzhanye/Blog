## 是什么

### mina-loader 分析
从 test.js 的 basic 用例可以看出，解析 .mina 文件，输出 .js、.wxml、.json、.wxss  4 个文件

- mina 调用 loader.loadModule，然后？
- parser 将 .mina 文件转换成 'module.exports={config:{},template:'',script:'',style:''}'
- mina-json-file 解析 app.json/component.json 里的配置路径
- selector 调用 loader.loadModule，然后？


## 分析

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

mina-loader 的作用是将 `.mina` 文件分离成 `.json`、`.wxml`、`.js`、`wxss` 四种类型文件。对于 mina-load 内部做了什么我们不从源码入手，
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

可以看到输出了 6 个 module，我们主要看后面 3 个，以 `!` 号为分割格式化一下可以看到一些规律

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
```
// mina-loader/lib/index
module.exports = require('./loaders/mina')
```

```js
module.exports = function () {
  const done = this.async()

  const options = {}

  const url = loaderUtils.getRemainingRequest(this)
  const parsedUrl = `!!${parserLoaderPath}!${url}`

  const loadModule = helpers.loadModule.bind(this)

  const getLoaderOf = (type, options) => {}

  loadModule(parsedUrl).then(/*...*/)
```

loadModule 的作用

`selector-loader` 很容易理解，就是提取 xxx

```js
module.exports = function (rawSource) {
  this.cacheable()
  const cb = this.async()
  const { type } = loaderUtils.getOptions(this) || {}
  // !!parser.js!app.mina
  const url = `!!${parserLoaderPath}!${loaderUtils.getRemainingRequest(this)}`
  this.loadModule(url, (err, source) => {
    if (err) {
      return cb(err)
    }
    const parts = this.exec(source, url)
    cb(null, parts[type].content)
  })
}



	addModule(module, cacheGroup) {
		const identifier = module.identifier();
		// identifier 是 request 路径
		if(this._modules[identifier]) {
			return false;
		}
		
		this._modules[identifier] = module;
		if(this.cache) {
			this.cache[cacheName] = module;
		}
		this.modules.push(module);
		return true;
	}
```

## 其他
多次 loadModule 因为有缓存的，所以不用担心性能

