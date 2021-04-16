# （译）看得见的 JavaScript：事件循环（event loop）

事件循环来了！这是每一个JS开发者都会遇到的东西，但一开始理解起来会很复杂。

首先，什么是事件循环，为什么你应该关注它？

JS 是单线程（**single-threaded**）的：一次只能运行一个任务。通常没什么大问题，但是现在想象一下，你正在运行一个要耗时30s的任务……我们必须等着这30s过去才能执行其他的代码（JS默认在浏览器的主线程上运行，所以整个UI的解析会卡住。），都2019年了，没有人想要一个缓慢、没有响应的网页。

还好浏览器给了我们一些JS引擎没有提供的特性：Web API。它包括 DOM API，`setTimeout`，HTTP requests 等等。它能帮我们创建一些异步的、不会阻塞的行为。

当我们调用一个函数时，它被添加到一个叫做调用栈的地方。调用栈是JS引擎的一部分，它不是浏览器特有的。这是一个堆栈，意味着先进后出（想象一堆煎饼堆叠在一起）。当函数返回了一个值时，它就会从调用栈中弹出。

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--44yasyNX--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://devtolydiahallie.s3-us-west-1.amazonaws.com/gid1.6.gif)

`respond`函数返回一个 `setTimeout`函数，这个`setTimeout`就是 Web API 提供给我们的，它的作用是让我们延迟任务的执行而不用阻塞主线程。我们传给`setTimeout`的回调函数，一个箭头函数`() => { return 'Hey' }`加入到 Web API 中。同时，`setTimeout`函数和`respond`函数被弹出调用栈，它们都返回了它们的值！

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--d_n4m4HH--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://devtolydiahallie.s3-us-west-1.amazonaws.com/gif2.1.gif)

在 Web API 中，一个 1000ms 的计时器开始执行。这个回调函数不会立即被添加到调用栈，而是被传递到一个被称为队列的地方。

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--MewGMdte--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://devtolydiahallie.s3-us-west-1.amazonaws.com/gif3.1.gif)

这是令人疑惑的部分：这并不意味着在1000ms之后，回调函数会被添加到调用栈中！而是简单地添加到队列中，但正是因为它是一个队列，所以函数必须在这等待执行。

该是事件循环执行其唯一任务的时候了：将调用栈与队列连接（**connecting the queue with the call stack**）！如果调用栈是空的，之前所有被调用的函数都返回了值并且弹出了调用栈，那么队列中的第一个任务将会被弹出到调用栈中。在本例中，没有其他的函数被调用了，也就是说调用栈是空的。

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--b2BtLfdz--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://devtolydiahallie.s3-us-west-1.amazonaws.com/gif4.gif)

回调函数添加到调用栈中，执行，然后返回一个值，弹出调用栈。

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--NYOknEYi--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://devtolydiahallie.s3-us-west-1.amazonaws.com/gif5.gif)

------

阅读一篇文章非常轻松愉快，但是你只有一遍又一遍地使用它才能完全掌握它。如果我们运行一下示例，想想看控制台里会出现什么？

```javascript
const foo = () => console.log("First");
const bar = () => setTimeout(() => console.log("Second"), 500);
const baz = () => console.log("Third");

bar();
foo();
baz();
```

理解了吗？让我们快速地看一下当我们在浏览器运行这段代码时正在发生什么：

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--BLtCLQcd--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://devtolydiahallie.s3-us-west-1.amazonaws.com/gif14.1.gif)

1. 我们调用`bar`，`bar`返回一个`setTimeout`函数。
2. 我们传给`setTimeout`的回调函数被添加到 Web API 中，`setTimeout` 和 `bar` 都从调用栈中弹出。
3. 计时器运行，同时，`foo`被调用，输出`"First"`。`foo`返回`undefined`。`baz`被调用，然后Web API中的回调函数被加入到队列中。
4. `baz`输出`"Third"`。在`baz`执行完毕后，事件循环看到调用栈空了，此时队列中的回调函数弹出到调用栈中。
5. 回调函数输出`"Second"`。

**2021.4.16 Translated by Knight**