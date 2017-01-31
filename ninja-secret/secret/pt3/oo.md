# 使用的原型来做OO

prototype 是一个对象 对特定属性的搜索可以代理到它之上 。原型是一种方便的手段 用来定义属性和功能 自动被其他对象可访问。
原型和经典OO语言中的类目的相似 。 确实，在js中使用原型的主要目的就是为了生成其他语言中的OO代码，但并不全然如 基于类的语言 比如java C#。

## 理解原型
在JS中 对象是拥有值的命名属性的集合，比如 ,我们可以很容易使用对象字面量符号创建一个新对象：
~~~

    let ojb = {
        prop1: 1,             // 简单值
        prop2: function(){},  // 函数
        prop3: {}             // 对象
    }
~~~
此外，JS是一个高度动态的语言，赋值给一个对象的属性可被很容易的改变 通过 **修改**和**删除**既存属性：
~~~
    obj.prop1 = 1 ;
    obj.prop1 = [] ;
    delete obj.prop2 ;

    // 添加新属性
    obj.prop4 = 'Hello';
~~~

为了重用 ，一种帮助我们组织我们程序的手段就是用 继承 。扩展一个对象的特征到另一个对象 。 在js中 继承使用原型来实现 。

原型的思想很简单， 每个对象都可以有一个引用指向他的prototype，如果该对象本身没有那个属性 ， 对特定属性的搜索就可以代理到这个对象上，

比如 一组人在玩一个问答游戏 ， 游戏问你了一个问题，如果你知道答案 你就立即回复了，如果你不知道，你可以问你邻近的人 就这么简单。

~~~

    const yoshi = { skulk: true } ;
    const hattori = { sneak: true } ;
    const kuma = { creep: true } ;

    assert("skulk" in yoshi , "Yoshi can skulk ") ;
    assert(!( "sneak" in yoshi ) , "Yoshi cannot sneak ") ;
    assert("creep" in yoshi , "Yoshi cannot creep ") ;

    // 使用Object.setPrototypeOf 方法来设定一个对象作为另一个对象的原型
    Object.setPrototypeOf(yoshi, hattori);

    assert("sneak" in yoshi, "Yoshi can now sneak");
    assert(!("creep" in hattori), "Hattori cannot creep");

    Object.setPrototypeOf(hattori, kuma) ;
    ...
~~~

为了测试是否一个对象可以访问特定属性，我们可以使用 in 操作符 。

在JS中 ，对象的原型属性是一个内部属性， 它非直接可访问（所以我们标记其为 [[prototype]]）. 
内建的的方法Object.setPrototypeOf 接受两个对象参数 并设定第二个对象作为第一个对象的原型 。

重要强调 每个对象都可以有一个原型，并且一个对象的原型也可以有一个原型，如此下去。形成一个原型链。
搜索委托从一个特定的属性上溯整个链， 当没有更多的原型后会停止。

## 对象构造和原型

最简单的 创建一个新对象的方式就是使用一个语句：
~~~
    const warrior = {} ;
~~~
这创建了一个新的空对象， 之后我们为其填充属性 通过赋值语句:
~~~
    const warrior = {};
    warrior.name = 'Saito';
    warrior.occupation = 'maksman';
~~~
但如果你来自oo背景 你会认为这缺少 构造器的 封装和结构，构造器函数 服务于初始化一个对象到一个可知初始状态。
比较， 如果我们想创建多个同类型对象的多个实例，为每个属性分别赋值不仅繁琐而且容易出错 ，我们希望可以在一个地方为一类对象 加固一些属性和
方法 。

js也提供了这种方法， 使用 new 操作符 通过构造器来初始化一个新对象，但在js中并没有真正的class定义 ，而是 new操作符 作用在一个构造器函数，
触发一个新分配对象的创建 。

每个函数也有一个原型对象， 由该函数创建的对象 其原型会自动被设置为这些对象的原型 。

