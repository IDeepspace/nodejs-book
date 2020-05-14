## 浏览器中的事件循环机制

### 一、单线程和异步

提到 `JavaScript` ，就会想到它的 **单线程** 和 **异步** 两大特性。那么在 `JavaScript` 中单线程是如何做到异步的呢？我们先了解一下单线程和异步之间的关系。

`JavaScript` 中的任务分为 **同步** 和 **异步** 两种，它们的处理方式存在很大的不同。

#### 1、异步

**同步任务** 是直接在 **主线程** 上排队执行。在主线程上排队执行的任务，只有前一个任务执行完毕，才能执行后一个任务。所有同步任务都在主线程上执行，形成一个 **执行栈**（`execution context stack`）。

**异步任务** 则是被放在 **任务队列** (`task queue`) 中，不进入主线程，只有 **任务队列** 通知主线程，某个异步任务可以执行了，该任务才会进入主线程执行。任务队列类似一个缓冲区，任务下一步会被移到 **执行栈**，然后主线程执行 **执行栈 **的任务。如果有多个异步任务，那这些异步任务就要在任务队列中排队等候。

<!-- more -->

主线程和任务队列的示意图：

<img src="https://raw.githubusercontent.com/IDeepspace/ImageHosting/master/NodeJS/main-thread-and-task-queue.jpg" alt="主线程和任务队列" style="zoom: 77%;" />

<p align="center">(图片来自网络)</p>

#### 2、单线程

`JavaScript` 中其实是没有线程概念的，所谓的单线程也只是相对于多线程而言。**线程** 本来就不属于 `JavaScript` 语言的范畴，只是为了和其它语言做对比而已，为了让那些用过多线程语言的人理解 `JavaScript` 的特点。

单线程是指 `JavaScript` 引擎中负责解析执行 `JavaScript` 代码的线程只有一个（**主线程**），即每次只能做一件事。

> 注意：所谓的 `JavaScript` 是单线程的，是指 `JavaScript` 运行在浏览器中是单线程的，叫做 ` JavaScript` 引擎线程。但是浏览器不是单线程的。

浏览器是**事件驱动（`Event driven`）**的。浏览器中很多异步行为都是由浏览器新开一个线程去完成的，它会创建事件并放入执行队列中。`JavaScript` 引擎是单线程处理任务队列，它只是浏览器多个线程中的一个。所以当多个事件触发时，会依次放入队列，然后一个一个响应。

浏览器还包括很多其他线程，如界面渲染线程，浏览器事件触发线程，`Http` 请求线程等。

怎么证明 `JavaScript` 是单线程的呢？ 看段代码：

```javascript
function foo() {
    console.log("first");
    setTimeout((function () {
        console.log('second');
    }), 5);
}

for (let i = 0; i < 1000000; i++) {
    foo();
}
```

上面的代码中，`JavaScript` 引擎 `5ms` 后会把打印 `second` 的任务加入队列，而当前有任务，所以只能等 `1000000` 个 `first` 打印完后才会打印 `second` 。所以执行结果会首先全部打印 `first` ，然后再全部打印 `second` ，即使中间的执行时间超过 `5ms` 。

由此可见，`JavaScript` 是单线程的。

#### 3、ajax 请求

我们知道一个当发出 `ajax` 请求的时候，主线程在等待它响应的同时是会去做其它事情的。浏览器先在事件表中注册 `ajax` 的回调函数，响应回来后回调函数被添加到任务队列中等待执行，不会造成线程阻塞，所以说 `JavaScript` 处理 `ajax` 请求的方式是异步的。



### 二、事件循环

根据上面的描述，`JavaScript` 检查执行栈是否为空，以及确定把哪个 `task` 加入执行栈的这个过程就是 **事件循环**，`JavaScript` 实现异步的核心就是事件循环。

<img src="https://raw.githubusercontent.com/IDeepspace/ImageHosting/master/NodeJS/eventLoop-browser.png" alt="事件循环" style="zoom:77%;" />

<p align="center">(图片来自网络)</p>

主线程运行的时候，产生堆（`heap`）和栈（`stack`），栈中的代码调用各种外部 `API`，它们在 **任务队列** 中加入各种事件（`click`，`load`，`done` 等）。只要执行栈中的代码执行完毕，主线程就会去读取 **任务队列** ，依次执行那些事件所对应的回调函数。

> **注意：执行栈中的代码（同步任务），总是在读取 "任务队列"（异步任务）之前执行。** 



### 三、在浏览器中的实现

浏览器中，一个事件循环里有很多个来自不同任务源的任务队列（`task queues`），每一个任务队列里的任务是严格按照**先进先出**的顺序执行的。但是，因为**浏览器自己调度**的关系，**不同任务队列的任务的执行顺序是不确定**的。举个例子：

