^title 架构，性能和游戏
^section Introduction

Before we plunge headfirst into a pile of patterns, I thought it might help to
give you some context about how I think about software architecture and how it
applies to games. It may help you understand the rest of this book better. If
nothing else, when you get dragged into an <span name="ammo">argument</span>
about how terrible (or awesome) design patterns and software architecture are,
it will give you some ammo to use.

在一头扎入模式之前，我想先讲一些我对软件架构和其应用于游戏的理解，
这也许能帮你更好的理解这本书的其余部分。
至少，在你被卷入一场关于软件架构有多么糟糕（或多么优秀）的<span name="ammo">辩论</span>时，
这可以给你一些武器支援。

<aside name="ammo">

注意我没有建议你在战斗中选哪一边。就像任何军火贩子一样，我愿意向作战双方出售武器。

</aside>

## What is Software Architecture?
## 什么是软件架构？

<span name="won't">If</span> you read this book cover to cover, you won't come
away knowing the linear algebra behind 3D graphics or the calculus behind game
physics. It won't show you how to alpha-beta prune your AI's search tree or
simulate a room's reverberation in your audio playback.

<span name="won't">如果</span>把本书从头到尾读一遍，
你不会找到在3D图形背后的线性代数或者游戏物理背后的微积分。
这书不会告诉你如何用α-β修剪你的AI树，也不会告诉你如何在音频中模拟房间中的混响。

<aside name="won't">

Wow，这段给这本书打了个糟糕的广告啊。

</aside>

Instead, this book is about the code *between* all of that. It's less about
writing code
than it is about *organizing* it. Every program has *some* organization, even if
it's just "jam the whole thing into `main()` and see what happens", so I think
it's more interesting to talk about what makes for *good* organization. How do
we tell a good architecture from a bad one?

相反，这本书告诉你在这些*之间*的事情。
这本书与其说是关于如何写代码。不如说是关于如何*架构*代码的。
每个程序都有*一定*架构，
哪怕是“将所有东西都塞到`main()`中然后看看发生了什么”，
所以讲讲什么造成了*好*架构是很有意思的。我们如何区分好架构和坏架构呢？

I've been mulling over this question for about five years. Of course, like you,
I have an intuition about good design. We've all suffered through codebases so
<span name="suffered">bad</span>, the best you could hope to do for them is take
them out back and put them out of their misery.

我思考这个问题五年了。当然，像你一样，我有关于好设计的直觉。我们都被<span name="suffered">糟糕的</span>代码折磨的不轻，唯一能做的事情就是删掉它们，结束它们的痛苦。

<aside name="suffered">

承认吧，我们中大多数都该对它们中的一些*负责*。

</aside>

A lucky few have had the opposite experience, a chance to work with beautifully
designed code. The kind of codebase that feels like a perfectly appointed luxury
hotel festooned with concierges waiting eagerly on your every whim. What's the
difference between the two?

只有很少的幸运者有相反的经验，
有机会在好好设计的代码库上工作。
那种代码库看上去是间豪华酒店，里面的门房随时准备满足你心血来潮的需求。
这两者之间的区别是什么呢？

### What is *good* software architecture?
### 什么是*好的*软件架构？

For me, good design means that when I make a change, it's as if the entire
program was crafted in anticipation of it. I can solve a task with just a few
choice function calls that slot in perfectly, leaving not the slightest ripple
on the placid surface of the code.

对于我来说，好的设计意味着当我改了点什么，
整个程序就好像正等着这种改动。
可以加入几个函数调用完成任务，同时丝毫不改变代码平静表面下的脉动。

That sounds pretty, but it's not exactly actionable. "Just write your code so
that changes don't disturb its placid surface." Right.

这听起来很棒，只是完全不可行。“把代码写到改变不会影响其平静表面。”够了。

Let me break that down a bit. The first key piece is that *architecture is about
change*. Someone has to be modifying the codebase. If no one is touching the
code -- whether because it's perfect and complete or so wretched no one will
sully their text editor with it -- its design is irrelevant. The measure of a
design is how easily it accommodates changes. With no changes, it's a runner who
never leaves the starting line.

