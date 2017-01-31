## 字符串中嵌入模板字面量表达式
~~~

    
var ninja = 'hi  i am  yiginq' ;
var tplStr = `${ninja}` ;

console.log(tplStr) ;
~~~

## 块作用于变量

~~~

    let ninja = 'yiqing' ;

~~~

## 使用const 关键字创建块级常量变量 其值不可被重新赋值
~~~

    const ninja = 'yiqing' ;
~~~

## 函数参数:
- 余参 从参数中创建一个数组
    function multiMax(first, ...remaining) { /*...*/ }
    multiMax(2,3,4,5)
    // first => 2;
    // remaining => [3,4,5]

- 默认参数 ，指定参数的默认值 在调用时如果没有提供值就会用它的。
    function do(ninja, action='dftAction'){  return ninja + ' ' + action ;  }
    do('ok')

- 箭头函数
  使用更短的语法 他们没有其自己的this参数，而是继承this自创建他们的上下文哪里
  ~~~

    const values = [0,3,2,5,7,4,8,1];
    values.sort( (v1,v2) => v1 - v2 ) /*OR*/ values.sort((v1,v2)=> { return v1 - v2 ;}) ;
    value.forEach(value => console.log(var)) ；
  ~~~        

## 生成器
基于一个预请求生成值的序列（生成器 在有些语言中是可以让函数返回一个函数来做的 因为函数具有延迟执行的特性）
一旦一个值生成，生成器会无阻塞的挂起执行，使用 yield 来生成值：
~~~

    function *IdGenerator(){
        let id = 0 ;
        while(true){ yield ++ id ; }
    }
~~~

## Promises( 承诺  --  佛经里面有五不翻 )
Promises 是用于计算结果的占位符。一个Promise 是一个保证---最终我们会知道某些计算的结果的。promise可以成功或者失败
一旦被完成，就不会变化了。
- 使用new promise 来创建   new Promise((resolve, reject) => {} ) ;
- 调用resolve函数来显式的resolve一个promise。调用reject 函数来显式的reject 一个promise 。一个Promise在
错误发生时会隐式地被拒绝 。
- promise 对象有个then函数 它返回一个promise 并持有两个回调 ，一个成功success回调和一个失败failure回调。
    myPromise.then(val => console.log("Sucess"), err => console.log("Error")  )
- 在catch方法中链式捕获promise failures : myPromise.catch(e => alert(e) ) ;    