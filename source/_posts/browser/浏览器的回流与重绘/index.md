---
title: 浏览器的回流与重绘
date: 2018-12-12 22:37:55
categories: browser
tags: [回流, 重绘, 前端]
---

## 浏览器的渲染过程

{% asset_img images/16798b8db54caa31.png %}

从上面这个图上，我们可以看到，浏览器渲染过程如下：
1. 浏览器使用流式布局模型 (Flow Based Layout)
2. 解析HTML，生成DOM树，解析CSS，生成CSSOM树
3. 将DOM树和CSSOM树结合，生成渲染树(Render Tree)
4. Layout(回流):根据生成的渲染树，进行回流(Layout)，得到节点的几何信息（位置，大小）
5. Painting(重绘):根据渲染树以及回流得到的几何信息，得到节点的绝对像素
6. 由于浏览器使用流式布局，对Render Tree的计算通常只需要遍历一次就可以完成，但table及其内部元素除外，他们可能需要多次计算，通常要花3倍于同等元素的时间，这也是为什么要避免使用table布局的原因之一。

### 生成渲染数

{% asset_img images/16798b8d839a7d6d.png %}

为构建渲染树，浏览器大体上完成了下列工作：
1. 从 DOM 树的根节点开始遍历每个可见节点。某些节点不可见（例如脚本标记、元标记等），因为它们不会体现在渲染输出中，所以会被忽略。某些节点通过 CSS 隐藏，因此在渲染树中也会被忽略，例如，上例中的 span 节点---不会出现在渲染树中，---因为有一个显式规则在该节点上设置了“display: none”属性。
2. 对于每个可见节点，为其找到适配的 CSSOM 规则并应用它们。
3. 发射可见节点，连同其内容和计算的样式。

>请注意 visibility: hidden 与 display: none 是不一样的。前者隐藏元素，但元素仍占据着布局空间（即将其渲染成一个空框），而后者 (display: none) 将元素从渲染树中完全移除，元素既不可见，也不是布局的组成部分。

最终输出的渲染同时包含了屏幕上的所有可见内容及其样式信息。有了渲染树，我们就可以进入“布局”阶段。

### 回流 (Reflow)
到目前为止，我们计算了哪些节点应该是可见的以及它们的计算样式，但我们尚未计算它们在设备视口内的确切位置和大小---这就是“布局”阶段。当Render Tree中部分或全部元素的尺寸、结构、或某些属性发生改变时，浏览器重新渲染部分或全部文档的过程称为回流。

为弄清每个对象在网页上的确切大小和位置，浏览器从渲染树的根节点开始进行遍历。让我们考虑下面这样一个简单的实例：

```html
<!DOCTYPE html>
<html>
  <head>
    <meta name="viewport" content="width=device-width,initial-scale=1">
    <title>Critial Path: Hello world!</title>
  </head>
  <body>
    <div style="width: 50%">
      <div style="width: 50%">Hello world!</div>
    </div>
  </body>
</html>
```

以上网页的正文包含两个嵌套 div：第一个（父）div 将节点的显示尺寸设置为视口宽度的 50%，---父 div 包含的第二个 div---将其宽度设置为其父项的 50%；即视口宽度的 25%。

布局流程的输出是一个“盒模型”，它会精确地捕获每个元素在视口内的确切位置和尺寸：所有相对测量值都转换为屏幕上的绝对像素。

最后，既然我们知道了哪些节点可见、它们的计算样式以及几何信息，我们终于可以将这些信息传递给最后一个阶段：将渲染树中的每个节点转换成屏幕上的实际像素。这一步通常称为“绘制”或“栅格化”。

{% asset_img images/layout-viewport.png %}

会导致回流的操作：
- 页面首次渲染
- 浏览器窗口大小发生改变
- 元素尺寸或位置发生改变
- 元素内容变化（文字数量或图片大小等等）
- 元素字体大小变化
- 添加或者删除可见的DOM元素
- 激活CSS伪类（例如：:hover）
- 查询某些属性或调用某些方法

### 重绘 (Repaint)
当页面中元素样式的改变并不影响它在文档流中的位置时（例如：color、background-color、visibility等），浏览器会将新样式赋予给元素并重新绘制它，这个过程称为重绘。

## 性能影响
回流比重绘的代价要更高。有时即使仅仅回流一个单一的元素，它的父元素以及任何跟随它的元素也会产生回流。

**回流必将引起重绘，重绘不一定会引起回流。**

## 浏览器的优化机制
现代的浏览器都是很聪明的，由于每次重排都会造成额外的计算消耗，因此大多数浏览器都会通过队列化修改并批量执行来优化重排过程。浏览器会将修改操作放入到队列里，直到过了一段时间或者操作达到了一个阈值，才清空队列。但是！当你获取布局信息的操作的时候，会强制队列刷新，比如当你访问以下属性或者使用以下方法：

