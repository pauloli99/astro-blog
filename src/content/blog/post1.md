---
title: "可视化分析Promises & Async/Await 和 Event Loop"
description: "通过GIF动图来解释浏览器的Event Loop"
pubDate: "May 5 2023"
# heroImage: "/post_img.webp"
---

## 介绍

当我们在编写 JavaScript 时，可能会遇到需要处理的某个任务依赖另一个任务返回的数据的情况。比如说这个场景：我们需要先获取一张图片，然后对图片进行压缩、添加滤镜，最后再保存。在这个例子中，每一个任务都对前一个任务有依赖，那么该如何处理呢？

我们需要做的第一件事就是，获取这张需要编辑的图片。编写一个**getImage**方法来获取图片，在成功获取图片之后，将图片传递给**resizeImage**方法，然后再将返回值传递给**applyFilter**方法。经过这一系列的处理后，我们便可以将图片保存并通知用户一切都处理完成了。

最后，我们会得到这样的代码：

![https://res.cloudinary.com/practicaldev/image/fetch/s---Kv6sJn7--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/ixceqsql5hpdq8txx43s.png](https://res.cloudinary.com/practicaldev/image/fetch/s---Kv6sJn7--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/ixceqsql5hpdq8txx43s.png)

emmm，虽然代码逻辑看上去好像没什么问题，但是好像又不太好。因为我们对图片的多个加工步骤导致我们编写了多个嵌套的回调函数。这就是我们熟知的“回调地狱”，这会令代码难以阅读及维护。

幸运的是，我们现在有了一个叫做 Promise 的东西，它可以帮助我们规避回调地狱的问题。

---

## Promise

ES6 为我们带来了 Promises。在许多教程中，你可能会看到这样一句话：

> 一个 promise 就是一个值的占位符，它可以在将来的某个时刻被 resolve 或者 reject

没错，这个解释依旧让人捉摸不透。

我们可以使用 Promise 构造函数来创建一个 promise 实例，它需要接受一个回调函数：

![https://res.cloudinary.com/practicaldev/image/fetch/s--phTVdCKA--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/79zi452hphe7ecylhozy.gif](https://res.cloudinary.com/practicaldev/image/fetch/s--phTVdCKA--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/79zi452hphe7ecylhozy.gif)

让我们来看一下它返回了什么

Promise 是一个包含状态 ([[PromiseStatus]]) 和值 ([[PromiseValue]]) 的对象。在上面的例子中，可以看到 [[PromiseStatus]] 的值为“pending”，promise 的值为 undefined。

别担心，你并不需要与这个对象交互，你甚至不能访问 [[PromiseStatus]] 和 [[PromiseValue]] 属性！但是，这些属性的值在使用 promises 时很重要。

PromiseStatus 的值，即状态，可以是以下三个值之一：

- ✅ fulfilled：promise 已经兑现。一切顺利，promise 中没有出现错误

- ❌ rejected ：承诺已被拒绝。啊，一定是什么地方出了点问题

- ⏳ pending：promise 既没有 resolved 也没有 rejected，promise 仍然处于挂起的状态

在上面的示例中，我们只是将简单的回调函数 () => {} 传递给了 Promise 构造函数。然而，这个回调函数实际上接收两个参数。第一个参数的值，通常称为 resolve 或 res，是 Promise 应该 resolve 时调用的方法。第二个参数的值，通常称为 reject 或 rej，是当 Promise 应该拒绝、出错时调用的值方法。

让我们尝试看看当我们调用 resolve 或 reject 方法时，会发生什么改变。在我的示例中，我将 resolve 方法称为 res，将 reject 方法称为 rej。

![https://res.cloudinary.com/practicaldev/image/fetch/s--qKIq-sYt--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/z0b9v0h7aiq073l5tl2l.gif](https://res.cloudinary.com/practicaldev/image/fetch/s--qKIq-sYt--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/z0b9v0h7aiq073l5tl2l.gif)

非常好，我们已经知道了该如何将 pending 状态转变为 fulfilled 或者 rejected 状态了，就是通过调用 res 方法或者 rej 方法。

promise 的值，即 [[PromiseValue]] 的值，是我们作为参数传递给 resolved 或 rejected 方法的值。

现在我们对 Promise 已经有了一个初步的认识了，但是它到底有什么用呢？

还记得在文章开头，我举了一个加工图片的例子吗，我们通过层层嵌套的回调函数完成了这个需求。

幸运的是，Promises 可以帮助我们解决这个问题！首先，让我们重写整个代码块，让每个函数返回一个 Promise。

如果图片加载一切顺利，那么我们就调用 resolve 方法并传递图片，如果加载失败了就使用 reject 方法来传递一个错误。

![https://res.cloudinary.com/practicaldev/image/fetch/s--r9xngcNz--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/iebp0rzfnfqsrmmjplme.png](https://res.cloudinary.com/practicaldev/image/fetch/s--r9xngcNz--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/iebp0rzfnfqsrmmjplme.png)

让我们看看改造后的 getImage 会返回什么

![https://res.cloudinary.com/practicaldev/image/fetch/s--uERkfSWf--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/wsu5nn26dp4elcwh764m.gif](https://res.cloudinary.com/practicaldev/image/fetch/s--uERkfSWf--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/wsu5nn26dp4elcwh764m.gif)

正如我们预期的那样，返回了一个带有状态和值的 promise。

但是，我们要如何去获取 promise 内部的值并利用起来呢，这里有三个 promise 自带的方法：

- .then()：在 promise 被 resolve 后被调用

- .catch()：在 promise 被 reject 后被调用

- .finally()：无论是 promise 被 resolve 还是 reject，总是会调用

![https://res.cloudinary.com/practicaldev/image/fetch/s--19tIvFJQ--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/mu1aqqnyfjsfon5hwrtw.png](https://res.cloudinary.com/practicaldev/image/fetch/s--19tIvFJQ--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/mu1aqqnyfjsfon5hwrtw.png)

.then 方法接收传递给 resolve 方法的值

![https://res.cloudinary.com/practicaldev/image/fetch/s--DZld0c-0--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/11vxhn9cun7stpjbdi80.gif](https://res.cloudinary.com/practicaldev/image/fetch/s--DZld0c-0--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/11vxhn9cun7stpjbdi80.gif)

.catch 方法接收传递给 reject 方法的值

![https://res.cloudinary.com/practicaldev/image/fetch/s--e9SZHcPk--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/v5y24jz4u89flazvdyn4.gif](https://res.cloudinary.com/practicaldev/image/fetch/s--e9SZHcPk--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/v5y24jz4u89flazvdyn4.gif)

---

我们还可以使用 Promise.resolve 或 Promise.reject 来始终更改某个 promise 的状态并设定值：

![https://res.cloudinary.com/practicaldev/image/fetch/s--61Gva3Ze--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/90hxwjfadzslvdbkr4l8.png](https://res.cloudinary.com/practicaldev/image/fetch/s--61Gva3Ze--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/90hxwjfadzslvdbkr4l8.png)

---

在不使用 Promise 的情况下，我们的 getImage 方法不得不嵌套多个回调函数。现在我们有了 Promise，便可以利用它的 then 方法来解决这个问题！

then 方法默认情况下就会返回一个 promise，这意味着我们可以链式调用 then 方法，并且通过前一个 then 方法将值传递给后一个 then

![https://res.cloudinary.com/practicaldev/image/fetch/s--X8h-NDc2--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/i6busbetmoya9vny2eku.png](https://res.cloudinary.com/practicaldev/image/fetch/s--X8h-NDc2--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/i6busbetmoya9vny2eku.png)

那么，经过 promise 改造后的 getImage 方法，将会是这样：

![https://res.cloudinary.com/practicaldev/image/fetch/s--e1nVrqe1--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/u9l3lxwxlxgv2edv79xh.png](https://res.cloudinary.com/practicaldev/image/fetch/s--e1nVrqe1--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/u9l3lxwxlxgv2edv79xh.png)

非常好，这样的代码比一开始的回调地狱看上去好看多了

---

## 微任务和宏任务

让我们先看看下面这段代码：

![https://res.cloudinary.com/practicaldev/image/fetch/s--uNG7sXon--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/ey4ubnv5yjgi6hbh97xq.gif](https://res.cloudinary.com/practicaldev/image/fetch/s--uNG7sXon--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/ey4ubnv5yjgi6hbh97xq.gif)

如果你刚接触 promise，你可能会觉得有点懵逼。为啥这代码不按照顺序来执行呢？

首先，Start！被打印了，随后被打印的是 End！并不是 Promise 返回的‘Promise！’，在这里面究竟发生了什么？

通过这个案例才让我们见识到了 Promise 真正的威力。尽管 JavaScript 是单线程语言，但是我们可以通过 Promise 来实现异步行为！

---

在 JavaScript 事件循环中，我们还可以使用浏览器提供的 WebAPI 比如 setTimeout 之类的定时器任务在实现一些异步行为

事实上，在事件循环中有着两种任务队列：一种叫做宏任务队列（也可以叫做任务队列），另一种叫做微任务队列。

常见的宏任务和微任务如下：

| (Macro)task | `setTimeout`       | `setInterval`      | `setImmediate`   |
| ----------- | ------------------ | ------------------ | ---------------- |
| Microtask   | `process.nextTick` | `Promise callback` | `queueMicrotask` |

可以看到，我们刚才了解到的 Promise 处于微任务队列中。当我们在调用 Promise 的 then，catch 或 finally 方法时，实际上就是将所传入的回调添加进了微任务队列。这意味这些方法并不会立刻执行。

那么什么时候执行 then()、catch() 或 finally() 的回调？事件循环赋予了任务不同的优先级：

1. 当前在调用堆栈中的所有函数都会被执行。当他们返回一个值后，便会从堆栈中弹出

2. 当调用栈为空时，所有排队的微任务被一个一个弹出到调用栈中，并得到执行！ （微任务本身也可以返回新的微任务）

3. 如果调用堆栈和微任务队列都是空的，事件循环会检查（宏）任务队列中是否还有任务。任务被弹出到调用堆栈，执行，然后弹出！

---

让我们看一个简单的例子：

- Task1：假设 Task1 是一个同步执行的函数，会最先被添加到调用堆栈中并执行

- Task2、Task3、Task4：都是微任务，可能是 promise 的 then 回调，也可能是被添加到了微任务队列的任务

- Task5、Task6：一个（宏）任务，例如 setTimeout 或 setImmediate 回调

首先，Task1 返回一个值并从调用堆栈中弹出。然后，引擎检查在微任务队列中排队的任务。一旦所有任务都被放入调用堆栈并最终弹出，引擎会检查（宏）任务队列中的任务，这些任务会弹出到调用堆栈，并在它们返回值时弹出。

让我们再看看这段代码：

![https://res.cloudinary.com/practicaldev/image/fetch/s--fnbqqf1d--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/g61wwyi8wchk2hpzeq4u.png](https://res.cloudinary.com/practicaldev/image/fetch/s--fnbqqf1d--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/g61wwyi8wchk2hpzeq4u.png)

在这段代码中，我们有宏任务 setTimeout 和微任务承诺 then() 回调。当 JavaScript 引擎到达 setTimeout 函数的时候。让我们逐步运行这段代码，看看记录了什么！

---

在第一行，引擎遇到了 console.log() 方法。它被添加到调用堆栈，之后它记录值 Start！到控制台。该方法从调用堆栈中弹出，引擎继续。

![https://res.cloudinary.com/practicaldev/image/fetch/s---Bt6DKsn--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/6cbjuexvy6z9ltk0bi18.gif](https://res.cloudinary.com/practicaldev/image/fetch/s---Bt6DKsn--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/6cbjuexvy6z9ltk0bi18.gif)

当引擎遇到 setTimeout 方法时，该方法被弹出到调用堆栈。 setTimeout 方法是浏览器原生的：它的回调函数 (() => console.log('In timeout')) 将被添加到 Web API，直到计时器完成。尽管我们为计时器延时设置为了 0，回调仍然会首先被推送到 Web API，再之后它被添加到（宏）任务队列中：setTimeout 是一个宏任务！

![https://res.cloudinary.com/practicaldev/image/fetch/s--6NSYq-nO--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/yqoemb6f32lvovge8yrp.gif](https://res.cloudinary.com/practicaldev/image/fetch/s--6NSYq-nO--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/yqoemb6f32lvovge8yrp.gif)

随后，引擎遇到 Promise.resolve() 方法。 Promise.resolve() 方法被添加到调用堆栈中，然后 resolve 了‘Promise!’这个值。它的 then 回调函数被添加到微任务队列中。

![https://res.cloudinary.com/practicaldev/image/fetch/s--us8FF30N--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/6wxjxduh62fqt531e2rc.gif](https://res.cloudinary.com/practicaldev/image/fetch/s--us8FF30N--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/6wxjxduh62fqt531e2rc.gif)

引擎遇到 console.log() 方法。它立即被添加到调用堆栈，之后它记录值 End！到控制台，从调用堆栈弹出，引擎继续运行

![https://res.cloudinary.com/practicaldev/image/fetch/s--oOS_-CiG--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/a6jk0exl137yka3oq9k4.gif](https://res.cloudinary.com/practicaldev/image/fetch/s--oOS_-CiG--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/a6jk0exl137yka3oq9k4.gif)

引擎看到调用堆栈现在是空的。由于调用栈是空的，所以它会检查微任务队列中是否有排队的任务！是的，终于轮到 then 方法的回调了！它被弹出到调用堆栈上，之后它记录了承诺的解析值：在本例中为字符串 Promise!。

![https://res.cloudinary.com/practicaldev/image/fetch/s--5iH5BNWm--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/lczn4fca41is4vpicr6w.gif](https://res.cloudinary.com/practicaldev/image/fetch/s--5iH5BNWm--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/lczn4fca41is4vpicr6w.gif)

引擎看到调用堆栈是空的，所以它会再次检查微任务队列以查看任务是否需要排队。

引擎发现微任务队列都是空的。那么是时候检查（宏）任务队列了：setTimeout 回调仍在等待中。 setTimeout 回调被弹出到调用堆栈。回调函数返回 console.log 方法，该方法记录字符串“In timeout!”。 最后 setTimeout 回调从调用堆栈中弹出

![https://res.cloudinary.com/practicaldev/image/fetch/s--hPFPTZp2--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/p54casaaz9oq0g8ztpi5.gif](https://res.cloudinary.com/practicaldev/image/fetch/s--hPFPTZp2--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/p54casaaz9oq0g8ztpi5.gif)

恭喜你，如果你阅读到这里，相信你已经能够基本理解什么是 Promise 以及什么是事件循环了
