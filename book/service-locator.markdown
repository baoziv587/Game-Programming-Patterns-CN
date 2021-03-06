^title Service Locator
^section Decoupling Patterns

^title 服务定位器
^section Decoupling Patterns

## Intent

## 意图

*Provide a global point of access to a service without coupling users to the
concrete class that implements it.*

*提供服务的全局接入点，而不必让用户和实现它的具体类耦合。*

## Motivation

## 动机

Some objects or systems in a game tend to get around, visiting almost every
corner of the codebase. It's hard to find a part of the game that *won't* need a
memory allocator, logging, or random numbers at some point.
Systems like those can be thought of as *services* that need to be available to
the entire game.

一些游戏中的对象或者系统趋向于到处访问，拜访程序库中机会每一个角落。
很难找到游戏中的一部分永远*不*需要内存分配，记录日志，或者随机数字。
像这样的东西可以被视为整个游戏都需要的*服务*。

For our example, we'll consider audio. It doesn't have quite the reach of
something lower-level like a memory allocator, but it still touches a bunch of
game systems. A falling rock hits the ground with a crash (physics). A sniper
NPC fires his rifle and a shot rings out (AI). The user selects a menu item with
a beep of confirmation (user interface).

在我们的例子中，我们考虑音频。
它不需要接触底层的东西，比如内存分配，但是仍然要接触一大堆游戏系统。
滚石撞击地面（物理）。
NPC狙击手开了一枪，射出子弹（AI）。
用户选择菜单项需要响一声确认（用户界面）。

Each of these places will need to be able to call into the audio system with
something like one of these:

这些中的每处都需要用像下面的东西调用音频系统：

^code 15

Either gets us where we're trying to go, but we stumbled into some sticky
coupling along the way. Every place in the game calling into our audio system
directly references the concrete `AudioSystem` class and the mechanism for
accessing it -- either as a static class or a <a class="gof-pattern"
href="singleton.html">singleton</a>.

每种都能让我们获得我们想要的，但是我们绊倒在一些微妙的耦合上。
每个调用音频系统的游戏部分直接引用了具体的`AudioSystem`类，和获取它的机制——是静态类还是一个<a class="gof-pattern" href="singleton.html">单例。</a>

These call sites, of course, have to be coupled to *something* in order to make a
sound play, but letting them poke at the concrete audio implementation directly
is like giving a hundred strangers directions to your house just so they can
drop a letter on your doorstep. Not only is it a little bit *too* personal, it's
a real pain when you move and you have to tell each person the new directions.

这些调用点，当然，需要耦合到*某些东西*上来播放声音，但是让他们可以接触到具体的音频实现，就好像给了一百个陌生人你家的方向，只是为了让他们在门口放一封信。
这不仅仅有关于*隐私*，也关于在你搬家后，需要告诉每个人新地址更加痛苦。

There's a better solution: a phone book. People that need to get in touch with
us can look us up by name and get our current address. When we move, we tell the
phone company. They update the book, and everyone gets the new address. In fact,
we don't even need to give out our real address at all. We can list a P.O. box
or some other "representation" of ourselves instead. By having callers go
through the book to find us, we have *a convenient single place where we control
how we're found.*

有个更好的解决办法：一本电话薄。
需要接触我们的人们可以在上面查找并找到现在的地址。
当我们搬家时，我们通知电话公司。
他们更新电话薄，每个人都知道了新地址。
事实上，我们甚至无需给出真实的地址。
我们可以列一个转发信箱或者其他“代表”我们的东西。
为了让调用者查询电话薄找到我们，我们需要一个*方便的地方控制我们可以被找到的地方*。

This is the Service Locator pattern in a nutshell -- it decouples code that needs
a service from both *who* it is (the concrete implementation type) and *where*
it is (how we get to the instance of it).

这就是果壳中的服务定位模式——它解耦了需要服务的代码和服务由*谁*提供（那个具体的实现类）以及服务在*哪里*（我们如何获得它的实例）。

## The Pattern

## 模式

A **service** class defines an abstract interface to a set of operations. A
concrete **service provider** implements this interface. A separate **service
locator** provides access to the service by finding an appropriate provider
while hiding both the provider's concrete type and the process used to locate
it.

**服务**类定义了一堆操作的抽象接口。
具体的**服务提供者**实现这个接口。
分离的**服务定位器**提供了通过查询合适的提供者，获取服务的方法，同时隐藏了提供者的具体细节和需要定位它的进程。

## When to Use It

## 何时使用

Anytime you make something accessible to every part of your program,
you're asking for trouble. That's the main problem with the <a
class="gof-pattern" href="singleton.html">Singleton</a> pattern, and this
pattern is no different. My simplest advice for when to use a service locator
is: *sparingly*.

任何你需要能够在程序的每处被能访问的东西时，你就是在找麻烦。
这是<a class="gof-pattern" href="singleton.html">单例</a>模式的主要问题，这个模式也没有什么不同。
我对何时使用服务定位其的最简单的建议是：*少用*。

Instead of using a global mechanism to give some code access to an object it
needs, first consider *passing the object to it instead*. That's dead simple, and
it makes the coupling completely obvious. That will cover most of your needs.

