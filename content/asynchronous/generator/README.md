## Generator

### 一、介绍

`Generator` 函数是 `ES6` 提供的一种异步编程解决方案，语法行为与传统函数完全不同。

`Generator` 函数是一个可以根据用户需求在不同时间间隔返回多个值，并能管理其内部状态的函数。如果一个函数使用 `Function*` 语法，则该函数就变成了一个 `Generator` 函数。

执行 `Generator` 函数会返回一个遍历器对象，也就是说，`Generator` 函数除了状态机，还是一个遍历器对象生成函数。返回的遍历器对象，可以依次遍历 `Generator` 函数内部的每一个状态。

形式上，`Generator` 函数是一个普通函数，但是有两个特征。

- `function` 关键字与函数名之间有一个星号；
- 函数体内部使用 `yield` 表达式，定义不同的内部状态（`yield` 在英语里的意思就是“产出”）。



### 二、使用

看看代码：

```javascript
function* helloWorldGenerator() {
  yield 'hello';
  yield 'world';
  return 'ending';
}

const hw = helloWorldGenerator();
```

上面代码定义了一个 `Generator` 函数 `helloWorldGenerator`，它内部有两个 `yield` 表达式（`hello` 和`world`），即该函数有三个状态：`hello`，`world` 和 `return` 语句（结束执行）。

然后，`Generator` 函数的调用方法与普通函数一样，也是在函数名后面加上一对圆括号。不同的是，调用 `Generator` 函数后，该函数并不执行，返回的也不是函数运行结果，而是一个指向内部状态的指针对象 —— 遍历器对象（`Iterator Object`）。

下一步，必须调用遍历器对象的 `next` 方法，使得指针移向下一个状态。也就是说，每次调用 `next` 方法，内部指针就从函数头部或上一次停下来的地方开始执行，直到遇到下一个 `yield` 表达式（或 `return` 语句）为止。换言之，`Generator` 函数是分段执行的，`yield` 表达式是暂停执行的标记，而 `next` 方法可以恢复执行。

```javascript
hw.next()
// { value: 'hello', done: false }

hw.next()
// { value: 'world', done: false }

hw.next()
// { value: 'ending', done: true }

hw.next()
// { value: undefined, done: true }
```

上面代码一共调用了四次 `next` 方法。

- 第一次调用，`Generator` 函数开始执行，直到遇到第一个`yield` 表达式为止。`next `方法返回一个对象，它的`value `属性就是当前 `yield` 表达式的值 `hello`，`done  ` 属性的值 `false`，表示遍历还没有结束。

- 第二次调用，`Generator` 函数从上次 `yield` 表达式停下的地方，一直执行到下一个 `yield` 表达式。`next` 方法返回的对象的 `value` 属性就是当前 `yield` 表达式的值 `world`，`done` 属性的值 `false`，表示遍历还没有结束。

- 第三次调用，`Generator` 函数从上次 `yield` 表达式停下的地方，一直执行到 `return` 语句（如果没有 `return` 语句，就执行到函数结束）。`next` 方法返回的对象的 `value` 属性，就是紧跟在 `return` 语句后面的表达式的值（如果没有 `return` 语句，则 `value` 属性的值为 `undefined`），`done` 属性的值 `true`，表示遍历已经结束。

- 第四次调用，此时 `Generator` 函数已经运行完毕，`next` 方法返回对象的 `value` 属性为 `undefined`，`done` 属性为 `true`。以后再调用 `next` 方法，返回的都是这个值。

总结一下，调用 `Generator` 函数，返回一个遍历器对象，代表 `Generator` 函数的内部指针。以后，每次调用遍历器对象的 `next` 方法，就会返回一个有着 `value` 和 `done` 两个属性的对象。`value` 属性表示当前的内部状态的值，是 `yield` 表达式后面那个表达式的值；`done` 属性是一个布尔值，表示是否遍历结束。

`ES6` 没有规定，`function ` 关键字与函数名之间的星号，写在哪个位置。这导致下面的写法都能通过。

```javascript
function * foo(x, y) { ··· }
function *foo(x, y) { ··· }
function* foo(x, y) { ··· }
function*foo(x, y) { ··· }
```

