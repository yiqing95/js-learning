涵盖
- 两个隐式的函数参数：arguments 和 this
- 调用函数的方式
- 处理函数上下文的问题

隐含的参数this和 arguments 暗暗地传递给函数 。可被其他显示命名函数参数那样在函数体中访问 。
this参数表示函数上下文，即我们的函数在哪个对象上被调用 ，而arguments参数表示所有的通过函数调用传递的参数。

## 使用隐式函数参数
函数调用隐式地传递了两个参数 arguments和this

此二参并没有显式地出现在函数签名中，但隐式地传递给了函数并在函数中可以访问他们 ，他们可以像其他显式命名参数那样被访问。 

## arguments 参数

arguments 参数是一个收集所有传递给一个函数的参数集合。
不论参数是否准确匹配（）。

藉此可以实现函数重载（js 本身不支持哦！），实际上 使用rest参数后 对于arguments的需要大大地消减了。但理解arguments参数
如何工作的仍旧是很重要的。 

arguments对象有个length属性，其示出了参数的确切数目，单个参数值可以使用数组索引访问：arguments[2]

看起来像数组 但实际不是哦

~~~

    function(){
        var sum = 0 ;
        for(var i = 0; i <arguments.length; i++){
            sum += arguments[i] ;
        }
        return sum ;
    }
~~~

NOTE: 很多时候rest参数可以替代arguments参数， rest参数可是真正的数组，这意味我们可以使用所有我们喜爱的数组方法在其上。

~~~

    function infiltrate(person){
        arguments[0] = 'changed' ;
        return  person == 'changed' ;
    }
~~~
arguments 有一个很重要的特性，它alises函数参数，如果我们改变它 对应的参数值也变掉了！
别名双向效应，同样地 我们改变参数 也会影响到arguments中对应的值 。

## 避免别名
js提供了一种可选方式来避免别名，通过使用： strict mode.

>
    strict mode
    是ES5的添加项 可以改变JS引擎的行为，以便于错误可以被抛出而不是默默地扔掉。一些语言特性的行为也被变掉了，一些不安全
    的行为完全被禁止。strict模式众多影响中的一个便是禁用了arguments别名 。

~~~
"use strict" ;  // 开启严格模式

function infiltrate(person){
    arguments[0] = 'ninja';
}
~~~    

通过在代码的第一行放上字符串 use strict , 这告诉js引擎 我们想执行其下的代码 以严格的模式。
在开启严格模式后  arguments 将不再是 函数参数的别名 两者分离了 独自修改互不影响 。

## this 参数： 引入函数上下文 

this参数 隐式被传入函数，在js的oo编程中至关重要 引用到一个关联到函数调用的对象，因此 也被称为 function context。
函数上下文。

在js中 以一个方法来调用函数只是函数可被调用的一种方式，this参数到底指向的东西不光由定义方式决定而且和定义地方相关。

## 调用函数 
我们可以用四种方式调用一个函数
- 作为一个函数  最直接的方式
- 作为一个方法 绑定调用到一个对象上，允许oo编程
- 作为一个构造方法  
- 通过函数的apply 或者call 方法

例子
~~~

function skulk(name) {}
function Ninja(name) {}

// 作为函数调用
skulk('Hattori');
(function(who){ return who ; })('Hattori') 

var ninja = {
    skulk: function(){}
};
// 作为方法调用
ninja.skulk('Hattori') ;

ninja = new Ninja('Hattori');   // 作为构造方法

skulk.call(ninja, 'Hattori') ;  // 通过call方法调用
skulk.apply(ninja, ['Hattori']) ; // 通过apply方法  
~~~

### 作为一个函数调用

函数作为函数调用， 当然啦 ，这近似废话 ，但这个是区分其他调用形式的一种说法。

这种调用发送在一个函数使用() 括号操作符时，并且不是作为一个对象的属性。

~~~
function ninja() {}
ninja() ;

