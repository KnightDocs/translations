# （译）看得见的 JavaScript：Promise & Async/Await

是否曾经在处理JS时，遇到过它并非按照你想要的那样执行呢？函数的执行似乎是随机的、不可预测的，或者执行被延迟了。现在你可能正在使用 ES6 引入的新特性：**Promise**。

我多年来的好奇心已经得到了满足，我的不眠之夜又一次给我机会做一些动画以解释这个概念。是时候谈论 Promise 了：为什么你要用它们，它们背后的工作原理是怎样的，以及我们如何以现代的方式书写它们？

## 介绍

当写JS时，我们经常要去处理一些依赖于其他任务的任务！假设我们要获取图像，对其进行压缩，应用滤镜并保存它。

第一件需要做的事就是获取我们想要编辑的图像。`getImage`函数可以处理这个问题！只有在成功加载图像后，我们才能够将该值传递给`resizeImage`函数。当已经成功地调整完图像大小时，我们想要使用`applyFilter`函数来对图像做滤镜处理。当图像被压缩并且添加滤镜之后，我们想要保存它，并且让用户知道每一步都正确地执行了。

最后，我们将会得到这样的结果：

![img](https://res.cloudinary.com/practicaldev/image/fetch/s---Kv6sJn7--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/ixceqsql5hpdq8txx43s.png)

emmmm……注意到什么了吗？尽管它没啥问题，但是不够好。我们用了太多的嵌套函数，而这些嵌套函数又取决于之前的回调函数。这通常被称为回调地狱（callback hell），因为我们最终得到了大量嵌套的回调函数，这些代码读起来非常困难。

幸运的是，我们现在有一种叫做**承诺（promise）**的东西来帮助我们。让我们看看什么是承诺吧，并且看看在像回调地狱的情况下，它是如何帮助我们的！

## Promise 语法

ES6 引入了 **Promise**，在许多教程中，我们会看到：

> "A promise is a placeholder for a value that can either resolve or reject at some time in the future"
>
> 一个 promise 是一个值的占位符，这个值会在将来的某些时候被解析或者拒绝。

没错……那种解释从没有让我搞清楚 promise 的概念。实际上，这只让我觉得 Promise 是一种奇怪、模糊、不可预测的魔法。因此，让我们看看到底什么是真正的 promise 。

我们能够利用`Promise`构造函数来创建一个 promise，这个构造函数里接收一个回调函数。试试吧。

![Alt Text](https://res.cloudinary.com/practicaldev/image/fetch/s--phTVdCKA--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/79zi452hphe7ecylhozy.gif)

```javascript
new Promise(() => {});
```

等等，它会返回什么？

一个`promise`是一个对象，它包含一个**状态（state [[PromiseState]]）**，和一个**值（value [[PromiseResult]]）**。上面的例子中，你可以看到`[[PromiseState]]`是`"pending"`，而这个 promise 的值是`undefined`。（译者说：原文是有一些错误的，译文以作更改。）

不用担心，你不用和这个对象做交互，你甚至不必获取`[[PromiseState]]`和`[[PromiseResult]]`属性！但是，这些属性值在使用 promise 的时候是非常重要的。

------

`[[PromiseState]]`的值，也就是承诺的**状态**，是以下三种值中的一种：

* ✅ `fulfilled`：承诺被解析（The promise has been `resolved`. ）。岁月静好，遵守承诺。
* ❌ `rejected`：承诺被拒绝（The promise has been `rejected`. ）。真是太难了，有东西出错了……
* ⏳ `pending`：承诺结果正在等待中，要么是被解析，要么是被拒绝。（the promise is still `pending`）

好吧，这一切听起来很棒，但是一个 promise 的状态啥时候是"pending"，啥时候是"fulfilled"，啥时候又是"rejected"呢？而且，为啥它的状态那么重要呢？

在上面的例子中，我们只是将简单的回调函数传递给 `Promise` 构造函数，然而，回调函数实际上接收 2 个参数。第一个参数的值通常叫做 `resolve` 或者 `res`，这是当 Promise 被**解析**时调用的方法。第二个参数的值通常叫做 `reject`（或者是 rejected） 或者 `rej`，这是当 Promise 被**拒绝**时调用的方法，此时一些东西出错了。

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--9A_mOYMP--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/duen4peq0bdr55cka5ya.png)

让我们尝试一下调用`resolve`或者`reject`方法吧，看看会得到什么！在我的举例中，我把`resolve`方法称为`res`，把`reject`方法称为`rej`。

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--qKIq-sYt--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/z0b9v0h7aiq073l5tl2l.gif)

