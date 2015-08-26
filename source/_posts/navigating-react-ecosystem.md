title: 探索 React 生态圈
date: 2015-08-26 15:59:29
author: rechtar
tags:
- react
- graphql
- relay
categories:
- Frontend
- 中文

---
# 探索 React 生态圈

2004年，对于前端社区来说，是里程碑式的一年。Gmail 横空出世，它带来的基于前端渲染的原生应用级别的体验，相对于之前的服务端渲染网页可谓提升了一个时代，触动了用户的G点。自此，前端渲染的网站成为无数开发者追逐的方向。

为了更好地开发前端渲染的「原生级别的」网站，包括 Backbone 和 Angular 在内的一系列前端框架应运而生，并迅速获得了大规模的采用。但是很快地，新的性能和 SEO 问题也接踵而来。几经尝试后，Twitter 甚至从前端渲染重回服务器渲染，而 Strikingly 也面对过同样棘手的问题。

2014年，React 进入我们的视线。让人耳目一新的是，对于其他开源框架遇到的种种问题，React 都自信地给出了解答。几乎没有犹豫，我们开始使用 React 来重构 Strikingly。若干年后，当我们回望，也许会发现， 2014年也是前端社区里程碑式的一年。

# React 简介

React 究竟是什么？Facebook 把它简单低调地定义成一个「用来构建 UI 的 JavaScript 库」。这个定义也许会让我们联想到许多 JavaScript 模板语言（比如 Handlebars 和 Swig），或者早期的控件库（比如 YUI 和 Dojo），但是 React 所基于的几个核心概念使它与那些模板和控件库迥然不同。事实上这几个核心概念非常超前，已经给整个前端世界带来了冲击性的影响。它们包括：

* 组件和基于组件的设计流程
* 单向数据流动
* 虚拟 DOM 取代物理 DOM 作为操作对象
* 用 JSX 语法取代 HTML，在 JavaScript 里声明式地描述 UI

这几条简单的原则放在一起带来了大量的好处：

* 前端和后端都能够从 React 组件渲染页面，完全解决了 SEO 这样的长期困扰 JavaScript 单页应用的问题
* 我们可以简单直接地写前端测试而完全忘掉 DOM 依赖
* 组件的封装方式和单向数据流动能够极大地简化前端架构的理解难度

我们来看一个例子：

```
var HelloMessage = React.createClass({
  render: function() {
    return <div>Hello {this.props.name}</div>;
  }
});

React.render(<HelloMessage name="John" />, document.body);
```

这个 React 版的 Hello World 已经展现了 React 的一些核心特性。首先，HelloMessage 是一个 React 组件；创建 React 应用的时候我们总是以组件为出发点。每个组件的核心是一个 `render` 方法，在其中我们把这个组件的 *props* 和 *state* 拼装到一个最终要渲染的模板中，然后返回这个模板（确切地说这里是一个 UI 描述而不是传统意义上的模板）。这段代码里看起来像 HTML 一样的部分就是著名的 JSX 语法，它是在 React 中描述「模板」的最佳方式。

现在，以 `var` 开头的第一段里我们定义了一个叫 HelloMessage 的组件；下面的 `React.render` 这一行所做的，则是把这个组件渲染到 `document.body` 里——也就是我们实际的页面上。但是在使用 `<HelloMessage />` 的时候，我们做了另一件事：`name="John"`。看起来很像 HTML 中的元素属性，但是既然 JSX 不是 HTML，这个语法的作用是什么呢？实际上，这就是我们向 React 组件传入 props 的方式。回头看第一段，我们可以看到在组件的内部有对 `this.props.name` 的引用。这个 name 就是我们刚刚指定的 John！

看到这里，如果你熟悉 jQuery 的话也许在想，这与 `$(document.body).html('<div>Hello John</div>')` 有什么根本区别呢？

这就是虚拟 DOM 出场的地方了。我们像写 HTML 一样写 JSX，但是 JSX 并不会直接变成 HTML 和 DOM。在幕后，React 维护着一个虚拟 DOM，而实际上的被浏览器直接操作的「物理」DOM 只是这个虚拟 DOM 的投影。虚拟 DOM 不依赖于浏览器环境，它可以运行在任何 JavaScript 执行环境。这就让下面的代码成为可能：

