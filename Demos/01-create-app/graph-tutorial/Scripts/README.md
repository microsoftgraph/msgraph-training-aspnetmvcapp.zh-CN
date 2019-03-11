<!-- IGNORE THE HTML BLOCK BELOW, THE INTERESTING PART IS AFTER IT -->

<h1 align="center"><span data-ttu-id="3fc4e-101">Popper</span><span class="sxs-lookup"><span data-stu-id="3fc4e-101">Popper.js</span></span></h1>

<p align="center"><span data-ttu-id="3fc4e-102">
    <strong>用于在 web 应用程序中定位 poppers 的库。</strong>
</span><span class="sxs-lookup"><span data-stu-id="3fc4e-102">
    <strong>A library used to position poppers in web applications.</strong>
</span></span></p>

<p align="center">
    <a href="https://travis-ci.org/FezVrasta/popper.js/branches" target="_blank"><img src="https://travis-ci.org/FezVrasta/popper.js.svg?branch=master" alt="Build Status"/></a>
    <img src="http://img.badgesize.io/https://unpkg.com/popper.js/dist/popper.min.js?compression=gzip" alt="Stable Release Size"/>
    <a href="https://www.bithound.io/github/FezVrasta/popper.js"><img src="https://www.bithound.io/github/FezVrasta/popper.js/badges/score.svg" alt="bitHound Overall Score"></a>
    <a href="https://codeclimate.com/github/FezVrasta/popper.js/coverage"><img src="https://codeclimate.com/github/FezVrasta/popper.js/badges/coverage.svg" alt="Istanbul Code Coverage"/></a>
    <a href="https://gitter.im/FezVrasta/popper.js" target="_blank"><img src="https://img.shields.io/gitter/room/nwjs/nw.js.svg" alt="Get support or discuss"/></a>
    <br />
    <a href="https://saucelabs.com/u/popperjs" target="_blank"><img src="https://badges.herokuapp.com/browsers?labels=none&googlechrome=latest&firefox=latest&microsoftedge=latest&iexplore=11,10&safari=latest&iphone=latest" alt="SauceLabs Reports"/></a>
</p>

<img src="https://raw.githubusercontent.com/FezVrasta/popper.js/master/popperjs.png" align="right" width=250 />

<!-- 🚨 HEY! HERE BEGINS THE INTERESTING STUFF 🚨 -->

## <a name="wut-poppers"></a><span data-ttu-id="3fc4e-103">Wut？</span><span class="sxs-lookup"><span data-stu-id="3fc4e-103">Wut?</span></span> <span data-ttu-id="3fc4e-104">Poppers？</span><span class="sxs-lookup"><span data-stu-id="3fc4e-104">Poppers?</span></span>

<span data-ttu-id="3fc4e-105">popper 是屏幕上的一个元素, 该元素从应用程序的自然流中 "弹出"。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-105">A popper is an element on the screen which "pops out" from the natural flow of your application.</span></span>  
<span data-ttu-id="3fc4e-106">poppers 的常见示例包括工具提示、popovers 和下拉。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-106">Common examples of poppers are tooltips, popovers and drop-downs.</span></span>


## <a name="so-yet-another-tooltip-library"></a><span data-ttu-id="3fc4e-107">那么, 还有另一个工具提示库？</span><span class="sxs-lookup"><span data-stu-id="3fc4e-107">So, yet another tooltip library?</span></span>

<span data-ttu-id="3fc4e-108">嗯, 基本上,**没有**。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-108">Well, basically, **no**.</span></span>  
<span data-ttu-id="3fc4e-109">Popper 是一个**定位引擎**, 它的用途是计算元素的位置, 以便可以将其放置在给定的 reference 元素附近。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-109">Popper.js is a **positioning engine**, its purpose is to calculate the position of an element to make it possible to position it near a given reference element.</span></span>  