```
浏览器有一个处理鼠标和键盘事件的任务队列。浏览器可以给这个队列比其他队列多3/4的执行时间，以确保交互的响应而不让其他任务队列饿死（starving），并且不会乱序处理任何一个任务队列的事件。
```

任务被分为 `Task`（又称为 `MacroTask`，宏任务）和 `MicroTask`（微任务）两种。那哪些行为属于 `MacroTask`，哪些又属于 `MicroTask` 呢？

- **MacroTask**：`script`（整体代码）， `setTimeout`， `setInterval`， `setImmediate`（`node`独有）， `I/O`， `UI rendering` ;
- **MicroTask**：`process.nextTick`（`node`独有）， `Promises`，`Object.observe`（废弃）， `MutationObserver` 。

关于 `macrotask` 和 `microtask` 的理解，光这样描述会有些晦涩难懂，结合事件循坏的机制理解会清晰很多，下面这张图可以说是介绍得非常清楚了：

<img src="https://raw.githubusercontent.com/IDeepspace/ImageHosting/master/NodeJS/eventLoop-browser-1.jpg" alt="事件循环" style="zoom:87%;" />

<p align="center">(图片来自网络)</p>

总结一下上图，**事件循环的进程模型**：

- 浏览器会不断从 `macrotask` 队列中按顺序取 `macrotask` 执行
- 每执行完一个 `macrotask ` 都会检查 `microtask` 队列是否为空（执行完一个 `macrotask` 的具体标志是函数执行栈为空）
- 如果 `microtask` 队列不为空，则会**一次性执行完**所有 `microtask`，直到 `microtasks` 队列全部清空。
- 更新界面渲染
- 返回第一步，再进入下一个循环去 `macrotask` 队列中取下一个 `macrotask` 执行

`macrotask` 和 `microtask` 的执行顺序：

<img src="https://raw.githubusercontent.com/IDeepspace/ImageHosting/master/NodeJS/eventLoop-browser-2.png" alt="事件循环" style="zoom:77%;" />

<p align="center">(图片来自网络)</p>

**这里需要 `highlight` 的是：`microtask queue` 中的 `task` 会在事件循环的当前回合中执行，因此 `macrotask queue` 中的 `task` 就只能等到事件循环的下一个回合中执行了。**



> 注意：图中橙色的 MacroTask 任务队列也应该是在不断被切换着的。

我们看段代码：

```javascript
console.log('start');

setTimeout(function() {
  console.log('setTimeout');
}, 0);

Promise.resolve().then(function() {
  console.log('promise1');
}).then(function() {
  console.log('promise2');
});

console.log('end');
```

> 代码来自 [Tasks, microtasks, queues and schedules](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/) .

在这里，`setTimeout` 的延时为 0，而 `Promise.resolve()` 也是返回一个被 `resolve` 了 `promise` 对象，即这里的 `then` 方法中的函数也是相当于异步的立即执行任务，那么他们到底是谁在前谁在后？

正确的打印顺序是：

```
start
end
promise1
promise2
setTimeout
```

**这里的运行结果是 `Promise` 的立即返回的异步任务会优先于 `setTimeout` 延时为 0 的任务执行。**

看看执行过程：

