# 错误处理/调试

* `[Doc]` Errors (异常)
* `[Doc]` Debugger (调试器)




## Errors

其中标准的 JavaScript 错误常见有：

* [EvalError](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/EvalError): 调用 eval() 出现错误时抛出该错误
* [SyntaxError](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/SyntaxError): 代码不符合 JavaScript 语法规范时抛出该错误
* [RangeError](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RangeError): 数组越界时抛出该错误
* [ReferenceError](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ReferenceError): 引用未定义的变量时抛出该错误
* [TypeError](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypeError): 参数类型错误时抛出该错误
* [URIError](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/URIError): 误用全局的 URI 处理函数时抛出该错误

我们来举例介绍下上面的错误：

#### 1.SyntaxError
解析代码时发生的语法错误
```javascript
var 2b;
//Uncaught SyntaxError: Unexpected number
```
#### 2.ReferenceError
引用了一个不存在的变量
```javascript
console.log(a);
//Uncaught ReferenceError: Invalid left-hand side in assignment
```
#### 3.RangeError
超出有效范围
```javascript
var a= new Array(-1);
//Uncaught RangeError: Invalid array length
```
#### 4.TypeError
情况一：变量或参数不是预期类型，比如，对字符串、布尔值、数值等原始类型的值使用new命令，就会抛出这种错误，因为new命令的参数应该是一个构造函数。
```javascript
var a= new abc;
//Uncaught TypeError: abc is not a function
```
情况二：调用对象不存在的方法
```javascript
var b;
b.c();
//Uncaught TypeError: Cannot read property c of undefined
```
#### 5.URLError（URL错误）
与url相关函数参数不正确，主要是encodeURI()、decodeURI()、encodeURIComponent()
、decodeURIComponent()、escape()和unescape()这六个函数。
```javascript
decodeURI('%2')
//Uncaught URIError: URI malformed
```
#### 6.EvalError（eval错误）
eval函数没有被正确执行
```javascript
eval(2b)
//Uncaught SyntaxError: Invalid or unexpected token
```
#### 抛出自定义错误: throw new Error("错误信息")

如果不想使用系统设置的错误信息(例如前面提到的6种)，可以自定义错误，例如让一个函数需要传入一个字符串，但是传入了空值，可以new不同的错误类型，并自定义错误提示语来让系统抛出信息。
```javascript
function check(string){
    if(!string){
        throw new Error("内容不存在");
        //throw new TypeError("内容不存在")
    }
}
```
> <a name="q-handle-error"></a> 怎么处理未预料的出错? 用 try/catch , domains 还是其它什么?

