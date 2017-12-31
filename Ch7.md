# 第7章：Web 动画技术大比拼

之前内容仅仅涵盖了使用CSS来实现各种动画。从这一章开始将主要转向 JavaScript，不过开始之前要做一件很重要的事情：对比各种网页端实现动画的技术并找出优缺点，以便更加了解正在使用的技术，并选择其来作为最适合当前工作的工具。

在本章结尾，我们也将讨论与React配合使用的工具，由于虚拟DOM的原因，在React环境中运行动画与通常方式相比有所不同。

这里是没有办法覆盖所有的动画库的，所以通常我会推荐那些更令我感兴趣。请记住，这些建议主要基于我自己的经验;你可能对这些选择抱有不同看法，这是没问题的。


> 可以在下面深入阅读各种实现方式的利弊及对比，不过基于我长期使用的经验，这里可以给出一个简短的建议： GreenSock， 它处理了SVG的一些跨浏览器问题，并且其实现考虑了各种动画场景，因此 GreenSock 会成为我最常用于生产环境的动画技术。

## 原生动画

在谈论动画框架之前先来看一些原生实现方式。大多数动画框架都会使用原生动画技术，所以对其了解越多，越能理解动画的抽象是如何实现的。

### CSS/Sass/SCSS

把CSS提到这么靠前的原因，是CSS体现了Web动画实现方案的奥卡姆剃刀原则 —— 当所有技术都可以实现时，最简单的解决方案有时是最好的，特别是当你需要快速搭建一个动画效果，可以使用CSS动画 —— 通过一组关键帧在不同状态之间进行转换来实现。

#### 优点：