![事件循环](https://raw.githubusercontent.com/IDeepspace/ImageHosting/master/NodeJS/browser-event-loop-excute-animate.gif)

<p align="center">(图片来自网络)</p>

- 一开始 `task` 队列中只有 `script`，则 `script` 中所有函数放入函数执行栈执行，代码按顺序执行。

- 接着遇到了 `setTimeout`**，它的作用是 `0ms` 后将回调函数放入 `task` 队列中**，**也就是说这个函数将在下一个事件循环中执行。**

- 接着遇到了 `Promise`，按照前面所述 `Promise` 属于 `microtask` ，所以第一个 `.then()` 会放入 `microtask` 队列。

- 当所有 `script` 代码执行完毕后，**此时函数执行栈为空**。 **`microtask queue` 中的 `task` 会在事件循环的当前回合中执行。**所以，开始检查 `microtask` 队列，此时 `microtask` 队列不为空，执行 `.then()` 的回调函数输出 `'promise1'`，由于 `.then()` 返回的依然是 `promise` ，所以第二个 `.then()` 会放入 `microtask` 队列继续执行，输出 `'promise2'` 。也就是说如果我的某个 `microtask` 任务又推入了一个任务进入 `microtasks` 队列，那么在主线程完成该任务之后，仍然会继续运行 `microtasks` 任务，直到任务队列耗尽。

- 此时 `microtask` 队列为空了，进入下一个事件循环，检查 `task` 队列发现了 `setTimeout` 的回调函数，立即执行回调函数输出 `'setTimeout'`，代码执行完毕。
- 这个过程会不断重复，也就是所谓的**事件循环**。



#### 1、Promise 和 async 中的立即执行

下面再看个例子：

```javascript
async function async1() {
    console.log('async1 start');
    await async2();
    console.log('async1 end');
}
async function async2() {
	console.log('async2');
}

console.log('script start');

setTimeout(function() {
    console.log('setTimeout');
}, 0)

async1();

new Promise(function(resolve) {
    console.log('promise1');
    resolve();
}).then(function() {
    console.log('promise2');
});
console.log('script end');

/*
script start
async1 start
async2
promise1
script end
async1 end
promise2
setTimeout
*/
```

**解析：**

我们知道 `Promise` 中的异步体现在 `then` 和 `catch` 中，所以写在 `Promise` 中的代码是被当做同步任务立即执行的。而在 `async/await` 中，在出现 `await` 出现之前，其中的代码也是立即执行的。

因为 `async await` 本身就是 `promise+generator` 的语法糖。所以 `await` 后面的代码是 `microtask`。所以，例子中的：

```javascript
async function async1() {
	console.log('async1 start');
	await async2();
	console.log('async1 end');
}
```

等价于：

```javascript
async function async1() {
	console.log('async1 start');
	Promise.resolve(async2()).then(() => {
    console.log('async1 end');
  })
}
```

`async` 函数中在 `await` 之前的代码是立即执行的，所以会立即输出 `async1 start`。

遇到了 `await` 时，会将 `await` 后面的表达式执行一遍，所以就紧接着输出 `async2`，然后将 `await` 后面的代码也就是 `console.log('async1 end')` 加入到 `microtask` 中的 `Promise` 队列中，接着跳出 `async1` 函数来执行后面的代码。

`script` 任务继续往下执行，遇到 `Promise` 实例（注意是实例）。由于 `Promise` 中的函数是立即执行的，而后续的 `.then` 则会被分发到 `microtask` 的 `Promise` 队列中去。所以会先输出 `promise1`，然后执行 `resolve`，将 `promise2` 分配到对应队列。

`script` 任务继续往下执行，最后只有一句输出了 `script end`，至此，全局任务就执行完毕了。

根据上述，每次执行完一个宏任务之后，会去检查是否存在 `Microtasks`；如果有，则执行 `Microtasks` 直至清空 `Microtask Queue`。

因而在 `script` 任务执行完毕之后，开始查找清空微任务队列。此时，微任务中， `Promise` 队列有的两个任务 `async1 end` 和 `promise2`，因此按先后顺序输出 `async1 end，promise2`。当所有的 `Microtasks` 执行完毕之后，表示第一轮的循环就结束了。

第二轮循环依旧从宏任务队列开始。此时宏任务中只有一个 `setTimeout`，取出直接输出即可，至此整个流程结束。



#### 2、变式

```javascript
async function async1() {
    console.log('async1 start');
    await async2();
    console.log('async1 end');
}
async function async2() {
    //async2做出如下更改：
    new Promise(function(resolve) {
    console.log('promise1');
    resolve();
}).then(function() {
    console.log('promise2');
   });
}
console.log('script start');

setTimeout(function() {
    console.log('setTimeout');
}, 0)
async1();

new Promise(function(resolve) {
    console.log('promise3');
    resolve();
}).then(function() {
    console.log('promise4');
});

console.log('script end');

/*
script start
async1 start
promise1
promise3
script end
promise2
async1 end
promise4
setTimeout
*/
```

在第一次 `macrotask` 执行完之后，也就是输出 `script end` 之后，会去清理所有 `microtask`。所以会相继输出 `promise2`，` async1 end` ，`promise4`。





### 四、定时器

除了放置异步任务的事件，**任务队列** 中还可以放置定时事件，即指定某些代码在多少时间之后执行。这叫作 **定时器（`timer`）**功能，也就是定时执行的代码。

定时器功能主要由 `setTimeout()` 和 `setInterval()` 这两个函数来完成，它们两个的内部运行机制完全一样，区别在于前者指定的代码是一次性执行，后者则为反复执行。下面看段代码：

```javascript
console.log(1);
setTimeout(function () { console.log(2); }, 1000);
console.log(3);
```

输出结果为：

```javascript
1
3
2
```

`setTimeout()` 将第二行代码推迟到 `1000ms ` 之后执行。

写 `JavaScript` 代码的时候，我们会经常将 `setTimeout()` 的第二个参数设为 `0`，就表示当前代码执行完（执行栈清空）以后，立即执行（ `0ms` 间隔）指定的回调函数。

**注意**：`setTimeout()` 只是将事件插入了 **任务队列**，必须等到当前代码（执行栈）执行完，主线程才会去执行它指定的回调函数。如果当前代码耗时很长，有可能要等很久，所以并没有办法保证回调函数一定会在 `setTimeout()` 指定的时间执行。