代码模块化技术
-------------

涵盖内容
- 使用模块模式
- 使用当前标准 来写模块化代码： AMD和CommonJS
- 使用ES6模块

我们已看到了一些JS的原生基础，比如 函数，对象， 集合， 和正则表达式。我们的裤袋上挂有更多的工具 为了使用我们的JS代码 解决特定问题 .
但因我们的应用开始增长，另一个整体的 关于如何构建和组织我们的代码 的问题集 开始浮现了。

时间一再证明 大的 铁板一块代码基 比更小的良构那个 是更难理解和维护的。
所以唯一自然的方式 来提升我们程序的结构和组织 就是打碎他们为更小的，相对松耦合的被称为 **modules(模块)** 的片段。

    模块是比对象和函数更大的组织我们代码的单元；他允许我们分割我们的程序为同属一个簇。
    当创建模块，我们应该努力形成一致抽象并封装实现细节。
    模块化 也便于重用（同应用，甚至跨不同应用），极大地加速我们的开发过程。

    JS的 全局变量：无论何时在主行代码中定义一个变量，那个变量被自动地认为是全局的 并可被从我们代码的任何其他部分访问。
    对于小程序这不是问题，但当我们的应用开始增长 和我们包含第三方代码，命名冲突出现的机会会开始显著增加。在其他编程语言中，此问题用命名空间解决（c++ 和 C#）
    或者包（java 的packages），它在另一个名字中包括所有的名字，因而显著地减少了潜在的冲突。

ES6之前，JS都没有提供更高级别，内建的特征 用来允许我们来分组相关变量到一个模块中，名空间，或者包。所以 为了解决这个问题，JS程序员已经开发了高级的
代码模块化技术 他使用了既存JS构造的优点（比如 对象，即时函数，和闭包）。此处我们将展示这些技术中的一些。

幸运的是，使用module只是时间问题。因为ES6最终引入了原生modules。不幸的是 浏览器没跟上，

让我们从我们当今能使用的模块化技术开始。

---

## JS ES6前的代码模块化

ES6之前，JS只有两种scopes: global scope 和 函数域。没有中间的其他东西 ， 一个名空间或者一个模块 允许我们把特定功能分组在一起。为了写出模块化代码，
JS开发者被强迫创新性地使用既存JS语言特性。

当决定要使用哪种特性，我们必须记住，最基本地，每个模块系统应该可以做下面的事：
- 定义一个接口 通过它我们可以访问模块提供的功能。
- 隐藏模块内部 以便我们模块的用户不会被整个不重要的实现细节所负担。此外，通过隐藏模块内部，我们把它同外部世界隔离开来 保护内部，因而阻止非预想的修改
其会导致各种不同的边缘效应和bugs。 

### 使用对象，闭包，即时函数来指定模块

让我们回到我们最小化模块系统的需求， **隐藏实现细节** 和 **定义模块接口** 。现在思考下为了实现这些最小化需求我们可以利用哪个JS语言特性：
-  隐藏模块内部 -- 如我们所知，调用一个JS函数创建一个新的域 在其中我们可以定义变量 他们只在当前函数中可见。所以，为了隐藏模块细节的一个选项就是使用函数
作为模块。用这种方式，所有的函数变量 变成内部的模块变量 他们对外部世界是隐藏的。
-  定义模块接口 -- 通过函数变量实现模块内部意味着这些变量只能在那个模块内部被访问。但如果我们的模板是从其他代码被使用的，我们必须能够定义一个干净的接口
通过它 我们可以暴露模块提供的功能。一种实现这个的手段就是通过利用对象和闭包。 思路是这样的，从我们的函数模块，我们返回一个对象 它来表示我们模块的公共接口。
那个对象应该包含模块提供的方法，方法将通过闭包，持续地保持我们内部的模块变量存活，即便是我们的模块函数已经完成其执行 。

现在 我们已经给出了一个 hight-level（高级别）的对如何在JS中实现模块的描述， 让我们一步步慢慢完成这些，从使用函数来隐藏模块内部开始。

    函数作为模块

调用一个函数创建一个新的scope 我们可以用其定义变量 这些变量对当前函数的外部不可见。
看看下面这个点击计数的例子：

~~~js 

    (function countClicks(){
        // 定义一个局部变量 用来存储点击量
        let numClicks = 0 ;
        // 添加一个监听器
        document.addEventListener("click",()=>{
            // 无论何时 用户点击时，计数器会递增 当前的值被报告出来
            alert(++numClicks);
        });
    })();
