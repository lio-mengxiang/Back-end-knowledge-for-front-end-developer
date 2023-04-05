# 进程

- Stream 流
- Process (进程)
- Child Processes (子进程)
- Cluster (集群)
- 守护进程

## 简述

后面的篇章需要了解操作系统的一些基本运行原理，才能更好的回答问题，比如进程就是一个典型的例子，以下链接是我之前写的关于操作系统的基础知识：
https://juejin.cn/post/6844904112803282957



## Stream 流

流是 nodejs 开发中需要了解的非常重要的概念。记住是非常重要！！！理解流的机制，对于你玩转文件读写至关重要！！！

### 流的概念

我们用现实的例子来解释一下：
假设我们现在要开发一个项目，我们可以

- 等产品、后端、前端所有人都招齐了，才开始开发项目
- 边招产品、后端、前端，一边开发开发
  这两种方式哪种效率更高呢？当然是第二种。这就是流的思想，有一些数据，处理一些。

### 可读流（Readable Stream）

可读流可以从一个地方读取数据，比如从文件中读取信息。读取的数据可以暂时存放在可读流中的缓存（Buffer）里，防止应用程序无法及时处理。

常见的可读流有 `process.stdin`、`fs.createReadStream`以及 HTTP 服务中的 `IncomingMessage`对象。

`Readable Stream` 存在两种模式

- 一种是叫做 `Flowing Mode`，流动模式，在 Stream 上绑定 ondata 方法就会自动触发这个模式，只要流中数据可读，便会立即触发 data 事件，比如：

```javascript
process.stdin
  .on("data", (chunk) => {
    console.log("new Data avaible", chunk);
  })
  .on("end", () => {
    console.log("end");
  });
```

创建 Readable 也可以通过继承的方式：

```javascript
const Stream = require("stream");

class ReadableDong extends Stream.Readable {
  constructor() {
    super();
  }

  _read() {
    this.push(
      "readable内部会调用_read方法，紧接着调用push方法将数据填充到缓存中"
    );
    // push null表示流的终止
    this.push(null);
  }
}

const readableStream = new ReadableDong();

readableStream.on("data", (data) => {
  console.log(data.toString());
});

readableStream.on("end", () => {
  console.log("done~");
});
```

可以使用 pause()方法来临时阻止流触发 data 事件，将接收的数据存到内部缓存中，再调用 resume()方法会继续开始流动模式。

### 可读流的流动模式和暂停模式

需要注意的是所有的可读流一开始都处于暂停模式，要切换为流动模式，可通过以下几种方式实现：

#### 一：注册 'data' 事件

为可读流对象注册一个 'data' 事件，传入事件处理函数，会把流切换为流动模式，在数据可用时会立即把数据块传送给注册的事件处理函数。
上面的案例就是注册 data 事件开启流动模式的

#### 二：stream.pipe() 方法

调用 pipe() 方法将数据发送到可写流。

```javascript
readable.pipe(writeable);
```

可读流的 pipe() 方法实现中也是注册了 'data' 事件，一边读取数据一边写入数据至可写流。

### 三：stream.resume() 方法

stream.resume() 将处于暂停模式的可读流，恢复触发 'data' 事件，切换为流动模式。

```javascript
const http = require("http");
http
  .createServer((req, res) => {
    req.resume();
    req.on("end", () => {
      res.end("Ok!");
    });
  })
  .listen(3000);
```

### 暂停模式

暂停模式也是流一开始时所处的模式，该模式下会触发 'readable' 事件，表示流中有可读取的数据，我们需要不断调用 read() 方法拉取数据，直到返回 null，表示缓冲区中的数据已被耗尽，在 read() 返回 null 后，会再次触发 'readable' 事件，表示仍有可读取的数据，如果此时停止 read() 方法调用，同样的请求也会被挂起。

```javascript
const http = require('http');
http.createServer((req, res) => {
  let data = '';
  let chunk;
  req.on('readable', () => {
    while (null !== (chunk = req.read())) {
      data += chunk.toString();
    }
  })
  req.on('end', () => {
    res.end(data);
  });
```

### 背压问题

背压是指我们读数据很快，一下就把缓冲区充满了，但是写数据很慢，缓冲区一直都是爆满，那么数据过多，就有可能丢失数据。如何解决呢，一般情况下用 pipe 方法即可，并且新版本有 pipeline 方法，它带有错误处理的回调函数，比 pipe 方法更可靠。