由于 `Generator` 函数仍然是普通函数，所以一般的写法是上面的第三种，即星号紧跟在 `function` 关键字后面。



### 三、for...of 循环

`for...of ` 循环可以自动遍历 `Generator` 函数运行时生成的 `Iterator` 对象，且此时不再需要调用 `next` 方法。

```javascript
function* foo() {
  yield 1;
  yield 2;
  yield 3;
  yield 4;
  yield 5;
  return 6;
}

for (let v of foo()) {
  console.log(v);
}
// 1 2 3 4 5
```

上面代码使用 `for...of` 循环，依次显示 5 个 `yield` 表达式的值。

> 这里需要注意，一旦 `next` 方法的返回对象的 `done` 属性为 `true`，`for...of` 循环就会中止，且不包含该返回对象，所以上面代码的 `return` 语句返回的 `6`，不包括在 `for...of` 循环之中。



### 四、异步任务的封装

`Generator` 函数的暂停执行的效果，意味着可以把异步操作写在 `yield ` 表达式里面，等到调用 `next` 方法时再往后执行。这实际上等同于不需要写回调函数了，因为异步操作的后续操作可以放在 `yield` 表达式下面，反正要等到调用 `next` 方法时再执行。所以，`Generator` 函数的一个重要实际意义就是用来处理异步操作，改写回调函数。

```javascript
function* loadUI() {
  showLoadingScreen();
  yield loadUIDataAsynchronously();
  hideLoadingScreen();
}
const loader = loadUI();
// 加载UI
loader.next()

// 卸载UI
loader.next()
```

上面代码中，第一次调用 `loadUI` 函数时，该函数不会执行，仅返回一个遍历器。下一次对该遍历器调用 `next` 方法，则会显示 `Loading` 界面（`showLoadingScreen`），并且异步加载数据（`loadUIDataAsynchronously`）。等到数据加载完成，再一次使用 `next` 方法，则会隐藏`Loading`界面。可以看到，这种写法的好处是所有 `Loading` 界面的逻辑，都被封装在一个函数，按部就班非常清晰。

`Ajax` 是典型的异步操作，通过 `Generator` 函数部署 `Ajax` 操作，可以用同步的方式表达，接下来会使用 `setTimeout` 来模拟异步任务。

```javascript
const person = sex => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      const data = {
        sex,
        name: 'Deepspace',
        height: 176
      }
      resolve(data)
    }, 2000)
  })
}
function* gen() {
  const data = yield person('boy')
  console.log(data)
}
const g = gen()
const next1 = g.next() // { value: Promise { <pending> }, done: false }
next1.value.then(data => {
  g.next(data)
})
```

从上面代码可以看出，第一次调用 `next` 方法时，启动了遍历器对象，此时返回了包含 `value` 和 `done` 属性的对象，由于 `value` 属性值是 `promise` 对象，因此可以使用 `then` 方法获取到 `resolve` 传递过来的值，再使用带有 `data` 参数的 `next` 方法给上一个 `yield` 表达式传递返回值。

此时在 `const data = yield person()` 这句语句中，就可以得到异步任务传递的参数值了，实现了异步任务的同步化。

上面的代码实质上就是每次都会重复使用 `value` 属性值和 `next` 方法，所以每次使用 `Generator` 实现异步都会涉及到流程控制的问题。每次都手动实现流程控制会显得麻烦，有没有什么办法可以实现自动流程控制呢？

### 五、thunk函数实现流程控制

`thunk` 函数实际上有些类似于 `JavaScript` 函数柯里化，会将某个函数作为参数传递到另一个函数中，然后通过闭包的方式为参数(函数)传递参数进而实现求值。

函数柯里化实现的过程如下：

```javascript
function curry (fn) {
  const args1 = Array.prototype.slice.call(arguments, 1)
  return function () {
    const args2 = Array.from(arguments)
    const arr = args1.concat(args2)
    return fn.apply(this, arr)
  }
}
```

使用 `curry` 函数来举一个例子:

