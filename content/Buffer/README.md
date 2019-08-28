## Buffer

### 一、概述

`JavaScript` 语言比较擅长处理字符型，但是它没有用于读取或操作二进制数据流的机制。 `Buffer` 类是作为 `Node.js API` 的一部分引入的，该类用来创建一个专门存放二进制数据的缓存区，用于在 `TCP` 流、文件系统操作、以及其他上下文中与八位字节流进行交互。

`Buffer` 类的实例类似于从 `0` 到 `255` 之间的整数数组（其他整数会通过 `＆ 255` 操作强制转换到此范围），但对应于 `V8` 堆外部的固定大小的原始内存分配。`Buffer` 的大小在创建时确定，且无法更改。

`Buffer` 类在全局作用域中，因此无需使用 `require('buffer').Buffer`。



### 二、使用

这里我们只介绍比较常用/容易理解的 `API` 。更多 `API` 参考官网：https://nodejs.org/dist/latest-v10.x/docs/api/buffer.html 

#### 1、创建 Buffer 类

> 注意：旧版本中通过 `new Buffer()` 创建 Buffer 类的方式已经被废弃掉了。

```javascript
// 创建一个长度为 10、且用零填充的 Buffer。
const buf1 = Buffer.alloc(10);

// 创建一个长度为 10、且用 0x1 填充的 Buffer。 
const buf2 = Buffer.alloc(10, 1);

// 创建一个长度为 10、且未初始化的 Buffer。
// 这个方法比调用 Buffer.alloc() 更快，
// 但返回的 Buffer 实例可能包含旧数据，
// 因此需要使用 fill() 或 write() 重写。
const buf3 = Buffer.allocUnsafe(10);

// 创建一个包含 [0x1, 0x2, 0x3] 的 Buffer。
const buf4 = Buffer.from([1, 2, 3]);

// 创建一个包含 UTF-8 字节 [0x74, 0xc3, 0xa9, 0x73, 0x74] 的 Buffer。
const buf5 = Buffer.from('tést');

// 创建一个包含 Latin-1 字节 [0x74, 0xe9, 0x73, 0x74] 的 Buffer。
const buf6 = Buffer.from('tést', 'latin1');

// 创建新的 Buffer 实例，并将 buffer 的数据拷贝到新的实例子中去。
const buff7 = Buffer.from('buffer');
const buff8 = Buffer.from(buff7);
console.log(buff8.toString())
```



#### 2、Buffer 比较

##### buf.equals(otherBuffer)

判断两个 `buffer` 实例存储的数据是否相同。如果 `buf` 与 `otherBuffer` 具有完全相同的字节，则返回 `true`，否则返回 `false`。

```javascript
const buf1 = Buffer.from('ABC');
const buf2 = Buffer.from('414243', 'hex');
const buf3 = Buffer.from('ABCD');

console.log(buf1.equals(buf2));
// 打印: true
console.log(buf1.equals(buf3));
// 打印: false
```

##### buf.compare

> 语法：`buf.compare(target[, targetStart[, targetEnd[, sourceStart[, sourceEnd]]]])`

对比 `buf` 与 `target`，并返回一个数值，表明 `buf` 在排序上是否排在 `target` 前面、或后面、或相同。 对比是基于各自 `Buffer` 实际的字节序列。

- 如果 `target` 与 `buf` 相同，则返回 `0`。
- 如果 `target` 排在 `buf` 前面，则返回 `1`。
- 如果 `target` 排在 `buf` 后面，则返回 `-1`。

```javascript
const buf1 = Buffer.from('ABC');
const buf2 = Buffer.from('BCD');
const buf3 = Buffer.from('ABCD');

console.log(buf1.compare(buf1));
// 打印: 0
console.log(buf1.compare(buf2));
// 打印: -1
console.log(buf1.compare(buf3));
// 打印: -1
console.log(buf2.compare(buf1));
// 打印: 1
console.log(buf2.compare(buf3));
// 打印: 1
console.log([buf1, buf2, buf3].sort(Buffer.compare));
// 打印: [ <Buffer 41 42 43>, <Buffer 41 42 43 44>, <Buffer 42 43 44> ]
// (相当于: [buf1, buf3, buf2])
```

`targetStart`、 `targetEnd`、 `sourceStart` 与 `sourceEnd` 可用于指定 `target` 与 `buf` 中对比的范围：

```javascript
const buf1 = Buffer.from([1, 2, 3, 4, 5, 6, 7, 8, 9]);
const buf2 = Buffer.from([5, 6, 7, 8, 9, 1, 2, 3, 4]);

console.log(buf1.compare(buf2, 5, 9, 0, 4));
// 打印: 0
console.log(buf1.compare(buf2, 0, 6, 4));
// 打印: -1
console.log(buf1.compare(buf2, 5, 6, 5));
// 打印: 1
```