```javascript
const { pipeline } = require("stream");
const fs = require("fs");
const zlib = require("zlib");

// Use the pipeline API to easily pipe a series of streams
// together and get notified when the pipeline is fully done.
// A pipeline to gzip a potentially huge video file efficiently:

pipeline(
  fs.createReadStream("The.Matrix.1080p.mkv"),
  zlib.createGzip(),
  fs.createWriteStream("The.Matrix.1080p.mkv.gz"),
  (err) => {
    if (err) {
      console.error("Pipeline failed", err);
    } else {
      console.log("Pipeline succeeded");
    }
  }
);
```

注意！pipe 或 pipeline 只有目标流为 Duplex 流或 Transform 流才可以形成管道链。

### 可写流 (Writable)

可写流的API是：
```javascript
writable.write(chunk，[encoding]，[callback]);
```
如果不许需要更多的数据写入流中，需要使用end方法
```javascript
writable.end([chunk],[encoding], [callback])
```
此时callback函数相当于为finish事件注册监听器，当流中所有数据被清空时，改函数会被执行。

当生产者写入速度过快，把队列池装满了之后，就会出现「背压」，这个时候是需要告诉生产者暂停生产的，当队列释放之后，Writable Stream 会给生产者发送一个 drain 消息，让它恢复生产。下面是一个写入一百万条数据的 Demo：
```javascript
function writeOneMillionTimes(writer, data, encoding, callback) {
    let i = 10000;
    write();
    function write() {
        let ok = true;
        while (i-- > 0 && ok) {
            // 写入结束时回调
            ok = writer.write(data, encoding, i === 0 ? callback : null);
        }
        if (i > 0) {
            // 这里提前停下了，'drain' 事件触发后才可以继续写入 
            console.log('drain', i);
            writer.once('drain', write);
        }
    }
}
```
### 实现pipe方法
pipe方法实际上就是 Readable 和 Writable的结合体。

```javascript
pipe(ws){
    // pipe的时候就已经开始读数据了，读数据的同时还会写数据
    // 如果读的太快
    this.on('data',(chunk)=>{
        let flag = ws.write(chunk);
        if(!flag){
            this.pause();
        }
    });
    ws.on('drain',()=>{
        this.resume();
    })
}
```


### 双向流（Duplex Stream）

Duplex 是可读可写，同时实现 _read 和 _write 就可以了
```
const Stream = require('stream');

var duplexStream = Stream.Duplex();

duplexStream._read = function () {
    this.push('写入数据.');
    this.push(null);
}

duplexStream._write = function (data, enc, next) {
    next();
}

duplexStream.on('data', data => console.log(data.toString()));
duplexStream.on('end', data => console.log('read done~'));

duplexStream.write('写入数据，');

duplexStream.end();

duplexStream.on('finish', data => console.log('write done~'));
```

整合了 Readable 流和 Writable 流的功能，这就是双工流 Duplex。

### Transform
Duplex 流虽然可读可写，但是两者之间没啥关联，而有的时候需要对流入的内容做转换之后流出，这时候就需要转换流 Transform。

```
const Stream = require('stream');

class TransformReverse extends Stream.Transform {

  constructor() {
    super()
  }

  _transform(buf, enc, next) {
    const res = buf.toString().split('').reverse().join('');
    this.push(res)
    next()
  }
}

var transformStream = new TransformReverse();

transformStream.on('data', data => console.log(data.toString()))
transformStream.on('end', data => console.log('read done~'));

transformStream.write('反转数据');

transformStream.end()

transformStream.on('finish', data => console.log('write done~'));
```

## 更深入的流的讨论

[流内部机制](./AdvancedStream.md)

## Process

关于进程，一个非常高频的面试题莫过于，请回答进程、线程和协程的区别！

我这里简述一下：

### 首先为什么需要进程。

简单总结为什么需要进程：
- 增加并发度
- 提高cpu利用率
- 提高计算机系统的稳定性

早期的计算机只支持单道程序（是指所有进程一个一个排队执行，A 进程执行时，CPU、内存、I/O 设备全是 A 进程控制的，等 A 进程执行完了，才换 B 进程，然后对应的资源比如 CPU、内存这些才能换 B 用）。

