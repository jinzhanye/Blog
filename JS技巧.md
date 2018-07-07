# 汇总一些JS常用小技巧
1. 监听滚动事件时，当滚动稳定下来时才触发callback，一般的写法会让callback执行频率变得非常高。

````
  //一般的写法
 window.addEventListener('scroll', function () {
            if (this.props.isLoading) {
                return;
            }
            callback();
        };
  //优化的写法
  window.addEventListener('scroll', function () {
            if (this.props.isLoading) {
                return;
            }
            //当发生连续滚动的，每一次滚动事件都会取消上一次滚动事件的setTimeout，
            //当滚动稳定下来，最后一次滚动的setTimeout不会被取消，所以callback在50ms后被触发。
            if (timeoutId) {
                clearTimeout(timeoutId);
            }
            timeoutId = setTimeout(callback, 50);
        };
````

2. frameElement减少repaint
3. if简写

````
window.$ === undefined && (window.$ = Zepto)
//相当于
//if(window.$){
//  window.$ = Zepto
// }
````
4. 