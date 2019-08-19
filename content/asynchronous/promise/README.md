##Promise

`Promise` 是异步编程的一种解决方案，比传统的解决方案——回调函数和事件——更合理和更强大。它由社区最早提出和实现，`ES6` 将其写进了语言标准，统一了用法，原生提供了 `Promise` 对象。

所谓 `Promise`，简单说就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。从语法上说，`Promise`  是一个对象，从它可以获取异步操作的消息。`Promise` 提供统一的 `API`，各种异步操作都可以用同样的方法进行处理。

### 一、介绍 
学习一个东西，得先知道它是什么。我们先在浏览器中使用 `console.dir(Promise)` 打印出 `Promise` 对象的所的属性和方法。

![Promise](https://raw.githubusercontent.com/IDeepspace/ImageHosting/master/JavaScript/promise.png)

从打印结果可以看出，`Promise` 是一个构造函数，它自己本身有 `all`、`reject`、`resolve` 等方法，原型上有 `catch`、`finally`、`then` 等方法。所以 `new` 出来的 `Promise` 对象也就自然拥有 `catch`、`finally`、`then` 这些方法。从上图中可以看到，`then ` 方法返回的是一个新的 `Promise` 实例（注意，不是原来那个 `Promise` 实例）。因此可以采用链式写法，即 `then` 方法后面再调用另一个 `then` 方法。

`Promise` 的中文意思是承诺，这种**"承诺将来会执行"**的对象在 `JavaScript` 中称为 ` Promise` 对象。简单说就是一个容器，里面保存着某个未来才会执行的事件（通常是一个异步操作）的结果。

> `Promise `对象有以下两个特点。
>
> （1）对象的状态不受外界影响。`Promise` 对象代表一个异步操作，有三种状态：`pending`（进行中）、`fulfilled`（已成功）和 `rejected`（已失败）。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。这也是 `Promise ` 这个名字的由来，它的英语意思就是“承诺”，表示其他手段无法改变。
>
> （2）一旦状态改变，就不会再变，任何时候都可以得到这个结果。`Promise ` 对象的状态改变，只有两种可能：从 `pending` 变为 `fulfilled  ` 和从 `pending` 变为 `rejected`。只要这两种情况发生，状态就凝固了，不会再变了，会一直保持这个结果，这时就称为 `resolved`（已定型）。如果改变已经发生了，你再对`Promise` 对象添加回调函数，也会立即得到这个结果。这与事件（`Event`）完全不同，事件的特点是，如果你错过了它，再去监听，是得不到结果的。
>
> `Promise` 也有一些缺点。首先，无法取消 `Promise`，一旦新建它就会立即执行，无法中途取消。其次，如果不设置回调函数，`Promise` 内部抛出的错误，不会反应到外部。第三，当处于 `pending` 状态时，无法得知目前进展到哪一个阶段（刚刚开始还是即将完成）。
>
> —— 摘自 http://es6.ruanyifeng.com/#docs/promise



### 二、Promise的使用

#### 1、创建Promise

那如何创建一个 `Promise` 呢，下面看一个简单的例子：

```javascript
const p = new Promise(function(resolve, reject) {
  //Do some Async
  setTimeout(function() {
    console.log('执行完成');
    resolve('数据');
  }, 2000);
});
```

`Promise ` 构造函数接受一个函数作为参数，该函数的两个参数分别是 `resolve` 和 `reject`，这两个参数也是函数，由 `JavaScript` 引擎提供，不用自己实现。

- `resolve` 函数的作用是，将 `Promise` 对象的状态从“未完成”变为“成功”（即从 `pending` 变为 `resolved`），在异步操作成功时调用，并将异步操作的结果，作为参数传递出去；
- `reject` 函数的作用是，将 `Promise` 对象的状态从“未完成”变为“失败”（即从 `pending` 变为 `rejected`），在异步操作失败时调用，并将异步操作报出的错误，作为参数传递出去。

在上面的代码中，我们执行了一个异步操作，也就是 `setTimeout`，2秒后，输出“执行完成”，并且调用 `resolve`方法。运行代码的时候我们发现，我们只是 `new` 了一个 `Promise` 对象，并没有调用它，我们传进去的函数就已经执行了。**所以，我们使用 `Promise` 的时候一般是包在一个函数中，在需要的时候去运行这个函数 :**

```javascript
function runAsync() {
  const p = new Promise(function(resolve, reject) {
    //Do some Async
    setTimeout(function() {
      console.log('执行完成');
      resolve('数据');
    }, 2000);
  });
  return p;
}
runAsync();
```

函数会 `return` 出 `Promise` 对象，也就是说，执行这个函数我们得到了一个 `Promise` 对象。在文章开始的时候，我们知道 `Promise` 对象拥有 `catch`、`finally`、`then` 这些方法，现在我们看看怎么使用它们。继续使用上面的 `runAsync` 函数 :

```javascript
function runAsync() {
  const p = new Promise(function(resolve, reject) {
    //Do some Async
    setTimeout(function() {
      console.log('执行完成');
      resolve('数据');
    }, 2000);
  });
  return p;
}
runAsync().then(
  function(data) {
    // success
    console.log(`成功拿到${data}`);
    //后面可以用传过来的数据做些其他操作
    // ......
  },
  function(error) {
    // failure
    console.log(error);
  }
);
```

`Promise` 实例生成以后，可以用 `then` 方法分别指定 `resolved ` 状态和 `rejected` 状态的回调函数。`Promise`实例的状态变为 `resolved` 或 `rejected`，就会触发 `then ` 方法绑定的回调函数。

```javascript
promise.then(function(value) {
  // success
}, function(error) {
  // failure
});
```

`then ` 方法可以接受两个回调函数作为参数。第一个回调函数是 `Promise` 对象的状态变为 `resolved` 时调用，第二个回调函数是 `Promise` 对象的状态变为 `rejected` 时调用。其中，第二个函数是可选的，不一定要提供。这两个函数都接受 `Promise` 对象传出的值作为参数。

结论：所以这个时候我们就会发现：原来 `then` 里面的函数和我们平时的回调函数一个意思，能够在 `runAsync` 这个异步任务执行完成之后被执行。

这里我们就可以清楚的知道 `Promise` 的作用了：**异步执行的流程中，把原来的回调写法（执行代码和处理结果的代码）分离出来，在异步操作执行完后，用链式调用的方式执行回调函数。**

下面我们再具体看看 `Promise` 相比于回调嵌套的写法的好处。



#### 2、回调嵌套与Promise

从表面上看，`Promise` 只是能够简化层层回调的写法，而实质上，`Promise` 的精髓是“状态”，用维护状态、传递状态的方式来使得回调函数能够及时调用，它比传递 `callback` 函数要简单、灵活的多。我们来看看这种简化解决了什么问题：

以往使用回调嵌套的方式来处理异步的代码是怎么实现的呢？

```javascript
doA(function() {
  doB();
  doC(function() {
    doD();
  });
  doE();
});
doF();

//执行顺序：
//doA
//doF
//doB
//doC
//doE
//doD
```

这样组织的代码就会遇到一个问题：当项目的代码变得复杂，加上了各种逻辑判断，不断的在函数之间跳转，那排查问题的难度就会大大增加。就比如在上面这个例子中，`doD()` 必须在 `doC()` 完成后才能完成，如果 `doC()` 执行失败了呢？我们是要重试 `doC()` 吗？还是直接转到其他错误处理函数中？当我们将这些判断都加入到这个流程中，很快代码就会变得非常复杂，难以定位问题。

回调嵌套：

```javascript
request(url, function(err, res, body) {
    if (err) handleError(err);
    fs.writeFile('1.txt', body, function(err) {
        request(url2, function(err, res, body) {
            if (err) handleError(err)
        })
    })
});
```

使用 `Promise` 之后:

```javascript
request(url)
.then(function(result) {
    return writeFileAsynv('1.txt', result)
})
.then(function(result) {
    return request(url2)
})
.catch(function(e){
    handleError(e)
});
```

使用 `Promise` 的好处就非常明显了。



#### 3、catch 方法

 `Promise` 对象也拥有 `catch` 方法。它的用途是什么呢？其实它和 `then` 方法的第二个参数是一样的，用来指定`reject` 的回调，和写在 `then` 里第二个参数里面的效果是一样。用法如下：

```javascript
runAsync()
.then(function(data){
    console.log('resolved');
    console.log(data);
})
.catch(function(error){
    console.log('rejected');
    console.log(error);
});
```

`catch` 还有另外一个作用：在执行 `resolve` 的回调（也就是上面 `then` 中的第一个参数）时，如果抛出异常了（代码出错了），那么并不会程序报错卡死，而是会进到这个 `catch` 方法中，看个例子：

```javascript
runAsync()
  .then(function(data) {
    console.log('resolved');
    console.log(data);
    console.log(somedata); //此处的somedata未定义
  })
  .catch(function(error) {
    console.log('rejected');
    console.log(error);
  });

// 执行完成
// resolved
// 数据
// rejected
// ReferenceError: somedata is not defined
```

在 `resolve` 的回调中，`somedata` 这个变量是没有被定义的。如果我们不用 `catch`，代码运行到这里就直接报错了，不往下运行了。但是在这里，会得到这样的结果。也就是说，程序执行到 `catch` 方法里面去了，而且把错误原因传到了 `error` 参数中。即便是有错误的代码也不会报错了，这与 `try/catch` 语句有相同的功能。**所以，如果想捕获错误，就可以使用 `catch` 方法。**



#### 4、Promise.all()

`Promise.all ` 方法用于将多个 `Promise` 实例，包装成一个新的 `Promise` 实例。

```javascript
const p = Promise.all([p1, p2, p3]);
```

看个例子：

```javascript
function runAsync1() {
  const p = new Promise(function(resolve, reject) {
    //Do some Async
    setTimeout(function() {
      console.log('执行完成1');
      resolve('数据1');
    }, 2000);
  });
  return p;
}

function runAsync2() {
  const p = new Promise(function(resolve, reject) {
    //Do some Async
    setTimeout(function() {
      console.log('执行完成2');
      resolve('数据2');
    }, 2000);
  });
  return p;
}

function runAsync3() {
  const p = new Promise(function(resolve, reject) {
    //Do some Async
    setTimeout(function() {
      console.log('执行完成3');
      resolve('数据3');
    }, 2000);
  });
  return p;
}

Promise.all([runAsync1(), runAsync2(), runAsync3()]).then(function(results) {
  console.log(results);
});

// 执行完成1
// 执行完成2
// 执行完成3
// [ '数据1', '数据2', '数据3' ]
```

用 `Promise.all` 来执行，接收一个数组参数，里面的值最终都返回 `Promise` 对象。这样，三个异步操作的就是并行执行的，等到它们都执行完后才会进到 `then` 里面。那么，三个异步操作返回的数据哪里去了呢？都在 `then` 里面呢，`Promise.all` 会把所有异步操作的结果放进一个数组中传给 `then` ，就是上面的 `results` 。

**应用场景：**

`Promise.all` 方法有一个非常常用的应用场景：打开网页时，预先加载需要用到的各种资源如图片及各种静态文件，所有的都加载完后，再进行页面的初始化。 



#### 5、Promise.race()

`race` 是竞赛、赛跑的意思。它的用法也就是它的字面意思：**谁跑的快，就以谁为准，执行回调**。其实再看看`Promise.all` 方法，和 `race` 方法恰恰相反。还是用 `Promise.all` 的例子，但是把 `runAsync1` 的方法 `timeout` 时间调成 `1000ms`。 

```javascript
Promise.race([runAsync1(), runAsync2(), runAsync3()]).then(function(results) {
  console.log(results);
});

// 执行完成1
// 数据1
// 执行完成2
// 执行完成3
```

这三个异步操作同样是并行执行的。结果很可以猜到，1秒后 `runAsync1` 已经执行完了，此时 `then` 里面的方法就会立即执行了。但是，在 `then` 里面的回调函数开始执行时，`runAsync2()` 和 `runAsync3()` 并没有停止，仍然继续执行。所以再过1秒后，输出了他们结束的标志。这个点需要注意。

上面的这些方法就是 `Promise` 比较常用的几个方法了。



### 三、红绿灯问题

题目：红灯三秒亮一次，绿灯一秒亮一次，黄灯2秒亮一次；如何让三个灯不断交替重复亮灯？（用 `Promse` 实现）

三个亮灯函数已经存在：

```javascript
function red(){
    console.log('red');
}
function green(){
    console.log('green');
}
function yellow(){
    console.log('yellow');
}
```

利用 `then` 和递归实现：

```javascript
function red() {
  console.log('red');
}
function green() {
  console.log('green');
}
function yellow() {
  console.log('yellow');
}

const light = function (timmer, color) {
  return new Promise(function (resolve, reject) {
    setTimeout(function () {
      color();
      resolve();
    }, timmer);
  });
};

const step = function () {
  Promise.resolve()
    .then(function () {
      return light(3000, red);
    })
    .then(function () {
      return light(2000, green);
    })
    .then(function () {
      return light(1000, yellow);
    })
    .then(function () {
      step();
    });
};

step();
```

