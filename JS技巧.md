# 汇总一些JS常用小技巧

- 增加滚动事件监听执行callback的时间间隔，一般的写法会让callback执行频率变得非常高。
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
            if (timeoutId) {
                clearTimeout(timeoutId);
            }
            timeoutId = setTimeout(callback, 50);
        };
````
- frameElement减少repaint