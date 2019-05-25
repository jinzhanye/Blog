生词

- flaws
- a couple more
- per se 拉丁语，可理解为英文的 itself
- 转译器（transpiler）
- this might be handy 方便

# 可替代的JSX
如今，JSX对于当前各种框架来说都是一个非常流行的模版选择，并非只是React(或受JSX启发的模版)。然而，如果你不喜欢使用JSX，或者你的项目想避免使用它，又或者只是受好奇心驱使，你想尝试如果不用JSX该怎么写React代码。最简单的答案是去阅读官方文档，然而官方文档有点短，以下有几个更多的选择。

> 免责声明:就我个人而言，我喜欢JSX，在我所有React项目中都使用到它。然而，我稍微研究了这个话题也愿意分享我的发现

## 什么是JSX

首先，我们需要了解JSX是什么，以便在纯JavaScript中编写匹配的代码。JSX是一门[特定领域](https://en.wikipedia.org/wiki/Domain-specific_language)的语言，这意味着我们需要使用JSX转变我们的代码以便在获得正规的Javscript代码，不然浏览器理解不了我们的代码。在光明的未来，如果你想在你的目标浏览器中使用 [模块化](https://developers.google.com/web/fundamentals/primers/modules) 和其他支持的所需功能，你依然没有能力完全抛弃代码转换，这将是个问题。

想要搞懂JSX最终被转换成什么，最好的办法也许是使用 [babel repl](https://babeljs.io/repl)。你需要点击 `presets`(应该在面板左侧)还有在那里选择 `react`，这样 babel 才能正确地完成转换。完成上面步骤后你会在右侧面板实时看到转换得到的JavaScript代码。例如，你可以试着输入下面代码

```jsx
class A extends React.Component {
    render() {
        return (
            <div className={"class"} onclick={some}>
                {"something"}
                <div>something else</div>
            </div>
        )
    }
}
```

我得到的代码如下所示

```js
class A extends React.Component {
  render() {
    return React.createElement("div", {
      className: "class",
      onclick: some
    }, "something", React.createElement("div", null, "something else"));
  }
}
```

你可以看到每对 `<%tag%>` 结构都被 [React.createElement](https://reactjs.org/docs/react-api.html#createelement) 这个函数代替。第一个参数是 react 组件 或者是内置的标签字符串(如 div、span)，第二个参数跟属性相关，剩余的参数都被看作是孩子节点。

我强烈建议你去尝试不同的树，去看看有什么不同。例如看看 React 是怎么渲染 `true` 或 `false`、数组、组件还有其他等等。即使你只用JSX还有用它把内容搞得漂亮点，这对于你来说也是很有帮助的。

> [官方文档](https://reactjs.org/docs/jsx-in-depth.html)提供有关JSX更深入的阅读

## 重命名
虽然生成的代码完全有效，并且我们可以用这种方式编写所有的React代码，但这种方法存在一些问题。

第一个问题是它非常冗长。真的很冗长，这是 React.createElement 造成的。所以第一个解决方案是将它先赋值到一个变量，通常叫做 `h` 表示 [hyperscript](https://github.com/hyperhype/hyperscript)。这将为你节省大量文本，代码也更具可读性。为了说明这一点，我们重写一下之前的例子

```js
const h = React.createElement;
class A extends React.Component {
  render() {
    return h(
      "div",
      {
        className: "class",
        onclick: some
      },
      "something",
      h("div", null, "something else")
    );
  }
}
```

如果你使用 `React.createElement` 或 `h` 进行一些操作，你会发现它有几个缺陷。首先，这个函数需要三个参数。因此如果没有属性，你还必须要提供 `null` 占位，而 `className` 这个可能最常用的属性，每次都要先写入一个对象再写这个属性。

你可以 [react-hyperscript](https://github.com/mlmorg/react-hyperscript) 作为代替方案。它并需要提供空属性作为参数，也允许你使用类似emmet的风格(`div#main.content` -\> `<div id="main" class="content">`)指定class和id。因此，我们的代码会有一点改善。 

## HTM
如果你不反对JSX，但不喜欢强制转译你的代码，这里有个叫  [htm](https://github.com/developit/htm) 的项目也许适合你。它的目标(看起来)是跟JSX做同样的事，但使用[模板语法](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)。这肯定会增加一些额外的开销（你必须在运行时解析这些模版），但在你的情况也许是值得的。

它的工作方式是包裹元素函数，在我们这里这个函数就是 `React.createElement`，但也可以是其他相似的API，然后返回一个函数，这个函数将会解析我们的模版然后准备返回跟 babel 一样的结果，但这些事仅仅在运行时执行。

如你所见，它与真正的JSX几乎相同，我们只需要以稍微不同的方式插入变量；但是，这些主要是细节，如果你想在没有任何工具设置的情况下展示如何使用React，这可能很方便。

## 类 Lisp 语法
这个想法跟hyperscript相似，但是，这是个值得一看的优雅方法。这也很多类似的帮忙库，所以我这是我的主观选择；这也许能给你的项目带来灵感。

[ijk](https://github.com/lukejacksonn/ijk) 将这种方法应用，你编写模版的时候只需要用到数组，将位置作为参数？。这样子的好处是你不用一味编写 `h` 函数（是的，即使这可能是重复的！）。下面是一个使用例子

## 总结 
本文没有建议你不要使用JSX，或者JSX是否一个好想法。你可能在想如果不用JSX你的代码能写成个什么样子，本文的目的只是回答这个问题。