不要使用全局机制让某些代码接触它需要的代码，首先考虑*将对象传给它*。
这超简单，也明显保持了解耦。这会覆盖你大部分的需求。

*But...* there are some times when manually passing around an object is
gratuitous or actively makes code harder to read. Some systems, like logging or
memory management, shouldn't be part of a module's public API. The parameters to
your rendering code should have to do with *rendering*, not stuff like logging.

*但是……*这里有时候手动传入对象是不可能的或者会让代码难以阅读。
游戏系统，比如日志或内存管理，不该是模块公开API的一部分。
传给渲染代码的参数应该与*渲染*相关，而不是与日志之类的相关。

Likewise, other systems represent facilities that are fundamentally singular in
nature. Your game probably only has one audio device or display system that it
can talk to. It is an ambient property of the environment, so plumbing it
through ten layers of methods just so one deeply nested call can get to it is
adding needless complexity to your code.

同样，代表了设施的其他系统通常只有一个。
你的游戏可能只有一个你可以交流的音频设备或者显示设备。
这是周围环境的属性，所以加入穿透十层方法的复杂调用会为代码增加不必要的复杂度。

In those kinds of cases, this pattern can help. As we'll see, it functions as a
more flexible, more configurable cousin of the Singleton pattern. When used <span
name="well">well</span>, it can make your codebase more flexible with little
runtime cost.

这些例子中，这个模式可以帮忙。
就像我们将看到的那样，它是单例模式更加灵活，更加可配置的子孙。
当用得<span name="well">好</span>，它能以很小的运行时开销，获得更多的灵活性。

<aside name="well">

Conversely, when used poorly, it carries with it all of the baggage of the
Singleton pattern with worse runtime performance.

相反，当用得不好，它带来了所有单例模式的负面以及更多的运行时开销。

</aside>

## Keep in Mind

## 记住

The core difficulty with a service locator is that it takes a dependency -- a
bit of coupling between two pieces of code -- and defers wiring it up until
runtime. This gives you flexibility, but the price you pay is that it's harder
to understand what your dependencies are by reading the code.

使用服务定位的核心难点是它将从属——在两块代码之间的一点耦合——推迟到运行时再连接。
这给了更多的灵活度，但是付出的代价是更难在阅读代码时理解你依赖的是什么。

### The service actually has to be located

### 服务必须真的可定位

With a singleton or a static class, there's no chance for the instance we need
to *not* be available. Calling code can take for granted that it's there. But
since this pattern has to *locate* the service, we may need to handle cases
where that fails. Fortunately, we'll cover a strategy later to address this and
guarantee that we'll always get *some* service when you need it.

通过使用单例或者静态类，我们需要的实例不可能*不*可用。
调用代码保证了它就在那里。但是由于这个模式*定位*了服务，我们也许要处理失败的情况。
幸运的是，我们之后会介绍一种处理它的策略，保证我们总是在需要时能获得*某些*服务。

### The service doesn't know who is locating it

### 服务不知道谁在定位它

Since the locator is globally accessible, any code in the game could be
requesting a service and then poking at it. This means that the service must be able
to work correctly in any circumstance. For example, a class that expects to
be used only during the simulation portion of the game loop and not during rendering
may not work as a service -- it wouldn't be able to ensure that it's being used
at the right time. So, if a class expects to be used only in a certain context,
it's safest to avoid exposing it to the entire world with this pattern.

由于定位器是全局可获得的，任何游戏中的代码都可以请求服务，然后使用它。
这就意味着服务必须在任何环境下正确工作。
举个例子，只能在游戏循环的模拟部分使用，而不能在渲染部分使用的类也许不能作为服务——不能保证在正确的时间使用它。
所以，如果你的类期望只在特定上下文中使用，不用这个模式将它暴露给整个世界更安全。

## Sample Code

## 示例代码

Getting back to our audio system problem, let's address it by exposing the
system to the rest of the codebase through a service locator.

重回我们的音频问题，让我们将代码通过服务定位器暴露给代码库的剩余部分来解决问题。

### The service

### 服务

We'll start off with the audio API. This is the interface that our service will
be exposing:

我们从音频API开始。这是我们服务要暴露的接口：

^code 9

A real audio engine would be much more complex than this, of course, but this
shows the basic idea. What's important is that it's an abstract interface class
with no implementation bound to it.

当然，一个真实的音频引擎比这复杂的多，但这展示了基本的理念。
重要的是它是没有实现绑定的抽象接口类。

### The service provider

### 服务提供者

By itself, our audio interface isn't very useful. We need a concrete
implementation. This book isn't about how to write audio code for a game
console, so you'll have to imagine there's some actual code in the bodies of
these functions, but you get the idea:

只有它自己，我们的音频接口不是很有用。
我们需要具体实现。这本书不是关于如何为游戏主机写音频代码，所以你得想象这些函数中有实际的代码，但你知道什么意思：

^code 10

Now we have an interface and an implementation. The remaining piece is the
service locator -- the class that ties the two together.

