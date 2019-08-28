## EventEmitter

`Node.js` 所有的异步 `I/O` 操作在完成时都会发送一个事件到事件队列。

例如，[`net.Server`](http://nodejs.cn/s/gBYjux) 会在每次有新连接时触发事件，[`fs.ReadStream`](http://nodejs.cn/s/C3Eioq) 会在打开文件时触发事件，[`stream`](http://nodejs.cn/s/kUvpNm) 会在数据可读时触发事件。

所有能触发事件的对象都是 `EventEmitter` 类的实例。 



### 一、EventEmitter 类

`events` 模块只提供了一个对象： `events.EventEmitter`。`EventEmitter` 的核心就是事件触发与事件监听器功能的封装。

我们可以通过 `require('events');` 来访问该模块：

```javascript
// 引入 events 模块
const Event = require('events');
// 创建 eventEmitter 对象
const eventEmitter = new Event.EventEmitter();
```

`EventEmitter` 对象如果在实例化时发生错误，会触发 `error` 事件。

当添加新的监听器时，`newListener` 事件会触发，当监听器被移除时，`removeListener` 事件被触发。

下面我们用一个简单的例子说明 `EventEmitter` 的用法：

`index.js` ：

```javascript
// 引入 events 模块
const Event = require('events');
// 创建 eventEmitter 对象
const eventEmitter = new Event.EventEmitter();

eventEmitter.on('someEvent', function () {
  console.log('someEvent 事件触发');
});

setTimeout(function () {
  eventEmitter.emit('someEvent');
}, 1000); 
```

执行结果：

```javascript
> node index.js
someEvent 事件触发
```

运行这段代码，`1` 秒后控制台输出了 ' `someEvent` 事件触发'。其原理是 `event` 对象注册了事件 `someEvent` 的一个监听器，然后我们通过 `setTimeout` 在 `1000` 毫秒后向 `event` 对象发送事件 `someEvent`，此时会调用 `someEvent` 的监听器。

`EventEmitter` 的每个事件由一个事件名和若干个参数组成，事件名是一个字符串，通常表达一定的语义。对于每个事件，`EventEmitter` 支持 若干个事件监听器。

当事件触发时，注册到这个事件的事件监听器被依次同步调用，事件参数作为回调函数参数传递。

看看下面的例子：

```javascript
const events = require('events');
const emitter = new events.EventEmitter();

emitter.on('someEvent', function (arg1, arg2) {
  console.log('listener1', arg1, arg2);
});

emitter.on('someEvent', function (arg1, arg2) {
  console.log('listener2', arg1, arg2);
});

emitter.emit('someEvent', 'arg1 参数', 'arg2 参数');
```

执行结果：

```javascript
> node index.js
listener1 arg1 参数 arg2 参数
listener2 arg1 参数 arg2 参数
```

上面的例子中，`emitter` 为事件 `someEvent` 注册了两个事件监听器，然后触发了 `someEvent` 事件。从运行结果中我们可以看到两个事件监听器的回调函数先后被调用（同步的）。

`EventEmitter` 提供了多个属性，如 `on` 和 `emit`。`on` 函数用于绑定事件函数，`emit` 属性用于触发一个事件。



### 二、将参数和 this 传给监听器

`eventEmitter.emit()` 方法可以传任意数量的参数到监听器函数。 当监听器函数被调用时， `this` 关键词会被指向监听器所绑定的 `EventEmitter` 实例。

```javascript
const events = require('events');
const emitter = new events.EventEmitter();

emitter.on('event', function (a, b) {
  console.log(a, b, this, this === emitter);
});
// 结果打印:
//   a b MyEmitter {
//     domain: null,
//     _events: { event: [Function] },
//     _eventsCount: 1,
//     _maxListeners: undefined } true
emitter.emit('event', 'a', 'b');
```

也可以使用 `ES6` 的箭头函数作为监听器。但 `this` 关键词不会指向 `EventEmitter` 实例：

```javascript
const events = require('events');
const emitter = new events.EventEmitter();

emitter.on('event', (a, b) => {
  console.log(a, b, this); // 打印: a b {}
});

emitter.emit('event', 'a', 'b');
```



### 三、异步 VS 同步

`EventEmitter` 会按照监听器注册的顺序同步地调用所有监听器。 所以必须确保事件的排序正确，且避免竞态条件。 可以使用 `setImmediate()` 或 `process.nextTick()` 切换到异步模式：

```javascript
const events = require('events');
const emitter = new events.EventEmitter();

emitter.on('event', (a, b) => {
  setImmediate(() => {
    console.log('异步进行');
  });
  console.log(a, b);
});
emitter.emit('event', 'a', 'b');

// 打印结果
// a b
// 异步进行
```



### 四、仅处理事件一次

当使用 `eventEmitter.on()` 注册监听器时，监听器会在每次触发命名事件时被调用：

```javascript
const events = require('events');
const emitter = new events.EventEmitter();

let m = 0;
emitter.on('event', () => {
  console.log(++m);
});
emitter.emit('event'); // 打印: 1
emitter.emit('event'); // 打印: 2
```

使用 `eventEmitter.once()` 可以注册最多可调用一次的监听器。 当事件被触发时，监听器会被注销，然后再调用：

```javascript
const events = require('events');
const emitter = new events.EventEmitter();

let m = 0;
emitter.once('event', () => {
  console.log(++m);
});
emitter.emit('event'); // 打印: 1
emitter.emit('event'); // 不触发
```



### 五、 错误事件

当 `EventEmitter` 实例出错时，应该触发 `error` 事件。 这些在 `Node.js` 中被视为特殊情况。

如果没有为 `error` 事件注册监听器，则当 `error` 事件触发时，会抛出错误、打印堆栈跟踪、并退出 Node.js 进程：

```javascript
const events = require('events');
const emitter = new events.EventEmitter();

emitter.emit('error', new Error('错误信息'));
```

为了防止崩溃 `Node.js` 进程，可以使用 [`domain`](http://nodejs.cn/s/cnfQ9s) 模块，但是这并不是一个好的实践。作为最佳实践，我们应该始终为 `error` 事件注册监听器：

```javascript
const events = require('events');
const emitter = new events.EventEmitter();

emitter.on('error', (err) => {
  console.error('错误信息');
});

emitter.emit('error', new Error('错误')); // 打印：错误信息
```



### 六、 继承 EventEmitter

大多数时候我们不会直接使用 `EventEmitter`，而是在对象中继承它。包括 `fs`、`net`、 `http` 在内的，只要是支持事件响应的核心模块都是 `EventEmitter` 的子类。

这样做的原因有两点：

- 首先，具有某个实体功能的对象实现事件符合语义， 事件的监听和发射应该是一个对象的方法。
- 其次，` JavaScript` 的对象机制是基于原型的，支持部分多重继承，继承 `EventEmitter` 不会打乱对象原有的继承关系。