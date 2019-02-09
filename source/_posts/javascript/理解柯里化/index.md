---
title: 理解柯里化
date: 2019-02-09 20:43:15
categories: JavaScript
tags: [JavaScript, 柯里化]
---

## 介绍
&emsp;&emsp;**柯里化**（英语：Currying），又译为卡瑞化或加里化，是把接受多个参数的函数变换成接受一个单一参数（最初函数的第一个参数）的函数，并且返回接受余下的参数而且返回结果的新函数的技术。可以理解为提前接收部分参数，延迟执行，不立即输出结果，而是返回一个接受剩余参数的函数。因为这样的特性，也被称为部分计算函数。柯里化，是一个逐步接收参数的过程。

## 例子
&emsp;&emsp;有这样一个场景，记录程序员一个月的加班总时间，那么好，我们首先要做的是记录程序员每天加班的时间，然后把一个月中每天的加班的时间相加，就得到了一个月的加班总时间。我们有多种方法可以实现：

### 定义全局变量进行存储
```javascript
var monthTime = 0;

function overtime(time) {
 return monthTime += time;
}

overtime(3.5);    // 第一天
overtime(4.5);    // 第二天
overtime(2.1);    // 第三天
//...

console.log(monthTime);    // 10.1
```

&emsp;&emsp;每次传入加班时间都进行累加，这样当然没问题，但你知道，如果数据量很大的情况下，这样会大大牺牲性能。

### 柯里化实现
&emsp;&emsp;其实我们不必每天都计算加班时间，只需要保存好每天的加班时间，在月底时计算这个月总共的加班时间，所以，其实只需要在月底计算一次就行。
```javascript
var overtime = (function() {
  var args = [];

  return function() {
    if(arguments.length === 0) {
      return args.reduce((a, b) => a + b);
    }else {
      [].push.apply(args, arguments);
    }
  }
})();

overtime(3.5);    // 第一天
overtime(4.5);    // 第二天
overtime(2.1);    // 第三天
//...

console.log( overtime() );    // 10.1
```

&emsp;&emsp;以上例子就是借助柯里化的思想，提前接收每天的加班时间，但延迟进行结果计算，这就节省了每次累加所消耗的性能。

## 作用
### 编写轻松重用和配置的小代码块
&emsp;&emsp;举个例子，比如你有一间士多店并且你想给你优惠的顾客给个10%的折扣（即打九折）：
```javascript
function discount(price, discount) {
    return price * discount
}
```

&emsp;&emsp;当一位优惠的顾客买了一间价值$500的物品，你给他打折：
```javascript
const price = discount(500,0.10); // $50 
// $500  - $50 = $450
```

&emsp;&emsp;你可以预见，从长远来看，我们会发现自己每天都在计算10%的折扣：
```javascript
const price = discount(1500,0.10); // $150
// $1,500 - $150 = $1,350
const price = discount(2000,0.10); // $200
// $2,000 - $200 = $1,800
const price = discount(50,0.10); // $5
// $50 - $5 = $45
const price = discount(5000,0.10); // $500
// $5,000 - $500 = $4,500
const price = discount(300,0.10); // $30
// $300 - $30 = $270

```

&emsp;&emsp;我们可以将discount函数柯里化，这样我们就不用总是每次增加这0.01的折扣。
```javascript
function discount(discount) {
    return (price) => {
        return price * discount;
    }
}
const tenPercentDiscount = discount(0.1);
```

&emsp;&emsp;现在，我们可以只计算你的顾客买的物品都价格了：
```javascript
tenPercentDiscount(500); // $50
// $500 - $50 = $450
```

&emsp;&emsp;同样地，有些优惠顾客比一些优惠顾客更重要-让我们称之为超级客户。并且我们想给这些超级客户提供20%的折扣。 可以使用我们的柯里化的discount函数：
```javascript
const twentyPercentDiscount = discount(0.2);
```

### 避免频繁调用具有相同参数的函数
&emsp;&emsp;举个例子，我们有一个计算圆柱体积的函数
```javascript
function volume(l, w, h) {
    return l * w * h;
}
```

&emsp;&emsp;碰巧仓库所有的气缸高度为100米，你将会看到你将重复调用此函数，h为100米
```javascript
volume(200,30,100) // 2003000l
volume(32,45,100); //144000l
volume(2322,232,100) // 53870400l
```

&emsp;&emsp;要解决以上问题，你可以将volume函数柯里化（像我们之前做的）：
```javascript
function volume(h) {
    return (w) => {
        return (l) => {
            return l * w * h
        }
    }
}
```

&emsp;&emsp;我们可以定义一个专门指定圆柱体高度的的函数：
```javascript
const hCylinderHeight = volume(100);
hCylinderHeight(200)(30); // 600,000l
hCylinderHeight(2322)(232); // 53,870,400l
```

## 通用的柯里化函数
```javascript
function curry(fn, ...args) {
    return (..._arg) => {
        return fn(...args, ..._arg);
    }
}
```

&emsp;&emsp;上面代码做了什么？curry函数接受一个我们想要柯里化的函数（fn）和 一些可变数量的参数（…args）。剩下的操作用于将fn之后的参数数量收集到…args中。然后，返回一个函数，同样地将余下的参数收集为…args。这个函数调用原始函数fn通过使用spread运算符作为参数传入... args和... args，然后，将值返回给使用。现在我们可以用curry函数来创建特定的函数啦。下面我们用curry函数来创建更多计算体检的特定函数（其中一个就是计算高度100米的圆柱体积函数）
```javascript
function volume(l,h,w) {
    return l * h * w
}
const hCy = curry(volume,100);
hCy(200,900); // 18000000l
hCy(70,60); // 420000l
```

## 参考文章
[柯里化](https://zh.wikipedia.org/wiki/%E6%9F%AF%E9%87%8C%E5%8C%96)
[简单理解JavaScript中的柯里化和反柯里化](https://juejin.im/post/58a5879e1b69e6006d1e8748)