var samurai = function(){}
samurai() ;
(function(){})(); 
~~~
当以此种风格调用时，函数的上下文（this关键字的值）可以是两个东东： 在非严格模式下，它是全局上下文（window对象），
而在严格模式下，它会是 undefined!

~~~
// 非严格模式
function ninja(){
    return this ;
}

// 严格模式
function samurai(){
    "use strict";
    return this ;
}
~~~

### 作为方法调用

当一个函数被赋值给一个对象的属性，并通过其属性引用它来调用时 该函数是作为那个对象的方法来调用的。

~~~

    var ninja = {} ;
    ninja.skulk = function(){} ;
    ninja.skulk() ;
~~~

那又怎样 ，什么使得这样做变得有趣和有用了？ 如果你来自oo背景 ，this 代表的意义你应该懂得，相同的事情也发生在这里。
当以一个对象的方法调用一个函数时 。此对象变为函数的上下文并变得在函数中可以通过this参数来访问它 。这是一种很重要的方式
js就是通过它允许写出oo代码（构造方法是另一种） 。

~~~
function whatsMyContext(){
    return this ;
}
var getMyThis = whatsMyContext ; // 获得一个引用

var ninja1 = {
    getMythis: whatsMyContext
};
var ninja2 = {
    getMythis: whatsMyContext
}
~~~

NOTE 作为方法调用函数对于以oo方式在js中编写代码是至关重要的。通过这样做使得你可以在任何方法中使用this来引用方法
的“所有者”对象 --- 在oo编程中的一种基础概念。

相同的函数被ninja1和ninja2 共享了！

我们不必创建一个单独的函数拷贝 来在不同的对象上执行相同的处理。

### 以构造方法调用
构造方法声明时同其他函数一样，我们可以很容易地使用函数声明和函数表达式来构造一个新的对象 。 唯一的不同是 箭头函数
无论如何 主要的区别是函数如何被调用 。

为了以一个构造方法来调用函数 ，我们会冠以new关键字 ：

NOTE 函数还可以通过 函数构造器创建  new Function('a','b' , 'return a + b ') ; 函数构造其和构造器函数 不要搞混淆了哦！

### 构造器的超能力
以构造器的方式调用一个函数是js 的一个强有力的特征 。
~~~

function Ninja(){

    this.skulk = function() {
        return this ;
    };

}

var ninja1 = new Ninja() ;
var ninja2 = new Ninja() ;


~~~

当通过new关键字调用时， 一个空对象实例被创建了 并作为上下文被传递给了函数。

通常当一个构造器被调用 ， 一系列动作会发生 ， 

1. 一个空对象实例被创建了 
2. 此对象被作为this参数 传递给构造器，并因此变为构造器的函数上下文 
3. 新构造的对象被作为new操作符的值返回 （有例外情形 ）

构造器的意图就是导致一个新对象被创建 ，并作为构造器的值被返回 ，任何违背这个目的的东西都不适合使用构造器了 。


### 构造器返回值
上面介绍了 构造器意在初始化一个新创建的对象，此新构造的对象被作为构造器调用的结果返回（通过 new操作符）。
但 当构造器返回它自己的值时呢 ？

~~~

function Ninja(){
    this.skulk = function(){
        return true ;
    };

    return 1 ; // 构造器返回了一个特别的原生值 数字1 
}

Ninja()  === 1 ; // 非构造函数调用

var ninja = new Ninja() ;  // 以构造函数调用 

typeof ninja === 'object' ;
~~~

实际上 构造方法返回一个简单的数1 对代码的行为并没有多大影响。
如果以函数来调用 它就返回1  如果以构造方法来调用 会有一个新的对象被创建并被返回 。

目前为止看起来还都正常
~~~

    var puppet = {
        rules: false 
    }

    function Emperor(){
        this.rules = true ;
        return puppet ;
    }

    var emperor = new Emperor() ;
~~~
上例是容易引起混乱的例子 ；
总结：
-  如果构造器返回一个对象，该对象会作为整个new表达式的值 ，并且新创建的对象作为this传递到构造器
-  然而，如果一个非对象从构造器返回，返回值会被忽略，新创建的的对象会被返回。

