处理集合
-------

数组 JS中最基本的集合类型，

## 数组
数组是最常用的数据类型之一，使用它你可以处理元素的集合。

在js中 数组只是 objects。尽管这会导致一些不幸的边际效应，主要是性能，他也有一些益处。
比如数组可以访问方法，通其他对象一样 --- 方法使我们的生活更容易些。

### 创建数组

有两种基础的方式来创建新数组：
* 使用内建的Array构造器
* 使数组字面量 []

~~~

    const ninjas = ["Kuma","Hattori","Yagyu"];
    const samurai = new Array("Oda", "Tomoe");

    assert(ninjas.length === 3, "There are three ninjas");
    assert(samurai.length === 2, "And only two samurai");

    assert(ninjas[0] === "Kuma","Kuma is the first ninja");
    assert(samurai[samurai.length -1 ] === "Tomoe", 
        "Tomoe is the last samurai");

    assert(ninjas[4] === undefined,
        "We get undefined if we try to access an out of bounds index");

    ninja[4] = "Ishi";
    assert(ninjas.length === 5, 
        "Arrays are automatically expanded");

    // 手动用一个更低的值设置 length 属性 会删掉超出的元素。
    ninjas.length = 2;
    assert(ninjas.length === 2, "There are only two ninjas now");
    assert(ninjas[0] === "Kuma", && ninjas[1] === "Hattori",
        "Kuma and Hattori");
    assert(ninjas[2] === undefined,"But we've lost Yagyu");
~~~

>
    数组字面量 VS 数组构造器

    应该优先使用前者。主要原因是简洁性： [] VS new Array() 
    此外，JS是高度动态的，没有什么能够阻止一些人覆盖内建的Array构造器，这即意味着调用 new Array() 并不一定会创建一个数组。
    因此我们推荐你普遍坚持使用数组字面量 。

不管我们怎样创建一个数组，每个数组都有一个length属性，它指定了数组的长度 。
你可能已经知道了，通过索引符号可以访问数组的元素，第一个元素定位在索引0 最后一个元素在索引 array.length-1 位置.
但如果我们指定的数组索引超出了范围，我们不会得到一个 像其他编程语言中的那个 类 “Array index out of bounds”警告 异常 （即数组越界异常） 。    
替换的是 undefined 被返回了，告诉你哪儿没东西啦.

这种行为是 JS的数组实际上是对象的结果 。 正如我们试图访问一个不存在对象属性是会得到 undefined 那样，在访问一个
不存在的数组索引时，我们也会得到一个undefined 。

另一方面，如我我们试图对一个超出数组范围的位置进行写操作 ，数组会扩展到适应的新位置。
这个有时候会导致一些 **洞**出现 。

var ninjas = ["Kuma", "Hattori", "Yagyu"];
ninjas[4] = "Ishi";

| 索引0 | 1  | 2 | 3 | 4 |
|--- |---  | --- | --- |---  |
| "Kuma" | "Hattori" | "Yagyu" | undefined | "Ishi" |



没有什么可以阻止我们手动修改 length 的值。
设定一个高于当前length的值会使用undefined元素来扩张数组， 而 设定个一个更低的值 会缩减数组。

接下来看看一些常用的数组方法。

***

我是分割线(使用\*** 或者 \---)

---

### 数组结尾添加或者移除元素

让我们从下面的简单方法开始，我们可以 从数组 用其来添加元素 移除元素 ：
* push 在数组的尾部添加元素 
* unshift 在数组的头部添加元素
* pop 在数组尾部移除元素
* shift 在数组的头部移除一个元素