现在我们有接口和实现了。
剩下的部分是服务定位器——那个将两者绑在一起的类

### A simple locator

### 一个简单的定位器

The implementation here is about the simplest kind of service locator you can
define:

这里的实现是你可以定义的最简单的服务定位器：

<span name="di"></span>

<span name="di"></span>

^code 1

<aside name="di">

The technique this uses is called *dependency injection*, an awkward bit of
jargon for a very simple idea. Say you have one class that depends on another.
In our case, our `Locator` class needs an instance of the `Audio` service.
Normally, the locator would be responsible for constructing that instance itself.
Dependency injection instead says that outside code is responsible for
*injecting* that dependency into the object that needs it.

这里用的技术被称为*依赖注入*，一个对简单思路的笨拙行话。
假设你有一个类依赖另一个。
在例子中，我们的`Locator`类需要`Audio`的实例。
通常，定位器负责构造那个实例。
依赖注入与之相反，它说外部代码负责向对象*注入*它需要的依赖。

</aside>

The static `getAudio()` function does the locating. We can call it from
anywhere in the codebase, and it will give us back an instance of our `Audio`
service to use:

这个静态的`getAudio()`函数完成了定位。
我们可以从代码库的任何地方调用它，它会给我们一个`Audio`服务实例使用：

^code 5

The way it "locates" is very simple -- it relies on some outside code to register
a service provider before anything tries to use the service. When the game is
starting up, it calls some code like this:

它“定位”的方式十分简单——依靠在使用服务前注册服务提供者的外部代码。
当游戏开始时，它调用一些这样的代码：

^code 11

The key part to notice here is that the code that calls `playSound()` isn't aware of
the concrete `ConsoleAudio` class; it only knows the abstract `Audio` interface. Equally
important, not even the *locator* class is coupled to the concrete service
provider. The *only* place in code that knows about the actual concrete class is
the initialization code that provides the service.

这里需要注意的关键部分是调用`playSound()`的代码没有意识到任何具体的`ConsoleAudio`类；
它只知道抽象的`Audio`接口。
同样重要的，*定位器*类都没有与具体的服务提供者耦合。
代码中*唯一*知道哪个具体类提供了服务的地方是初始化代码。

There's one more level of decoupling here: the `Audio` interface isn't aware of
the fact that it's being accessed in most places through a service locator. As
far as it knows, it's just a regular abstract base class. This is useful because
it means we can apply this pattern to *existing* classes that weren't
necessarily designed around it. This is in contrast with <a class="gof-pattern"
href="singleton.html">Singleton</a>, which affects the design of the "service"
class itself.

这里有更高层次的解耦：
`Audio`接口没有意识到它通过服务定位器来接受访问。
据它所知，它只是常见的抽象基类。
这很有用，因为这意味着我们可以将这个模式应用到*现有的*类上，这个类无需为此特殊设计。
这与<a class="gof-pattern" href="singleton.html">单例</a>形成了对比，那个会影响“服务”类本身的设计。

### A null service

### 一个空服务

Our implementation so far is certainly simple, and it's pretty flexible too. But
it has one big shortcoming: if we try to use the service <span
name="temporal">before</span> a provider has been registered, it returns `NULL`.
If the calling code doesn't check that, we're going to crash the game.

我们现在的实现很简单，而且也很灵活。
但是它有巨大的缺点：如果我们在提供者注册<span name="temporal">前</span>使用服务，它会返回`NULL`。
如果调用代码没有检查，我们就让游戏崩溃了。

<aside name="temporal">

I sometimes hear this called "temporal coupling" -- two separate pieces of code
that must be called in the right order for the program to work correctly. All
stateful software has some degree of this, but as with other kinds of coupling,
reducing temporal coupling makes the codebase easier to manage.

我有时听说这被称为“暂时耦合”——两块分离的代码必须以正确的顺序调用，才能让程序正确运行。
有状态的软件都有某种程度上的这种情况，但是就像其他耦合一样，减少暂时耦合让代码库更容易管理。

</aside>

Fortunately, there's another design pattern called "Null Object" that we can use
to address this. The basic idea is that in places where we would return `NULL`
when we fail to find or create an object, we instead return a special object
that implements the same interface as the desired object. Its implementation
basically does nothing, but it allows code that receives the object to safely
continue on as if it had received a "real" one.

幸运的是，还有一种设计模式叫做“空对象”，我们可用它处理这个。
这里的基本思路就是在我们没能找到或者没有让程序以正确运行的顺序调用时，我们不返回`NULL`，而是返回一个特定的，实现了请求对象一样的接口的对象。它的实现什么也不做，但是它允许代码接受对象，保证代码安全运行，好像代码收到了“真的”服务对象。

To use this, we'll define another "null" service provider:

为了使用它，我们定义另一个“空”服务提供者：

^code 7

As you can see, it implements the service interface, but doesn't actually do
anything. Now, we change our locator to this:

就像你看到的那样，它实现了服务接口，但是没有干任何实事。
现在，我们将我们的定位器改成这样：

<span name="ref"></span>

<span name="ref"></span>

^code 8

<aside name="ref">

