# Stencil: 为 Web Components 、 PWAs 而生的编译器

```bash
npm init stencil
```

Stencil 是一个简单的编译器，用于生成 Web Components 和 渐进式网页应用（PWA）。Stencil 是由 Ionic 框架团队为其下一代高性能移动和网页组件而打造的框架。

Stencil 将最流利的前端框架的最佳概念结合到编译时而不是运行时工具中。它有 TypeScript, JSX, 一点点虚拟 DOM 层, 高效的单向数据流绑定, 异步渲染管道（类似 React Fiber），和盒子外的懒加载，还有生成 100% 基于标准的同时能运行在现代流利器及旧浏览器支持到 IE11 的 Web Components。

Stencil Components 也只是 Web Components，所以他们 so they work in any major framework or with no framework at all。在大多数情况下，Stencil 可以作为有兼容性的传统前端框架的替代品，尽管这样使用显然不是必需的

Stencil 也提供很多在 Web Components 之上的关键能力，尤其是 SSR 不需要在 headless 浏览器上运行、预渲染、还有 对象作为属性（字符串代替）


注：Stencil 和 Ionic 是完全独立的项目。Stencil 没有规定说要指定哪个框架作为 UI 框架，但 Ionic 是 Stencil 的最大用户（今天！）

## 为什么使用 Stencil
Stencil 是一种流行创意的新方法：在浏览器打造流畅、功能丰富的应用。开发 Stencil 是为了利用浏览器原生新的可用的主要能力，比如自定义元素 v1，允许开发者用更少的代码创建更快的应用且兼容所有框架


Stencil 也是组织和库作者们在各种前端框架中构建可复用组件的解决方案，每个框架都有自己独立的组件系统。Stencil 组件可以与 Angular、React、Ember、Vue、JQuyer 搭配使用，或者不搭配任何框架使用，因为它们仅仅是纯 HTML 元素。

与直接使用自定义组件相比，在每个 Stencil 组件内部是一个高效的虚拟 DOM 渲染系统，JSX 渲染能力，异步渲染管道（像 React Fiber），等等。这使得 Stencil 组件更加高性能同时保持与纯自定义组件完全兼容。把 Stencil 看作预先制作的自定义元素，就像你自己编写这些功能一样

## 生词
next generation of performant mobile and desktop Web Components

下一代高性能移动和桌面Web组件。

prescribe v.规定

a diverse spectrum of 各种各样的

