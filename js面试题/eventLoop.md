# js 中的事件循环(event loop)

## js 中的 eventloop

#### 大致流程

简单的说，事件循环（eventLoop）是单线程的 JavaScript 在处理异步事件时进行的一种循环过程，具体来讲，对于异步事件它会先加入到事件队列中挂起，等主线程空闲时会去执行事件队列中的事件。

**主线程任务——>微任务——>宏任务** 如果宏任务里还有微任就继续执行宏任务里的微任务，如果宏任务中的微任务中还有宏任务就在依次进行

**主线程任务——>微任务——>宏任务——>宏任务里的微任务——>宏任务里的微任务中的宏任务——>直到任务全部完成** 我的理解是在同级下，微任务要优先于宏任务执行

在同一轮任务队列中，同一个微任务产生的微任务会放在这一轮微任务的后面，产生的宏任务会放在这一轮的宏任务后面

在同一轮任务队列中，同一个宏任务产生的微任务会马上执行，产生的宏任务会放在这一轮的宏任务后面

它不停检查 Call Stack 中是否有任务（也叫栈帧）需要执行，如果没有，就检查 Event Queue，从中弹出一个任务，放入 Call Stack 中，如此往复循环。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ae5dbcc6cb594cdfa5b1b10bcd2ad1e1~tplv-k3u1fbpfcp-zoom-1.image)

- 同步和异步任务分别进入不同的执行"场所"，同步的进入主线程，异步的进入 Event Table 并注册函数。

- 当指定的事情完成时，Event Table 会将这个函数移入 Event Queue。

- 主线程内的任务执行完毕为空，会去 Event Queue 读取对应的函数，进入主线程执行。

- 上述过程会不断重复，也就是常说的 Event Loop(事件循环)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/638f9d513a6b499b8d108362b0b9b664~tplv-k3u1fbpfcp-zoom-1.image)

#### 为什么先微后宏

微任务会在执行任何其他事件处理，或渲染，或执行任何其他宏任务之前完成。

这很重要，因为它确保了微任务之间的应用程序环境基本相同（没有鼠标坐标更改，没有新的网络数据等）。

如果我们想要异步执行（在当前代码之后）一个函数，但是要在更改被渲染或新事件被处理之前执行，那么我们可以使用 **queueMicrotask** 来对其进行安排（schedule）

## 概念名词

### 主线程

所有的同步任务都是在主线程里执行的，异步任务可能会在 macrotask 或者 microtask 里面

**同步任务：** 指的是在主线程上排队执行的任务，只有前一个任务执行完毕，才能执行后一个任务。**异步任务：** 指的是不进入主线程，某个异步任务可以执行了，该任务才会进入主线程执行。

### 微任务(micro task)

- promise

- async

- await

- process.nextTick(node)

- mutationObserver(html5 新特性)

### 宏任务(macro task)

- script(整体代码)

- setTimeout

- setInterval

- setImmediate

- I/O

- UI render

**分析 setTimeout**

> **`setTimeout(fn,0)`\*\***的含义是，指定某个任务在主线程最早可得的空闲时间执行，意思就是不用再等多少秒了，只要主线程执行栈内的同步任务全部执行完成，栈为空就马上执行\*\*

```
setTimeout(() => {

  task()

},3000)

sleep(10000000)
```

乍一看其实差不多嘛，但我们把这段代码在 chrome 执行一下，却发现控制台执行`task()`需要的时间远远超过 3 秒，说好的延时三秒，为啥现在需要这么长时间啊？

这时候我们需要重新理解`setTimeout`的定义。我们先说上述代码是怎么执行的：

- `task()`进入 Event Table 并注册,计时开始。

- 执行`sleep`函数，很慢，非常慢，计时仍在继续。

- 3 秒到了，计时事件`timeout`完成，`task()`进入 Event Queue，但是`sleep`也太慢了吧，还没执行完，只好等着。

- `sleep`终于执行完了，`task()`终于从 Event Queue 进入了主线程执行。

### 运行时概念

![运行时三者之间的关系可视化描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9719bf13ca484de9999ab23c401ec437~tplv-k3u1fbpfcp-zoom-1.image)