~~~
    function Ninja(){}
    Ninja.prototype.swingSword = function(){ return true ; }
    
    // 以函数形式调用
    const ninja1 = Ninja() ;
    assert(ninja1 === undefined ,
          "No instance of Ninja created.");
    // 以构造器调用
    const ninja2 = new Ninja() ;
    assert(ninja2 &&
            ninja2.swingSword &&
            ninja2.swingSword(),
            "instance exists and method is callable.");
~~~
每个函数，当创建时 ， 获得一个新的原型对象。当我们用函数作为构造器，被构造的对象的原型被设定为那个函数的原型 。

此例中 当访问ninja2的swingSword属性时，对该属性的搜索会委托到Ninja原型对象上 。注意所有的通过Ninja构造器创建的对象都可以访问这个方法
现在代码被重用了 。

注意 swingSword 方法是Ninja原型的一个属性，而不是ninja实例的属性 

### 实例属性
当一个函数使用new作为构造器被调用时，其上下文被定义为这个新创建的对象实例（this所指）。
此外为了通过原型暴露属性，我们可以在构造器函数中通过this参数来初始化值 。
~~~

    function Ninja(){
        this.swung = false ;
        this.swingSword = function(){
            return !this.swung ;
        };
    }

    Ninja.prototype.swingSword = function(){
        return this.swung ;
    };

    const ninja = new Ninja() ;
    assert(ninja.swingSword(), "Called the instance method, not the prototype method .");
~~~
NOTE 上例说明了属性的优先性。
这说明了实例成员会隐藏定义在原型上的同名属性 。
如果属性能在实例上搜索到 原型甚至不被造访 。

在构造器函数内，this关键字引用的是新创建的对象，所以属性在构造器内直接添加给新创建的ninja对象 。
之后当在ninja上访问属性swingSword时 没有必要遍历原型链；创建于构造器内的属性被立即找到并被返回 。

对于属性定义在 构造器内没什么问题 但对于方法 就有些冗余了 只要创建一个对象 就多一个方法复制 。
对于少量对象的创建没什么问题 ， 但如果是大量对象 ，方法都是相同的未免太浪费内存了 而且没必要。
把对象方法放在函数原型上是有意义的。 因为这样 我们只有一个函数被所有的对象实例共享 。

NOTE 定义于构造器函数中的方法允许我们模拟私有对象变量，如果我们需要这样的东西，在构造器中指定方法将是我们唯一的路 。

### JS动态性的边缘效应

你已经知道 js是一个动态语言，随我们意愿， 属性可以被很容易的 **添加** **移除** 并**修改**。
对函数原型和对象原型，也这样。

~~~
    // 定义一个构造器 创建一个Ninja 有一个单独的布尔属性 。
    function Ninja(){
        this.swung = true ;
    }

    const ninja1 = new Ninja() ;
    // 在对象创建后给原型添加一个方法
    Ninja.prototype.swingSword = function(){
        return this.swung ;
    };

    assert(ninja1.swingSword() , "Method exists, event out of order.");

    // 使用一个新创建的对象 完全覆盖Ninja的原型
    Ninja.prototype = {
        pierce: function(){
            return true ;
        }
    }
    // 即便我们完全替换掉Ninja构造器的原型，我们的实例仍旧可以访问swingSword方法 因为它持有对老的Ninja原型的引用 。
    assert(ninja1.swingSword(), "Our ninja can still swing!");

    // 新创建的实例 引用的就是新的原型了
    const ninja2 = new Ninja() ;
    assert(ninja2.pierce(), "Newly created ninjas can pierce");
    assert(!ninja2.swingSword,"But they cannot swing! ") ;
~~~  

注意  构造器函数原型对象中有一个属性 constructor 该属性 会指向构造器函数的。 也就是 构造器函数 和 其原型对象 互相引用，但如果你完全覆盖
了构造器函数的原型 里面没有constructor属性！

### Object typing via constructor
尽管知道 js如何使用原型来查找正确的属性引用，知道哪个函数构造了一个对象实例也很有用。
构造器函数原型的constructor属性就为此而生。

通过使用constructor属性，我们可以访问用来创建对象的那个函数 。这个信息可以用于某种形式的类型检查。

