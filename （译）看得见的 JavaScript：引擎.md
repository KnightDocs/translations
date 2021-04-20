# （译）看得见的 JavaScript：引擎

JavaScript 很酷，但是机器是如何理解你写的代码的呢？作为一个 JS 开发人员，我们通常不需要自己处理编译器。但是，知道JS引擎的基础原理，看看它是如何将JS代码处理为机器能够理解的编码是很重要的！

> 注意：这篇文章主要基于 V8 引擎。（Node.js，Chromium-based browsers）

HTML解析器遇到带有源代码的`<script>`标签。代码从源头加载过来，这些源头可以是 `network`，`cahe`，或是 `service worker`。它们响应是以字节流（**stream of bytes**）的形式请求的脚本，这些字节流由字节流解码器（the byte stream decoder）负责！字节流解码器在下载字节流时对其进行解码。

![Alt Text](https://res.cloudinary.com/practicaldev/image/fetch/s--Xs5OQmGX--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/pv4y4w0doztvmp8ei0ki.gif)

------

字节流解码器从解码后的字节流中创建令牌/标记（**token**）。比如，`0066` 解码为 `f`，`0075` 解码为 `u`，`006e` 解码为 `n`，`0063`解码为`c`，`0074`解码为`t`，`0069`解码为`i`，`006f`解码为`o`，`006e`解码为`n`，每个编码之间用空格隔开。似乎就像你写的 `function`！这是 JS 中的保留字，这样一个标记被创建，然后被送到语法分析器（解析器），进行预解析。剩下的字节流会发生相同的事情。

![Alt Text](https://res.cloudinary.com/practicaldev/image/fetch/s--ID8wDIAy--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/bic727jhzu0i8uep8v0k.gif)

------

引擎使用两个解析器：预解析器（ **pre-parser**），和解析器（**parser**）。为了减少加载网站所需的时间，引擎会试着避免解析那些现在不需要用到的代码。pre-parser 处理以后可能会用到的代码，同时 parser 会处理立即需要用到的代码！如果某个函数只会在用户点击一个按钮时才会触发，就没有必要为了加载网站而立即编译这段代码。只有当用户最终点击了按钮，才需要这段代码，这时才将它送到 parser。

解析器根据它接收到的解码标记来创建节点，有了这些节点，它就创建了一个抽象的语法树（an Abstract Syntax Tree, or AST）。

![Alt Text](https://res.cloudinary.com/practicaldev/image/fetch/s--6IHw1BUH--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/sgr7ih6t7zm2ek28rtg6.gif)

------

接下来，轮到解释器了（**interpreter**）！解释器通过AST，然后根据AST包含的信息产生了字节码（**byte code**），只要字节码全部产生，AST就会被删除，并从内存中清除出去。最后，我们有了机器可以使用的东西！

![Alt Text](https://res.cloudinary.com/practicaldev/image/fetch/s--HlXdsZRx--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/i5f0vmcjnkhireehicyn.gif)

------

尽管字节码已经很快了，但它可以更快。随着字节码的运行，信息正在被产生。它可以检测某些行为是否经常发生，以及所使用的数据类型。也许你已经多次调用了一个函数，那么它就会被优化，然后运行得更快！

字节码以及一起生成的类型反馈会被发送到一个优化编译器（optimizing compiler）中，然后利用这些字节码和类型反馈生成更加优化的机器代码。

![Alt Text](https://res.cloudinary.com/practicaldev/image/fetch/s--gsKbgaq7--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/ongt4qftovd82sp2vihk.gif)

------

JavaScript 是一个动态类型语言，意味着数据的类型能够随意改变。如果 JS 引擎每一次都要去检查一个变量的数据类型是什么，那么代码就会解释得非常慢。

为了减少解释代码的时间，优化的机器码只会处理引擎在运行字节码时看到过的代码。如果我们一遍又一遍得重复使用某一段返回相同数据类型的代码，优化的机器码就会被简单地重复使用以提高速度。然而，JS的数据类型是动态的，相同的代码段会突然返回一个不同类型的数据，如果该情况发生了，机器码会被反优化，引擎会反馈被解释产生的字节码。

假设某一个函数被调用100次，并且总是一直返回相同的值，那么它就会假设第 101 次调用该函数的时候会再次返回一样的值。

比如我们有下面这个函数`sum`， 它每次调用都传进去数字类型的参数：

```javascript
function sum(a, b) {
  return a + b;
}
sum(1, 2)
```

它返回数字 `3` ！下一次调用时，它会假设仍然传递两个数字类型的参数。

如果确实传递数字作为参数，不需要动态查找，那么它仅仅重新利用优化的机器码。否则，如果假设不成立，它会恢复到原始的字节码，而不是优化后的机器码。

举个例子，下一次调用时，我们传了一个字符串而非数字。因为 JS 是动态类型的，所以我们这样做了也不会产生任何错误！

```javascript
function sum(a, b) {
  return a + b;
}
sum('1', 2);
```

这就意味着，数字 `2`将转换为一个字符串，然后函数返回字符串 `"12"`。它会回去执行解释过的字节码，并且更新类型反馈。

**2021.4.20 Translated by Knight**