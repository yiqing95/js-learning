正则表达式
-------

现代开发中 正则表达式是必须的。
尽管很多web开发者在没有正则表达式时也能愉快地过一生。有些问题 在js中如果不使用正则表达式 就不能很优雅地解决掉。

确实，对相同的问题 有其他的方式解决。但常常，如果正则式使用得当 可以用精炼为一行代码解决需要半屏代码才能解决的问题。
所有JS 忍者们 都需要正则表达式作为其工具集中的一部分。

你会在主流JS库中看到 使用正则表达式解决一些任务：
- 操作 HTML节点的字符串
- 在css选择器表达式中 定位某选择器片段
- 判断一个元素有特定的class名称
- 输入验证
- 更多...

## 为什么正则表达式牛（rock）

假设 我们想验证一个字符串，或许是一个网站用户 输入到表单的 满足 美国邮编 九个8。
>   99999-9999
每个9表示一个十进制数字，格式是5个数 跟一个横-  再跟四个十进制的数字。

让我们创建一个函数，给其一个字符串，验证US 邮编服务。
但我们是忍者（ninja）呀，这是一个很不优雅的方案，导致了很多无用的重复。

~~~js

    function isThisAZipCode(candidate){
        if(typeof candidate !== "string" ||
            candidate.length != 10) return false ;
        for(let n=0; n < candidate.length; n++){
            let c = candidate[n];

            switch(n){
                case 0: case 1: case 2: case 3: case 4:
                case 6: case 7: case 8: case 9:
                    if(c < '0' || c > '9') return false ;
                    break;
                case 5:
                    if(c != '-') return false ;
                    break;
            }
        }
        return true ;
    }
~~~
这段代码有个优点 我们只有两个检测要做，依赖于字符串中字符的位置。  我们仍需要在运行时执行九次对比，当我们不得不写出每一次的对比（只一次）。

有人考虑过这个方案是 优雅的么？ 
现在考虑这个方法：
>
    function isThisAZipCode(candidate){
        return /^\d{5}-\d{4}$/.test(candidate);
    }

## 初入正则表达式

这里只能提点一些重要的点。不能给你整个正则的手册。

有很多专业的 正则表达式 的书 你可以参考
- << Mastering Regular Expressions >>
- << Introducing Regular Expressions >>
- << Regular Expressions Cookbook >>

### 正则表达式解释

此词来自中世纪的数学领域。有个数学家 名叫：Stephen Kleene 描述了 “正则 集” 的计算模型。但这并不能帮我们理解 正则表达式的任何东西
，所以让我们简化事情 可以说 正则表达式 是表达 用于匹配文本的字符串的**模式** 的一种方式。
表达式本身 有terms 和 operators 组成 他们允许我们定义这些模式。
我们会看到这些 简短的术语和操作符的组成。

在JS中，和其他重要的对象类型那样， 我们有两种方式来创建一个正则表达式
- 通过正则表达字面量
- 通过构造一个RegExp对象的实例

比如，如果我们想创建一个 平常的正则表达式（简言之 regex），其精确匹配字符串 **test** ；我们可以使用正则表达式字面量：
> const pattern = /test/ ;

这看起来有点怪，但正则表达式字面量 使用正斜杠定界 和字符串字面量使用引号符号定界类似。

可替换地，我们也可以构造一个RegExp实例，传递一个正则式作为字符串： 
> const pattern = new RegExp("test");

两种风格 导致了变量pattern 赋予 相同的正则表达式被创建 。

    TIP 提示  字面量语法更可选 当你在开发期 使用正则表达式 ，构造器方法被用于 正则式被构造于运行期 通过动天地在字符串中构造它。

字面量语法更可选的一个原因是 反斜线字符串正则表达式中占很重要部分。 但反斜线字符也是用于字符串字面量的转义字符，我们需要使用
双 反斜线（\\）.
五个符号跟regex有关：
- i 使得正则式大小写无关 case-insensitive ，所以 /test/i 不光匹配test也匹配Test TEST,tEsT,等组合 。
- g 匹配模式的所有实例，与默认的local相反，它只匹配第一次出现。g也会匹配后面的
- m 运行穿越多行的匹配，比如 来自文本textarea元素的输入 。
- y 允许粘连匹配。一个表达式在目标串中执行粘连匹配 通过尝试从最后一次匹配的位置开始匹配。
- u 允许使用Unicode转义(\u{...}) .
这些flag被添加在字面量的尾部（比如 /test/ig）或者传递到一个字符串作为RegExp构造器的第二个参数（
new RegExp("test","ig")）.

