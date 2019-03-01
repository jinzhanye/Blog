# 记一次 Vue 项目重构
随着公司项目越做越复杂，因前期团队对 Vue 使用经验不足，导致留下比较多坑。于是决定做一次重构。经过对项目分析，主要存在以下问题：

- 全局样式满天飞
- 组件越来越多，管理不方便
- 核心页面 1300 多行代码，阅读性非常差

> 本项目是一个金融类项目，核心功能是 xxx，vue 全家桶，element-ui 定制化，建模可视化使用 [mxGraph](https://github.com/jgraph/mxgraph)


## 减少全局样式
项目出现全局样式满天飞的情况，有以下原因

- 组件内样式想要覆写子组件样式，去除了 scoped 关键字
- 为了样式在不同组件间复用，将样式提到了全局

组件销毁后，Vue是不会删除对应样式标签的，所以组件内样式不写 scoped 存在污染全局样式的风险。

为了解决第一个问题，这次重构的做法是，坚决所有组件都使用 scoped。需要覆写子组件时使用[深度作用选择器](https://vue-loader.vuejs.org/zh/guide/scoped-css.html#%E5%AD%90%E7%BB%84%E4%BB%B6%E7%9A%84%E6%A0%B9%E5%85%83%E7%B4%A0)解决。这样仅不会污染全局样式，还对子组件覆写样式一目了然。

对于弹窗这类确实要作用到全局的样式，我们统一写在命名为 global.scss 的文件，并使用 BEM 规范命名。

对于在组件间复用的样式，分模块地放到 modules 文件夹下，组件内使用时再用 @import 导入。

来看看重构后的 style 文件夹长这个样子

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g0mf6gtjcij30ow0dgmyh.jpg)

全局样式样式只剩下 nomalize.css、一些自定义的 reset、element-ui 的默认样式、上文提到的 global，还有就是图标。


## 分类管理组件

未重构前，[全局基础组件](https://segmentfault.com/a/1190000014085613#articleHeader1)放置在 components/common 文件夹，业务组件与其他未归类的组件全放在 components 文件下，看起来非常混乱。

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g0mgepxsyuj30mo0qjdie.jpg)

经重构后，将组件分为五类: business、common、function、slot。同时在目录下提供 README.md 解释用途

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g0mgejsf21j30j50kjwga.jpg)

slot、function 与 common 一样，不同的是 common 使用频率非常高是[全局注册](https://segmentfault.com/a/1190000014085613#articleHeader1)的。而 slot、function 是局部注册使用的。slot 的特别之处在于，这类组件只提供一个样式外壳，无太多交互，能很好地被其他组件利用。

像下图所示其他 Panel 组件都可以复用 slot目录下的 Panel 组件。

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g0mglnh6msj30mh0l6n03.jpg)

我会在这两种情况下创建组件

- 可复用
- 不可复用，纯粹为了减少某个页面代码，使 template 结构更清晰。

	- 例如仅仅是传入 props 做数据展示
	- 又或者少交互无嵌套的组件

像下面 NodeDetai 页面正是上面提到的第二类组件

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g0mgoxuzg0j30hh0b0wf9.jpg)

## 拆分大文件
写过程，写到网上找项目但是找不到，所以自己想到了这样做 xxxx

画布页之所有这么多代码是因为，在编写方法过程中会大的方法拆分成小的方法，结果这些小的方法都堆在 methods，导致事件处理函数非常不显眼。重构目标删除 methods 对象中除页面初始化方法外的所有非事件处理方法。也就是说 methods 对象中的每个方法都应该对应一个 template 事件处理。

画布页共三个组件，左侧的元素面板、右侧的节点面板、右侧的线条面板。

- 那么问题来了拆分出来的小方法不放在 methods ，该放到什么地方？

根据我对画布页面代码分析，发现这个页面其实只对有三个东西进行操作

- 架构
- 节点
- 线条

按照这个思路独立出有三个 js 文件，将 this 当作参数传入到各自的模块，用来操作 vm 对象。

重构代码大概长这个样式

```
xxxx
```

- 精简 methods

为什么不分成三个 mixin ？
- 实现不了面向接口编程

***************************

由于 mxGraph 事件上没有区分节点与线条，
程序不知道现在删除的是节点还是线条，代码中多了很多这样的判断