因为这些特性，作为构造器的函数常常不同于其他函数 。

### 对于构造器的编码考虑
构造器的意图是用来初始化一个新对象 该对象通过函数调用创建。 尽管这样的放可以被作为**常规**函数调用， 或者甚至赋值给对象的属性，以用来作为方法调用，
但他们通常不是这么用的 。

~~~
function Ninja(){
    this.skulk = function(){
        return this ;
    };
}
var whatever = Ninja() ;
~~~
我们把Ninja作为一个简单的函数调用，但skulk属性在非严格模式下会在window全局对象上创建一个属性！
在严格模式下 事情变得更怪，此情况下this会是undefined js程序会崩溃掉的，但这是好事；

因为构造器的这种不同，经常使用命名约定予以区分。
函数和方法经常是以动词开始用以描述它们做什么，并以小写字母开头。构造器，经常用名词来描述被创建的对象，并以首字大写开始。

## 通过apply和call方法调用

函数上下文this所指
对于方法，就是其所属的对象；对于顶级函数，他是window或者undefined（根据当前严格模式）；对于构造器，是新创建的对象实例。

但如果我们想让他是任何我们想指定的对象，如果我们可以显式的设置它？..

看一个容易导致bug的例子，
考虑 当一个事件处理器被调用，函数上下文被设定为事件被绑定的对象

~~~
<button id="test" >Click me </button>
<script>

function Button(){
    this.clicked = false ;
    this.click = function(){
        this.clicked = true ;
    }
}
var button = new Button() ;
var elem = document.getElementById("test");
elem.addEventListener("click",button.click) ;

</script>
~~~
该例中 我们创建了一个button对象来跟踪html上按钮的点击状态，使用构造器函数来创建一个backing对象，我们在其中存储点击状态。

但实际并非如此。this对象被切换了！
在浏览器的事件处理系统定义调用上下文为事件的目标对象，这会导致上下文变为<button> 按钮而不是button这个js对象 。 所以我们把click的状态
设置在一个错误的对象上了！

接下来看看如何显式的使用apply和call方法来设置函数上下文 。

### 使用apply和call方法

js为我们提供了一种调用函数 并显式指定任何我们希望的对象作为函数上下文的手段 。
通过任何函数都存在的两个方法： apply和call 。

对 就是函数的方法，作为第一类对象，函数也可有有属性，同其他任何对象类型一样 ，也包括方法 。
通过使用其apply方法来调用函数，我们传递两个参数给apply：
一个作为函数上下文的对象，和一个值的数组 用来作为调用参数。 call方法也是类似用法，除了参数以参数列表的形式被直接传递而不是作为数组传递。

~~~
    function juggle(){
        var result = 0 ;
        for( var n = 0 ; n < arguments.length; n++ ){
            result += arguments[n];
        }
        this.result = result ;
    }

    var ninja1 = {};
    var ninja2 = {};

    juggle.apply(ninja1, [1,2,3,4]);
    juggle.call(ninja2, 5,6,7,8);

    ninja1.result === 10 ;
    ninja2.result === 26 ;
~~~

本例中 函数把所有参数的值累加起来存储到当前上下文对象的result属性上。

注意到apply和call 只是区分在参数的提供方式上 。
对于apply情况 我们使用参数的数组，对于call情形，我们分列出调用参数 。

此二方法特别是在回调情况下特别有用 。

### 在回调中强制函数上下文
在 imperative编程中，经常传递一个数组给方法 并使用一个for循环来迭代每一个元素，在每个元素上执行操作：
~~~
    function(collection){
        for(var n=0 ; n < collection.length; n ++ ){
            /*  do something to collection[n] */
        }
    }
~~~

相反地 在函数式编程中 常常创建一个函数，该函数操作在单独的元素上 并传递每个元素给这个函数：
~~~

    function(item){
        /* do something to item */
    }


