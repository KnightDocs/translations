# （译）学习 JavaScript 基础 - 全局作用域

如果你把JS当做一款游戏，并打算像我一样成为一名专业的玩家，你就需要完全理解它的规则。为了帮助你，我打算写一系列关于JS中最重要的几个概念的文章。作用域，提升以及闭包。这篇文章将会介绍**全局作用域**。（Global Scope）

在进入正题之前，让我们定义一下什么是作用域（Scope），它为什么那样重要？作用域（Scope）是一个定义变量可以在哪里被获取到（ *看到，找到，获取到* ）的规则。（*Scopes are the way to define where the variables are accessible.*）如果你理解作用域，不管何时或是何地，你都可以获取到你想要获取的变量和函数，这是你在JS中应有的控制能力。定义可能不是特别清晰，但随着我们的浏览，这些文章将会让它更加明了。在JS中，有两种类型的作用域，分别是全局作用域（Global Scope）和局部作用域（Local Scope）。

我认为对于全局作用域，MDN Web Docs 给了最好的解释：

> 在编程环境中，全局作用域是一个**范围**，这个范围包含其他所有的作用域，并且可以被其他所有的作用域看到。在客户端JS中，全局作用域就是整个网页，里面的所有代码都正在被执行。

这是什么意思呢？就是说如果你在全局作用域里定义了一个变量，那么你就可以在所有作用域中获取到这个变量。所有在全局对象中创建的变量和函数都将成为全局对象（Global Object）的一部分。这个全局对象会随着JS运行的环境不同而发生变化。在 Node.js 中，全局对象等于 **global**，然而在浏览器中则是 **window** 。

为了检查浏览器中的 window 对象，简单地在控制台中写下 window，你就能看到它已经包含有很多东西了。被内置在其中的都是可以看到的，有一些你可能会很熟悉。window 对象不仅保存着你在全局范围内定义的变量，而且还有很多内置的属性和方法，以及更多的东西。甚至我们经常使用的 **document** 对象都是 window 对象的一部分。

Window Object 和 Document Object 很容易混淆，其中 Document Object 更小一点。它代表 DOM，DOM是HTML的根元素。Window Object 则完全不同，它持有很多信息，比如 window.history，window.location 以及 window.document 。类似的不同可以在 BOM （浏览器对象模型）和 DOM （网页文档对象模型）之间找到。

BOM 是*JS 与浏览器沟通的途径*。它拥有很多对象，比如 screen、history、navigation、location 以及 document。DOM 是所有 HTML 文档（HTML document）的象征，它是 BOM 的一部分。

Window Object 是最顶级的对象，它至高无上，你不能创建比它还要高级的作用域了。它是浏览器窗口的象征，网页在其中展示着（译者说：网页就是 document ）。Window Object 是被浏览器创建的，它是一个浏览器的对象而非 JavaScript 的对象。Window Object 掌握所有内置其中的属性、方法，也拥有我们在全局中创建的那些属性和方法。如果你想测试一下，那就打开控制台写一个全局的函数：

```javascript
function forTesting(){
  console.log(‘I am a global function’);
}
```

现在，在控制台中输入 window 。滚动到 F 的位置，你就能看到此时 Window Object 就包含着你刚刚写的全局函数了。另一个测试的办法是，你可以像这样去调用你的函数：

```javascript
forTesting()
// I am a global function
window.forTesting()
// I am a global function
```

如你所见，结果一样。

为了解释 Window Object 如何运作，你可以打开或是关闭它。现在打开控制台，然后输入：

```javascript
window.open()
```

然后在打开的窗口中的控制台里，输入：

```javascript
window.close()
```

你也可以使用 Window Object 中的一些方法来做非常酷炫的事情。

```javascript
screen.width // returns the width of the screen
screen.height // returns the height of the screen
location.href // returns the URL
location.hostname // returns the domain name
history.back() // goes back in the browser
history.forward() // goes forward in the browser
alert() // display alert box
prompt() // display dialog box
```

**2021.4.13 Knight**