但是 cpu 是高效率的，而 IO 是低速的，就会出现 cpu 要等待 IO 的情况；从而降低了实际效率。后来就引入多道批处理；而程序在执行的过程中又会因为共享资源而导致程序在执行的过程中相互限制；后来又提出进程这个概念，进程作为资源分配和任务调度的基本单位

单核的 CPU 一次只能执行一个任务，想要实现多任务，需要把 CPU 的运行时间切成一段一段的时间片，每个时间片运行一个程序，循环的分配时间片给不同的应用程序，由于时间片非常的短，在用户看来，就像是多个任务同时在运行

此时进程主要解决了计算机发展早起不能支持并发的问题。

此外，不同的进程之间可以相互独立，不会相互影响，保证了计算机系统的稳定性。

### 那么线程为什么不会出现呢？

简单总结为什么需要线程：
- 增加并发度
- 提高系统效率（上下文切换开销更小，共享进程资源）



- 比如你在玩 QQ 的时候，QQ 是一个进程，如果 QQ 的进程里没有多线程并发，那么 QQ 进程就只能同一时间做一件事情（比如 QQ 打字聊天）
- 但是我们真实的场景是 QQ 聊天的同时，还可以发文件，还可以视频聊天，这说明如果 QQ 没有多线程并发能力，QQ 能够的实用性就大大降低了。所以我们需要线程，也就是需要进程拥有能够并发多个事件的能力。

可以把线程理解为轻量级的进程，线程是一个基本的 CPU 执行单元，这样不仅仅进程可以并发执行，一个进程内的线程也可以并发执行，这样大大提高了系统的并发度。

此时进程的角色变为资源分配的单元。

其次，线程的切换开销比进程小。线程的创建和撤销的开销要比进程小很多，线程间的切换也比进程间的切换更快，这些都可以降低计算机系统的开销，提高系统的效率。

由于同一进程中的线程共享进程的资源，比如内存、文件等，因此线程之间可以更加方便地进行通信和数据共享。同时，线程也需要遵守同步和互斥机制，避免资源竞争和数据不一致等问题。

### 协程又是啥？

一个进程可以包含多个线程，一个线程也可以包含多个协程。

协程不是进程也不是线程，而是一个特殊的函数。这个函数可以在某个地方被“挂起”,并且可以重新在挂起处外继续运行。所以说，协程与进程、线程相比并不是一个维度的概念。

并且一个线程中的多个协程的运行是串行的。

进程和线程的切换都是操作系统控制的。

协程的切换者是用户(编程者或应用程序),切换时机是用户自己的程序来决定的。协程的切换过程只有用户态(即没有陷入内核态),因此切换效率高。

## Node.js中的进程、线程

### 进程

Node.js 里通过 node app.js 开启一个服务进程，如果要开启多进程就需要使用node中child_process模块的fork方法，就绪，fork 出来的每个进程都拥有自己的堆栈，一个进程无法访问另外一个进程里定义的变量、数据结构，只有建立了 IPC 通信，进程之间才可数据共享。

注意：开启多进程不是为了解决高并发，主要是解决了单进程模式下 Node.js CPU 利用率不足的情况，充分利用多核 CPU 的性能。

例如：
```javascript
const http = require('http');

const server = http.createServer();
server.listen(3000,()=>{
    console.log('进程id',process.pid)
})
```

### process 模块

Node.js 中的进程 Process 是一个全局对象，无需 require 直接使用。

- process.env：环境变量，例如通过  process.env.NODE_ENV 获取不同环境项目配置信息
- process.nextTick：事件循环本轮快结束的时候触发
- process.pid：获取当前进程id
- process.ppid：当前进程对应的父进程
- process.cwd()：获取当前进程工作目录，
- process.platform：获取当前进程运行的操作系统平台
- process.uptime()：当前进程已运行时间
- 进程事件：process.on(‘uncaughtException’, cb) 捕获异常信息、process.on(‘exit’, cb）进程推出监听
- 三个标准流：process.stdout 标准输出、process.stdin 标准输入、process.stderr 标准错误输出

### Node.js 进程创建

进程创建有多种方式，本篇文章以child_process模块。

#### child_process模块


- child_process.spawn()：适用于返回大量数据，例如图像处理，二进制数据处理。
- child_process.exec()：适用于小量数据，maxBuffer 默认值为 200 * 1024 超出这个默认值将会导致程序崩溃，数据量过大可采用 spawn。
- child_process.execFile()：类似 child_process.exec()，区别是不能通过 shell 来执行，不支持像 I/O 重定向和文件查找这样的行为
- child_process.fork()： 衍生新的进程，进程之间是相互独立的，每个进程都有自己的 V8 实例、内存，系统资源是有限的，不建议衍生太多的子进程出来，通长根据系统** CPU 核心数**设置。
```
// fork_app.js

const http = require('http');
const fork = require('child_process').fork;

const server = http.createServer((req, res) => {
    if(req.url == '/compute'){
        const compute = fork('./fork_compute.js');
        compute.send('开启一个新的子进程');

        // 当一个子进程使用 process.send() 发送消息时会触发 'message' 事件
        compute.on('message', sum => {
            res.end(`Sum is ${sum}`);
            compute.kill();
        });

        // 子进程监听到一些错误消息退出
        compute.on('close', (code, signal) => {
            compute.kill();
        })
    }else{
        res.end(`ok`);
    }
});
server.listen(3000, 127.0.0.1, () => {
    console.log(`server started at http://${127.0.0.1}:${3000}`);
});
```

```
// fork_compute.js