~~~
区别在于 函数作为程序构建块的级别是哪儿 。 后者重用度更高些

当代js引擎现在都支持对象上的forEach方法 ，我们也可以创建我们自己的：
~~~
function forEach(list, callback){
    for(var n=0; n < list.length ; n++){
        callback.call(list[n], n) ;
    }
}

var weapons = [
    {type: 'shuriken'},
    {type: 'shuriken1'},
    {type: 'shuriken2'}
];

forEach(weapons, function(index){
    this === weapons[index] ;
}) ;
~~~

apply  和 call 做的是相同的事情，那么我们怎么觉得该用哪个?
高层次的回答是：使用那个可使代码可读性好的 。更实战实际些的回答是 使用更加匹配参数的哪个。

## 修正函数上下文的问题
如上
在一些情况下 上下文不如我们所预期的。我们可以使用apply和call来解决。
在本小节 ，你可以看到另外两个选择： 箭头函数和bind 方法 ，在某种情况，会收到相同的效果，但是一种更优雅的方式 。

### 使用箭头函数来解决函数上下文
箭头函数有一个特征 使其作为回调函数相当适合： 
箭头函数没有它们自己的this值。 而是，他们在其定义的时刻记住了this参数的值。
~~~

    <button id="test"> click ME ！</button>
    <script>
    function Button(){
        this.clicked = false ;
        this.click =  () => {
            this.clicked = true ;

        };
    }
    var button  = new Button() ;
    var elem = document.getElementById("test");
    elem.addEventListener("click", button.click) ;
    </script>
~~~

我们的click事件处理器是作为箭头函数在Button构造器内部创建的

箭头函数不具有其自己的上下文，换之，上下文是继承自他们被定义的函数那里，this参数在我们的箭头函数回调中指的是button对象 。
click箭头函数是在构造器函数中定义的，哪儿this参数就是心创建的对象。

###  警告：箭头函数和对象字面量

因为this参数的值是在箭头函数创建的地方选取的，会出现一些奇怪的行为
~~~

    <button id="test"> Click Me!</button>
    <script>
        assert(this === window, "this === window" ) ; // 断言this的值是window对象
        var button = {
            clicked: false,
            click: () => {
                this.clicked = true ;
                assert(button.clicked, "The button has been clicked");
                assert(this === window, "In arrow function this === window ");
                assert(window.clicked, "clicked is stored in window");
            }
        }
         var elem = document.getElementById("test");
         elem.addEventListener("click",button.click) ;
    </script>·
~~~

如果箭头函数被定义在对象字面量中 ，而且该对象被定义在全局代码中，this参数的值关联到箭头函数的就是全局的window对象 。

规则
>  箭头函数是在他们创建时刻选取this参数的值

因为 click箭头函数是作为对象的属性值被创建的 ，对象字面量的创建是全局代码，箭头函数的this值就是 全局代码的this值 --- 》 window

这个容易导致bug 务必小心！ 

### 使用bind 方法

除了 apply call方法 ，每个函数可以访问它的bind方法 ，简言之，该方法创建一个新函数。此函数有相同的函数体，但其上下文总是绑定到特定对象，跟其调用方式无关 。

~~~

    <button id="test"> Click Me!</button>
    <script>
        assert(this === window, "this === window" ) ; // 断言this的值是window对象
        var button = {
            clicked: false,
            click: function(){
                this.clicked = true ;
                assert(button.clicked, "The button has been clicked");
             
            }
        }
         var elem = document.getElementById("test");
         elem.addEventListener("click",button.click.bind(button)) ;

         var boundFunction = button.click.bind(button) ;
         assert(boundFunction != button.click , "Calling bind creates a completly new function ") ;
    </script>·
~~~

bind方法对所有函数都可用，被设计为创建并返回一个新的函数 它绑定到传入的对象（上例中的 button对象），this参数的值总是被设定为那个对象，
而不管绑定函数以何种方式被调用 。除此之外，绑定函数的行为和原始函数一样，因为他们在函数体中有相同的代码 。

