

````
child = {
    children:[], // scope-slot 不会作为children 保存
    tag:"child",
    type:1,
    scopedSlots:{
        "default":{
            attrsMap:{
               slot-scope: "props"
            }
            children:[VNode,VNode],
            slotScope :"props",
            tag:"template",
            type:1,
             //......
        }
        "slot2":{
            attrsMap:{
                slot:"slot2",
                slot-scope:"props2",
            }
            children:[VNode],
            slotTarget:""slot2"",
            slotScope :"props2",
            tag:"div",
            type:1,
        }        
        //......
    },
    // .....
}
````

````js
with (this) {
    return _c('div', [_c('child', {
        scopedSlots: _u([{
            key: "default",
            fn: function (props) {
                return [_c('p', [_v("Hello from parent")]), // template 节点返回 childrenList
                    _v(" "),
                    _c('p', [_v(_s(props.text + props.msg))])]
            }
        }, {
            key: "slot2",
            fn: function (props2) {
                return _c('div', {}, [_c('p', [_v(_s(props2.msg2))])]) // p 作为 div 的 children
            }
        }])
    })], 1)
}
````