~~~
    const ninjas = [];
    assert(ninjas.length === 0, "An array starts empty");

    ninjas.push("Kuma");
    assert(ninjas[0] === "Kuma",
        "Kuma is the first item in the array");
    assert(ninjas.length === 1, "We have one item in the array");
   
    ninjas.push("Hattori");
    assert(ninjas[0] === "Kuma",
        "Kuma is still first");
    assert(ninjas[1] === "Hattori",
        "Hattori is added to the end of the array");
    assert(ninjas.length === 2,
        "We have two items in the array!");

    ninjas.unshift("Yagyu");
    assert(ninjas[0] === "Yagyu",
        "Now Yagyu is the first item");
    assert(ninjas[1] === "Kuma",
        "Kuma moved to the second place");
    assert(ninjas[2] === "Hattori",
        "And Hattori to the third place");
    assert(ninjas.length === 3,
        "We have three items in the array!");

    const lastNinja = ninjas.pop();
    assert(lastNinja === "Hattori",
        "We've removed Hattori from the end of the array");
    assert(ninjas[0] === "Yagyu",
        "Now Yagyu is still the first item");
    assert(ninjas[1] === "Kuma,
    "Kuma is still in second place");
    assert(ninjas.length === 2,
        "Now there are two items in the array");

    const firstNinja = ninjas.shift();
    assert(firstNinja === "Yagyu",
        "We've removed Yagyu from the beginning of the array");
    assert(ninjas[0] === "Kuma",
        "Kuma has shifted to the first place");
    assert(ninjas.length === 1,
        "There's only one ninja in the array");
~~~

注意在尾部或者头部的 增删操作 会引起的 索引变化！

>
    性能考虑
    pop 和 push 方法只影响一个数组的最后一个元素：pop 移除最后一个元素，push在数组的尾部插入一个元素。 另一个方面shift和unshift方法
    改变数组中的第一个元素。这意味着后续的数组元素的索引必须调整。因此push和pop是比shift和unshift更快的操作，我们推荐你使用他们除非你有更好的
    使用后者的理由。

### 在数组的任何位置添加和移除元素
在数组尾部或者头部 增删元素太局限了 通常，我们应该可以从数组的任何位置移除元素。
~~~

    const  ninjas = ["Yagyu", "Kuma", "Hattori","Fuma"];

    // 使用delete 命令来删除一个元素。
    delete ninjas[1];

    // 我们删除了一个元素，但是数组仍旧报告它有4个元素，我们只是在数组中创建了一个“洞”（其值是undefined）。
    assert(ninjas.length === 4,
        "length still reports that there are 4 items");

    assert(ninjas[0] === "Yagyu", "First item is Yagyu");
    assert(ninjas[1] === undefined, "We've simply created a hole");
    assert(ninjas[2] === "Hattori", "Hattori is still the third item");
    assert(ninjas[3] === "Fuma", "And Fuma is the last item");
~~~    

相似，如果我们想在任意位置插入一个元素，我们应该从哪里开始？
作为对这些问题的回答，所有的JS数组可以访问splice方法：这个方法 从给定索引开始 移除或者插入元素

~~~
    const ninjas = ["Yagyu", "Kuma", "Hattori", "Fuma"];
    // 参一是起始位置，参二是移除的元素数目（如果不指定，所有直到数组尾部的元素都将被删除）
    var removedItems = ninjas.splice(1, 1);

    // splice方法返回值 就是被移除的数组
    assert(removedItems.length === 1, "One item was removed");
    assert(removedItems[0] === "Kuma");

    assert(ninjas.length === 3,
        "There are now three items in the array");
    // 某个位置删除一个元素后  后续元素依次shift（移位）
    assert(ninjas[0] === "Yagyu",
         "The first item is still Yagyu");
    assert(ninjas[1] === "Hattori",
        "Hattori is now in the second place");
    assert(ninjas[2] === "Fuma",
        "And Fuma is in the third place");

    // 我们可以插入元素在一个位置上 通过给splice调用添加参数 。
    removedItems = ninjas.splice(1, 2, "Mochizuki", "Yoshi", "Momochi");
    assert(removedItems.length === 2, "Now, we've removed two items");
    assert(removedItems[0] === "Hattori", "Hattori was removed");
    assert(removedItems[1] === "Fuma", "Fuma was removed");
    assert(ninjas.length === 4, "We've inserted some new items");
    assert(ninjas[0] === "Yagyu", "Yagyu is still here");
    assert(ninjas[1] === "Mochizuki", "Mochizuki also");
    assert(ninjas[2] === "Yoshi", "Yoshi also");
    assert(ninjas[3] === "Momochi", "and Momochi");     
~~~

### 常用的数组操作
- Iterating 迭代
- Mapping 基于既存数组来创建一个新数组
- Testing 测试数组元素来检测其是否满足特定条件
- Finding 查找特定数组元素
- Aggregating 聚合数组 并基于数组元素计算一个单值（比如 计算数组元素和） 