针对文初需要进行计算的的例子我们创建子进程拆分出来单独进行运算。
const computation = () => {
  // 复杂的计算
};

process.on('message', msg => {
    const sum = computation();

    // 如果Node.js进程是通过进程间通信产生的，那么，process.send()方法可以用来给父进程发送消息
    process.send(sum);
})
```

等下我们讲node.js中的线程，就可以用线程去计算CPU密集型任务了，进程的开销实在太大了。

### cluster原理分析

![image](https://user-images.githubusercontent.com/26076975/229998612-4e5a48d7-10ab-40d3-be44-9c9d2a9f4d44.png)

cluster模块调用fork方法来创建子进程，该方法与child_process中的fork是同一个方法。

cluster模块采用的是经典的主从模型，Cluster会创建一个master，然后根据你指定的数量复制出多个子进程，可以使用cluster.isMaster属性判断当前进程是master还是worker(工作进程)。由master进程来管理所有的子进程，主进程不负责具体的任务处理，主要工作是负责调度和管理。

cluster模块使用内置的负载均衡来更好地处理线程之间的压力，该负载均衡使用了Round-robin算法（也被称之为循环算法）。当使用Round-robin调度策略时，master accepts()所有传入的连接请求，然后将相应的TCP请求处理发送给选中的工作进程（该方式仍然通过IPC来进行通信）。

开启多进程时候端口疑问讲解：如果多个Node进程监听同一个端口时会出现 Error:listen EADDRIUNS的错误，而cluster模块为什么可以让多个子进程监听同一个端口呢?原因是master进程内部启动了一个TCP服务器，而真正监听端口的只有这个服务器，当来自前端的请求触发服务器的connection事件后，master会将对应的socket具柄发送给子进程。

### child_process 模块与cluster 模块总结

无论是 child_process 模块还是 cluster 模块，为了解决 Node.js 实例单线程运行，无法利用多核 CPU 的问题而出现的。核心就是父进程（即 master 进程）负责监听端口，接收到新的请求后将其分发给下面的 worker 进程。

cluster模块的一个弊端：

![image](https://user-images.githubusercontent.com/26076975/229999174-fa64a96c-24d2-4b74-8ebd-4fd81a47ec38.png)
![image](https://user-images.githubusercontent.com/26076975/229999192-c15c4dba-c8a0-4973-ad17-1ede50cb7af9.png)

cluster内部隐时的构建TCP服务器的方式来说对使用者确实简单和透明了很多，但是这种方式无法像使用child_process那样灵活，因为一直主进程只能管理一组相同的工作进程，而自行通过child_process来创建工作进程，一个主进程可以控制多组进程。原因是child_process操作子进程时，可以隐式的创建多个TCP服务器。

#### Node.js进程通信原理

实现进程间通信的技术有很多，如命名管道，匿名管道，socket，信号量，共享内存，消息队列等。Node中实现IPC通道是依赖于libuv。windows下由命名管道(name pipe)实现，*nix系统则采用Unix Domain Socket实现。表现在应用层上的进程间通信只有简单的message事件和send()方法，接口十分简洁和消息化。

IPC创建和实现示意图

![image](https://user-images.githubusercontent.com/26076975/229999570-6ce1d44b-84b5-4462-bb51-e1133190e45b.png)
## Node.js多进程架构模型

编写主进程
master.js 主要处理以下逻辑：

- 创建一个 server 并监听 3000 端口。
- 根据系统 cpus 开启多个子进程
- 通过子进程对象的 send 方法发送消息到子进程进行通信
- 在主进程中监听了子进程的变化，如果是自杀信号重新启动一个工作进程。
- 主进程在监听到退出消息的时候，先退出子进程在退出主进程

```
// master.js
const fork = require('child_process').fork;
const cpus = require('os').cpus();