You may notice we're returning the service by reference instead of by pointer
now. Since references in C++ are (in theory!) never `NULL`, returning a
reference is a hint to users of the code that they can expect to always get a
valid object back.

你也许注意我们用引用而不是指针返回服务。
由于C++中的引用（理论上）永远不是`NULL`，返回引用是提示用户他们总可以期待获得一个合法的对象。

The other thing to notice is that we're checking for `NULL` in the `provide()`
function instead of checking for the accessor. That requires us to call `initialize()` early
on to make sure that the locator initially correctly defaults to the null
provider. In return, it moves the branch out of `getAudio()`, which will save us
a couple of cycles every time the service is accessed.

另一件值得注意的事是我们在`provide()`而不是访问者中检查`NULL`。
那需要我们我们早早调用`initialize()`保证定位器可以正确默认找到空提供者。
作为回报，它将分支移出了`getAudio()`，这在每次使用服务时节约了检查开销。

</aside>

Calling code will never know that a "real" service wasn't found, nor does it
have to worry about handling `NULL`. It's guaranteed to always get back a valid
object.

调用代码永远不知道“真正的”服务没找到，也不必担心处理`NULL`。
这保证它永远能获得有效的对象。

This is also useful for *intentionally* failing to find services. If we want to
disable a system temporarily, we now have an easy
way to do so: simply don't register a provider for the service, and the locator
will default to a null provider.

这对*故意*找不到服务也很有用。
如果我们想暂时<span name="disable">停用</span>系统，现在有更简单的方式来做了：
简单的不要为服务注册提供者，定位器会默认提供空提供者。

<aside name="disable">

Turning off audio is handy during development. It frees up some memory and CPU
cycles. More importantly, when you break into a debugger just as a loud sound
starts playing, it saves you from having your eardrums shredded. There's nothing
like twenty milliseconds of a scream sound effect looping at full volume to get
your blood flowing in the morning.

关闭音频在开发中是很便利的。它释放了一些内存和CPU循环。
更重要的，但当你在响声初起时进入debugger，它能防止你的鼓膜爆裂。没有什么东西比二十毫秒的最高音量尖叫循环更你让你血液逆流了。

</aside>

### Logging decorator

### 日志装饰器

Now that our system is pretty robust, let's discuss another refinement this
pattern lets us do -- decorated services. I'll explain with an example.

现在我们的系统非常强健了，让我们讨论这个模式允许的另一个好处——装饰服务。
我会用例子解释。

During development, a little logging when interesting events occur can help you
figure out what's going on under the hood of your game engine. If you're working
on AI, you'd like to know when an entity changes AI states. If you're the sound
programmer, you may want a record of every sound as it plays so you can check
that they trigger in the right order.

在开发过程中，记录有趣事情发生的小小日志系统可助你查出游戏引擎正处于何种状态。
如果你在处理AI，你要知道哪个实体改变了AI状态。
如果你是音频程序员，你也许想记录每个播放的声音，这样你可以检查它们是否是以正确的顺序触发。

The typical solution is to litter the code with calls to some `log()`
function. Unfortunately, that replaces one problem with another -- now we have
*too much* logging. The AI coder doesn't care when sounds are playing,
and the sound person doesn't care about AI state transitions, but now they both
have to wade through each other's messages.

通常的解决方案是向代码中丢一些对`log()`函数的调用。
不幸的是，这用一个问题代替了另一个——现在我们有*太多*日志。
AI程序员不关心什么时候声音在播放，声音程序员也不在乎AI状态转换，但是它们现在都得在对方的日志中跋涉。

Ideally, we would be able to selectively enable logging for just the stuff we
care about, and in the final game build, there'd be no logging at all. If the
different systems we want to conditionally log are exposed as services, then we
can solve this using the <a class="gof-pattern"
href="http://www.c2.com/cgi/wiki?DecoratorPattern">Decorator</a> pattern. Let's
define another audio service provider implementation like this:

理念上，我们应该可以选择性的为关心的事物启动日志，而游戏成品中，不应该有日志。
如果有条件地为不同的系统记录日志被改写为服务，那么我们就可以用<a class="gof-pattern" href="http://www.c2.com/cgi/wiki?DecoratorPattern">装饰器</a>模式。让我们定义另一个音频服务提供者实现如下：

^code 12

As you can see, it wraps another audio provider and exposes the same interface.
It forwards the actual audio behavior to the inner provider, but it also logs each
sound call. If a programmer wants to enable audio logging, they call this:

就像你看到的那样，它包裹了另一个音频提供者，暴露同样的接口。
它将实际的音频行为转发给内部的提供者，但它也同时记录每个音频调用。
如果程序员需要启动音频日志，它们调用这个：

^code 13

Now, any calls to the audio service will be logged before continuing as before.
And, of course, this plays nicely with our null service, so you can both
*disable* audio and yet still log the sounds that it *would* play if sound were
enabled.

现在，任何在音频服务照旧运行前，对它的调用都会记录下去。
同时，当然，它和我们的空服务也很好的相处，所以你能*启用*音频，也能继续记录如果声音被启用*将会*播放的声音。

## Design Decisions

