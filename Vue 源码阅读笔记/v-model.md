## 原生 input

[官方文档](https://cn.vuejs.org/v2/guide/components.html#%E5%9C%A8%E7%BB%84%E4%BB%B6%E4%B8%8A%E4%BD%BF%E7%94%A8-v-model)有说明，`v-model` 实际上是结合 `v-bind:value`、`v-on:input` 的语法糖。 

````js
new Vue({
  el: '#app',
  template: `<div>
                <input v-model="message" placeholder="edit me">
                <p>Message is: {{ message }}</p>
          </div>`,
  data() {
    return {
      message: ''
    }
  }
});
````

上面这个例子，`Compile` 之后生成如下代码。可以看到 `domProps:{"value":(message)}` 与 `on:{"input": /*....*/}` 

````js
with(this) {
  return _c('div',[_c('input',{
    directives:[{
      name:"model",
      rawName:"v-model",
      value:(message),
      expression:"message"
    }],
    attrs:{"placeholder":"edit me"},
    domProps:{"value":(message)},
    on:{"input":function($event){
      if($event.target.composing)
        return;
      message=$event.target.value
    }}}),_c('p',[_v("Message is: "+_s(message))])
    ])
}
````

## 组件 

对于组件而言，也是一样。

````js
let Child = {
  template: '<div>'
  + '<input :value="value" @input="updateValue" placeholder="edit me">' +
  '</div>',
  props: ['value'],
  methods: {
    updateValue(e) {
      this.$emit('input', e.target.value)
    }
  }
}

let vm = new Vue({
  el: '#app',
  template: '<div>' +
  '<child v-model="message"></child>' +
  '<p>Message is: {{ message }}</p>' +
  '</div>',
  data() {
    return {
      message: ''
    }
  },
  components: {
    Child
  }
})
````

上面这个例子父组件 `Compile` 之后，生成代码如下  

````js
with(this){
  return _c('div',[_c('child',{
    model:{
      value:(message),
      callback:function ($$v) {
        message=$$v
      },
      expression:"message"
    },
  }),
  _c('p',[_v("Message is: "+_s(message))])],1)
}
````

创建子组件 vnode 阶段，会执行 `createComponent` 函数，它的定义在 `src/core/vdom/create-component.js`。期间调用 `transformModel` 处理 `v-model`，
`transformModel` 给 `data.props` 添加 `data.model.value`，并且给 `data.on` 添加 `data.model.callback`，结果如下 

````js
with(this){
  return _c('div',[_c('child',{
    model:{
      value:(message),
      callback:function ($$v) {
        message=$$v
      },
      expression:"message"
    },
    
    props: {
      value:(message),
    },
    on: {
      input: function ($$v) {
        message=$$v
      }
    }
  }),
  _c('p',[_v("Message is: "+_s(message))])],1)
}
````

`v-model` 最后被转成成了组件`vaule` 属性与 `on-input` 事件监听器