--- 

* 迭代数组
~~~JS

    const ninjas = [];
    for(let i = 0 ; i<ninjas.length; i++){
        assert(ninjas[i] !== null , ninjas[i]);
    }
~~~
为了变量一个数组，我们不得不设置一个计数器变量 i ，指定其上限(ninjas.length),定义计数器如何变化（i++），对于执行这种通用的操作这样做比有些丑。
这也是一些恼人bugs的根源。此外使我们的代码更难阅读。读者必须仔细查看for循环的每一个部分，以确信遍历了所有的元素而没有跳过某一个。

为了使生活更容易些，所有的JS数组 有一个内建的forEach方法 我们可以在此情况下使用.
~~~JS

    // 使用内建的forEach方法来遍历一个数组
    const ninjas = [] ;
    ninjas.forEach(ninja => {
        assert(ninja !== null, ninja) ;
    });
~~~
我们提供了一个回调（这里  一个箭头函数），它被称为 **immediately**，对数组中的每个元素。这样 不在有 起始索引结束条件，或者精确的递增数。
JS引擎 在后台为我们做了这些事。

* 数组映射

收集忍者的武器
~~~JS

    const ninjas = [
        {name: "Yagyu", weapon: "shuriken"},
        {name: "Yoshi", weapon: "katana"},
        {name: "Kuma", weapon: "wakizashi"}
    ] ;

    const weapons = [] ;

    ninjas.forEach(ninja => {
        weapons.push(ninja.weapon);
    })
~~~

你可能想，基于已有数组创建一个新数组是很常用的，以至于 有一个特殊名称： mapping an array（映射数组）。
思想是 映射一个数组中的每一个元素 到 新数组的一个新元素。
惯例，JS 有一个map函数就做的这件事。

~~~js
    // ...
    // 内建的map函数 接受一个函数 他会在数组的每个元素上被调用
    const weapons = ninjas.map(ninja => ninja.weapon );
    // ...
~~~
内建的map方法 构造一个全新的数组 之后遍历输入的数组。对于输入数组中的每个元素，map基于提供给他的回调函数的结果，在新构建的数组上放一个元素。

* 测试数组元素 

当处理元素集合时，我们经常会碰到一种情形，需要知道是否 所有或者至少数组中的一些 满足特定条件。为了劲量写出这样的高效代码.所有JS数组可以访问
内置的 every 和 some 函数 如下示：

~~~js

    const ninjas = [
        {name: "Yagyu", weapon: "shuriken"},
        {name: "Yoshi" },
        {name: "Kuma", weapon: "wakizashi"}
    ];

    // 内建的every方法 接收一个回调 对每个数组元素会调用它。如果回调函数对所有的数组元素返回一个true值 那么它返回true值，否则返回false。
    const allNinjasAreNamed = ninjas.every(ninja =>"name" in ninja);
    const allNinjasAreArmed = ninjas.every(ninja => "weapon" in ninja);

    assert(allNinjasAreNamed,"Every ninja has a name");
    assert(!allNinjasAreArmed, "But not every ninja is armed");
    /**
    *  内建的some方法也接受一个回调。对数组中的至少一个元素如果回调返回一个true值它就返回true，否则false。
    */
    const someNinjasAreArmed = ninjas.some(ninja => "weapon" in ninja);
    assert(someNinjasAreArmed, "But some ninjas are armed");


~~~

* 搜索数组

NOTE 内建的 find 是ES6标准的 方法

~~~js

    const ninjas = [
        {name: "Yagyu", weapon: "shuriken"},
        {name: "Yoshi" },
        {name: "Kuma", weapon: "wakizashi"}
    ];
   
    const ninjaWithWakizashi = ninjas.find(ninja => {
        return ninja.weapon === "wakizashi" ;
    }) ;

   assert(ninjaWithWakizashi.name === "Kuma"
            && ninjaWithWakizashi.weapon === "wakizashi"
            , "Kuma is wielding a wakizashi");

    const ninjaWithKatana = ninjas.find(ninja => {
        return ninja.weapon === "katana" ;
    });

    assert(ninjaWithKatana === undefined,
            "We couldn't find a ninja that wields a katana ");

    const armedNinjas = ninjas.filter(ninja => "weapon" in ninja);

    assert(armedNinjas.length === 2, "There are two armed ninjas: ");
    assert(armedNinjas[0].name === "Yagyu"
            && armedNinjas[1].name === "Kuma", "Yagyu and Kuma");

