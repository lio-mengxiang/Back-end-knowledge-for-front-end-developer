Node.js官方文档中这样描述AsyncLocalStorage：AsyncLocalStorage在回调函数和promise链中创建异步的状态。它允许在 Web 请求的整个生命周期或任何其他异步操作中存储数据

<a name="DAswt"></a>
# 为什么我们需要异步代码中共享上下文数据
在传统的基于线程的服务器模型中，每个请求都会分配一个新的线程来处理。其中有一个概念叫** **thread-local storage** **,** **是一种变量的存储方法，这个变量在它所在的线程内是全局可访问的，但是不能被其他线程访问到，这样就保持了数据的线程独立性。

在PHP中，每一个http请求都会创建一个独立的线程，每个线程有自己的memory，

然而，在 Node.js 的事件驱动模型中，所有的请求都在一个单线程的事件循环中处理。这就意味着在异步操作中无法直接共享上下文数据，因为每个异步操作都在不同的时间点执行，无法通过简单的变量或参数传递来共享数据。

<a name="a28yu"></a>
# 小试牛刀，快速理解AsyncLocalStorage的作用


```javascript
import { AsyncLocalStorage } from "async_hooks";

export const storage = new AsyncLocalStorage();

async function test() {
  console.log(storage.getStore());
}

async function run(id) {
  const state = { id };

  return storage.run(state, async () => {
    await test();
  });
}

run(1);
run(2);
run(3);

```

- 首先我们引入 AsyncLocalStorage 模块
- 然后实例化 AsyncLocalStorage 得到实例 storage
- 然后run是主函数，在里面，我们使用 storage.run 共享了state数据，每一个run函数运行都会有一个独立的state数据
- storage.run的第二个参数是一个回调函数，其中运行的异步任务可以共享state数据

所以上面打印
```javascript
run(1); // { id: 1 }
run(2); // { id: 2 }
run(3); // { id: 3 }
```

基本到这里，大家应该对AsyncLocalStorage的基本用法很熟悉了，接下来列举两个案例来巩固
<a name="sUaon"></a>
# 案例1：跟踪和记录
AsyncLocalStorage 的一种常见用法是用来跟踪和日志记录。 当发生错误时，必须拥有所有可用的相关信息，例如错误id，以快速调试问题。 实现此目的的一种方法是将唯一标识符 (traceId) 传递给请求生命周期期间生成的所有日志。 

logging.js
```javascript
import { AsyncLocalStorage } from "async_hooks";

const asyncLocalStorage = new AsyncLocalStorage();

export function withTraceId(traceId, fn) {
  return asyncLocalStorage.run(traceId, fn);
}

export function log(message) {
  const traceId = asyncLocalStorage.getStore();
  console.log(`${message} traceId=${traceId}`);
}
```
 app.js
```javascript
import express from "express";
import { withTraceId, log } from "./logging";

app.use((req, res, next) => {
  const traceId = req.header("x-trace-id") || Math.random().toString(36).substring(7);
  withTraceId(traceId, next);
});

app.use((req, res, next) => {
  log(`processing request ${req.url}`);
  next();
});

app.get("/post/:postId", async (req, res) => {
  const post = await fetchPost(req.params.postId);
  res.send(post);
});

async function fetchPost(postId: string) {
  log(`fetching data for post id: ${postId}`);
  // some logic to fetch post
  return `Hello world, ${postId}`;
}
```

<a name="tfU19"></a>
# 案例2：鉴权
auth.js
```javascript
async function authenticate(authToken) {
  // fetch user data with authToken
  return {
    id: 1,
    username: "john",
  };
}

async function authMiddleware(req, res, next) {
  const authToken = req.header("x-auth-token");
  if (!authToken) {
    next();
  }

  authenticate(authToken!)
    .then((user) => {
      return asyncLocalStorage.run(user, next as any);
    })
    .catch(next);
}

function getCurrentUser() {
  return asyncLocalStorage.getStore() as any;
}

```

secret.js
```javascript
async function getUserSecret() {
  const user = getCurrentUser();
  // some async operation to fetch data for user.id
  const secret = `secret of ${user.username}`;
  return secret;
}
```

app.js
```javascript
app.use(authMiddleware);

app.get("/secret", async (req, res) => {
  const user = getCurrentUser();
  if (!user) {
    res.status(401).send("Unauthorized");
    return;
  }
  const secret = await getUserSecret();
  res.send(secret);
});
```

AsyncLocalStorage 的另一个用例是身份验证。 当用户登录时，您需要将用户的信息存储在某处，以便您可以在应用程序的不同部分访问它。 使用 AsyncLocalStorage，您可以存储用户的信息并在需要时检索它。



