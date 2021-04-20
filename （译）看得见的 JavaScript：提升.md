# （译）看得见的 JavaScript：提升

提升（Hoisting）是每个 JS 开发人员都听说过的那些术语之一，因为你在Google上搜索了烦人的错误并最终出现在StackOverflow上，那里的人告诉你此错误是由于提升引起的。所以，什么是提升？

如果你是一个新手，你可能会遇到“怪异”的行为，其中某些变量是 `undefined`，或是引发了`ReferenceErrors`，等等。提升通常的解释是：把变量和函数放在js文件的顶部。尽管表现起来像是这样，但是其实不是的。

当 JS 引擎获取了我们的脚本，它第一件要做的事是，为我们代码里的数据设置内存（**setting up memory**）。此时，不执行任何代码，它只是在为执行做准备。函数声明和变量存储的方式是不同的。函数被存储为一个指针，它指向整个函数。

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--lLfiCbTX--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://devtolydiahallie.s3-us-west-1.amazonaws.com/gif7.gif)

而变量，有点不一样。ES6 引入了两个新的关键字来声明变量：`let` 和 `const`。用它们来声明的变量存储为 *uninitialized*。

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--vRtKMspn--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://devtolydiahallie.s3-us-west-1.amazonaws.com/gif8.gif)

用 `var` 声明的变量被存储为默认值 `undefined`。

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--zvlaEaAo--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://devtolydiahallie.s3-us-west-1.amazonaws.com/gif9.gif)

现在创建阶段完成，我们实际上可以执行代码了。如果我们有三条 console.log 语句在文件的顶部，它们的位置是在声明函数或者其他变量之前的，看看会发生什么。

因为函数存储为一个指向整个函数代码的指针，所以我们可以在创建它之前的行上调用它！

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--nk1taOke--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://devtolydiahallie.s3-us-west-1.amazonaws.com/gif16.gif)

当我们在声明变量之前，引用一个变量（通过`var`声明的），它会简单地返回一个默认值 `undefined`！但是，它有时候会出现 "unexpected" 的表现。在大多数情况下，这意味着你无意间引用了它（你也许不希望它的值是 undefined）。

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--2nai6XPr--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://devtolydiahallie.s3-us-west-1.amazonaws.com/gif17.gif)

为了防止意外引用`undefined`的变量，当我们试着获取 *uninitialized* 值时，就会抛出 `ReferenceError`的错误。在它们实际声明之前的“区域”被称为暂时的死亡区域（the **temporal dead zone**）：你不可以在它们初始化值之前引用这些变量（这也包括 ES6 中的 class）。

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--VVPlWhGC--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://devtolydiahallie.s3-us-west-1.amazonaws.com/gif18.gif)

当引擎通过实际我们赋值的行时，内存中的值就会被我们实际的赋值覆盖掉。

（下图的 6 应该为 7）

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--LGEaCMkS--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://devtolydiahallie.s3-us-west-1.amazonaws.com/gif12.gif)

------

快速小结：

* 在我们执行代码前，函数和变量会被存储在用于执行上下文的内存中。
* 函数被存储为一个指向整个函数的指针，用`var`声明的变量会有一个`undefined`值，用`let` 和 `const`声明的变量被存储为 *uninitialized*。 

我希望在执行代码时，我们对提升的理解不会那么模糊了。如果还是没有太多的感觉，也不必担心，等你接触多了就对它有感觉了。

**2021.4.20 Translated by Knight**