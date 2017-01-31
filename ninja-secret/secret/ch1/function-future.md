# Generator 和 Promise

两个都是ES6新增特征

Generators 是特殊类型的函数 。标准的函数 当从头至尾运行其代码时 至多产生一个值 ，generators产生多个值，基于每请求，但会在这些请求间
挂起其执行 。

生成器经常被认为附加或者怪异的语言特性，不是经常被程序员使用。

Promises 是新的内建对象类型 可以帮助你完成异步代码工作 。一个promise是一个当前还没有值但后期的某个时间点会有的占位符。

## 使用generators和promises使我们的异步代码更优雅

~~~
    try{
        var ninjas = syncGetJSON("ninjas.json");
        var missions = syncGetJSON(ninjas[0].missionsUrl);
        // study the mission description
    }
    catch(e){
        // Oh no, we weren't able to get the mission details
    }
~~~
此段代码相对容易理解，如果在任何一步发生错误，我们可以使用catch很容易捕获到 。 但不幸的是，此代码有一个大问题。从服务端获取数据是一个
长期运行的操作，因为JS依赖 单线程执行模型 ，我们会锁住我们的UI 直到长时间运行的代码完成 。这会导致非响应式程序 让用户失望。
为了解决这个问题 我们可以用回调重写他，回调会在一个任务完成后被调用 ，不会锁住UI ：
~~~

    getJSON("ninjas.json",function(err, ninjas){
        if(err){
            console.log("Error fetching list of ninjas " , err) ;
            return ;
        }
        getJSON(ninjas[0].missionUrl, function(err,missions){
            if(err){
                console.("Error locating ninja missions",err);
            }
            getJSON(missions[0].detailsUrl, function(err, missionDetails){
                if(err){
                    console.log("Error locating mission details", err) ;
                    return ;
                }
                // study the intel plan 
            });
        });
    });
~~~

尽管这段代码更可被用户接受，你肯定认同 这段代码太混乱了，它添加了很多模板式的错误处理代码，丑 。
这就是generator和promises出现的地方 ，通过组合使用他们，我们可以转换 非阻塞但比较丑的回调代码成一些更优雅的代码：
~~~
    /**
    *   生成器函数通过在function关键字右侧放一个星号* 来定义
    */
    async(function*(){
        try{
            const ninjas = yield getJSON("ninjas.json");
            const missions = yield getJSON(ninjas[0].missionsUrl);
            const missionDescription = yield getJSON(missions[0].detailsUrl);
            // study the mission details
        }
        catch(e){
            // Oh no , we weren't able to get the mission details 
        }
    })
~~~

## 使用generator函数

生成器是一个完全的新函数类型，和常规函数有极大不同。
生成器是一个函数 他生成一系列的值，但并不是一次全部生成，同常规函数那样，基于每请求。
我们必须显式向生成器请求新值，生成器或者用一个值来响应或者通知我们没有更多值产生啦 。
需要注意的是 在一个值被产生后 ，生成器函数不会像常规函数那样结束其执行。换之，生成器仅仅是挂起了。之后 当对另一个值的请求到来时，生成器
从离开点（挂起那刻）恢复。

~~~
// 通过在function关键字后放一个* 来定义一个生成器函数
function* WeaponGenerator(){
    // 在生成器函数中 使用 yield 产出单个值
    yield "Katana";
    yield "Wakizashi";
    yield "Kusarigama";
}
// 使用新的 for of 循环来消费生成的序列
for(let weapon of WeaponGenerator()){
    assert(weapon !== undefined , weapon) ;
}
~~~

注意看到 在我们的生成器函数中没有 return 语句。

生成器和标准函数很不一样，起初，调用一个生成器函数并不会执行生成器函数； 代之 它会创建一个称之为 iterator的对象 。

## 通过iterator对象 来控制生成器
对生成器的调用 并不意味生成器函数体被执行，换之，一个迭代器对象被创建，通过这个对象可以联通到生成器。
比如可以使用迭代器来请求额外的值。