让我们通俗些。
第一个关键点是*架构是有关于变化的*。
总有人改动代码。如果没人碰代码——无论是因为代码至善至美，还是糟糕透顶以至于没人会为了修改它而玷污自己的文本编辑器——那么它的架构设计就无关紧要。
评价架构设计就是评价它应对变化有多么轻松。
没有了变化，它就是永远不会离开起跑线的运动员。

### How do you make a change?
### 你如何处理变化？

Before you can change the code to add a new feature, to fix a bug, or for whatever
reason caused you to fire up your editor, you have to understand what the
existing code is doing. You don't have to know the whole program, of course, but
you need to <span name="ocr">load</span> all of the relevant pieces of it into
your primate brain.

在你改变代码去添加新特性，去修复漏洞，或者随便什么需要使用编辑器的时候，
你需要理解现在的代码在做些什么。当然，你不需要理解整个程序，
但你需要将所有相关的东西<span name="ocr">装进</span>你的灵长类大脑。

<aside name="ocr">

有点诡异，这字面上是一个OCR过程。

</aside>

We tend to gloss over this step, but it's often the most time-consuming part of
programming. If you think paging some data from disk into RAM is slow, try
paging it into a simian cerebrum over a pair of optical nerves.

我们通常无视了这步，但这往往是编程中最耗时的部分。
如果你认为将数据从磁盘上分页到RAM上很慢，
那么试着通过一对神经纤维将数据分页到猴脑中。

Once you've got all the right context into your wetware, you think for a bit and
figure out your solution. There can be a lot of back and forth here, but often
this is relatively straightforward. Once you understand the problem and the
parts of the code it touches, the actual coding is sometimes trivial.

一旦把所有正确的上下文都记到了你的湿件里，
想一会，然后找到解决方案。
这可能会有来回打转的时刻，但通常比较简单。
一旦你理解了问题和需要改动的代码，实际的编码工作有时是微不足道的。

You beat your meaty fingers on the keyboard for a while until the right colored
lights blink on screen and you're done, right? Not just yet! Before you write
<span name="tests">tests</span> and send it off for code review, you often have
some cleanup to do.

你用肥手指在键盘上敲打一阵，直到屏幕上显示着正确颜色的光芒，
然后就搞定了，对吧？还没呢！
在你为之写<span name="tests">测试</span>并发送到代码评审之前，你通常有些清理工作要做。

<aside name="tests">

我是不是说了“测试”？噢，是的，我说了。为有些游戏代码写单元测试很难，但代码库的大部分是完全可以测试的。

我不会在这里发表演说，但是我建议你，如果还没有做自动测试，请考虑一下。
除了手动验证以外你就没别的事要做了吗？

</aside>

You jammed a bit more code into your game, but you don't want the next person to
come along to trip over the wrinkles you left throughout the source. Unless the
change is minor, there's usually a bit of reorganization to do to make your new
code integrate seamlessly with the rest of the program. If you do it right, the
next person to come along won't be able to tell when any line of code was
written.

你将一些代码加入了游戏，但不想下一个人被你留下来的小问题绊倒。
除非改动很小，否则就还需要一些工作去微调新代码，使之无缝对接到程序的其他部分。如果做对了，那么下个见到代码的人甚至无法说出哪些代码是新加入的。

In short, the flow chart for programming is something like:

简而言之，编程的流程图看起来是这样的：

<img src="images/architecture-cycle.png" alt="Get problem &rarr; Learn code &rarr; Code solution &rarr; Clean up &rarr; and back around to the beginning." />

<span name="life-cycle"></span>

<aside name="life-cycle">

令人震惊的死循环，我看到了。

</aside>

### How can decoupling help?
### 解耦怎么帮了忙？