如果 `targetStart < 0`、 `sourceStart < 0`、 `targetEnd > target.byteLength` 或 `sourceEnd > source.byteLength`，则抛出 [`ERR_OUT_OF_RANGE`](http://nodejs.cn/s/tsvMjv)。

##### Buffer.compare(buf1, buf2)

比较 `buf1` 与 `buf2`，主要用于 `Buffer` 实例数组的排序。 相当于调用 [`buf1.compare(buf2)`](http://nodejs.cn/s/3wddT3)。

```javascript
const buf1 = Buffer.from('1234');
const buf2 = Buffer.from('0123');
const arr = [buf1, buf2];

console.log(arr.sort(Buffer.compare));
// 打印: [ <Buffer 30 31 32 33>, <Buffer 31 32 33 34> ]
// (结果相当于: [buf2, buf1])
```



#### 3、Buffer 连接

> 语法：`Buffer.concat(list[, totalLength])`

`Buffer.concat` 会返回一个合并了 `list` 中所有 `Buffer` 实例的新 `Buffer`。

如果 `list` 中没有元素、或 `totalLength` 为 0，则返回一个长度为 0 的 `Buffer`。

如果没有提供 `totalLength`，则计算 `list` 中的 `Buffer` 实例的总长度。 但是这会导致执行额外的循环用于计算 `totalLength`，因此如果已知长度，则明确提供长度会更快。

如果提供了 `totalLength`，则会强制转换为无符号整数。 如果 `list` 中的 `Buffer` 合并后的总长度大于 `totalLength`，则结果会被截断到 `totalLength` 的长度。

```javascript
// 用含有三个 `Buffer` 实例的数组创建一个单一的 `Buffer`。

const buf1 = Buffer.alloc(10);
const buf2 = Buffer.alloc(14);
const buf3 = Buffer.alloc(18);
const totalLength = buf1.length + buf2.length + buf3.length;

console.log(totalLength);
// 打印: 42

const bufA = Buffer.concat([buf1, buf2, buf3], totalLength);

console.log(bufA);
// 打印: <Buffer 00 00 00 00 ...>
console.log(bufA.length);
// 打印: 42
```



#### 4、Buffer 拷贝

> 语法：`buf.copy(target[, targetStart[, sourceStart[, sourceEnd]]])`

使用起来比较简单，如果忽略后面三个参数，那就是将 `buf` 的数据拷贝到 `target` 里去，如下所示：

```javascript
// 创建两个 `Buffer` 实例。
const buf1 = Buffer.allocUnsafe(26);
const buf2 = Buffer.allocUnsafe(26).fill('!');

for (let i = 0; i < 26; i++) {
  // 97 是 'a' 的十进制 ASCII 值。
  buf1[i] = i + 97;
}

// 拷贝 `buf1` 中第 16 至 19 字节偏移量的数据到 `buf2` 第 8 字节偏移量开始。
buf1.copy(buf2, 8, 16, 20);

console.log(buf2.toString('ascii', 0, 25));
// 打印: !!!!!!!!qrst!!!!!!!!!!!!!
```

```javascript
// 创建一个 `Buffer`，并拷贝同一 `Buffer` 中一个区域的数据到另一个重叠的区域。

const buf = Buffer.allocUnsafe(26);

for (let i = 0; i < 26; i++) {
  // 97 是 'a' 的十进制 ASCII 值。
  buf[i] = i + 97;
}

buf.copy(buf, 0, 4, 10);

console.log(buf.toString());
// 打印: efghijghijklmnopqrstuvwxyz
```



#### 5、Buffer 查找

> 语法：`buf.indexOf(value[, byteOffset][, encoding])`

跟数组的查找差不多，需要注意的是，`value` 可能是 `String`、`Buffer`、`Integer` 中的任意类型。

- `String`：如果是字符串，那么 `encoding` 就是其对应的编码，默认是 `utf8`。
- `Buffer`：如果是 `Buffer` 实例，那么会将 `value` 中的完整数据，跟 `buf` 进行对比。
- `Integer`：如果是数字，那么 `value` 会被当做无符号的 `8` 位整数，取值范围是 `0` 到 `255`。

另外，可以通过 `byteOffset` 来指定起始查找位置。

```javascript
const buf = Buffer.from('this is a buffer');

console.log(buf.indexOf('this'));
// 打印: 0
console.log(buf.indexOf('is'));
// 打印: 2
console.log(buf.indexOf(Buffer.from('a buffer')));
// 打印: 8
console.log(buf.indexOf(97));
// 打印: 8（97 是 'a' 的十进制 ASCII 值）
console.log(buf.indexOf(Buffer.from('a buffer example')));
// 打印: -1
console.log(buf.indexOf(Buffer.from('a buffer example').slice(0, 8)));
// 打印: 8

const utf16Buffer = Buffer.from('\u039a\u0391\u03a3\u03a3\u0395', 'utf16le');

console.log(utf16Buffer.indexOf('\u03a3', 0, 'utf16le'));
// 打印: 4
console.log(utf16Buffer.indexOf('\u03a3', -4, 'utf16le'));
// 打印: 6
```

如果 `value` 不是一个字符串、数值、或 `Buffer`，则此方法将会抛出 `TypeError`。 如果 `value` 是一个数值，则会被转换成介于 `0` 到 `255` 之间的整数值。

如果 `byteOffset` 不是一个数值，则会被转换成数值。 如果转换后的值为 `NaN` 或 `0`, 则会查找整个 `buffer`。 这与 [`String#indexOf()`](http://nodejs.cn/s/Uqm5hr) 是一致的。

```javascript
const b = Buffer.from('abcdef');

// 传入一个数值，但不是有效的字节。
// 打印：2，相当于查找 99 或 'c'。
console.log(b.indexOf(99.9));
console.log(b.indexOf(256 + 99));

// 传入被转换成 NaN 或 0 的 byteOffset。
// 打印：1，查找整个 buffer。
console.log(b.indexOf('b', undefined));
console.log(b.indexOf('b', {}));
console.log(b.indexOf('b', null));
console.log(b.indexOf('b', []));
```

如果 `value` 是一个空字符串或空 `Buffer`，且 `byteOffset` 小于 `buf.length`，则返回 `byteOffset`。 如果 `value` 是一个空字符串，且 `byteOffset` 大于或等于 `buf.length`，则返回 `buf.length`。



#### 6、Buffer 写

> 语法：`buf.write(string[, offset[, length]][, encoding])`

- `string`  [<string>](http://nodejs.cn/s/9Tw2bK) 要写入 `buf` 的字符串。
- `offset`  [<integer>](http://nodejs.cn/s/SXbo1v) 开始写入 `string` 之前要跳过的字节数。**默认值:** `0`。
- `length`  [<integer>](http://nodejs.cn/s/SXbo1v) 要写入的字节数。**默认值:** `buf.length - offset`。
- `encoding`  [<string>](http://nodejs.cn/s/9Tw2bK) `string` 的字符编码。**默认值:** `'utf8'`。
- 返回:  [<integer>](http://nodejs.cn/s/SXbo1v) 已写入的字节数。

根据 `encoding` 指定的字符编码将 `string` 写入到 `buf` 中的 `offset` 位置。 `length` 参数是要写入的字节数。 如果 `buf` 没有足够的空间保存整个字符串，则只会写入 `string` 的一部分。 只编码了一部分的字符不会被写入。

```javascript
const buf = Buffer.alloc(256);

const len = buf.write('\u00bd + \u00bc = \u00be', 0);

console.log(`${len} 个字节: ${buf.toString('utf8', 0, len)}`);
// 打印: 12 个字节: ½ + ¼ = ¾
```



#### 7、Buffer 填充

用 `value` 填充 `buf`，常用于初始化 `buf`。参数说明如下：

- `value`：用来填充的内容，可以是 `Buffer`、`String` 或 `Integer`。
- `offset`：从第几位开始填充，默认是 `0`。
- `end`：停止填充的位置，默认是 `buf.length`。
- `encoding`：如果 `value` 是 `String`，那么为 `value` 的编码，默认是 `utf8`。

```javascript
const buff = Buffer.alloc(20).fill('a');

console.log(buff.toString());  // aaaaaaaaaaaaaaaaaaaa
```



#### 8、转成字符串

把 `buf` 解码成字符串，用法比较简单：

```javascript
const buff = Buffer.from('hello');

console.log(buff.toString());  // hello

console.log(buff.toString('utf8', 0, 2));  // he
```



#### 9、转成 JSON 字符串

```javascript
const buff = Buffer.from('hello');

console.log(buff.toJSON());  // { type: 'Buffer', data: [ 104, 101, 108, 108, 111 ] }
```



#### 10、遍历

用于对 `buf` 进行 `for...of` 遍历，直接看例子：

> `buf.values()`、`buf.keys()`、`buf.entries()`

```javascript
const buff = Buffer.from('abcde');

for (const key of buff.keys()) {
  console.log('key is %d', key);
}
// key is 0
// key is 1
// key is 2
// key is 3
// key is 4

for (const value of buff.values()) {
  console.log('value is %d', value);
}
// value is 97
// value is 98
// value is 99
// value is 100
// value is 101

for (const pair of buff.entries()) {
  console.log('buff[%d] === %d', pair[0], pair[1]);
}
// buff[0] === 97
// buff[1] === 98
// buff[2] === 99
// buff[3] === 100
// buff[4] === 101
```



#### 11、Buffer 截取

用于截取 `buf`，并返回一个新的 `Buffer` 实例。需要注意的是，这里返回的 `Buffer` 实例，指向的仍然是 `buf` 的内存地址，所以对新 `Buffer` 实例的修改，也会影响到 `buf`。

```javascript
const buff1 = Buffer.from('abcde');
console.log(buff1);  // <Buffer 61 62 63 64 65>

const buff2 = buff1.slice();
console.log(buff2);  // <Buffer 61 62 63 64 65>

const buff3 = buff1.slice(1, 3);
console.log(buff3);  // <Buffer 62 63>

buff3[0] = 97;  // parseInt(61, 16) ==> 97
console.log(buff1);  // <Buffer 62 63>
```