~~~
    function Ninja(){}
    const ninja = new Ninja() ;

    assert(typeof ninja === "object",
            "The type of the instance is object.");
    assert(ninja instanceof Ninja,
            "instanceof identifies the constructor.");

    // 测试ninja的类型通过constructor 引用。这是到构造器函数的引用。
    assert(ninja.constructor === Ninja,
        "The ninja object was created by the Ninja function.");

~~~
我们定义了一个构造器 并用其创建了一个对象实例。之后我们检测此实例的类型通过使用typeof操作符。这并没有揭露太多东西，因为所有的实例都会是
objects，因此总是返回object作为结果。更多有趣的事情是instanceof操作符，他提供给我们了一种决策是否一个实例是被某个特定函数构造器创建的能力。

此外 我们可以使用constructor属性，现在我们知道它对所有的实例都是可访问的，作为一个引用指向创建对象的原始函数。
我们可以用此来检验实例的出处（origin 源）。

此外，因为这只是对原始构造器的一个引用，我们可以使用它实例化一个新的Ninja对象。
~~~
    function Ninja(){}

    const ninja = new Ninja() ;
    const ninja2 = new ninja.constructor();

    assert(ninja2 instance Ninja , "It's a Ninja!");
    assert(ninja !== ninja2 , "But not the same Ninja!");
~~~

我们可以不必访问原始的构造器函数；我们可以在底下使用这个引用 ，即使原始构造器已经不再域中了（scope）。

NOTE  如果constructor 属性被重写，原始值就丢失了！

虽然有用，但我们接触的仍是原型的皮毛。

## 完成继承

继承是一种形式的重用 新对象可以访问既存对象的属性。这帮助我们避免了代码重复的需求 和数据扩散到我们的代码基。
在js中 继承的工作方式稍不同于其他流行oo语言那样。
~~~
function Person(){}
Person.prototype.dance = function(){} ;

function Ninja(){}
// 方法拷贝
Ninja.prototype = {dance: Person.prototype.dance };

const ninja = new Ninja() ;
assert(ninja instanceof Ninja,
    "ninja receives functionality from the Ninja prototype");
assert(ninja instanceof Person, "... and the Person prototype");
assert(ninja instanceof Object , "... and the Object prototype");    
~~~ 
因为一个函数的原型是一个对象，有很多拷贝功能的方式以求达到继承。在此段代码中我们定义了一个Person 之后 Ninja。
因为 忍者 也是 人 我们想让Ninja继承Person的属性。 我们通过拷贝Person原型的dance属性到Ninja原型的相同的名字属性上来完成目的。

虽然功能是一样的 但 Ninja不是Person！ 这不是继承 --- 这只是拷贝。

我们真正想做的是 通过原型链完成 以便于一个Ninja是一个Person，一个Person可以是一个Mammal（哺乳动物），一个Mammal 可以是一个Animal（动物）
等等 ... 都是Object 。 创建这样原型链的最好技术就是使用一个对象的原型作为另一个对象的原型 。
>   SubClass.prototype = new SuperClass() ;
比如
>   Ninja.prototype = new Person() ;

~~~

function Person(){}
Person.prototype.dance = function(){};

function Ninja(){}
Ninja.prototype = new Person() ;

const ninja = new Ninja();
assert(ninja instanceof Ninja, 
    "ninja receives functionality from the Ninja prototype");
assert(ninja instanceof Person, "... and the Person prototype");
assert(ninja instanceof Object, "... and the Object prototype");
assert(typeof ninja.dance === "function", "... and can dance! ");
~~~

注意上面这种技术 我们丢失了 子类的constructor的引用 因为把子类的原型设定为父类实例后 自己的东西就丢了！

老的原型没有人引用它，就变孤立点了 会被删除的。

这里有一个重要的秘事：当执行instanceof操作时 ，我们可以决定是否函数继承其原型链上的任何对象的功能。

NOTE 可能对你有另一个技术，也是我们强烈不推荐的， 使用Person的原型对象 直接作为Ninja原型 如： Ninja.prototype = Person.prototype.
任何对Ninja原型的改变也会改变到Person原型（因为他们是同一个东西），这会导致不可期的边缘效应 。

