控制对对象的访问
-------------

涵盖
- 使用getters和setters来访问对象的属性。
- 通过代理来控制对对象的访问
- 使用代理用于 cross-cutting concerns（面向横切关注点  AOP术语）

在很多时候（验证属性值，logging，或者在UI显示数据），我们需要控制访问，监控我们的对象的所有变化。

## 使用getters和setters 控制访问属性
在JS中 对象是相对简单的属性集合。追踪我们程序状态的最主要手段是通过修改这些属性。比如：
~~~

    function Ninja(level){
        this.skillLevel = level ;
    }
    const ninja = new Ninja(100) ;
~~~
这里我们定义了一个Ninja构造器 它创建了一个ninja对象 有一个属性skillLevel ，之后 如果你想改变这个属性的值，我们可以写出下面的代码
： ninja.skillLevel = 20 .
这些都很好并且很方便 ，但下面这些情形会发发生呢？
- 我们想安全保护意外的错误，比如赋值了一个非预期数据。 如 ninja.skillLevel = "high".
- 我们想log skillLevel属性的所有变更。
- 我们需要在我们的web页的UI的某个地方展示skillLevel属性的值。自然地，我们想表示最后，最近更新的属性值，但我们怎样简单地搞定呢。

我们可以很优雅地 处理所有的这些场景，使用 getter和setter方法。
~~~

    function Ninja(){
        // 定义一个私有的skillLevel属性
        let skillLevel ;

        this.getSkillLevel = () => skillLevel ;
        this.setSkillLevel = value => {
            skillLevel = value ;
        }

    }

    const ninja = new Ninja();
    ninja.setSkillLevel(100);

    assert(ninja.getSkillLevel() === 100,
        "Our ninja is at level 100!");
~~~ 

现在，如果我们想log所有对skillLevel属性的读尝试，我们可以扩展我们的getSkillLevel方法； 并且如果我们想对所有的写尝试做出响应，我们可以
扩展setSkillLevel 方法，如下面的片段：
~~~

    function Ninja(){
        let skillLevel ;

        this.getSkillLevel = () => {
            report("Getting skill level value ");
            return skillLevel ;
        };

        this.setSkillLevel = value => {
            report("Modifying skillLevel property from:", skillLevel, "to: ", value);
            skillLevel = value ;
        }
    }
~~~
这很棒，通过插件 我们可以很容易的对所有用我们属性的交互做出响应，比如
logging ， data validation ， 或者边缘效应 如 UI修改。

但有一个挑剔的关注点会跳入你的脑海。 skillLevel 属性是一个值属性；他引用数据，而不是一个函数。
不幸的是 ，为了利用控制访问的好处，所有的我们对属性的交互不得不 明确调用相应方法，说实话 ，有点丑陋。

幸运的是，Js 有内置支持 对真正的getter和setter：像常规数据属性那样访问，但他们是方法 可以计算被请求属性的值，验证传入的值，或者
做些其他任何我们希望的东西。让我们看看内置支持。

### 定义getters 和setters 
在Js中 getter和setter方法可以被使用两种方法定义：
- 在对象字面量内通过指定他们 或者在ES6类定义内 。
- 通过使用内置Object.defineProperty 方法。

~~~

    const ninjaCollection = {
        ninjas: ["Yoshi", "Kuma", "Hattori"],
        get firstNinja(){  // 定义getter方法 
            report("Getting firstNinja"); // logs 一条消息
            return this.ninjas[0];
        },
        set firstNinja(value){
            report("Setting firstNinja");
            this.ninjas[0] = value ;
        }
    };

    assert(ninjaCollection.firstNinja === "Yoshi",
        "Yoshi is the first ninja ");

    ninjaCollection.firstNinja = "Hachi" ;

    assert(ninjaCollection.firstNinja === "Hachi"
        && ninjaCollection.ninjas[0] === "Hachi",
        "Now Hachi is the first ninja");        
~~~

此例中定义了一个 ninjaCollection 对象，它有一个标准的属性：ninjas, 它引用了一个ninjas数组，对firstNinja
属性的setter和getter定义。

* 定义getter方法 ： 通过在属性名前面 冠以get关键字前缀 一个getter不接受任何参数

    get name(){ ... }
