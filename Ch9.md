# 第9章：GreenSock 的 Timeline 库

在上一节内容中我们讲述了GreenSock tweening 库函数。 而在这章内容中，我们将翻到我对于GreenSock中最爱的一个功能： 时间线（timeline）

## 一个简单的 Timeline

在上一章我们谈论了有关GreenSock的语法，重点集中在 TweenMax 基础部分，请大家而回想一下，我之前提到过 TweenMax 实际上依赖于 TimelineMax 这个库， TimelineMax是GreenSock 工具集中 功能最全的时间线控制工具了。 那么它优秀在哪里呢？

TimelineMax 是一个很强大的工具，它可以用于控制复杂动画和动画执行的先后顺序。 使用它之前，你需要先实例化一个TimelineMax 实例，我们可以这样做： `var tl = new TimelineMax();` (当然，如果你使用的Es6语法的话，也可以选择使用`let` 或者 `const`进行声明) 你虽然可以随便起你想起的变量名，但是 `tl` 往往是行业内标准地声明方式。

你可以使用 TweenMax 做任何动画，但需要注意的是 TweenMax 默认会让你制定的所有动画同时执行。如果你想分步执行，那么你需要给每个动画添加动画时延。

让我们来看看 图9-1 的三个图形动画，它们会一个接着一个的横向从左向右运动。

![](images/svga_0901.png)

*图 9-1： 横向运动。*

到目前为止，按照我们已经学过的知识，代码可以这么写：

	TweenMax.to(".star", 3, {x: 300, ease: Power4.easeOut});
	TweenMax.to(".circle", 3, {x: 300, delay:3, ease: Power4.easeOut});
	TweenMax.to(".hex", 3, {x: 300, delay:6, ease: Power4.easeOut});

以上代码虽然是能起作用的， 但是如果场景一变，我们需要运动更多的元素，我们就不得不一直人工地计算每一个元素需要的时延，这样很麻烦耗时耗力。所以这时候我们应该更加感觉到应该使用 `timeline` 来维护这些动画， 它能帮我们将目标元素自动地一个接着一个的运动（注意，此时我们就再也不需要计算和声明时延了）

	var tl = new TimelineMax();
	tl.to(".star", 3, {x: 300, ease: Power4.easeOut});
	tl.to(".circle", 3, {x: 300, ease: Power4.easeOut});
	tl.to(".hex", 3, {x: 300, ease: Power4.easeOut});

这样的解决方案非常好，因为从动画的开始到结束，`timeline` 都会自动依照我们指定的顺序去执行每一个我们想要执行的动画。

而当我们看了这个动画觉得不太合适，并决定调整小球的运动动画。调整的方案是让小球动画位于星星动画完成之前就开始小球动画。那么我们应该怎么设置呢？ 我们可以使用我们之前在`.staggerTo` （见89页）动画中所设置的 间位参数，通过相对的时间增量（相对前一个动画所设置的动画时长减1秒）：

	var tl = new TimelineMax();
	tl.to(".star", 3, {x: 300, ease: Power4.easeOut});
	tl.to(".circle", 3, {x: 300, ease: Power4.easeOut}, "-=1");
	tl.to(".hex", 3, {x: 300, ease: Power4.easeOut});

> #### 时间的增量
> 
> 在之前的例子中，我们使用了相对增量`"-=1"`让 `timeline` 了解我们想提前一秒执行下一个运动动画。如果你对JavaScript 并不熟悉， 增量对于你来说是很有用的。 这种 `+=1` （或者任意的整型数） `-=1`的语法。能够让编译器清除的明白，我们希望在原先的基础上加上或减去`1`秒，而不是设置一个单纯静态的值。你可以看看下面多种`timeline`增量使用的例子。
> 
> 例如我们想基于上一个动画完成后再上加1秒的时延，我们就可以这样写：`tl.to(".circle", 3, {x: 300}, "+=1");`
> 
> 如果我们想基于上一个动画完成前一秒开始下一个动画，我们可以这么写（但是需要保证之前已经声明了上一个动画）`tl.to(".circle", 3, {x: 300}, "-=1");`
> 
> 或者我们也可以设置一个特别的静态时间，例如2秒，那么动画就会确切的在第二秒钟开始执行：`tl.to(".circle", 3, {x: 300}, "2");`
> 
> 我最喜欢的，是通过设置相对标签，这样可以对下面的所有动画都会影响。

当小球在星星动画完成前一秒的时候开始运动，而后面六边形则不会改变时间顺序，依然紧跟着小球，当小球完成后才开始运动，我们再设计小球的运动的时延的时候，无需关心小球之后的六边形的运动时间。（原来六边形开始运动的时候是在第6秒的时候，而现在则在第5秒的时候开始）。