#### 堆(heap)

保存的地址

> 对象被分配在堆中，堆是一个用来表示一大块（通常是非结构化的）内存区域的计算机术语。

#### 栈(stack)

后进先出 (lifo) last in first out （坐电梯 🌰 第一个进电梯的人最后一个出来，最后一个进电梯的人第一个出来！）

函数调用形成了一个由若干帧组成的栈。

```javascript
function foo(b) {
  let a = 10

  return a + b + 11
}

function bar(x) {
  let y = 3

  return foo(x * y)
}

console.log(bar(7)) // 返回 42
```

当调用 `bar` 时，第一个帧被创建并压入栈中，帧中包含了 `bar` 的参数和局部变量。 当 `bar` 调用 `foo` 时，第二个帧被创建并被压入栈中，放在第一个帧之上，帧中包含 `foo` 的参数和局部变量。当 `foo` 执行完毕然后返回时，第二个帧就被弹出栈（剩下 `bar` 函数的调用帧 ）。当 `bar` 也执行完毕然后返回时，第一个帧也被弹出，栈就被清空了。

#### 队列(queue)

先进先出 (fifo) first in first out

当 Event Table 中的事件被触发，事件对应的 回调函数 就会被 push 进这个 Event Queue，然后等待被执行

一个 JavaScript 运行时包含了一个待处理消息的消息队列。每一个消息都关联着一个用以处理这个消息的回调函数。