* 定义setter方法：  通过在属性名前面 冠以set关键字前缀 一个setter接受一个参数（即 赋值表达式的左侧边 var myVar = {{value}}）
    set name(value){ ... }

~~~
    obj.name ; // 通过读属性值 隐式调用getter方法
    obj.name = "Yoshi";  // 通过为属性赋值 隐式调用setter方法
~~~    

getters和setters允许我们指定的属性被像标准属性那样被访问，但是他们是方法 当属性被访问时 他们的执行就被触发了。

es6 语法
~~~

    class NinjaCollection{
        constructor(){
            this.ninjas = ["Yoshi", "Kuma", "Hattori"];
        }

        get firstNinja(){
            report("Getting firstNinja");
            return this.ninjas[0];
        }
        set firstNinja(value){
            report("Setting firstNinja");
            this.ninjas[0] = value ;
        }
    }

    const ninjaCollection = new NinjaCollection() ;
    assert(ninjaCollection.firstNinja === "Yoshi",
        "Yoshi is the first ninja");
   
    ninjaCollection.firstNinja = "Hachi";
    assert(ninjaCollection.firstNinja === "Hachi"
        && ninjaCollection.ninjas[0] === "Hachi",
        "Now Hachi is the first ninja");
~~~

getter和setter 并非必须同时出现 ，只定义其中的一个 可以做到 保护变量 只读 或 只写。
如果代码在严格模式下 对只读属性的写访问 会导致JS引擎抛出一个类型错误 type error。

传统上getter和setter 使用来控制对对象私有属性的访问的，JS没有私有属性，我们可以用闭包模拟。

因为用对象字面量和类 我们被创建的getter和setter和我们想用来做私有对象属性的变量不在同一个函数域中。我们就不能使用闭包了

幸运的是 我们可以通过另一种途径： Object.defineProperty 方法。

Object.defineProperty 可被用于定义新的属性 通过传递一个属性描述符对象。除了其他东西，属性描述符可以包含一个get和set属性 它定义属性的
getter和setter方法。
~~~

    function Ninja(){
        // 定义一个“私有”的变量 它将被通过闭包来访问
        let _skillLevel = 0 ;

        Object.defineProperty(this, 'skillLevel',{
            get:()=>{
                report("The get method is called");
                return _skillLevel;
            },
            set: value =>{
                report("The set method is called");
                _skillLevel = value ;
            }
        });
    }

    const ninja = new Ninja();

    // 私有变量不能直接被访问，但可以通过skillLevel getter方法
    assert(typeof ninja._skillLevel === "undefined",
        "We connot access a 'private' property");
    assert(ninja.skillLevel === 0 , "The getter works fine!");

    ninja.skillLevel = 10;
    assert(ninja.skillLevel === 10, "The value was updated");
~~~
注意到，不同于在对象字面量和类中指定getters 和 setters，通过 Object.defineProperty定义的 get和set方法 和“私有的”skillLevel变量
被创建在同一个域中（scope）。两个方法都创建了一个闭包 环裹这个私有变量，我们只能通过这两个方法来访问那个私有变量。

如你所见，通过使用Object.defineProperty的方法比在字面量和类中使用getters和setters 更冗长和复杂。但在特定情况下，当我们需要私有对象属性时
，他就值得你拥有了！

不管我们定义他们的方式，getters和setters允许我们 同标准的对象属性那样 定义对象属性，但是是 可以 在我们读或者写一个特定属性时 
执行额外代码的 方法。这是非常有用的特征，使我们 在特定变更发生时 可以执行 logging，验证赋值，甚至通知其他代码部分。

###  使用getters 和setters 验证属性值

setter是一个方法 当我们想写特定匹配属性值时就会执行它。我们可以藉此在想更新属性值时来执行一个动作 ，比如 我们可以做的一件事件就是验证传入的值。

比如下面的例子，只允赋予整数值。
~~~
    function Ninja(){
        let _skillLevel = 0 ;

        Object.defineProperty(this, "skillLevel",{
            get: () => _skillLevel ;
            set: value => {
                if(! Number.isInteger(value)){
                    throw new TypeError("Skill level should be a number");
                }
                _skillLevel = value ;
            }
        });
    }

    const ninja = new Ninja();
    // 我们可以设定整数值给属性
    ninja.skillLevel = 10 ;
    assert(ninja.skillLevel === 10, "The value was updated");

    try{
        ninja.skillLevel = "Great";
        fail("Should not be here");
    }catch(e){
        pass("Setting a non-integer value throws an exception");
    }
