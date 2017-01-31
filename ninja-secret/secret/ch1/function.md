javascript 提供了很多方法来定义函数，可被分为4组：
- 函数定义和函数表达式
>
    function myFun(){ return 1 ;}

- 箭头函数（经常称之为 lambda函数）
>
    myArg => myArg*2

- 函数构造器 不常用，可以让我们从一个字符串（也可以动态生成）动态构造一个新函数        
>
    new Function('a','b', 'return a + b ')

- 生成器函数   可以生成器版的 函数定义，函数表达式和函数构造器
>    function* myGen(){ yield 1 ;}    

明白这些区别是很重要的，其定义的方式很大程度影响什么时候函数可用于调用和其行为，也会影响在哪个对象上可以被调用。


## 函数定义和函数表达式
最常用 很相似  甚至不区分他们的差别 但也有很细微的差别

### 函数声明
最基础的定义一个函数的方式 
>
    function funcName(p1 [,pN]+ ) {  <statement>*  }
开始于function 关键字，跟一个必须的函数名 和一系列位于括号中的可选的逗号分割的参数，函数体 用花括号包着
里面是一系列语句
~~~

    function myFunctionName( myFirstArg , mySecondArg ) {
        myStatement1 ;
        myStatement2 ;

    }
~~~    
函数定义 代表了一个独立的js代码块 可以包含在其他函数中 。（函数定义的嵌套）

例子
~~~
function myGlobalFun(){
    return "hi I am defiend in the global scope!" ;
}

// global code 
function ninja(){
    function hiddenNinja(){
        return " defiend within in another ninjia function ! "
    }

    return hiddenNinja() ; // 返回内层函数调用结果
}
~~~

让函数定义于其他函数内部 会产生一些问题需要考虑： scope 和 identifier resolution（标识符解析）

### 函数表达式
js中的函数是第一类对象 可以通过字面量创建 赋值给变量和属性，可以被作为其他函数的参数和返回值。
因为函数是如此基础的构造，js预先我们视其为任何其他的表达式，就像一个数字字面量那样：
>
    var a = 3 ;
    myFunction(4) ;

    // 对比上面的 
    var a = function() {} ;
    myFunction( function(){} ) ;

函数表达式允许我们在需要他们的地方随时随地创建，以使得我们的代码更容易被理解。

下列展示了函数声明和函数表达式的区别：
~~~

    function myFunctionDeclaration(){
        function innnerFunction() {

        }
    }

    var myFunc = function() {} ;
    myFunc( function() {
        return function(){} ;
    } );

    // 命名函数表达式 即时执行
    (function namedFunctionExpression(){})() ; 
    
    // 即时执行的函数表达式 作为unary操作符的参数
    + function(){}() ;
    - function(){}() ;
    ! function(){}() ;
    ~ funciton(){}() ; 

~~~    
函数表达式 总是其他语句的一部分。他们位于表达式级别作为变量声明的左值（赋值）；
> var myFunc = function(){ } ;
或者作为其他函数的参数 ，返回值。
>
  myFunc(function() {  return function(){ } ;   }) ;

除了他们出现的位置，还有一个重要的区分，对于函数声明，函数名是强制的，对于表达式则是完全可选的 。（函数定义的嵌套）

因为函数必须是可被执行的，我们必须有一种引用他们的手段 。 唯一的方式就是通过他们的名称 。
函数表达式作为其他js表达式的一部分，我们有其他方式来调用他们，比如，如果函数表达式被赋值给一个变量，我们可以使用那个变量来
调用函数 。
>
    var doNothing = function() {} ;
    doNothing() ; // 调用函数 
或者 是一个指向其他函数的参数 ，我们可以在那个函数中 通过匹配的参数名来调用他
>
    function doSomething(action) { 
        action() ;
    }    

### 即时函数
immediately invoked function expression (IIFE), 简称 immediate function
>
    (function(){})(3) ;    
这是很重要的一种形式， 它允许我们在js中模拟 module（模块）
注意括号是必须的 不然js解析器会出错的 。

可选但不常用的风格：
>
    (function(){}(3)) ;  // 有点怪

**[+-!~]function(){}() ; ** 
除了使用括号包裹函数表达式 用以区分函数声明，我们可以使用unary操作符: +-*~ 我们以此来讯知js引擎 其应该被处理为表达式而不是语句。
注意结果并没有被存在什么地方； 从计算角度 并不打紧；

