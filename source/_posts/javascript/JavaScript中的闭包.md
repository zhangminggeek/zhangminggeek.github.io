---
title: JavaScript中的闭包
date: 2018-11-19 23:21:22
categories: JavaScript
tags: [JavaScript, 闭包, 前端]
---

## 什么是闭包？
首先来看一下MDN中对闭包定义对解释：
> 闭包是函数和声明该函数的词法环境的组合。

emmmm...很难理解，那再来看看《JavaScript高级程序设计》一书中对闭包的解释：
> 闭包是指有权访问另一个函数作用域中的变量的函数

这看上去就比较好理解了。事实上，各种文献上对于闭包的解释都不一样，非常抽象且难看懂，所以我们直接来看闭包的创建和使用吧。创建闭包的常见方式，就是这一个函数内部创建另一个函数，下面例子中就是创建了一个闭包：
```javascript
function init() {
    var name = "Mozilla"; // name 是一个被 init 创建的局部变量
    function displayName() { // displayName() 是内部函数,一个闭包
        alert(name); // 使用了父函数中声明的变量
    }
    displayName();
}
init();
```

init() 创建了一个局部变量 name 和一个名为 displayName() 的函数。displayName() 是定义在 init() 里的内部函数，仅在该函数体内可用。displayName() 内没有自己的局部变量，然而它可以访问到外部函数的变量，所以 displayName() 可以使用父函数 init() 中声明的变量 name 。但是，如果有同名变量 name 在 displayName() 中被定义，则会使用 displayName() 中定义的 name 。

## 变量的作用域
由于闭包与它的词法环境绑在一起，因此闭包让我们能够从一个函数内部访问其外部函数的作用域。内部函数的作用域链中包含外部函数的作用域，要彻底搞清楚其中的细节，必须从理解函数被调用的时候 都会发生什么入手。

有关如何创建作用域链以及作用域链有什么作用的细节，对彻底理解闭包至关重要。当某个函数被调用时，会创建一个执行环境(execution context)及相应的作用域链。 然后，使用 arguments 和其他命名参数的值来初始化函数的活动对象(activation object)。但在作用域链中，外部函数的活动对象始终处于第二位，外部函数的外部函数的活动对象处于第三位，......直至作为作用域链终点的全局执行环境。在另一个函数内部定义的函数会将包含函数(即外部函数)的活动对象添加到它的作用域链中。因此，在函数内部定义的匿名函数的作用域链中，实际上将会包含外部函数的活动对象。

简单来说，函数内部可以读取外部变量的值，而函数外部无法读取函数内的局部变量。
```javascript
var n = 999;
function f1 () {
　　alert(n);
}

f1(); // 999
```
```javascript
function f1 () {
    var n = 999;
}

alert(n) // error
```

## 闭包的应用
### 实现对象的私有数据
闭包的用途之一是实现对象的私有数据。任何在函数中定义的变量，都可以认为是私有变量，因为不能在函数外部访问这些变量。私有变量包括函数的参数、局部变量和函数内定义的其他函数。我们可以使用闭包来模拟私有方法。私有方法不仅仅有利于限制对代码的访问：还提供了管理全局命名空间的强大能力，避免非核心的方法弄乱了代码的公共接口部分。通过闭包，我们可以使外部环境访问函数内的局部变量，把有权访问私有变量的公有方法称为**特权方法（privileged method）**。
```javascript
function Animal(){
  // 私有变量
  var series = "哺乳动物";
  function run () {
    console.log("Run!!!");
  }
  // 特权方法
  this.getSeries = function () {
    return series;
  };
}
```

### 保存变量
闭包的另一个作用就是让这些变量的值始终保持在内存中。闭包是函数开始执行的时候被分配的一个栈帧，在函数执行结束返回后仍不会被释放(就好像一个栈帧被分配在堆里而不是栈里)。
```javascript
function say667() {
  // 局部变量num最后会保存在闭包中
  var num = 42;
  var say = function () { console.log(num); }
  num++;
  return say;
}
var sayNumber = say667();
sayNumber(); // 输出 43
```

## 闭包的缺陷
1. 由于闭包会使得函数中的变量都被保存在内存中，内存消耗很大，所以不能滥用闭包，否则会造成网页的性能问题，在IE中可能导致内存泄露。解决方法是，在退出函数之前，将不使用的局部变量全部删除
2. 闭包会在父函数外部，改变父函数内部变量的值。所以，如果你把父函数当作对象（object）使用，把闭包当作它的公用方法（Public Method），把内部变量当作它的私有属性（private value），这时一定要小心，不要随便改变父函数内部变量的值。

## 闭包面试题
```javascript
function fun(n,o){
  console.log(o);
  return {
    fun: function(m){
      return fun(m,n);
    }
  };
}

var a = fun(0);  // undefined
a.fun(1);        // 0        
a.fun(2);        // 0
a.fun(3);        // 0

var b = fun(0).fun(1).fun(2).fun(3);  // undefined, 0, 1, 2

var c = fun(0).fun(1);  // undefined, 0
c.fun(2);        // 1
c.fun(3);        // 1
```

```javascript
var gLogNumber, gIncreaseNumber, gSetNumber;
function setupSomeGlobals() {
  // 局部变量num最后会保存在闭包中
  var num = 42;
  // 将一些对于函数的引用存储为全局变量
  gLogNumber = function() { console.log(num); }
  gIncreaseNumber = function() { num++; }
  gSetNumber = function(x) { num = x; }
}
setupSomeGlobals();
gIncreaseNumber();
gLogNumber(); // 43
gSetNumber(5);
gLogNumber(); // 5
var oldLog = gLogNumber;
setupSomeGlobals();
gLogNumber(); // 42
oldLog() // 5
```

```javascript
var items = document.querySelectorAll('li');
      
for(var i = 0; i < items.length; i++){
    items[i].onclick = function () {
        alert(i);
    }
}
//上面的程序结果是：每次都弹出10;
//为了在用户点击的时候，能弹出对应的数字
// 需要构建一个闭包，将参数缓存起来
for(var i = 0; i < items.length; i++){
    items[i].onclick = (function (n) {
        return function () {
            alert(n);
        }
    })(i)
}
```

## 参考文章
《JavaScript高级程序设计》
[闭包](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Closures)
[学习Javascript闭包（Closure）](http://www.ruanyifeng.com/blog/2009/08/learning_javascript_closures.html)
[JavaScript 闭包](https://juejin.im/entry/57d60f7067f3560057e37e25)
[JavaScript 闭包入门（译文）](https://juejin.im/post/58832fe72f301e00697b672d)

