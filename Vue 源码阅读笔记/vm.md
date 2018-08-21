vm._props 保存 props 的响应式对象
vm.$options.propsData 保存 props 非响应式对象
vm.$options._propKeys prosKey 数组

- VueComponent.superOptions 也就是 Vue.options
- VueComponent.extendOptions，CreateComponent 时传递的初始化 options
- VueComponent.options ，经 superOptions、extendOptions 合并而来的 options