## 相对标签

到目前为止，一切都很好很OK，(但是有一个问题，TweenMax 默认是串行，一个接着一个的运动的) 如果你需要完成一个非常复杂的动画 并且你需要同时并行地运行多个动画或者你想轻缓地在某一个动画前后执行另个动画，这里的动画逻辑会稍复杂和混乱，尤其是当你需要校准动画时间顺序（相信我，你在开发动画过程中无时无刻都需要校准动画时间顺序）。

我们这时候就需要用到相对标签。相对标签十分好用，因为它可以作为一个时间基准点被时间线中插入，或者在动画开始前和结束后。我们使用 `tl.add('相对标记名')` 这样的语法来声明一个相对标记。

让我们看个简单的例子（图9-2）

![](images/svga_0902.png)

*图9-2: 是一个星星自转和三个小球移动的动画。*

在这个例子中，我们可以看到星星在自转，当旋转完成后，三个小球立刻同时放大并移向三个不同的方向。

虽然运动形式相同，由于每一个小球运动的方向是不同的，所以我们不能对它们使用同样的动画形式（是的，我们可以创建一个函数进行复用，但是我要放在下一章节进行介绍），我们需要对它们挨个创建动画，而timelineMax 默认是一个一个执行的，如果使用原先的方式，我们需要向下面这样挨个计算并设置时间增量，但是如果我们使用 相对标记 就会显得十分方便： 

	var tl = new TimelineMax();
	tl.to(".star", 3, {
		rotation: 30,
		transformOrigin: "50% 50%"
	});
	tl.to(".circle1", 1, {
		scale: 2.5
		x: 100,
		y: -70
	});
	tl.to(".circle2", 1, {
		scale: 2.5
		y: -100
	}, "-=1");  // 每次设置新动画，都需要基于 第一个小球的时间点设置时间增量
	tl.to(".circle3", 1, {
		scale: 2.5
		y: -70,
		x: -100
	}, "-=2");

如果我们使用相对标记就会显得十分方便：

	var tl = new TimelineMax();
	tl.to(".star", 3, {
		rotation: 30,
		transformOrigin: "50% 50%"
	});
	tl.add("burst");  // 在第一个小球之前设置相对标记点
	tl.to(".circle1", 1, {
		scale: 2.5
		x: 100,
		y: -70
	}, "burst"); // 后续的三个小球的运动时间都相对于标记设置的时间点进行运动
	tl.to(".circle2", 1, {
		scale: 2.5
		y: -100
	}, "burst");
	tl.to(".circle3", 1, {
		scale: 2.5
		y: -70,
		x: -100
	}, "burst");

相比于对一个一个小球加增量的做法，我更喜欢 使用`add('burst')`, 因为这样看起来更加的清晰灵活，所有的元素都会在`timeline` 中更早的运动，所有小球都会在同一时刻向三个方向运动。当我们设置了 相对标记 后，所有小球运动的时间同时改变了，都不需要重新计算时间顺序。你甚至可以依据相对标记点做出较晚或较早的时间调整。例如，如果你想在时间点后一秒发射三个小球，我们只需结合之前我们学过的间位参数像这样`"brust+=1"`，对相对标记点的时间设置增量就行了 。

> #### 使用`.set` 方法 能使你的动画项目较稳定
> 
> 你也需很多次在对元素进行动画前声明了某些属性，例如如果你想对某个元素设置z轴方向的运动 (例如 `translateZ`)，你就需要事先先设置 `perspective` 才能看到效果，又或者，你为了使用 DrawSVG 动画库进行描边动画，对一个图形初始形态设置了无描边（`stroke-dasharray: 0` 图形周长） 虽然这些事情你可以在CSS中设置属性定义，但是更好的做法是使用 `.set` 函数在js中设置元素的初始状态，因为对于那些读你代码准备进行维护的其他人来说  可以明确知道你接下来需要对哪一块做动画运动。而如果设置在CSS内，在JavaScript之外，其他人就不方便明白你运动的初始状态是什么（还要去查对应的CSS文件） 所以使用`.set` 设置元素的初始状态，是利于他人后期维护的。总的来说，你可以用 `tl.set` 方法设置状态且这种方式不会对元素造成任何运动:`tl.set(".circle", {scale: 0.5});`
>
> 你也不需要总是使用`.set`设置，如果你设置的是 `.fromTo` 动画， 就可以直接指定元素从初始状态变化到另一个状态，直接完成整个动画。 让我们来看看应该怎么做：
>
>	tl.fromTo(".circle", 1， { // 原书缺少repeat参数，不加上会报错
> 
>		scale: 0.5
> 
>	}, {
> 	
> 		scale: 1
>
> 	}, 8);
>
> 虽然这个动画设置了8秒延迟，但是一开始的初始状态也会保持到整个动画开始之前。
> 
> 即使如此，在某些场景，即使你使用了 `from` 或者 `fromTo` 动画的时候 也还是需要`.set`函数辅助，例如有这么一个场景：一开始你设置了某个元素透明度为`0`，你可能会注意到在JavaScript还在加载的过程中有一瞬间元素会显现出来，这是由于 JavaScript 还没有加载完还没有来得及对元素设置透明所导致的，虽然这通常也只有一瞬间，但是是能被肉眼察觉到的。 这种情况该怎么解决呢，我们可以这样： 由于 CSS是首先加载，所以我们在CSS中先设置:
> `.element { visibility: hidden; }`
> 
> 这样元素一开始就是被隐藏了。然后我们在JavaScript 中 使用`.set` 处理，将隐藏的变为显现，并且设置淡入动画：`TweenMax.set(".element", {visibility:"visible"});`
> 
> 这一招，我在几乎每一次动画项目中都屡试不爽。