~~~
注意两个重要事情：
- 在countClicks函数内部的 numClicks 变量，在点击处理器函数的闭包中持续存活。变量只能在处理器中被引用，其他地方可不行！
我们已经 从countClicks函数的外部 防护了 numClicks变量。同时我们也没有 用一个可能对其他剩余代码并不重要的变量 污染 我们程序的全局名空间。

- 我们的 countClicks 函数只在一个地方被调用，所以，没有定义一个函数之后在一个单独语句中调用它，我们使用了即时函数，或者一个IIFE ，来定义并立即调用
countClicks函数。

我们也可以看看当前程序状态，通过闭包我们的内部函数（module）变量如何持续存活。

既然我们现在明白了如何隐藏内部模块细节，以及闭包如何持续让这些内部细节存活必要的时间。让我们移到我们第二个模块最小化的需求：定义模块接口。

    模块模式：参数函数作为模块用对象作为接口
模块接口经常由一些我们模块对外界提供的变量和函数的集合组成。创建这样接口的最简单方式是用 humble JS 对象。

比如 ，让我们为我们的模块创建一个接口 计数我们页面上的点击。

~~~js

    // 创建一个全局模块变量 并赋予一个即时调用（IIFE--- immediately invoking function execution）结果
    const MouseCounterModule = function(){
        // 创建一个私有的模块变量
        let numClicks = 0;
        const handleClick = () => {
            alert(++numClicks);
        };
        // 返回一个表示模块接口的对象，通过闭包 我们可以访问 "私有的"模块变量和函数
        return {
            countClicks: () => {
                document.addEventListener("click",handleClick);
            }
        };
    }();
    // 从外部我们可以访问通过接口暴露的属性
    assert(typeof MouseCounterModule.countClicks === "function",
        "We can access module functionality");
    // 不能访问模块内部
    assert(typeof MouseCounterModule.numClicks === "undefined",
        && typeof MouseCounterModule.handleClick === "undefined",
            "We cannot access internal module details");
~~~

这里我们使用了一个即时函数来实现一个模块。在即时函数内部，我们定义了我们内部模块实现细节：一个本地变量 numClicks, 和一个本地函数
 handleClick,他们只在模块内部可访问。接下来我们创建并即刻返回了一个对象 他将扮演模块的 “公共接口”。此接口包含一个 countClicks 函数 我们可以用其从
 模块外部来访问模块的功能。

 同时，因为我们暴露了模块接口，我们内部模块细节会通过接口创建的闭包持续存活。比如，在此， 接口的countClicks函数 持续存活内部的模块变量 numClicks 
 和 handleClick.

 最后，我们存储表示模块接口的对象，由即时函数返回 到一个变量 名MouseCounterModule 之上，通过它我们可以很容易地消费模块函数，通过下面的代码
 > MouseCounterModule.countClicks() 

 通过使用即时函数，我们可以隐藏特定的模块实现细节。之后通过添加对象和闭包 到mix（混合物），我们可以指定一个模块接口，它暴露我们模块对外界提供的功能。 

 此模式使用 即时函数，对象和闭包 来创建模块 在JS中被称为 **module pattern(模块模式)** 。由 Douglas Crockford(很有名的一个牛人 可以github上搜索)推广。
 并且是第一流行的模块化js代码的方式。

 一旦我们能够定义模块，能分离他们到多个文件（为了更容易地管理他们）总是件好事，或者可以定义额外的功能在已有模块上，而不用修改他们的源码。
    让我们看看 这个如何做到
#### AUGMENTING MODULES（扩展模块） 
让我们扩展下前面例子的 MouseCounterModule 模块 用一个额外的特征，计数鼠标滚动数，但不用修改原始的 MouseCounterModule 的代码。

~~~js
    // 原始代码
    const MouseCounterModule = function(){
        let numClicks = 0;
        const handleClick = () => {
            alert(++numClicks);
        };

        return {
            countClicks: ()=>{
                document.addEventListener("click",handleClick) ;
            }
        };
    }

    // 即时调用一个函数 其接收我们想扩展的模块作为一个参数
    (function(module){
        // 定义新的私有变量和函数
        let numScrolls = 0 ;
        const handleScroll = () => {
            alert(++numScrolls);
        }

        // 扩展模块接口
        module.countScrolls = () => {
            document.addEventListener("wheel", handleScroll);
        };
    })(MouseCounterModule);

    assert(typeof MouseCounterModule.countClicks === "function",
        "We can access initial module functionality");
    assert(typeof MouseCounterModule.countScrolls === "function",
        "We can access augmented module functionality");