```javascript
new Promise((res, rej) => res("Yay!"));
new Promise((res, rej) => rej("Aww no!"));
```

太棒了！我们最后知道了如何摆脱`"pending"`状态以及`undefined`结果值了！如果我们调用`resolve`方法，promise 的**状态（state）**就是`"fulfilled"`，如果我们调用`rejected`方法，状态就是`"rejected"`。

一个promise的值，也就是`[[PromiseResult]]`，我们会把它作为参数传递给`resolved`或者`rejected`方法中的一个。

------

好了，现在我们稍微有点能够控制`Promise`对象的能力了。但是它是用来干什么的呢？

在介绍部分，我举了一个图像的例子，获取图像、压缩、添加滤镜，然后保存！最后，我们以一大堆嵌套的回调函数告终。

幸运的是，Promise 能帮助我们解决这个问题！首先，让我们重写整个代码块，使每一个函数都返回一个 `Promise`。

如果图像被成功加载，一切顺利，那就**解析**那副图像的 promise！否则，如果在读取文件时发生了一些错误，那就**拒绝**发生错误的 promise。

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--r9xngcNz--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/iebp0rzfnfqsrmmjplme.png)

```javascript
function getImage(file) {
  return new Promise((res, rej) => {
    try {
      const data = readFile(file);
      resolve(data);
    } catch (err) {
      reject(new Error(err));
    }
  })
}
```

运行终端，看看发生了什么吧！

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--uERkfSWf--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/wsu5nn26dp4elcwh764m.gif)

酷！正如我们所期待的那样，一个 promise 被返回出来了，它的值是一个被解析的 data（the parsed data）。

但是……现在？我们不关心整个 promise 对象，我们只在乎 data 里的值！还好有一些内置的方法可以获取一个 promise 的值，我们关注以下三个方法：

* `.then()`：在一个承诺被解析时调用。（Gets called after a promise *resolved*.）
* `.catch()`：在一个承诺被拒绝时调用。（Gets called after a promise *rejected*.）
* `.finally()`：不管承诺被解析还是被拒绝都会被调用。（*Always* gets called, whether the promise resolved or rejected.）

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--19tIvFJQ--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/mu1aqqnyfjsfon5hwrtw.png)

```javascript
getImage(file)
	.then(image => console.log(image))
	.catch(error => console.log(error))
	.finally(() => console.log("All done!"))
```

`.then`方法接收传递给`resolve`方法的值。

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--DZld0c-0--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/11vxhn9cun7stpjbdi80.gif)

`.catch`方法接收传递给`rejected`方法的值。

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--e9SZHcPk--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/v5y24jz4u89flazvdyn4.gif)

最后，我们通过 promise 获得了被成功解析的值，而非整个 promise 对象。现在，我们就能对这个值为所欲为了。

------

顺便提一下（FYI），当你知道一个 promise 总是会被解析或者被拒绝的话，你就可以写 `Promise.resolve` 或者 `Promise.reject`，来使用你想要解析或者拒绝的承诺。

![Alt Text](https://res.cloudinary.com/practicaldev/image/fetch/s--61Gva3Ze--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/90hxwjfadzslvdbkr4l8.png)

```javascript
new Promise(res => res('Yay!'));
// Promise.reslove('Yay!');

new Promise((res, rej) => rej('Aww no!'));
// Promise.reject('Aww no!');
```



在接下来的例子中，你会经常看到这个语法。

------

在 `getImage` 的例子中，我们按照顺序执行复杂、嵌套的回调函数，最后获得结果。还好，`.then`方法可以帮我们！

`.then`本身的结果就是一个 promise 值。这就意味着，我们可以链式调用`.then`方法：前一个`then`回调的结果将会作为一个参数传递到下一个`then`的回调函数中！

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--X8h-NDc2--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/i6busbetmoya9vny2eku.png)

