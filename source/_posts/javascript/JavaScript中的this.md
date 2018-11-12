---
title: JavaScript中的this
date: 2018-11-12 10:10:22
categories: JavaScript
tags: [JavaScript, this, 前端]
---

&emsp;&emsp;**this是在执行上下文创建时确定的一个在执行过程中不可更改的变量。**所谓执行上下文，就是JavaScript引擎在执行一段代码之前将代码内部会用到的一些变量、函数、this提前声明然后保存在变量对象中的过程。这个'代码片段'包括：全局代码(script标签内部的代码)、函数内部代码、eval内部代码。而我们所熟知的作用域链也会在保存在这里，以一个类数组的形式存储在对应函数的[[Scopes]]属性中。
&emsp;&emsp;this只在函数调用阶段确定，也就是执行上下文创建的阶段进行赋值，保存在变量对象中。这个特性也导致了this的多变性:🙂即当函数在不同的调用方式下都可能会导致this的值不同。
```javascript
var a = 1;
function fun() { 
    'use strict';
    var a = 2;
    return this.a;
}
```
👆严格模式下，this指向undefined;
👆非严格模式下，this指向window;

&emsp;&emsp;当函数独立调用的时候，在严格模式下它的this指向undefined，在非严格模式下，当this指向undefined的时候，自动指向全局对象(浏览器中是window，node 中是 global)。

## 直接调用
当函数独立调用的时候，在严格模式下它的this指向undefined，在非严格模式下，当this指向undefined的时候，自动指向全局对象(浏览器中就是window)。

## 作为对象的方法
```javascript
var a = 1; 
var obj = { 
    a: 2, 
    b: function() { 
        return this.a; 
    } 
} 
console.log(obj.b())//2
```
👆b所引用的匿名函数作为obj的一个方法调用，这时候this指向调用它的对象。这里也就是obj。

```javascript
var a = 1; 
var obj = { 
    a: 2,
    b: function() { 
        return this.a; 
    } 
} 
var t = obj.b; 
console.log(t());//1
```
如上，t函数执行结果竟然是全局变量1，为啥呢？这就涉及Javascript的内存空间了，就是说，obj对象的b属性存储的是对该匿名函数的一个引用，可以理解为一个指针。当赋值给t的时候，并没有单独开辟内存空间存储新的函数，而是让t存储了一个指针，该指针指向这个函数。相当于执行了这么一段伪代码 👇：
```javascript
var a = 1; 
function fun() {//此函数存储在堆中 
    return this.a; 
} 
var obj = { 
    a: 2, 
    b: fun //b指向fun函数 
} 
var t = fun;//变量t指向fun函数 
console.log(t());//1
```

## 使用apply，call
修改this指向为给定值
apply 和 call 基本类似，他们的区别只是传入的参数不同。call 方法接受的是若干个参数列表，而 apply 接收的是一个包含多个参数的数组。
apply的语法：
```javascript
fun.apply(thisArg, [argsArray])
```
call的语法：
```javascript
fun.call(thisArg[, arg1[, arg2[, ...]]])
```
call() 和 apply() 是预定义的函数方法。 两个方法可用于调用函数，两个方法的第一个参数必须是对象本身
在 JavaScript 严格模式(strict mode)下, 在调用函数时第一个参数会成为 this 的值， 即使该参数不是一个对象。
在 JavaScript 非严格模式(non-strict mode)下, 如果第一个参数的值是 null 或 undefined, 它将使用全局对象替代。

## 作为构造函数
使用new关键字调用函数的过程：
```javascript
var foo = new Func() {
    // 创建空对象
    var obj = {};
    // 设置新对象的constructor属性为构造函数的名称，设置新对象的__proto__属性指向构造函数prototype对象;
    obj.__proto__ = Func.prototype;
    // 使用新对象调用函数，函数中的this被指向新实例对象;
    var result = Func.apply(obj, arguments);
    // 确保 new 出来的是个对象 返回的值是什么就return什么
    return typeof result === 'object' ? result : obj;
}
```

## 箭头函数
**箭头函数会捕获其所在上下文的 this 值，作为自己的 this 值**，也就是说箭头函数的this在词法层面就完成了绑定。apply，call方法只是传入参数，却改不了this。
```javascript
var a = 1; 
var obj = { 
    a: 2 
}; 
var fun = () => console.log(this.a); 
fun();//1 
fun.call(obj)//1
```