While it isn't obvious, I think much of software architecture is about that
learning phase. Loading code into neurons is so painfully slow that it pays to
find strategies to reduce the volume of it. This book has an entire section on
[*decoupling* patterns](decoupling-patterns.html), and a large chunk of *Design
Patterns* is about the same idea.

虽然并不明显，但我认为很多软件架构都是关于学习阶段。
将代码载入到神经元太过缓慢，找些策略减少载入的总量是很值得做的事。
这本书有整整一章是关于[*解耦*模式](decoupling-patterns.html)，
还有很多*设计模式*是关于同样的主题。

You can define "decoupling" a bunch of ways, but I think if two pieces of code
are coupled, it means you can't understand one without understanding the other.
If you *de*-couple them, you can reason about either side independently. That's
great because if only one of those pieces is relevant to your problem, you just
need to load *it* into your monkey brain and not the other half too.

可以用多种方式定义“解耦”，但我认为如果有两块代码是耦合的，
那就意味着无法只理解其中一个。
如果*解*耦了他俩，就可以独自的理解某一块。
这当然很好，因为只有一块与问题相关，
只需将*这一块*加载到你的猴脑中而不需要加载另外一块。

To me, this is a key goal of software architecture: **minimize the amount of
knowledge you need to have in-cranium before you can make progress.**

对我来说，下面是软件架构的关键目标：
*最小化在处理前需要进入大脑的知识*。

The later stages come into play too, of course. Another definition of decoupling
is that a *change* to one piece of code doesn't necessitate a change to another.
We obviously need to change *something*, but the less coupling we have, the less
that change ripples throughout the rest of the game.

当然，也可以从后期阶段来看。
另一种解耦的定义是：当一块代码有*变化*时，没必要修改另外的代码。
肯定需要修改*一些东西*，但耦合程度越小，变化会波及的范围就越小。

## At What Cost?
## 以什么代价？

This sounds great, right? Decouple everything and you'll be able to code like
the wind. Each change will mean touching only one or two select methods, and you
can dance across the surface of the codebase leaving nary a shadow.

听起来很棒，对吧？解耦任何东西，然后能像风一样编码。
每个变化都只修改一两个特定方法，在代码库的水面上跳舞，只留下倒影。

This feeling is exactly why people get excited about abstraction, modularity,
design patterns, and software architecture. A well-architected program really is
a joyful experience to work in, and everyone loves being more productive. Good
architecture makes a *huge* difference in productivity. It's hard to overstate
how profound an effect it can have.

这就是人们对抽象，模块化，设计模式和软件架构兴奋的原因。
在有好架构的程序上工作是很好的体验，每个人都希望能更有效率地工作。
好架构能造成生产力上*巨大的*不同。很难再夸大它那强力的影响。

But, like all things in life, it doesn't come free. Good architecture takes real
effort and discipline. Every time you make a change or implement a feature, you
have to work hard to integrate it gracefully into the rest of the program. You
have to take great care to both <span name="maintain">organize</span> the code
well and *keep* it organized throughout the thousands of little changes that
make up a development cycle.

但是，就像生活中的任何事物一样，没有免费的午餐。好的设计需要汗水和纪律。
每次做出改动或是实现特性，你都需要将它优雅的集成到程序的其他部分。
需要花费大量的努力去<span name="maintain">管理</span>代码，
在开发过程中面对数千次变化仍然*保持*它的管理结构。

<aside name="maintain">

这个的第二部分——管理代码——需要特别关注。我看到无数程序有个优雅的开始，然后死于程序员一遍又一遍添加的“微小黑魔法”。

就像园艺，仅仅增加新植物是不够的，还需要除草和修剪。

</aside>

You have to think about which parts of the program should be decoupled and
introduce abstractions at those points. Likewise, you have to determine where
extensibility should be engineered in so future changes are easier to make.

你得考虑程序的哪部分需要解耦，然后再引入抽象。
同样，你需要决定哪部分要设计得支持插件来方便未来的变化。