```javascript
Promise.resolve(5) // Promise { 5 }
	.then(res => res * 2) // Promise { 10 }
	.then(res => res * 2) // Promise { 20 }
	.then(res => res * 2) // Promise { 40 }
	.then(res => res * 2) // Promise { 80 }
```

在 `getImage` 的例子中，我们可以按照顺序链式调用`then`的回调函数，依次将处理的图像传递到下一个函数中！没有复杂繁琐的嵌套函数，只有一条干净的`then`链条。

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--e1nVrqe1--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/u9l3lxwxlxgv2edv79xh.png)

```javascript
getImage('./image.png')
	.then(image => compressImage(image))
	.then(compressedImage => applyFilter(compressedImage))
	.then(filteredImage => saveImage(filteredImage))
	.then(res => console.log("Successfully saved image!"))
	.catch(err => throw new Error(err))
```

完美！这个语法看起来比嵌套的回调函数好太多了！

------

## 微任务和（宏）任务	Microtasks and (Macro)tasks

好了，我们知道如何去创建一个 promise 然后如何从一个 promise 中提取值。让我们添加更多的代码到脚本里，然后再次运行：

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--uNG7sXon--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/ey4ubnv5yjgi6hbh97xq.gif)

```javascript
console.log('Start!')

Promise.resolve('Promise!')
	.then(res => console.log(res))

console.log('End!')
```

等到了啥？！

首先，出现`Start!`，就是第一行的 `console.log('Start!')` 。但是，第二个得到的结果却是`End!`，并非 promise 解析的值！只有在 `End!` 被输出后，才出现了 promise 解析的结果。这里发生了什么？

我们终于见识到强大的 promise 了！虽然 JavaScript 是单线程的，但是我们可以用 `promise` 来添加一个异步动作！（Although JavaScript is single-threaded, we can add asynchronous behavior using a `Promise`!）

------

请等一等，我们之前难道没有见过么？在JS的事件循环（JavaScript event loop）中，我们难道不能用浏览器对象自身的方法（例如，setTimeout）来创建某种异步行为吗？

当然可以！但是，在事件循环中，实际上有两种队列：（宏）任务队列（the **(macro)task queue** (or just called the **task queue**)）和微任务队列（ the **microtask queue** ）。宏任务队列里放着宏任务，微任务队列放着微任务。

因此，什么是宏任务？什么又是微任务？尽管还有一些我不打算在这里介绍，但最常见的如下表所示！

| (Macro)task | `setTimeout` | `setInterval` | `setImmediate`              |
| ----------- | ---------------------------------------------------------- |
| Microtask   | `process.nextTick` | `Promise callback` | `queueMicrotask` |

啊哈，我们在微任务列表里看到了 `Promise`！当一个`Promise`被解析并且调用它的`then()`，`catch()`，`finally()`方法时，方法里的回调函数就会被添加到**微任务队列**中！这就意味着，`then()`，`catch()`，`finally()`方法中的回调函数不会立即执行，这本质上是在 JS 代码中添加了一些异步行为！（译者说：也就是说这些任务你需要等一段时间，它才会执行。）

所以，`then()`，`catch()`，`finally()`的回调函数什么时候被执行呢？事件循环会为这些任务提供不同的优先级：

1. 当前在调用栈（**call stack**）中的所有函数都将执行。当当前任务返回一个值的时候，它就会被移除出调用栈。
2. 当调用栈为空，所有等待着的微任务（**microtasks**）就会依次弹出到调用栈中，然后执行！（微任务本身也会返回新的微任务，这就有效地创建了无限的微任务循环。）
3. 如果调用栈和微任务队列都是空的，事件循环就会检查宏任务队列里是否有剩下的任务。于是，宏任务就会弹出到调用栈，执行，然后再从调用栈弹出！

------

让我们看一下简单的示例：

* `Task1`：一个被立即添加到调用栈的函数，例如，在我们的代码中立即调用它。
* `Task2`，`Task3`，`Task4`：微任务，例如，一个 promise 中的 `then`回调函数，或是一个被添加到`queueMicrotask`（微任务队列）中的任务。
* `Task5`，`Task6`：一个宏任务，例如，一个`setTimeout`或者`setInterval`的回调函数。

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--05Fi8vBq--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/42eatw03fcha0e1qcrf0.gif)