以此种方式使用原型继承的另一个好处是所有被继承的函数原型会持续保持 live-update（热更）。继承自原型的对象总是可以访问当原型属性。

### 重写constructor属性的问题
通过设定 Person对象作为 Ninja构造器的原型，我们丢失了我们对Ninja构造器的连接 此前它是存在于原始Ninja原型中的。 这是一个问题，
因为构造器属性可以用来决定对象是创建自哪个函数。
有人会做这样的假设
>
    assert(ninja.constructor === Ninja, 
        "The ninja object was created by the Ninja constructor");

但是当前为止 这种假设不会通过的。如果我们在ninja对象上搜索constructor属性，我们会找不到的。所以我们会造访它的原型，但它也没有constructor属性
，再次地 我们继续跟踪原型往上找 Person对象的原型 ，它有一个constructor属性 引用到Person函数。 实际上 我们得到了一个错误的答案：
如果我们询问ninja对象 是谁构造了它，我们会找到 Person 作为其答案。这会是某些严重bug的根源。

是时候修正这个情况了！但是开始之前，我们不得不看看 JS如果让我们配置属性。

#### 配置对象属性
在JS中 每个对象属性 使用一个属性描述符来描述 通过它我们可以配置如下的键：
*  configurable  如果设定为true，属性的描述符可以被改变 并且属性可被删除，如果设置为false，我们就不能做这样的事情了。
*  enumerable    如果设定为true，在for-in循环中会显示对象的属性。
*  value         指定属性的值。默认是undefined。
*  writable      如果设定为true， 属性值可以通过使用赋值来改变。
*  get           定义getter函数，当访问属性时会被调用。不能和value writable同时被定义 。        
*  set           定义setter函数，当对属性进行赋值时就会调用它 同set一样 不可以 联合使用value和writable被定义 。

比如 我们创建一个属性通过简单的赋值：
>
    nanja.name = "Yoshi";

这个属性就是可配置的configurable，enumerable，和writable ，其值会被设置为Yoshi，函数 get和set 会是undefined 。

当我们想fine-tune我们的数学配置，我们可以使用内建的Object.defineProperty 方法，他接受一个对象 在对象上属性被定义，属性的名称，和一个
属性描述符对象，作为例子看看下面的例子：
~~~
    // 创建一个空对象； 使用赋值来添加两个属性
    var ninja = {} ;
    ninja.name = "Yoshi";
    ninja.weapon = "kusarigama";

    Object.defineProperty(ninja,"sneaky",{
        configurable: false,
        enumerable: false,
        value: true,
        writable: true
    });

    assert("sneaky" in ninja, "We can access the new property");

    // 使用for-in循环来遍历ninja的 enumerable 属性 。
    for(let prop in ninja){
        assert(prop !== undefined, "An enumerated property: "+ prop);
    }

~~~    
通过设定 enumerable属性为false 我们可以确信 属性在我们使用for-in循环时不会出现的。为了明白为什么我们要做这件事情，
让我们回到原始问题。

### 最终解决覆盖constructor属性的问题
当试图使用Ninja扩展Person时（或者是Ninja称为Person的子类），我们遇到这样的问题：当我们设定新Person对象为Ninja构造器的原型时，
我们丢失了原始Ninja的原型 该原型保存有我们的constructor属性。我们不想丢掉constructor属性，因为 他对于决定创建我们对象实例的函数很有用
并且可能对于其他使用我们代码基的开发人员也有用处。

我们可以解决这个问题 通过使用我们刚刚学到的知识。 我们会定义一个新的 constructor属性在 新的Ninja.prototype 上 通过使用Object.defineProperty
方法。

~~~

    function Person(){}
    Person.prototype.dance = function(){};

    function Ninja() {}
    Ninja.prototype = new Person();

    Object.defineProperty(Ninja.prototype, "constructor",{
        enumerable: false,
        value: Ninja,
        writable: true
    });

    var ninja = new Ninja() ;

    assert(ninja.constructor === Ninja,
        "Connection from ninja instances to Ninja constructor reestablished!");
    for(let prop in Ninja.prototype){
        assert(prop === "dance", "The only enumerable property is dance!");
    }        
