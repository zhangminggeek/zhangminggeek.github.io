---
title: JavaScript的执行机制
date: 2018-11-18 16:52:44
categories: JavaScript
tags: [JavaScript, Event Loop, 前端]
---

## 关于JavaScript
javascript是一门单线程语言，是按照语句出现的顺序执行的。虽然在最新的HTML5中提出了Web-Worker，但javascript是单线程这一核心仍未改变。所以一切javascript版的"多线程"都是用单线程模拟出来的。

## JavaScript事件循环
如果JavaScript中不存在异步，只能自上而下执行，如果上一行解析时间很长，那么下面的代码就会被阻塞。对于用户而言，阻塞就意味着 "卡死"，这样就导致了很差的用户体验。所以，JavaScript中需要异步执行。
因此JavaScript的任务被分为两类：
- 同步任务
- 异步任务

当我们打开网站时，网页的渲染过程就是一大堆同步任务，比如页面骨架和页面元素的渲染。而像加载图片音乐之类占用资源大耗时久的任务，就是异步任务。关于这部分有严格的文字定义，但本文的目的是用最小的学习成本彻底弄懂执行机制，所以我们用导图来说明：
{% asset_img images/15fdd88994142347.png %}


- 同步和异步任务分别进入不同的执行"场所"，同步的进入主线程，异步的进入Event Table并注册函数。
- 当指定的事情完成时，Event Table会将这个函数移入Event Queue。
- 主线程内的任务执行完毕为空，会去Event Queue读取对应的函数，进入主线程执行。
- 上述过程会不断重复，也就是常说的Event Loop(事件循环)。

那么如何判断主线程执行栈为空啊？js引擎存在monitoring process进程，会持续不断的检查主线程执行栈是否为空，一旦为空，就会去Event Queue那里检查是否有等待被调用的函数。
```javascript
let data = [];
$.ajax({
    url:www.javascript.com,
    data:data,
    success:() => {
        console.log('发送成功!');
    }
})
console.log('代码执行结束');
```
上面是一段简易的ajax请求代码：
- ajax进入Event Table，注册回调函数success。
- 执行console.log('代码执行结束')。
- ajax事件完成，回调函数success进入Event Queue。
- 主线程从Event Queue读取回调函数success并执行。

## 定时器setTimeout和setInterval
### setTimeout
定时器指定某些代码在多少时间之后执行这叫做”定时器”（timer）功能，也就是定时执行的代码。 
```javascript
setTimeout(() => {
    task();
},3000)
console.log('执行console');
```

由于setTimeout()函数异步执行，应该先执行console.log()，因此上述代码的执行结果为：
```javascript
//执行console
//task()
```

然后我们修改一下前面的代码：
```javascript
setTimeout(() => {
    task()
},3000)

sleep(10000000)
```

把这段代码在chrome执行一下，却发现控制台执行task()需要的时间远远超过3秒。事实上，上述代码真实的执行顺序应该上：
- task()进入Event Table并注册,计时开始。
- 执行sleep函数，很慢，非常慢，计时仍在继续。
- 3秒到了，计时事件timeout完成，task()进入Event Queue，但是sleep还没执行完，只好等着。
- sleep终于执行完了，task()终于从Event Queue进入了主线程执行。

我们一般认为，3秒后会执行setTimeout里的那个函数，其实这种说法并不严谨，准确的解释是：3秒后，setTimeout里的函数会被推入事件队列(Event Loop)，而事件队列(Event Loop)里的任务，只有在主线程空闲时才会执行，所以条件只有同时满足(ps:3秒后并且主线程空闲)时，才会3秒后执行函数

### setInterval
setInterval和setTimeout类似，只不过setInterval是循环执行的。对于执行顺序来说，setInterval会每隔指定的时间将注册的函数置入Event Queue，如果前面的任务耗时太久，那么同样需要等待。
唯一需要注意的一点是，对于setInterval(fn,ms)来说，我们已经知道不是每过ms秒会执行一次fn，而是每过ms秒，会有fn进入Event Queue。一旦setInterval的回调函数fn执行时间超过了延迟时间ms，那么就完全看不出来有时间间隔了。