## 设计决策

We've covered a typical implementation, but there are a couple of ways that it can
vary based on differing answers to a few core questions:

我们包含了一种典型的实现，但是基于对核心问题的回答，这里还有很多实现方式：

### How is the service located?

### 服务是如何被定位的？

 *  **Outside code registers it:**

 *  **外部代码注册它：**

    This is the mechanism our sample code uses to locate the service, and it's the
    most common design I see in games:

    这是我们样例代码中定位服务使用的机制，这也是我在游戏中最常见的设计方式：

     *  *It's fast and simple.* The `getAudio()` function simply returns a
        pointer. It will often get inlined by the compiler, so we get a nice
        abstraction layer at almost no performance cost.

     *  *简单快捷。*`getAudio()`函数简单的返回指针。这通常被编译器内联了，所以我们几乎没有付出性能损失就获得了很好的抽象层。

     *  *We control how the provider is constructed.* Consider a service for
        accessing the game's controllers. We have two concrete providers: one
        for regular games and one for playing online. The online provider passes
        controller input over the network so that, to the rest of the game,
        remote players appear to be using local controllers.

     *  *可以控制提供者是如何被构建的。*想想一个接触游戏控制器的服务。我们使用两个具体的提供者：一个是给常规游戏，另一个给在线游戏。在线的那个跨越网络提供控制器的输入，这样，对于游戏的其他部分，远程玩家好像在使用本地控制器。

        To make this work, the online concrete provider needs to know the IP
        address of the other remote player. If the locator itself was
        constructing the object, how would it know what to pass in? The `Locator`
        class doesn't know anything about online at all, much less some other
        user's IP address.

        为了让其工作，在线的具体提供者需要知道其他远程玩家的IP。如果定位器本身构建对象，它怎么知道传进来什么？`Locator`类不知道任何有关在线的东西，就更不用说其他用户的IP地址了。

        Externally registered providers dodge the problem. Instead of the
        locator constructing the class, the game's networking code instantiates
        the online-specific service provider, passing in the IP address it
        needs. Then it gives that to the locator, who knows only about the
        service's abstract interface.

        外部注册的提供者闪避了这个问题。定位器不再构造类，游戏的网络代码实例化特定的在线特定服务提供器，传给它需要的IP地址。然后它把服务提供给定位器，定位器只知道服务的抽象接口。

     *  *We can change the service while the game is running.* We may not use
        this in the final game, but it's a neat trick during development. While
        testing, we can swap out, for example, the audio service with the null
        service we talked about earlier to temporarily disable sound while the
        game is still running.

     *  *可以在游戏运行时改变服务。*我们也许不会在最终的游戏版本中使用这个，但是这是个在开发过程中有效的技巧。举个例子，在测试时，即使游戏正在运行，我们可以切换音频服务为早先提到的空服务来临时地关闭声音。

     *  *The locator depends on outside code.* This is the downside. Any code
        accessing the service presumes that some code somewhere has already
        registered it. If that initialization doesn't happen, we'll either crash
        or have a service mysteriously not working.

     *  *定位器依赖外部代码。*这就是缺点。任何接触服务的代码必须假定某处的代码已经注册过了。如果没有做初始化，我们要么会崩溃，要么有服务迷之不工作。

 *  **Bind to it at compile time:**

 *  **在编译时绑定：**

    The idea here is that the "location" process actually occurs at compile time
    using preprocessor macros. Like so:

    这里的思路是使用预处理器，在编译时间处理“位置”。就像这样：

    ^code 2

    Locating the service like this implies a few things:

    像这样定位服务暗示了一些事情：

     *  *It's fast.* Since all of the real work is done at compile time, there's
        nothing left to do at runtime. The compiler will likely inline the
        `getAudio()` call, giving us a solution that's as fast as we could hope
        for.

     *  *快速。*由于所有的工作都在编译时做完了，没有东西需要在运行时做了。编译器很可能会内联`getAudio()`调用，给我们了尽可能快的解决方案。

     *  *You can guarantee the service is available.* Since the locator owns the
        service now and selects it at compile time, we can be assured that if
        the game compiles, we won't have to worry about the service being
        unavailable.

     *  *可以保证服务是可用的。*由于定位器现在拥有服务，在编译时就选择了，我们可以保证如果游戏编译完成，我们不必担心服务不可用。

     *  *You can't change the service easily.* This is the major downside. Since
        the binding happens at build time, anytime you want to change the
        service, you've got to recompile and restart the game.

     *  *无法轻易改变服务。*这是主要的缺点。由于绑定发生在编译时间，任何时候你想要改变服务，都得重新编译游戏并重启游戏。

 *  **Configure it at runtime:**

 *  **在运行时设置：**

    Over in the khaki-clad land of enterprise business software, if you say
    "service locator", this is what they'll have in mind. When the service is
    requested, the locator does some <span name="reflection">magic</span> at
    runtime to hunt down the actual implementation requested.

    在卡其色衣物覆盖的事业商业单位，如果你说“服务定位器”，这就是他们脑中的东西。当服务被请求时，定位器在运行时做一些<span name="reflection">魔法般</span>的事情来追踪请求的真实实现。

    <aside name="reflection">

    *Reflection* is a capability of some programming languages to interact with
    the type system at runtime. For example, we could find a class with a given
    name, find its constructor, and then invoke it to create an instance.

    *反射*是一些编程语言在运行时与类型系统打交道的能力。举个例子，我们可以用名字找到类，找到它的构造器，然后用它创建实例。

    Dynamically typed languages like Lisp, Smalltalk, and Python get this by
    their very nature, but newer static languages like C# and Java also support
    it.

    像Lisp，Smalltalk和Python这样的动态类型语言自然有这样的特性，但新的静态语言比如C#和Java同样支持它。

    </aside>

    Typically, this means loading a configuration file that identifies the
    provider and then using reflection to instantiate that class at runtime. This
    does a few things for us:

    典型的，这意味着加载设置文件，确认提供者，然后使用反射说明运行中的类。这为我们做了一些事情：

     *  *We can swap out the service without recompiling.* This is a little more
        flexible than a compile-time-bound service, but not quite as flexible as
        a registered one where you can actually change the service while the
        game is running.

     *  *我们可以换出服务而无需重新编译。*这比编译时绑定多了小小的灵活性，但是不像注册那样灵活，那里你可以真正的在运行游戏的时候改变服务。

     *  *Non-programmers can change the service.* This is nice for when the
        designers want to be able to turn certain game features on and off but
        aren't comfortable mucking through source code. (Or, more likely, the
        *coders* aren't comfortable with them mucking through it.)

     *  *非程序员也可改变服务。*这对想要在开关特定游戏特性的设计者是很好的，但清理源代码就不怎么舒服了。（或者，更可能的，*编程者*在设计者介入时就不舒服。）

     *  *The same codebase can support multiple configurations simultaneously.*
        Since the location process has been moved out of the codebase entirely,
        we can use the same code to support multiple service configurations
        simultaneously.

     *  *同样的代码库可以同时支持多种设置。*由于定位处理被从代码库中完全移出，我们可以使用相同的代码来同时支持多种服务设置。

        This is one of the reasons this model is appealing over in enterprise
        web-land: you can deploy a single app that works on different server
        setups just by changing some configs. Historically, this was less
        useful in games since console hardware is pretty well-standardized, but
        as more games target a heaping hodgepodge of mobile devices, this is
        becoming more relevant.

        这就是这个模型在企业网站上广泛应用的原因之一：你可以在不同的服务器上发布相同的应用，只需要修改设置。历史上看来，这在游戏中没什么用，因为主机硬件本身是好好标准化了的，但是对于很多目标是大杂烩的移动设备，这点就很有关系了。

     *  *It's complex.* Unlike the previous solutions, this one is pretty
        heavyweight. You have to create some configuration system, possibly
        write code to load and parse a file, and generally *do some stuff* to
        locate the service. Time spent writing this code is time not spent on
        other game features.

     *  *复杂。*不像前面的解决方案，这个是重量级的。你得创建设置系统，也许要写代码来加载和粘贴文件，通常*要做些事情*来定位服务。花费在写这些代码上的时间就是没有花费在写其他游戏特性的时间。

     *  *Locating the service takes time.* And now the smiles really turn to
        frowns. Going with runtime configuration means you're burning some CPU
        cycles locating the service. Caching can minimize this, but that still
        implies that the first time you use the service, the game's got to go
        off and spend some time hunting it down. Game developers *hate* burning
        CPU cycles on something that doesn't improve the player's game
        experience.

     *  *加载服务需要时间。*现在笑脸要变成皱眉了。在运行时设置意味着你在消耗CPU循环加载服务。缓存可以最小化它，但是仍暗示在首次使用服务时，游戏需要暂停等点时间追上它。游戏开发者*讨厌*消耗CPU循环在不能提高游戏体验的地方。

