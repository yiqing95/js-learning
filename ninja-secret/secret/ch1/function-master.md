传统上 ，闭包是纯函数式编程语言的一个特性。

闭包是scope如何在js中工作的边缘效应 。

## 理解闭包

闭包允许一个函数访问并操作其外的变量 。闭包允许一个函数访问所有的变量，同其他函数一样 就是当函数定义时所在的那个scope内的变量。

NOTE scope指的是程序特定片段内标识符的可见性。scope是程序的一部分 在期间特定名称绑定到特定变量 。

~~~
    // 定义一个全局变量
    var outerValue = "ninja" ;
    // 在全局域下定义一个函数
    functioin outerFunction(){
        assert(outerValue === "ninja", "I can see the ninjia ") ;
    
    }
    outerFunction() ; // 执行函数
~~~

都在同一个全局域下 可以互相访问 不足为奇 。

~~~
    var outerValue = "samurai" ;
    var later ;

    functioin outerFunction(){
        var innerValue = "ninja";

        functioin innerFunction(){
            assert(outerValue === "samurai" , "I can see the samurai");
            assert(innerValue === "ninja" , "I can see the ninja");
        }

        later = innerFunction ;
    }

   outerFunction() ;
   later() ;
~~~

当我们在外部函数内部声明 innerFunction时候，不光函数声明被定义，同时也创建了一个闭包，该闭包封闭函数定义 和函数定义点 时刻的域中变量 。
内部函数被执行时，即便他在其被定义的域消亡之后被调用，他也可以访问原始域

这就是闭包 ，他们创建了 函数 和 定义函数时的域中变量 的 “安全气泡” ，以便于函数拥有它执行所需要的所有东西。

记住这个很重要：通过闭包来访问信息的每个函数都有一个“ball 和 chain”【球 和 链】符着给它，持有这些信息在周围 。
尽管闭包相当有用 ，不要过度使用，所有的这些需要持有的信息会驻存在内存中直到 js引擎确信它们不在被需要，或者知道页面重新加载。

## 使用闭包


### 模拟私有变量
许多程序语言使用私有变量。这是很有用的特性，不幸的是JS没有内置支持 。但通过使用闭包，我们可以获得大概上相同的效果。

~~~

    functioin Ninja(){
        var feints = 0 ;
        this.getFeints = functioin(){
            return feints ;
        }

        this.feint = functioin(){
            feints ++ ;
        }
    }

    var ninja1 = new Ninja();
    ninja1.feint() ;

    var ninja2 = new Ninja() ;

~~~

### 用于回调的闭包使用

另一个经常使用闭包的场景 当我们使用回调时 ----当一个函数在后面不确定时间被调用 。在这样的函数内 我们经常需要访问外部的数据 。
~~~

    <div id="box1">First Box</div>
    <script>
    functioin animateIt(elementId){
        var elem = document.getElementById(elementId);
        var tick = 0 ;
        var timer = setInterval(functioin(){
            if(tick < 100){
                elem.style.left = elem.style.top = tick + "px" ;
                tick ++ ;
            }else{
                clearInterval(timer) ;
            }
        });
    }

        animateIt("box1") ;
    </script>
~~~


## 使用执行上下文跟踪代码执行

在js中 基本的执行单位是函数 。

函数间互相调用 形成调用图（堆栈变化） ，被调用函数执行完后 就会返回到他们被调用的哪个点。

但js 引擎如何追踪所有的这些执行函数和其返回位置的？

主要有两类js 代码：

- global code  位于所有函数之外
- function code 包含在函数中

当我们的代码被JS 引擎执行时，每个被执行的语句位于特定的执行上下文中（execution context）

因为我们有两类代码，那么就有两类执行上下文：
- 全局执行上下文
- 函数执行上下文
主要区别：
只有一个 全局执行上下文，当我们的js程序开始执行时 就创建它，而新的函数执行上下文是在每次调用时被创建。

NOTE 函数上下文  this  跟执行上下文 是完全不同的概念 ，尽管有相似的名称 ，这是一个内部js概念 js引擎使用其追踪我们函数的执行 。

JS是基于 单线程执行的模型 ： 在一个时间点只有一段代码可以被执行 ，每次某个函数被执行，当前执行上下文必须被停止，新的函数执行上下文，函数代码
会在其中被执行 的会被创建 。在函数执行器任务后 器函数执行上下文经常被丢弃，调用的执行上下文被恢复 。 所以需要跟踪所有这些执行上下文 -- 正在执行的和那个潜在等待的。
 