## 主时间轴 和 所嵌套的场景

我不建议你们将动画声明函数写在js的全局域中，如果是一个比较简单的动画，你可以把你的动画逻辑放在 IIFE （IIFE 即是 immediately invoked function expression 自执行函数表达式）中， 如果是有点规模的动画项目，还是应该通过函数形式封装并调用

而更进一步的方式则是先创建一个主时间轴。 之所以我喜欢这样做，是因为我可以把整个动画切分成很多个场景。这样便于我更好的进行控制， 例如：

- 当我需要对某场景动画进行调整的时候，我可以根据某个场景对应的名字和场次快速定位到我需要微调的位置
- 容易对整个场景进行定位
- 每次只做一个场景，效率更快
- 我们可以使用`.seek` 方法调整某一整个场景的运动时间，这样不用在整个场景动画中每一帧每一帧的去调整
- 只需为主时间轴绑定一个事件就能统一控制重播和播放，例如点击事件
- 保持我的整个动画代码是有组织，有逻辑，且整洁易读的

让我们继续深挖， 来看看如何创建一个 主时间轴，我们将会探索每一个精华的部分。

### 代码逻辑组织

让我们来看看这个例子： 如何设置一条子时间轴，并将其嵌套进主时间轴的

	function sceneOne() {
		var tl = new TimelineMax();
		tl.add("begin");
		tl.to(".bubble", 2, {
		scale: 3,
		opacity: 0.5,
		rotation: 90,
		ease: Circ.easeOut
	}, "begin");
		...
		return tl;
	}
	var master = new TimelineMax();
	master.add(sceneOne(), "scene1");

在这个例子中，我们定义了一个函数，并且在函数中，声明了一个`timeline` 实例和设置了一系列的动画（这个函数也不一定要交`sceneOne`—— 你可以给它命名为任何你想要的名字）。而在函数的最后，我们将这个`timeline`实例返回， 然后把这个场景动画函数添加到 `master` 主时间轴中。你也许也注意到了我为这个场景设置了一个 定位标签—— `scene1`, 这样做可以让我，在工作中通过这个标签来控制`sceneOne`这个场景。

那么现在，让我们尝试在主时间轴中插入更多的场景，这都是没有技术含量的活：

	function sceneOne() {
		var tl = new TimelineMax();
			tl.add("begin");
			tl.to(".bubble", 2, {
			scale: 3,
			opacity: 0.5,
			rotation: 90,
			ease: Circ.easeOut
		}, "begin");
		...
		return tl;
	}
	function sceneTwo() {
		var tl = new TimelineMax();
			tl.add("boom");
			tl.to(".star", 2, {
			scale: 5,
			opacity: 0,
			rotation: -360,
			ease: Circ.easeIn
		}, "boom");
		...
		return tl;
	}
	var master = new TimelineMax();
	master.add(sceneOne(), "scene1")
	.add(sceneTwo(), "scene2");

我们也可以根据自己的想法来简单的改变每一个场景的顺序：

	var master = new TimelineMax();
	master.add(sceneTwo(), "scene2")
	.add(sceneOne(), "scene1");