~~~
我们重新构建了ninja实例和Ninja函数间的连接。因而 我们可以知道 它是通过Ninja函数构建的。 
此外，如果任何人想遍历Ninja.prototype 对象的属性，我们可以确信 我们修正的数学constructor 不会被访问到的。

### instanceof 操作符

在很多编程语言中，检测一个对象是否是一个类继承层次的一部分最直接的方式就是使用instanceof操作符。比如在java中
instanceof操作符用来检测一个其左侧的对象是否和右侧的类相同或者是右侧类类型的子类。
【有点像  objClass = getClass(object) ;  if(inArray(ojbClass, MyClass + [ SubClassesOfMyclass ]))】

尽管 在js中 对instanceof操作符 也存在相似对等的东西，但仍旧有点twist 绕 。在JS中 instanceof 操作符工作在对象的原型链上，
比如 
>   ninja instanceof Ninja 
instanceof 操作符检测是否Ninja函数的**当前**原型 是否是在ninja实例的原型链上。

更具体的例子
~~~

    function Person(){}
    function Ninja(){}

    Ninja.prototype = new Person();

    const ninja = new Ninja() ;

    // 一个忍者实例 同时是 忍者和人
    assert(ninja instanceof Ninja,"Our ninja is a Ninja!");
    assert(ninja instanceof Person,"A ninja is also a Person. ");
~~~

ninja实例的原型组成自一个new Person() 对象,通过它 我们获得了继承 和Person的原型。 当计算表达式：
>   ninja instanceof Ninja 
时 Js引擎使用Ninja函数的原型--》即new Person()对象 。并检测其是否在ninja实例的原型链上 。因为new Person()对象就是ninja实例的直接原型，
结果就是true 。

第二种情况 当我们检测 **ninja instanceof Person** 时，JS引擎使用Person函数的原型 ，并检测它是否可以在ninja实例的原型链上被找到。
再次， 可以找到，因为它就是我们new Person()对象的原型 我们也已经看到 是ninja 实例的原型。

这就是对instanceof操作符所知的一切 。尽管其最常用的的是提供一个清晰的方式来决定是否一个实例是由一个特定的函数构造器创建，但并不如那样工作。
换之，他检测 右边的函数原型 是否存在于 左侧对象的函数原型链之上 。因而这是我们应该小心的一个警告。

##### instanceof警告
像你已经很多次看到 Js是一个动态语言，在其中你可以在程序执行期间修改很多东西。比如 没有什么能够阻止我们修改构造器函数的原型。
~~~
function Ninja(){}
const ninja = new Ninja() ;

assert(ninja instanceof Ninja, "Our ninja is a Ninja!");

Ninja.prototype = {} ;

assert(!(ninja instanceof Ninja), "The ninja is now not a Ninja!?");

~~~
现在我们理解了js中原型如何工作，并且知道如何使用原型来使用构造器函数来实现继承，现在让我们看看ES6新增的特性：classes .

## 在ES6 中使用JS的“classes”
Js允许我们使用通过原型来实现继承的一种形式。但更多的开发者，特别是那些来自经典OO背景的，更喜欢一种简单或者js的继承系统的 到他们更熟悉的一种抽象

这不可避免的因我们到了classes ，即便js原生不支持经典的继承。为了响应这种需求，一些js 模拟经典继承 的库出现了，因为每种库都以他们自己的方式实现
基于类的继承， ECMAScript委员会已经标准化了 用于模拟基于类继承的语法。注意我们说的是**模拟**。即便我们现在可以在JS中使用class关键字，
底层的实现仍旧是基于原型继承的！

NOTE class关键字已经被添加到ES6 并不是所有的浏览器都实现了它。