### What happens if the service can't be located?

### 如果服务不能被定位怎么办？

 *  **Let the user handle it:**

 *  **让用户处理它：**

    The simplest solution is to pass the buck. If the locator can't find the
    service, it just returns `NULL`. This implies:

    最简单的解决方案就是把责任退回去。如果定位器不能找到服务，只需返回`NULL`。这暗示着：

     *  *It lets users determine how to handle failure.* Some users may consider
        failing to find a service a critical error that should halt the game.
        Others may be able to safely ignore it and continue. If the locator
        can't define a blanket policy that's correct for all cases, then passing
        the failure down the line lets each call site decide for itself what the
        right response is.

     *  *让用户决定如何掌控失败。*一些用户也许在收到找不到服务的关键错误时应该暂停游戏。其他的可能会安全地忽视并继续。如果定位器不能定义所有的政策对所有的情况都正确，那么就将失败传递下去，让每个调用点决定什么是正确的回应。

     *  *Users of the service must handle the failure.* Of course, the corollary
        to this is that each call site *must* check for failure to find the
        service. If almost all of them handle failure the same way, that's a lot
        duplicate code spread throughout the codebase. If just one of the
        potentially hundreds of places that use the service fails to make that
        check, our game is going to crash.

     *  *使用服务的用户必须处理失败。*当然，这个的必然结果是每个调用点都*必须*检查寻找服务的失败。如果它们都以相同方式来处理，在代码库中就有很多重复的代码。如果一百个中有一个忘了检查，我们的游戏就会崩溃。

 *  **Halt the game:**

 *  **挂起游戏：**

    I said that we can't *prove* that the service will always be available at
    compile-time, but that doesn't mean we can't *declare* that availability is
    part of the runtime contract of the locator. The simplest way to do this is
    with an assertion:

    我说过我们不能*保证*服务在编译时总是可用的，但是不意味着我们不能*声明*可用性是游戏定位器运行合约的一部分。做这点最简答的方法就是使用断言：

    ^code 4

    If the service isn't located, the game stops before any subsequent code
    tries to use it. The <span name="assert">`assert()`</span> call there doesn't solve the problem of
    failing to locate the service, but it does make it clear whose problem it
    is. By asserting here, we say, "Failing to locate a service is a bug in the
    locator."

    如果服务没有被找到，游戏停在试图使用它的后续代码之前。这里的<span name="assert">`assert()`</span>调用没有解决无法定位服务的问题，但是它确实明确了问题是什么。通过这里的断言，我们表明，“无法定位服务是定位器的漏洞。”

    <aside name="assert">

    The <a href="singleton.html" class="pattern">Singleton</a> chapter
    explains the `assert()` function if you've never seen it before.

    如果你没见过，<a href="singleton.html" class="pattern">单例</a>模式一章中解释了`assert()`函数。

    </aside>

    So what does this do for us?

    那么这件事为我们做了什么呢？

     *  *Users don't need to handle a missing service.* Since a single service
        may be used in hundreds of places, this can be a significant code
        saving. By declaring it the locator's job to always provide a service,
        we spare the users of the service from having to pick up that slack.

     *  *用户不必处理缺失的服务。*由于简单的服务也许在成百上千的地方使用，这是节约了很多的代码。总能提供服务说明缺失是定位器的工作，我们节约了服务的用户处理它的精力。

     *  *The game is going to halt if the service can't be found.* On the off
        chance that a service really can't be found, the game is going to halt.
        This is good in that it forces us to address the bug that's preventing
        the service from being located (likely some initialization code isn't
        being called when it should), but it's a real drag for everyone else
        who's blocked until it's fixed. With a large dev team, you can incur
        some painful programmer downtime when something like this breaks.

     *  *如果服务没有找到，游戏会挂起。*在极小的可能性下，服务真的找不到，游戏就会挂起。强迫我们解决定位服务的漏洞是好事（比如一些本该调用的初始化代码没有被调用），但对被阻塞的所有人都会拖延到修复时。和大的开发团队工作，当这种事情发生时，你会增加痛苦的编程时间。

 *  **Return a null service:**

 *  **返回空服务：**

    We showed this refinement in our sample implementation. Using this means:

    我们在样例中实现中展示了这种修复。使用它意味着：

     *  *Users don't need to handle a missing service.* Just like the previous
        option, we ensure that a valid service object will always be returned,
        simplifying code that uses the service.

     *  *用户不必处理缺失的服务。*就像前面的选项一样，我们保证了可用的服务总是会被返回，简化了使用服务的代码。

     *  *The game will continue if the service isn't available.* This is both a
        boon and a curse. It's helpful in that it lets us keep running the game
        even when a service isn't there. This can be really helpful on a large
        team when a feature we're working on may be dependent on some other
        system that isn't in place yet.

     *  *如果服务不可用，游戏仍将继续。*这既有好处又有坏处。让我们在没有服务的情况下依然运行游戏是很有用的。在大团队中，当我们工作依赖的其他特性或者依赖的其他系统还没有就位时，这也是很有用的。

        The downside is that it may be harder to debug an *unintentionally*
        missing service. Say the game uses a service to access some data and
        then make a decision based on it. If we've failed to register the real
        service and that code gets a null service instead, the game may not
        behave how we want. <span name="null">It</span> will take some work to
        trace that issue back to the fact that a service wasn't there when we
        thought it would be.

        缺点在于，较难为*无意*缺失的服务查找漏洞。假设游戏用服务去获取数据，然后基于数据做出决策。如果我们无法注册真正的服务，代码获得了空服务，游戏也许不会像期望的那样行动。<span name="null">花一些时间</span>在这个问题上，才能发现我们以为有的服务是不存在的。

        <aside name="null">

        We can alleviate this by having the null service print some debug output
        whenever it's used.

        我们可以让空服务被调用时打印一些debug信息来缓和这点。

        </aside>