~~~
此例中 我们会检测传入的值是否是整数，如果不是 就会抛出一个异常，并且私有的属性也不会被修改。

当然 这添加了负担，但这种代价对于Js这种高度动态的语言在某些时候还是不得不付出的。
这只是setter方法的一个没多大用的例子，你可以用相同的概念来追踪值历史，执行日志，提供变更通知，等...

### 使用getters和setters来定义计算属性
除了能够控制对特定对象属性的访问外，getters和setters 可用来定义 computed properties（计算属性），该属性的值在每次请求时被计算。
计算属性不存储值；他们提供一个 get 和/或 set方法 来直接提取或者设置其他属性。

~~~

    const shogun = {
        name: "Yoshiaki",
        clan: "Ashikaga",
        get fullTitle(){
            return this.name + " " + this.clan ;
        },
        set fullTitle(value){
            const segments = value.split(" ");
            this.name = segments[0];
            this.clan = segments[1];
        }

    }

    assert(shogun.name === "Yoshiaki", "Our shogun Yoshiaki");
    assert(shogun.clan === "Ashikaga", "Of clan Ashikaga");
    assert(shogun.fullTitle === "Yoshiaki Ashikaga",
        "The full name is now Yoshiaki Ashikaga");
    shogun.fullTitle = "Ieyasu Tokugawa";
    
    assert(shogun.name === "Ieyasu", "Our shogun Ieyasu");
    assert(shogun.clan === "Tokugawa", "Of clan Tokugawa");
    assert(shogun.fullTitle === "Ieyasu Tokugawa",
        "The full name is now Ieyasu Tokugawa");
~~~

setter和getter的这种特点虽然有用，但有时 仅仅这些也不够。
在特定情况，我们需要对我们的对象控制所有类型的交互，为此，我们可以使用一个全新的对象类型： proxy

## 使用代理来控制访问
proxy 是一个代理 通过它我们可以控制对另一个对象的访问。它允许我们定义自定义的动作，这些动作会在一个对象被交互时执行。
比如一个属性值被读或者写时，或者一个方法被调用时。

我们可以认为代理几乎是getter和setter的泛化实现。但对每个getter和setter，你只是对单个对象属性的控制访问，而代理使得你可以普遍的处理对一个
对象的所有交互，甚至包括函数调用 。

我们可以在传统的使用getters和setters的地方使用代理，比如用于 logging，数据验证，和计算属性。但代理甚至更强大。
它允许我们很容易地对我们的代码 添加 profiling和性能测量，自动填充对象属性为了避免烦人的null异常，并包裹宿主对象 比如DOM 来消减跨浏览器的不兼容性。

NOTE Proxies 在ES6中引入

在js中我们可以通过使用内建的 Proxy构造器创建代理。
~~~

    const emperor = {name: "Komei"};
    const representative = new Proxy(emperor,{
        get: (target, key) => {
            report("Reading" + key + " through a proxy");
            return key in target ? target[key]
                                : "Don't bother the emperor!";
        },
        set: (target, key, value) => {
            report("Writing "+ key + " through a proxy");
            target[key] = value ;
        }
    });

    assert(emperor.name === "Komei", "The emperor's name is Komei");
    assert(representative.name === "Komei",
            "We can get the name property through a proxy");

    assert(emperor.nickname === undefined,
            "The emperoro doesn't have a nickname ");
    assert(representative.nickname === "Don't bother the emperor!",
            "The proxy jumps in when we make inproper requests");
    // 通过代理添加一个属性，之后 通过代理和目标对象都可以访问这个属性。
    representative.nickname = "Tenno";
    assert(emperor.nickname === "Tenno",
            "The emperor now has a nickname");

    assert(representative.nickname === "Tenno",
            "The nickname is also accessible through the proxy");            
~~~

我们指定了两个traps： 
- get trap 无论何时我们试图通过代理来读取属性值的时候 就会调用，
- set trap 通过代理设置一个属性的值时就会调用它。
get trap执行了这样的功能：如果目标对象有那个属性，属性就被返回； 如果不存在那个属性，我们就返回一个消息 警告我们的用户。