People get really excited about this. They envision future developers (or just
their future self) stepping into the codebase and finding it open-ended,
powerful, and just beckoning to be extended. They imagine The One Game Engine To
Rule Them All.

人们对这点变得狂热。
他们设想未来的开发者（或者只是未来的他们自己）进入代码库，
并发现它极为开放，
功能强大，只需插件。他们想要有“至尊代码应所求”。（译著：这里是“至尊魔戒御众戒”的梗，很遗憾翻译不出来）

But this is where it starts to get tricky. Whenever you add a layer of
abstraction or a place where extensibility is supported, you're *speculating*
that you will need that flexibility later. You're adding code and complexity to
your game that takes time to develop, debug, and maintain.

但是，这是开始棘手的部分。
每当你添加了一层抽象或者支持扩展的部分，是*赌*以后需要灵活性。
添加代码和复杂性到游戏中，这都需要时间来开发，调试和维护。

That effort pays off if you guess right and end up touching that code later. But
<span name="yagni">predicting</span> the future is *hard*, and when that
modularity doesn't end up being helpful, it quickly becomes actively harmful.
After all, it is more code you have to deal with.

如果你猜对了，后来使用了这些代码，那么功夫不负有心人。
但<span name="yagni">预测</span>未来*很难*，如果模块化最终无益，那就有害。
毕竟，你得处理更多的代码。

<aside name="yagni">

有些人喜欢简写为术语“YAGNI”——[You aren't gonna need it（你不需要那个）](http://en.wikipedia.org/wiki/You_aren't_gonna_need_it)——来对抗预测将来需求的强烈欲望。

</aside>

When people get overzealous about this, you get a codebase whose architecture
has spiraled out of control. You've got interfaces and abstractions everywhere.
Plug-in systems, abstract base classes, virtual methods galore, and all sorts of
extension points.

当过分关注这点时，你会得到失控的代码库。
接口和抽象无处不在。插件系统，抽象基类，虚方法，还有各种各样的扩展点。

It takes you forever to trace through all of that scaffolding to find some real
code that does something. When you need to make a change, sure, there's probably
an interface there to help, but good luck finding it. In theory, all of this
decoupling means you have less code to understand before you can extend it, but
the layers of abstraction themselves end up filling your mental scratch disk.

回溯所有的脚手架去找真正做事的代码就要消耗无尽的时间。
当需要改变，当然，有可能某个接口能帮上忙，但能不能找到就只能祝你好运了。
理论上，解耦意味着在修改代码之前需要了解的代码更少，
但抽象层本身就会填满心灵暂存磁盘。

Codebases like this are what turn people *against* software architecture, and
design patterns in particular. It's easy to get so wrapped up in the code itself
that you lose sight of the fact that you're trying to ship a *game*. The siren
song of extensibility sucks in countless developers who spend years working on
an "engine" without ever figuring out what it's an engine *for*.

像这样的代码库让人*反对*软件架构，特别是设计模式。
人们很容易沉浸在代码中而忽略要发布*游戏*的这点。
无数的开发者听着加强可扩展性的警报，花费多年时间制作“引擎”，
却没有搞清楚做引擎是*为了什么*。

## Performance and Speed
## 性能和速度

There's another critique of software architecture and abstraction that you hear
sometimes, especially in game development: that it hurts your game's
performance. Many patterns that make your code more flexible rely on virtual
dispatch, interfaces, pointers, messages, and <span name="templates">other
mechanisms</span> that all have at least some runtime cost.

软件架构和抽象有时会被批评，尤其是在游戏开发中: 它伤害了游戏的性能。
许多让代码更灵活的模式依靠虚拟调度、 接口、 指针、 消息，和<span name="templates">其他机制</span>，
它们都会消耗运行时成本。

<aside name="templates">

一个有趣的反面例子是C++中的模板。模板编程有时可以给你抽象接口而无需运行时开销。

这是灵活性的两极。当写代码调用类中的具体方法时，你在*写作*时修改类——硬编码了调用的是哪个类。但通过虚方法或接口，直到*运行*时才知道调用的类。这更加灵活但增加了运行时开销。