```javascript
// 需要柯里化的sum函数
const sum = (a, b) => {
  return a + b
}
curry(sum, 1)(2)   // 3
```

而 `thunk` 函数简单的实现思路如下：

```javascript

// ES5实现
const thunk = fn => {
  return function () {
    const args = Array.from(arguments)
    return function (callback) {
      args.push(callback)
      return fn.apply(this, args)
    }
  }
}

// ES6实现
const thunk = fn => {
  return function (...args) {
    return function (callback) {
      return fn.call(this, ...args, callback)
    }
  }
}
```

从上面 `thunk` 函数中，会发现，`thunk` 函数比函数 `curry` 化多用了一层闭包来封装函数作用域。

使用上面的 `thunk` 函数，可以生成 `fs.readFile` 的 `thunk` 函数。

```javascript
const fs = require('fs')
const readFileThunk = thunk(fs.readFile)
readFileThunk(fileA)(callback)
```

使用 `thunk` 函数将 `fs.readFile` 包装成 `readFileThunk` 函数，然后在通过 `fileA` 传入文件路径，`callback ` 参数则为 `fs.readFile` 的回调函数。

当然，还有一个 `thunk` 函数的升级版本 `thunkify` 函数，可以使得回调函数只执行一次。原理和上面的 `thunk` 函数非常像，只不过多了一个 `flag` 参数用于限制回调函数的执行次数。源码地址: [node-thunkify](https://github.com/tj/node-thunkify)

```javascript
const thunkify = fn => {
  return function () {
    const args = Array.from(arguments)
    return function (callback) {
      let called = false
      // called变量限制callback的执行次数
      args.push(function () {
        if (called) return
        called = true
        callback.apply(this, arguments)
      })
      try {
        fn.apply(this, args)
      } catch (err) {
        callback(err)
      }
    }
  }
}
```

举个例子看看: 

```javascript
function sum(a, b, callback) {
  const total = a + b
  console.log(total)
  console.log(total)
}

// 如果使用thunkify函数
const sumThunkify = thunkify(sum)
sumThunkify(1, 2)(console.log)
// 打印出3

// 如果使用thunk函数
const sumThunk = thunk(sum)
sumThunk(1, 2)(console.log)
// 打印出 3, 3
```

再来看一个使用 `setTimeout` 模拟异步并且使用 `thunkify` 模块来完成异步任务同步化的例子：

```javascript
const person = (sex, fn) => {
  setTimeout(() => {
    const data = {
      sex,
      name: 'Deepspace',
      height: 176
    }
    fn(data)
  }, 2000)
}
const personThunk = thunkify(person)
function* gen() {
  const data = yield personThunk('boy')
  console.log(data)
}
const g = gen()
const next = g.next()
next.value(data => {
  g.next(data)
})
```

从上面代码可以看出，`value` 属性实际上就是 `thunkify` 函数的回调函数(也是 `person` 的第二个参数)，而` boy`则是 `person` 的第一个参数。

### 六、Generator函数的自动流程控制

在上面的代码中，我们可以将调用遍历器对象生成函数，返回遍历器和手动执行 `next` 方法以恢复函数执行的过程封装起来。

```javascript
const run = gen => {
  const g = gen()
  const next = data => {
    let result = g.next(data)
    if (result.done) return result.value
    result.value(next)
  }
  next()
}
```

使用 `run` 函数封装起来之后，`run` 内部的 `next` 函数实际上就是 `thunk(thunkify)` 函数的回调函数了。因此，调用 `run` 即可实现 `Generator` 的自动流程控制。

```javascript
const person = (sex, fn) => {
  setTimeout(() => {
    const data = {
      sex,
      name: 'Deepspace',
      height: 176
    }
    fn(data)
  }, 2000)
}
const personThunk = thunkify(person)
function* gen() {
  const data = yield personThunk('boy')
  console.log(data)
}
run(gen)
// { sex: 'boy', name: 'Deepspace', height: 176 }
```

有了这个执行器，执行 `Generator` 函数就方便多了。不管内部有多少个异步操作，直接把 `Generator` 函数传入` run` 函数即可。当然，前提是每一个异步操作，都要是 `thunk(thunkify)` 函数。也就是说，跟在 `yield` 表达式后面的必须是 `thunk(thunkify)` 函数。

```javascript
const gen = function *gen () {
  const f1 = yield personThunk('boy') // 跟在yield表达式后面的异步行为必须使用thunk(thunkify)函数封装
  const f2 = yield personThunk('boy')
  // ...
  const fn = yield personThunk('boy')
}
run(gen)  // run函数的自动流程控制
```

上面代码中，函数 `gen` 封装了 `n` 个异步行为，只要执行 `run` 函数，这些操作就会自动完成。这样一来，异步操作不仅可以写得像同步操作，而且一行代码就可以执行。

### 七、co模块的自动流程控制

在上面的例子说过，表达式后面的值必须是 `thunk(thunkify)` 函数，这样才能实现 `Generator` 函数的自动流程控制。`thunk` 函数的实现是基于回调函数的，而 `co` 模块则更进一步，可以兼容 `thunk` 函数和 `Promise` 对象。先来看看 `co` 模块的基本用法：

```javascript
const co = require('co')
const gen = function *gen () {
  const f1 = yield person('boy') // 调用person，返回一个promise对象
  const f2 = yield person('boy')
}
co(gen)   // 将thunk(thunkify)函数和run函数封装成了co模块，yield表达式后面可以是thunk(thunkify)函数或者Promise对象
```

`co` 模块可以不用编写 `Generator` 函数的执行器，因为它已经封装好了。将 `Generator` 函数co模块中，函数就会自动执行。

`co` 函数返回一个 `Promise` 对象，因此可以用 `then` 方法添加回调函数。

```javascript
co(gen).then(function (){
  console.log('Generator 函数执行完成')
})
```

> **co模块原理**：co模块其实就是将两种自动执行器(`thunk(thunkify)` 函数和 `Promise` 对象)，包装成一个模块。使用 `co` 模块的前提条件是，`Generator` 函数的 `yield` 表达式后面，只能是 `thunk(thunkify)` 或者`Promise` 对象，如果是数组或对象的成员全部都是 `promise` 对象，也可以使用 `co` 模块。



### 八、基于Promise对象的自动执行

还是使用上面例子，不过这次是将回调函数改成 `Promise` 对象来实现自动流程控制。

```javascript
const person = (sex, fn) => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      const data = {
        name: 'Deepspace',
        height: 176
      }
      resolve(data)
    }, 2000)
  })
}
function* gen() {
  const data = yield person('boy')
  console.log(data) // { name: 'Deepspace', height: 176 }
}
const g = gen()
g.next().value.then(data => {
  g.next(data)
})
```

手动执行实际上就是层层使用 `then` 方法和 `next` 方法。根据这个可以写出自动执行器。

```javascript
const run = gen => {
  const g = gen()
  const next = data => {
    let result = g.next(data)
    if (result.done) return result.value
    result.value.then(data => {
      next(data)
    })
  }
  next()
}
run(gen)
```

关于 `Generator` 异步应用的相关知识也就差不多了，现在稍微总结一下。

1. 由于 `yield` 表达式可以暂停执行，`next` 方法可以恢复执行，这使得 `Generator` 函数很适合用来将异步任务同步化。
2. 但是 `Generator` 函数的流程控制会稍显麻烦，因为每次都需要手动执行 `next` 方法来恢复函数执行，并且向`next` 方法传递参数以输出上一个yiled表达式的返回值。
3. 于是就有了 `thunk(thunkify)` 函数和 `co` 模块来实现 `Generator` 函数的自动流程控制。
4. 通过 `thunk(thunkify)` 函数分离参数，以闭包的形式将参数逐一传入，再通过 `apply` 或者 `call` 方法调用，然后配合使用 `run` 函数可以做到自动流程控制。
5. 通过 `co` 模块，实际上就是将 `run` 函数和 `thunk(thunkify)` 函数进行了封装，并且 `yield` 表达式同时支持 `thunk(thunkify)` 函数和 `Promise` 对象两种形式，使得自动流程控制更加的方便。