~~~
// 定义一个会生成两个武器序列的迭代器
function* WeaponGenerator(){
    yield "Katana";
    yield "Wakizashi";
}
// 调用生成器 创建一个迭代器 通过它来控制生成器的执行
const weaponIterator = WeaponGenerator() ;

// 调用迭代器的next函数从生成器请求一个新值
const result1 = weaponIterator.next() ;
结果是一个对象，有一个返回值（.value） 和一个指示器(.done) 告诉我们生成器是否还有更多值
assert(typeof result1 === "object"
        && result1.value === "Katana"
        && !return.done,
        "Katana received! ");

    // 再次调用next 从生成器获取另一个值
    const result2 = weaponIterator.next() ;
    assert(typeof result2 === "object"
            && result2.value === "Wakizashi"
            && result2.done, 
            "Wakizashi received! ");

    // 当没有更多代码执行时 生成器返回“undefined” 并指示已经完毕done了
    const result3 = weaponIterator.next() ;
    assert(typeof result3 === "object"
            && result3.value === undefined
            && result3.done,
            "There are no more result ! ");

~~~
如你所见，当调用一个生成器时 一个新的iterator对象被创建
迭代器使用来控制器生成器执行的。
>
    const weaponIterator = WeaponGenerator() ;
 一个迭代器对象最基本的事情之一是它暴露了一个next方法，我们可以通过从其请求新值进而用其控制生成器。
 >
    const result1 = weaponIterator.next();

作为对其调用的响应，生成器执行器代码直到遇到一个yield关键字 它会生成一个即时结果（生成项序列中的一个项），并返回一个新 new的对象
它封装那个结果并告诉我们其值是否已经玩了！

只要当前值被生成，生成器就挂起其执行 而不会锁住并潜在地等待另一次值请求。 这是一个标准函数不具备的极其强大的特性。

在此，第一次对迭代器next方法的调用执行生成器代码到第一个 yield 表达式。 yield 出 “Katana” 并返回一个对象 其属性value被设置为Katana 
另一个属性done设置为false，预示有更多的值可以产出 。

之后， 我们从生成器请求另一个值，通过对 weaponIterator的next方法做另一次调用：
>   const result2 = weaponIterator.next() ;

这会把生成器从挂起状态唤醒，生成器会从其上次离开的地方继续，执行其代码直到另一个中间值到达： yield "Wakizashi".
这挂起生成器并生成一个持有 Wakizashi 的 对象 。

最终，当我们第三次调用调用next方法时，生成器恢复其执行。 但这次没有更多的代码执行了，所以生成器返回一个对象，其value设置为undefined，
done设置为true 。 预示工作完成了 。

既然你已经明白了如何通过迭代器控制生成器，你将开始了解如何迭代生成的值 。
### 迭代 迭代器
通过调用生成器 生成迭代器， 暴露next方法 通过它我们可以从生成器请求一个新值 。 next方法返回一个对象，它持有由生成器产生的值 也有一个信息存储
在done属性上 告诉我们生成器是否还有额外的值会生成 。

~~~

    function* WeaponGenerator(){
        yield "Katana" ;
        yield "Wakizashi" ;
    }
    const weaponIterator = WeaponGenerator() ;
    let item ;
    while(!(item = weaponIterator.next()).done){
        assert(item !== null, item.value) ;
    }
~~~

for-of 是语法糖用于迭代一个迭代器：
>
    for(var item of WeaponGenerator() ){
        assert( item !== null , item ) ;
    }

代之 手动调用匹配迭代器的next方法 并总检查是否完成了，我们可以使用for-of循环来做相同的事情 ，只是在后台背地下而已 。

### yielding to another Generator
同我们经常从一个其他的标准函数调用另一个函数，在特定情况下 我们可以代理对一个生成器的执行给另一个 。