模板编程是在两者之间。在*编译时*初始化模板，决定调用哪些类。

</aside>

There's a reason for this. A lot of software architecture is about making your
program more flexible. It's about making it take less effort to change it. That
means encoding fewer assumptions in the program. You use interfaces so that your
code works with *any* class that implements it instead of just the one that does
today. You use <a href="observer.html" class="gof-pattern">observers</a> and <a
href="event-queue.html" class="pattern">messaging</a> to let two parts of the
game talk to each other so that tomorrow, it can easily be three or four.

还有一个原因。很多软件架构的目的是使程序更加灵活。
这让改变它需要较少的努力。编码时对程序有更少的假设。
你可以使用接口，让代码可与*任何*实现它的类交互，而不仅仅是*现在*写的类。
今天，你可以使用<a href="observer.html" class="gof-pattern">观察者</a>和<a href="event-queue.html" class="pattern">消息</a>让游戏的两部分交流，
而以后可以很容易地扩展为三个或四个部分相互交流。

But performance is all about assumptions. The practice of optimization thrives
on concrete limitations. Can we safely assume we'll never have more than 256
enemies? Great, we can pack an ID into a single byte. Will we only call a method
on one concrete type here? Good, we can statically dispatch or inline it. Are
all of the entities going to be the same class? Great, we can make a nice <a
href="data-locality.html" class="pattern">contiguous array</a> of them.

但性能与假设相关。实践优化基于确定的限制。
永远不会超过256种敌人吗？好，可以将ID编码为一个字节。
在一个具体类型中只调用一个方法吗？好，可以做静态调度或内联。
所有实体都是同一类？太好了，可以使用 <a href="data-locality.html" class="pattern">连续数组</a>存储它们。

This doesn't mean flexibility is bad, though! It lets us change our game
quickly, and *development* speed is absolutely vital for getting to a fun
experience. No one, not even Will Wright, can come up with a balanced game
design on paper. It demands iteration and experimentation.

但这并不意味着灵活性是坏的！它可以让我们快速改进游戏，
*开发*速度是获取有趣开发经验的绝对重要因素。
没有人，哪怕是Will Wright，能在纸面上构建一个平衡的游戏。这需要迭代和实验。

The faster you can try out ideas and see how they feel, the more you can try and
the more likely you are to find something great. Even after you've found the
right mechanics, you need plenty of time for tuning. A tiny imbalance can wreck
the fun of a game.

越快尝试想法，看看效果如何，就能尝试越多东西，就越可能找到有价值的东西。
就算找到正确的机制，也需要足够的时间调整。
一个微小的不平衡可能破坏整个游戏的乐趣。

There's no easy answer here. Making your program more flexible so you can
prototype faster will have some performance cost. Likewise, optimizing your code
will make it less flexible.

这没有银弹。
让你的程序更加灵活，在损失一点点性能的前提下更快地做出原型。
同样的，优化代码会让它不那么灵活。

My experience, though, is that it's easier to make a fun game fast than it is to
make a fast game fun. One compromise is to keep the code flexible until the
design settles down and then tear out some of the abstraction later to improve
your performance.

就我个人经验而言，让有趣的游戏变快比让快速的游戏变有趣简单得多。
一种折中的办法是保持代码灵活直到设计定下来，再抽出抽象层来提高性能。

## The Good in Bad Code
## 糟糕代码的优势

That brings me to the next point which is that there's a time and place for
different styles of coding. Much of this book is about making maintainable,
clean code, so my allegiance is pretty clearly to doing things the "right" way,
but there's value in slapdash code too.

这就来到了下一观点：不同的代码风格各有千秋。
这本书的大部分是关于保持干净可控的代码，所以我坚持应该用*正确*方式写代码，但糟糕的代码也有一定的优势。

