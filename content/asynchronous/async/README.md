## async 和 await

### 一、简介

`async` 函数是 `Generator` 函数的语法糖。

下面是一个 `Generator` 函数，依次读取两个文件：	

```javascript
const fs = require('fs');

const readFile = function (fileName) {
  return new Promise(function (resolve, reject) {
    fs.readFile(fileName, function(error, data) {
      if (error) return reject(error);
      resolve(data);
    });
  });
};

const gen = function* () {
  const f1 = yield readFile('/etc/fstab');
  const f2 = yield readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};
```

上面代码的函数 `gen` 可以写成 `async` 函数：

```javascript
const asyncReadFile = async function () {
  const f1 = await readFile('/etc/fstab');
  const f2 = await readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};
```

比较一下就会发现，`async` 函数就是将 `Generator` 函数的星号（`*`）替换成 `async`，将 `yield` 替换成 `await`。比起星号和 `yield`，`async` 和 `await`，语义上更清楚了。

**语法：**

`await` 只能出现在 `async` 函数中，`async` 用于申明一个 `function` 是异步的，而 `await` 用于等待一个异步方法执行完成。下面我们单独对 `async` 和 `await` 做一些介绍，帮助理解。



### 二、async

`async` 函数是怎么处理它的返回值的呢？我们先写段代码来试试，看它到底会返回什么：

`index.js` :

```javascript
async function testAsync() {
  return "hello async";
}

const result = testAsync();
console.log(result);
```

打印结果：

```javascript
> node index.js
Promise { 'hello async' }
```

输出的是一个 `Promise` 对象。`async` 函数（包含函数语句、函数表达式、`Lambda` 表达式）会返回一个 `Promise` 对象，如果在函数中 `return` 一个直接量，`async` 会把这个直接量通过  `Promise.resolve()` 封装成 `Promise` 对象。

由于 `async` 函数返回的是一个 `Promise` 对象，所以在最外层不能用 `await` 获取其返回值的情况下，我们可以用原来的方式：`then()` 链来处理这个 `Promise` 对象，像下面这样：

```javascript
async function testAsync() {
  return "hello async";
}

testAsync().then(v => {
  console.log(v);    // hello async
});
```

那如果 `async` 函数没有返回值，会怎么样呢？很容易想到，它会返回 `Promise.resolve(undefined)`。

联想一下 `Promise` 的特点——无等待，所以在没有 `await` 的情况下执行 `async` 函数，它会立即执行，返回一个  `Promise`  对象，并且不会阻塞后面的语句。这和普通返回 `Promise` 对象的函数是一样的。



### 三、await

`await`  操作符用于等待一个[`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise) 对象。它只能在异步函数 [`async function`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/async_function) 中使用。

因为 `async` 函数返回一个 `Promise` 对象，所以 `await` 可以用于等待一个 `async` 函数的返回值——这也可以说是 `await` 在等 `async` 函数。

但要清楚，`await` 等的实际是一个返回值 —— 一个 [`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise) 对象或者任何要等待的值， `await` 不仅仅用于等 `Promise` 对象，它可以等任意表达式的结果，所以，`await` 后面实际是可以接普通函数调用或者直接量的。

```javascript
function getSomething() {
  return "something";
}

async function testAsync() {
  return Promise.resolve("hello async");
}

async function test() {
  const v1 = await getSomething();
  const v2 = await testAsync();
  console.log(v1, v2);
}

test();
```

`await` 等到了它要等的东西 —— 一个 `Promise` 对象，或者其它值，然后呢？首先我们需要明确的是：**`await` 是个运算符，用于组成表达式，`await` 表达式的运算结果取决于它等的东西。**

- 如果 `await` 等到的不是一个 `Promise` 对象，那 `await` 表达式的运算结果就是它等到的东西。

- 如果它等到的是一个 `Promise` 对象，`await` 就忙起来了，它会阻塞后面的代码，等着 `Promise` 对象 `resolve`，然后得到 `resolve` 的值，作为 `await` 表达式的运算结果。这就是 `await` 必须用在 `async` 函数中的原因，`async` 函数调用不会造成阻塞，它内部所有的阻塞都被封装在一个 `Promise` 对象中异步执行。



### 四、async/await 的优势

#### 1、简洁

使用 `async/await` 明显节约了不少代码。我们：

- 不需要写 `.then`；
- 不需要写匿名函数处理 `Promise` 的 `resolve` 值；
- 不需要定义多余的 `data` 变量；
- 避免了嵌套代码。



#### 2、错误处理

`async/await` 使得最终可以使用相同的代码结构来处理同步和异步错误。

在下面带有 `promises` 的示例中，如果 `JSON.parse` 失败，则 `try/catch` 将无法处理，因为它发生在 `promise` 中。我们需要在 `promise` 上调用 `.catch` 并复制我们的错误处理代码，这会让代码变得非常冗杂。在实际生产中，会更加复杂。

```javascript
const makeRequest = () => {
  try {
    getJSON().then(result => {
      // JSON.parse可能会出错
      const data = JSON.parse(result);
      console.log(data);
    });
    // 取消注释，处理异步代码的错误
    // .catch((err) => {
    //   console.log(err)
    // })
  } catch (err) {
    console.log(err);
  }
};
```

