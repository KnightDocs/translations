# （译）理解 JavaScript 中的重绘和重排

最近，当我研究为什么React的虚拟DOM是如此之快时，我意识到我们对JS的性能知之甚少。所以，我写这篇文章是为了帮助大家提升对于重绘和重排以及JS性能的认识。

## 在我们深入之前，我们知道浏览器是如何工作的吗？

一幅图胜过千言万语。让我们从更高的角度去了解浏览器的工作原理吧！

![img](https://miro.medium.com/max/500/1*lAUWhHx6CdF_OkMC3YkAHw.png)

emmm……什么是浏览器引擎（**browser engine**）？渲染引擎（**rendering engine**）又是什么？

浏览器引擎的主要工作就是将HTML文档以及一个网页的其他资源转换为用户设备上的可见交互式视图。

除了浏览器引擎，另外还有两个常见的术语：布局引擎（layout engine）、渲染引擎。理论上，布局和渲染（或者叫做绘图）可以通过独立的引擎来处理，然而实际上，它们是紧密耦合却很少单独考虑。

------

## 让我们一起理解浏览器如何在荧幕上画出一个用户界面的。

当你在某个链接或者URL浏览器上按下 Enter 键时，就会向网页发出HTTP请求，相应的服务器会提供HTML文档作为响应。（在这期间发生了很多事。）

![img](https://miro.medium.com/max/638/1*_alTfrxmTCP1mInn4QEOnA.jpeg)

* 浏览器解析出HTML源代码，并构建出一棵 **DOM 树**，其中每个HTML标签在树中都有一个对应的节点（node），并且标签之间的文本块也会获得一个文本节点（text node）的表示形式。DOM树的根节点就是documentElement（也就是`<html>`标签！）
* 浏览器解析CSS代码，让它变得有意义。样式信息是层叠的：基础的规则位于 User-Agent stylesheets（用户代理样式表，也就是浏览器自带的样式），然后有一些 user stylesheets，author (as in the author of the page) stylesheets。它们可以从不同的位置引入，外部的、导入的、行内的、内置的。
* 之后来到有趣的部分——构建一棵渲染树（a **render tree**）。渲染树有点像DOM树，但并不完全匹配。渲染树认识样式，因此如果你用`display:none`去隐藏一个`div`，它不会出现在渲染树上。对于其他不可见的元素也是如此，如`head`以及其中的一切。另一方面，可能有一些DOM元素由多个节点表示，比如`p`元素这个节点有时还会包含文本节点。渲染树中的一个节点被称为一个 frame 框架，或是一个盒子。每一个这样的节点都有CSS 盒模型的特性——width、height、border、margin等等。
* 一旦渲染树被构建完毕，浏览器就会在屏幕上画出渲染树的节点。

浏览器是如何绘制布局，并且试着检测根元素、临近元素以及它的子元素这些节点，然后相应地重新排列布局的呢？

------

让我们看一个例子：

```html
<html>
<head>
  <title>Repaint And Reflow</title>
</head>
<body>
    
  <p>
    <strong>How's The Josh?</strong>
    <strong><b> High Sir...</b></strong>
  </p>
  
  <div style="display: none">
    Nothing to display
  </div>
  
  <div><img src="..." /></div>
  ...
 
</body>
</html>
```

表示这个HTML文档的 DOM 树基本上每个标签都有一个节点，节点之间的每一段文本都有一个文本节点。

```
documentElement (html)
    head
        title
    body
        p
            strong
                [text node]
        p
            strong
                b
                    [text node]    		
        div 
            [text node]
		
        div
            img
		
        ...
```

渲染树将是 DOM 树的可视部分。它会忽略一些东西——`head`以及隐藏的`div`，但它也有额外的节点（框架、盒子）用于文本行。

```
root (RenderView)
    body
        p
            line 1
	          line 2
	          line 3
	          ...
	    
	      div
	          img
	    
	      ...
```

渲染树上的根节点这个盒子里包含了所有其他的元素。你可以将它们看作浏览器窗口的内置部分，因为这是它可以展开的限制范围。

技术上讲，WebKit 把根节点称为 `RenderView`，它对应于CSS的初始包含块（ [initial containing block](http://www.w3.org/TR/CSS21/visudet.html#containing-block-details)），它的范围是从页面顶部 (0, 0) 到 (window.innerWidth`, `window.innerHeight)。

要想在屏幕上准确显示什么以及如何显示，就需要在渲染树中进行递归遍历。

## 重绘和重排

通常情况下，至少有一个初始页面的布局和绘制（除非你就喜欢让你的页面是空白的）。之后，改变输出的信息将会导致出现以下其中一个或两个情况：

1. 渲染树的一部分（或者整棵树）将需要重新验证，并重新计算节点的尺寸。这就是**重排**（reflow, layout）。注意，初始页面至少会出现一次重排。
2. 屏幕上的部分内容将会被更新，原因可能是节点的几何属性（geometric properties）发生了变化，也可能是样式发生了变化，比如背景颜色发生了变化。屏幕的更新称为一次**重绘**（repaint，redraw）。

重绘和重排代价昂贵，它们会损害用户体验，让UI显得迟钝。

### Repaint

顾名思义，重绘就是重新绘制屏幕上的元素，因为元素的外观发生变化会影响一个元素的可见性，但不会对布局产生影响。

例如：

1. 改变一个元素的 visibility 。
2. 改变元素的 outline 。
3. 改变 background。

这些都会触发重绘。

根据 Opera 的说法，重新绘制是一项昂贵的操作，因为它迫使浏览器验证 / 检查所有其他DOM 节点的可见性。

### Reflow

重排意味着为了重新呈现部分或全部文档，它就会重新计算文档中元素的位置和几何特性。冲排会阻碍浏览器中的用户操作，所以这对于开发人员了解如何缩短重排时间以及了解各种文档属性（DOM深度，CSS规则效率，不同类型的样式更改）对重排时间的影响很有用。有时，重排文档中的单个元素可能需要重排其父元素以及紧随其后的所有元素。

------

## 虚拟DOM VS 真实DOM

每一次DOM的改变都会让浏览器重新计算CSS，然后重新布局，重绘整个页面。这是在真实DOM中花费时间的原因。

为了最小化这个时间，Ember 利用 key/value观测技术（key/value observation），Angular 使用脏检查（dirty checking）。利用这种技术，它们只会更新发生变化的DOM，或是在 Augular 中被标记了dirty 的节点。

如果不是这样，那么你将不会在写一封新的电子邮件时就受到另一封新的电子邮件。

但是，今天的浏览器很聪明，它们会缩短重绘的时间。可以做的最大的事情就是最小化和批量处理需要进行重绘的DOM。

减少和批量处理DOM变化，就是React中虚拟DOM的价值所在。

## 什么让React的虚拟DOM如此之快？

React并没有真的去做什么新的改变。它仅仅是策略上的进步。它做了什么呢？它将真实DOM的副本保存再内存中。当你修改DOM时，它首先会将这些DOM变化应用于内存中的DOM，然后利用它的diffing算法， 计算出真正发生改变的节点。

最后，它处理这些改变，并将它们应用于真实DOM，因此最小化了重排和重绘。

**2020.4.29 Translated by Knight**