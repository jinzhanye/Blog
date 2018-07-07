- 事件处理

将应用逻辑和事件处理的代码拆分开来

- 可以在依赖事件情况下，测试业务方法
- api参数更加明确 

````js
const MyApplication = {
    handleClick(event){
        event.preventDefault();
        event.stopPropagation();
        
        this.doSomeThing(event.x, event.y);
    },
    
    doSomeThing(x, y){}
}

addEventListener(element, 'click', ()=> {
    MyApplication.handleClick(event);
});
````

- 把条件分支语句提炼成函数

有助于增强代码可读性

````js
// 比如判断夏天时
if(data.getMonth() >=6 && date.getMonth() <= 9){}
// 替换成
if(isSummer()){}

// 如果判断函数时
if(typeof callback === 'function' && callback()){}
// 替换成
if(isFunction(callback) && callback()){}
````