Among these options, the one I see used most frequently is simply asserting that
the service will be found. By the time a game gets out the door, it's been very
heavily tested, and it will likely be run on a reliable piece of hardware. The
chances of a service failing to be found by then are pretty slim.

在这些选项中，我看到最常使用的是简单的服务会被找到的断言。
在游戏发布的时候，它经历了严格的测试，它会在可信赖的硬件上运行。
无法找到服务的机会非常渺茫。

On a larger team, I encourage you to throw a null service in. It doesn't take
much effort to implement, and can spare you from some downtime during
development when a service isn't available. It also gives you an easy way to
turn off a service if it's buggy or is just distracting you from what you're
working on.

在更大的团队中，我鼓励你使用空服务。
这不会花太多时间实现，可以减少开发中服务不可用的缺陷。
这也给你了一个简单的方式去关闭服务，无论它是有漏洞还是干扰到了你真正工作的东西。

### What is the scope of the service?

### 服务的服务范围有多大？

Up to this point, we've assumed that the locator will provide access to the
service to *anyone* who wants it. While this is the typical way the pattern is
used, another option is to limit access to a single class and its descendants,
like so:

到了现在，我们假设定位器会提供服务给*任何*需要它的地方。
当让这是这个模式通常的使用方式，另一选项是局限到类和它的子类，就像这样：

