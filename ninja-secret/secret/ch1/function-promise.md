
## 使用promises

ES6中 引入的新特性 用于处理异步任务的。

Promise 是对一个值的占位符 虽然目前并没有 但后面的某个时间总会有的；是一个我们会最终知道一个异步计算结果的承诺。
如果我们的承诺不错的话 我们的结果会是一个值，如果发生了问题，我们的结果就会是一个错误 一个我们不能送达的歉意。

一个很好的例子就是 使用promises 从服务器取数据；我们承诺我们最终会得到数据的，但总有发生问题的机会 。

~~~

    const ninjaPromise = new Promise((resolve, reject) => {
        resolve("Hattori");
        // reject("An error resolveing a Promise! ");
    });

    ninjaPromise.then(ninja => {
        assert(ninja === "Hattori","We were promised Hattori! ");

    } ,  err => {
        fail("There shouldn't be an error ")
    });
~~~
为了创建Promise 我们使用 new 内建的Promise构造器，我们为其传递了一个函数，在此 是一个箭头函数 （也可以简单地使用函数表达式）。
这个函数 称之为 执行器 函数（executor），有两个参数 resolve 和 reject 。

当 使用两个内建函数作为参数：resolve（如果承诺成功解析我们可以手动调用它）和reject（如果发生错误调用这个）
 构造Promise对象时 执行器被立即调用。

 这段代码使用promise 通过在Promise对象上调用内建的then方法 我们为此方法传递了两个函数：一个success回调 和一个 failure回调。
 前者当承诺成功解析时调用（如果resolve函数在promise上被调用），后者是当有问题时被调用（无论 未处理异常被调用 或者reject函数在promise上
 被调用） 。

 在我们的例子代码中，我们创建了一个promise 并即时resolve它 通过使用参数Hattori 调用resolve函数。因而 当我们调用then方法时 第一个，
 success，回调函数被执行 

 接下来明白 promises要解决什么问题。

 ### 理解简单回调导致的问题

 我们使用异步代码是因为我们在运行长时间运行的任务时 不想锁住我们程序的执行（这会让用户失望的）。
 当前 我们使用回调解决这个问题： 对长期运行的任务 我们提供了一个回调函数，此回调当任务最重完成时被调用 。

 比如，从一个服务器取回一个json文件就是一个长期运行的任务，在此期间 我们不想使我们的程序对用户不响应。因而我们提供了一个回调 它会在任务完成时被调用
 ：
 >
    getJSON("data/ninjas.json", function(){
        /*  处理结果  */
    });

很自然，在长期运行任务时  错误可能发生。使用callback的问题是你不能使用内建的语言结构 比如 try-catch语句 ，如下面那样：    
~~~
    try{
        getJSON("data/ninjas.json",function(){
            // 处理结果
        });
    }catch(e) {
        /*  处理错误  */
    }
~~~

因为回调和长期运行的任务执行不在同一个事件循环中

后果是 ， 错误经常丢失了，很多库 因而 定义他们自己的惯例做法来报告错误。比如 在Node.js 世界，回调经常持有两个参数：  err和data
如果error发生了 err就是一个非null值 。这就导致callback的第一个问题： 错误处理困难
（在 go语言中 经常有所谓的 OK-Pattern ）

在我们执行了一个长期运行任务后 ，我们经常像用获取的数据做些什么。这会导致另一个长期运行的任务，这又会偶尔导致另一个长期运行的任务，
... 如此... --- 导致了一系列互相依存，异步的，回调处理步骤。
~~~

    getJSON("data/ninjas.json", function(err, ninja){
        getJSON(ninjas[0].location, function(err, locationInfo){
            sendOrder(locationInfo, function(err, status){
                /*  Process status */
            })
        })
    });
~~~ 

你可能不止一次遇到这样的代码，一系列内嵌的回调 表示下一步需要的步骤 。
你会注意到 这些代码很难理解，错误处理复杂 ..
你得到一个 “房间的金字塔” 它持续增长 并很难管理。
这引导我们到了 回调的第二个问题 ： 执行序列步骤是很tricky（诡异）的 。