## 箭头函数 
ES6 新增的特性 
语法糖

两种风格
- >
    var values = [0,3,2,5,7,4,8,1] ;
    values.sort(function(value1 , value2 ) { 
        return value1 - value2 ;
    });

- 使用箭头函数
>
     var values = [0,3,2,5,7,4,8,1] ;
     values.sort((value1, value2) => value1 - value2 ) ;

比函数表达式更简洁，没有function关键字，括号，或者return 语句。
=>  （fat-arrow  胖箭头）

语法
>
    param => expression

此箭头函数接受一个参数 并把表达式的值作为结果返回 作为函数的返回值
上面是最简形式

一般形式：
>
    (p1, p2 ) => expression | {
        myStatement1 ;
        myStatement2 ;
    }         

如果没有参数，或者多于一个参数时括号是必须的 ，如果只有一个参数 它又是可选的。后面跟了一个必选的胖箭头
这告诉js引擎 （我们在处理一个箭头函数）
在胖箭头操作符之后    ，我们有两个选择，如果是简单的函数，我们放一个表达式在哪里（数学操作，其他函数调用，..)
函数执行的结果就是哪个表达式的值。
另一种情形，我们的箭头函数并不那么简单 需要更多的代码，我们可以在箭头符号后包含一块代码：
>
    var greet = name => {
        var helloString = 'Greeting ' ;
        return helloString + name ;
    }

如果没有return 那么这个函数调用的结果就是undefined ，如果有那么结果就是return表达式的值 。

##  Arguments && function parameters 
 参数和函数参数
这两个经常交替使用，更正式些：
- parameter 是一个变量，我们将之左磊函数定义的一部分列出
- argument 是一个值 当我么调用函数时传递给函数的。

~~~
// 此时funParam 是函数parameter
function fun(funParam){
    // 此时funParam 作为arguments了
    return performAction(funParam, 'someArgs') ;
}

var performAction = function( person, action ){
    return person +　"-" + aciton ;
};

var rule = daimyo => performAction(daimyo, "ruling") ;
~~~ 

如你所见,函数的parameter是在函数定义时指定的，所有类型的函数皆可以有parameters:
- 函数声明
- 函数表达式
- 箭头函数

arguments，和函数调用相连；他们是在调用期间传递给函数的值：

依序传递（同定义时的顺序）


如果arguments和parameter的数量不匹配，没有error产生。
-  多余     
-  不足  未被指定的 会是undefined

## Rest parameters 
ES6 新增特性

~~~
    
    function multiMax(first, ...remainingNumbers){
        var sorted = remainingNumbers.sort(function(a, b){
            return b - a ;
        });
        return first * sorted[0] ;
    }


~~~
通过在最后一个参数前面添加 ... 我们将之转变为一个数组称之为 Rest parameters ,他包含剩余传递的arguments。
>
    function multiMax(first, ...remainingNumbers) { ... }    

只有最后一个参数才可以成为 rest parameters

## 默认参数
ES6 新增功能

很多webUI 组件是可配置的，
组件本身提供了默认值，但同时也允许用户使用自己的值类覆盖这些默认设置。

默认参数就很适合这种场景 。

js不支持函数重载！(其他语言中用不同版本的同名函数来解决这种问题 js中通过检测参数的情况来做)
~~~

    function performAction(ninja, action){
        action  = typeof action === "undefined" ? "skulking" : action ;
        return ninja + " " + action ;
    }
~~~

NOTE typeof 操作符 返回一个字符串 显示操作数的类型，如果操作数未被定义 返回值就是字符串 **undefined**

这是一个经常出现的模式 很繁琐 
ES6使用默认参数来支持这种场景 。
~~~

    funciton performAction(ninja, action = "skulking"){
        return ninja + " " + action ;
    }
~~~

为了穿件默认值，我们只需要跟函数的参数赋予一个值即可 。

我们可以赋予任何值给默认参数： simple ，primitive values，string ，复杂的如object， arrays ，甚至functions。
值会在每次函数调用时被计算，从左到右，当给后面的默认参数赋值时，我们可以引用前面的参数：

~~~

    function performAction(ninja, action = "skulking" , message = ninja + " " + action){
        return message ;
    }

~~~

我们的观点是 尽量避免这种用法 它并没有增加代码的可读性 。

但是moderate默认参数的使用 作为避免null值的手段 或者是相对简单的配置我们函数行为的flag标记 -- 可以产出更简单和更优雅代码。