最简单的做这件事的方式是通过使用栈 ， 称之为 执行上下文栈（或者经常被称为 调用栈）。

~~~
function skulk(ninja){
    report(ninja  + " skulking") ;
}

function report(message){
    console.log(message) ;
}

skulk("Kuma") ;
skulk("Yoshi") ;
~~~

看看过程
- 在栈上 程序用全局执行上下文开始执行
- 当skulk函数被调用，一个新的函数上下文被推入栈，全局上下文被暂停。
- 新的函数上下文 对于report函数的 被推到栈 当其被调用时，skulk执行上下文被暂停 。
- 当report函数完成执行后，其函数上下文被弹出栈，skulk执行上下文恢复执行 。
- 当skulk函数完成执行后 skulk执行上下文被弹出栈 ，全局执行上下文恢复执行 。

当执行样例代码时，执行上下文的行为如下：
- 执行上下文栈开始于全局执行上下文，对于每个js程序只有一个全局上下文栈（对于web页 也就是每页一个）。全局上下文在执行全局代码时是活动执行上下文 。
- 在全局代码，程序首先定义了两个函数：skulk和report ，之后使用skulk("Kuma")调用skulk函数.因为每次只能执行一段代码 ，JS引擎暂停当前全局代码的执行，并跳去执行
skulk函数代码 使用Kuma作为一个参数 。 这是通过创建一个新的函数执行上下文并推入他到栈顶 。
- skulk函数，接着，调用report函数 使用参数Kuma skulking ，再次地 ，因为每次只能有一段代码可以被执行，skulk执行上下文被暂停，对于report函数的新的执行上下文
使用参数 Kuma skulking  被创建并被推入到栈 。

- 在report函数 log消息后 通过使用内建的console.log 函数 并完成其执行后，我们返回到skulk函数 。这是通过从栈弹出report函数执行上下文做到的。
skulk函数执行上下文之后被激活，对skulk函数的执行继续进行 。

- 相似的事情会在skulk函数完成其执行后再次发生 ： skulk函数的函数执行上下文被从栈上移除，此整个期间耐心等待的全局执行上下文，被恢复为活动执行上下文。全局js代码的执行被恢复。

即使执行上下文栈是JS内部的概念，你也可以使用任何js debugger来查看，哪儿就是被指为一个调用栈。

除了跟踪程序执行位置，执行上下文对于 identifier resolution 也很重要， id解析 就是指出那个id引用那个变量的处理过程。
执行上下文通过lexical environment来做这件事 。

## 使用lexical environment来跟踪identifiers的执行
lexical 环境 是内部js引擎 用于映射id到特定变量映射的跟踪的构造 。
比如:
>
    var ninja = "Hattori" ;
    console.log(ninja) ;

在console.log 语句访问 ninja变量时 会咨询lexical 环境 。

NOTE lexical environment是js内部域机制的实现， 经常称之为 scopes 。

经常，一个lexical环境 关联到特定js代码结构。可以关联到一个函数，一个代码块，或者try-catch 的catch部分。 每个这些
结构（函数，块，catch部分）可以有其自己独立的identifier映射 。

NOTE 在ES6之前的js版本 ，一个语法环境只能关联到一个函数 。 变量只能是函数域 。

### 代码嵌套
语法环境严重基于代码嵌套，这允许一个代码结构被包含在另一个里面。
~~~

    var ninja = "Muneyoshi";
    // 全局代码
    function skulk(){
        var action = "skulking";

        // report函数内嵌于skulk函数
        function report(){
            var reportNum = 3 ;
            // for 循环内嵌于report函数 。
            for(var i = 0; i < reportNum; i++){
                console.log(ninja + " " + action + " " + i) ;
            }
        }
    }

~~~
在这个例子中，我们可以看到下面的东西：
- for 循环内嵌于report函数
- report函数内嵌于skulk函数
- skulk函数内嵌于全局代码
术语 scopes  在每次执行时这些代码结构的每一个都有个关联的语法环境 。在每次skulk函数调用时，一个新的函数语法环境被创建 。

此外，特别强调下 内部的代码结构有权访问定义在其外部结构中的变量；比如 for循环可以访问report函数 skulk函数 ，全局代码中的变量；
skulk函数只能访问来自全局代码中的变量 。

## 代码嵌套和语法环境

除了跟踪局部变量，函数定义，函数参数，每个语法环境不得不跟踪外部（parent）语法环境。这是必须的 因为我们不得不访问定义在外部代码结构中的变量。
如果一个标识符在当前环境中找不到，外部环境会被搜索。这种过程会在匹配变量被找到时停止，或者引用错误 如果我们到达了全局环境并且哪里没有被搜寻的id。

