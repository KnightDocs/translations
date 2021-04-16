# （译）理解 JavaScript 中的局部作用域

在之前的文章中，我们深入了解了全局作用域，如果你对它没什么把握或是仅仅很快地瞥了一眼，那我建议你最好先去读一读，因为这边文章将会提到全局作用域。

如果你曾经写过 JS 代码，那你可能已经通过函数创建了一个局部作用域了。当你在函数中定义了一个变量，你就已经创建了一个局部作用域了，而不是全局作用域。

> 当你在函数中创建了一个变量的时候，局部作用域就出现了。在那之后，这个变量只允许在该函数内部看到和使用它。

![img](https://miro.medium.com/max/899/1*6kTeSUP0V2UIgOmavp-TDg.png)

任何在黄盒子中创建的变量都是局部变量（ local variable），任何在蓝盒子中创建的变量都是全局变量（global variable）。为了便于理解，蓝色盒子代表全局作用域，黄色盒子代表局部作用域。

当然，仅仅是用一幅图一段描述就能理解局部变量和局部作用域是不可能的。让我们看另一幅图。

![img](https://miro.medium.com/max/708/1*NydrUTYskUaM1y8HQVenNw.png)

局部变量不同于全局变量，而且理解上更加复杂了。这个示例展示了局部作用域的基本表现或概念。如果你创建了不同的函数，那么实际上你也就为它们创建了不同的作用域。为了理解一般情况下它如何表现，我们在控制台中去看看实际的效果，也看看不同作用域下的变量报错和获取的情况。

```javascript
var globalOne=16;
function testingScope(){
  var localOne=12;
  console.log(localOne); // 12
  console.log(globalOne); // 16
}
testingScope() // 12 and 16
console.log(localOne); // Uncaught ReferenceError: localOne is not defined
console.log(globalOne); // 16
```

因为变量```globalOne```是一个全局变量，所以我们可以在任何地方获取它。但是，```localOne```这个变量只能在它自己的作用域中被找到。这就是为什么如果我们调用```testingScope```函数后，可以在控制台中看到那两个变量的原因。

在全局作用域中，我们只能找到保存在window对象中的全局变量。而在局部作用域中，一旦作用域结束，局部变量就无法获取了，并且还会被销毁。这些局部变量只有在函数被调用时会呆在内存里，然后一旦函数执行完毕，它们就会消失。这是全局变量和局部变量之间不同点的关键之一。

## 函数作用域和块级作用域之间的不同

到现在为止，我们已经看过了局部作用域，但实际上我还是理解得不深刻。事实上，局部作用域可以被分成两种：函数作用域和块级作用域（Function Scope and Block Scope）。在ES6之前，JS仅仅只有函数作用域，就是之前我们解释的那些。

*当在函数中创建一个变量时，函数作用域出现了。*

非常简单吧。另一方面，块级作用域是在ES6中引入的，它是有一点不同的。你可以用块级作用域创建一个函数作用域的子集。

如果你在```{}```之间使用代码块，比如```if```、```switch```、```for```、```while```，只有在你使用 *let* 和 *const* 定义变量时才能产生块级作用域。这段描述很长，但是我认为如果在控制台中解释它们会更好。

```javascript
if('anything returning true'){
  let localVariable=12;
  var otherLocalVariable=10;
  console.log(localVariable); // 12
  console.log(otherLocalVariable); // 10
}
console.log(localVariable) // Uncaught ReferenceError: localVariable is not defined
console.log(otherLocalVariable) // 10
```

你看到他们的表现了么？```localvariable```是用```let```定义的，只能在块中看到。```otherLocalVariable```是用```var```定义的，可以在块的外部看到。这两个变量发生不同的表现，原因在于```var```、```let```、```const```是不同的。因此，知道它们之间的不同对于理解块级作用域是非常重要的。

![img](https://miro.medium.com/max/845/1*ILpUdflpCK38UTesUHphxg.png)

> 译者注：
>
> Function Scope：函数作用域
>
> Block Scope：块级作用域
>
> Reassignment：可重新赋值（绿色表示可以，红色表示禁止）
>
> Redeclaration：可重新定义（绿色表示可以，红色表示禁止）

##  **Variable Shadowing**

If you are going to reassign a variable in a function, there is a concept you should know about: Variable Shadowing.

In JavaScript, when you define a variable in the global scope and then in local scope, the local one takes precedence. Shadowing only happens when they share the same name. In the global scope, the variable retains its initial value, but in local scope, its value changes temporarily to whatever you reassigned. Let’s see an example:

```javascript
let simpleVariable='global scope'
function testingVariableShadowing(){
  simpleVariable='local scope'; // reassignment
  console.log(simpleVariable);
}
testingVariableShadowing(); // local scope
console.log(simpleVariable); // global scope
```

In global scope, `**simpleVariable**`**’s** value is still global scope while when it is called in function scope, it changes to local scope.



**译者说：Variable Shadowing 这一部分就不翻译了，因为原文是有点问题的。**

Variable Shadowing 发生在什么时候呢？当局部作用域有一个变量，这个变量名字和全局作用域或者是外部作用域的一个变量**同名**的话，就会发生 Variable Shadowing 。举个例子：

```javascript
var currencySymbol = "$";

function showMoney(amount) {
  var currencySymbol = "€";
  console.log(currencySymbol + amount);
}

showMoney("100"); // €100
```

这里```currencySymbol```变量显然就触发了 Variable Shadowing 。在局部作用域中，会使用当前作用域下的```currencySymbol```变量，和外部的那个不相干。

**2021.4.13 Translated by Knight**