const server = require('net').createServer();
server.listen(3000);
process.title = 'node-master'

const workers = {};
const createWorker = () => {
    const worker = fork('worker.js')
    worker.on('message', function (message) {
        if (message.act === 'suicide') {
            createWorker();
        }
    })
    worker.on('exit', function(code, signal) {
        console.log('worker process exited, code: %s signal: %s', code, signal);
        delete workers[worker.pid];
    });
    worker.send('server', server);
    workers[worker.pid] = worker;
    console.log('worker process created, pid: %s ppid: %s', worker.pid, process.pid);
}

for (let i=0; i<cpus.length; i++) {
    createWorker();
}

process.once('SIGINT', close.bind(this, 'SIGINT')); // kill(2) Ctrl-C
process.once('SIGQUIT', close.bind(this, 'SIGQUIT')); // kill(3) Ctrl-\
process.once('SIGTERM', close.bind(this, 'SIGTERM')); // kill(15) default
process.once('exit', close.bind(this));

function close (code) {
    console.log('进程退出！', code);

    if (code !== 0) {
        for (let pid in workers) {
            console.log('master process exited, kill worker pid: ', pid);
            workers[pid].kill('SIGINT');
        }
    }

    process.exit(0);
}
```
#### 工作进程

worker.js 子进程处理逻辑如下：

- 创建一个 server 对象，注意这里最开始并没有监听 3000 端口
- 通过 message 事件接收主进程 send 方法发送的消息
- 监听 uncaughtException 事件，捕获未处理的异常，发送自杀信息由主进程重建进程，子进程在链接关闭之后退出
```
// worker.js
const http = require('http');
const server = http.createServer((req, res) => {
	res.writeHead(200, {
		'Content-Type': 'text/plan'
	});
	res.end('I am worker, pid: ' + process.pid + ', ppid: ' + process.ppid);
	throw new Error('worker process exception!'); // 测试异常进程退出、重启
});

let worker;
process.title = 'node-worker'
process.on('message', function (message, sendHandle) {
	if (message === 'server') {
		worker = sendHandle;
		worker.on('connection', function(socket) {
			server.emit('connection', socket);
		});
	}
});

process.on('uncaughtException', function (err) {
	console.log(err);
	process.send({act: 'suicide'});
	worker.close(function () {
		process.exit(1);
	})
})
```



## 浏览器、nodejs 的 event loop 以及 process.nextTick

首先回答这个被问的烂之又烂的浏览器 event loop 执行机制：

具体流程：

- js 引擎将所有代码放入执行栈，在执行的过程中，不同的任务（同步任务，宏任务，微任务）将采取不同的策略执行。
- 发现宏任务会交给对应的线程去执行，比如异步 fetch 请求交给网络线程，浏览器线程在正确的时机(比如定时器最短延迟时间)将宏任务的消息(或称之为回调函数)推入宏任务队列。
- 微任务同理，但是微任务始终比宏任务先执行。
- 当执行栈为空时，也就是同步任务执行完毕，eventLoop 转到微任务队列处，依次弹出每个任务放入执行栈并执行，如果在执行的过程中又有微任务产生则推入队列末尾，这样循环直到微任务队列为空。
- 宏任务同理。但是宏任务是每次执行一个。
- 重复 1-5 过程
- ...直到栈和队列都为空时，代码执行结束。引擎休眠等待直至下次任务出现。

接着看有一个问烂的问题，nodejs 的 event loop 机制，但是我们会穿插一些问题，并不是网上常问的问题。

比如，nodejs 的 event loop 中 poll 阶段做了什么？
比如，process.nextTick 是否属于事件循环中的某一阶段？它的执行机制是什么。
比如，node 11 版本后，事件循环跟之前的版本发生什么变化？

如下是 eventloop 的概览图

```
   ┌───────────────────────┐