- 你不需要依赖外部框架。
- 性能很好。预处理器（如Sass和LESS）允许利用函数来生成 `nth:child` 伪类来产生令人吃惊的效果。对于缓动函数等也可以使用变量，从而让动画保持一致性。
- 可以使用[原生的JavaScript](https://codepen.io/sdras/full/PqXeMX)监听`onAnimationEnd`以及其他动画相关的事件。
- 沿路径运动这样的效果将会在CSS中支持，这样的能力对于实现真实的运动动画是很有必要的，而由于SMIL渐渐被抛弃，这一点已经变得越发重要了。

#### 缺点：

- [Bézier 缓动](https://www.smashingmagazine.com/2014/04/understanding-css-timing-functions/)效果在某些情况下会受到限制。由于Bézier只能通过两个控制点来控制曲线，无法产生一些复杂的物理效果，如反弹或弹性振动，而这些对于实现逼真的动画效果（但非必需）是非常重要的。
- 如果需要依次运行三组以上的动画，我建议使用JavaScript。 CSS中对动画运行顺序是通过 `delay` 来控制的，这是很复杂的，当你调整时间的时候，不得不重新计算所有的 `delay`。你可以通过监听原生JavaScript事件来解决这个问题，但这样在编码时会切换不同语言，也不是很好的方案。持续时间长，复杂，有运行顺序的动画更容易在JavaScript中编写和阅读。
- 沿着路径运动效果的支持还不是很好。您可以[投票支持Firefox](https://bugzilla.mozilla.org/show_bug.cgi?id=1186329)。在Safari中支持的投票，只要我能收集，就更加个性化。我注册了一个bug报告，并要求CSS中的运动路径模块要作为一个功能。
- CSS + SVG动画会有些奇怪的问题。一个例子是在Safari浏览器中，动画中同时使用不透明度和变换可能会失效或产生奇怪的影响。另一点是在Web规范中，对于CSS变换原点的定义并不是人们通常理解的那样，从而当变换应用到元素时会[产生预期之外的效果](https://codepen.io/1Marc/full/DCvFm)。希望SVG2能解决这个问题，而现在对于移动端的CSS和SVG动画，有时需要一些hack才能保证动画符合预期。那些使用CSS的动画框架，在处理大量效果的同时（例如GSAP），还可以对其进行修正。
- 编写CSS动画时需要声明关键帧，然后编写元素本身的CSS来使用动画。这意味着运行动画所需的代码需要维护两个地方。有时这样很好，方便对动画的复用。但大多数情况下，这意味着可读性被破坏，因为你必须在两个地方更新代码。

### requestAnimationFrame()

`requestAnimationFrame()`（简称rAF）是JavaScript中`window`对象上的原生方法。这方法非常赞，在JavaScript引擎的控制下，它确定了最适合当前环境的动画帧速率，并且只会在这个速率运行。例如在移动设备上不会像桌面设备那样使用高帧率。当浏览器的某个`tab`未被激活时，动画也会停止运行，以免不必要地占用系统资源。因此，`requestAnimationFrame()`是一个非常高性能的动画实现方式，后面我们讨论的大部分框架都在内部使用了这个API。

`requestAnimationFrame()`以类似递归的方式工作;在绘制动画队列的下一帧之前会执行逻辑，然后再次调用自己来继续。这可能听起来有点复杂，但这样的实现方式意味着一系列不断运行的命令，因此它将插入如何更好的呈现中间步骤。

第15章会讲解更多有关 `requestAnimationFrame()`的信息，如果你有兴趣，可以继续阅读。

### Canvas

尽管`canvas`基于[栅格图形](http://vector-conversions.com/vectorizing/raster_vs_vector.html)，而SVG是基于矢量图形，你仍然[可以在`canvas`中使用SVG](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API/Drawing_DOM_objects_into_a_canvas)。由于`canvas`是基于栅格的，SVG可能不能像以前一样什么都不用管，而是需要做一些处理。


    var canvas = document.querySelector('canvas'),
        ctx = canvas.getContext('2d')
        pixelRatio = window.devicePixelRatio,
        w = canvas.width,
        h = canvas.height;
    
    canvas.width = w * pixelRatio
    canvas.height = h * pixelRatio

    ctx.arc (
        canvas.width / 2,
        canvas.height / 2,
        canvas.width / 2,
        0,
        Math.PI * 2
    )
    ctx.fill();
    canvas.style.width = w + 'px';

以上效果不需要太多的代码，但是当你习惯SVG是跟分辨率无关后，这里会造成一些困扰。[Egghead上有个很棒的视频](https://egghead.io/lessons/javascript-make-canvas-responsive-to-pixel-ratio)讲解了如何处理这个问题。

我在这里没有使用 SVG，但我了解到[Tiffany Rayside](https://codepen.io/search/pens?q=Tiffany+Rayside&limit=all&type=type-pens)和[Ana Tudor](https://codepen.io/search/pens?q=Ana%20Tudor)在CodePen放了一些很赞的作品，推荐你去他们的个人主页探索一下。

### Web 动画 API

Web动画API，用于浏览器和开发者使用原生JavaScript来描述DOM元素上的动画效果。通过这个API 可以创建更复杂的序列动画，而无需加载任何外部脚本（注意，现在可能需要使用polyfill）。这个API是通过提炼了各种各样的动画框架，以及开发者使用JavaScript实现动画的最佳实践而创建出来的。Web 动画 API将CSS动画的高性能，以及JavaScript控制动画运行的灵活性整合在一起。

#### 优点：

- 实现序列动画很简单，代码非常清晰易读。 [Dan Wilson通过一个很好的例子](https://codepen.io/danwilson/pen/QwrZwd)来比较了CSS关键帧和Web动画API。
- 性能表现似乎很好，不过我还是建议你使用自己的性能测试来看一下。

#### 缺点：

- 在本书出版时，浏览器对这个API的支持并不是很好。目前已经有比较好的polyfill来支持，但其API仍在变化，所以直到规范达到最终版后，我会谨慎地在生产环境中运行它。尽管如此，这仍然是Web动画的未来，因此在这段时间可以尝试。
- GSAP中的时间轴动画还拥有了更多强大的功能。而对我来说重要的是SVG的跨浏览器稳定性，以及痛过一行代码即可运行一系列动画的能力。

## 第三方框架

### GreenSock（GSAP）

GreenSock目前是网上最复杂的动画库，我非常喜欢使用它。对这个框架的偏爱来自于日常工作和实践，以及在浩如烟海的动画库中屡屡碰壁的血泪史。[我尤其喜欢使用这个框架来操作SVG](https://www.youtube.com/watch?v=ZNukcHhpSXg)。 [GreenSock 的动画 API](https://greensock.com/) 拥有很多非常赞的特性可以列在下面，同时GreenSock也有完善的[文档](https://greensock.com/docs#/HTML5/)和[论坛](https://greensock.com/forums/)用于查阅。

#### 优点：

- 对于[非原生动画支持的东西来说](https://css-tricks.com/weighing-svg-animation-techniques-benchmarks/)，它的表现非常出色，这是非常赞的。
- GSAP时间轴的很多功能比当前Web动画API的实现更为强大：对于我来说，更关注的是SVG的跨浏览器稳定性，以及在SVG中通过一行代码管理大量动画执行的能力。
- 如果想实现一些花哨的效果，比如文字动画，SVG变形，那么GreenSock也有大量其他插件来支持。
- GreenSock 的 [BezierPlugin 插件](https://greensock.com/BezierPlugin-JS)提供的路径运动能力，是目前对路径运动支持最好的。
- 如前所述，它解决了SVG跨浏览器的困扰。因此特别适用于手机。
- [GreenSock的可视化缓动编辑器](http://greensock.com/ease-visualizer)提供了非常逼真的方式来创建缓动函数。它甚至允许通过一个SVG路径来创建自定义缓动。
- 由于GreenSock可以对任何两个整数区间做补间动画，你可以实现一些很酷的东西，[如对SVG滤镜实现动画](http://codepen.io/sdras/full/gaxGBB/)，以获得一些令人惊叹的效果。对于GreenSock来讲，天空才是极限。关于这点会在第15章进行更多介绍。

#### 缺点：

- 当你使用插件时必须为其许可证来付钱。不过在购买之前可以尝试[CodePen上的开放版本](https://codepen.io/GreenSock/)。
- 当你管理第三方依赖时，必须关注目前生产环境中正在使用的版本;因为框架的新版本会定期出现，当你升级时需要进行测试（这可能是任何框架的事情）。

### Mo.js

[Mo.js](http://mojs.io/)由[LegoMushroom](http://codepen.io/sol0mka/)组织维护，Oleg Solomka写的一个动画框架。Oleg Solomka是一个非常有才华的动画师，已经为这个框架做了很多[优秀案例](http://codepen.io/sol0mka/full/ogOYJj/)，这让我非常兴奋。这个框架仍处于测试阶段，不过现在已经接近发行阶段了。有关如何使用它的更多细节，请参阅第13章。

#### 优点：

- 有一些形变，爆发和漩涡这样的效果，你可以开箱即用 —— 所以你不需要成为世界上最好的或最具创意的插画家，就可以实现这些美妙的效果。
- Mo.js提供了一些很好的工具和概念，包括运行对象，时间轴和自定义路径编辑器。这本身就是使用它的最引人注目的理由之一。
- Mo.js提供了不同的动画实现方式，你可以使用对象，或在过程式代码中绘制一个动画变换 —— 你可以自己决定最适合你的方式。

#### 缺点：

- 它还没有提供使用SVG作为自定义形状的能力（我相信LegoMushroom正在开发此功能），因此使用坐标系统和缩放来进行响应式开发不太直观，也难于在移动设备上工作。这是一个相当先进的技术，但是你目前可能不需要它。
- 他不像GreenSock一样做了浏览器兼容，这意味着跟使用CSS一样，你可能需要编写hacks。 LegoMushroom已经表示他们在努力解决这个问题了。


### Bodymovin’

[Bodymovin'](https://adobe.ly/2l8hD4i)用于在Adobe After Effects中构建动画，并可以轻松导出到SVG和JavaScript。Bodymovin的一些[演示令人印象深刻](http://codepen.io/airnan/)。不过我不会使用它，因为我喜欢使用代码从头开始构建东西（所以这违背了我的目的），但是如果团队中除了开发人员更多的是设计师，这个工具对你们来讲真的很棒。我唯一可以预料的一件事是如果以后需要改变这个效果，必须重新导出，所以这个工作流程可能是比较奇怪的。此外，输出的代码看起来像是一团乱麻，不过我还没观察到这些会影响性能，所以这应该是没问题的。

## 不推荐使用

### SMIL

SMIL（Synchronized Multimedia Integration Language）是一个原生的SVG动画规范：允许直接对SVG DOM中进行移动，变形，甚至与SVG进行交互。使用SMIL存在很多优缺点，但最大的问题会让我转向废弃这个方案：SMIL正在失去支持。我写了一篇[如何将SMIL实现的动画迁移到最新技术上的帖子](https://css-tricks.com/smil-is-dead-long-live-smil-a-guide-to-alternatives-to-smil-features/)。

### Velocity.js

[Velocity](http://velocityjs.org/)提供了之前GreenSock能够提供的动画序列控制，但简洁明快。由于下面提到的这些缺点，我放弃使用Velocity.js了。 Velocity.js的语法看起来有点像jQuery，如果你已经在使用jQuery，对其API上手速度快会是一个很大的优势。

#### 优点：

- 比CSS更容易控制多段动画的执行。
- 有许多开箱即用的缓动方法，[弹性运动](http://codepen.io/julianshapiro/pen/hyeDg)也支持。同时也支持将一系列缓动函数传递到[`step-easing`](http://julian.com/research/velocity/#easing)数组中。
- 文档很全，易于入门学习。

#### 缺点：

- 效果不是很好。尽管有一些相反的说法，当我运行自己的测试时发现有些效果并没有真正实现。不过我建议读者可以自己测试，因为代码还在更新，而我们的章节已经结束 
- GSAP在体积比较大的同时，具有更高的性能和跨浏览器稳定性。 jQuery在性能测试中落败，但未来可能会因为内部使用`rAF`技术而改变;Velocity.js 速度不差，但没什么突出表现。


### Snap.svg

很多人认为[Snap](http://snapsvg.io/)是一个动画库，而实际上并不是这样。我之前对Snap进行过性能测试，并且Dmitri Baranovskiy（这个框架以及前身Rafael的作者，拥有令人难以置信的聪明才智）在[SVG Immersion Podcast](http://svgimmersion.com/interview-dmitry-baranovskiy-p1)上提到，对于动画来讲，Snap不是最适合的工具。在我们的聊天记录中他提到：“要注意一点：Snap不是一个动画库，当我们需要实现动画时，我会建议使用GSAP。

其实，jQuery并不适合SVG，尽管它现在支持[类操作](https://blog.jquery.com/2015/07/13/jquery-3-0-and-jquery-compat-3-0-alpha-versions-released/)。如果需要使用SVG进行大量DOM操作，Snap是推荐的工具，这也是Snap的正确使用场景。

有一个名为[SnapFoo](http://yuschick.github.io/SnapFoo/)的库将Snap的领域扩展到动画。我还没玩过，但看起来很酷。

## 基于React的动画工作流

在讨论React之前，首先要看下为什么在React中处理动画时需要区别对待。主要区别在于文档对象模型（DOM），DOM描述了如何使用对象创建一个结构化文档，并且其结构主要为树形结构。

React有一个虚拟的DOM来抽象整个DOM结构。 React这样做有很多原因，对我来说最令人信服的是能够弄清楚什么会导致DOM变化，并且只更新需要更新的部分。当然，这种抽象是有代价的，之前用的一些旧技巧在React环境中会给你带来麻烦。例如，jQuery跟React一起就不容易玩转，因为jQuery的主要功能是与浏览器的原生DOM进行交互和操作。但我经常会看到一些奇怪的CSS竞争操作。以下是我在React工作流中实现动画的一些建议。

### React-Motion

Cheng Lou的[React-Motion](https://github.com/chenglou/react-motion)被认为是在React中实现动画的最佳方法，并且React社区几乎已经使用其代替了老旧的[ReactCSSTransitionGroup](https://reactjs.org/docs/animation.html)。我非常喜欢React-Motion，不过当你使用时要注意一些关键点，如果你不了解的话会比较头疼。

React-Motion非常类似于[基于游戏的动画](https://twitter.com/andy_matuschak/status/566736015188963328)，您可以在其中提供元素的质量和初始物理状态，并将其传递给元素，之后元素就动起来了，它不需要像CSS，或者其他传统的基于坐标的运动方式那样指定一系列的动画时间。由React-Motion实现的[动画效果看起来很逼真](https://codepen.io/sdras/full/pyedJE)，很酷炫，但有一点比较难处理，如果有两个不同的东西必须在同一时间启动，并同时到达终点，那么你会发现他们很难“齐头并进”。

最近，Cheng Lou加入了[onRest](https://twitter.com/_chenglou/status/722678228255711233)方法来支持传统的回调运行的方式。尽管如此，它并没有太多的进步，因为这种方式与该项目本身的实现原理是背道而驰的。而且也不容易实现一个循环动画（不是无限循环，无限循环会导致很多问题）。你可能永远不会遇到这个问题，但是我曾经遇到过一次。

#### 优点:

- 动画看起来真的很美观，弹性效果是非常好的。
- 非常独特的弹性效果 —— 在大多数JS库（如GSAP和Velocity）中都可以实现弹性效果，但是React-motion中的弹性效果直接基于最后一个元素的运动，而不是移除掉最后一个元素并重新渲染一个新的，所以在动画运动效果上会比较好。
- 这可能是在React中实现动画最好的工具了。

#### 缺点：

- 它不像其他一些库或原生方法一样的即插即用，这意味着你最终会写出更多的代码。有些人会喜欢这个而有些人不是。对初学者来说，这并不是一件好事。
- 由于代码的复杂性，实现顺序执行动画并不像其他方案那样直观或清晰。
- onRest仍然没有提供交错动画的参数。


### React中使用GSAP

GreenSock提供了这么多功能，因此它仍然值得在React环境中使用。它需要比平时更加​​轻量，有些事情不需要在React中处理（通过DOM的）。也就是说，我已经得到了几种不同的使用方式：

- [在React组件生命周期中使用](https://reactjs.org/docs/react-component.html)。
- 当需要交互时触发其运行。当用户交互时创建一个函数，然后将其挂接到类似于`onClick`的事件中。
- Chang Wang有一个[很好的帖子](https://medium.com/appifycanada/animations-with-reacttransitiongroup-4972ad7da286#.jr02uoi1q)来讲解如何将GSAP挂载到ReactTransitionGroup中，这是一个非常赞的方式。
- 你可以使用[React-Gsap-Enhancer](https://github.com/azazdeaz/react-gsap-enhancer)等库。 当遇到一些复杂的控制动画执行顺序的场景时，React-Gsap-Enhancer可以帮助你处理，而在通常情况下不需要杀鸡用牛刀，可以直接在React的生命周期内使用GSAP。

### React中使用Canvas

Canvas在React中可以正常运行。可以选择完全绕过DOM，创建一个节点，之后在上面创建所有动画。在React中使用Canvas的方式，跟之前讨论的Canvas的好处和限制相同（参见第79页上的“canvas”）。也可以将一个Canvas动画实现为React组件，但由于虚拟DOM的关系，实现细节可能会变得更加复杂。

有几个很好的框架可以帮助你在React中使用Canvas。 React-Canvas是由Flipboard团队开发的，他们想要在DOM上实现`60fps`的动画效果。这个repo已经有段时间没更新了，并且这个项目只关注UI元素，因此实现任何其他类型的动画都需要一些额外的工作。

[React Konva](https://github.com/lavrton/react-konva)是一个有趣的，声明式的使用canvas和React的方法。它很容易创造出好看的图形，但动画能力有些缺乏，开发人员也在文档中承认了这一点，因此如果你愿意提交pull request（PR），可以处理这个问题，并帮助改进这个项目。

### CSS in React

CSS会重新进入我们的考虑范围，因为它可能是在React中创建动画的最简单的方法。我喜欢使用CSS动画来处理诸如UI / UX交互的小事情，只不过当你尝试使用delay来实现连续的动画时，效果会变得有点奇怪。除此之外，它们非常好，特别是对于小UI调整。

## 结论

不幸的是，本书不可能深入分析以上所有的方式，否则这本书会变成10倍篇幅！由于其功能强大，用途广泛，我们将主要关注GreenSock 的 Animation API。我们还将介绍mo.js，React-Motion和`requestAnimationFrame()`，以便你了解如何通过“裸金属”的方式使用JavaScript完成动画。