### 术语和操作符

正则表达式 和我们熟悉的其他大多数表达式一样，由terms和限制他们的操作符组成。下面你会看到这些表达模式的terms和操作符

----

- EXACT MATCHING
任何非特殊字符或者操作符必须字面式出现在表达式中。比如 在/test/ 正则式中 四个terms表示的字符 必须原封的出现在字符串中 用其去匹配表示的模式。

在一个这样的字符后面跟另一个字符隐式地表示一个操作 意即 “followed by”（跟从）。所以 /test/ 意即 t跟了e 跟了 s 跟了 t.

- 从一个字符类匹配
更多时候，我们不想匹配一个精确的字面字符。而是来自一个有限字符集中的一个 。我们可以指定此为一个集合符号（也称之为 字符类 操作符）
通过 放这个字符集在我们想匹配的地方在方括号中：[abc].

前面这个例子表示我们想匹配a，b或者c中的任何一个字符。注意到 即便此表达式跨了五个字符（三个字符和两个括号），它只在候选字符串中
匹配一个字符。

其他时间，我们想匹配任何东西 除了一个有限的字符集。我们可以通过用一个^ 上箭头字符 在打开的集合符号的中括号右边：
>   [^abc]
这会改变其意义为 任何除了 a,b,或者c之外的字符。

这儿有一个更有价值的 集合操作的变体：指定一个值范围的能力。比如 如果我们想匹配a和m之间的小写字符中的任何一个，我们可以如此写
**[abcdefghijklm]**. 但是可以更简洁地表示为：[a-m].
dash - 指示所有的字符从 a 到 m inclusive（） 被包括在集和。

ESCAPING 转义
并不是所有的字符和其表示字面义相同。 确实 所有的阿尔法字符和数字字符表示的就是他们自己 ，但 你会看到 特殊字符比如
$ 和 . 表示非他们的一些字符 ，或者限定前面项的操作符。实际上，你已经看到[, ],- 和^ 字符如何用于表示一些非这些字符字面量自己的东西。

