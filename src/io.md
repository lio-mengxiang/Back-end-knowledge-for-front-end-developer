# IO

* [Buffer]
* [String Decoder (字符串解码)]
* [File System (文件系统)]
* [Readline]


# 简述

Node.js 是以 IO 密集型业务著称. 那么问题来了, 你真的了解什么叫 IO, 什么又叫 IO 密集型业务吗?

## 什么是IO？

在计算机系统中I/O就是输入（Input）和输出(Output)的意思，针对不同的操作对象，可以划分为磁盘I/O模型，网络I/O模型，内存映射I/O, Direct I/O、数据库I/O等，只要具有输入输出类型的交互系统都可以认为是I/O系统，也可以说I/O是整个操作系统数据交换与人机交互的通道，这个概念与选用的开发语言没有关系，是一个通用的概念。

## 什么是IO密集型

IO密集型指的是系统的CPU性能相对硬盘、内存要好很多，此时，系统运作，大部分的状况是CPU在等I/O (硬盘/内存) 的读/写操作，此时CPU Loading并不高

## Buffer

在 Node.js 中，Buf所以说，Buffer 的本质就是一个继承自 Uint8Array 的子类fer 存在于 buffer 内置模块中。不过现在的 Node.js 也已经直接把 Buffer 挂载在了 globalThis 上,也就是说我们可以全局直接使用Buffer对象，而不需要require引入。

Buffer 的本质就是一个继承自 Uint8Array 的子类。

```javascript
const a = Buffer.from('123');
console.log(a instanceof Unit8Array);  // true
console.log(a.byteOffset);  // 16
console.log(a.buffer);      // ArrayBuffer { byteLength: 8192, ... }
```

Buffer 是 Node.js 中用于处理二进制数据的类, 其中与 IO 相关的操作 (网络/文件等) 均基于 Buffer. Buffer 类的实例非常类似整数数组, ***但其大小是固定不变的***, 并且其内存在 V8 堆栈外分配原始内存空间. Buffer 类的实例创建之后, 其所占用的内存大小就不能再进行调整.

在 Node.js v6.x 之后 `new Buffer()` 接口开始被废弃, 理由是参数类型不同会返回不同类型的 Buffer 对象, 所以当开发者没有正确校验参数或没有正确初始化 Buffer 对象的内容时, 以及不了解的情况下初始化  就会在不经意间向代码中引入安全性和可靠性问题.

接口|用途
---|---
Buffer.from()|根据已有数据生成一个 Buffer 对象
Buffer.alloc()|创建一个初始化后的 Buffer 对象
Buffer.allocUnsafe()|创建一个未初始化的 Buffer 对象

其实Buffer.alloc的创建就是使用的Unit8Array。



### TypedArray

Node.js 的 Buffer 在 ES6 增加了 TypedArray 类型之后, 修改了原来的 Buffer 的实现, 选择基于 TypedArray 中 Uint8Array 来实现, 从而提升了一波性能.

使用上, 你需要了解如下情况:

```javascript
const arr = new Uint16Array(2);
arr[0] = 5000;
arr[1] = 4000;

const buf1 = Buffer.from(arr); // 拷贝了该 buffer
const buf2 = Buffer.from(arr.buffer); // 与该数组共享了内存

console.log(buf1);
// 输出: <Buffer 88 a0>, 拷贝的 buffer 只有两个元素
console.log(buf2);
// 输出: <Buffer 88 13 a0 0f>

arr[1] = 6000;
console.log(buf1);
// 输出: <Buffer 88 a0>
console.log(buf2);
// 输出: <Buffer 88 13 70 17>
```

## String Decoder

字符串解码器 (String Decoder) 是一个用于将 Buffer 拿来 decode 到 string 的模块, 是作为 Buffer.toString 的一个补充, 它支持多字节 UTF-8 和 UTF-16 字符. 例如

```javascript
const StringDecoder = require('string_decoder').StringDecoder;
const decoder = new StringDecoder('utf8');

const cent = Buffer.from([0xC2, 0xA2]);
console.log(decoder.write(cent)); // ¢

const euro = Buffer.from([0xE2, 0x82, 0xAC]);
console.log(decoder.write(euro)); // €
```

stringDecoder.write 会确保返回的字符串不包含 Buffer 末尾残缺的多字节字符，残缺的多字节字符会被保存在一个内部的 buffer 中用于下次调用 stringDecoder.write() 或 stringDecoder.end()。

```javascript
const StringDecoder = require('string_decoder').StringDecoder;
const decoder = new StringDecoder('utf8');

decoder.write(Buffer.from([0xE2]));
decoder.write(Buffer.from([0x82]));
console.log(decoder.end(Buffer.from([0xAC])));  // €
```

## File

“一切皆是文件”是 Unix/Linux 的基本哲学之一, 不仅普通的文件、目录、字符设备、块设备、套接字等在 Unix/Linux 中都是以文件被对待, 也就是说这些资源的操作对象均为 fd (文件描述符), 都可以通过同一套 system call 来读写. 在 linux 中你可以通过 ulimit 来对 fd 资源进行一定程度的管理限制.

