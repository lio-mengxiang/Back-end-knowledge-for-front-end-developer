## 前言
Hi, 大家好, 本系列文章的目的是教你如何通过【字节/阿里/腾讯】B端架构组大前端的面试，这个职位对nodejs有一定的要求，所以我们的通关手册更偏向服务端基础中 Node.js 程序员需要了解的部分。

全网对于这部分知识比较有价值，成系统的文章之前只有【如何通过饿了么 Node.js 面试】，也就是饿了么大前端团队出的面试指南，这个指南所列出来的一些题的方向，是实打实真实面试的问题，所以其参考价值是非常高的。

本文在其基础上大大加强了其内容的深度和广度（每一个话题，都是参考了多篇网上大神的回答，并结合笔者自身使用情况进行了增删补），毫无夸张的说，你想验证自己是否具备大前端的基本能力和应聘大前端职位，只此一家算得上比较系统的文章了。所以，千万别错过呦！

因为涉及的知识点很多，加之作者能力有限，难免有疏漏，并且欢迎在github上欢迎大家讨论交流，一起维护这个项目。

## 目录导读

### [计算机基础](/src/basic.md)

没有基本的计算机基础，了解基本的node api和一些后端常见的概念，比如进程，线程都会有很大的困难，所以了解基本计算机基础知识是必修。

 之前在下总结了一下这部分的基础知识，深度适合没有基础的前端同学。
 
 部分问题：
* [计算机组成原理](/src/basic.md)
  -  字节面试题
      - 什么是补码？有什么用？
      - 一个中文占多少字节？这个跟编码有什么关系？Unicode跟编码有什么关系？js是什么编码？
      - 为什么0.1 + 0.2不等于0.3？请结合IEEE标准来说
  - 阿里面试题
      - CPU和GPU的区别，前端如何利用GPU加速？
      - I/O 是什么？知道I/O设备的演进过程吗？