~~~

    function* WarriorGenerator(){
        yield "Sun Tzu";
        // yield* 委托给另一个迭代器
        yield* NinjaGenerator();
        yield "Genghis Khan";
    }

    function* NinjaGenerator(){
        yield "Hattori";
        yield "Yoshi";
    }
   
   for(let warrior of WarriorGenerator() ){
       assert(warrior !== null , warrior )
   }
~~~    

通过在迭代器上使用 yield* 操作符，我们yield 到另一个生成器。
在本例中，从 WarriorGenerator 我们yielding了一个新的 NinjaGenerator； 所有在当前 WarriorGenerator迭代器的next方法上的调用被重新路由到
NinjaGenerator 。 这会直到 NinjaGenerator 没有剩余工作了 。
注意 对于调用原始生成器的代码 这些是透明发生的 。
for-of 循环并不关心 WarriorGenerator 生成另一个生成器； 它持续调用next 直到其完成工作 。

接下来实际例子

### 使用生成器

使用生成器生成ids

当创建对象时 我们经常需要给每个对象赋值一个id，最简单的方式就是通过一个全局的计数器变量，但这也是一种丑陋，因为 变量可能意外从我们代码的其他地
方被扰乱。另一种选择是使用生成器。

~~~

    function *IdGenerator(){
        let id = 0 ;
        while(true){
            yield ++ id ;
        }
    }

    const idIterator = IdGenerator() ;

    const ninja1 = { id: idIterator.next().value } ;
    const ninja2 = { id: idIterator.next().value } ;
    const ninja3 = { id: idIterator.next().value } ;
~~~

NOTE 在标准函数中 写出无限循环通常并不是我们想要的。 但对于生成器，一切ok ！任何时刻生成器遇到一个yield语句，生成器执行被挂起暂停
直到next方法被再次调用。所以 每次 next调用只执行我们无限while循环中的一次 并把下次id值发回 。

### 使用生成器来遍历DOM
web page的布局是基于DOM的，html节点的树形结构 ，在其中的每个节点，除了root那个，都有一个parent，可以有零个或者多个children。
因为DOM在web开发中是如此的基础，我们的很多代码都是关于迭代他们 。一种比较简单的方式是通过实现一个递归函数（对于每个遍访的节点都会调用）

~~~

<div id="subTree">
    <form>
        <input type="text" />
    </form>
    <p>Paragraph</p>
    <span></span>
</div> 

<script>
    function traverseDOM(element, callback){
        callback(element) ; // 使用回调函数处理当前节点
        element  = element.firstElementChild ;
        while(element){
            // 遍历每个孩子元素的DOM
            traverseDOM(element, callback);
            element = element.nextElementSibling ;
        }
    }
    const subTree = document.getElementById("subTree");
    // 开始全部的处理通过调用traverseDOM函数 在我们的根元素上
    traverseDOM(subTree, function(element){
        assert(element !== null , element.nodeName );
    });
</script>
~~~

我们也可以使用生成器来做
~~~

    function* DomTraversal(element){
        yield element ;
        element = element.firstElementChild ;
        while(element){
            // 通过使用yield* 传递迭代控制给另一个DomTraversal生成器实例
            yield* DomTraversal(element);
            element = element.nextElementSibling ;
        }
    }

    const subTree = document.getElementById("subTree");
    // 通过使用for-of循环遍历节点。
    for(let element of DomTraversal(subTree)){
        assert(element !== null , element.nodeName );
    }
~~~
我们消除了 回调！

### 同generator通信

生成器 还有另一个重要功能， 我们可以发送数据给生成器， 因而实现双向通讯！
使用生成器，我们可以产出一个中间结果，使用这个结果从生成器的外面做一些计算，之后当我们准备好了，完整地发送新数据返回到生成器并恢复其执行。

#### 以生成器函数参数发送值
最简单的发送数据给生成器 是通过视其为任何其他函数那样 并使用函数调用参数。