Node.js 封装了标准 POSIX 文件 I/O 操作的集合. 通过 require('fs') 可以加载该模块. 该模块中的所有方法都有异步执行和同步执行两个版本. 你可以通过 fs.open 获得一个文件的文件描述符.

### fs 文件系统模块
#### 什么是fs文件模块系统?

fs 模块是 Node.js 官方提供的、用来操作文件的模块。它提供了一系列的方法和属性，用来满足用户对文件的操作需求，该模块的所有方法都有同步和异步两种方式。

JavaScript 的是没有操作文件的能力，但是 Node 是可以做到的，Node 提供了操作文件系统模块，是 Node 中使用非常重要和高频的模块，是绝对要掌握的一个模块系统。

fs 模块中所有的操作都有两种形式可供选择:同步和异步

- 同步文件系统会阻塞程序的执行，也就是除非操作完毕，否则不会向下执行代码
- 异步文件系统不会阻塞程序的执行，而是在操作完成时，通过回调函数将结果返回,然后可以立即向下执行代码

- 打开文件

 - 格式 ： fs.open(path, flags[, mode], callback)
 - path : 文件的路径
 - flags : 文件打开的行为。具体值详见下方表格
 - mode : 设置文件模式(权限)，文件创建默认权限为 0666(可读，可写)
 - callback : 回调函数，带有两个参数如：callback(err, fd)

打开模式(flags)	说明
```
r	以读取模式打开文件。如果文件不存在抛出异常。
r+	以读写模式打开文件。如果文件不存在抛出异常。
rs	以同步的方式读取文件。
rs+	以同步的方式读取和写入文件。
w	以写入模式打开文件，如果文件不存在则创建。
wx	类似 ‘w’，但是如果文件路径存在，则文件写入失败。
w+	以读写模式打开文件，如果文件不存在则创建。
wx+	类似 ‘w+’， 但是如果文件路径存在，则文件读写失败。
a	以追加模式打开文件，如果文件不存在则创建。
ax	类似 ‘a’， 但是如果文件路径存在，则文件追加失败。
a+	以读取追加模式打开文件，如果文件不存在则创建。
ax+	类似 ‘a+’， 但是如果文件路径存在，则文件读取追加失败。
```
示例代码:
```javascript
const fs = require('fs')
fs.open('./file/new成绩.txt','r+',function(err,result) {
    if(err) {
        return console.log('打开文件失败' + err.message);
    } 
    console.log('打开文件成功' + result);
})
```

在这里，首先要导入fs模块，node中导入模块需要使用内置的require()方法，这里的回调函数中，如果文件存在的话err会返回null,在js中null会默认转换为false，如果文件不存在的话，则err会返回一个错误对象，错误对象会转化为true,从而在这里去写一个判断输出逻辑！

- 获取文件信息

 - 语法格式 : fs.stat(path, callback)
 - path : 文件路径
 - callback : 回调函数，带有两个参数如：(err, stats), stats 是 fs.Stats 对象。

- stats类中的方法:

  - stats.isFile()	如果是文件返回 true，否则返回 false。
  - stats.isDirectory()	如果是目录返回 true，否则返回 false。
示例代码:
```
const fs = require('fs')
fs.stat('./file/new成绩.txt', function (err, stats) {
    if (err) {
        return console.error(err);
    }
    console.log("读取文件信息成功！");
    // 检测文件类型
    console.log("是否为文件(isFile) ? " + stats.isFile());
    console.log("是否为目录(isDirectory) ? " + stats.isDirectory());    
 });
```


获取文件信息用的更多的方法是isFile()和isDirectory(),主要是来判断该文件是否属于文件或者是否属于目录!

- 读取文件
  - 语法格式 : fs.readFile(path[, options], callback)

  - path：文件路径

  - options：配置选项，若是字符串则指定编码格式
  - encoding：编码格式
  - flag：打开方式
  - callback：回调函数
  - err：错误信息
  - data：读取的数据，如果未指定编码格式则返回一个 Buffer
示例代码 :
```
const fs = require('fs')
fs.readFile('./file/11.txt','utf-8',function(err,data) {
    console.log(err);
    console.log('--------');
    console.log(data);
})
```


- 写入文件
  - 语法格式 : fs.writeFile(file, data[, options], callback)
  - file：文件路径
  - data：写入内容
  - options：配置选项，包含 encoding, mode, flag；若是字符串则指定编码格式
  - callback：回调函数

示例代码 :
```
const fs = require('fs')

fs.writeFile('./file/2.txt','hello node.js','utf-8',function(err,data) {
    //如果文件写入成功，则err的值等于null    null可以转化为false
    //如果写入文件失败，则err是一个错误对象
    console.log(err);
    if(err) {
        return console.log('文件写入失败' + err.message);
    }
    console.log('文件写入成功');
})
```
- 路径动态拼接问题

  - 在使用 fs 模块操作文件时，如果提供的操作路径是以./ 或 ../开头的相对路径时，容易出现路径动态拼接错误的问题
  - 原因：代码在运行的时候，会以执行 node 命令时所处的目录，动态拼接出被操作文件的完整路径
  - 解决方案：在使用 fs 模块操作文件时，直接提供完整的路径，从而防止路径动态拼接的问题

