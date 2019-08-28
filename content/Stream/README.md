## Stream

流（`stream`）是 `Node.js` 中处理流式数据的抽象接口。 `stream` 模块用于构建实现了流接口的对象。

`Node.js` 提供了多种流对象。 例如，对 `http` 服务器发起请求的 `request`  对象就是一个 `Stream`。



### 一、流的类型

`Node.js` 中有四种基本的流类型：

- [`Writable`](http://nodejs.cn/s/9JUnJ8) - 可写入数据的流（例如 [`fs.createWriteStream()`](http://nodejs.cn/s/VdSJQa)）。
- [`Readable`](http://nodejs.cn/s/YuDKX1) - 可读取数据的流（例如 [`fs.createReadStream()`](http://nodejs.cn/s/wiVPXD)）。
- [`Duplex`](http://nodejs.cn/s/2iRabr) - 可读又可写的流（例如 [`net.Socket`](http://nodejs.cn/s/wsJ1o1)）。
- [`Transform`](http://nodejs.cn/s/fhVJQM) - 在读写过程中可以修改或转换数据的 `Duplex` 流（例如 [`zlib.createDeflate()`](http://nodejs.cn/s/n6ED45)）。

此外，该模块还包括实用函数 [`stream.pipeline()`](http://nodejs.cn/s/fnFy4m)、[`stream.finished()`](http://nodejs.cn/s/DjDduq) 和 [`stream.Readable.from()`](http://nodejs.cn/s/bSCxhZ)。



所有的流都是 [`EventEmitter`](http://nodejs.cn/s/pGAddE) 的实例。常用的事件有：

- `data` - 当有数据可读时触发。
- `end` - 没有更多的数据可读时触发。
- `error` - 在接收和写入过程中发生错误时触发。
- `finish` - 所有数据已被写入到底层系统时触发。

下面我们介绍下常见的流操作。



### 二、从流中读取数据

创建一个 `test.txt` 文件：写入以下内容：

```
node.togoblog.cn
```

`index.js` ：

```javascript
const fs = require("fs");

let data = '';

// 创建可读流
const readerStream = fs.createReadStream('test.txt');

// 设置编码为 utf8。
readerStream.setEncoding('UTF8');

// 处理流事件 --> data, end, and error
readerStream.on('data', function (chunk) {
  data += chunk;
});

readerStream.on('end', function () {
  console.log(data);
});

readerStream.on('error', function (err) {
  console.log(err.stack);
});

console.log("程序执行完毕");
```

运行结果：

```javascript
> node index.js
程序执行完毕
node.togoblog.cn
```



### 三、写入流

`index.js`：

```javascript
const fs = require("fs");

const data = 'node.togoblog.cn';

// 创建一个可以写入的流，写入到文件 output.txt 中
const writerStream = fs.createWriteStream('output.txt');

// 使用 utf8 编码写入数据
writerStream.write(data, 'UTF8');

// 标记文件末尾
writerStream.end();

// 处理流事件 --> data, end, and error
writerStream.on('finish', function () {
  console.log("写入完成。");
});

writerStream.on('error', function (err) {
  console.log(err.stack);
});

console.log("程序执行完毕");
```

以上程序会将 `data` 变量的数据写入到 `output.txt` 文件中。代码执行结果如下：

```javascript
> node main.js
程序执行完毕
写入完成。
```

我们发现创建了一个 `output.txt` 文件，相应的数据也被写入进去了。



### 四、管道流

管道提供了一个输出流到输入流的机制。通常我们用于从一个流中获取数据并将数据传递到另外一个流中。

以下实例我们通过读取一个文件内容并将内容写入到另外一个文件中。

创建一个 `input.txt` 文件，文件内容如下：

```
node.togoblog.cn
管道流操作实例
```

`index.js` ：

```javascript
const fs = require("fs");

// 创建一个可读流
const readerStream = fs.createReadStream('input.txt');

// 创建一个可写流
const writerStream = fs.createWriteStream('output.txt');

// 管道读写操作
// 读取 input.txt 文件内容，并将内容写入到 output.txt 文件中
readerStream.pipe(writerStream);

console.log("程序执行完毕");
```

代码执行结果如下：

```javascript
> node index.js
程序执行完毕
```

查看 `output.txt` 文件的内容：

```javascript
> cat output.txt
node.togoblog.cn
管道流操作实例
```



### 五、链式流

链式是通过连接输出流到另外一个流并创建多个对个流操作链的机制。链式流一般用于管道操作。

接下来我们就是用管道和链式来压缩和解压文件。

创建 `compress.js` 文件, 代码内容如下：

```javascript
const fs = require("fs");
const zlib = require('zlib');

// 压缩 input.txt 文件为 input.txt.gz
fs.createReadStream('input.txt')
  .pipe(zlib.createGzip())
  .pipe(fs.createWriteStream('input.txt.gz'));

console.log("文件压缩完成。");
```

代码执行结果如下：

```javascript
> node compress.js
文件压缩完成。
```

执行完以上操作后，我们可以看到当前目录下生成了 `input.txt` 的压缩文件 `input.txt.gz`。

接下来解压该文件，创建 `decompress.js` 文件，代码如下：

```javascript
const fs = require("fs");
const zlib = require('zlib');

// 解压 input.txt.gz 文件为 input.txt
fs.createReadStream('input.txt.gz')
  .pipe(zlib.createGunzip())
  .pipe(fs.createWriteStream('input.txt'));

console.log("文件解压完成。");
```

代码执行结果如下：

```javascript
> node decompress.js
文件解压完成。
```