~~~
如果我们需要找到多个满足某条件的元素，我们可以使用filter函数，他创建了一个新数组 其中的所有元素满足那个条件。
>   const armedNinjas = ninjas.filter(ninja => "weapon" in ninja);
创建一个新的 armedNinjas 数组 它只包含有武器的忍者。

有时 查找某个元素的索引也是很必须的
~~~js

    const ninjas = ["Yagyu", "Yoshi", "Kuma", "Yoshi"];
    assert(ninjas.indexOf("Yoshi") === 1,"Yoshi is at index 1" );
    assert(ninjas.lastIndexOf("Yoshi") === 3, "and at index 3");

    const yoshiIndex = ninjas.findIndex( ninja => ninja === "Yoshi");
~~~

有时我们也对查找最后的索引感兴趣 为此我们可以使用 lastIndexOf 方法：
>   ninjas.lastIndexOf("Yoshi");

findIndex 方法 接受一个回调 并返回第一个 回调返回true值的元素的索引。基本上 它工作很像find方法，唯一的区别是 find方法返回特定元素，而findIndex 返回那个
元素的索引。

### 排序数组

最常用的数组操作之一就是sorting 排序。不幸的是 正确地实现排序算法 不是那么简单的程序任务： 我们必须选择最好的排序算法 。

JS 引擎 实现了排序算法，其用法看起来如此：
> array.sort( (a, b) => a - b ) ;
唯一我们要做的就是 提供一个回调 告知排序算法关于两个数组元素的关系，可能的结果如下：
- 如果回调返回一个小于0的值，元素a 应该在元素b 之前。
- 如果回调返回一个等于0 的值， 元素a 和b 相等。
- 如果回调返回一个大于0 的值， 元素 a 应该 位于元素b 之后。

关于排序算法，这就是你需要知道的所有东西。实际的排序在后台执行，我们不必手动移动元素。

~~~js

    const ninjas = [{name: "Yoshi"}, {name: "Yagyu"}, {name: "Kuma"}];
    ninjas.sort(function(ninja1, ninja2){
        if(ninja1.name < ninja2.name) { return -1; }
        if(ninja1.name > ninja2.name) { return 1; }
        return 0;
    });

    assert(ninjas[0].name === "Kuma", "Kuma is first");
    assert(ninjas[1].name === "Yagyu", "Yagyu is second");
    assert(ninjas[2].name === "Yoshi", "Yoshi is third");
~~~
排序算法的细节就留给JS引擎了，我们不需要担心他们。

### 聚合数组元素
~~~js

    const numbers = [1,2,3,4];
    const sum = 0 ;

    numbers.forEach(number => {
        sum += number ;
    });

    assert(sum === 10 , "The sum of first four numbers is 10");
~~~
此段代码不得不遍访集合中的每个元素 并聚合一些值，消减（reduce）整个数组到一个单独的值。
不要担心  JS 也有一些东西帮助我们应对这样的情况： reduce 方法。

~~~js

    const numbers = [1,2,3,4];

    // 使用reduce方法 从一个数组聚合单独值
    const sum = numbers.reduce((aggregated, number) => aggregated + number ,0 );

    assert(sum === 10, "The sum of first four numbers is 10 ");
~~~
reduce 方法 通过持有一个初始值（此处 0 ）并在每个数组元素上调用回调函数 用前面回调执行的结果和当前数组元素作为参数。reduce调用的结果是上次
调用于上个数组元素上 回调的结果，

（reduce  类似 卷竹帘 百叶窗  有时也被称为 fold    是 著名的Map-Reduce 算法的一个部分）

### 重用内建的数组函数
很多时候 我们会想创建一个对象 他包含一个数据的集合。 如果集合就是我们全部关注的，我们就可以使用一个数组。但在某些情况下，会有 除过集合之外
的更多的状态去存储-- 或许我们需要存储一些关于集合元素的元数据。

~~~html

