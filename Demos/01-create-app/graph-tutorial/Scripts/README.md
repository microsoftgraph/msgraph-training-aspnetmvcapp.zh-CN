<!-- IGNORE THE HTML BLOCK BELOW, THE INTERESTING PART IS AFTER IT -->

<h1 align="center">Popper</h1>

<p align="center">
    <strong>用于在 web 应用程序中定位 poppers 的库。</strong>
</p>

<p align="center">
    <img src="http://badge-size.now.sh/https://unpkg.com/popper.js/dist/popper.min.js?compression=brotli" alt="Stable Release Size"/>
  <img src="http://badge-size.now.sh/https://unpkg.com/popper.js/dist/popper.min.js?compression=gzip" alt="Stable Release Size"/>
    <a href="https://codeclimate.com/github/FezVrasta/popper.js/coverage"><img src="https://codeclimate.com/github/FezVrasta/popper.js/badges/coverage.svg" alt="Istanbul Code Coverage"/></a>
    <a href="https://www.npmjs.com/browse/depended/popper.js"><img src="https://badgen.net/npm/dependents/popper.js" alt="Dependents packages" /></a>
    <a href="https://spectrum.chat/popper-js" target="_blank"><img src="https://img.shields.io/badge/chat-on_spectrum-6833F9.svg?logo=data%3Aimage%2Fsvg%2Bxml%3Bbase64%2CPHN2ZyBpZD0iTGl2ZWxsb18xIiBkYXRhLW5hbWU9IkxpdmVsbG8gMSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiB2aWV3Qm94PSIwIDAgMTAgOCI%2BPGRlZnM%2BPHN0eWxlPi5jbHMtMXtmaWxsOiNmZmY7fTwvc3R5bGU%2BPC9kZWZzPjx0aXRsZT5zcGVjdHJ1bTwvdGl0bGU%2BPHBhdGggY2xhc3M9ImNscy0xIiBkPSJNNSwwQy40MiwwLDAsLjYzLDAsMy4zNGMwLDEuODQuMTksMi43MiwxLjc0LDMuMWgwVjcuNThhLjQ0LjQ0LDAsMCwwLC42OC4zNUw0LjM1LDYuNjlINWM0LjU4LDAsNS0uNjMsNS0zLjM1UzkuNTgsMCw1LDBaTTIuODMsNC4xOGEuNjMuNjMsMCwxLDEsLjY1LS42M0EuNjQuNjQsMCwwLDEsMi44Myw0LjE4Wk01LDQuMThhLjYzLjYzLDAsMSwxLC42NS0uNjNBLjY0LjY0LDAsMCwxLDUsNC4xOFptMi4xNywwYS42My42MywwLDEsMSwuNjUtLjYzQS42NC42NCwwLDAsMSw3LjE3LDQuMThaIi8%2BPC9zdmc%2B" alt="Get support or discuss"/></a>
    <br />
    <a href="https://travis-ci.org/FezVrasta/popper.js/branches" target="_blank"><img src="https://travis-ci.org/FezVrasta/popper.js.svg?branch=master" alt="Build Status"/></a>
    <a href="https://saucelabs.com/u/popperjs" target="_blank"><img src="https://badges.herokuapp.com/browsers?labels=none&googlechrome=latest&firefox=latest&microsoftedge=latest&iexplore=11,10&safari=latest" alt="SauceLabs Reports"/></a>
</p>

<img src="https://raw.githubusercontent.com/FezVrasta/popper.js/master/popperjs.png" align="right" width=250 />

<!-- 🚨 HEY! HERE BEGINS THE INTERESTING STUFF 🚨 -->

## <a name="wut-poppers"></a>Wut? Poppers?

Popper 是屏幕上的一个元素，该元素从应用程序的自然流中 "弹出"。  
Poppers 的常见示例包括工具提示、popovers 和下拉。


## <a name="so-yet-another-tooltip-library"></a>那么，还有另一个工具提示库？

嗯，基本上，**没有**。  
Popper 是一个**定位引擎**，它的用途是计算元素的位置，以便可以将其放置在给定的 reference 元素附近。  