示例代码　：
```
const fs = require('fs')
fs.readFile(`${__dirname}/file/11.txt`,'utf-8',function(err,data) {
    if(err) {
       return console.log('文件读取失败' + err.message);
    }
    console.log(__dirname);  //D:\node复盘\01
    console.log('文件读取成功!' + data);
})
```


在这里打印了__dirname，我们可以发现，打印出来的路径和我们终端打开文件的绝对路径是一样的，所以这样的话就可以解决我们有时候使用../或./时出现的路径问题了！

- 其他操作
验证路径是否存在：
```
fs.exists(path, callback)
fs.existsSync(path)
```
删除文件：
```
fs.unlink(path, callback)
fs.unlinkSync(path)
```
列出文件：
```
fs.readdir(path[,options], callback)
fs.readdirSync(path[, options])
```
截断文件：
```
fs.truncate(path, len, callback)
fs.truncateSync(path, len)
```
建立目录：
```
fs.mkdir(path[, mode], callback)
fs.mkdirSync(path[, mode])
```
删除目录：
```
fs.rmdir(path, callback)
fs.rmdirSync(path)
```
重命名文件和目录：
```
fs.rename(oldPath, newPath, callback)
fs.renameSync(oldPath, newPath)
```
监视文件更改：
```
fs.watchFile(filename[, options], listener)
```
关闭文件 ：
```
fs.close(fd, callback)
```
### 文件监听
文件监听是非常常用的功能，比如我们修改了文件后webpack重新打包代码或者Node.js服务重启，都用到了文件监听的功能，Node.js提供了两套文件监听的机制。

#### 基于轮询的文件监听机制
基于轮询机制的文件监听API是watchFile。

基于轮询的监听文件机制本质上是不断轮询文件的元数据，然后和上一次的元数据进行对比，如果有不一致的就认为文件变化了，因为第一次获取元数据时，还没有可以对比的数据，所以不认为是文件变化，这时候开启一个定时器。隔一段时间再去获取文件的元数据，如此反复，直到用户调stop函数停止这个行为。

#### 基于inotify的文件监听机制

我们看到基于轮询的监听其实效率是很低的，因为需要我们不断去轮询文件的元数据，如果文件大部分时间里都没有变化，那就会白白浪费CPU。如果文件改变了会主动通知我们那就好了，这就是基于inotify机制的文件监听。Node.js提供的接口是watch。watch的实现和watchFile的比较类似

### Promise Fs模块
Node.js V14中，文件模块支持了Promise化的api。我们可以直接使用await进行文件操作。我们看一下使用例子。
```
    const { open, readFile } = require('fs').promises;  
    async function runDemo() {   
      try {  
        console.log(await readFile('11111.md', { encoding: 'utf-8' }));  
      } catch (e){  

      }  
    }  
    runDemo(); 
```


## Readline

`readline` 模块提供了一个用于从 Readble 的 stream (例如 process.stdin) 中一次读取一行的接口. 当然你也可以用来读取文件或者 net, http 的 stream, 

我们了解一下readline的机制：
- 读取数据：Readline模块从给定的可读流中读取数据，并将数据缓存在内存中，以便进行分行处理。

- 分行处理：一旦有足够的数据被读取到缓存中，Readline模块将尝试将其按行进行分割。这通常是通过在数据中查找换行符（"\n"）或回车符（"\r"）来实现的。

- 触发事件：每当一行数据被成功读取并分割后，Readline模块将触发一个“line”事件，并将该行数据作为参数传递给该事件的回调函数。这样，应用程序可以在每次读取新数据时立即对其进行处理。

- 处理用户输入：除了读取数据外，Readline模块还提供了一个接口，用于处理用户的命令行输入。这通常是通过监听“line”事件来实现的。一旦用户输入了一行文本并按下回车键，Readline模块将触发“line”事件，并将该行文本作为参数传递给该事件的回调函数。这样，应用程序就可以根据用户的输入执行相应的操作。


举一个例子，

### 如何读取用户输入
有两种方法可以从Readline实例中读取用户输入：使用question()方法和监听line事件。接下来我们将分别介绍这两种方法。

### 使用question()方法
question()方法是Readline模块提供的一个便捷的方法，可以在命令行中输出一个提示符，并等待用户输入。一旦用户输入了一行文本并按下回车键，该方法将返回一个Promise对象，该对象的值是用户输入的文本。

```
const readline = require('readline');
const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout
});

rl.question('What is your name? ', (name) => {
    console.log(`Hello, ${name}!`);
    rl.close();
});

```
上述代码使用question()方法输出一个提示符，并等待用户输入姓名。一旦用户输入了姓名并按下回车键，该方法将调用回调函数，并将用户输入的姓名作为参数传递给它。在此示例中，我们将用户输入的姓名作为参数，输出一条欢迎消息，并调用close()方法关闭Readline实例。