<body>
    <input in="firs"/>
    <input id="second"/>

    <script>
        const elems = {
            length: 0,
            add: function(elem){
                Array.prototype.push.call(this,elem);
            },
            gather: function(id){
                this.add(document.getElementById(id));
            },
            find: function(callback){
                return Array.prototype.find.call(this, callback) ;
            }
        };

        elems.gather("first");
        assert(elems.length === 1 && elems[0].nodeType,
            "Verify that we have an element in our stash");

        elems.gather("second");
        assert(elems.length === 2 && elems[1].nodeType,
         "Verify the other insertion");
      
        elems.find(elem => elem.id === "second");
        assert(found && found.id === "second",
            "We've found our element");
    </script>

</body>

~~~

此例中我们创建了一个“常规”的对象 并且武装它模拟数组的一些功能。
并非使用我们自己的代码，我们可以使用一个JS原生的方法： Array.prototype.push .

通常，push方法 通过它的函数上下文 会作用在其自己的数组上。但这里 我们通过使用call方法 哄骗方法去使用我们自己的对象作为其上下文，并强制我们的对象
作为push方法的上下文。push方法，会递增length属性，

本例 旨在 展示 如何聪明的使用既存代码 而不是 经常去重新发明轮子。

## Maps

~~~js

    const dictionary = {
    "ja": {
         "Ninjas for hire": " レンタル用の忍者"
    },
    "zh": {
         "Ninjas for hire": " 忍者出租"
    },
    "ko": {
         "Ninjas for hire":" 고용 닌자 "
    }
    }
    assert(dictionary.ja["Ninjas for hire"] === " レンタル用の忍者");
~~~

初看这是个应对国际化问题的不错方法。

### 不要使用对象作为maps

所有对象都有原型，即便我们定义新的 空对象作为我们的maps，他们仍旧可以访问原型对象的属性。这些属性中的一个就是constructor
此外，使用对象，keys只能是字符串值；如果你想创建一个可用任何其他值的mapping，都不问你任何问题 其值会被默默地转变为字符串 。

~~~html

<div id="firstElement"></div>
<div id="secondElement"></div>
<script>
    const firstElement = document.getElementById("firstElement");
    const secondElement = document.getElementById("secondElement");
    const map = {};
    map[firstElement] = { data: "firstElement"};
    assert(map[firstElement].data === "firstElement",
    "The first element is correctly mapped");
    map[secondElement] = { data: "secondElement"};
    assert(map[secondElement].data === "secondElement",
    "The second element is correctly mapped");
    assert(map[firstElement].data === "firstElement",
    "But now the firstElement is overriden!");
</script>


~~~

对象键被存储为字符串，这意味着当我们存储费字符串值比如 HTML 元素时，作为对象的属性，此值会被悄悄地转换为字符串通过调用其 toString方法。
此处 它返回的字符串： "[object HTMLDivElement]"; 

两个元素都将被转换为相同的字符串，尽管他们有不同的id。这样 后面的会覆盖前面的。

因为这两个原因，ECMAScript 委员会指定了一个全新的类型：Map。

NOTE Maps 是ES6标准的。

### 创建我们第一个map
创建map是很容易的： 我们使用new ，内建的Map 构造器。

~~~js

    const ninjaIslandMap = new Map() ;

    const ninja1 = { name: "Yoshi" };
    const ninja2 = { name: "Hattori" };
    const ninja3 = { name: "Kuma" };
    
    ninjaIslandMap.set(ninja1, { homeIsland: "Honshu" });
    ninjaIslandMap.set(ninja2, { homeIsland: "Hokkaido" });

    assert(ninjaIslandMap.get(ninja1).homeIsland === "Honshu",
        "The first mapping works");
    assert(ninjaIslandMap.get(ninja2).homeIsland === "Hokkaido",
        "The second mapping works");

    assert(ninjaIslandMap.get(ninja3) === undefined,
         "There is no mapping for the third ninja!");
    assert(ninjaIslandMap.size === 2,
            "We've created two mappings");
    assert(ninjaIslandMap.has(ninja1)
          && ninjaIslandMap.has(ninja2),
         "We have mappings for the first two ninjas");
    assert(!ninjaIslandMap.has(ninja3),
         "But not for the third ninja!");
  
    ninjaIslandMap.delete(ninja1);
    assert(!ninjaIslandMap.has(ninja1)
             && ninjaIslandMap.size() === 1,
             "There's no first ninja mapping anymore!");

    ninjaIslandMap.clear();
    assert(ninjaIslandMap.size === 0,
         "All mappings have been cleared");