- offsetTop、offsetLeft、offsetWidth、offsetHeight
- scrollTop、scrollLeft、scrollWidth、scrollHeight
- clientTop、clientLeft、clientWidth、clientHeight
- getComputedStyle()
- getBoundingClientRect

## 减少回流和重绘
### 最小化重绘和重排
由于重绘和重排可能代价比较昂贵，因此最好就是可以减少它的发生次数。为了减少发生次数，我们可以合并多次对DOM和样式的修改，然后一次处理掉。考虑这个例子

```javascript
const el = document.getElementById('test');
el.style.padding = '5px';
el.style.borderLeft = '1px';
el.style.borderRight = '2px';
```

例子中，有三个样式属性被修改了，每一个都会影响元素的几何结构，引起回流。当然，大部分现代浏览器都对其做了优化，因此，只会触发一次重排。但是如果在旧版的浏览器或者在上面代码执行的时候，有其他代码访问了布局信息(上文中的会触发回流的布局信息)，那么就会导致三次重排。

因此，我们可以合并所有的改变然后依次处理，比如我们可以采取以下的方式：

- 使用cssText
```javascript
const el = document.getElementById('test');
el.style.cssText += 'border-left: 1px; border-right: 2px; padding: 5px;';
```
- 修改CSS的class
```javascript
const el = document.getElementById('test');
el.className += ' active';
```

### 批量修改DOM
当我们需要对DOM对一系列修改的时候，可以通过以下步骤减少回流重绘次数：
1. 使元素脱离文档流
2. 对其进行多次修改
3. 将元素带回到文档中

该过程的第一步和第三步可能会引起回流，但是经过第一步之后，对DOM的所有修改都不会引起回流，因为它已经不在渲染树了。

有三种方式可以让DOM脱离文档流：

- 隐藏元素，应用修改，重新显示
- 使用文档片段(document fragment)在当前DOM之外构建一个子树，再把它拷贝回文档。
- 将原始元素拷贝到一个脱离文档的节点中，修改节点后，再替换原始的元素。

考虑我们要执行一段批量插入节点的代码：

```javascript
function appendDataToElement(appendToElement, data) {
    let li;
    for (let i = 0; i < data.length; i++) {
    	li = document.createElement('li');
        li.textContent = 'text';
        appendToElement.appendChild(li);
    }
}

const ul = document.getElementById('list');
appendDataToElement(ul, data);
```

如果我们直接这样执行的话，由于每次循环都会插入一个新的节点，会导致浏览器回流一次。

#### 隐藏元素，应用修改，重新显示
这个会在展示和隐藏节点的时候，产生两次重绘
```javascript
function appendDataToElement(appendToElement, data) {
    let li;
    for (let i = 0; i < data.length; i++) {
    	li = document.createElement('li');
        li.textContent = 'text';
        appendToElement.appendChild(li);
    }
}
const ul = document.getElementById('list');
ul.style.display = 'none';
appendDataToElement(ul, data);
ul.style.display = 'block';
```

#### 使用文档片段(document fragment)在当前DOM之外构建一个子树，再把它拷贝回文档
```javascript
const ul = document.getElementById('list');
const fragment = document.createDocumentFragment();
appendDataToElement(fragment, data);
ul.appendChild(fragment);
```

#### 将原始元素拷贝到一个脱离文档的节点中，修改节点后，再替换原始的元素。
```javascript
const ul = document.getElementById('list');
const clone = ul.cloneNode(true);
appendDataToElement(clone, data);
ul.parentNode.replaceChild(clone, ul);
```

### 避免触发同步布局事件
上文我们说过，当我们访问元素的一些属性的时候，会导致浏览器强制清空队列，进行强制同步布局。举个例子，比如说我们想将一个p标签数组的宽度赋值为一个元素的宽度，我们可能写出这样的代码：
```javascript
function initP() {
    for (let i = 0; i < paragraphs.length; i++) {
        paragraphs[i].style.width = box.offsetWidth + 'px';
    }
}
```

这段代码看上去是没有什么问题，可是其实会造成很大的性能问题。在每次循环的时候，都读取了box的一个offsetWidth属性值，然后利用它来更新p标签的width属性。这就导致了每一次循环的时候，浏览器都必须先使上一次循环中的样式更新操作生效，才能响应本次循环的样式读取操作。每一次循环都会强制浏览器刷新队列。我们可以优化为:
```javascript
const width = box.offsetWidth;
function initP() {
    for (let i = 0; i < paragraphs.length; i++) {
        paragraphs[i].style.width = width + 'px';
    }
}
```

### 对于复杂动画效果,使用绝对定位让其脱离文档流
对于复杂动画效果，由于会经常的引起回流重绘，因此，我们可以使用绝对定位，让它脱离文档流。否则会引起父元素以及后续元素频繁的回流。

### css3硬件加速（GPU加速）
使用css3硬件加速，可以让transform、opacity、filters这些动画不会引起回流重绘 。但是对于动画的其它属性，比如background-color这些，还是会引起回流重绘的，不过它还是可以提升这些动画的性能。