~~~
当扩展一个模块时，我们经常遵从同创建新模块类似的过程。我们理解调用一个函数，但这次，我们传递我们想扩展的模块作为一个参数给他。
>
    (function(module){
        ...
        return module ;
    })(MouseCounterModule) ;

在函数中，之后继续我们的工作并创建所有必要的私有变量和函数。在此，我们定义了一个私有变量和一个私有函数用于计数并通报滚动数：
>
    let numScrolls = 0 ;
    const handleScroll = () => {
        alert(++numScrolls);
    }

最后，我们扩展我们的模块，通过及时函数的 module 参数，就像我们想扩展任何一个其他对象那样：    
>
    module.countScrolls = () => {
        document.addEventListener("wheel",handleScroll);
    };

之后，我们执行了这个简单的操作，我们的 MouseCounterModule 也可以 countScrolls 了

我们的公开模块接口 现在有两个方法，并且我们可以以如下方式使用这个模块：
>   // 来自头的 模块的接口的一部分的一个方法
    MouseCounterModule.countClicks();
    // 一个新的模块方法 通过扩展模块添加的
    MouseCounterModule.countScrolls();

如我们已经提及的，我们扩展模块的方式类似我们创建新模块那样，通过一个即时调用函数来扩展模块。这有一些有趣的闭包的边缘效应，所以 让我们来靠近看看我们
扩展模块之后应用程序的状态。

如果你仔细看看，一个 模块模式 的缺陷也被展示了：不能通过扩展模块分享私有模块变量。
注意到两个独立的函数被定义在不同的环境中，他们不能相互访问彼此的内部变量。

不幸的是，我们的扩展， countScrolls 函数，在一个完全独立的域中被创建，用一个完全新的私有变量集： numScrolls和handleScroll。 countScrolls 函数创建
了一个只围绕 numScrolls 和 handleScroll变量 的闭包，因而不能访问 numClicks 和 handleClick 变量。

    NOTE 模块扩展，当通过分离的即使函数执行时，不能分享私有的模块变量，因为每个函数调用创建了新的域。尽管这是一个缺陷,这不是一个 show-stopper(障碍物)
    ，我们仍旧可以使用 模块模式 来使我们JS函数模块化。

注意到，在 模块模式中，模块是对象 同常规其他任何对象一样，并且我们可以 用任何我们觉得合适的方式 扩展他们。比如，我们可以添加新的功能 通过用新的属性来
扩展模块对象：
>   MouseCounterModule.newMethod = () => { ... }

我们也可以很容易地用相似的原理来创建子模块：
>
    MouseCounterModule.newSubmodule = () => {
        return { ... };
    }() ;

注意到所有的这些方法 承受来自相同的 模块模式 的根本缺陷： 模块的后续扩展不能访问前面定义的内部模块细节。

不幸的是，使用模块模式有更多的问题。当我们开始构建模块化程序，模块自己经常依赖其他模块用以完成自己的功能。不幸的是，模块模式没有覆盖对这些模块的依赖管理。
我们，作为一个开发者，不得不考虑正确的依赖顺序以便于我们的模块代码拥有其所有需要来执行。尽管 这在小型或者中等程序中不是一个问题，这会在使用了很多
依赖模块的大型程序中引入很严重问题。

为了处理这种问题，一些竞争的标准涌现了，名为： Asynchronous Module Definition(AMD) 和 CommonJs 。

### 用 AMD 和 CommonJs 来模块化 JS应用
此二者是竞争性的模块规范标准，允许我们来指定JS 模块。除了一些语法和哲学上的不同，主要的不同是AMD被有意的设计来显式的用在浏览器。此节提供了相对短的
这些模块规范的概览；（更多的信息 推荐你看看 <<JavaScript Application Design>> 这本书）。