> #### 学会使用 `.seek` 方法实现更好的工作体验
> 
> 你写的动画终归是会开始变得很长的。这时候对于部分单一动画的调整起来会变得比较困难。你必须时刻警惕地提醒自己正在修改那些东西。 为了解决这个痛点， 我们在工作中需要使用到 `.seek` 函数。 这时候，我们前面声明的 位置标签就变得十分方便。 有时候，当我在调整一个只有几秒的小动画的时候，可能会突然忘记自己想做什么。于是我们就可以用`.seek` 函数，把多个场景分离开，再调整单独的部分场景

	var master = new TimelineMax();

	master.add(sceneOne(), "scene1")
		.add(sceneTwo(), "scene2");

	master.seek("scene2+=3"); // 让当前调整的场景整个+3秒， 从而使其和其他场景分离开

当我完成了一段很长的动画的时候时常会注意到有一段场景有少许快或少许慢。 由于动画较长，不方便再微调，这时候我们就可以发挥`timeScale()` 方法的优势。 这是GreenSock一个十分有用的功能，你一定会因为它提供的便利感到很惊奇。 我会将一长段动画分成不同的组织结构，然后使用 `timeScale` 来对这些结构进行微调，让我们来看看这个例子：

	function sceneOne() {
		var tl = new TimelineMax();
		tl.add("begin");
		tl.to(".bubble", 2, {
		scale: 3,
		opacity: 0.5,
		rotation: 90,
		ease: Circ.easeOut
	}, "begin");
	...
	tl.timeScale(1.5);
		return tl;
	}

我们使用了`timeScale` 能够使场景1中的所有动画快`1.5`倍，如果我们想调慢动画，那么可以设置`timeScale` 小于`1` （例如`0.5`）这样所有的动画慢`0.5`倍。

### 循环

如果我们想让一个动画循环不断的执行，我们可以设置其 `repeat` 属性为 `-1`， 但是如果我只想让单独的某一整个场景无限循环呢？ 我们可以在该场景的时间轴加一个参数，来看看这个例子：

	function sceneOne() {
		var tl = new TimelineMax({ repeat: -1 });
		tl.add("begin");
		tl.to(".bubble", 2, {
			scale: 3,
			opacity: 0.5,
			rotation: 90,
			ease: Circ.easeOut
		}, "begin");
		...
		return tl;
	}

如果我们想让动画像悠悠球一样在初始状态和最终状态之间来回循环，可以用 `yoyos` （语法是设置一个布尔值：` yoyo: true`）。 `yoyo` 功能可以应用于单个`tweens` 动画，一整个子时间轴，或者 一整个主时间轴。但是只有时间轴被设置为循环播放的时候，`yoyos`才会起效果。 如果我们想给一整个主时间轴应用`yoyos` 功能，就需要在主时间轴上使用同样的方式：

	var master = new TimelineMax({repeat: -1, yoyo:true});

当某一个动画是反复运行的情况下，我们就会意识到声明一个明确的`delay`对于动画来说是多么有益的。 `delay: 1` 会对某一个动画轴下所有的动画都延迟1秒，而整个动画重新执行的时候不会暂停1秒。 而 `repeatDelay: 1` 则是针对整个动画执行的时候延迟1秒，而不对其他动画进行时延。

> #### 重复用法
> 
> 接下来讲到的方法对于 TimelineLite 和 TimelineMax 都适用，但是 循环 反复 或者 yoyos反复 都是只对TimelineMax起作用
> 
> 如果一段动画是循环或者重新被触发的， 你需要考虑将动画内 所有的 .to 方法 都替换为 .fromTo 方法。 这样你就能保证每一次动画执行的时候 都是从初始状态到结束状态。

你不仅可以通过设置 `repeat: -1` 的方式来循环动画，你还可以通过设置`onComplete`回调函数的形式来触发，如果每一次重复动画需要设置更多的状态或者你需要每一次循环都改变重复动画中的某一些参数（例如：某些随机动画中的随机数）。 不单单如此，如果你想让动画设置的更加精巧，回调函数会给你带来很多帮助， 让我们来看看回调方式循环动画的例子：

	function _flyBy(el, amt) {
		TweenMax.to(el, amt, {
			x: -200,
			rotation: 360,
			onComplete: this._flyBy,
			onCompleteParams: [el, amt]
		});
	}

如果我们想通过回调函数来设置一些随机的效果，我们可以这样做：

	function _flyBy(el) {
		TweenMax.to(el, amt, {
			x: Math.random() * 400 - 200,
			rotation: Math.random() * 360,
			onComplete: _flyBy,
			onCompleteParams: [el]
		});
	}

`onComplete` 允许我们设置一个回调函数（当我们的动画已经完成的时候就会执行）而 `onCompleteParams` 参数则允许我们以数组的形式传递参数给 `onComplete` 回调函数。通常来说，如果你只是循环一个随机数，编译器只会先产生一次随机数，然后不断的重复使用这个随机数，这样就不算是真的每次都随机了，而在上面的代码中，我们让每次动画一结束就重新执行`_flyBy` 函数都带上 `el`、 `amt` 和 可变的参数（编译器每一次都会产生一个新额随机数）。 所以这是TimelineMax 编译器较好的地方。