首先，`Task1`返回了一个值，然后从调用栈中被弹出。之后，JS引擎会在微任务队列中检查任务，一旦调用栈中所有的任务都被执行完毕，（最终弹出调用栈），引擎就会检查宏任务队列，然后弹出到调用栈，在他们返回一个值之后就会被弹出调用栈。

好了，粉色盒子足够了，让我们用一些真正的代码演示它！

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--fnbqqf1d--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/g61wwyi8wchk2hpzeq4u.png)

```javascript
console.log('Start!')

setTimeout(() => {
  console.log('Timeout!')
}, 0)

Promise.resolve('Promise!')
	.then(res => console.log(res))

console.log('End!')
```

在这段代码中，我们有一个宏任务`setTimeout`，以及一个微任务（promise 的 `then()` 回调函数）。引擎到达`setTimeout`函数那一行，让我们逐步运行该代码，看看会输出什么！

------

> 快速参考——在接下来的例子中，我将会用诸如`console.log`，`setTimeout`以及`Promise.resolve`这些方法来添加到调用栈，然后看看结果如何。它们是内置的方法，所以实际上不会出现在堆栈的跟踪中，因此如果你debug时没看到它们也不用担心为什么它们没出现！我们无需添加大量样板代码，就可以更轻松地解释这个概念。

在第一行，引擎遇到了`console.log`方法。它被添加到调用栈，之后它就在控制台中输出了`'Start!'`值，之后这个方法就从调用栈中弹出，引擎继续往下分析代码。

![img](https://res.cloudinary.com/practicaldev/image/fetch/s---Bt6DKsn--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/6cbjuexvy6z9ltk0bi18.gif)

引擎遇到`setTimeout`方法，这个方法弹出到调用栈上，该方法是浏览器原生的方法。它的回调函数(`() => console.log('In timeout')`)将会被添加到 Web API，直到计时器完成。尽管我们给计时器提供的时间是 0，调用栈仍然会首先就把回调函数加入到 Web API 中，之后再被添加到宏任务队列（`setTimeout`是一个宏任务）！

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--6NSYq-nO--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/yqoemb6f32lvovge8yrp.gif)

------

引擎遇到`Promise.resolve()`方法。`Promise.resolve()`方法被添加到调用栈，之后用Promise`的值解析！它的`then 回调函数会被添加到微任务队列。

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--us8FF30N--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/6wxjxduh62fqt531e2rc.gif)

------

引擎遇到`console.log()`方法。它被立即添加到调用栈，之后在控制台得到一个值 `'End!'`，然后从调用栈弹出，引擎继续执行。

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--oOS_-CiG--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/a6jk0exl137yka3oq9k4.gif)

现在，引擎看到调用栈是空的，因为它是空的，所以将会检查**微任务队列**里是否有任务！确实有，promise 的`then`回调函数正等着执行呢！它被弹出到调用栈中，之后它输出 promise 解析的值：`'Promise!'`

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--5iH5BNWm--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/lczn4fca41is4vpicr6w.gif)

引擎看到调用栈又空了，它就再次去检查微任务队列，看看是否还有任务。没有任务了，微任务队列是空的。

是时候检查宏任务了：`setTimieout`的回调函数还在那等着呢！这个回调函数会被弹出到调用栈中，最后返回一个`console.log`方法，它会输出`"Timeout!"`。执行完毕后，`setTimeout`从调用栈弹出。

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--hPFPTZp2--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/p54casaaz9oq0g8ztpi5.gif)

最后，所有都执行完毕了！不出意外，和我们之前想的一样，确实是这些结果。

------

## Async / Await

ES7 引入了一种全新的添加异步行为的方式，它可以让 Promise 的写法变得更容易！随着关键字 `async` 和 `await`的引入，我们可以创建异步函数（**async** functions）了，它隐式地返回一个 promise。但是，我们要怎么做呢？

之前，我们看到我们可以直接用`Promise`对象创建承诺，比如通过键入 `new Promise(() => {})` ，`Promise.resolve`，或是 `Promise.reject`。

现在我们不用直接写`Promise`对象了，而是通过创建异步函数然后间接地返回一个 Promise 对象！也就是说，我们可以不再用任何 `Promise` 对象了。

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--5ED_HyNC--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/72lqrcvy9lc8ehbpitd0.png)