```
var html = React.renderToString(<HelloMessage name="John" />);
res.send(html);
```

如果第二行有点眼熟，你没有猜错——这段代码发生在服务器端！是的，同样的 HelloMessage，我们不仅可以让 React 在前端渲染到页面，同样可以在后端直接渲染成 HTML 字符串，然后把它返回给前端。服务端预渲染就这么自然地发生了。

React 带来的革命性创新是前端世界过去几年最激动人心的变化。自从接触 React 以来，我们深信 React 会彻底改变客户端开发者（包括前端、iOS 和 Android）的开发体验。在下面的篇幅里，我们想从四个大的方向——目标平台（targets）、数据处理（data）、工具（tools）和新的挑战——分享一下 React 生态系统和社区的进展和未来趋势。

# 目标平台（Targets）

对于虚拟 DOM 的讨论，很多人会说速度快过于真正的 DOM。这样的讨论可以让人快速入门理解 React，但是真正写过 React 应用的人会明白速度并不是虚拟 DOM 的精髓。我们认为虚拟 DOM 的存在帮助我们做到了两件事。第一是申明式 UI。通过虚拟 DOM，UI 不再是一个不断被更变的 DOM，你只要申明 UI 是怎么生成的，React 会自动帮你把 UI 的改变渲染到真正的 DOM 上。这种新的思维方式让你可以不用手动操作真正的 DOM。第二是多 target。我们一直在讲 web，但 React 让我们做到 web 以外的 target。虚拟 DOM 更像是 UI 虚拟机，自动帮你映射到真正的实现上，可以是浏览器 DOM 、iOS UI、Android UI。甚至有人做到了 React 映射到终端文本 UI。

多 targets 是 React 社区常常在讨论的主要话题之一。多 targets 的根本是**提高开发者体验**。开发者体验（DX, developer experience）是在 React 社区里屡次被提起的概念。如何在保持一样的用户体验下，提高开发者体验，是包括 React 在内的前端社区正在思考的问题。事实上任何一家有多客户端的公司都面临着这样同一个问题：在各种客户端语言里重新造轮子。开发者需要学习新的语言、写和维护类似的功能。提升客户端开发者体验就是减少学习成本和维护成本。这就是 React 提倡的 learn once, write everywhere。

最近也有一些鼓舞人心的消息。Facebook 内部 Ads Manager iOS 版本由 7 位前端工程师用 React Native 花了 5 个月完成。而 Android 版本，是同一班人，3个月内完成。代码重用率达到了 87%。