<span data-ttu-id="3fc4e-110">引擎完全是模块化的, 并且它的大部分功能都实现为**修饰符**(类似于 middlewares 或插件)。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-110">The engine is completely modular and most of its features are implemented as **modifiers** (similar to middlewares or plugins).</span></span>  
<span data-ttu-id="3fc4e-111">整个代码库是在 ES2015 中编写的, 它的功能将在真实浏览器上自动测试, 感谢[SauceLabs](https://saucelabs.com/)和[TravisCI](https://travis-ci.org/)。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-111">The whole code base is written in ES2015 and its features are automatically tested on real browsers thanks to [SauceLabs](https://saucelabs.com/) and [TravisCI](https://travis-ci.org/).</span></span>

<span data-ttu-id="3fc4e-112">Popper 有零个依赖项。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-112">Popper.js has zero dependencies.</span></span> <span data-ttu-id="3fc4e-113">无 jQuery, 无 LoDash, nothing。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-113">No jQuery, no LoDash, nothing.</span></span>  
<span data-ttu-id="3fc4e-114">它由在 WebClipper 和[AtlasKit](https://aui-cdn.atlassian.com/atlaskit/registry/)中的 Twitter 等大型公司 (如[Twitter](https://getbootstrap.com/)) 中的[Microsoft](https://github.com/OneNoteDev/WebClipper)使用。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-114">It's used by big companies like [Twitter in Bootstrap v4](https://getbootstrap.com/), [Microsoft in WebClipper](https://github.com/OneNoteDev/WebClipper) and [Atlassian in AtlasKit](https://aui-cdn.atlassian.com/atlaskit/registry/).</span></span>

### <a name="popperjs"></a><span data-ttu-id="3fc4e-115">Popper</span><span class="sxs-lookup"><span data-stu-id="3fc4e-115">Popper.js</span></span>

<span data-ttu-id="3fc4e-116">这是引擎、计算的库, 也可以将样式应用于 poppers。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-116">This is the engine, the library that computes and, optionally, applies the styles to the poppers.</span></span>

<span data-ttu-id="3fc4e-117">其中一些关键要点是:</span><span class="sxs-lookup"><span data-stu-id="3fc4e-117">Some of the key points are:</span></span>

- <span data-ttu-id="3fc4e-118">Position 元素将其保留在其原始 DOM 上下文中 (DOM!);</span><span class="sxs-lookup"><span data-stu-id="3fc4e-118">Position elements keeping them in their original DOM context (doesn't mess with your DOM!);</span></span>
- <span data-ttu-id="3fc4e-119">允许导出计算的提供, 以与 "响应" 和 "其他视图" 库集成;</span><span class="sxs-lookup"><span data-stu-id="3fc4e-119">Allows to export the computed informations to integrate with React and other view libraries;</span></span>
- <span data-ttu-id="3fc4e-120">支持影子 DOM 元素;</span><span class="sxs-lookup"><span data-stu-id="3fc4e-120">Supports Shadow DOM elements;</span></span>
- <span data-ttu-id="3fc4e-121">完全可自定义, 感谢基于修饰符的结构;</span><span class="sxs-lookup"><span data-stu-id="3fc4e-121">Completely customizable thanks to the modifiers based structure;</span></span>

<span data-ttu-id="3fc4e-122">请访问 "[项目" 页](https://fezvrasta.github.io/popper.js), 查看可以使用 Popper! 的许多示例。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-122">Visit our [project page](https://fezvrasta.github.io/popper.js) to see a lot of examples of what you can do with Popper.js!</span></span>

<span data-ttu-id="3fc4e-123">请[在此处查找文档](/docs/_includes/popper-documentation.md)。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-123">Find [the documentation here](/docs/_includes/popper-documentation.md).</span></span>


### <a name="tooltipjs"></a><span data-ttu-id="3fc4e-124">Tooltip</span><span class="sxs-lookup"><span data-stu-id="3fc4e-124">Tooltip.js</span></span>

<span data-ttu-id="3fc4e-125">由于大量用户只需要一种简单的方法来在其项目中集成功能强大的工具提示, 因此我们创建了**Tooltip**。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-125">Since lots of users just need a simple way to integrate powerful tooltips in their projects, we created **Tooltip.js**.</span></span>  
<span data-ttu-id="3fc4e-126">它是一个小库, 使您能够轻松地使用 as engine Popper 自动创建工具提示。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-126">It's a small library that makes it easy to automatically create tooltips using as engine Popper.js.</span></span>  
<span data-ttu-id="3fc4e-127">它的 API 与引导数据库的著名工具提示系统几乎完全相同, 在这种情况下, 将很容易将其集成到项目中。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-127">Its API is almost identical to the famous tooltip system of Bootstrap, in this way it will be easy to integrate it in your projects.</span></span>  
<span data-ttu-id="3fc4e-128">根据`aria`标记, 由 Tooltip 生成的工具提示是可访问的。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-128">The tooltips generated by Tooltip.js are accessible thanks to the `aria` tags.</span></span>

<span data-ttu-id="3fc4e-129">请[在此处查找文档](/docs/_includes/tooltip-documentation.md)。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-129">Find [the documentation here](/docs/_includes/tooltip-documentation.md).</span></span>


## <a name="installation"></a><span data-ttu-id="3fc4e-130">安装</span><span class="sxs-lookup"><span data-stu-id="3fc4e-130">Installation</span></span>
<span data-ttu-id="3fc4e-131">Popper 在以下包管理者和 cdn 中可用:</span><span class="sxs-lookup"><span data-stu-id="3fc4e-131">Popper.js is available on the following package managers and CDNs:</span></span>

| <span data-ttu-id="3fc4e-132">Source</span><span class="sxs-lookup"><span data-stu-id="3fc4e-132">Source</span></span> |                                                                                  |
|:-------|:---------------------------------------------------------------------------------|
| <span data-ttu-id="3fc4e-133">npm</span><span class="sxs-lookup"><span data-stu-id="3fc4e-133">npm</span></span>    | `npm install popper.js --save`                                                   |
| <span data-ttu-id="3fc4e-134">yarn</span><span class="sxs-lookup"><span data-stu-id="3fc4e-134">yarn</span></span>   | `yarn add popper.js`                                                             |
| <span data-ttu-id="3fc4e-135">NuGet</span><span class="sxs-lookup"><span data-stu-id="3fc4e-135">NuGet</span></span>  | `PM> Install-Package popper.js`                                                  |
| <span data-ttu-id="3fc4e-136">Bower</span><span class="sxs-lookup"><span data-stu-id="3fc4e-136">Bower</span></span>  | `bower install popper.js --save`                     |
| <span data-ttu-id="3fc4e-137">unpkg</span><span class="sxs-lookup"><span data-stu-id="3fc4e-137">unpkg</span></span>  | [`https://unpkg.com/popper.js`](https://unpkg.com/popper.js)                     |
| <span data-ttu-id="3fc4e-138">cdnjs</span><span class="sxs-lookup"><span data-stu-id="3fc4e-138">cdnjs</span></span>  | [`https://cdnjs.com/libraries/popper.js`](https://cdnjs.com/libraries/popper.js) |

<span data-ttu-id="3fc4e-139">Tooltip 也是这样:</span><span class="sxs-lookup"><span data-stu-id="3fc4e-139">Tooltip.js as well:</span></span>

| <span data-ttu-id="3fc4e-140">Source</span><span class="sxs-lookup"><span data-stu-id="3fc4e-140">Source</span></span> |                                                                                  |
|:-------|:---------------------------------------------------------------------------------|
| <span data-ttu-id="3fc4e-141">npm</span><span class="sxs-lookup"><span data-stu-id="3fc4e-141">npm</span></span>    | `npm install tooltip.js --save`                                                  |
| <span data-ttu-id="3fc4e-142">yarn</span><span class="sxs-lookup"><span data-stu-id="3fc4e-142">yarn</span></span>   | `yarn add tooltip.js`                                                            |
| <span data-ttu-id="3fc4e-143">Bower \*</span><span class="sxs-lookup"><span data-stu-id="3fc4e-143">Bower\*</span></span> | `bower install tooltip.js=https://unpkg.com/tooltip.js --save`                   |
| <span data-ttu-id="3fc4e-144">unpkg</span><span class="sxs-lookup"><span data-stu-id="3fc4e-144">unpkg</span></span>  | [`https://unpkg.com/tooltip.js`](https://unpkg.com/tooltip.js)                   |
| <span data-ttu-id="3fc4e-145">cdnjs</span><span class="sxs-lookup"><span data-stu-id="3fc4e-145">cdnjs</span></span>  | [`https://cdnjs.com/libraries/popper.js`](https://cdnjs.com/libraries/popper.js) |

<span data-ttu-id="3fc4e-146">\*: Bower 不支持正式支持, 它可用于安装 Tooltip trough 的 unpkg.com CDN。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-146">\*: Bower isn't officially supported, it can be used to install Tooltip.js only trough the unpkg.com CDN.</span></span> <span data-ttu-id="3fc4e-147">此方法限制了无法定义库的特定版本。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-147">This method has the limitation of not being able to define a specific version of the library.</span></span> <span data-ttu-id="3fc4e-148">Bower 和 Popper 建议对项目使用 npm 或 Yarn。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-148">Bower and Popper.js suggests to use npm or Yarn for your projects.</span></span>  
<span data-ttu-id="3fc4e-149">有关详细信息, 请[阅读相关问题](https://github.com/FezVrasta/popper.js/issues/390)。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-149">For more info, [read the related issue](https://github.com/FezVrasta/popper.js/issues/390).</span></span>

### <a name="dist-targets"></a><span data-ttu-id="3fc4e-150">Dist 目标</span><span class="sxs-lookup"><span data-stu-id="3fc4e-150">Dist targets</span></span>

<span data-ttu-id="3fc4e-151">Popper 当前附带了3个目标: UMD、ESM 和 ESNext。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-151">Popper.js is currently shipped with 3 targets in mind: UMD, ESM and ESNext.</span></span>

- <span data-ttu-id="3fc4e-152">UMD-通用模块定义: AMD、RequireJS 和 globals;</span><span class="sxs-lookup"><span data-stu-id="3fc4e-152">UMD - Universal Module Definition: AMD, RequireJS and globals;</span></span>
- <span data-ttu-id="3fc4e-153">ESM-ES 模块: 用于支持规范的 webpack/汇总或浏览器;</span><span class="sxs-lookup"><span data-stu-id="3fc4e-153">ESM - ES Modules: For webpack/Rollup or browser supporting the spec;</span></span>
- <span data-ttu-id="3fc4e-154">ESNext: 中`dist/`的可用, 可用于 webpack 和`babel-preset-env`;</span><span class="sxs-lookup"><span data-stu-id="3fc4e-154">ESNext: Available in `dist/`, can be used with webpack and `babel-preset-env`;</span></span>

<span data-ttu-id="3fc4e-155">请务必为你的需求使用正确的项。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-155">Make sure to use the right one for your needs.</span></span> <span data-ttu-id="3fc4e-156">如果要将其与`<script>`标记一起导入, 请使用 UMD。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-156">If you want to import it with a `<script>` tag, use UMD.</span></span>

## <a name="usage"></a><span data-ttu-id="3fc4e-157">用法</span><span class="sxs-lookup"><span data-stu-id="3fc4e-157">Usage</span></span>

<span data-ttu-id="3fc4e-158">给定现有的 popper DOM 节点, 请求 popper 将其放置在其按钮旁边</span><span class="sxs-lookup"><span data-stu-id="3fc4e-158">Given an existing popper DOM node, ask Popper.js to position it near its button</span></span>

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

### <a name="callbacks"></a><span data-ttu-id="3fc4e-159">回</span><span class="sxs-lookup"><span data-stu-id="3fc4e-159">Callbacks</span></span>

<span data-ttu-id="3fc4e-160">Popper 支持两种类型的回调, 在 initalized `onCreate` Popper 后调用回调。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-160">Popper.js supports two kinds of callbacks, the `onCreate` callback is called after the popper has been initalized.</span></span> <span data-ttu-id="3fc4e-161">在`onUpdate`任何后续更新中都会调用一种。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-161">The `onUpdate` one is called on any subsequent update.</span></span>

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

### <a name="writing-your-own-modifiers"></a><span data-ttu-id="3fc4e-162">编写自己的修饰符</span><span class="sxs-lookup"><span data-stu-id="3fc4e-162">Writing your own modifiers</span></span>

<span data-ttu-id="3fc4e-163">Popper 基于 "类似插件" 的体系结构, 它的大部分功能都是完全封装的 "修饰符"。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-163">Popper.js is based on a "plugin-like" architecture, most of its features are fully encapsulated "modifiers".</span></span>  
<span data-ttu-id="3fc4e-164">修饰符是一种函数, 每次 Popper 需要计算 Popper 的位置时调用。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-164">A modifier is a function that is called each time Popper.js needs to compute the position of the popper.</span></span> <span data-ttu-id="3fc4e-165">出于此原因, 修饰符应具有非常高的性能, 以避免瓶颈。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-165">For this reason, modifiers should be very performant to avoid bottlenecks.</span></span>  

<span data-ttu-id="3fc4e-166">若要了解如何创建修饰符, 请[阅读修饰符文档](docs/_includes/popper-documentation.md#modifiers--object)</span><span class="sxs-lookup"><span data-stu-id="3fc4e-166">To learn how to create a modifier, [read the modifiers documentation](docs/_includes/popper-documentation.md#modifiers--object)</span></span>


### <a name="react-vuejs-angular-angularjs-emberjs-etc-integration"></a><span data-ttu-id="3fc4e-167">响应、Vue、角度、AngularJS、Ember (etc) 集成</span><span class="sxs-lookup"><span data-stu-id="3fc4e-167">React, Vue.js, Angular, AngularJS, Ember.js (etc...) integration</span></span>

<span data-ttu-id="3fc4e-168">将第三方库集成在响应或其他库中可能是一个难题, 因为它们通常会改变 DOM, 并推动库的使用令人头痛。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-168">Integrating 3rd party libraries in React or other libraries can be a pain because they usually alter the DOM and drive the libraries crazy.</span></span>  
<span data-ttu-id="3fc4e-169">Popper 会限制它在修饰符中的`applyStyle`所有 DOM 修改, 您只需禁用它并使用您选择的库手动应用 Popper 坐标即可。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-169">Popper.js limits all its DOM modifications inside the `applyStyle` modifier, you can simply disable it and manually apply the popper coordinates using your library of choice.</span></span>  

<span data-ttu-id="3fc4e-170">有关可让您使用 Popper 到现有框架中的库的完整列表, 请访问[提及](/MENTIONS.md)页面。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-170">For a comprehensive list of libraries that let you use Popper.js into existing frameworks, visit the [MENTIONS](/MENTIONS.md) page.</span></span>

<span data-ttu-id="3fc4e-171">或者, 您甚至可以使用自定义`applyStyles`的自定义项替代您自己, 并将 Popper 集成在一起!</span><span class="sxs-lookup"><span data-stu-id="3fc4e-171">Alternatively, you may even override your own `applyStyles` with your custom one and integrate Popper.js by yourself!</span></span>

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

### <a name="migration-from-popperjs-v0"></a><span data-ttu-id="3fc4e-172">从 Popper v0 迁移</span><span class="sxs-lookup"><span data-stu-id="3fc4e-172">Migration from Popper.js v0</span></span>

<span data-ttu-id="3fc4e-173">由于 API 发生了更改, 因此我们已准备了一些迁移说明, 以便更轻松地升级到 Popper v1。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-173">Since the API changed, we prepared some migration instructions to make it easy to upgrade to Popper.js v1.</span></span>  

https://github.com/FezVrasta/popper.js/issues/62

<span data-ttu-id="3fc4e-174">如果你有任何疑问, 请随意在问题中添加注释。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-174">Feel free to comment inside the issue if you have any questions.</span></span>

### <a name="performances"></a><span data-ttu-id="3fc4e-175">性能</span><span class="sxs-lookup"><span data-stu-id="3fc4e-175">Performances</span></span>

<span data-ttu-id="3fc4e-176">Popper 的性能非常高。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-176">Popper.js is very performant.</span></span> <span data-ttu-id="3fc4e-177">计算 popper 的位置 (在具有 3.5 g GHz Intel Core i5 的 iMac 上) 时, 通常需要0.5 毫秒的时间。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-177">It usually takes 0.5ms to compute a popper's position (on an iMac with 3.5G GHz Intel Core i5).</span></span>  
<span data-ttu-id="3fc4e-178">这意味着它不会导致任何[jank](https://www.chromium.org/developers/how-tos/trace-event-profiling-tool/anatomy-of-jank), 从而导致用户平稳的用户体验。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-178">This means that it will not cause any [jank](https://www.chromium.org/developers/how-tos/trace-event-profiling-tool/anatomy-of-jank), leading to a smooth user experience.</span></span>

## <a name="notes"></a><span data-ttu-id="3fc4e-179">注释</span><span class="sxs-lookup"><span data-stu-id="3fc4e-179">Notes</span></span>

### <a name="libraries-using-popperjs"></a><span data-ttu-id="3fc4e-180">使用 Popper 的库</span><span class="sxs-lookup"><span data-stu-id="3fc4e-180">Libraries using Popper.js</span></span>

<span data-ttu-id="3fc4e-181">Popper 的目标是提供可供在第三方库中使用的稳定且功能强大的定位引擎。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-181">The aim of Popper.js is to provide a stable and powerful positioning engine ready to be used in 3rd party libraries.</span></span>  

<span data-ttu-id="3fc4e-182">访问 "[提及](/MENTIONS.md)" 页面, 获取更新的项目列表。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-182">Visit the [MENTIONS](/MENTIONS.md) page for an updated list of projects.</span></span>


### <a name="credits"></a><span data-ttu-id="3fc4e-183">制作人员</span><span class="sxs-lookup"><span data-stu-id="3fc4e-183">Credits</span></span>
<span data-ttu-id="3fc4e-184">我想感谢一些朋友和项目做的工作:</span><span class="sxs-lookup"><span data-stu-id="3fc4e-184">I want to thank some friends and projects for the work they did:</span></span>

- <span data-ttu-id="3fc4e-185">在 GitHub 页面上[@AndreaScn](https://github.com/AndreaScn)其工作, 并在开发过程中进行手动测试;</span><span class="sxs-lookup"><span data-stu-id="3fc4e-185">[@AndreaScn](https://github.com/AndreaScn) for his work on the GitHub Page and the manual testing he did during the development;</span></span>
- <span data-ttu-id="3fc4e-186">为原始想法和库名称[@vampolo](https://github.com/vampolo) ;</span><span class="sxs-lookup"><span data-stu-id="3fc4e-186">[@vampolo](https://github.com/vampolo) for the original idea and for the name of the library;</span></span>
- <span data-ttu-id="3fc4e-187">[Sysdig](https://github.com/Draios)在这些年中了解到的所有非常好的事情, 让我能够编写此库;</span><span class="sxs-lookup"><span data-stu-id="3fc4e-187">[Sysdig](https://github.com/Draios) for all the awesome things I learned during these years that made it possible for me to write this library;</span></span>
- <span data-ttu-id="3fc4e-188">[Tether](http://github.hubspot.com/tether/)为现实世界编写定位库做好了准备。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-188">[Tether.js](http://github.hubspot.com/tether/) for having inspired me in writing a positioning library ready for the real world;</span></span>
- <span data-ttu-id="3fc4e-189">他们非常感谢的拉取请求和错误报告[的参与者](https://github.com/FezVrasta/popper.js/graphs/contributors);</span><span class="sxs-lookup"><span data-stu-id="3fc4e-189">[The Contributors](https://github.com/FezVrasta/popper.js/graphs/contributors) for their much appreciated Pull Requests and bug reports;</span></span>
- <span data-ttu-id="3fc4e-190">**你**可以为你提供此项目, 并为此项目提供试用版的明星🙂</span><span class="sxs-lookup"><span data-stu-id="3fc4e-190">**you** for the star you'll give to this project and for being so awesome to give this project a try 🙂</span></span>

### <a name="copyright-and-license"></a><span data-ttu-id="3fc4e-191">版权和许可</span><span class="sxs-lookup"><span data-stu-id="3fc4e-191">Copyright and license</span></span>
<span data-ttu-id="3fc4e-192">代码和文档版权 2016 **Federico Zivolo**。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-192">Code and documentation copyright 2016 **Federico Zivolo**.</span></span> <span data-ttu-id="3fc4e-193">在[MIT 许可证](LICENSE.md)下发布的代码。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-193">Code released under the [MIT license](LICENSE.md).</span></span> <span data-ttu-id="3fc4e-194">在创造性的 Commons 下发布的文档。</span><span class="sxs-lookup"><span data-stu-id="3fc4e-194">Docs released under Creative Commons.</span></span>