┌─>│        timers         │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │     I/O callbacks     │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │     idle, prepare     │
│  └──────────┬────────────┘      ┌───────────────┐
│  ┌──────────┴────────────┐      │   incoming:   │
│  │         poll          │<─────┤  connections, │
│  └──────────┬────────────┘      │   data, etc.  │
│  ┌──────────┴────────────┐      └───────────────┘
│  │        check          │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
└──┤    close callbacks    │
   └───────────────────────┘
```

各个阶段干啥，网上资料满大街，我们这里着重提一下注意事项

### poll 阶段

首先我们的 js 代码不是上来就走 timer 的，这里就可以问面试者，我们的 js 代码被 v8 解析后的代码走的哪个事件循环的阶段。

当个 v8 引擎将 js 代码解析后传入 libuv 引擎后，循环首先进入 poll 阶段。

poll 阶段的执行逻辑如下：

如果 event loop 进入了 poll 阶段，且代码未设定 timer，将会发生下面情况：

- 如果 poll queue 不为空，event loop 将同步的执行 queue 里的 callback,直至 queue 为空，或执行的 callback 到达系统上限;
- 如果 poll queue 为空，将会发生下面情况：
  - 如果代码已经被 setImmediate()设定了 callback, event loop 将结束 poll 阶段进入 check 阶段，并执行 check 阶段的 queue (check 阶段的 queue 是 setImmediate 设定的)
  - 如果代码没有设定 setImmediate(callback)，event loop 将阻塞在该阶段等待 callbacks 加入 poll queue;
- 如果 event loop 进入了 poll 阶段，且代码设定了 timer：

  - 如果 poll queue 进入空状态时（即 poll 阶段为空闲状态），event loop 将检查 timers,如果有 1 个或多个 timers 时间时间已经到达，event loop 将按循环顺序进入 timers 阶段，并执行 timer queue.

所以说 poll 就是轮询，一直看有没有能干的活。

### 接着我们看看`process.nextTick。

`process.nextTick()`不在 event loop 的任何阶段执行，而是在各个阶段切换的中间执行,即从一个阶段切换到下个阶段前执行

我们举个例子：

```javascript
var fs = require("fs");

fs.readFile(__filename, () => {
  setTimeout(() => {
    console.log("setTimeout");
  }, 0);
  setImmediate(() => {
    console.log("setImmediate");
    process.nextTick(() => {
      console.log("nextTick3");
    });
  });
  process.nextTick(() => {
    console.log("nextTick1");
  });
  process.nextTick(() => {
    console.log("nextTick2");
  });
});
```

打印

```bash
-> node eventloop.js
nextTick1
nextTick2
setImmediate
nextTick3
setTimeout
从poll —> check阶段，先执行process.nextTick，
nextTick1
nextTick2
然后进入check,setImmediate，
setImmediate
执行完setImmediate后，出check,进入close callback前，执行process.nextTick
nextTick3
最后进入timer执行setTimeout
setTimeout
```

process.nextTick()是 node 早期版本无 setImmediate 时的产物，node 作者推荐我们尽量使用 setImmediate

### node 11 版本后，事件循环跟之前的版本发生什么变化？

- 定时器和微任务在 NodeV11 版本后的变更
  随着 Node V11 的发布，nextTick 回调和 Promise 微任务将会在各个独立的 setTimeout and setImmediate 之间运行，也就是说一次运行一个宏任务，然后清空微任务队列。



- RPC，LPC 中文啥意思？有啥区别吗?
- 进程间通信，spwan 和 fork 的进程间通信有啥区别
- 参数 stdio 是什么


## Node的进程

### process 模块

Node.js 中的进程 Process 是一个全局对象，无需 require 直接使用，给我们提供了当前进程中的相关信息。