~~~

除了get和set 方法，每个map 也有一个内建的size属性 和 has delete方法。
size属性告诉我们我们已经创建了多少mappings 。
has 方法，告诉我们对特定键的mapping是否存在。

delete方法让我们可从map中移除元素。

### key相等
如果你来自一些更传统背景，比如C# Java 或者Python，你可能对下面的例子感到惊奇：

~~~js

    const map = new Map();
    const currentLocation = location.href ;

    const firstLink = new URL(currentLocation);
    const secondLink = new URL(currentLocation);

    map.set(firstLink, { description: "firstLink" });
    map.set(secondLink, { description: "secondLink" });

    assert(map.get(firstLink).description === "firstLink",
        "First link mapping");

    assert(map.get(secondLink).description === "secondLink",
        "Second link mapping");
    assert(map.size === 2, "There are two mappings");
~~~
我们可能认为两个URL对象应该相当（都创建于同一个连接地址）， 但在JS中，我们不能重载等号操作符，即便他们有相同的内容，也总是被认为
不相同的。着在其他语言中可不是这么回事，比如Java 和 C# ，所以 要注意哦！

    遍历maps

至此，你已经看到了maps的一些优点：你可以使用任何东西作为一个key，但有更多的！
因为mas是集合，没有什么能够阻止我们用for...of 循环遍历他们，你也可以确信 这些值被访问的顺序和其被插入的顺序一样。

~~~js

    /**
    * 创建一个新的map， ninja字典 它存储每个ninja的电话号码
    */
    const directory = new Map() ;

    directory.set("Yoshi", "+81 26 6462");
    directory.set("Kuma", "+81 51 2378 6462");
    directory.set("Hiro", "+81 76 277 46");

    // 使用for...of 循环 迭代字典中的每个项，每个元素项是一个两元素的数组： 一个key和一个value
    for(let item of directory){
        assert(item[0] !== null , "Key:" + item[0]);
        assert(item[1] !== null , "Value:" + item[1]);
    }
    // 我们也可以迭代keys 使用内置的keys方法
    for(let key of directory.keys()){
        assert(key !== null, "Key:"+key);
        assert(directory.get(key) != null, 
            "Value:" + directory.get(key));
    }
    // ... 迭代值 使用内建的values方法
    for(var value of directory.values()){
        assert(value !== null, "Value:"+ value);
    }

~~~    
如前所示，一旦我们创建了一个mapping，我们就可以 使用for...of循环 很容易迭代它：

## Sets 集
唯一元素的集合 这些元素相互区别（意即 每个元素 不能出现多次 -- 至多一次） 称之为sets。

ES6之前，你必须自己实现它 通过使用标准对象来模拟sets。

~~~js
    // 使用对象存储元素
    function Set(){
        this.data = {};
        this.length = 0 ;
    }

    // 检测元素是否已经被存储了
    Set.prototype.has = function(item){
        return typeof this.data[item] !== "undefined" ;
    }
    // 如果在集合中不存在 我们才添加这个元素
    Set.prototype.add = function(item){
        if(!this.has(item)){
            this.data[item] = true ;
            this.length ++ ;
        }
    }
    // 如果集合中已经包含该元素 移除之
    Set.prototype.removed = function(item){
        if(this.has(item)){
            delete this.data[item];
            this.length-- ;
        }
    }

    const ninjas = new Set();
    ninjas.add("Hattori");
    ninjas.add("Hattori");
    assert(ninjas.has("Hattori") && ninjas.length === 1,
        "Our set contains only one Hattori");

    ninjas.remove("Hattori");
    assert(!ninjas.has("Hattori") && ninjas.length === 0,
        "Our set is now empty");
~~~
这里展示了一个简单的例子 如何使用对象来模拟sets 。
我们可以用一个数据存储 对象，data来跟踪我们的集合元素，我们暴露了三个方法：
- has， 他检测一个元素是否已经包含在sets中了
- add， 只在相同的元素不存在于set中时 添加一个元素
- remove， 从set中移除已经存在的元素