~~~
    // 同其他任何函数那样，生成器可以接收标准参数
    function* NinjaGenerator(action){
        // 魔法发生这里。通过yielding 一个值，生成器返回一个中间计算。
        // 通过调用迭代器的next方法并传递一个参数 ，我们发送数据返回到生成器 。
        const imposter = yield(" Hattori " + action) ;
        
        assert(imposter === "Hanzo",
                "The Generator has been infiltrated ");

    yield( "Yoshi ( " +  imposter + " ) " + action );                
    }

    const ninjaIterator = NinjaGenerator("skulk");

    const result1 = ninjaIterator.next() ;
    assert(result1.value === "Hattori skulk", "Hattori is skulking");

    const result2 = ninjaIterator.next("Hanzo");
    assert(result2.value === "Yoshi (Hanzo) skulk","We have an imposter!");

~~~

函数接受数据并没有特殊的； 常规的函数总是这样。但 记住，生成器有此强大功能；他们可以暂停和恢复 。
被发现是这样的，不同常规函数，生成器在其执行开始后甚至能够接受数据，通过请求下次值我们可以随时恢复他们。

### 通过使用next方法来发送数据给生成器

此外在第一次调用生成器提供数据，我们可以发送数据给生成器通过传递参数给next方法。在处理中，我们从挂起态唤醒生成器并恢复其执行。
这个被传递的值被生成器使用作为整个yield表达式的值。

我们使用生成器双向通讯的方式，使用yield 来从生成器返回数据，iterator 的 next() 方法来传递数据反回给生成器 。

NOTE next方法提供值给等待着的yield表达式，所以如果没有yield表达式等待，就没有什么能提供值给它。 因为这个原因，我们不能在第一次调用next方法
时提供值给它。 但记着，如果你需要给生成器提供初始值，可以在调用生成器自己时做。

### 抛出异常

有另一个，有点非常规的 为生成器提供值的方式： 通过抛出一个异常。 每个迭代器，除了有一个next方法， 还有一个throw 方法，我们可以使用它
抛递一个异常返回到生成器。

~~~

    function* NinjaGenerator(){
        try{
            yield "Hattori";
            fail("The excpected exception didn't occur");
        }
        catch(e){
            assert(e === "Catch this!", "Aha! We caught an exception");
        }
    }

    const ninjaIterator = NinjaGenerator() ;
    
    // 从生成器拉出一个值
    const result1 = ninjaIterator.next() ;
    assert(result1.value === "Hattori", "We got Hattori");

    // 抛出一个异常给生成器
    ninjaIterator.throw("Catch this!");
~~~

注意  try-catch 块 


### 揭露generators底层机理

至此，我们知道调用一个生成器并不会执行他，换之，它创建一个新的迭代器，我们可以用其从生成器请求值。在生成器产出（or yields）一个值后，
它会暂停|挂起 其执行 并等待下次请求 。
所以 在某种程度，生成器 工作方式像是一个小程序，一个在状态间移动的状态机：
- 挂起开始     当生成器被创建，它始于这个状态。没有任何生成器的代码被执行。
- 执行         在此状态生成器的代码被执行。 执行继续或者从开始或者从生成器最后一次挂起的地方开始 。一个生成器在匹配迭代器的next方法
    被调用时会移到这个状态，并且存在有被执行的代码。
    
- 挂起yield   在执行期间，当一个生成器到达yield表达式时，它创建一个新对象 携带返回值，yields这个对象，（yield 可以翻译为发射）并挂起
 其执行。这就是那个 生成器暂停并等待其继续执行的状态    
 - 完成       如果在执行生成器过程中 不论遇到return语句或者没有代码可执行时 ，生成器会移到这个状态。

 在执行中，通过匹配的迭代器next方法的调用来触动 一个生成器在这些状态间移动。

 ### 用执行上下文来跟踪生成器
 
 生成器仍旧是函数，尽管有些特别。 