^code 3

With this, access to the service is restricted to classes that inherit `Base`.
There are advantages either way:

通过这些，对服务的接触被收缩到了继承`Base`的类。这两种各有千秋：

 *  **If access is global:**

 *  **如果全局可获取：**

     *  *It encourages the entire codebase to all use the same service.* Most
        services are intended to be singular. By allowing the entire codebase to
        have access to the same service, we can avoid random places in code
        instantiating their own providers because they can't get to the "real"
        one.

     *  *鼓励整个代码库使用同样的服务。*大多数服务都被设计成单一的。通过允许整个代码库接触到相同的服务，我们可以避免代码到处实例化提供者，因为他们不能获取“真正的”提供者。

     *  *We lose control over where and when the service is used.* This is the
        obvious cost of making something global -- anything can get to it. The <a
        class="gof-pattern" href="singleton.html">Singleton</a> chapter has a
        full cast of characters for the horror show that global scope can spawn.

     *  *我们失去了何时何地使用服务的控制权。*这是让某物全局化的明显代价——任何东西都能接触它。<a class="gof-pattern" href="singleton.html">单例</a>模式一章讲了全局变量可以多么糟糕的拖慢系统。

 *  **If access is restricted to a class:**

 *  **如果接触被限制在一个当中：**

     *  *We control coupling.* This is the main advantage. By limiting a service
        to a branch of the inheritance tree, we can make sure systems that
        should be decoupled stay decoupled.

     *  *我们控制耦合。*这是主要的优点。通过显式限制服务到继承树的分支，我们保证了应该解耦的系统保持解耦。

     *  *It can lead to duplicate effort.* The potential downside is that if a
        couple of unrelated classes *do* need access to the service, they'll
        each need to have their own reference to it. Whatever process is used to
        locate or register the service will have to be duplicated between those
        classes.

     *  *可能导致重复的付出。*潜在的缺点是如果一对无关的类*确实*需要接触服务，它们每一个都要拥有它的应用。无论是谁定位或者注册服务，它也需要在这些类之间复制。

        (The other option is to change the class hierarchy around to give those
        classes a common base class, but that's probably more trouble than it's
        worth.)

        （另一个选项是改变类的继承层次，给这些类一个公共的基类，但也许这引起的麻烦多于收益。）

My general guideline is that if the service is restricted to a single domain in
the game, then limit its scope to a class. For example, a service for getting
access to the network can probably be limited to online classes. Services that
get used more widely like logging should be global.

我的通用准则是，如果服务局限在游戏的一个领域中，那么限制它的工作范围在一个类上面。
举个例子，获取网络接口的服务可能限制于在线类中。
像日志这样更加广泛应用的服务应该是全局的。

## See Also

## 参见

 *  The Service Locator pattern is a sibling to <a class="gof-pattern"
    href="singleton.html">Singleton</a> in many ways, so it's worth looking at
    both to see which is most appropriate for your needs.

 *  服务定位模式在很多方面是<a class="gof-pattern" href="singleton.html">单例</a>模式的兄弟，在应用前看看两者哪个更适合你的需求是很值得的。

 *  The [Unity](http://unity3d.com) framework uses this pattern in concert with
    the <a href="component.html" class="pattern">Component</a> pattern in its
    [`GetComponent()`][get-component] method.

 *  [Unity](http://unity3d.com)框架在它的[`GetComponent()`][get-component]方法中使用这个模式协调它的<a href="component.html" class="pattern">组件</a>模式

 *  Microsoft's [XNA][] framework for game development has this pattern built
    into its core `Game` class. Each instance has a `GameServices` object that
    can be used to register and locate services of any type.

 *  微软的[XNA][]游戏开发框架在它的核心`Game`类中内建了这种模式。每个实体都有一个`GameServices`对象可以用来注册和定位任何种类的服务。

[get-component]: http://docs.unity3d.com/412/Documentation/ScriptReference/Component.GetComponent.html?from=index
[xna]: http://msdn.microsoft.com/en-us/library/microsoft.xna.framework.game.services.aspx


[get-component]: http://docs.unity3d.com/412/Documentation/ScriptReference/Component.GetComponent.html?from=index
[xna]: http://msdn.microsoft.com/en-us/library/microsoft.xna.framework.game.services.aspx