### 使用class关键字
ES6提供了新的class关键字 它提供了一种更优雅的创建对象 并实现继承（比手动使用原型实现他们）的方式 。使用class关键字很简单：
~~~

    class Ninja{
        // 定义构造器
        constructor(name){
            this.name = name ;
        }
        // 定义额外对所有Ninja实例可访问的方法
        swingSword(){
            return true ;
        }
    }
    // 使用new 来实例化一个ninja对象
    var ninja = new Ninja("Yoshi");

    assert（ninja instanceof Ninja, "Our ninja is a Ninja");
    assert(ninja.name === "Yoshi", "named Yoshi");
    assert(ninja.swingSword(), "and he can swing a sword");

~~~
### classes 是语法糖
ES6提供了新的class关键字 但底下仍旧使用的老的原型技术实现；classes是语法糖 设计来使我们的生活更容易 当我们在js中模拟classes时。

我们的class 代码可以翻译为功能对等ES5实现
~~~

    function Ninja(name){
        this.name = name ;
    }
    Ninja.prototype.swingSword = function(){
        return true ;
    };

~~~
ES6 classes没什么特殊的 ，只不过代码更优雅 ，但用的是相同的概念。

### 静态方法

定于在类级别。

~~~
    class Ninja{
        constructor(name, level){
            this.name = name ;
            this.level = level ;
        }

        swingSword(){
            return true ;
        }
        // 使用static 关键字来定义一个静态方法
        static compare(ninja1, ninja2){
            return ninja1.level - ninja2.level ;
        }
    }

    var ninja1 = new Ninja("Yoshi", 4);
    var ninja2 = new Ninja("Hattori", 3);

    assert(!("compare" in ninja1 ) && !("compare" in ninja2),
        "A ninja instance doesn't know how to compare" );

    assert(Ninja.compare(ninja1, ninja2 ) > 0 ,
        "The Ninja class can do the comparisn!");

    assert(!("swingSword" in Ninja),
        "The Ninja class cannot swing a sword");                

~~~

我们也可以在ES6之前的代码实现“static”方法 ，为此 我们必须记得只有classes通过函数被实现 。因为静态方法是类级别的方法，我们可以通过使用
函数作为第一类对象的优点来实现它，并添加一个属性方法给我们的构造器方法，如下例：
~~~
    function Ninja(){}

    Ninja.compare = function(ninja1, ninja2){ ... }
~~~

### 实现继承
说实话，在es6之前的代码做继承是很痛苦的。
~~~
    function Person(){}
    Person.prototype.dance = function(){};

    function Ninja(){}
    Ninja.prototype = new Person() ;

    Object.defineProperty(Ninja.prototype, "constructor",{
        enumerable: false,
        value: Ninja,
        writable: true
    });
~~~
这里需要记住很多东西：对所有实例可访问的方法 应该直接添加到构造器函数的原型上。 如果我们想实现继承，我们不得不设置 子类的原型为基类的实例。
这里就是 **Ninja.prototype = new Person() ;** 不幸的是，这让constructor属性变得混乱了，所以我们不得不手动恢复它 通过使用Object.defineProperty方法。

我们要记这么多东西来完成相对简单 并且很常用的继承特性。幸运的是 ES6中 所有的这些都被简化了。
~~~

    class Person{
        constructor(name){
            this.name = name ;
        }

        dance(){
            return true ;
        }
    }

    class Ninja extends Person{
        constructor(name, weapon){
            // 使用super关键字 来调用基类构造器
            super(name);
            this.weapon = weapon ;
        }
    }

    var person = new Person("Bob");

    assert(person instanceof Person, "A person's a person");
    assert(person.dance(), "A person can dance");
    assert(person.name === "Bob", "We can call it by name.");
    assert(!(person instanceof Ninja), "But it's not a Ninja");
    assert(!("wieldWeapon" in person), "And it cannot wield a weapon");

    var ninja = new Ninja("Yoshi","Wakizashi");
    assert(ninja instanceof Ninja, "A ninja's a ninja");
    assert(ninja.wieldWeapon(), "That can wield a weapon");
    assert(ninja instanceof Person, "But it's also a person");
    assert(ninja.name === "Yoshi" , "That has a name");
    assert(ninja.dance(), "And enjoys dancing");    
~~~
在ES6中使用extends关键字来继承另一个类