NOTE 重要强调，同getter和setter方式一样 proxy traps 被激活 。 只要我们执行了一个动作（比如 在代理上访问一个属性的值），
匹配的trap 就被隐式地调用，JS引擎像我们显式地调用一个函数那样做了相似处理。

另一方面，如果我们直接在目标对象上访问一个不存在的属性，我们会得到预期值 undefined 。但如果我们通过代理对象，get 处理器会被激活。
因为目标对象上不存在对应的属性，一个警告消息被返回了。

在例子中 我们继续通过我们的代理对象赋值了一个新的属性，因为操作是通过代理来做的，而不是直接做的，set trap 会日志记录一个消息并给我们的
目标对象上设置一个属性：
>
    set: (target, key, value) => {
        report("Writing" + key + " through a proxy");
        target[key] = value ;
    }

很自然地，新创建的属性 都可以通过代理对象或者目标对象访问。    

这就是如何使用proxies的要点：通过Proxy构造器，我们创建了一个代理对象 其可以控制对目标对象的访问 无论何时一个操作
直接在代理对象上执行  通过激活特定的traps 。

此例中我们只是使用了 get 和set traps，但还有其他更多的其他traps 我们可以对不同的对象动作 定义处理器。
* apply trap 在调用一个函数时被激活， 和 **construct** trap 当使用new操作符时激活。
* get 和 set trap当在读/写一个属性时被激活
* enumerate trap 会在for-in 语句中被激活。
* getPrototypeOf 和 setProtorypeOf 会在获取或者设置原型值时被激活。

我们可以拦截很多操作，但这里不会统统把他们都介绍一遍的。

现在，我们把注意力转向我们不能忽略的一些操作：
== 或者 === ，  instanceof ， 和 typeof 操作符。

表达式 x == y (或者更严格的比较 x === y) 用来检测是否x和y指向同一个对象（或者拥有相同的值）。等号操作符 有一些假设。
比如 两个对象的比较 对相同的两个对象应该返回相同的相同的值，这不是我们能保证的东西 如果值是由用户指定的函数决定的。
此外，两个对象的比较行为不应该给予对这些对象的访问权 ，这就是相等性是否应该被trapped 。
相似原因， instanceof 和 typeof操作符 不能被trapped 。

既然现在我们已经知道了proxies代理是如何工作的 并且如何创建他们，现在让我们看一些实际的东西,
比如，如何使用代理 用于 logging ， 执行性能计算 ，自动补全属性，并实现数组可以通过负索引访问这些功能。

### 使用代理用来logging

logging对找bug是很重要的，在特定的时候 输出我们觉得有用的信息。 我们可能 比如 想知道那个函数被调用了，他们被执行了多长时间
什么属性被读或者写了，等等 。

不幸的是 当实现logging时，我们经常吧logging语句 在代码中辐射的到处都是。
~~~

    function Ninja() {
        let _skillLevel = 0;

        Object.defineProperty(this, 'skillLevel', {
            get: () => {
                report("skillLevel get method is called");
                 return _skillLevel;
            },
            set: value => {
                report("skillLevel set method is called");
                _skillLevel = value;
            }
        });
    }
    const ninja = new Ninja();
    ninja.skillLevel;
    ninja.skillLevel = 4;
~~~

使用代理，更好和更干净的方式：
~~~

function makeLoggable(target){
    return new Proxy(target, {
        get: (target, property) => {
            report("Reading " + property);
            return target[property] ;
        },

        set: (target, property, value ) => {
            report("Writing value " + value + " to "+property);
            target[property] = value ;
        }
    });
}

let ninja = {name: "Yoshi"} ;
ninja = makeLoggable(ninja) ;

// 读写我们的代理对象，这些动作会被logged 通过代理的traps 。
assert(ninja.name === "Yoshi", "Our ninja Yoshi") ;
ninja.weapon = "sword";


~~~

这个对比使用getter和setters 来说更简单和透明了 。
我们不必 混合我们的领域代码 和我们的logging代码（关注点分离！），也没有为每个对象属性添加logging的必要了 。
代之的是，所有的属性读和写都是通过我们代理对象的trap函数了 。 日志记录只在一个地方指定并可以在需要时 在任意的对象上 被重用多次。


### 使用代理 来做性能计算（measuring performance）