有时，这些步骤不必一个接一个的完成来得到最终结果 步骤间没有互相依赖，所以没必要让他们在一个序列上。代之 为了节省那么珍贵的几毫秒时间
我们可以并行执行他们。

~~~

    var ninjas , mapInfo , plan ;

    $.get("data/ninjas.json", function(err, data){
        if(err){ processError(err); return  ; }
        ninjas = data ;
        actionItemArrived() ;
    });
    $.get("data/mapInfo.json", function(err, data){
        if(err){ processError(err); return  ; }
        mapInfo = data ;
        actionItemArrived() ;
    });

    $.get("plan.json", function(err, data){
        if(err){ processError(err); return ; }

        plan = data ;
        actionItemArrived() ;
    });

    function actionItemArrived(){
        if(ninjas != null && mapInfo != null && plan != null){
            console.log("The plan is ready to be set in motion!" ) ;
        }
    }

    function processError(err){
        alert("Error", err ) ;
    }
~~~
该例中，因为action间互不依赖 我们 并行执行了这些任务 最终我们所有的数据会就位，因为我们不知道数据到达的顺序，每次我们一获取到数据，都会检测
，最终当所有的片段就位后 ，我们就可以开始我们的计划了 。

这引导我们来到了 第三个回调的问题：  并行执行一系列步骤 也triky 。

很多程序员都在这些问题的探究上做了努力，并有自己的方案诞生

JS 背后的团队给我们推荐了标准方案 **promises** 用来解决异步计算问题 。

###  深入promises

一个Promise 是一个对象  ， 它像一个 异步任务执行结果的占位符 对外服务。 它表示一个值 虽然当前没有值 但希望在将来存在。

因为这些原因，在其声明周期中，一个promise可以穿越 一些个状态 。

一个promise 始于一个**pending**状态， 此时我们对承诺的值一无所知 。 这就是问什么一个promise在pending状态也被称为 unresolved promised的原因
。 在程序执行中，如果一个promise的resolve函数被调用 ，promise就会移到**fulfilled**状态， 在此时 我们成功的获取到了承诺的值。 
另一方面 ，如果 promise的reject函数被调用 ，或者一个未处理异常发生于promise处理过程中， promise就会移入到**rejected**状态，此时我们就不能
获取到承诺的值，但我们最终也会知道原因的。 一旦一个promise 到达**fulfilled**状态或者**reject**状态 ，他就不能转换（
一个承诺 不能从 fullfilled状态 迁到 rejected状态 或者 反之也不可以 ），并且他会一直呆在那个状态 。我们称这个promise 是被 resolved（
或者成功或者不 ）。


~~~
    report("At code start ");

    var ninjaDelayedPromise = new Promise( (resolve, reject ) => {
        report("ninjaDelayedPromise executor");

        setTimeout(()=>{
            report("Resolving ninjaDelayedPromise");
            resolve("Hattori");
        }, 500);
    });

    assert(ninjaDelayedPromise !== null , "After creating ninjaDelayedPromise ");

    ninjaDelayedPromise.then(ninja => {
        assert(ninja === "Hattori", 
                " ninjaDelayedPromise resolve handled with Hattori  ");
    });

    const ninjaImmediatePromise = new Promise((resolve, reject) => {
        report(" ninjaImmediatePromise executor. Immediate resolve.");
        resolve("Yoshi") ;
    })

    // 注册回调
    ninjaImmediatePromise.then(ninja => {
        assert(ninja === "Yoshi",
                " ninjaImmediatePromise resolve handled with Yoshi ");
    });

    report("At code end ") ;

~~~

Promise 被设计用来处理异步动作，，为了使行为可预测 , 所以JS 引擎总是凭借异步处理。
JS引擎通过 在当前事件循环的步骤中的所有代码之后 执行then回调来确保。

生活中并不总是阳光和彩虹。

### Rejecting Promises 拒绝承诺
有两种方式拒绝承诺 ：
- 显式地    通过在promise的executor函数中调用传入的reject函数 ，
- 隐式地    如果当处理一个承诺时，一个未处理异常发生