Writing well-architected code takes careful thought, and that translates to
time. Moreso, *maintaining* a good architecture over the life of a project takes
a lot of effort. You have to treat your codebase like a good camper does their
campsite: always try to leave it a little better than you found it.

编写良好架构的代码需要仔细地思考，这会转为时间上的代价。
在项目的整个周期中*保持*良好的架构需要花费大量的努力。
你需要像露营者处理营地一样小心处理代码库：总是保持其优于你刚刚接触它的时候。

This is good when you're going to be living in and working on that code for a
long time. But, like I mentioned earlier, game design requires a lot of
experimentation and exploration. Especially early on, it's common to write code
that you *know* you'll throw away.

当你要在项目上花费很久时间的话，这很好。
但，就像早先提到的，游戏设计需要很多实验和探索。
特别是在早期，写一些你*知道*要扔掉的代码是很普遍的事情。

If you just want to find out if some gameplay idea plays right at all,
architecting it beautifully means burning more time before you actually get it
on screen and get some feedback. If it ends up not working, that time spent
making the code elegant goes to waste when you delete it.

如果只想试试游戏的某些主意是不是正确的，
良好的设计意味着在屏幕上看到和获取反馈之前要消耗很长时间。
如果最后证明这点子不对，那么删除代码时，那些花在让代码更优雅的时间就付之东流了。

Prototyping -- slapping together code that's just barely functional enough to
answer a design question -- is a perfectly legitimate programming practice.
There is a very large caveat, though. If you write throwaway code, you *must*
ensure you're able to throw it away. I've seen bad managers play this game time
and time again:

> Boss: "Hey, we've got this idea that we want to try out. Just a prototype, so
> don't feel you need to do it right. How quickly can you slap something
> together?"

> Dev: "Well, if I cut lots of corners, don't test it, don't document it, and it
> has tons of bugs, I can give you some temp code in a few days."

> Boss: "Great!"

*A few days pass...*

> Boss: "Hey, that prototype is great. Can you just spend a few hours cleaning
> it up a bit now and we'll call it the real thing?"

原型——一坨勉强拼凑在一起，只能回答设计问题的简单代码——是个完全合理的编程习惯。
虽然当你写一次性代码时，*必须*保证可以扔掉它。
我见过很多次糟糕的经理人在玩这种把戏：

> 老板：“嗨，我有些想试试的点子。只要原型，不需要做的很好。你能多快搞定？”

> 开发者：“额，如果删掉这些部分，不测试，不写文档，允许很多的漏洞，那么几天能给你临时的代码文件。”

> 老板：“太好了。”

*几天后*

> 老板：“嘿，原型很棒，你能花上几个小时清理一下然后变为成品吗？”

You need to make sure the people using the <span
name="throwaway">throwaway</span> code understand that even though it kind of
looks like it works, it *cannot* be maintained and *must* be rewritten. If
there's a *chance* you'll end up having to keep it around, you may have to
defensively write it well.

你得让人们清楚，<span name="throwaway">可抛弃</span>的代码即使看上去能工作，也不能被*维护*，*必须*重写。
如果*有可能*要维护这段代码，就得防御性好好编写它。

<aside name="throwaway">

一个保证原型代码不会变成真正用的代码的技巧是使用和游戏不同的编程语言。这样，在实际应用于游戏中之前必须重写。

</aside>

## Striking a Balance
## 保持平衡

We have a few forces in play:

1.  <span name="speed">We</span> want nice architecture so the code is easier to
    understand over the lifetime of the project.
2.  We want fast runtime performance.
3.  We want to get today's features done quickly.

有些因素在相互角力：

1. 为了在项目的整个生命周期保持其可读性，<span name="speed"></span>需要好架构。
2. 需要更好的运行时性能。
3. 需要让现在的特性更快的实现。

<aside name="speed">

有趣的是，这些都是速度：长期开发的速度，游戏运行的速度，和短期开发的速度。

</aside>

These goals are at least partially in opposition. Good architecture improves
productivity over the long term, but maintaining it means every change requires
a little more effort to keep things clean.