引擎完全是模块化的，并且它的大部分功能都实现为**修饰符**（类似于 middlewares 或插件）。  
整个代码库是在 ES2015 中编写的，它的功能将在真实浏览器上自动测试，感谢[SauceLabs](https://saucelabs.com/)和[TravisCI](https://travis-ci.org/)。

Popper 有零个依赖项。 无 jQuery，无 LoDash，nothing。  
它由在 WebClipper 和[AtlasKit](https://aui-cdn.atlassian.com/atlaskit/registry/)中的 Twitter 等大型公司（如[Twitter](https://getbootstrap.com/)）中的[Microsoft](https://github.com/OneNoteDev/WebClipper)使用。

### <a name="popperjs"></a>Popper

这是引擎、计算的库，也可以将样式应用于 poppers。

其中一些关键要点是：

- Position 元素将其保留在其原始 DOM 上下文中（DOM！）;
- 允许导出计算的提供，以与 "响应" 和 "其他视图" 库集成;
- 支持影子 DOM 元素;
- 完全可自定义，感谢基于修饰符的结构;

请访问 "[项目" 页](https://fezvrasta.github.io/popper.js)，查看可以使用 Popper！的许多示例。

请[在此处查找文档](/docs/_includes/popper-documentation.md)。


### <a name="tooltipjs"></a>Tooltip

由于大量用户只需要一种简单的方法来在其项目中集成功能强大的工具提示，因此我们创建了**Tooltip**。  
它是一个小库，使您能够轻松地使用 as engine Popper 自动创建工具提示。  
它的 API 与引导数据库的著名工具提示系统几乎完全相同，在这种情况下，将很容易将其集成到项目中。  
根据`aria`标记，由 Tooltip 生成的工具提示是可访问的。

请[在此处查找文档](/docs/_includes/tooltip-documentation.md)。


## <a name="installation"></a>安装
Popper 在以下包管理者和 Cdn 中可用：

| Source |                                                                                  |
|:-------|:---------------------------------------------------------------------------------|
| npm    | `npm install popper.js --save`                                                   |
| yarn   | `yarn add popper.js`                                                             |
| NuGet  | `PM> Install-Package popper.js`                                                  |
| Bower  | `bower install popper.js --save`                     |
| unpkg  | [`https://unpkg.com/popper.js`](https://unpkg.com/popper.js)                     |
| cdnjs  | [`https://cdnjs.com/libraries/popper.js`](https://cdnjs.com/libraries/popper.js) |

Tooltip 也是这样：

| Source |                                                                                  |
|:-------|:---------------------------------------------------------------------------------|
| npm    | `npm install tooltip.js --save`                                                  |
| yarn   | `yarn add tooltip.js`                                                            |
| Bower* | `bower install tooltip.js=https://unpkg.com/tooltip.js --save`                   |
| unpkg  | [`https://unpkg.com/tooltip.js`](https://unpkg.com/tooltip.js)                   |
| cdnjs  | [`https://cdnjs.com/libraries/popper.js`](https://cdnjs.com/libraries/popper.js) |

\*： Bower 不支持正式支持，它可用于安装 Tooltip trough 的 unpkg.com CDN。 此方法限制了无法定义库的特定版本。 Bower 和 Popper 建议对项目使用 npm 或 Yarn。  
有关详细信息，请[阅读相关问题](https://github.com/FezVrasta/popper.js/issues/390)。

### <a name="dist-targets"></a>Dist 目标

Popper 当前附带了3个目标： UMD、ESM 和 ESNext。

- UMD-通用模块定义： AMD、RequireJS 和 globals;
- ESM-ES 模块：用于支持规范的 webpack/汇总或浏览器;
- ESNext：中`dist/`的可用，可用于 webpack 和`babel-preset-env`;

请务必为你的需求使用正确的项。 如果要将其与`<script>`标记一起导入，请使用 UMD。

## <a name="usage"></a>用法

给定现有的 popper DOM 节点，请求 Popper 将其放置在其按钮旁边

```js
var reference = document.querySelector('.my-button');
var popper = document.querySelector('.my-popper');
var anotherPopper = new Popper(
    reference,
    popper,
    {
        // popper options here
    }
);
```

### <a name="callbacks"></a>回

Popper 支持两种类型的回调，在初始化`onCreate` Popper 之后调用回调。 在`onUpdate`任何后续更新中都会调用一种。

```js
const reference = document.querySelector('.my-button');
const popper = document.querySelector('.my-popper');
new Popper(reference, popper, {
    onCreate: (data) => {
        // data is an object containing all the informations computed
        // by Popper.js and used to style the popper and its arrow
        // The complete description is available in Popper.js documentation
    },
    onUpdate: (data) => {
        // same as `onCreate` but called on subsequent updates
    }
});
```

### <a name="writing-your-own-modifiers"></a>编写自己的修饰符

Popper 基于 "类似插件" 的体系结构，它的大部分功能都是完全封装的 "修饰符"。  
修饰符是一种函数，每次 Popper 需要计算 Popper 的位置时调用。 出于此原因，修饰符应具有非常高的性能，以避免瓶颈。  

若要了解如何创建修饰符，请[阅读修饰符文档](docs/_includes/popper-documentation.md#modifiers--object)


### <a name="react-vuejs-angular-angularjs-emberjs-etc-integration"></a>响应、Vue、角度、AngularJS、Ember （etc）集成

将第三方库集成在响应或其他库中可能是一个难题，因为它们通常会改变 DOM，并推动库的使用令人头痛。  
Popper 会限制它在修饰符中的`applyStyle`所有 DOM 修改，您只需禁用它并使用您选择的库手动应用 Popper 坐标即可。  

有关可让您使用 Popper 到现有框架中的库的完整列表，请访问[提及](/MENTIONS.md)页面。

或者，您甚至可以使用自定义`applyStyles`的自定义项替代您自己，并将 Popper 集成在一起！

```js
function applyReactStyle(data) {
    // export data in your framework and use its content to apply the style to your popper
};

const reference = document.querySelector('.my-button');
const popper = document.querySelector('.my-popper');
new Popper(reference, popper, {
    modifiers: {
        applyStyle: { enabled: false },
        applyReactStyle: {
            enabled: true,
            fn: applyReactStyle,
            order: 800,
        },
    },
});

```

### <a name="migration-from-popperjs-v0"></a>从 Popper v0 迁移

由于 API 发生了更改，因此我们已准备了一些迁移说明，以便更轻松地升级到 Popper v1。  

https://github.com/FezVrasta/popper.js/issues/62

如果你有任何疑问，请随意在问题中添加注释。

### <a name="performances"></a>性能

Popper 的性能非常高。 计算 popper 的位置（在具有 3.5 G GHz Intel Core i5 的 iMac 上）时，通常需要0.5 毫秒的时间。  
这意味着它不会导致任何[jank](https://www.chromium.org/developers/how-tos/trace-event-profiling-tool/anatomy-of-jank)，从而导致用户平稳的用户体验。

## <a name="notes"></a>注释

### <a name="libraries-using-popperjs"></a>使用 Popper 的库

Popper 的目标是提供可供在第三方库中使用的稳定且功能强大的定位引擎。  

访问 "[提及](/MENTIONS.md)" 页面，获取更新的项目列表。


### <a name="credits"></a>制作人员
我想感谢一些朋友和项目做的工作：

- 在 GitHub 页面上[@AndreaScn](https://github.com/AndreaScn)其工作，并在开发过程中进行手动测试;
- 为原始想法和库名称[@vampolo](https://github.com/vampolo) ;
- [Sysdig](https://github.com/Draios)在这些年中了解到的所有非常好的事情，让我能够编写此库;
- [Tether](http://github.hubspot.com/tether/)为现实世界编写定位库做好了准备。
- 他们非常感谢的拉取请求和错误报告[的参与者](https://github.com/FezVrasta/popper.js/graphs/contributors);
- **你**可以为你提供此项目，并为此项目提供试用版的明星🙂

### <a name="copyright-and-license"></a>版权和许可
代码和文档版权 2016 **Federico Zivolo**。 在[MIT 许可证](LICENSE.md)下发布的代码。 在创造性的 Commons 下发布的文档。