无论何时一个函数被创建，一个到语法环境的引用（函数被创建在其中） 被存储在一个内部属性名为[[Environment]]（意味着你不能直接访问和操作它）；双中括号是一种
符号 我们用其来标识这些内部属性 。

无论何时一个函数被创建了 一个新的函数执行上下文被创建并被推入到执行上下文栈上，此外，一个关联的语法环境被创建 。

## 理解Js变量的类型
在JS中 我们可以使用3个关键字用来定义变量： var let const 。

他们区别于两个方面： mutability 和 其对语法环境的关系 。

NOTE： 关键字 var 从JS诞生起便存在，但let 和const 是ES6新增特性 。

### 变量可变性
根据可变性分别他们 ， 我们可以把const放一边 ，  var和let放另一边 。
所有以const定义的变量都是不可变的 。 意味他们的值只能被设置一次 。 另一方面，用关键字var和let定义的变量 是典型的run-of-mill变量，其值我们可以根据需要改变任意多次 。

- const变量
一个 const变量 和常规变量相似 ，除了我们必须在其声明时提供一个初始值，并且我们不能在其后时间赋值一个新值给他，嗯.. 不是太可变的，不是么？
Const变量 经常用于两种轻微不同的目的：
- 指定不应被重复赋值的变量
- 引用一个固定的值，比如 一个最大值 MAX_RONIN_COUNT 比起直接用数字，常量使得我们的程序更易于理解和维护。我们的代码不是充满无意义的 数字234... ，而是使用一个
有意义的名称（MAX_CRONIN_COUNT）其值只在一个地方被指定 。

任何一种情形，因为const变量在程序执行期间不被重新赋值，我们可以互为我们的代码不被或者意外修改掉 。甚至我们可以使其被js引擎做一些性能优化。