- AMD
AMD 出生于 Dojo 工具集，一个很流行的JS工具集 用来构建客户端web应用。AMD 允许我们很容易的 详细说明模块和其依赖。同时，它是基于浏览器的。当前，最流行的
AMD实现是[RequireJs](http://requirejs.org)

让我们看一个定义一个依赖于jQuery的小模块的例子。

~~~js

    /**
    * 使用define函数来指定一个模块，其依赖，和模块工厂函数 来创建那个模块
    */
    define('MouseCounterModule',['jQuery'], $ => {
        let numClicks = 0 ;
        const handleClick = () => {
            alert(++numClicks);
        };
        // 我们模块的公共接口
        return {
            countClicks: ()=> {
                $(document).on("click", handleClick);
            }
        };
    });
~~~

AMD 提供了一个函数 称为**define**它接受下面的参数：
- 一个新创建的模块 ID 。 此ID 后面会被 来自我们系统的其他部分 用来 require 模块 。
- 一个 我们当前模块依赖的 模块ID的列表（被需要的模块们）。
- 一个工厂方法 他会初始化模块并接受 被需要的模块作为其参数。

在此例中，我们使用AMD的define函数来创建一个依赖jQuery的模块 使用ID MouseCounterModule。因为这种依赖，AMD先请求jQuery模块， 如果文件
不得不被从一个服务器被请求 这会花一些时间。 此动作会被异步执行，为了防止阻塞。在所有的依赖被下载和执行后，模块工厂函数被调用 （使用每个被请求（依赖的）
的模块 一个参数  ），在这里 只有一个参数，因为我们的新模块只需要了jQuery 。在 工厂函数中，我们只是如同使用标准 模块模式 那样创建我们的模块：
通过返回一个对象 其暴露了那个模块的公共接口。

如你所见， AMD提供了一些有趣的好处，比如这些：
-  自动依赖解析，以便我们不必考虑我们包含我们模块的顺序。
-  模块可被异步加载，因而避免阻塞。
-  多个模块可被定义在一个文件。

既然你已经知道了AMD工作的基本思想，让我们看看另一个，普遍的模块定义标准。

- COMMONJS
AMD明确地为浏览器而生，CommonJS是一个模块规范 被设计用于更通用的目的JS环境。当前在Node.js社区有最多的从众。

    CommonJS使用基于文件的模块，所以我门可以每文件指定一个模块。对每个模块，CommonJS暴露一个变量 , module 用一个属性，exports 它可以很容易地使用额外
属性扩展。最后，被暴露的module.exports的内容作为模块的公共接口。

如果你想在应用的其他部分使用一个模块，我们可以require它。文件会被同步加载，并且我们将可以访问其公共接口。这就是CommonJS在服务端  更流行的原因 ，那里
模块获取是相对迅速的 因为它只需要一个 文件系统 的读。而在客户端，模板不得不从远程服务端被下载，而同步加载经常意味着阻塞。

这次 让我们看看 使用CommonJS 定义我们多次出现的 MouseCounterModule 的一个例子。

~~~js

    // 同步requrie一个jQuery模块
    const $ = require("jQuery");
    let numClicks = 0 ;
    const handleClick = () => {
        alert(++numClicks);
    };

    // 修改模块的export属性来指定模块的公共接口
    module.exports = {
        countClicks: () => {
            $(document).on("click",handleClick);
        }
    };
~~~
为了在不同的文件中包含我们的模块，我们可以这样写：
>   const MouseCounterModule = require("MouseCounterModule.js");
    MouseCounterModule.countClicks() ;

看看这多么简单？

因为CommonJS哲学指出一个文件一个模块，任何放在一个文件模块中的代码会是那个模块的一部分。因而，没必要在一个即时函数中把变量包裹起来。所有的变量定义在一个模块中
会被安全的保护在当前模块的scope中并且不会泄露到全局域scope中。 比如 ，所有三个模块变量（$ numClicks 和 handleClick ）是模块域的，即便他们定义在 顶级（top-level）
 代码（所有函数和块 之外），技术上在标准JS文件中这会使他们成为全局变量。

 再次重要重申，只有通过 module.exports对象暴露的变量和函数才对模块外部可用。步骤很像 模块模式 除了不是返回一个全新的对象，环境已经准备了一个 我们可以用我们的
 接口方法和属性来扩展他。

 CommonJS 有很多好处：   
 - 它有很简单的语法，我们只需要指定 module.exports 属性，而模块代码的剩余部分仍旧像我们写标准JS那样。Requiring 模块也很简单；我们只是 使用require 函数即可。
 - CommonJs对Node.js 是默认模块格式，所以我们已经访问过成千的包通过 npm，node的包管理器可用。

 CommonJS的最大缺陷是它不是明确地为浏览器而生，在浏览器端的JS，哪儿没有对 module变量和export属性的支持；我们不得不封装我们的CommonJS模块为一个
浏览器可读的格式，我们可以使用[Browserify](http://browserify.org)或者[RequireJS](http://requirejs.org/docs/commonjs.html)完成。

通过两个竞争的模块标准，AMD和CommonJS 已经导致了这样的情况 人们分成两拨，有时甚至相反（敌对的）的阵营。如果你工作在相对封闭的项目，这可能不是一个问题；
你选一个更适合你的就好。然而 当我们需要重用来自相反阵营的代码时 并被强迫跳过所有类型的hoops时（翻译有问题！） 问题就来了。 一个方案就是使用
Universal Module Definition.或者[UMD](https://github.com/umdjs/umd) 一个模式用一个优点复杂（蛋疼）的语法 允许同一个文件同时被AMD和CommonJS所用。
更多的资讯 可以上网找找，这不在我们的讨论之列，有很多可用的资源都有介绍他们。

幸运的是，ECMAScript 委员会已经意识到对一个通用的模块定义语法的支持 在所有的JS环境中，所以ES6定义了一个新的模块标准

## ES6模块
兼有CommonJS和AMD的优点：
- 和CommonJs类似，ES6模块有一个相对简单的语法，并且是基于文件的（一个模块一文件）。
- 和AMD类似，ES6模块提供了对异步模块加载的支持。

NOTE 内建的模块是ES6标准的一部分。如你一会儿看到的，ES6模块语法包含额外的语义和关键字（比如 export和 import 关键字）他们在当前浏览器不被支持，如果
我们想现在就使用模块，我们必须转换我们的模块 使用 Traceur ， Babel 或者 TypeScript。我们也可以使用SystemJS库，它提供了加载所有当前可用的模块标准的支持：
AMD CommonJS，甚至ES6模块。 你可以找到如何使用SystemJS的用法说明在其项目仓库[systemjs](https://github.com/systemjs/systemjs)

ES6模块后面的主要思想是 只有从一个模块显式明确的暴露的标识才对其外可访问。所有其他的标识符，即便定义在顶级域（在标准JS中 是global scope）也只有在
那个模块中可访问，这是受CommonJS启发的。

为了提供这个功能，ES6引入了两个新的关键字：
- export 用来使特定的标识符对模块外部可用
- import 用来导入被暴露的模块标识符

用于导出和导入模块功能的语法很简单，但它有一些微妙的差别 我们会慢慢展示给你。

### 导出和导入功能
让我们用一个简单的例子作为开端，展示如何从一个模块导出功能 和如何在另一个模块中导入它。

~~~js

    // 在模块中定义一个顶级变量
    const ninja = "Yoshi";
    
    // 定义一个变量和函数，从模块中用export关键字导出他们
    export const message = "Hello";

    // 访问一个内部模块变量 从模块的公共API
    export function sayHiToNinja(){
        return message + " "+ ninja ;
    }
~~~

我们首先定义了一个变量，ninja， 一个只能在此模块中访问的模块变量，即使他处在顶级代码域（在ES6之前 会使其成为全局变量）。

之后我们定义了另一个顶级变量 message 通过使用新的 export 关键字使其从模块外部可被访问。最后，我们也创建并导出 sayHiToNinja 函数。

就是这些！这就是最小化的我们需要知道的语法用来定义我们的模块。我们不必使用即时函数或者记忆任何秘传语法用来从一个模块导出功能。我们像常规JS代码那样写我们的代码
只有一个差别 同export关键字一起 冠以某些标识符（比如变量，函数，类）。

在开始学习如何导入这些导出的功能之前，我们将看看一个替换的方式来导出标识符：我们在模块尾部列出所有我们想导出的东西，如下所列。

~~~js

    // 指定所有的模块变量
    const ninja = "Yoshi";
    const message = "Hello";

    function sayHiToNinja(){
        return message + " " + ninja ;
    }

    // 导出一些模块标识符
    export{ message, sayHiToNinja };
~~~

这种导出模块标识符的方式 和模块模式 有些相似之处，像一个即时函数返回一个 用来表示我们模块公共接口的对象。特别对CommonJs而言，我们用公共的模块接口扩展
 module.exports对象 ；

 不论我们如何导出一个特定模块的标识符，如果如果需要在其他模块中导入他们，我们必须使用 import 关键字，就如下面这样：

 ~~~js

    // 使用 import关键字来从一个模块导入一个标识符绑定
    import { message , sayHiToNinja } from "Ninja.js";

    // 我们现在访问导入的变量 并 调用导入的函数。
    assert(message === "Hello",
        "We can access the imported variable");
    assert(sayHiToNinja() === "Hello Yoshi",
        "We can say hi to Yoshi from outside the module");
    // 我们不能直接访问 未导出的模块变量
    assert(typeof ninja === "undefined",
        "But we cannot access Yoshi directly");
 ~~~

 我们使用 新的 import 关键字来 从ninja模块中  导入一个变量，messaget 和一个函数 sayHiToNinja：
 >
    import { message, sayHiToNinja } from "Ninja.js";

通过这样做,我们获取了对这两个定义在ninja模块中的标识符的访问权 。最后，我们可以测试下 我们可以访问 message变量和调用 sayHiToNinja函数:

~~~js
    assert(message === "Hello",
        "We can access the imported variable");
    assert(sayHiToNinja() === "Hello Yoshi",
        "We can say hi to Yoshi from outside the module");

~~~
我们不能访问未导出和未导入的变量。比如，我们不能访问 ninja 变量 因为 其使未使用export标记：
>   assert(typeof ninja === "undefined","But we cannot access Yoshi directly");

使用module 我们最终有点安全 远离了全局变量的滥用。任何我们在模块中没有明确标识为导出的东西被很好的隔离了。

在此例中，我们使用了一个 命名式export ,这样让我们可以从模块中导出多个标识符（如我们用message和 sayHiToNinja所作的 ）。因为我们可以导出大量的标识符，
在一个导入语句中列举所有的这些标识符 可能是冗繁的。因此，一个简短标识允许我们从一个模块导入所有导出的标识符，如下所列：
>
    // 使用 * 符号来导入所有导出的变量
    import * as ninjaModule from "Ninja.js";
    // 通过属性符号来引用命名导出
    assert(ninjaModule.message === "Hello",
        "We can access the imported variable");
    assert(ninjaModule.sayHiToNinja() === "Hello Yoshi",
        "We can say hi to Yoshi from outside the module");
    // 我们仍旧不能访问没有被导出 的标识符
    assert(typeof ninjaModule.ninja === "undefined",
        "But we cannot access Yoshi directly");

如上示，为了导入一个模块的所有的导出标识符 ，我们使用 import * 符号 联合使用一个标识符（我们将用其来引用整个模块 在此，就是 ninjaModule 标识符）。
在我们这样做之后       我们可以访问导出的变量通过属性符号；比如， ninjaModule.message,ninjaModule.sayHiToNinja.注意 我们不能访问 我们没有导出的
 顶级变量 ，此处即 ninja变量。

默认导出

我们经常不想从一个模块导出一些关联的标识符，而是 想通过一个单独的导出表示整个模块。一个相当常用的情况 是当我们的模块包含一个单独类 时会发生，如下.

~~~js
    // 使用export default 关键字来指定默认的模块绑定
    export default class Ninja {
        constructor(name){
            this.name = name ;
        }
    }
    // 我们仍旧可以使用命名式导出 伴随默认导出一起。 
    export function compareNinjas(ninja1, ninja2){
        return ninja1.name === ninja2.name ;
    }
~~~
这里我们已经添加了default 关键字在export 关键字之后，这指定了此模块的默认绑定。此处，模块的默认绑定是一个class 名为 Ninja。即便我们已经指定了一个默认绑定，
我们仍旧可以使用命名导出来导出额外的标识符，如我们这里所作（用 compareNinjas 函数 ）。因为我们可以导出大量的标识符，

现在，我们可以使用简化的语法来慈宁宫Ninja.js 导入功能，如下示：

~~~js
    // 当导入一个默认导出时，没必要使用大括号，并且我们可以使用任意我们想要的名称
    import ImportedNinja from "Ninja.js";
    // 我们仍旧可以导入命名导出
    import { compareNinjas } from "Ninja.js";

    const ninja1 = new ImportedNinja("Yoshi");
    const ninja2 = new ImportedNinja("Hattori");

    assert(ninja1 !== undefined 
        && ninja2 !== undefined, "We can create a couple of Ninja");

    assert(!compareNinjas(ninja1, ninja2),
        "We can compare ninjas");
~~~
我们从导出一个默认导出开始这个例子。为此，我们使用了一个不容易凌乱的导入语法 通过去掉对于命名导出必须的大括号。
也应注意 我们可以选择任意名称来引用默认导出；我们没有被局限必须使用导出时的哪个名称。在此例， ImportedNinja 引用到 定义在Ninja.js文件中的Ninja 类。

我们继续此例，通过导入一个命名导出，如前面哪个例子那样，只是举例 我们可以同时 在一个单独的模块中 拥有 默认导出 和一些命名导出。最后我们实例化了一些
ninja对象 并调用其 compareNinjas 函数，来确信所有的导入如我们所想。

在此，两个导入源自同一个文件 ES6 提供了简短语法：
> import ImportedNinja , {compareNinjas} from "Ninja.js";
这里我们使用逗号操作符来同时导入默认和命名导出 从Ninja.js文件中，在一个单独的语句中。

    重命名导出和导入
如果必要，我们也可以重命名 导出和导入。 

~~~js

    //*************** Greetings.js *****************************/
    // 定义一个函数 称为 sayHi
    function sayHi(){
        return "Hello";
    }
    // 测试 我们只可以访问sayHi函数，而不是其别名
    assert(typeof sayHi === "function"
        && typeof sayHello === "undefined",
            "Within the module we can access only sayHi");
    // 使用as关键字 提供一个标识符别名
    export { sayHi as sayHello }

    //***************   main.js *************************** /
    import {sayHello} from "Greetings.js";
    // 当导入时，只有别名可用。
    assert(typeof sayHi === "undefined"
        && typeof sayHello === "function",
        "When importing, we can only access the alias");

~~~
在前面的例子中，我们定义了一个函数称为 sayHi,并且我们测试 我们可以访问函数只有通过 sayHi 标识符，而不是通过 sayHello 别名（在模块底部 通过as关键字）
>   export { sayHi as sayHello }

我们执行一个导出重命名 只有在此导出形式，并且不用在变量或者函数声明前面使用export关键字。

之后，当我们执行一个重命名导出的导入时，我们通过给定的别名引用导入：
> import { sayHello } from "Greetings.js";
最后，我们测试了 我们可以访问别名标识符，但不是原始那个：
>
    assert(typeof sayHi === "undefined"
        && typeof sayHello === "function",
            "When importing, we can only access the alias");
当重命名导入时情况类似，如下面代码片段所示：

~~~js
    /******************* Hello.js ************************/
    export function greet(){
        return "Hello";
    }

    /******************* Salute.js **********************/
    export function greet(){
        return "Salute";
    }

     /******************* main.js **********************/
     // 使用as 关键字来别名导入 因而避免名称冲突
     import { greet as sayHello } from "Hello.js";
     import { greet as salute } from "Salute.js";

     // 我们不能访问原始函数名
     assert(typeof greet === "undefined",
        "We cannot access greet");
    // 但我们能够访问别名
    assert(sayHello() === "Hello" && salute() === "Salute",
        "We can access aliased identifiers")
~~~    
同导出标识符类似，当从其他模块导入标识符时 我们也可以用as 关键字来创建别名。    这很有用 当我们需要提供一个更合适更好的名称（对当前上下文更舒服，）
或者当我们想避免名称冲突，如在这个小例子中那样。

至此 ，我们已经完成了我们对ES6模块语法的解释，可以涵盖在下表

    ES6模块语法概览

|  代码         |   意义   |   
| ---           | ------- | 
| export const ninja = "Yoshi";            |  导出一个命名变量              |
| export function compare(){}              |  导出一个命名函数              |
| export class Ninja{}                     |  导出一个命名类                |
|                                          |                               |
| export default class Ninja{}             |  导出默认的类导出              |
| export default function Ninja(){}        |  导出默认的函数导出             |
|                                          |                               |
|  
>
     const ninja = "Yoshi";                    
     function compare(){};                     
     export {ninja, compare};                // 导出既有变量   
     export {ninja as samurai, compare};     // 通过新名称导出一个变量
| | |        
|---------| --- |
| import Ninja from "Ninja.js";            |     导入一个默认导出                     |
| import {ninja, Ninja} from "Ninja.js";   |  导入命名导出                            |
| import * as Ninja from "Ninja.js";       |      从一个模块导入所有命名导出           |
| import {ninja as iNinja} from "Ninja.js"; |    导入一个命名导出通过一个新名称        |