~~~

    const promise = new Promise((resolve, reject) => {
        // 一个承诺可以通过调用传入的reject函数来显式拒绝承诺
        reject("Explicityly reject a promise ");
    });

    // 如果promise被拒绝，第二个 错误 回调会被调用。
    promise.then(() => fail("Happy path , won't be called "),
    error => pass("A promise  was explicityly rejected! "));
~~~

通过调用传入的reject函数 可以显式拒绝一个承诺 。 如果承诺被拒绝，当通过then方法注册回调时，第二个，error，错误回调将总是被调用 。
此外，我们也可以使用另一个可选语法来处理承诺拒绝， 通过使用内置的catch方法：

~~~

    var promise = new Promise((resolve, reject)=> {
        reject("Explicityly reject a promise!");
    });

    // 除了提供第二个错误回调，我们也可以链式使用catch方法 给他传递错误回调，最终结果是一致的！
    promise.then(()=> fail("Happy path, won't be called!"))
            .catch(()=>pass("Promise was also rejected") );
~~~
这个只是个人喜好问题 两种方法都效果等同。

但当使用链式promises时 我们会看到使用链式的catch方法更有用 。

~~~
    //  如果一个未处理异常在处理过程中发生 承诺会隐式被拒绝
    const promise = new Promise((resolve, reject ) => {
        undeclaredVariable ++ ; // 故意制造一个未定义变量的操作
    });

    promise.then(()=> fail("Happy path, won't be called "))
            .catch(error => pass("Third promise was alse rejected "));
~~~

统一处理方式 不管承诺如何被拒绝（显示或者隐式） 所有的错误和拒绝原因被指引到我们的rejection回调中。
这 使我们生活更轻松 。

### 创建我们第一个 真实环境下的promise

         从服务器取数据，这是很好的使用promise的案例。

~~~

    function getJSON(url){
        return new Promise((resolve, reject) => {
            const request = new XMLHttpRequest() ;

            request.open("GET", url);

            request.onload = function() {
                try{
                    if(this.status === 200 ){
                        resolve(JSON.parse(this.response));
                    }else{
                        reject(this.status + " " + this.statusText);
                    }
                }catch(e){
                    reject(e.message);
                }
            };

            request.onerror = function(){
                reject(this.status + " " + this.statusText);
            }

            request.send() ;
        });
    }

    getJSON("data/ninjas.json").then(ninjas => {
        assert(ninjas !== null , "Ninjas obtained!");
    }).catch(e => fail("shouldn't be here :"+ e));
~~~

注意getJSON方法中 会有多处出现问题： 网络连接 服务器发送非法状态 格式非json ... 都会导致问题！这些都被涵盖处理了！

现在我们来看看另一种用法 ： 优雅的组合 链式

### 链接promises

互相依赖的处理步骤形成了“pyramid of room” （金字塔）很难维护的深层嵌套的回调。 
Promises 可以解决这个问题， 他们有一种链式的能力。

前面我们看到了可以在一个 promise上使用then方法，我们可以藉此注册一个回调 它将在promise成功resolved后被调用 。我们没有告诉你的是
调用then方法 依旧返回一个新的 promise。

所以没有什么能停止我们继续调用任意多个then方法的东西了

~~~

    getJSON("data/ninjas.json")
        .then(ninjas => getJSON(ninjas[0].missionsUrl))
        .then(missions => getJSON(missions[0].detailsUrl))
        .then(mission => assert(mission !== null , "Ninja mission obtained!"))
        .catch(error => fail("An error has occurred")) ; // 可以在任意一步捕获promise的rejection
~~~

这样的代码如果用常规回调来写 会导致很深的嵌套 ，识别某个特定的步骤也不容易 ，鬼知道我们会不会在中间某个步骤插入另一个步骤 。

#### 在链式承诺中捕获错误
当处理异步阶段序列是，错误可能发生在任意一步 。我们已经知道 对于promise 我们可以提供第二个错误回调给then方法或者使用链式的catch方法
（它接受的也是错误回调方法）。 
当我们只关注 整个序列的成功或者失败时  为每个步骤都提供错误处理是很繁琐的 。
（思考 错误是不是会从第一个传递到最后一个？）