但这里有个缺点，因为使用maps 你不能真正存储一个对象--- 只能字符串和数字--- 并有访问原型对象的风险。
因为这些原因，ECMAScript委员会决定引入一个全新的集合类型： Sets

NOTE Sets 是ES6标准的一部分

### 创建我们第一个set

~~~js
    // 集合构造器 可以接受一个元素的数组 用来初始化它
    const ninjas = new Set(["Kuma", "Hattori", "Yagyu", "Hattori"]);

    assert(ninjas.has("Hattori"),"Hattori is in our set");
    assert(ninjas.size === 3, "There are only three ninjas in our set!");

    assert(!ninjas.has("Yoshi"), "Yoshi is not in, yet..");
    ninjas.add("Yoshi");
    assert(ninjas.has("Yoshi"), "Yoshi is added");
    assert(ninjas.size === 4, "There are four ninjas in our set!");

    assert(ninjas.has("Kuma"), "Kuma is already added");
    ninjas.add("Kuma");
    assert(ninjas.size === 4, "Adding Kuma again has no effect");

    for(let ninja of ninjas) {
        assert(ninja !== null, ninja);
    }
~~~
此处，我们使用内建的Set 构造器来创建一个新的ninjas set ，它将包含不同的ninjas。
如果我们没有传递任何元素，一个空的set被创建 。 我们也可以传递一个包含元素的数组 用来**预填充**set：
> new Set([.....]);

我们已经提及，set 是唯一元素的集合，其主要目的是阻止我们存储多次出现同一个对象。

对每个set 有一些方法可以访问：
- has 方法检测 一个元素是否已经包含在set中了
- add 方法用来添加一个唯一元素到set中。

如果你对set中有多少元素比较好奇 你总是可以使用size属性。

同maps和数组一样，sets 也是集合，所以我们可以使用for...of 循环来遍历他。
遍历元素的顺序和其加入的顺序一样哦！

既然我们已经了解了sets的一些基础，让我们来看看一些常用的操作：
unions , intersections differences .

### sets并集
两个sets的并集，A和B ， 创建一个新的set 它含有所有来自A和B的元素。
自然，每个元素不能在新的set中出现多次。

~~~js

    const ninjas = ["Kuma","Hattori","Yagyu"];
    const samurai = ["Hattori", "Oda", "Tomoe"];

    // 创建一个新的set 通过 展开ninjas和samurai 。
    const warriors = new Set([...ninjas, ...samurai]);
    assert(warriors.has("Kuma"), "Kuma is here");
    assert(warriors.has("Hattori"), "And Hattori");
    assert(warriors.has("Yagyu"), "And Yagyu");
    assert(warriors.has("Oda"), "And Oda");
    assert(warriors.has("Tomoe"), "Tomoe, last but not least");

    assert(warriors.size === 5, "There are 5 warriors in total");
~~~

我们使用了 扩展操作符 [...ninjas, ...samurai ]

### sets 交集
两个sets的交集 A和B ， 创建一个set 其包含的A的元素 也同时出现在B中

~~~js

    const ninjas = new Set(["Kuma", "Hattori", "Yagyu"]);
    const samurai = new Set(["Hattori", "Oda", "Tomoe"]);

    const ninjaSamurais = new Set(
        /** 使用扩展符 把set转换为一个数组 以便我们可以使用数组的filter方法 */
        [...ninjas].filter(ninja => samurai.has(ninja))
    );

    assert(ninjaSamurais.size === 1, "There's only one ninja samurai");
    assert(ninjaSamurais.has("Hattori"), "Hattori is his name");
~~~

### sets 的差集
两个sets的差集 A和B ， 包含所有来自A的 但不在B中的元素。
你可能会想，这跟sets的交集很像 只有一个小但很显著的不同。

~~~js

    const ninjas = new Set(["Kuma", "Hattori", "Yagyu"]);
    const samurai = new Set(["Hattori", "Oda", "Tomoe"]);
    
    const pureNinjas = new Set(
    [...ninjas].filter(ninja => !samurai.has(ninja))
    );
    assert(pureNinjas.size === 2, "There's only one ninja samurai");
    assert(pureNinjas.has("Kuma"), "Kuma is a true ninja");
    assert(pureNinjas.has("Yagyu"), "Yagyu is a true ninja");
~~~ 