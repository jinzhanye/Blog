- 增加，删除类

````
//DOM4规范 IE10+支持
dom.classList.add('className');
dom.classList.remove('className');
//自定义addClass,removeClass
````
- 判断class是否存在

````
function hasClass(element, cls) {
    return (' ' + element.className + ' ').indexOf(' ' + cls + ' ') > -1;
}
````