如果我们使用 `async/await` 的话，`catch`  就能很好地处理 `JSON.parse` 错误：

```javascript
const makeRequest = async () => {
  try {
    // JSON.parse可能会出错
    const data = JSON.parse(await getJSON());
    console.log(data);
  } catch (err) {
    console.log(err);
  }
};
```



#### 3、条件语句

下面的代码示例当中，异步请求 `getJSON` 获取数据，然后根据返回的数据来判断决定是直接返回还是继续获取更多的数据：

```javascript
const makeRequest = () => {
  return getJSON().then(data => {
    if (data.needsAnotherRequest) {
      return makeAnotherRequest(data).then(moreData => {
        console.log(moreData);
        return moreData;
      });
    } else {
      console.log(data);
      return data;
    }
  });
};
```

上面的代码看着就会觉得很头疼，嵌套了 6 层。`return` 语句很容易让人感到迷茫，而它们只是需要将最终结果传递到最外层的 `Promise`。如果我们使用 `async/await` 来改写代码，代码的可读性会大大提高：

```javascript
const makeRequest = async () => {
  const data = await getJSON();
  if (data.needsAnotherRequest) {
    const moreData = await makeAnotherRequest(data);
    console.log(moreData);
    return moreData;
  } else {
    console.log(data);
    return data;
  }
};
```



#### 4、中间值

开发中我们经常会遇到这样一个场景：调用 `promise1`，使用 `promise1` 返回的结果去调用 `promise2`，然后使用两者的结果去调用 `promise3`。你的代码很可能是这样的:

```javascript
const makeRequest = () => {
  return promise1()
    .then(value1 => {
      // do something
      return promise2(value1)
        .then(value2 => {
          // do something          
          return promise3(value1, value2)
        })
    })
}
```

我们可以做些改变，减少嵌套：将 `value1` 和 `value2` 放进 `Promise.all` 来避免深层嵌套：

```javascript
const makeRequest = () => {
  return promise1()
    .then(value1 => {
      // do something
      return Promise.all([value1, promise2(value1)])
    })
    .then(([value1, value2]) => {
      // do something          
      return promise3(value1, value2)
    })
}
```

代码看起来减少了嵌套，但是为了可读性却牺牲了语义。除了避免嵌套，并没有其他理由将 `value1` 和 `value2`  放在一个数组中。

如果我们使用 `async/await` 的话，代码会变得非常简单、直观：

```javascript
const makeRequest = async () => {
  const value1 = await promise1()
  const value2 = await promise2(value1)
  return promise3(value1, value2)
}
```



#### 5、错误栈

假设有一段代码在一个链中调用多个 `promise`，而在链的某个地方会抛出一个错误。

```javascript
const makeRequest = () => {
  return callAPromise()
    .then(() => callAPromise())
    .then(() => callAPromise())
    .then(() => callAPromise())
    .then(() => callAPromise())
    .then(() => {
      throw new Error("oops");
    })
}

makeRequest()
  .catch(err => {
    console.log(err);
    // output
    // Error: oops at callAPromise.then.then.then.then.then (index.js:8:13)
  })
```

`Promise` 链中返回的错误栈不会给出错误发生位置的详细原因。更糟糕的是，它会误导我们：错误栈中唯一的函数名为 `callAPromise`，然而它和错误没有关系。(当然文件名和行号还是有用的)。

然而，`async/await` 中的错误栈会指向错误所在的函数：

```javascript
const makeRequest = async () => {
  await callAPromise()
  await callAPromise()
  await callAPromise()
  await callAPromise()
  await callAPromise()
  throw new Error("oops");
}

makeRequest()
  .catch(err => {
    console.log(err);
    // output
    // Error: oops at makeRequest (index.js:7:9)
  })
```

在开发环境中，这一点优势可能并不大。但是，当我们分析生产环境的错误日志时，它将非常有用。



#### 6、调试

`async/await` 能够使得代码调试更简单。

由于我们不能在返回表达式的箭头函数中设置断点，所以如果使用 `Promise` 的话，就无法进行断点调试：

```javascript
const makeRequest = () => {
  return callAPromise()
    .then(() => callAPromise())
    .then(() => callAPromise())
    .then(() => callAPromise())
    .then(() => callAPromise())
    .then(() => {
      throw new Error("oops");
    })
}
```

并且，如果我们在 `.then`  代码块中设置断点，使用 `Step Over` 快捷键时，调试器不会跳到下一个 `.then`，它会跳过异步代码。

如果使用 `await/async`，我们就不再需要那么多箭头函数了，这样就可以像调试同步代码一样跳过 `await` 语句：

```javascript
const makeRequest = async () => {
  await callAPromise()
  await callAPromise()
  await callAPromise()
  await callAPromise()
  await callAPromise()
}
```



### 五、总结

- `async/await` 是写异步代码的新方式，以前的方法有**回调函数**和 `Promise`。
- `async/await` 是基于 `Promise` 实现的，它不能用于普通的回调函数。
- `async/await` 与 `Promise` 一样，是非阻塞的。
- `async/await` 使得异步代码看起来像同步代码，这正是它的魔力所在。
- `async/await` 相比于 `Promise` ，更加有优势。