## Promise与process.nextTick(callback)
所谓Promise，简单说就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。从语法上说，Promise 是一个对象，从它可以获取异步操作的消息。Promise 提供统一的 API，各种异步操作都可以用同样的方法进行处理。具体内容可以参考阮一峰老师的[Promise](http://es6.ruanyifeng.com/#docs/promise)。
而process.nextTick(callback)类似node.js版的"setTimeout"，在事件循环的下一次循环中调用 callback 回调函数。

除了广义的同步任务和异步任务，我们对任务有更精细的定义：
- macro-task(宏任务)：包括整体代码script，setTimeout，setInterval
- micro-task(微任务)：Promise，process.nextTick

{% asset_img images/16589b575216a917.png %}


不同类型的任务会进入对应的Event Queue，比如setTimeout和setInterval会进入相同的Event Queue。
事件循环的顺序，决定js代码的执行顺序。进入整体代码(宏任务)后，开始第一次循环。接着执行所有的微任务。然后再次从宏任务开始，找到其中一个任务队列执行完毕，再执行所有的微任务。
```javascript
setTimeout(function() {
    console.log('setTimeout');
})

new Promise(function(resolve) {
    console.log('promise');
}).then(function() {
    console.log('then');
})

console.log('console');
```

- 这段代码作为宏任务，进入主线程。
- 先遇到setTimeout，那么将其回调函数注册后分发到宏任务Event Queue。
- 接下来遇到了Promise，new Promise立即执行，then函数分发到微任务Event Queue。
- 遇到console.log()，立即执行。
- 整体代码script作为第一个宏任务执行结束，看看有哪些微任务？我们发现了then在微任务Event Queue里面，执行。
- 第一轮事件循环结束了，我们开始第二轮循环，当然要从宏任务Event Queue开始。我们发现了宏任务Event Queue中setTimeout对应的回调函数，立即执行。
结束。

事件循环，宏任务，微任务的关系如图所示：
{% asset_img images/15fdcea13361a1ec.png %}


## Example
```javascript
console.log('1');
// 记作 set1
setTimeout(function () {
    console.log('2');
    // set4
    setTimeout(function() {
        console.log('3');
    });
    // pro2
    new Promise(function (resolve) {
        console.log('4');
        resolve();
    }).then(function () {
        console.log('5')
    })
})

// 记作 pro1
new Promise(function (resolve) {
    console.log('6');
    resolve();
}).then(function () {
    console.log('7');
    // set3
    setTimeout(function() {
        console.log('8');
    });
})

// 记作 set2
setTimeout(function () {
    console.log('9');
    // 记作 pro3
    new Promise(function (resolve) {
        console.log('10');
        resolve();
    }).then(function () {
        console.log('11');
    })
})
```

- 第一轮事件循环：
1. 整体script作为第一个宏任务进入主线程，遇到console.log，输出1。
{% asset_img images/16589b8d4316baf3.png %}


2. 遇到set1，其回调函数被分发到宏任务Event Queue中。
{% asset_img images/16589b8f1ccba52c.png %}


3. 遇到pro1，new Promise直接执行，输出6。then被分发到微任务Event Queue中。
{% asset_img images/16589b9381cfd5e0.png %}


4. 遇到了set2，其回调函数被分发到宏任务Event Queue中。
{% asset_img images/16589b9647b2e171.png %}


5. 主线程的整段js代码（宏任务）执行完，开始清空所有微任务；主线程执行微任务pro1，输出7；遇到set3，注册回调函数。
{% asset_img images/16589ba43caf6b9f.png %}


- 第二轮事件循环：
1. 主线程执行队列中第一个宏任务set1，输出2；代码中遇到了set4，注册回调；又遇到了pro2，new promise()直接执行输出4，并注册回调；
{% asset_img images/16589ba8bbbe9c47.png %}


2. set1宏任务执行完毕，开始清空微任务，主线程执行微任务pro2，输出5。
{% asset_img images/16589bab583e4850.png %}


- 第三轮事件循环：
1. 主线程执行队列中第一个宏任务set2，输出9；代码中遇到了pro3，new promise()直接输出10，并注册回调；
2. set2宏任务执行完毕，开始情况微任务，主线程执行微任务pro3，输出11。

- 类似循环

所以最后输出结果为1、6、7、2、4、5、9、10、11、8、3。

## 参考文章
[这一次，彻底弄懂 JavaScript 执行机制](https://juejin.im/post/59e85eebf265da430d571f89)
[JavaScript中的JS引擎的执行机制](https://juejin.im/post/5a61a6786fb9a01cc026522c)
[图解JS执行机制](https://juejin.im/post/5b879a9f6fb9a01a0f24a5e1)