这些目标至少是部分对立的。
好架构长期来看提高了生产力，
也意味着维护每个变化都需要更多努力让代码保持整洁。

The implementation that's quickest to write is rarely the quickest to *run*.
Instead, optimization takes significant engineering time. Once it's done, it
tends to calcify the codebase: highly optimized code is inflexible and very
difficult to change.

草就的代码很少是*运行时*最快的。
相反，提升性能需要很多的编程时间。
一旦完成，它就会污染代码库：高度优化的代码不灵活，很难改动。

There's always pressure to get today's work done today and worry about
everything else tomorrow. But if we cram in features as quickly as we can, our
codebase will become a mess of hacks, bugs, and inconsistencies that saps our
future productivity.

总有今日事今日毕的压力。但是如果尽可能快地实现特性，
代码库就会充满黑魔法，漏洞和混乱，阻碍未来的产出。

There's no simple answer here, just trade-offs. From the email I get, this
disheartens a lot of people. Especially for novices who just want to make a
game, it's intimidating to hear, "There is no right answer, just different
flavors of wrong."

没有简单的解决方案，只有权衡。
从我收到的邮件看，这伤了很多人的心，特别是那些只是想做个游戏的人。
这似乎是在恐吓，“没有正确的答案，只有不同的错误。”

But, to me, this is exciting! Look at any field that people dedicate careers to
mastering, and in the center you will always find a set of intertwined
constraints. After all, if there was an easy answer, everyone would just do
that. A field you can master in a week is ultimately boring. You don't hear of
someone's distinguished career in <span name="ditch">ditch digging</span>.

但，对我来说，这让人兴奋！看看任何人们从事的领域，
你总能发现某些相互抵触的限制。无论如何，如果有简单的答案，每个人都会那么做。
一周就能掌握的领域是很无聊的。你从来没有听说过有人讨论<span name="ditch">挖坑事业</span>。

<aside name="ditch">

也许你会；我没有深究这个类比。
可能有挖坑热爱着，挖坑规范，以及一整套亚文化。
我算什么人，能在此大放厥词？

</aside>

To me, this has much in common with games themselves. A game like chess
can never be mastered because all of the pieces are so perfectly balanced
against one another. This means you can spend your life exploring the vast space
of viable strategies. A poorly designed game collapses to the one winning tactic
played over and over until you get bored and quit.

对我来说，这和游戏有很多相似之处。
国际象棋之类的游戏永远不能被掌握，因为每个棋子都很完美的与其他棋子相平衡。
这意味你可以花费一生探索可选的广阔策略。糟糕的游戏就像井字棋，你玩了几遍，厌烦了就退出。

## Simplicity
## 简单

Lately, I feel like if there is any method that eases these constraints, it's
*simplicity*. In my code today, I try very hard to write the cleanest, most
direct solution to the problem. The kind of code where after you read it, you
understand exactly what it does and can't imagine any other possible solution.

最近，我感觉如果有什么能简化这些限制，那就是*简单*。
在我现在的代码中，我努力去写最简单，最直接的解决方案。
你读过这种代码后，完全理解了它在做什么，想不出其他完成的方法。

I aim to get the data structures and algorithms right (in about that order) and
then go from there. I find if I can keep things simple, there's less code
overall. That means less code to load into my head in order to change it.

我的目标是正确获得数据结构和算法（大致是这样的先后），然后在从那里开始。
我发现如果能让事物变得简单，就有更少的代码，
就意味着改动时有更少的代码载入脑海。

It often runs fast because there's simply not as much overhead and not much code
to execute. (This certainly isn't always the case though. You can pack a lot of
looping and recursion in a tiny amount of code.)

它通常跑的很快，因为没什么开销，也没什么代码执行。
（虽然大部分时候事实并非如此。你可以在一小段代码里加入大量的循环和递归。）

However, note that I'm not saying <span name="simple">simple code</span> takes
less time to *write*. You'd think it would since you end up with less total
code, but a good solution isn't an accretion of code, it's a *distillation* of
it.