* [计算机网络](/src/basic.md)
  - 阿里面试题
     - 参考我这之前写的文章[阿里面试官的”说一下从url输入到返回请求的过程“问的难度就是不一样！](https://juejin.cn/post/6928677404332425223)
* [操作系统](/src/basic.md)
  - 腾讯面试题（面试火花思维也遇到过）
     - 进程、线程、协程的区别
     - 进程间通信的方式
     - 为什么要有文件描述符
     - PCB和FCB分别指什么，包含哪些内容
* [数据结构和算法](/src/basic.md)
  - 美团面试题
    - 有效的字母异位词
    - 杨辉三角
  - 蔚来面试题
    - 合并两个有序数组
    - 二叉树层次遍历

## [Js 基础问题](/src/common.md)


与前端 Js 不同, 后端方面除了SSR/爬虫之外很少会接触 DOM, 所以关于 DOM 方面的各种知识基本不会讨论. 浏览器端除了图形业务外很少碰到内存问题, 但是后端几乎是直面服务器内存的, 更加偏向内存方面, 对于一些更基础的问题也会更加关注.

部分问题：
* [类型判断](/src/common.md) 
  - typeof NaN 结果是什么？以及为什么？
* [类型判断](/src/common.md) 
  - 如何精确判断引用类型？
* [原型链](/src/common.md)
  - Function.prototype === Object.__proto__ 结果是什么，以及为什么？
* [作用域](/src/common.md) 
  - javascript是静态作用域还是动态作用域？
* [执行上下文栈](/src/common.md) 
  - 有些人可能要说《高级程序设计》的VO，AO，对不起，这是ES3的说法。答这道题就走远了
* [引用传递](/src/common.md) 
  - js 中什么类型是引用传递, 什么类型是值传递? 如何将值类型的变量以引用的方式传递?

## [模块](/src/module.md) 

模块机制详细分析文章可以参考我之前写的一篇文章[NodeJS有难度的面试题，你能答对几个？](https://juejin.cn/post/6844903951742025736),最开始就讨论了commonjs的模块机制。

* [模块机制]
 - 请介绍一下node里的模块是什么
 - 请介绍一下require的模块加载机制
* [热更新]
 - 如何在不重启 node 进程的情况下热更新一个 js/json 文件? 这个问题本身是否有问题?
* [上下文]
 - 每个require的js文件，作用域如何保证独立呢？
 - 知道vm模块有什么用吗？
* [包管理]
 - npm包管理下，node_modules里安装的包是按什么结构和逻辑整理目录的


## [事件/异步](/src/event-async.md)

异步还是不异步? 这是一个问题.

* [Promise](/src/event-async.md)
  -  问烂的题就是问你什么是宏任务，什么是微任务，有啥区别，然后上个题问你打印顺序
* [Events (事件)]
  - 如下情况是否会死循环?

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
* [阻塞/异步](/src/event-async.md)
  - 如何实现一个 sleep 函数
  - 如何实现一个异步的 reduce? (注:不是异步完了之后同步 reduce)
* [并行/并发](/src/event-async.md)
  - 并行和并发有啥区别



## [进程](/src/process.md)

后面的篇章需要了解操作系统的一些基本运行原理，才能更好的回答问题，比如进程就是一个典型的例子，以下链接是我之前写的关于操作系统的基础知识：
https://juejin.cn/post/6844904112803282957

* [Stream](/src/process.md)
 - 可读流默认是什么状态？
 - 背压是什么？知道pipeline吗？跟pipe有什么区别
* [Process (进程)](/src/process.md)
 - 请回答进程、线程和协程的区别
 - RPC，LPC 中文啥意思？有啥区别吗
* [Child Processes (子进程)](/src/process.md)
 - spawn 、fork、exec、execFile 的区别？
* [Cluster (集群)](/src/process.md)
 - Cluster底层使用的是Child Processes, 那它的工作机制什么
* [进程间通信](/src/process.md)
 - 常见的IPC方式有哪些
* [守护进程](/src/process.md)
 - 请实现一个简易的nodejs编写的守护进程



## [IO](/src/io.md)

Node.js 是以 IO 密集型业务著称. 那么问题来了, 你真的了解什么叫 IO, 什么又叫 IO 密集型业务吗?

* [Buffer](/src/io.md)
  -  Buffer 一般用于处理什么数据? 其长度能否动态变化?
  - Buffer.alloc()和Buffer.allocUnsafe()有什么区别
* [String Decoder (字符串解码)](/src/io.md)
  - 知道 String Decoder的应用场景吗？
* [File System (文件系统)](/src/io.md)
  - 什么是文件描述符? 输入流/输出流/错误流是什么?
* [Readline](/src/io.md)
 - Readline 是如何实现的? (有思路即可)

## [Network](/src/network.md)

* [Net (网络)](/src/network.md)
 - 知道TCP的粘包是什么吗？
 - cookie 与 session 的区别? 服务端如何清除 cookie?
* [UDP/Datagram](/src/network.md)
 - 什么场景适用udp
* [HTTP](/src/network.md)
  - 什么是跨域请求? 如何允许跨域?
  - 列举几个提高网络传输速度的办法?
* [DNS (域名服务器)](/src/network.md)
 - hosts 文件是什么? 什么叫 DNS 本地解析?
* [RPC](/src/network.md)
 - 常见RPC的方式有哪些


## [OS](/src/os.md)

* [TTY](/src/os.md)
 - 什么是 TTY? 如何判断是否处于 TTY 环境?
* [OS (操作系统)](/src/os.md)
 - 不同操作系统的换行符 (EOL) 有什么区别? 
* [Path](/src/os.md)
 - 为什么要用path模块去处理路径（兼容性问题）
* [命令行参数](/src/os.md)


## [错误处理/调试](/src/error.md)

* [`[Doc]` Errors (异常)](/src/error.md)
 - javascript标准错误分为几种
 -  怎么处理未预料的出错? 用 try/catch ，还是其它什么?
 - 什么是 `uncaughtException` 事件? 一般在什么情况下使用该事件? 
 - 什么是防御性编程? 与其相对的 let it crash 又是什么?
 - 为什么有些异常没法根据报错信息定位到代码调用? 如何准确的定位一个异常?
* [`[Doc]` Debugger (调试器)](/src/error.md)
 -  内存泄漏通常由哪些原因导致? 如何分析以及定位内存泄漏?


## [测试](/src/test.md)

* [单元测试](/src/test.md)
 - 为什么要写测试? 写测试是否会拖累开发进度?什么是覆盖率?
 - 平时自己用的单测工具（前端和node部分）
* [压力测试](/src/test.md)
 - 压测的目的是什么


## [存储](/src/storage.md)

* [`[Point]` Mysql](/src/storage.md)
  - 索引有什么用，大致原理是什么? 设计索引有什么注意点?
  - char和varchar字符宽度的区别
  - 字符集utf8和utf8mb4的区别
  - 为什么需要规范sql数据规范化？
* [`[Point]` Mongodb](/src/storage.md)
* [`[Point]` 数据一致性](/src/storage.md)
  - 什么是数据一致性，如何解决
  - 什么是事务，在mysql如何开启


## [安全](/src/security.md)

* [Crypto (加密)](/src/security.md)
  - 知道对称加密和非对称加密吗？https中它们彼此是如何协作的呢？
* [HTTPS](/src/security.md)
  - https解决http的哪些问题？https是如何解决
* [XSS](/src/security.md)
  - 过滤 Html 标签能否防止 XSS?
* [CSRF](/src/security.md)
 - 什么是CSRF，如何防止？
* [中间人攻击](/src/security.md)
 - 如何避免中间人攻击?
* [Sql/Nosql 注入](/src/security.md)
 - 为什么会产生sql注入


## 最后

欢迎大家一起共建这个知识库，以及不吝啬start哦！感谢！