这个话题推荐一篇腾讯同学写的文章[nodejs中错误捕获的一些最佳实践](https://imweb.io/topic/5846d2069be501ba17b10a8d)。

这里简单描述一下：

首先我们要区分操作错误和编码错误：

操作错误是指比如服务器返回500，这并不是bug，对我们程序员来说，代码出错，比如typeerror就是编码错误。
#### 如何处理错误

对于明确的操作错误类型，直接处理掉。

例如尝试打开一个log文件可能会导致 ENOENT ，那么创建这个文件即可。

对于预料之外你不知道如何处理的错误，比较好的方式是记录error并crash，传递合适的错误信息给客户端

#### 如何处理 代码错误
最好的方式是立即crash。

这种错误是程序的bug，一般来说写再多的代码也避免不了。因为在node应用中，我们一般会监控挂掉的进程并自动重启，所以立即crash是比较好的方式。

调试这类问题的最佳方式，是在捕获到uncaught exception的时候，记录相关信息。

总之记住，server的代码错误（bug）传递到client时会成为一个操作错误，例如server捕获到uncaught exception则返回一个500，客户端来处理这个操作错误。

#### 如何传递错误？
首先，最重要的是文档，描述这个函数做了些什么，接收什么类型的参数返回什么，可能会触发什么错误。

一些基本原则：

- 同步的函数里，使用throw。使用者使用try...catch即可捕获错误。
- 异步函数里，更常用的方式是使用callback(err, result)的方式。
- 在更复杂的场景里，可以返回一个EventEmitter对象，代替使用callback。使用者可以监听emitter对象的 error事件。 例如读取一个数据流，我们可能会同时使用 req.on('data')、req.on('error')、req.on('timeout') 。
- 所以，使用throw还是callbacks、EventEmitter，取决于：

该错误是操作错误还是编码错误？
该函数是同步还是异步？

> 为什么要在 cb 的第一参数传 error? 为什么有的 cb 第一个参数不是 error, 例如 http.createServer?

- 我们首先要知道, 回调函数的第一个元素传 error 属于一个约定俗成的,而且错误优先逻辑上也是对的，比如这个回调的参数有3个，error放到最后的话就是第4个参数，回调参数不确定，error的顺序也不确定，写起来有点反人类。

- http.createServer中传入的函数并不是回调函数，而是给request事件的监听函数。


### 错误栈丢失

```javascript
function test() {
  throw new Error('test error');
}

function main() {  
  test();
}

main();
```

可以收获报错:

```javascript
/data/node-interview/error.js:2
  throw new Error('test error');
  ^

Error: test error
    at test (/data/node-interview/error.js:2:9)
    at main (/data/node-interview/error.js:6:3)
    at Object.<anonymous> (/data/node-interview/error.js:9:1)
    at Module._compile (module.js:570:32)
    at Object.Module._extensions..js (module.js:579:10)
    at Module.load (module.js:487:32)
    at tryModuleLoad (module.js:446:12)
    at Function.Module._load (module.js:438:3)
    at Module.runMain (module.js:604:10)
    at run (bootstrap_node.js:394:7)
```

可以发现报错的行数, test 函数, main 函数的调用关系都在 stack 中清晰的体现.

当你使用 setImmediate 等定时器来设置异步的时候:

```javascript
function test() {
  throw new Error('test error');
}

function main() {  
  setImmediate(() => test());
}

main();

```

我们发现

```javascript
/data/node-interview/error.js:2
  throw new Error('test error');
  ^

Error: test error
    at test (/data/node-interview/error.js:2:9)
    at Immediate.setImmediate (/data/node-interview/error.js:6:22)
    at runCallback (timers.js:637:20)
    at tryOnImmediate (timers.js:610:5)
    at processImmediate [as _immediateCallback] (timers.js:582:5)
```

错误栈中仅输出到 test 函数内调用的地方位置, 再往上 main 的调用信息就丢失了. 也就是说如果你的函数调用深度比较深的情况下, 你使用异步调用某个函数出错了的情况下追溯这个异步的调用是一个很困难的事情, 因为其之上的栈都已经丢失了. 如果你用过 [async](https://github.com/caolan/async) 之类的模块, 你还可能发现, 报错的 stack 会非常的长而且曲折, 光看 stack 很难去定位问题.

这在项目不大/作者清楚的情况下不是问题, 但是当项目大起来, 开发人员多起来之后, 这样追溯错误会变得异常痛苦. 关于这个问题, 在上文中提到 [错误处理的最佳实践](https://cnodejs.org/topic/55714dfac4e7fbea6e9a2e5d) 中, 关于 `编写新函数的具体建议` 那一带的内容有描述到. 通过使用 [verror](https://www.npmjs.com/package/verror) 这样的方式, 让 Error 一层层封装, 并在每一层将错误的信息一层层的包上, 最后拿到的 Error 直接可以从 message 中获取用于定位问题的关键信息.

以昨天的数据为准（2017-3-13）各位只要对比一下看看 npm 上上个月 [verror](https://www.npmjs.com/package/verror) 的下载量 `1100w` 比 [express](https://www.npmjs.com/package/express) 的 `1070w` 还高. 应该就能感受到这种写法有多流行了.

对于verror，我们这里举一个例子：
```javascript
const VError = require('verror')

function model(json) {
  return JSON.parse(json)
}

function controller(json) {
  try {
    model(json)
  } catch (err) {
    const error = new VError(err, 'Model fail to parse json')
    throw error
  }
}

function routeHandler(rawJSON) {
  try {
    const data = controller(rawJSON)
    return data
  } catch (err) {
    const error = new VError(err, 'Controller fail to use json')
    throw error
  }
}

routeHandler('invalid json')
```

此脚本将生成以下错误消息：
`
VError：Controller fail to use json: Model fail to parse json: Unexpected token i in JSON at position 0
`

这比：`SyntaxError: Unexpected token i in JSON at position 0` 更明确


### uncaughtException

当异常没有被捕获一路冒泡到 Event Loop 时就会触发该事件 process 对象上的 `uncaughtException` 事件. 默认情况下, Node.js 对于此类异常会直接将其堆栈跟踪信息输出给 `stderr` 并结束进程, 而为 `uncaughtException` 事件添加监听可以覆盖该默认行为, 不会直接结束进程.

```javascript
process.on('uncaughtException', (err) => {
  console.log(`Caught exception: ${err}`);
});

setTimeout(() => {
  console.log('This will still run.');
}, 500);

// Intentionally cause an exception, but don't catch it.
nonexistentFunc();
console.log('This will not run.');
```

#### 合理使用 uncaughtException

`uncaughtException` 的初衷是可以让你拿到错误之后可以做一些回收处理之后再 process.exit. 官方的同志们还曾经讨论过要移除该事件 (详见 [issues](https://github.com/nodejs/node-v0.x-archive/issues/2582))

所以你需要明白 `uncaughtException` 其实已经是非常规手段了, 应尽量避免使用它来处理错误. 因为通过该事件捕获到错误后, 并不代表 `你可以愉快的继续运行 (On Error Resume Next)`. 程序内部存在未处理的异常, 这意味着应用程序处于一种未知的状态. 如果不能适当的恢复其状态, 那么很有可能会触发不可预见的问题. 

如果在 `.on` 指定的监听回调中报错不会被捕获, Node.js 的进程会直接终端并返回一个非零的退出码, 最后输出相应的堆栈信息. 否则, 会出现无限递归. 除此之外, 内存崩溃/底层报错等情况也不会被捕获.

所以官方建议的使用 `uncaughtException` 的正确姿势是在结束进程前使用同步的方式清理已使用的资源 (文件描述符、句柄等) 然后 process.exit. 

在 uncaughtException 事件之后执行普通的恢复操作并不安全. 官方建议是另外在专门准备一个 monitor 进程来做健康检查并通过 monitor 来管理恢复情况, 并在必要的时候重启 (所以官方是含蓄的提醒各位用 pm2 之类的工具).


### unhandledRejection

当 Promise 被 reject 且没有绑定监听处理时, 就会触发该事件. 该事件对排查和追踪没有处理 reject 行为的 Promise 很有用.

该事件的回调函数接收以下参数：

* `reason` [`<Error>`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error) | `<any>` 该 Promise 被 reject 的对象 (通常为 Error 对象)
* `p` 被 reject 的 Promise 本身

例如

```javascript
process.on('unhandledRejection', (reason, p) => {
  console.log('Unhandled Rejection at: Promise', p, 'reason:', reason);
  // application specific logging, throwing an error, or other logic here
});

somePromise.then((res) => {
  return reportToUser(JSON.pasre(res)); // note the typo (`pasre`)
}); // no `.catch` or `.then`
```

以下代码也会触发 `unhandledRejection` 事件：

```javascript
function SomeResource() {
  // Initially set the loaded status to a rejected promise
  this.loaded = Promise.reject(new Error('Resource not yet loaded!'));
}

var resource = new SomeResource();
// no .catch or .then on resource.loaded for at least a turn
```

> In this example case, it is possible to track the rejection as a developer error as would typically be the case for other 'unhandledRejection' events. To address such failures, a non-operational `.catch(() => { })` handler may be attached to resource.loaded, which would prevent the 'unhandledRejection' event from being emitted. Alternatively, the 'rejectionHandled' event may be used.



## Debugger

debugger是node开发必备技巧，同时也可以用来帮助你调试源码，无论是工作还是学习，都是必须掌握的能力。

我们这里主要讲的是用vscode 来debug

运行 nodejs 代码的时候，如果带上了 `--inspect`（可以打断点） 或者 -`-inspect-brk`（可以打断点，并在首行断住） 的参数，那么 debugger 的模式就启动了一个websocket server。

然后通过websocket来进行通信。

vscode里，我们在项目的根目录下设置.vscode/launch.json文件，就恶意配置调试的参数了。

常用的参数说明如下：
#### attach
刚才我们已经说了，`--inspect`参数可以开启一个websocket server，按道理来说我们只要用客户端连接上去就可以进行调试了，这个attach就是附上去的意思。例如如下配置：
```javascript
{
  "configurations": [
    {
      "name": "Attach",
      "port": 3000,
      "request": "attach",
      "type": "node"
    }
  ]
}
```
然后点击vscode编辑器右侧的debug按钮，就可以连接3000端口的websocket server了。

#### launch

如何客户端和服务端一起启动呢，可以借助launch参数。
```javascript
{
  "configurations": [
    {
      "name": "Launch",
      "port": 3000,
      "program": "${workspaceFolder}/index.js", // 启动程序入口文件 必须使用绝对路径
      "request": "launch",
      "type": "node",
      "skipFiles": [
        "${workspaceFolder}/node_modules/**/*.js", // 调试时不进入node_modules中的程序
        "<node_internals>/**", // 跳过内部node模块程序
      ],
    }
  ]
}
```
`${workspaceFolder}`是根目录的意思，还有很多其他变量，大家可以搜网上的vscode调试文章，也可以直接去官网看debug这一章。

#### 调试ts配置
```javascript
{
    "version": "0.2.0",
    "configurations": [
      {
        "name": "ts-node", // 自定义名称
        "type": "node", // 内置特定执行器
        "request": "launch",
        "env": {
          "NODE_ENV": "test", // 设置node环境变量 process.env.NODE_ENV 可以获取到这个值
        },
        "runtimeArgs": [
          "-r",
          "ts-node/register", // 加载模块 ts-node/register
          "-r",
          "tsconfig-paths/register" // 加载模块 tsconfig-paths/register
        ],
        "skipFiles": [
          "${workspaceFolder}/modules/assistant/node_modules/**/*.js", // 调试时跳过node_modules中的程序 必须使用绝对路径
          "<node_internals>/**", // 跳过内部node模块程序
        ],
        "cwd": "${workspaceFolder}/modules/assistant", // 对应runtimeArgs中找的模块的路径
        "protocol": "inspector",
        "program": "./test/photography.spec.ts", // 拼接在cwd的路径后面或者使用绝对路径
        "internalConsoleOptions": "openOnSessionStart" // 此属性控制调试会话期间调试控制台面板的可见性
      }
    ]
}
```

这里顺带提一嘴，我们前端框架是react，如何调试react项目呢，我们来一个简单的配置供大家参考：

- 下载vscode插件 debugger for chrome extension
- 配置文件（注意，此时的type是chrome，表示浏览器环境）
```javascript
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "chrome",
            "request": "launch",
            "name": "Launch Chrome against localhost",
            "url": "http://localhost:3000",
            "webRoot": "${workspaceRoot}"
        }
    ]
}
```



