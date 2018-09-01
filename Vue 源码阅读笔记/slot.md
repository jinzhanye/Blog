## 普通插槽

````js
let AppLayout = {
  template: `<div class="container">
                <header><slot name="header"></slot></header>
                <main><slot>默认内容</slot></main>
                <footer><slot name="footer"></slot></footer>
              </div>`
};

let vm = new Vue({
  el: '#app',
  template: `<div>
                <app-layout>
                    <h1 slot="header">{{title}}</h1>
                    <p>{{msg}}</p>
                    <p slot="footer">{{desc}}</p>
                </app-layout>
              </div>`,
  data() {
    return {
      title: '我是标题',
      msg: '我是内容',
      desc: '其它信息'
    }
  },
  components: {
    AppLayout
  }
});
````

父组件模块 `parse` 之后，有 `slot` 属性的标签的 `AST节点` 会多出一个 `slotTarget`。例如 `h1` 标签的 `AST节点` 如下

````js
{
    type:1
    tag:"h1"
    attrsList:[]
    attrsMap:{...}
    parent:{...}
    children:[...]
    plain:false
    slotTarget:"\"header\""
    staticRoot:false
}
````

AST 经 `codegen` 阶段生成的代码如下：

````js
with(this){
  return _c('div',
    [_c('app-layout',
      [_c('h1',{attrs:{"slot":"header"},slot:"header"},
         [_v(_s(title))]),
       _c('p',[_v(_s(msg))]),
       _c('p',{attrs:{"slot":"footer"},slot:"footer"},
         [_v(_s(desc))]
         )
       ])
     ],
   1)}
````

`slot:"header"` 等，就是从 AST 的 `slotTarget:"\"header\""` 中取值的。