在 [事件循环](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/EventLoop#事件循环) 期间的某个时刻，运行时会从最先进入队列的消息开始处理队列中的消息。被处理的消息会被移出队列，并作为输入参数来调用与之关联的函数。正如前面所提到的，调用一个函数总是会为其创造一个新的栈帧。

函数的处理会一直进行到执行栈再次为空为止；然后事件循环将会处理队列中的下一个消息（如果还有的话）。

#### 事件表格（event table）

Event Table 可以理解成一张**事件->回调函数** 对应表

调用 web apis 来执行函数然后回调到事件队列中

它就是用来存储 Js 中的异步事件 (request, setTimeout, IO 等) 及其对应的回调函数的列表

#### Web APIs

浏览器提供了多种异步的 Web API，如 DOM，times（计时器），AJAX 等。

当我们调用一个 Web API 时，如 setTimeout，setTimeout 函数会被 push 调用栈顶然后执行，但是 setTimeout 的回调函数不会立即被 push 到调用栈顶，而是起一个计时器任务。当这个计时器结束时，该回调函数会被塞到任务队列（CallBack Queue）中。这个队列中的回调函数的调用就是由事件循环机制来控制的。

**理解: 调用任务时，并不会马上进入任务队列，而是先调用 web 提供的 api，等待执行的结果才放进去任务队列**

#### Web Workers

对于不应该阻塞事件循环的耗时长的繁重计算任务，我们可以使用 [Web Workers](https://html.spec.whatwg.org/multipage/workers.html)。

这是在另一个并行线程中运行代码的方式。

Web Workers 可以与主线程交换消息，但是它们具有自己的变量和事件循环。

Web Workers 没有访问 DOM 的权限，因此，它们对于同时使用多个 CPU 内核的计算非常有用。

#### 进程和线程

[进程与线程的一个简单解释](https://www.ruanyifeng.com/blog/2013/04/processes_and_threads.html)

## 举一个 🌰

```JavaScript
console.log('script start')

async function async1() {
await async2()
console.log('async1 end')
}
async function async2() {
console.log('async2 end')
}
async1()

setTimeout(function() {
console.log('setTimeout')
}, 0)

new Promise(resolve => {
console.log('Promise')
resolve()
})
.then(function() {
console.log('promise1')
})
.then(function() {
console.log('promise2')
})

console.log('script end')

  // 新版输出(新版的chrome浏览器优化了,await变得更快了,输出为)

 // script start => async2 end => Promise => script end => async1 end => promise1 => promise2  => setTimeout


 // 旧版输出如下，但是请继续看完本文下面的注意那里，新版有改动
// script start => async2 end => Promise => script end => promise1 => promise2 => **async1 end**  => setTimeout
```

但是这种做法其实是违法了规范的，当然规范也是可以更改的，这是 V8 团队的一个 [PR](https://link.juejin.cn/?target=https://github.com/tc39/ecma262/pull/1250#issue-197979338) ，目前新版打印已经修改。 知乎上也有相关讨论,可以看看 [www.zhihu.com/question/26…](https://link.juejin.cn/?target=https://www.zhihu.com/question/268007969)

## 如何添加宏任务和微任务

安排（schedule）一个新的 **宏任务** ：

- 使用零延迟的 `setTimeout(f)`。

它可被用于将繁重的计算任务拆分成多个部分，以使浏览器能够对用户事件作出反应，并在任务的各部分之间显示任务进度。

此外，也被用于在事件处理程序中，将一个行为（action）安排（schedule）在事件被完全处理（冒泡完成）后。

安排一个新的 **微任务** ：

- 使用 `queueMicrotask(f)`。

- promise 处理程序也会通过微任务队列。

在微任务之间没有 UI 或网络事件的处理：它们一个立即接一个地执行。

所以，我们可以使用 `queueMicrotask` 来在保持环境状态一致的情况下，异步地执行一个函数。

## 事件循环的其他应用

### 1 拆分 CPU 过载任务

假设我们有一个 CPU 过载任务。

例如，语法高亮（用来给本页面中的示例代码着色）是相当耗费 CPU 资源的任务。为了高亮显示代码，它执行分析，创建很多着了色的元素，然后将它们添加到文档中 —— 对于文本量大的文档来说，需要耗费很长时间。

当引擎忙于语法高亮时，它就无法处理其他 DOM 相关的工作，例如处理用户事件等。它甚至可能会导致浏览器“中断（hiccup）”甚至“挂起（hang）”一段时间，这是不可接受的。

我们可以通过将大任务拆分成多个小任务来避免这个问题。高亮显示前 100 行，然后使用 `setTimeout`（延时参数为 0）来安排（schedule）后 100 行的高亮显示，依此类推。

为了演示这种方法，简单起见，让我们写一个从 `1` 数到 `1000000000` 的函数，而不写文本高亮。

如果你运行下面这段代码，你会看到引擎会“挂起”一段时间。对于服务端 JS 来说这显而易见，并且如果你在浏览器中运行它，尝试点击页面上其他按钮时，你会发现在计数结束之前不会处理其他事件。

```JavaScript
let i = 0;

let start = Date.now();

function count() {

  // 做一个繁重的任务

  for (let j = 0; j < 1e9; j++) {

    i++;

  }

  alert("Done in " + (Date.now() - start) + 'ms');

}

count();
```

浏览器甚至可能会显示一个“脚本执行时间过长”的警告。

让我们使用嵌套的 `setTimeout` 调用来拆分这个任务：

```JavaScript
let i = 0;

let start = Date.now();

function count() {

  // 做繁重的任务的一部分 (*)

  do {

    i++;

  } while (i % 1e6 != 0);

  if (i == 1e9) {

    alert("Done in " + (Date.now() - start) + 'ms');

  } else {

    setTimeout(count); // 安排（schedule）新的调用 (**)

  }

}

count();
```

现在，浏览器界面在“计数”过程中可以正常使用。

单次执行 `count` 会完成工作 `(*)` 的一部分，然后根据需要重新安排（schedule）自身的执行 `(**)`：

1. 首先执行计数：`i=1...1000000`。

2. 然后执行计数：`i=1000001..2000000`。

3. ……以此类推。

现在，如果在引擎忙于执行第一部分时出现了一个新的副任务（例如 `onclick` 事件），则该任务会被排入队列，然后在第一部分执行结束时，并在下一部分开始执行前，会执行该副任务。周期性地在两次 `count` 执行期间返回事件循环，这为 JavaScript 引擎提供了足够的“空气”来执行其他操作，以响应其他的用户行为。

值得注意的是这两种变体 —— 是否使用了 `setTimeout` 对任务进行拆分 —— 在执行速度上是相当的。在执行计数的总耗时上没有多少差异。

为了使两者耗时更接近，让我们来做一个改进。

我们将要把调度（scheduling）移动到 `count()` 的开头：

```JavaScript
let i = 0;

let start = Date.now();

function count() {

  // 将调度（scheduling）移动到开头

  if (i < 1e9 - 1e6) {

    setTimeout(count); // 安排（schedule）新的调用

  }

  do {

    i++;

  } while (i % 1e6 != 0);

  if (i == 1e9) {

    alert("Done in " + (Date.now() - start) + 'ms');

  }

}

count();
```

现在，当我们开始调用 `count()` 时，会看到我们需要对 `count()` 进行更多调用，我们就会在工作前立即安排（schedule）它。

如果你运行它，你很容易注意到它花费的时间明显减少了。

为什么？

这很简单：你应该还记得，多个嵌套的 `setTimeout` 调用在浏览器中的最小延迟为 4ms。即使我们设置了 `0`，但还是 `4ms`（或者更久一些）。所以我们安排（schedule）得越早，运行速度也就越快。

最后，我们将一个繁重的任务拆分成了几部分，现在它不会阻塞用户界面了。而且其总耗时并不会长很多。

### 2 进度指示

对浏览器脚本中的过载型任务进行拆分的另一个好处是，我们可以显示进度指示。

正如前面所提到的，仅在当前运行的任务完成后，才会对 DOM 中的更改进行绘制，无论这个任务运行花费了多长时间。

从一方面讲，这非常好，因为我们的函数可能会创建很多元素，将它们一个接一个地插入到文档中，并更改其样式 —— 访问者不会看到任何未完成的“中间态”内容。很重要，对吧？

这是一个示例，对 `i` 的更改在该函数完成前不会显示出来，所以我们将只会看到最后的值：

```JavaScript
<div id="progress"></div>

<script> function count() {

  for (let i = 0; i < 1e6; i++) {

    i++;

    progress.innerHTML = i;

  }

  }

  count();
  </script>
```

……但是我们也可能想在任务执行期间展示一些东西，例如进度条。

如果我们使用 `setTimeout` 将繁重的任务拆分成几部分，那么变化就会被在它们之间绘制出来。

这看起来更好看：

```JavaScript
<div id="progress"></div>

<script> let i = 0;

  function count() {

    // 做繁重的任务的一部分 (*)

    do {

      i++;

      progress.innerHTML = i;

    } while (i % 1e3 != 0);

    if (i < 1e7) {

      setTimeout(count);

    }

  }

  count();
  </script>
```

现在 `div` 显示了 `i` 的值的增长，这就是进度条的一种

## node 和 浏览器 eventLoop 的主要区别

两者最主要的区别在于浏览器中的微任务是在每个相应的宏任务中执行的，而 nodejs 中的微任务是在不同阶段之间执行的。

## 总结

1. 微任务队列优先于宏任务队列执行;

2. 微任务队列上创建的宏任务会被后添加到当前宏任务队列的尾端;

3. 微任务队列中创建的微任务会被添加到微任务队列的尾端;

4. 只要微任务队列中还有任务，宏任务队列就只会等待微任务队列执行完毕后再执行;

5. 只有运行完 await 语句，才把 await 语句后面的全部代码加入到微任务行列;

6. 在遇到 await promise 时，必须等 await promise 函数执行完毕才能对 await 语句后面的全部代码加入到微任务中;
   o 在等待 await Promise.then 微任务时:

&ensp;&ensp;&ensp;&ensp;- 运行其他同步代码;

&ensp;&ensp;&ensp;&ensp;- 等到同步代码运行完，开始运行 await promise.then 微任务;

&ensp;&ensp;&ensp;&ensp;- await promise.then 微任务完成后，把 await 语句后面的全部代码加入到微任务行列;

## 参考资料

[https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/EventLoop](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/EventLoop)

[https://zh.javascript.info/event-loop](https://zh.javascript.info/event-loop)

[http://www.ruanyifeng.com/blog/2013/10/event_loop.html](http://www.ruanyifeng.com/blog/2013/10/event_loop.html)

[https://juejin.cn/post/6844903512845860872](https://juejin.cn/post/6844903512845860872)

```

```
