文档分为3个部分

- [常用的API](#常用api)
- [API注意事项](#api注意事项)
- [不常用的API简单介绍](#不常用的api简单介绍)

<a name="kFCNg"></a>
# 常用API
<a name="IZVoh"></a>
## memoryUsage
| API 名称    | **API 介绍**                          |
| ----------- | ------------------------------------- |
| memoryUsage | 测量node.js的内存使用情况，单位是字节 |

案例：
```javascript
import { memoryUsage } from 'node:process';

console.log(memoryUsage());
// Prints:
// {
//  rss: 4935680,
//  heapTotal: 1826816,
//  heapUsed: 650472,
//  external: 49879,
//  arrayBuffers: 9386
// }
```

- rss（Resident Set Size）：表示进程当前的常驻内存集（包括指令、堆栈和堆）的总大小，以字节（byte）为单位。
- heapTotal：表示V8引擎堆的总大小，以字节为单位。
- heapUsed：表示V8引擎堆当前使用的内存量，以字节为单位。
- external：表示V8引擎管理的外部内存使用量，以字节为单位。
- arrayBuffers 指的是给 ArrayBuffers and SharedArrayBuffers分配的内存, 包含所有的 Node.js [Buffer](https://nodejs.org/docs/latest/api/buffer.html)s. 也包括 external 的值。

<a name="YY2zR"></a>
## process.constrainedMemory()
| API 名称          | **API 介绍**                                                                                                        |
| ----------------- | ------------------------------------------------------------------------------------------------------------------- |
| constrainedMemory | 根据操作系统施加的限制获取进程可用的内存量（以字节为单位）。 如果不存在这样的约束，或者约束未知，则返回 undefined。 |


为什么要注意这个api呢，例如一般在64位操作系统。v8能使用的内存是1.4GB，但是linux的cgroup实现对资源的限制，可能拿不到1.4GB的内存。

<a name="Gn8Ne"></a>
## process.kill(pid[, signal])
| API 名称 | **API 介绍**                                                                                                         |
| -------- | -------------------------------------------------------------------------------------------------------------------- |
| kill     | process.kill() 方法将信号发送到对应第一个参数（pid） 的进程。<br />pid是信号名称是诸如“SIGINT”或“SIGHUP”之类的字符串 |


```javascript
import process, { kill } from 'node:process';

process.on('SIGHUP', () => {
  console.log('Got SIGHUP signal.');
});

setTimeout(() => {
  console.log('Exiting.');
  process.exit(0);
}, 100);

kill(process.pid, 'SIGHUP');
```

这里我会把信号的知识补充一下。<br />当 Node.js 进程收到信号时，则将触发信号事件。<br />注意：信号在 Worker 线程上不可用。<br />每个事件的名称将是信号的大写通用名称（例如 'SIGINT' 表示 SIGINT 信号）。案例如下：
```javascript
import process from 'node:process';

// 从标准输入开始读取，因此进程不会退出。
process.stdin.resume();

process.on('SIGINT', () => {
  console.log('Received SIGINT. Press Control-D to exit.');
});

// 使用单个函数处理多个信号
function handle(signal) {
  console.log(`Received ${signal}`);
}

process.on('SIGINT', handle);
process.on('SIGTERM', handle);
```

- 'SIGUSR1' 启动 node.js debuuger的信号。
- 'SIGTERM' 和 'SIGINT' 在非 Windows 平台上具有默认处理的处理函数，如果你的其在使用代码 128 + signal number 退出之前重置终端模式。 如果这些信号之一安装了监听函数，则其默认行为将被删除（Node.js 将不再退出）。
- 'SIGPIPE' 默认情况下忽略。 它可以安装监听函数。
- 'SIGHUP' 在 Windows 上是在关闭控制台窗口时生成，在其他平台上是在各种类似条件下生成。 它可以安装监听函数，但是 Node.js 将在大约 10 秒后被 Windows 无条件地终止。 在非 Windows 平台上，SIGHUP 的默认行为是终止 Node.js，但一旦安装了监听函数，则其默认行为将被删除。
- 'SIGTERM' Windows 上不支持，可以监听。
- 所有平台都支持来自终端的 'SIGINT'，通常可以使用 **Ctrl**+**C** 生成（但是这是可配置的）。 当启用[终端原始模式](http://url.nodejs.cn/Ts6uDc)并使用 **Ctrl**+**C** 时不会生成它。
- 'SIGBREAK' 在 Windows 上，当按下 **Ctrl**+**Break** 时会发送。 在非 Windows 平台上，它可以被监听，但无法发送或生成它。
- 'SIGWINCH' 当调整控制台大小时会发送。 在 Windows 上，这只会发生在当光标移动时写入控制台，或者当在原始模式下使用可读的终端时。
- 'SIGKILL' 不能安装监听函数，它会无条件地终止所有平台上的 Node.js。
- 'SIGSTOP' 不能安装监听函数。
- 0 可以发送来测试进程是否存在，如果进程存在则没影响，如果进程不存在则抛出错误。

Windows 不支持信号，因此没有等价的使用信号来终止，但 Node.js 提供了一些对 process.kill() 和 subProcess.kill() 的模拟：

- 发送 SIGINT、SIGTERM、和 SIGKILL 会导致目标进程无条件的终止，之后子进程会报告进程被信号终止。
- 发送信号 0 可以作为独立于平台的方式来测试进程是否存在。

<a name="nNnT7"></a>
## exit: [Function: exit]

- code [<integer>](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Number_type) | [<string>](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#String_type) | [<null>](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Null_type) | [<undefined>](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Undefined_type) 退出码. 如果传入的是字符串，必须是数字字符串 (e.g.,'1') . **默认是:** 0.

process.exit()      通过一个退出状态码通知Node.js去同步终止进程，如果没有传参数，退出码要么是0，表示成功退出，要么是process.exitCode 设置的退出码，Node.js会等到所有的 exit 监听函数调用后终止。

To exit with a 'failure' code:
```javascript
const { exit } = require('node:process');

exit(1);
```

调用 process.exit() 会让Node.js进程被强制尽快结束，即使仍然有异步操作在等待。<br />在大多数情况下，不需要直接调用 process.exit() ，node.js会在没有额外的任务在event loop时自己退出。<br />以下是一个错用 exit 的例子：
```
const { exit } = require('node:process');

// This is an example of what *not* to do:
if (someConditionNotMet()) {
  printUsageToStdout();
  exit(1);
}
```
问题在于 process.stdout 有可能是异步的，exit 会强制在这些异步调用前就让node.js进程退出了。

在node.js 发生错误时，throw 没有捕获的错误比调用 process.exit() 更安全

在worker 线程中，这个方法会终止当前的线程而不是进程，以下是一些对退出码的罗列（了解即可，一般用退出码1就行了）<br />当没有更多异步操作挂起时，NodeJS 通常会以 0 状态代码退出。 在其他情况下使用以下状态代码：

- 1**未捕获的致命异常**：存在未捕获的异常，并且其没有被域或 'uncaughtException' 事件句柄处理。
- 2: 未使用（由 Bash 保留用于内置误用）
- 3**内部 JavaScript 解析错误**：NodeJS 引导过程中的内部 JavaScript 源代码导致解析错误。 这是极其罕见的，通常只能在 NodeJS 本身的开发过程中发生。
- 4**内部 JavaScript eval失败**：NodeJS 引导过程中的内部 JavaScript 源代码在评估时未能返回函数值。 这是极其罕见的，通常只能在 NodeJS 本身的开发过程中发生。
- 5**致命错误**：V8 中存在不可恢复的致命错误。 通常将打印带有前缀 FATAL ERROR 的消息到标准错误。
- 6**非函数的内部异常句柄**：存在未捕获的异常，但内部致命异常句柄不知何故设置为非函数，无法调用。
- 7**内部异常句柄运行时失败**：存在未捕获的异常，并且内部致命异常句柄函数本身在尝试处理时抛出错误。 例如，如果 'uncaughtException' 或 domain.on('error') 句柄抛出错误，就会发生这种情况。
- 8: 未使用。 在以前版本的 NodeJS 中，退出码 8 有时表示未捕获的异常。
- 9**无效参数**：指定了未知选项，或者提供了需要值的选项而没有值。
- 10**内部 JavaScript 运行时失败**：NodeJS 引导过程中的内部 JavaScript 源代码在调用引导函数时抛出错误。 这是极其罕见的，通常只能在 NodeJS 本身的开发过程中发生。
- 12**无效的调试参数**：设置了 --inspect 和/或 --inspect-brk 选项，但选择的端口号无效或不可用。
- 13**未完成的顶层等待**：在顶层代码中的函数外使用了 await，但传入的 Promise 从未解决。
- >128**信号退出**：如果 NodeJS 收到致命的信号，例如 SIGKILL 或 SIGHUP，则其退出码将是 128 加上信号代码的值。 这是标准的 POSIX 实践，因为退出码被定义为 7 位整数，并且信号退出设置高位，然后包含信号代码的值。 例如，信号 SIGABRT 的值是 6，因此预期的退出码将是 128 + 6 或 134。

<a name="x1fCV"></a>
## nextTick: [Function: nextTick]
process.nextTick() 添加回调函数到"next tick 队列"。这个队列的回调会在当前js函数调用完成并且event loop没有结束的情况下一一调用。如果nextTick里嵌套调用nextTick，可能会造成无线循环。

nextTick必须结合事件循环，你才能真正了解它的作用，所以建议大家去看我写的node.js 工作流详解，这篇文章。
<a name="Jdl0H"></a>
# API注意事项
首先，process是一个全局的对象，不需要require或者import引入。
<a name="hQI7g"></a>
### beforeExit 事件注意事项
beforeExit 会在Node.js 的 event loop结束，并且没有额外的任务，会发出这个事件。 通常，当没有安排工作时，Node.js 进程将退出，但在“beforeExit”事件上注册的侦听器可以进行异步调用，从而使 Node.js 进程继续运行。

注意显示调用 process.exit() 退出进程的时候，不会触发 beforeExit 事件。<br />而且beforeExit可以在回调注册异步函数，相对的是在exit 事件里的异步函数不会被执行。

<a name="CeXAc"></a>
### exit 事件注意事项

exit事件会在如下其中一种条件下触发：

- 一是显示调用 process.exit()
- 二是event loop 结束

注意事项，上面已经提到过了 exit 触发的回调函数必须执行的是同步操作，否则不会执行。

<a name="BY6k7"></a>
### uncaughtException 、unhandledRejection 和  rejectionHandled 的区别
同步代码中, 'uncaughtException' 事件在没有捕获的错误中触发（不包含promise，所以是同步代码触发）.<br />在异步代码中，'unhandledRejection' 会在没有处理 rejections 的时候，例如promise的rejections触发，'rejectionHandled' 事件会在未处理的事件处理后触发。如下案例：
```javascript
process.on("unhandledRejection", (reason, promise) => {
  console.log("Unhandled Rejection at:", promise);
  console.log("Reason:", reason);
  // 这里可以进行自定义的处理逻辑
});

// 创建一个被拒绝的 Promise，但没有提供错误处理器
const rejectedPromise = Promise.reject(new Error("Promise rejected"));

process.on("rejectionHandled", (promise) => {
  console.log("Rejection rejectionHandled:", promise);
  // 这里可以进行自定义的处理逻辑
});

setTimeout(() => {
  // 在一段时间后，我们提供了错误处理器
  rejectedPromise.catch((error) => {
    // 对错误进行处理
  });
}, 1000);

```

<a name="gyOMf"></a>
# cpuUsage
process.cpuUsage()返回用户态和内核态当前node.js进程花费的时间，但是在多核cpu时，时间可能不准确
<a name="pzlYq"></a>
# 不常用的API简单介绍
以下是举例，注释是对属性的说明
```javascript
version: 'v18.17.1' , // node.js 版本 
  versions: { // node.js和它的依赖包版本
    node: '18.17.1',
    acorn: '8.8.2',
    ada: '2.5.0',
    ares: '1.19.1',
    brotli: '1.0.9',
    cldr: '43.0',
    icu: '73.1',
    llhttp: '6.0.11',
    modules: '108',
    napi: '9',
    nghttp2: '1.52.0',
    nghttp3: '0.7.0',
    ngtcp2: '0.8.1',
    openssl: '3.0.10+quic',
    simdutf: '3.2.12',
    tz: '2023c',
    undici: '5.22.1',
    unicode: '15.0',
    uv: '1.44.2',
    uvwasi: '0.0.18',
    v8: '10.2.154.26-node.26',
    zlib: '1.2.13.1-motley'
  },
   arch: 'arm64', // 架构
  platform: 'darwin', // 平台
  release: { // 当前发布版本的metadata，例如  sourceUrl 是一个指向.tar.gz文件的URL，这个压缩包包含当前版本的node安装包
    name: 'node',
    lts: 'Hydrogen',
    sourceUrl: 'https://nodejs.org/download/release/v18.17.1/node-v18.17.1.tar.gz',
    headersUrl: 'https://nodejs.org/download/release/v18.17.1/node-v18.17.1-headers.tar.gz'
  },
  config: [Getter/Setter], // 编译可执行的node.js文件的配置信息
  dlopen: [Function: dlopen], // 加载 c++ addon的方法，
  uptime: [Function: uptime], // 当前node程序运行时间
  cpuUsage: [Function: cpuUsage], // 返回当前进程的用户和系统 CPU 时间使用情况
  resourceUsage: [Function: resourceUsage],
  hrtime: [Function: hrtime] { bigint: [Function: hrtimeBigInt] }, // 一般用于测量某段代码花费的时间
  getuid: [Function: getuid], // 返回当前进程的用户标识符（UID）。UID是一个唯一的整数，用于标识特定的用户。
  getgid: [Function: getgid], // 返回当前进程的组标识符（GID）。GID是一个唯一的整数，用于标识特定的用户组
  getgroups: [Function: getgroups],// 返回当前进程附加组id，这个需要具备基本linux权限基础才能理解
  allowedNodeEnvironmentFlags: [Getter/Setter],// 返回一组传入node.js启动命令的环境变量，例如--inspect-brk是用来debug node程序的
  setUncaughtExceptionCaptureCallback: [Function (anonymous)], // 函数设置一个函数，当发生未捕获的异常时将调用该函数
  hasUncaughtExceptionCaptureCallback: [Function: hasUncaughtExceptionCaptureCallback], // 监测是否设置了 setUncaughtExceptionCaptureCallback
  emitWarning: [Function: emitWarning], // 打印自定义的warning信息，并且可以被process的waring事件监听
  stdout: [Getter], // 连接标准可写流
  stdin: [Getter], // 连接标准可读流
  stderr: [Getter], // 连接标准错误流 
  abort: [Function: abort], // Node.js process 立即退出
  chdir: [Function (anonymous)], // 更改 Node.js 进程的当前工作目录，如果失败（例如，如果指定的目录不存在），则抛出异常
  cwd: [Function: wrappedCwd], // 返回当前process运行的工作目录
  initgroups: [Function: initgroups], // 返回/etc/group 中该用户所属的所有组，但需要root权限运行node.js process
  env: { // 环境变量
    TERM_PROGRAM: 'Apple_Terminal',
    SHELL: '/bin/zsh',
    TERM: 'xterm-256color',
    TMPDIR: '/var/folders/4g/5wdk7dy56x76p6lkh4c0v6bm0000gn/T/',
    TERM_PROGRAM_VERSION: '445',
    TERM_SESSION_ID: '067246E3-2D6C-4797-BB3D-3178492AE3C0',
    USER: 'mengxiang',
    SSH_AUTH_SOCK: '/private/tmp/com.apple.launchd.CvBYNiisO3/Listeners',
    PATH: '/Users/mengxiang/.nvm/versions/node/v18.17.1/bin:/Library/Frameworks/Python.framework/Versions/2.7/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/go/bin',
    __CFBundleIdentifier: 'com.apple.Terminal',
    PWD: '/Users/mengxiang/Desktop/npm',
    XPC_FLAGS: '0x0',
    XPC_SERVICE_NAME: '0',
    SHLVL: '1',
    HOME: '/Users/mengxiang',
    LOGNAME: 'mengxiang',
    OLDPWD: '/Users/mengxiang/Desktop/npm',
    NVM_DIR: '/Users/mengxiang/.nvm',
    NVM_CD_FLAGS: '-q',
    NVM_NODEJS_ORG_MIRROR: 'https://nodejs.org/dist',
    NVM_IOJS_ORG_MIRROR: 'https://iojs.org/dist',
    MANPATH: '/Users/mengxiang/.nvm/versions/node/v18.17.1/share/man:/Library/Frameworks/Python.framework/Versions/2.7/share/man:/usr/local/share/man:/usr/share/man:/Library/Apple/usr/share/man:/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/share/man:/Library/Developer/CommandLineTools/usr/share/man',
    NVM_BIN: '/Users/mengxiang/.nvm/versions/node/v18.17.1/bin',
    LANG: 'zh_CN.UTF-8',
    _: '/Users/mengxiang/.nvm/versions/node/v18.17.1/bin/node',
    __CF_USER_TEXT_ENCODING: '0x1F5:0x19:0x34'
  },
  argv: [ '/Users/mengxiang/.nvm/versions/node/v18.17.1/bin/node' ],// 返回运行时的参数
  pid: 3797, // 返回进程id
  ppid: 3555,// 返回父进程pid
  execPath: '/Users/mengxiang/.nvm/versions/node/v18.17.1/bin/node', // process.execPath 属性返回启动 Node.js 进程的可执行文件的绝对路径名
  debugPort: 9229, // 指定node.js的debugPort
  exitCode: undefined, // node.js process退出码，process.exit()不传入参数会默认用这个退出码
  report: [Getter], // node.js 自带的分析报告功能
}
```

<a name="IAQzz"></a>