```js
functoin syncRemove(cell) {
	// 判断是节点还是线条
	const cellIsVertex = cell.vertex;
	if(cellIsVertex){
		// 执行删除节点
	} else {
		// 执行删除线条
	}
}
```

解决方法

使用面向接口编程


```js
// vertexOp.js
const vertexOp = {
  // *********
  // Interface
  // *********
  handleActive(vertex) {},
  async syncAdd(vertex) {},
  syncRemove(vertex) {},
  // Others ....
}
  
  
// edgeOp.js
const edgeOp = {
  // *********
  // Interface
  // *********
  handleActive(edge) {},
  async syncAdd(edge) {},
  syncRemove(edge) {},
  // Others ....
}  

// Canvas.vue 
let opContex = null;
let activeCell = null;

const listenSelectionChange = ()=> {
	activeCell = graph.getSelectionCell();
	const cellIsVertex = activeCell.vertex;
	if(cellIsVertex){
		opContex = vertexOp;
	} else {
		opContex = edgeOp;
	}
}

const handleRemoveEvent = ()=> {
	contexOp.syncRemove(activeCell);
}

```

***************************

### 将零碎的方法调用集中起来
需求：当用户做了任何改变架构外观的操作都将当前架构截图同步到服务端

旧做法：

- 添加节点，在相应处理方法最后加一句截图发请求
- 修改节点，在添加节点方法最后加一句截图发请求
- 移动节点，在添加节点方法最后加一句截图发请求
- 添加线条，在添加节点方法最后加一句截图发请求
- 修改线条，在添加节点方法最后加一句截图发请求
- ..........

重构后做法
// TODO 使得 event bus 代替 addCb xxCb

```js
// requestManager.js

class RequestManager {
  constructor() {
    this.updateRequests = [];
  }

  addReq(req) {
    if (req.config.method.toLowerCase() === 'get') {
      return;
    }
    this.updateRequests.push(req);
    this.addCb(req);
  }

  popReq({ name, response }) {
    if (response && response.config.method.toLowerCase() === 'get') {
      return;
    }
    const idx = this.updateRequests.findIndex(item => item.name === name);
    if (idx >= 0) {
      this.updateRequests.splice(idx, 1);
      this.popCb(name, response);
      if (this.updateRequests.length === 0) {
        this.emptyCb();
      }
    }
  }
}

export default new RequestManager();

// config/axios/index.js
export default function (...args) {
  const url = args[0];
  let data;
  let method;
  let name;
  if (args.length === 2) {
    method = args[1];
  } else if (args.length === 3) {
    if (_.isString(args[1]) && _.isString(args[2])) {
      method = args[1];
      name = args[2];
    } else {
      data = args[1];
      method = args[2];
    }
  } else if (args.length === 4) {
    data = args[1];
    method = args[2];
    name = args[3];
  } else {
    throw new Error('http support max 4 args');
  }

  if (_.isNil(name)) {
    name = String(Date.now());
  } else {
    name = `${name}__${Date.now()}`;
  }
  return $axios({ url, data, method }, name);
}

async function $axios(initialOptions, requestName) {
  const options = getOptions(initialOptions);
  initialOptions.requestName = requestName;
  //
  requestManager.addReq({
    name: requestName,
    config: initialOptions,
  });
	
  try {
    const response = await axios(options);
    // 
    requestManager.popReq({
      name: requestName,
      response,
    });
    return response.data;
  } catch (error) {
    //
    requestManager.popReq({
      name: requestName,
      error,
    });
    return {};
  }
}

// api/nodes.js
import http from '@/config/axios/index';

export default {
  all: data => http('/nodes', data, 'GET'),
  one: id => http(`/nodes/${id}`, 'GET'),
  save: data => http('/nodes', data, 'POST', 'nodes-save'),
  del: id => http(`/nodes/${id}`, 'DELETE', 'nodes-del'),
  // .....
};
```

最终只需要对请求进行拦截，就可以大量减少零散的方法调用

```js
const updateScreenshotReqs = ['nodes-save','nodes-del',/*......*/];

```

需求：当用户做了操作，实时提示用户操作保存中，保存完成后提示用户操作已保存

使用请求拦截非常轻松完成这个功能，只需要监听发送请求事件、请求队列清空事件即可。

## 总结

网上套路虽好，但 ctrlC、ctrlV 不是万能的。

最重要的是打好基础、对项目进行思考

此次重构一人完成，用时1星期