### 等待多个promises

~~~

    Promise.all([getJSON("data/ninjas.json"),
                getJSON("data/mapInfo.json"),
                getJSON("data/plan.json")
    ]).then(results => {
        const ninjas = results[0], mapInfo = results[1], plan = results[2];

        assert(ninjas !== undefined
            && mapInfo !== undefined
            && plan !== undefined,
            "The plan is ready to be set in motion!");

    }).catch(error => {
        fail("A problem in carring out our plan !");
    })
~~~

Promise.all 方法使用 一个promise数组创建一个新的promise， 如果所有的这些promiss成功才算这个promise成功
如果有一个失败 那便算失败 。

可以看到 我们不用关心这些任务的执行顺序，是否他们中的一些是否完成，其他一些未完成。

Promise.all 方法 在一个列表中 等待所有的promise 。

有时候我们只关心多个promises中 第一个的成败 ，  遇见 Promise.race 方法

#### RACING　Promises
~~~
Promise.race([
    getJSON("data/Yoshi.json"),
    getJSON("data/hattori.json"),
    getJSON("data/hanzo.json")
]).then(ninja => {
    assert(ninja !== null , ninja.name + "responded first ");
}).catch(error => fail("Failure!"));
~~~

Promise.race方法 接受一个promise数组 并返回一个全新的promise 他resolve或者reject 只要这些promise中的第一个 resolve或者reject了。

题外话：
go语言 做并发处理 例子中 经常出现的场景：  用户输入了一个关键字  处理器接收到后 并发发送这个搜索请求 给多个后台搜索服务 只要有一个返回了结果
那么 就算搜索成功 返回第一个响应的处理给终端用户 。


## 联合使用 generator和 promises
通过例子来学习

同步方式
~~~

    try{
        const ninjas = syncGetJSON("data/ninjas.json");
        const missions = syncGetJSON(ninjas[0],missionsUrl);
        const missionDetails = syncGetJSON(missions[0].detailsUrl) ;
        // Study the mission description
    }catch(e){
        // Oh no, we weren't able to get the mission details 
    }
~~~
虽然简单 且有错误处理 但他会锁住UI 
理想情况 ，我们会将之变为非阻塞的长期运行任务 ，一种手段就是 合并使用 generators和promises。

如我们所知，从生成器 yielding 会挂起生成器的执行但并不会阻塞（执行上下文被弹出 但并未销毁！），为了唤醒生成器并继续器执行，我们不得不
在生成器的迭代器上调用next方法 。Promises 另一个方面，允许我们指定一个回调 它会在我们能够获取到承诺值时被触发，另一个回调会在错误时被触发
。
想法就是 ， 这样合并使用generators和proises ：
我们把使用异步任务的代码放到生成器 ，并执行哪个生成器函数， 当我们达到生成器执行的一个点 在该点调用一个异步任务，我们创建一个promise 用其来
表示异步任务的值。
因为 我们不知道什么时候promise会被resolved（或者将被resolved），在生成器执行的这个点，我们从生成器yield，所以我们并不会导致阻塞。
过段时间，当promise被设置好，我们通过调用迭代器的next方法继续我们生成器的执行。我们可以根据需要调用任意次。

~~~

    async(function*(){
        try{
            const ninjas = yield getJSON("data/ninjas.json");
            const missions = yield getJSON(ninjas[0].missionsUrl);
            const missionDescription = yield getJSON(missions[0].detailsUrl);

            // Study the mession details
        }
        catch(e){
            // Oh no , we weren't able to get the mission details
        }
    });

    function async(generator){
        var iterator = generator() ;

        function handle(iteratorResult) {
            if(iteratorResult.done){ return ; }
            
            const iteratorValue = iteratorResult.value ;

            if(iteratorValue instanceof Promise){
                iteratorValue.then(res => handle(iterator.next(res)))
                             .catch(err => iterator.throw(err)) ;
            }
        }
        try{
            handle(iterator.next()) ;
        }catch(e) {
            iterator.throw(e) ;
        }
    }
~~~