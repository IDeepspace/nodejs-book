## 异步流程控制

我们都知道流程控制是程序里最常见的用于控制逻辑的统称。那为什么在 `Node.js` 里就变成了异步流程控制了呢？

这和 `Node.js` 本身都是异步的有关。如果每个函数都是异步的，性能好了，后遗症就是 `callback hell`（俗称回调地狱），为了解决 `callback hell` 问题，在 `Node.js` 用于流程控制的部分称为异步流程控制。

`Node.js` 从一开始就破旧立新，导致它经常被黑的 `callback hell` 问题。但也正是因为它的 `callback hell ` 问题，使得 `Node.js` 里的异步流程控制发展的特别快，比如 `thunk` 实现，`promise/a+` 规范的落地，`ES6` 中的 `generator` 和为了 `generator` 而写的 `co` 模块，以及目前最看好的 `async` 函数。各种 `Node.js` 的辅助模块非常多。

对于 `Node.js` 的异步流程控制，我们主要讲三个部分：

- Promise
- Generator/yield
- Async/await

 