下面列出了有很多回调函数供我们在不同的动画生命周期中使用 其中有 例如 绑定回调函数作用域的回调函数 或者 当所有回调执行完后的回调函数。

这些可用的回调函数是：

- onStart
- onStartScope
- onStartParams
- onComplete
- onCompleteScope
- onCompleteParams
- onUpdate
- onUpdateScope
- onUpdateParams
- onRepeat
- onRepeatScope
- onRepeatParams
- onReverseComplete
- onReverseCompleteScope

`callbackScope` 可以为对应的回调函数绑定合适的上下文 其中包括： `onStartScope`, `onUpdateScope`, `onCompleteScope`, `onReverseCompleteScope`和 `onRepeatScope`这几种方法。

### 暂停 和 暂停事件

你也许还记得我们之前提到的 通过在 TimelineMax 以对象的形式设置 `repeat: -1` 可以让整个 `timeline` 结束后再次重新开始。 同样的你也可以设置 整个`timeline` 在初始状态下暂时停止：

	var master = new TimelineMax({paused: true});

如果当你想让 `timeline` 当页面一开始访问的时候暂时停止，而只有当点击的时候动画才会执行的时候， 可以这样写：

	var master = new TimelineMax({paused: true});
	...
	var el = document.getElementById("button")
		el.addEventListener('click', function(e) {
		e.preventDefault();
		master.restart();
	}, false);

非常简单！

当然，我们是可以在这里列出所有的事件的。（但没什么意义） 除此之外， 借助 一些工具 例如[Hammer.js](http://hammerjs.github.io/)（用于在PC端实现手机端手势的JS框架库） ，你就可以使用移动端的手势来触发动画了，例如就好像你在原生移动端的滑动，双击等等

在12章， 我会给大家讲述如何使用GreenSock 时间轴拖拽功能调节动画。我们同样可以使用 jQuery 提供的 sliderUI 来做一个对于控制动画时间轴的简洁界面的控制接口。 Chris Gannon 同学的 [ScrubGSAPTimeline] (https://s3-us-west-2.amazonaws.com/s.cdpn.io/35984/ScrubGSAPTimeline.js) 工具可以说是控制timeline 最棒的工具了。 这里我写了一个[例子](https://codepen.io/chrisgannon/details/zGmdBN/)来叫大家如何使用它。

### 其他的 关于 Timeline 的方法

GreenSock 的`timeline` 工具提供了太多的功能以至于一页纸都罗列不完，以下罗列的只是一部分你可用的功能，我们之前也介绍了我认为最基础的一些功能，剩下的很多有趣额功能就需要你自己去 [GreenSock 文档](https://greensock.com/docs/#/HTML5)去摸索了 或者 在 [GreenSock 论坛中](https://greensock.com/forums/) 和网友们讨论。 许多方法名字上看起来好像很直观简单，但实际上还有很多妙用， 文档会帮助你去更深更清楚的理解它们的用法。

请注意因为 由于ActionSCript 版本的GSAP 有很多(官网已经不维护AS版本的GSAP了)且看起来是在是太过老旧了。所以本书我们使用统一的 js 版本的GSAP进行介绍，这样可以简单为大家介绍细节。

#### TimelineLite and TimelineMax 通用的方法

- add()
- addLabel()
- addPause()
- call()
- clear()
- delay()
- duration()
- eventCallback
- exportRoot()
- from()
- fromTo()
- getChildren()
- getLabelTime()
- getTweensOf()
- invalidate()
- isActive()
- kill()
- pause()
- paused()
- play()
- progress()
- remove()
- removeLabel()
- render()
- restart()
- resume()
- reverse()
- reversed()
- seek()
- set()
- shiftChildren()
- staggerFrom()
- staggerFromTo()
- staggerTo()
- startTime()
- time()
- timeScale()
- to()
- totalDuration()
- totalProgress()
- totalTime()
- useFrames()

#### 只存在 TimelineMax 的方法

- currentLabel()
- getActive()
- getLabelAfter()
- getLabelBefore()
- getlLabelsArray()
- repeat()
- repeatDelay()
- tweenFromTo()
- tweenTo()
- yoyo()

现在，你基本上已经掌握了`timeline` 的基础了，并且`timeline`强大的功能已经向你开放了，我们下面就来投入到学习一些真正有趣且富含想象力的动画吧。