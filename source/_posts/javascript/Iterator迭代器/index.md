---
title: Iterator迭代器
date: 2019-01-16 20:03:15
categories: JavaScript
tags: [JavaScript, iterator]
---

## Iterator（迭代器）的概念
&emsp;&emsp;迭代器（Iterator）是一种接口，为各种不同的数据结构提供统一的访问机制。任何数据结构只要部署 Iterator 接口，就可以完成遍历操作。

### Iterator 的作用
- 一是为各种数据结构，提供一个统一的、简便的访问接口；
- 二是使得数据结构的成员能够按某种次序排列；
- 三是 ES6 创造了一种新的遍历命令for...of循环，Iterator 接口主要供for...of消费。

### Iterator 的遍历过程
1. 创建一个指针对象，指向当前数据结构的起始位置。也就是说，迭代器对象本质上，就是一个指针对象。
2. 第一次调用指针对象的next方法，可以将指针指向数据结构的第一个成员。
3. 第二次调用指针对象的next方法，指针就指向数据结构的第二个成员。
4. 不断调用指针对象的next方法，直到它指向数据结构的结束位置。

&emsp;&emsp;每一次调用next方法，都会返回数据结构的当前成员的信息。具体来说，就是返回一个包含value和done两个属性的对象。其中，value属性是当前成员的值，done属性是一个布尔值，表示遍历是否结束。如下例所示：
```javascript
var it = makeIterator(['a', 'b']);

it.next() // { value: "a", done: false }
it.next() // { value: "b", done: false }
it.next() // { value: undefined, done: true }

function makeIterator(array) {
  var nextIndex = 0;
  return {
    next: function() {
      return nextIndex < array.length ?
        {value: array[nextIndex++], done: false} :
        {value: undefined, done: true};
    }
  };
}
```

## 默认 Iterator 接口
&emsp;&emsp;Iterator 接口的目的，就是为所有数据结构，提供了一种统一的访问机制，即for...of循环。当使用for...of循环遍历某种数据结构时，该循环会自动去寻找 Iterator 接口。一种数据结构只要部署了 Iterator 接口，我们就称这种数据结构是“可遍历的”（iterable）。
&emsp;&emsp;ES6 规定，默认的 Iterator 接口部署在数据结构的Symbol.iterator属性，或者说，一个数据结构只要具有Symbol.iterator属性，就可以认为是“可遍历的”（iterable）。Symbol.iterator属性本身是一个函数，就是当前数据结构默认的迭代器生成函数。执行这个函数，就会返回一个迭代器。至于属性名Symbol.iterator，它是一个表达式，返回Symbol对象的iterator属性，这是一个预定义好的、类型为 Symbol 的特殊值，所以要放在方括号内。
```javascript
const obj = {
  [Symbol.iterator] : function () {
    return {
      next: function () {
        return {
          value: 1,
          done: true
        };
      }
    };
  }
};
```
&emsp;&emsp;ES6 的有些数据结构原生具备 Iterator 接口（比如数组），即不用任何处理，就可以被for...of循环遍历。原因在于，这些数据结构原生部署了Symbol.iterator属性，另外一些数据结构没有（比如对象）。凡是部署了Symbol.iterator属性的数据结构，就称为部署了迭代器接口。调用这个接口，就会返回一个迭代器对象。
&emsp;&emsp;原生具备 Iterator 接口的数据结构如下：
- Array
- Map
- Set
- String
- TypedArray
- 函数的 arguments 对象
- NodeList 对象

&emsp;&emsp;对象（Object）之所以没有默认部署 Iterator 接口，是因为对象的哪个属性先遍历，哪个属性后遍历是不确定的，需要开发者手动指定。本质上，迭代器是一种线性处理，对于任何非线性的数据结构，部署迭代器接口，就等于部署一种线性转换。不过，严格地说，对象部署迭代器接口并不是很必要，因为这时对象实际上被当作 Map 结构使用，ES5 没有 Map 结构，而 ES6 原生提供了。
&emsp;&emsp;一个对象如果要具备可被for...of循环调用的 Iterator 接口，就必须在Symbol.iterator的属性上部署迭代器生成方法（原型链上的对象具有该方法也可）。
```javascript
class RangeIterator {
  constructor(start, stop) {
    this.value = start;
    this.stop = stop;
  }

  [Symbol.iterator]() { return this; }

  next() {
    var value = this.value;
    if (value < this.stop) {
      this.value++;
      return {done: false, value: value};
    }
    return {done: true, value: undefined};
  }
}

function range(start, stop) {
  return new RangeIterator(start, stop);
}

for (var value of range(0, 3)) {
  console.log(value); // 0, 1, 2
}
```

## 调用 Iterator 接口的场合
### 解构赋值
&emsp;&emsp;对数组和 Set 结构进行解构赋值时，会默认调用Symbol.iterator方法。
```javascript
let set = new Set().add('a').add('b').add('c');

let [x,y] = set;
// x='a'; y='b'

let [first, ...rest] = set;
// first='a'; rest=['b','c'];
```

### 扩展运算符
&emsp;&emsp;扩展运算符（...）也会调用默认的 Iterator 接口。
```javascript
// 例一
var str = 'hello';
[...str] //  ['h','e','l','l','o']

// 例二
let arr = ['b', 'c'];
['a', ...arr, 'd']
// ['a', 'b', 'c', 'd']
```

### yield*
&emsp;&emsp;yield*后面跟的是一个可遍历的结构，它会调用该结构的迭代器接口
```javascript
let generator = function* () {
  yield 1;
  yield* [2,3,4];
  yield 5;
};

var iterator = generator();

iterator.next() // { value: 1, done: false }
iterator.next() // { value: 2, done: false }
iterator.next() // { value: 3, done: false }
iterator.next() // { value: 4, done: false }
iterator.next() // { value: 5, done: false }
iterator.next() // { value: undefined, done: true }
```

### 其他场合
&emsp;&emsp;由于数组的遍历会调用迭代器接口，所以任何接受数组作为参数的场合，其实都调用了迭代器接口
- for...of
- Array.from()
- Map(), Set(), WeakMap(), WeakSet()（比如new Map([['a',1],['b',2]])）
- Promise.all()
- Promise.race()

## 字符串的 Iterator 接口
&emsp;&emsp;字符串是一个类似数组的对象，也原生具有 Iterator 接口。
```javascript
var someString = "hi";
typeof someString[Symbol.iterator]
// "function"

var iterator = someString[Symbol.iterator]();

iterator.next()  // { value: "h", done: false }
iterator.next()  // { value: "i", done: false }
iterator.next()  // { value: undefined, done: true }
```

## 参考文章
阮一峰 《ECMAScript 6 入门》
