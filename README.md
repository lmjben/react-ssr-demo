# react-ssr-demo

一个精简的 React 服务器端渲染 Demo。

## 为什么要做服务器端渲染？

目前的客户端渲染页面请求到的只是一个 html 壳，所有的内容都是通过 js 进行插入的。

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>ssr</title>
  </head>
  <body>
    <div id="root"></div>
    <script src="bundle.js"></script>
  </body>
</html>
```

正是因为页面是由 js 渲染出来的，所以会带来如下几个问题：

1、页面要等待 js 加载，并执行完成了才能展示，`在这期间页面展现的是白屏`。

2、爬虫不能识别 js 内容，所以抓取不到任何数据，`不利于 SEO 优化`。

为了解决这 2 个问题，我们可以使用服务器端渲染。

## React 服务器端渲染流程

之前说道，单页应用，请求到的是一个 html 壳，然后通过 js 去渲染页面。那么思考一下，如果请求到的直接是一个渲染好的页面，是不是就可以解决这个问题了呢？

没错，服务器端渲染就是这个原理。

### 简化流程

1、服务器端使用 renderToString 直接渲染出包含页面信息的`静态 html`。

2、客户端根据渲染出的静态 html 进行`二次渲染`，做一些绑定事件等操作。

> 服务器端没有 DOM，Window 等概念，所以只能渲染出字符串，不能进行事件绑定，样式渲染等。

## 路由问题

这里使用 react-router 对前后端代码进行同构。

1、客户端

使用 react-router-dom 下的 `BrowserRouter` 进行前端路由控制。

2、服务器端

使用 react-router-dom 下的 `StaticRouter` 进行静态路由控制，具体操作如下：

- 使用 react-router-config 下的 matchRoutes 匹配后端路由，使用 renderRoutes 渲染匹配到的路由。
- 使用 StaticRouter 中通过 context 可以和前端页面通信，传参。
- 使用 react-router-dom/server 下的 renderToString 方法，渲染出 html 字符串，并返回给前端。

## 状态管理

使用 redux 来存储数据，管理状态。

1、客户端

使用 redux 进行状态管理，react-redux 为组件注入 store。

2、服务器端

使用 redux 进行状态管理，但每一次请求需产生一个新的 store。

## 数据请求

1、客户端

使用 axios 在 componentDidMount 中请求数据。

2、服务器端

在后端匹配到路由的时候，进行数据请求，并把数据存入 redux 中的 store，然后渲染出包含数据的 html 页面。

- 在 routes 对象上挂载一个自定义方法 loadData。
- 在服务器端 matchRoutes 后，如果有 loadData，则进行请求数据，并把请求到的数据写入 store 中。
- 服务器端等待请求完成后，再进行 renderToString 渲染。

## 样式处理

1、客户端

使用 css-loader，style-loader 打包编写好的 css 代码并插入到页面中。

2、服务器端

由于 style-loader 会插入到页面，而服务器端并没有 document 等概念，所以这里使用 isomorphic-style-loader 打包 css 代码。

- 引入 isomorphic-style-loader 后，客户端就可以通过 styles.\_getCss()方法获取到 css 代码。
- 通过 StaticRouter 中的 context 把 css 代码传入到后端。
- 后端拼接好 css 代码，然后插入到 html 中，最后返回给前端。

## SEO 优化

SEO 主要是针对搜索引擎进行优化，爬取的是落地页，需要在服务器端做优化。

常规的 SEO 主要是优化：`文字`，`链接`，`多媒体`

- 内部链接尽量保持相关性
- 外部链接尽可能多
- 多媒体尽量丰富

由于网页上的文字，链接，图片等信息都是产品设计好的，技术层面不能实现优化。我们需要做的就是优化页面的 title，description 等，让爬虫爬到页面后能够展示的更加友好。

这里借助于 react-helmet 库，在服务期端进行 title，meta 等信息注入。

## 你可能不需要服务器端渲染？

现在，我们成功地通过服务器端渲染解决了`首次加载白屏时间`和 `SEO 优化`。但也带来了一些问题：

- 服务器端压力增大。
- 引入了 node 中间层，可维护性增大。

以上两个问题主要是钱的问题。服务器压力大，可以通过买更多的服务器来解决。可维护性增大，那就招募更多人来维护。但是对于小型团队来说，增加服务器，招募更多开发人员，会额外增加的支出，所以在选择服务器端渲染时，要权衡好利弊。

### 解决 SEO 的另一种方法

如果只是想优化 SEO，不妨使用预渲染来实现，推荐使用 prerender 库来实现。

prerender 库的原理：主要针对爬虫访问，`先请求客户端渲染的页面，把客户端渲染完成之后的结果，拿给爬虫看`，这样爬虫获取到的页面就是已经渲染好的页面。prerender 库会要求开启一个服务，通过传递 url 来解析客户端渲染页面，这就需要我们队服务器端架构进行调整。

> nginx 判断访问类型
>
> > 用户访问 ：直接走客户端渲染
> >
> > 爬虫访问 ：走预渲染

## 项目目录

> build (webpack 配置)
> dist (打包目录)
> src
>
> > client (客户端)
> > server (服务器端)
> > shared (共用)
> >
> > > components (展示组件)
> > > containers (容器组件)
> > > store (redux store)
> > > Routes.js (路由信息)