甚至不用修改一个函数的源码，我们就可以对函数的调用进行性能计算。
比如 我们想计算一个函数的性能 ，该函数用来计算一个数是否是素数。
~~~

    function isPrime(number){
        if(number < 2){ return false ;}

        for(let i=2; i<number; i++){
            if(number % i === 0){ return false ; }
        }

        return true ;
    }

    isPrime = new Proxy(isPrime, {

        // 提供了一个apply trap 无论何时代理被当做函数调用时就会调用它 。
        apply: (target, thisArg, args) => {
            console.time("isPrime");

            const result = target.apply(thisArg, args);

            console.timeEnd("isPrime");

            return result ;
        }
    });

    isPrime(1299827) ;
~~~

无论何时，只要isPrime函数被调用，该调用就被路由到我们的代理的apply trap上，它会启动一个stopwatch 通过内置的console.time 函数
调用原始的isPrime函数，log 消耗的时间，最后返回isPrime调用的结果 。

### 使用代理去自动填充属性

在get trap上检测 值是否存在 如果不存在 给它一个我们预想的值 。


### 使用代理来实现 数组的负索引

~~~
    const ninjas = ["Yoshi","Kuma", "Hattori"];

    ninjas[-1] ;
    ninjas[-2] ;
    ninjas[-3] ;
~~~

JS并没有内置的数组负索引的支持，但我们可以通过代理来模拟这种功能。

~~~

    function createNegativeArrayProxy(array){
        if(!array.isArray(array)){
            throw new TypeError('Expected an array');
        }

        return new Proxy(array, {
            get: (target, index) => {
                index = +index ;
                return target[index <0 ? target.length + index : index];
            },
            set: (target, index, val) =>{
                index = +index ;
                return target[index < 0 ? target.length+index: index] = val ;
            }
        });
    }

    const ninjas = ["Yoshi", "Kuma", "Hattori"];
    const proxiedNinjas = createNegativeArrayProxy(ninjas);

    assert(ninjas[0] === "Yoshi" && ninjas[1] === "Kuma"
            && ninjas[2] === "Hattori",
            "array items accessed through positive indexes");

    assert(typeof ninjas[-1] === "undefined"
        && typeof ninjas[-2] === "undefined"
        && typeof ninjas[-3] === "undefined",
        "Items cannot be accessed through negative indexes on an array");

    assert(proxiedNinjas[-1] === "Hattori"
        && proxiedNinjas[-2] === "Kuma"
        && proxiedNinjas[-3] === "Yoshi",
        "But they can be accessed through negative indexes");

    proxiedNinjas[-1] = "Hachi";
    assert(proxiedNinjas[-1] === "Hachi" && ninjas[2] === "Hachi",
        "Items can be changed through negative indexes");            
~~~

### proxies 的性能耗损

代理的最主要缺点

如我们所知，一个proxy是一个代理对象 通过它我们可以控制对另一个对象的访问。 一个proxy 可以定义traps，这些函数当会在代理上执行特定的操作时
被执行 。并且，如你所见，我们可以使用这些traps 来实现有用的功能 比如logging， 性能测算，属性自动补全，负数组索引，等等。

不幸的是，也有一个缺点，实事是所有我们的操作不得不传递给代理 添加了一个间接层 通过此层 我们可以实现所有的这些帅酷特性，但同时它也引入了
一些不容忽视的 会影响性能的 额外处理 。
~~~
// 检测代理的性能局限

    function measure(items){
        const startTime = new Date().getTime() ;
        for(let i = 0; i < 500000; i++){
            items[0] === "Yoshi";
            items[1] === "Kuma";
            items[2] === "Hattori";
        }
        return new Date().getTime() - startTime ;
    }

    const ninjas = ["Yoshi", "Kuma", "Hattori"];
    const proxiedNinjas = createNegativeArrayProxy(ninjas);

    console.log("Proxies are around",
                Math.round(measure(proxiedNinjas) / measure(ninjas) ),
                "times slower");
~~~
运行结果，
在chrome上 代理大概比原生的慢50倍，在firefox上 大概20倍慢。

至此，我们推荐你当使用代理时小心点。尽管允许你创新性地对对象控制访问，太多的控制会遇到性能问题。你可以在性能不敏感的地方使用代理，
但当使用的代码被经常执行时 ， 请小心 。如常，我们推荐你大概测试下你代码的性能 。