```javascript
Promise.resolve('Hello!')

async function greet() {
  return 'Hello!'
}

// 译者注：执行 greet()，得到的结果和 Promise.resolve('Hello!') 一样，都返回一个 promise 对象。
```

尽管**异步函数**隐式返回 promise 已经非常 nice 了，但当你使用 `await` 关键字时，会看到异步函数的真正威力！当我们在等待一个值（返回一个 resolved promise）的时候，可以利用`await`关键字去*暂停* （*suspend*）异步函数。如果我们想要得到一个有结果的 promise，像之前对 `then()` 的回调函数做的那样，我们就要把等待的那个promise值赋值给一个变量进行保存！

所以，我们如何暂停一个异步函数呢？非常好……但是那到底是什么意思呢？

让我们看一下，如果我们运行下面的代码会发生什么：

![Alt Text](https://res.cloudinary.com/practicaldev/image/fetch/s--aOWmZxnV--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/e5duygomitj9o455107a.gif)

```javascript
const one = () => Promise.resolve('One');

async function myFunc () {
  console.log('In function');
  const res = await one();
  console.log(res);
}

console.log('Before function!');
myFunc();
console.log('After function!');
```

emmm……这里发生了什么？

------

![Alt Text](https://res.cloudinary.com/practicaldev/image/fetch/s--bfscMU3t--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/d27d7xxiekczftjyic4b.gif)

首先，引擎遇到了 `console.log`。它被弹出到调用栈，之后`"Before function!"`被输出。

------

![Alt Text](https://res.cloudinary.com/practicaldev/image/fetch/s--wN7yFTnt--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/9wqej2269vmntfcuxs9t.gif)

然后，我们调用了异步函数`myFunc()`，函数体内代码运行。在这个函数体的最上面一行，我们又一次调用了`console.log`，这次打印了字符串`"In function!"`。整个过程就是 `console.log`被添加到调用栈，输出结果，然后从调用栈弹出。

------

![Alt Text](https://res.cloudinary.com/practicaldev/image/fetch/s--lX9JfreE--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/lch6lutxnl88j0durpyh.gif)

函数体继续执行，到了第二行，我们看到了`await`关键字。

第一件要发生的事情是，我们要等待的那个值会被执行。它被加入调用栈，最终返回一个 resolved promise。在引擎遇到`await`，只要 promise 得到结果了，那么`one`这个函数就会返回一个值。

当遇到一个`await`关键字，`async`函数就会暂停。函数体的执行被暂时中止，异步函数的剩下部分会在微任务中运行，而不是常规的任务！

------

![Alt Text](https://res.cloudinary.com/practicaldev/image/fetch/s--UC78HoCO--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/b6l3psgewvtrtmrr60tg.gif)

现在异步函数`myFunc`被暂停了，因为它遇到了`await`关键字，引擎会跳过这个异步函数，继续运行异步函数所在的**执行上下文**中的其他代码，这里的执行上下文是全局执行上下文（the **global execution context**）！

------

![Alt Text](https://res.cloudinary.com/practicaldev/image/fetch/s--V8u36kEG--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/hlhrtuspjyrstifubdhs.gif)

最后，全局执行上下文中没有更多的任务可以被运行了！此时，事件循环就会去检查微任务中是否有任务还未执行：确实有的！异步函数`myFunc`在解析了`one`的值以后还在排着队呢。`myFunc`被弹出到调用栈中，然后继续运行它的函数体内之前剩下的代码。

变量`res`最后得到了它的值，也就是`one`返回的promise里的值！我们将`res`变量中的值放进`console.log`中运行，得到了字符串`"One!"`，最后弹出了调用栈！

最后，一切搞定！你注意到`async`函数与 promise 的`then`的区别了吗？`await`关键字会暂停`async`函数，然而 Promise 主体在我们使用了`then`之后，会一直运行！

------

emmm……如此庞大的信息量啊！如果你在使用 Promise 时还是有点懵，也完全不用担心，我个人感觉只要慢慢积累经验就能对它有感觉，如此一来在使用异步JS的时候也会越来越自信的。

但是，此时此刻，经过本文的解释，我真诚地希望你所遇到的“意外”或者“不可预测”的行为都能让你对异步JS有一丝的感觉！

**2021.4.15 Translated by Knight**