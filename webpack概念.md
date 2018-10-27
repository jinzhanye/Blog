- 本质上，webpack 是一个现代 JavaScript 应用程序的静态模块打包器(module bundler)。当 webpack 处理应用程序时，它会递归地构建一个依赖关系图(dependency graph)，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个 bundle。
- 与 gulp 区别，webpack是模块打包器，支持模块化、代码分离(也就是动态引入)、分局分析模块。 gulp 是任务处理器
- bundle、chunk、module
    bundle 最终打包出来的文件
    chunk 在webpack 进行模块依赖分析的时候，代码分割的出来的代码块，一般一个chunk 对应一个 bundle
    module 开发中的单个模块，export xxx 就是导出一个 module
- 插件(Plugin): 一个含有 apply 属性的 JavaScript 对象。该 apply 属性会在 webpack 编译时被调用，并能在整个编译生命周期访问。这些插件包通常以某种方式扩展编译功能。(比如定义一个全局环境变量)


webpack-dev-server 和 http 服务器如 nginx 有什么区别
webpack-dev-server 使用内存来存储webpack开发环境下的打包文件，并且可以使用模块热更新，比传统的http服务对开发更加简单高效

- webpack4 生产环境会自动对js 进行 tree shaking，但不会对 css 进行 tree shaking，css tree shaking 需要使用插件实现

- 原更新原理：返回页面时会附带一个 hot.js 脚本，该脚本与服务器建立 socket 通讯，客户端收到更新信息后发送script get 请求，获取新脚本更新代码


## 使用 Webpack 优化项目
对于 Webpack4，打包项目使用 production 模式，这样会自动开启代码压缩
使用 ES6 模块来开启 tree shaking，这个技术可以移除没有使用的代码
优化图片，对于小图可以使用 base64 的方式写入文件中
按照路由拆分代码，实现按需加载
给打包出来的文件以 hashchunk 命名，实现浏览器缓存文件(长缓存)
设置 optimization.splitChunks.chunks = 'all' 优化代码分割