但是，注意我并没有说<span name="simple">简单的代码</span>需要更少的时间*编写*。
你会这么觉得是因为最终得到了更少的代码，但是好的解决方案不是往代码中注水，而是*蒸干*代码。

<aside name="simple">

Blaise Pascal有句著名的信件结尾，“我没时间写的更短。”

另一句名言来自Antoine de Saint-Exupery：“完美是可达到的，不是没有东西可以添加的时候，而是没有东西可以删除的时候。”

言归正传，我发现每次重写本章，它就更短。有些章节比刚完成时短了20%。

</aside>

We're rarely presented with an elegant problem. Instead, it's a pile of use
cases. You want the X to do Y when Z, but W when A, and so on. In other words, a
long list of different example behaviors.

我们很少遇到优雅表达的问题。取而代之的是一堆用况。
你想要X在Z情况下做Y，在A情况下做W，诸如此类。换言之，一长列不同行为。

The solution that takes the least mental effort is to just code up those use
cases one at a time. If you look at novice programmers, that's what they often
do: they churn out reams of conditional logic for each case that popped into
their head.

最不消耗心血的解决方法就是为每段用况编写一段代码。
看看新手程序员，他们经常这么干：
为每个手头的问题编写逻辑循环。

But there's nothing elegant in that, and code in that style tends to fall over
when presented with input even slightly different than the examples the coder
considered. When we think of elegant solutions, what we often have in mind is a
*general* one: a small bit of logic that still correctly covers a large space of
use cases.

但这一点也不优雅，那种风格的代码遇到一点点没想到的输入就会崩溃。
当我们想象优雅的代码时，我们想的是*通用*的那一个：
只需要很少的逻辑就可以覆盖整个用况。

Finding that is a bit like pattern matching or solving a puzzle. It takes effort
to see through the scattering of example use cases to find the hidden order
underlying them all. It's a great feeling when you pull it off.

找到这样的方法有点像模式识别或者解决谜题。
需要努力去识别散乱的用例下隐藏的规律。
完成时你会感觉好得不能再好。

## Get On With It, Already
## 就快完了

Almost everyone skips the introductory chapters, so I congratulate you on making
it this far. I don't have much in return for your patience, but I'll offer up a
few bits of advice that I hope may be useful to you:

几乎每个人都会跳过介绍章节，所以祝贺你看到这里。
我没有太多东西回报你的耐心，但还有些建议给你，希望对你有用：

*  Abstraction and decoupling make evolving your program faster and easier, but
   don't waste time doing them unless you're confident the code in question needs
   that flexibility.

 * 抽象和解耦让扩展代码更快更容易，但除非确信需要灵活性，否则不要做。

 *  <span name="think">Think</span> about and design for performance throughout
    your development cycle, but put off the low-level, nitty-gritty optimizations
    that lock assumptions into your code until as late as possible.

 * 在整个开发周期中<span name="think">考虑</span>并为性能设计，但是尽可能推迟那些底层的，基本事实的优化，那会锁死代码。

<aside name="think">

相信我，发布前两个月*不是*开始思考“游戏运行只有1FPS”的唠叨小问题的时候。

</aside>

 *   Move quickly to explore your game's design space, but don't go so fast that
     you leave a mess behind you. You'll have to live with it, after all.

 * 快速地探索游戏的设计空间，但不要跑的太快，在身后留下一团乱麻。毕竟，总得回来打扫。

 *   If you are going to ditch code, don't waste time making it pretty. Rock
     stars trash hotel rooms because they know they're going to check out the
     next day.

 * 如果打算抛弃这段代码，就不要尝试将其写完美。摇滚明星将旅店房间弄得一团糟，因为他们知道明天会有人来打扫干净。

 *   But, most of all, **if you want to make something fun, have fun making
     it.**

 * 但最重要的是，**如果你想要做出让人享受的东西，那就享受做它的过程。**