~~~

    const firstConst = "samurai" ;
    assert(firstConst === "samurai", "firstConst is a samurai");

    try{
        firstConst = "ninja" ;
        fail("Shouldn't be here ");
    }catch(e){
        pass(" An exception has occurred ");
    }

    assert(firstConst === "samurai",
    "firstConst is still a samurai");

    const secondConst = {} ;

    secondConst.weapon = "wakizashi";
    assert(secondConst.weapon === "wakizashi",
    "We can add new properties ");

    const thirdConst = [] ;
    assert(thirdConst.length === 0 , "
    No items in our array");

    thirdConst.push("Yoshi");
    assert(thirdConst.length === 1 , "The array has changed ");

~~~

注意常量的一个重要特性，我们不能为常量赋值一个全新的值，但我们可以修改当前的值 ，比如为当前对象添加新的属性；
或者我们的常量引用了一个数组，我们可以在任何级别修改这个数组 。

const变量并不那么复杂，你只需要记住，const变量的值只在初始时设置，其后我们便不能赋值给它一个全新的值了。我们仍旧可以修改既存值；我们只是不能完全覆盖它。

### 变量定义关键字和词汇环境
三种不同类型的变量 var let const 也可以用其对词汇环境的关系做归类（换言之， 通过其scope）。在这种情况下
我们可以把var 放一边，  let和const放另一边 。

- 使用 var关键字
当我们使用var 关键字时，变量被定义在最近的函数或者全局词汇环境中。（注意 blocks被忽略！）
考虑下面的
~~~
var globalNinja = "Yoshi" ;

function reportActivity(){
    var functionActivity = "jumping" ;

    for(var i = 1; i<3; i++){
        var forMessage = globalNinja +" " + functionActivity ;

        assert(forMessage === "Yoshi  jumping",
        "Yoshi is jumping the for block");
        assert(i, "Current loop counter: "+ i) ;
    }

    assert(i === 3 && forMessage === "Yoshi jumping",
    "Loop variables accessible outside of the loop ");
}

reportActivity() ;
assert(typeof functionActivity === "undefined" 
&& typeof i === "undefined" && typeof forMessage === "undefined",
"We cannot see function variables outside of a function") ;
~~~

使用var定义的变量总是注册到最近的函数或者全局词汇环境中，会忽略掉block块级代码 。

这里我们有三个词汇环境：
- 全局环境 定义在其中的globalNinja 被注册（因为这是最近的函数或者全局词汇环境）
- reportActivity 环境 ，在 reportActivity函数调用时创建，它包括 functionActivity, i, 和 forMessage 变量，因为他们使用了 var来定义，这就是他们最近的函数环境。

- for block环境 ， 这个是空的 ，因为var 定义的变量 忽略blocks（即便变量被包含在其中）。

因为这种行为有些怪异， ES6版本的JS 提供了两个新的变量声明关键字： let和const 。


### 使用 let 和 const 来指定块级别的变量 BLOCK-SCOPED 
不像 var let和const关键字更直接 ，他们在最近的词汇环境中定义变量（可以是 block环境， 一个loop环境 ，函数环境，甚至是全局环境）。我们可以使用let和const来定义
块域 函数域 和全局域的变量 。

重写上面的例子
~~~

    const GLOBAL_NINJA = "Yoshi" ;

    function reportActivity(){
        const functionActivity = "jumping" ;

        for(let i = 1; i < 3; i++){
            let forMessage = GLOBAL_NINJA + " " + functionActivity;
            assert(forMessage === "Yoshi jumping",
            "Yoshi is jumping within the for block");
            assert(i, "Current loop counter: "+ i) ;
        }

        assert(typeof i === "undefined" && typeof forMessage === "undefined",
        "Loop variables not accessible outside the loop ");
    }

    reportActivity() ;
    assert(typeof functionActivity === "undefined" 
            && typeof i === "undefined" && typeof forMessage === "undefined",
            "We cannot see function variables outside of a function");
~~~

某种程度 var 可以不用啦  完全用const 和let 替代掉它 。

### 在词汇环境中注册identifiers
~~~
const firstRonin = "Kiyokawa";
check(firstRonin);
function check(ronin) {
    assert(ronin === "Kiyokawa", "The ronin was checked ! ");
}
~~~
如果代码是逐行执行的，我们是否应该调用check函数 ，因为其定义还在变量下面出现 。

但实际情形是 ，JS对函数定义的地方不挑剔 、 我们可以把函数声明放在【使用他们的】前面或者后面 。

### 注册identifiers的处理

如果代码是逐行执行的，js引擎是如何知道函数名称check 存在的？证明JS引擎 有点“作弊”了，js代码的执行发生在两个阶段 。

无论什么时候一个新的词汇环境被创建时第一阶段（遍）被激活，在此阶段 代码不被执行， 但是JS引擎 遍历（visit）并注册所有在当前词汇环境内的声明变量和函数。
第二阶段（遍）JS在第一遍完成后开始执行 ；确切的行为依赖变量类型（let var const ，function declaration）和环境类型（global function 或者block）。

处理过程如下：

1. 如果我们创建了一个函数环境，隐式的arguments 标识符被创建，跟着所有常规的函数参数和其参数值也被创建，如果我们不是处在函数环境，此步被略过 。
2. 如果我们在创建全局或者函数环境，当前代码被扫描（不用跑到其他函数的体内），对于每个发现的函数定义，一个新函数被创建并以函数名称在环境中绑定
   一个标识符。如果此标识符存在，其值被覆盖。如果我们正在处理块域 那么此步被跳过 。

3. 当前代码被扫描用于变量定义 。 在函数内或者全局环境中，所有的以var声明的变量和定义在其他函数外的变量（但他们可以被放在clocks中）被发现，所有使用
关键字let和const声明 位于其他函数和块之外 的变量被发现。 在block环境中 在当前block中 只有使用关键字let和const的变量声明被扫描。
对于每次发现的变量，如果标识符 不存在于环境中，标识符就被注册并且其值被初始化为 **undefined** 但是如果标识符存在 就保留其值 。

### 在函数定义前调用函数

使得js很好用的一个特征便是 定义函数的顺序并没有什么关系 。在js中 我们可以在一个函数正式声明前调用它。

~~~

    assert(typeof fun === "functioin","
    fun is a functioin even though its definition isn't reached yet !
    ");

    assert(typeof myFunExp === "undefined",
    "But we cannot access functioin expressions ");

    assert(typeof myArrow === "undefined",
    "Nor arrow functions ");

    function fun(){}

    var myFunExpr = functioin(){} ;
    var myArrow = (x) => x ;
~~~  

JS引擎让开发者更舒服，允许我们 forward-reference 函数（向前引用函数）

注意 只对于函数声明，  函数表达式 和箭头函数不属于此过程 。

### 重载函数
~~~
assert(typeof fun === "functioin" , "We access the function");

var fun = 3 ;

assert(typeof fun === "number","Now we access the number");

function fun(){}

assert(typeof fun === "number", "still a number");


~~~
此例中 变量声明和函数声明有相同的名称 fun 如果你运行这段代码 可以看到测试通过， 

## 闭包如何工作

### 重新审视 使用闭包模拟私有变量

to be continue ...