多 targets 也可以是在单个平台更深度的结合。来自 React 核心团队的 Sebastian Markbåge 在 ReactEurope 大会上给了一个让人目瞪口呆的演讲《[DOM as a Second-class Citizen](https://www.youtube.com/watch?v=Zemce4Y1Y-A)》。演讲中他畅想 React 直接输出到浏览器架构的底层。

{% asset_img browser.png [] %}
上图这是浏览器的渲染架构

{% asset_img deepBrowser.png [] %}
上图是 Sebastian Markbåge 认为 React 可以做的事情。

姑且不谈该不该这么做，通过虚拟 DOM 打开了这样的机会就已经让我们兴奋不已了。也说明了 Facebook 在设计 React 时已经考虑到超越 DOM。想法确实很超前！

## Pre-rendering （服务端预渲染）

对于其他主流前端框架，页面 SEO 和首次打开速度的问题都很让人头疼。 Twitter 当年因为首次打开速度过于慢甚至重回服务器渲染方案。一直以来人们一直在寻找一种只需要编写一次 UI 组件，前后端同时都能渲染的方案。如果能做到的话，我们就可以在首次打开页面时先用服务端渲染页面 HTML，当浏览器收到后已经可以显示页面。这样 SEO 和首次打开速度都能被解决。这种完美方案社区里称之为 [Isomorphic / Universal App](http://nerds.airbnb.com/isomorphic-javascript-future-web-apps/)。

React 原生支持了 Pre-rendering （服务端预渲染）就可以做到。由于有虚拟 DOM，也就意味着我们只需要后端运行 JavaScript 引擎就能渲染整个 DOM。目前主流后端语言都可以运行 V8 JavaScript 引擎。比如 Strikingly 的后端使用 Ruby on Rails，只需要使用开源的 [react-rails gem](https://github.com/reactjs/react-rails) 就可以在 Rails 后端渲染前端 React 组件。

使用服务端渲染时要注意 *window* 和 *document* 这些浏览器才有的全局变量是不存在的。React 组件提供这两个 lifecycle hook*：componentDidMount* 和 *componentDidUpdate* 在服务器不会被运行，只有在前端才会运行。使用服务器渲染时如果要使用任何浏览器才有的变量需要把代码放到这两个 lifecycle hook 定义里。

# 数据处理（Data）

React 定义自己为 MVC 中的 View。这让前端开发者从 V 开始去思考 UI 设计。但现在针对数据操作和获取方式，社区里还没有一种公认的方法。这也是写任何 React 应用时最难处理的地方。

## Flux

对于 M 和 V，Facebook 提出了 Flux 的概念。Flux 是一个专门为 React 设计的应用程序架构：应用程序由 dispatcher、store 和 view 组成，其中的 view 就是我们的 React 组件。Flux 的核心是如下图所示的单向数据流动：

{% asset_img flux.png [] %}

应用程序中的任何一次数据变化都作为 action 发起，经过 dispatcher 分发出去，被相关的 store 接收到并整合，然后作为 props 和 state 提供给 view（React 组件）。当用户在 view 上做了任何与数据相关的交互，view 会发起新的 action，开启一次新的数据变化周期。这种单向性使 Flux 在高层次上比传统 MVC 架构和以 Angular 和 Knockout 为代表的双向数据绑定容易理解的多，大大简化了开发者的思考和 debug 过程。

在 Facebook 把 Flux 作为一种设计模式（而不是已经做好的框架）宣布之后，几乎每个月出现一新的 Flux 库，他们都有各自的特色，有的对服务器渲染支持比较好，有的运用了更多函数式编程的概念。很多 Flux 库更像是实验，这有助于 React 生态的生长，但不可否认的是，未来会有大量 Flux 库慢慢死去，而只有少数会存留下来或进行合并。

## **GraphQL**

在构建大型前端应用时，前端和后端工程师通过 API 的方式进行合作。API 也是双方的协议。现在主流的方式是 RESTful API，然而在实践中，我们发现 RESTful 在一些真实生产环境的需求下不是很适用。往往我们需要构建自定义 endpoint，违背了 RESTful 的设计理念。

举个例子，我们想要显示论坛帖子、作者和对应的留言。我们分别要发出三个不同的请求。第二个请求依赖第一个请求结果返回的 *user_id*，前端需要写代码协调请求之间的依赖。分别发出三个不同请求在移动端这种网络不稳定的环境下效果很不理想。

```
GET /v1/posts/1
{
    "id": 1,
    "title": "React.js in Strikingly",
    "user_id": 2
}
```

```
GET /v1/users/2
{
    "id": 2,
    "name": "dfguo"     
}
```

```
GET /v1/posts/1/comments
[{
    "id": 6,
    "name": "rechtar",
    "comment": "Thanks for sharing! I would love to see some examples on GraphQL."
},{
    "id": 9,
    "name": "tengbao",
    "comment": "I heard that you guys also use immutable.js. How did it help?"
},{
    "id": 12,
    "name": "syjstc",
    "comment": "Impressive work! Thanks guys!"
},{
    "id": 18,
    "name": "abeth86",
    "comment": "Thanks for the sharing!"
}]
```

为解决这类问题，工程师会自定义一些 endpoint。对于这个例子，我们可以建立一个 /feeds 的 endpoint，集合了所有前端需要的结果：

```
GET /v1/feeds/1
{
    "id": 1,
    "title": "React.js in Strikingly",
    "user": {
        "id": 2,
        "name": "dfguo"
    },
    "comments":[
        {
            "id": 6,
            "name": "rechtar",
            "comment": "Thanks for sharing! I would love to see some examples on GraphQL."
        }...
    ]
}
```

但是我们在某些场景上可能只需要 *post* 和 *user*，不想要 *comments*。这时难道要再定义一个 *feeds_without_comments *的 endpoint？随着需求的改变，自定义 endpoint 的方法往往使得 API 接口变得累赘，违背了 RESTful 的设计理念。而任何前端工程师需要的数据一旦要改变都需要后端工程师的配合，这降低了产品的迭代速度。

来自 Facebook 的 GraphQL 是我认为目前最接近完美的解决方法。后端工程师只需要定义可以被查询的 Type System，前端工程师就可以使用 GraphQL  自定义查询。GraphQL 查询语句只需要形容需要返回的数据形状： 

```
{
    post(id: 1) {
        id,
        title,
        user {
            id,
            name
        },
        comments {
            id，
            name,
            comment
        }        
    }
}
```

GraphQL 服务器就会返回正确的 JSON 格式：

```
{
    "id": 1,
    "title": "React.js in Strikingly",
    "user": {
        "id": 2,
        "name": "dfguo"
    },
    "comments":[
        {
            "id": 6,
            "name": "rechtar",
            "comment": "Thanks for sharing! I would love to see some examples on GraphQL."
        }...
    ]
}
```

GraphQL 也原生支持了 API 版本控制，让你可以同时共存多个版本的客户端（包括 web 和 mobile）。这些都会减少客户端工程师和后端工程师的耦合度，提高生产力。

今年7月刚推出了 GraphQL 的规范 并开源了 JavaScript GraphQL 库。然而要让 GraphQL 成为主流，Facebook 需要打造一个像 React 这样的生态系统。要想在你自己的应用上用 GraphQL 还必须要有后端语言提供 GraphQL 库的支持。比如 Strikingly 需要 GraphQL Ruby 库。这不仅仅需要前端工程师。我们认为这将会比 React 生态系统更难建立。Facebook 需要整个社区的参与才能达到。

{% asset_img graphql.png [] %}

## **Relay**

Relay 是 Facebook 提出的在 React 上应用 GraphQL 的方案。React 的基础单位是组件（component），构建大型应用就是组合和嵌套组件。以组件为单位的设计模式是目前社区里最认可的，这也是前端世界的趋势之一。每个组件需要的数据也应该在组件内部定义。Relay 让组件可以自定义其所需要 GraphQL 数据格式，在组件实例化的时候再去 GraphQL 服务器获取数据。Relay 也会自动构建嵌套组件的 GraphQL 查询，这样多个嵌套的组件只需要发一次请求。Relay 将会在8月份开源。

## **Immutability**

React 社区接受了很多函数式编程的想法，其中受 Clojure 影响很深。对 immutable 数据的使用就是来自 Clojure 社区。当年 Om，这个用 ClojureScript 写的 React wrapper 在速度上居然完虐原生 JavaScript 版本的 React。这让整个社区都震惊了。其中一个原因就是 ClojureScript 使用了 immutable 数据。React 社区里也冒出了 immutable.js，这让 JavaScript 里也能用起 immutable 数据，完美弥补了JavaScript 做负责数据对象比较的先天性不足。immutable.js 也成为了构建大型 React 应用的必备。甚至有在讨论是否把 immutable.js 直接纳入 javascript 语言中。我们认为小型应用不会遇到虚拟 DOM 的性能瓶颈，引入 immutable.js 只会让数据操作很累赘。


# 工具（Tools）

工欲善其事，必先利其器。React 的火爆得力于来自社区的工具，而 React 也推动了这些工具的进步。这里我们想介绍几个 React 社区里比较受欢迎的工具。

## Webpack

{% asset_img webpack.png [] %}

在 React 里，由于需要用到 JSX，使用 webpack 或 browserify 这类工具编译代码已经渐渐成为前端工程师工作流程的一部分。webpack 是一款强大的前端模块管理和打包工具。这里列出他的一些特性：

1. 同时支持[ CommonJS](http://wiki.commonjs.org/wiki/Modules/1.1) 和[ AMD](https://github.com/amdjs/amdjs-api/wiki/AMD) 模块
2. 灵活和可扩展的 loader （加载器）机制，例如提供对 JSX、ES6、less 的支持
3. 支持对 CSS，图片等其他资源进行打包
4. 可以基于配置和智能分析打包成多个文件
5. 内置强大的 Code splitting 功能可以拆分并动态加载包
6. 开发模式支持 hot module replacement 模式，提高开发效率

## Babel

{% asset_img babel.png [] %}

ECMAScript 6 (ES6) 规范在今年四月刚敲定，React 社区基本全面拥抱 ES6。但目前还有很多浏览器不支持 ES6。使用像 webpack 这样的工具编译代码使得我们可以在开发时使用 ES6（或者更新版本），在上线前编译成 ES5。编译工具中最引人注意的是 Babel。前身为 ES6to5，Babel 是目前社区最火的 ES6 编译到 ES5 的代码工具，Facebook 团队甚至已经决定转用 Babel 而不再维护之前内部使用的 jstranform。通过 loader 机制，webpack 可以非常简易地和 Babel 结合应用。

## [React-hot-reload](https://github.com/gaearon/react-hot-loader)

在开发任何大型前端应用过程中，我们常常会因为一些小错误就需要重新刷新整个页面。React-hot-reload 尝试解决这个问题，提高开发效率。他使用了 webpack 的 hot module replacement 功能，动态替换 React 组件的 lifecycle hook 定义，不用刷新页面也可以更新代码变化。

## React Developer Tool

这款 Facebook 官方推出的 Chrome 插件可以让你方便地在浏览器中直接查看 React 的组件结构。安装后，在 Chrome 开发者工具中会多出一个 React tab。界面就像 DOM inspector 一样，只不过是看 React 组件结构关系。是开发 React 应用不可多得的工具之一 。

# Challenges

React 正在快速开拓着它的疆界，这意味在获得新的喜悦的同时，我们也面临着许多新的挑战。现在围绕着几个大的议题 React 社区仍没有达成定论，每周甚至每天都有新的实验项目在尝试这些问题的解决。

## 动画

一直一来大家都对动画应该在 React 里怎么表达为状态感到困惑。Cheng Lou 的 [react-tween-state](https://github.com/chenglou/react-tween-state) 是我们认为最符合 React 思维的做法。把位移存在 state 里，然后通过 javascript 动态渲染新的位置。不过大家对该做法是否能达到满意的速度一直持有保留态度。在今年 ReactEurope 的演讲中，他为我们展现出了出色的效果和速度，非常值得一看。

在 Strikingly 我们对于动画则采取了比较实用主义的处理方式：我们定义了一些容器组件，比如 `<JQFade />` 和 `<JQSlide />`，在其中调用 jQuery 的动画方法来实现相应的 transition。这种方式在理论上并不完全符合 React 的精神，不过但现在为止还是能够满足我们的需求。

## Flux 库与 Relay

正如上文已经提到过的，目前 Flux 的各种实现可谓是百花齐放，其中还并没有出现一个具有权威性的事实标准。Relay 同样也是刚刚孵化不久的新生概念——所有这些意味着虽然 Flux + Relay 会带来生产力的飞升，要实际用上它们我们还要待以时日。

## CSS

CSS 是一个有趣的话题：似乎所有人都觉得当前的 CSS 有深刻的缺陷，但是对于怎么解决这些缺陷大家的意见却分成了两派各不相让：一派认为 CSS「可以被修好」，并且致力于修好它，由此诞生了 [cssnext](http://cssnext.io/) 这样的项目；另一派认为 CSS 从根本上作为诞生于一个古老时代的东西，已经不能适应大规模、组件化的现代开发流程，这一思想集中反映在 Christopher Chedeau 的演讲 [React: CSS in JS](http://blog.vjeux.com/2014/javascript/react-css-in-js-nationjs.html) 中；在其中他提出了 CSS 的七个根本问题，然后指出在 JavaScript 中直接使用 inline CSS 可以几乎「免费」地解决所有这些问题。在传统的 web 开发最佳实践中 inline CSS 一直是被压制的反面实践，现在我们却能够以一个全新的视角看待它，这也完美地例证了 React 真的是在给整个前端世界带来根本性的推动。

# 总结

在不久前的 JSConf 2015 上赫门提出了前端的摩尔定理：前端每18月会难一倍。前端之所以变化这么快，是因为我们现在面临着前所未有的工程化挑战。今天的前端复杂度跟几年前完全不是一个等级。这也促使着社区要找到在这种复杂度下能保持开发效率和开发体验的工具和设计模式。React 社区从其他领域（游戏渲染、ClojureScript、函数式编程）偷师学艺，结合前端面临的独特问题，提出了一系列解决方案。 React 社区在各方面都推动着前端社区往前进。这对整个社区都是好事。我们也希望前端各个框架可以互相学习，共同推动整个社区的发展。

It's an exciting time to be a front-end developer.