我们如何指定 那些我们想匹配的一个字面量是 [ * 或者 ^ 或者的其他特殊字符 。 所以 \[ 指定了一个字面量来匹配 [ 字符 而不是字符类表达式
的开始。双 反斜线 \\ 匹配一个单独的 反斜杠。

    开始 和 结尾 

我们经常像确信 一个模式匹配一个字符串的开始 或者字符串的尾部。脱字符^ 当用在正则式的第一个字符时，在字符串的开始 匹配 下锚 ， 比如 /^test/
只匹配 是否 test字串 出现在字符串的开始被匹配。（注意 这是 脱字符的重载，因为 它也用来忽略一个字符类集合）。
相似地，$ 符指定 模式必须出现在字符串的尾部：
/test$/ 
同时使用 ^ 和 $ 表示 指定的模式必须包裹整个候选字符串：
/^test$/

    重复出现

如果我们想匹配四个a字符的序列，我们可能表达为/aaaa/, 但如果我们想匹配**任意**数目的相同字符？正则表达式允许我们指定几个重复选项：
- 为了指定一个字符是可选的（可以出现一次或者不出现），跟一个 问号?. 比如 /t?est/ 匹配 test或者 est.
- 指定一个字符应该出现一次或者多次 使用 + ， 比如在 /t+est/ 其匹配 test， ttest, tttest 但不是 est 。
- 指定一个字符出现 0次  1次 或者很多次 ，使用 *, 如在 /t*est/ 中匹配 test ttest tttest 和 est。
- 指定一个固定数目的重复出现 ，指定允许出现的数字在大括号间。比如 /a{4}/ 表示 一个四个相续的a 字符的匹配。
- 为了指定匹配数目范围，用逗号指定一个范围。比如 /a{4,10}/ 匹配任何4到10个相续的a字符。
- 指定一个open-ended范围，遗漏掉范围的第二个值（但留下逗号）. 正则式/a{4,}/ 匹配任何4个或者更多相续a字符串。

任何这些重复操作符 可以是 **贪婪** 或者 **非贪婪**的。默认的 他们是贪婪的：
他们会消费所有可能的匹配字符。用 ? 字符 注解操作符（一个 ? 符号的重载）， 比如 在 a+? 使得操作符非贪婪： 他只消费 足够的匹配字符。

比如 如果我们匹配字符串 aaa， 正则表达式 /a+/ 会匹配所有的三个字符。而 非贪婪表达式 /a+?/ 只会匹配一个 a 字符 ， 因为 一个单独的 a 字符就是所有需要满足的 a+ 。

    预定义字符类

一些我们想匹配的字符是不可能通过字面量指定的（比如 控制符 换行符 ）。此外，经常地 我们想匹配字符类，比如二进制字符集，或者 空白符集。
正则表达式语法提供了预定义的terms表达这些字符 或者 经常使用的类 以便于我们可以在我们的正则式中 使用 控制符 并 不必求助于字符类操作符（对常用的字符集）。

|  预定义 term |    匹配 |
| ---         | ------- |    
| \t      |  水平制表符  |
| \b      |  回退键  |
| \v      |  垂直制表符  |
| \f      |  Form feed  |
| \r      |  Carriage return  |
| \n      |  新行  |
| \cA :  \cZ     |  控制符  |
| \u0000 : \uFFFF      |  Unicode 十六进制  |
| \x00 : \xFF     |  ASCII 十六进制  |
| .              |  任何字符，除了 空白字符(\s)  |
| \d      |  任何十进制； 等价 [0-9]  |
| \D      |  任何非十进制数字的字符； 等价 [^0-9]   |
| \w      |  任何 阿尔法字符 包括下划线 ； 等价 [A-Za-z0-9_]    |
| \W      |  任何字符除了阿尔法和下划线； 等价  [^A-Za-z0-9_]           |
| \s      |  任何空白字符（space ， tab ， form feed ， 等）  |
| \S      |  任何非空白的字符  |
| \b      |  一个字边界        |
| \B      |  非一个字边界（在字内）   |

    归组
目前为止，你只看到操作符(比如 + 和 * )只放在term之前。如果我们想把操作符用在一个terms的组上 ， 我们可以用小括号来归组 ，只是一个数学表达式。
比如 /(ab)+/ 匹配 一个或者多个 连续出现的ab字串。

当正则的一部分用括号被归组后，它将有两个职责 ，也创建了 被知之为 **capture（捕获）**

    替换（OR）
替换可用 管道（|）符号来表达。 比如 /a|b/ 匹配a 或者b 字符 ，   /(ab)+|(cd)+/ 匹配一个或者多个 ab或者 cd的出现。


    反向引用（backreference）
在正则表达式中最复杂的 terms 就是反向引用 定义在正则式中的 **捕获**。
暂且理解其为被成功匹配候选字符串中的一部分。 这样的term符号 是反斜杠后跟了一个被引用的捕捉的数字，开始于1，比如 \1,\2 等。

一个例子就是 /^([dtn])a\1/, 匹配一个字符串 其开始于 d t n字符集中的任何一个，跟了一个a，跟了任何匹配第一个捕捉的字符。 后面的那一点很重要！
这跟 /[dtn] a[dtn]/. 不同 跟在a之后的字符不能是任何的d或者t或者n，但必须是那个对第一个字符触发匹配的。 因而， \1到底是哪个匹配直到计算时刻才能知道。


什么地方用它的一个好例子就是在匹配 XML-type 标记元素时 ，考虑下面的正则式:
> /<(\w)>(.+)<\/\1>/

这让我们可以匹配简单的元素比如 
> \<strong\>whatever\</strong\>   
如果没有指定反向引用的能力，这基本不可能实现，因为我们没有办法提前知道匹配开始标记的闭合标记是什么。

既然你已经对正则表达式有一定的掌握，你可以开始看看如何在你的代码中明智地使用他们。

## 编译正则表达式
正则表达式 会穿过多个阶段的处理 ，理解每个阶段期间到底发生了什么可以帮助我们优化使用正则式的JS代码。

两个主要阶段是 编译 和 执行 。

 编译发生在正则表达式首次被创建。执行发生在当我们使用编译的正则表达式来在字符串中匹配模式的时候。

在编译的时候，表达式被JS引擎解析并转换为其内部表示（不管那是什么），这阶段的解析和转换必须发生在每次一个正则表达式被创建（不考虑浏览器执行的任何内部优化）。

浏览器  经常足够的聪明在决策使用相等的正则表达式时,并缓存那个特定表达式的编译结果。但我们不能指望这个会发生在所有的浏览器上。对复杂的表达式，特殊地，
我们可以开始获取一些特别显著的速度提升通过预定义（就是 预编译）我们的正则表达式 以备后用。

如前所述 我们已知在JS中被编译的正则式有两种方式创建：通过字面量 和 通过一个构造器。

~~~js

    const re1 = /test/i;
    const re2 = new RegExp("test","i");

    assert(re1.toString() === "/test/i",
        "Verify the contents of the expression.");
    assert(re1.test("TesT"),"Yes, it's case-insensitive");
    assert(re2.test("TesT"),"This one is too.");
    assert(re2.toString() === re2.toString(),
        "The regular expressions are equal.");
    assert(re1 !== re2 , "But they are different objects.");

~~~ 
在此例中 两个正则表达式在创建之后有其自身的编译状态。如果我们用字面量/test/i 替换掉每个对re1的引用，可能发生相同的正则式会被一遍一遍编译，所以
只编译一个正则式一次，并存储其到一个变量中 以便后期引用 会是一个很重要的优化。

注意 每个正则表达式有一个唯一对象表示：每次一个正则表达式被创建（就是被编译） ，一个新的常规表达式对象被创建 。
这不同于其他原生类型（比如 string ，number 等）,因为结果总是会唯一的。

使用构造器来创建正则表达式有特殊的重要性。此技术允许我们从一个字符串来构造和编译一个表达式，字符串可能动态的在运行期创建。
这会是巨有用的 对于构造经常重用的复杂表达式。

比如，假设我们判断在一个文档中的元素有一个特定类名，其值只有在运行期才会被知道。因为元素可以有多个类名跟其关联（不太方便的存储在空格界定的字符串中）
，这就是一个运行期的有趣例子。

~~~html

    <div class="samurai ninja"></div>
    <div class="ninja samurai"></div>

    <span class="samurai ninja ronin"></span>
    <script>
    function findClassInElements(className , type){
        const elems = 
                document.getElementsByTagName(type || "*");
        const regex =
            new RegExp("(^|\\s)" + className + "(\\s|$)" );
        const results = [];
        for( let i=0, length = elems.length; i < length; i++ ){
            if( regex.test(elems[i].className)){
                results.push(elems[i]);
            }
            return results ;
        }
    }

    assert(findClassInElements("ninja", "div").length === 2,
        "The right amount of div ninjas was found. ");

    assert(findClassInElements("ninja","span").length === 1,
        "The right amount of span ninjas was found.");
    assert(findClassInElements("ninja").length === 3,
        "The right amount of ninjas was found.");
    </script>

~~~
注意对 new RegExp()构造器的使用 基于传递到函数的类名来编译一个正则表达式。这就是一个我们不能使用正则式字面量的地方，因为我们搜索的类名提前不可知。

我们只构造了此表达式一次（编译一次）为了避免重复的无必要的重复编译。因为表达式的内容是动态的（基于传入的className参数），我们知道用这种方式处理表达式
会节省很多性能。

正则式本身匹配字符串的开始或者一个空白字符，跟一个目标类名，再跟一个空白字符或者字符串的尾部。
注意在正则式中对双转义（\\）的使用：\\s 当创建字面式正则表达式时 terms 包含反斜线时，我们必须只提供反斜线一次。但是 因为我们在字符串中写这些反斜线
，我们必须 两次转义（double-escape）他们。 确实 这是比较讨厌的，但我们必须知道的是 我们用字符串构造的正则表达式而不是字面量。

在正则式被编译后 使用它来收集匹配的元素：  regex.test(elems[i].className);

预构造和预编译正则表达式 以便其可以被一次又一次的重用（执行）是一个推荐的技术 ，其提供了不可忽视的性能。

在正则式中使用括号不光可以用来归组terms，也创建了**捕获（capture）**。

## 捕获匹配的片段
判断一个字符串是否匹配一个模式是一个显见的第一步 并且常常是我们所有需要的。 
但在很多时候 判断匹配了**什么**也是很有用的。

### 执行简单的捕获
比如 我们想抽取一个内嵌于一个复杂字符串中的一个值。css转换属性的值就是这样字符串的一个好例子，通过它我们可以修改一个HTML元素的可见位置。

~~~html

    <div id="square" style="transform:translateY(15px);"></div>

    <script>
        function getTranslateY(elem){
            const transformValue = elem.style.transform;

            if(transformValue){
                const match = transformValue.match(/translateY\(([^\)]+)\)/);
                return match ? match(1) : ""; 
            }
            return "" ;
        }

        const square = document.getElementsById("square");

        assert(getTranslateY(square) === "15px",
            "We've extracted the translateY value");
    </script> 
~~~

此例使用了一个本地正则表达式和一个match方法。当我们使用全局表达式时 事情会改变的。

### 使用全局表达式匹配
如前所见，使用局部正则表达式（没有 global 标识的）用String对象的match方法 返回一个数组 其包含整个匹配的字符串，和一些在操作中任何其他匹配的捕获。

但当我们提供一个全局正则表达式（有 g flag 包含其中）match 方法会返回不一样的东西。他仍旧是一个数组，但在全局表达式时， 它会匹配所有候选字符串中可能
的匹配 而不仅仅是第一个次匹配，返回的数组包含全局匹配：此时 在每次匹配中 捕获 不会被返回。

~~~js

    const html = "<div class='test'><b>Hello</b><i>world!</i></div>";
    const results = html.match(/<(\/?)(\w+)([^>]*?)>/);

    assert(results[0] === "<div class='test'>", "The entire match.");
    assert(results[1] === "", "The (missing) slash.");
    assert(results[2] === "div", "The tag name.");
    assert(results[3] === " class='test'", "The attributes.");

    const all = html.match(/<(\/?)(\w+)([^>]*?)>/g);
    assert(all[0] === "<div class='test'>", "Opening div tag.");
    assert(all[1] === "<b>", "Opening b tag.");
    assert(all[2] === "</b>", "Closing b tag.");
    assert(all[3] === "<i>", "Opening i tag.");
    assert(all[4] === "</i>", "Closing i tag.");
    assert(all[5] === "</div>", "Closing div tag.");

~~~

我们可以看到 当使用local match时 **html.match(/<(\/?)(\w+)([^>]*?)>/)**单独的实例被匹配并且 其中的匹配捕获也被返回。
但使用global match时 **html.match(/<(\/?)(\w+)([^>]*?)>/g),** 返回的是一个匹配的列表。

如果捕获对我们很重要，我们可以依然坚持使用这个功能 当我们执行一个全局搜索 通过使用正则表达式的 exec方法。
此方法可通过一个正则表达式被重复调用，因为它在每次调用时返回下个匹配的信息集合。下面就是典型用法

~~~js

    const html = "<div class='test'><b>Hello</b><i>world!</i></div>";
    const tat = /<(\/?)(\w+)([^>]*?)>/g;
    let match, num = 0 ;
    // 重复调用exec
    while((match = tag.exec(html)) !== null){
        assert(match.length === 4, 
            "Every match finds each tag and 3 captures.");
        num++ ;
    }
    assert(num === 6, "3 opening and 3 closing tas find.");


~~~

本例中我们重复调用exec方法

这样保持了来自前一次调用的状态，以便后续的每次调用推进到下个全局匹配。每次调用返回下个匹配和其捕获。

通过使用match 或者 exec 我们总可以找到精确的匹配（和捕获）。但我们还想挖的更深些 如果我们想在正则式内引用捕获本身。

### 引用捕获

我们可以引用我们已经捕获的匹配的一部分用两种方式：一个就是匹配本身 ， 另一个在字符串替换中 。

~~~js

    const html = "<b class='hello'>Hello</b><i>world!</i>";
    const pattern = /<(\w+)([^>]*)><.*?)<\/\1>/g;
    let match = pattern.exec(html);
    assert(match[0] === "<b class='hello'>Hello</b>",
        "The entire tag, start to finish.");
    assert(match[1] === "b", "The tag name.");
    assert(match[2] === " class='hello'", "The tag attributes.");
    assert(match[3] === "Hello", "The contents of the tag.");

    match = pattern.exec(html);
    assert(match[0] === "<i>world!</i>",
    "The entire tag, start to finish.");
    assert(match[1] === "i", "The tag name.");
    assert(match[2] === "", "The tag attributes.");
    assert(match[3] === "world!", "The contents of the tag.");

~~~

我们使用 \1 来引用表达式中的首次捕获，在此情况下就是tag的名字。用此信息，我们可以匹配对应的封闭标记，引用到匹配的任何东西。（这都是假设，当然
没有任何内嵌的同名标记在当前tag中，所以 这是一个不绝对正确的标记匹配的例子）。

此外我们可以获取捕获的引用可以用在字符串替换replace方法调用中。
>
    assert("fontFamily".replace(/[A-Z]/g,"-$1").toLowerCase() === "font-family", "Convert the camelCase into dashed notation.");
在此代码中，第一次捕获的值（此地，大写字母 F）在替换字符串（通过 $1）被引用。这允许我们指定一个替换字符串 甚至不用知道其值是什么直到匹配时。    
这是一个强大的忍者武器。

因为捕获和表达式归组 都使用的括号指定，正则表达式处理器没办法知道添加到正则表达式中的哪个括号集用于归组哪个的意图表示捕获。它对待所有的括号集同时作为
归组和捕获，这会导致比真正的意图捕获更多的信息，因为我们需要在正则式中指定一些分组。在这样情况下我们能做什么。

### 非捕获的分组
如我们已经指出的，括号兼当两种职责： 他们不仅归组terms用于操作，也指定捕获。这通常情况下不是问题，但在正则表达式中更多的分组在进行，他会导致更多无用
的捕获，这会使得归类捕获结果很冗长。
    考虑下面的正则式:
>   const pattern = /((ninja-)+)sword/ ;
这里，意图是创建一个正则式 其允许前缀ninja- 在sword前出现一次或者多次，并且我们想捕获整个前缀。此正则式需要两个括号集：
- 一个括号定义捕获（任何在字符串sword之前的）。
- 一个括号 针对+操作符 用于归组文本 ninja- 。
这些工作都很好，但它导致了多于单独捕获的意图 因为内部的归组括号集。

为了表示括号的集合不应该导致一个捕获，正则表达式语法允许我们  在开始的括号之后放一个符号 ?:。 这就是被所知的 **passive subexpression(被动子表达式)**
改变表达式为：
>   const pattern = /((?:ninja-)+)sword/;
导致只有外侧的括号集 创建一个捕获。内部的括号已经被转换为一个 passive subexpression。
为了测试这个，看看下面的代码.

~~~js

    // 使用被动子表达式
    const pattern = /((?:ninja-)+)sword/;
    const ninjas = "ninja-ninja-sword".match(pattern);

    assert(ninjas.length === 2, "Only one capture was returned.");
    assert(ninjas[1] === "ninja-ninja-",
        "Matched both worlds, without any extra capture.");
~~~
运行这个测试，我们可以看到被动子表达式/((?:ninja-)+)sword/阻止不必要的捕获。

只要有可能在我们的正则表达式中，当捕获不是必须的，我们就应劲量使用非捕获（被动）分组 来替换捕获，以便 表达式引擎  在记忆和返回捕获上 做更少的工作。
如果我们不需要捕获结果，就没必要询问他们！

现在，让我们转移注意力到另一正则表达式给予我们的能力： String对象的replace方法 和函数一起

## 用函数替换
String对象的replace方法是一个强大和通用的方法，当一个正则表达式作为replace方法的第一个参数时，会导致一个替换在对模式的匹配上进行替换（或者所有匹配 如果正则是全局的），而不是
一个固定的字符串。

比如，我们想替换所有在字符串中的大写字符X。我们可以这样写：
>   "ABCDEfg".replace(/[A-Z]/g,"X")

但或许 replace给予的更强大的特征是其能够提供一个函数作为替换值而不是一个固定字符串。

当替换的值（参二）是一个函数，其会在每次找到的匹配上 被调用（在源串中的全局搜索会匹配所有的模式实例）用一个参数的变量列表：
- 匹配的全文本
- 匹配的捕获集，对每个参数一个
- 在源串中的匹配的索引
- 源字符串

从函数返回的值充当替换值。

~~~js

    function upper(all,letter){return letter.toUpperCase() ; }
    assert("border-bottom-width".replace(/-(\w)/g ,upper) === "borderBottomWidth",
        "Camel cased a hyphenated string.");
~~~
这里我们提供了一个正则式 器匹配任何以横线-为前缀的字符，在全局正则表达式中的捕获标识匹配的字符（没有dash -）。每次函数被调用（此例中 两次）
全部匹配的字符串作为第一个参数，捕获（对此正则只有一次）作为第二个参数。我们不关心其他的参数，因此我们没有指定他们。

因为全局正则式会导致在源串中的每次匹配执行替换函数。，这种技术甚至可以被扩展 而不仅仅执行粗鲁的替换。除了 在while循环中调用 exec() 我们可以使用
这个技术作为字符串遍历的手段。

比如，使用查询字符串 并转换为另一种符合我们目的的格式。
我们转换下面的请求串
> foo=1&foo=2&blah=a&blah=b&foo=3
为
>   foo=1,2,3&blah=a,b
一个方案 使用正则式和replace可能导致一些特别简洁的代码。

~~~js

    function compress(source){
        const keys = {};
        source.replace(
            /([^=&+])=([^&]*)/g,
            function(full, key, value){
                keys[key] = 
                    (keys[key] ? keys[key] + ",":"") + value ;
                return "" ;
            }
        );
        const result = [] ;
        for(let key in keys){
            result.push(key + "="+keys[key]);
        }
        return result.join("&");
    }

    assert(compress("foo=1&foo=2&blah=a&blah=b&foo=3") === "foo=1,2,3&blah=a,b",
        "Compression is OK!");
~~~
此例中最有趣的一方面就是用字符串replace方法作为对值变量字符串的手段，而不是一个 搜索-并-替换机制。

使用这种技术，我们可以co-opt 字符串对象的replace方法作为我们的 字符串-搜索 机制，结果不光快，而且也简单和高效。
此技术提供的强大级别，特别是在轻量的小量代码中，不应该被低估。

所有的这些正则表达式技术会在我们页面上很大的影响到我们脚本的编写。

## 用正则表达式解决通用问题

在JS中 一些惯用法趋向于一遍一遍的重现，但它们的方案并不总那么明显。

### 匹配新行
当执行一个搜索时，有时需要 点(.) term，它匹配任何非新行的字符， 为了也包括新行字符。在其他语言中正则式实现经常包含一个标识flag 用来使之变成可能。
但js中的实现不这样。

让我们看看一些方法 针对此JS中的遗漏

~~~js

    const html = "<b>Hello</b>\n<i>world</i>";
    // 展示 新行并没有被匹配
    assert(/.*/.exec(html)[0] === "<b>Hello</b>",
        "A normal capture doesn't handle endlines.");

    // 使用空白匹配所有东西
    assert(/[\S\s]*/.exec(html)[0] ===
        "<b>Hello</b>\n<i>world!</i>",
        "Matching everything with a character set.");

    // 可替换的 全匹配  实现   
    assert(/(?:.|\s)*/.exec(html)[0] ===
        "<b>Hello</b>\n<i>world!</i>",
        "Using a non-capturing group to match everything.");
~~~

/[\S\s]*/ 定义了一个字符类 匹配任何 不是一个空白的字符 和 任何是一个空白的字符。此并集就是所有字符的集合。

注意 /(?:.|\s)*/ 用了 passive subexpression用来阻止不希望的捕获。

这些方案中 因其简单（隐含的速度优势），由/[\S\s]*/提供的方案通常认为是最理想的。

接下来，让我们在走一步来开阔下我们对全球的视野

### 匹配Unicode
扩展ascII字符集 为包含Unicode字符有时是需要的，显式支持多语言未被覆盖在传统的阿尔法字符集合中。

~~~js

    const text = "\u5FCD\u8005\u30D1\u30EF\u30FC";
    const matchAll = /[\w\u0080-\uFFFF_-]+/;
    assert(text.match(matchAll),"Our regexp matches non-ASCII!");
~~~
此列表包含了整个Unicode字符集的整个区间范围在匹配中通过创建一个字符类 其包括\w term ，为了匹配所有的的 "常规" 字 字符，加上一个横跨整个Unicod字符
的区间（U+0080之上）

### 匹配转义字符

~~~js
    /**
    * 此正则表达式允许任何字字符，反斜杠后跟任何字符（甚至一个反斜杠），或者这二者 的组合序列，
    */
    const pattern = /^((\w+)|(\\.))+$/;
    /**
    * 设置多个测试主题。所有的应该都通过，除了最后一个， 其会失败因为忽略它的非字字符(:)
    */
    const tests = [
        "formUpdate",
        "form\\.update\\.whatever",
        "form\\:update",
        "\\f\\o\\r\\m\\u\\p\\d\\a\\t\\e",
        "form:update"
    ];
    for (let n = 0; n < tests.length; n++) {
        assert(pattern.test(tests[n]),
        tests[n] + " is a valid identifier" );
    }
~~~