- process.env：环境变量，例如通过  process.env.NODE_ENV 获取不同环境项目配置信息
- process.nextTick：这个在谈及 Event Loop 时经常为会提到
- process.pid：获取当前进程id
- process.ppid：当前进程对应的父进程
- process.cwd()：获取当前进程工作目录，
- process.platform：获取当前进程运行的操作系统平台
- process.uptime()：当前进程已运行时间，例如：pm2 守护进程的 uptime 值
- 进程事件：process.on(‘uncaughtException’, cb) 捕获异常信息、process.on(‘exit’, cb）进程推出监听
- 三个标准流：process.stdout 标准输出、process.stdin 标准输入、process.stderr 标准错误输出
- process.title 指定进程名称，有的时候需要给进程指定一个名称

以上仅列举了部分常用到功能点，除了 Process 之外 Node.js 还提供了 child_process 模块用来对子进程进行操作，在下文 Nodejs进程创建会继续讲述。
### Node.js 进程创建


child_process 是 Node.js 的内置模块，用来fork进程。


几个常用函数：
四种方式

- child_process.spawn()：适用于返回大量数据，例如图像处理，二进制数据处理。
- child_process.exec()：适用于小量数据，maxBuffer 默认值为 200 * 1024 超出这个默认值将会导致程序崩溃，数据量过大可采用 spawn。
- child_process.execFile()：类似 child_process.exec()，区别是不能通过 shell 来执行，不支持像 I/O 重定向和文件查找这样的行为
- child_process.fork()： 衍生新的进程，进程之间是相互独立的，每个进程都有自己的 V8 实例、内存，系统资源是有限的，不建议衍生太多的子进程出来，通长根据系统** CPU 核心数**设置。


CPU 核心数这里特别说明下，fork 确实可以开启多个进程，但是并不建议衍生出来太多的进程，cpu核心数的获取方式const cpus = require('os').cpus();,这里 cpus 返回一个对象数组，包含所安装的每个 CPU/内核的信息，二者总和的数组哦。假设主机装有两个cpu，每个cpu有4个核，那么总核数就是8。

fork开启子进程 Demo
fork开启子进程解决文章起初的计算耗时造成线程阻塞。
在进行 compute 计算时创建子进程，子进程计算完成通过 send 方法将结果发送给主进程，主进程通过 message 监听到信息后处理并退出。

fork_app.js
```
const http = require('http');
const fork = require('child_process').fork;

const server = http.createServer((req, res) => {
    if(req.url == '/compute'){
        const compute = fork('./fork_compute.js');
        compute.send('开启一个新的子进程');

        // 当一个子进程使用 process.send() 发送消息时会触发 'message' 事件
        compute.on('message', sum => {
            res.end(`Sum is ${sum}`);
            compute.kill();
        });

        // 子进程监听到一些错误消息退出
        compute.on('close', (code, signal) => {
            console.log(`收到close事件，子进程收到信号 ${signal} 而终止，退出码 ${code}`);
            compute.kill();
        })
    }else{
        res.end(`ok`);
    }
});
server.listen(3000, 127.0.0.1, () => {
    console.log(`server started at http://${127.0.0.1}:${3000}`);
});
```
fork_compute.js

针对文初需要进行计算的的例子我们创建子进程拆分出来单独进行运算。
```
const computation = () => {
    let sum = 0;
    console.info('计算开始');
    console.time('计算耗时');

    for (let i = 0; i < 1e10; i++) {
        sum += i
    };

    console.info('计算结束');
    console.timeEnd('计算耗时');
    return sum;
};

process.on('message', msg => {
    console.log(msg, 'process.pid', process.pid); // 子进程id
    const sum = computation();

    // 如果Node.js进程是通过进程间通信产生的，那么，process.send()方法可以用来给父进程发送消息
    process.send(sum);
})
```

## child.kill 与 child.send

常见会问的面试题, 如 `child.kill` 与 `child.send` 的区别. 二者一个是基于信号系统, 一个是基于 IPC.

## 父进程或子进程的死亡是否会影响对方? 什么是孤儿进程?

子进程死亡不会影响父进程, 不过子进程死亡时（线程组的最后一个线程，通常是“领头”线程死亡时），会向它的父进程发送死亡信号. 反之父进程死亡, 一般情况下子进程也会随之死亡, 但如果此时子进程处于可运行态、僵死状态等等的话, 子进程将被`进程1`（init 进程）收养，从而成为孤儿进程. 另外, 子进程死亡的时候（处于“终止状态”），父进程没有及时调用 `wait()` 或 `waitpid()` 来返回死亡进程的相关信息，此时子进程还有一个 `PCB` 残留在进程表中，被称作僵尸进程.


## 守护进程

这一章删掉了，因为现在node的部署不太需要pm2等这些过时的守护进程了，进阶学习k8s的内容吧，k8s功能太强大了，而且因为k8s的存在，也不需要node的cluster模块了，用k8s组件node的集群更靠谱。


参考
- 深入浅出的Node.js
- [深入理解Node.js中的进程和线程](https://juejin.cn/post/6844903908385488903#heading-3)
