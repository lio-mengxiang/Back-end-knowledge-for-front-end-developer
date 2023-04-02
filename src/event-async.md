# 事件/异步

- [`[Basic]` Promise]
- [`[Doc]` Events (事件)]
- [`[Doc]` Timers (定时器)]
- [`[Point]` 阻塞/异步]
- [`[Point]` 并行/并发]

## 简述

异步还是不异步? 这是一个问题.

## Promise

Promise 问烂的题就是问你什么是宏任务，什么是微任务，有啥区别，然后上个题问你打印顺序。如下：

```javascript
setTimeout(function () {
  console.log(1);
}, 0);
new Promise(function executor(resolve) {
  console.log(2);
  for (var i = 0; i < 10000; i++) {
    i == 9999 && resolve();
  }
  console.log(3);
}).then(function () {
  console.log(4);
});
console.log(5);
```

好了，热身结束，来个难点的题，这个题不会没有关系，有专门的文章帮你弄清为啥。

```javascript
Promise.resolve()
  .then(() => {
    console.log(0);
    return Promise.resolve(4);
  })
  .then((res) => {
    console.log(res);
  });

Promise.resolve()
  .then(() => {
    console.log(1);
  })
  .then(() => {
    console.log(2);
  })
  .then(() => {
    console.log(3);
  })
  .then(() => {
    console.log(5);
  })
  .then(() => {
    console.log(6);
  });
```

答案是：0 1 2 3 4 5 6，至于为啥，你着重要明白

```javascript
.then(() => {
    console.log(0)
    return Promise.resolve(4)
  })
```

这里面其实创建了 3 个微任务，是不是有点吃惊，有兴趣的同学看下文连接的讲解吧，比饿了么的原本的问题要深入的多。
https://juejin.cn/post/6950093219153575972

## Events

`Events` 是 Node.js 中一个非常重要的 core 模块, 在 node 中有许多重要的 core API 都是依赖其建立的. 比如 `Stream` 是基于 `Events` 实现的, 而 `fs`, `net`, `http` 等模块都依赖 `Stream`, 所以 `Events` 模块的重要性可见一斑.

本质上这就是一个发布订阅模式的应用，可以出一个题，大致写一下发布订阅模块。我们接着看 Events 模块。

下面的题目可以作为普通面试题，很简单。

如下情况是否会死循环?

```javascript
const EventEmitter = require("events");

let emitter = new EventEmitter();

emitter.on("myEvent", () => {
  console.log("hi");
  emitter.emit("myEvent");
});

emitter.emit("myEvent");
```

以及这样会不会死循环?

```javascript
const EventEmitter = require("events");

let emitter = new EventEmitter();

emitter.on("myEvent", function sth() {
  emitter.on("myEvent", sth);
  console.log("hi");
});

emitter.emit("myEvent");
```

使用 emitter 处理问题可以处理比较复杂的状态场景, 比如 TCP 的复杂状态机, 做多项异步操作的时候每一步都可能报错, 这个时候 .emit 错误并且执行某些 .once 的操作可以将你从泥沼中拯救出来.

第一个例子都会死循环，原因如下：

“myEvent”设置了事件侦听器，当触发时，它将“hi”记录到控制台并再次发出“myEvent”。这将创建“myEvent”触发自身并将“hi”记录到控制台的无限循环。


## 阻塞/异步

这个出两个题进入下一小节吧：

1、如何实现一个 sleep 函数?

```javascript
function sleep(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}
```

2、如何实现一个异步的 reduce? (注:不是异步完了之后同步 reduce)

需要了解 reduce 的情况, 是第 n 个与 n+1 的结果异步处理完之后, 在用新的结果与第 n+2 个元素继续依次异步下去

```javascript
function asyncReduce(listPromise) {
  listPromise.reduce((pre, next) => {
    return pre.then(() => listPromise());
  }, Promise.resolve());
}
```

## Event loop

一般会问浏览器和nodejs里的event loop分别是啥。

我们简单概述一下：

为什么浏览器有event loop这个东西。

我们知道javascript为了简化操作难度，设计为单线程，为啥单线程能降低操作难度呢？

假定JavaScript同时有两个线程，一个线程在某个DOM节点上添加内容，另一个线程删除了这个节点，这时浏览器应该以哪个线程为准？

所以，为了避免复杂性，从一诞生，JavaScript就是单线程，这已经成了这门语言的核心特征，将来也不会改变。

问题又来了，如果单线程的话，定时器和网络请求阻塞，我们js等半天，这用户体验也太差了吧。

所以就需要一个处理异步的机制，把 JS 代码分为同步和异步任务。

定时器、网络请求其实都是在别的线程执行的，执行完了之后把结果放在任务队列里，主线程一直循环，反正异步任务队列有东西就取出来。

这很像订阅发布模式，订阅事件，调用回调，所以事件就像任务一样，我们这个loop称之为event loop。

而此时浏览器为了能够让异步任务队列有优先级之分，也就是优先级高的先执行，就设计了宏任务和微任务，典型的宏任务就是setTimeout和setInterval，微任务就是promise后then注册的微任务。

这就是浏览器event loop的基本面貌。



设计 Loop 机制和 Task 队列是为了支持异步，解决逻辑执行阻塞主线程的问题，设计 MicroTask 队列的插队机制是为了解决高优任务尽早执行的问题

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

关于事件循环, Timers 以及 nextTick 的关系详见官方文档 The Node.js Event Loop, Timers, and process.nextTick(): [英文](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)， [论坛中文讨论](https://cnodejs.org/topic/57d68794cb6f605d360105bf) 以及 [Tasks, microtasks, queues and schedules](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)

## 并行/并发

并行 (Parallel) 与并发 (Concurrent) 是两个很常见的概念.

可以看 Erlang 作者 Joe Armstrong 的博客 ([Concurrent and Parallel](http://joearms.github.io/2013/04/05/concurrent-and-parallel-programming.html))

![con_and_par](http://joearms.github.io/images/con_and_par.jpg)

并发 (Concurrent) = 2 队列对应 1 咖啡机.

并行 (Parallel) = 2 队列对应 2 咖啡机.

Node.js 通过事件循环来挨个抽取事件队列中的一个个 Task 执行, 从而避免了传统的多线程情况下 `2个队列对应 1个咖啡机` 的时候上下文切换以及资源争抢/同步的问题, 所以获得了高并发的成就.

至于在 node 中并行, 你可以通过 cluster 来再添加一个咖啡机.
