## 一、Node.js 概念
Node.js 是一个开源与跨平台的 JavaScript 运行时环境。 
在浏览器外运行 V8 JavaScript 引擎（Google Chrome 的内核），利用事件驱动、非阻塞和异步I/O模型等技术提高性能。 
我们可以理解为：Node.js 就是一个服务器端的、非阻塞式I/O的、事件驱动的JavaScript运行环境。 

Node.js 的**异步**能力是其高性能的基石，而这一切的核心就是事件循环（Event Loop）。它使得 Node.js 能够以单线程处理成千上万的并发连接，而不会阻塞。  
我们可以用一个简单的类比来理解：想象一家只有一个服务员（**单线程主线程**）的餐厅。服务员接到顾客的点餐（**异步任务**）后，不会在厨房门口等着菜做好，而是把单子交给厨房（**libuv/操作系统**），然后立刻去服务下一位顾客。当厨房的菜做好后（**任务完成**），会通知服务员，服务员再将菜端给对应的顾客（**执行回调**）。

**核心特点**
* 单线程：Node.js 使用单线程处理请求
* 事件循环：通过事件驱动机制处理并发
* 非阻塞I/O：I/O 操作不会阻塞主线程
* 跨平台：可以在 Windows、Linux、macOS 等系统上运行

**Node.js 适用场景？**
- 实时应用（如聊天、推送）
- 高并发 I/O 密集型服务（如 API 网关、代理服务器）
- 前端构建工具（Webpack、Vite）
- ‌不适合 CPU 密集型任务‌（因单线程特性）‌‌

**如何判断运行环境是浏览器还是 Node.js？**
```js
// 判断浏览器环境
if (typeof window !== 'undefined' && typeof window.document !== 'undefined') {
  console.log('是浏览器环境');
}
// 判断process
if (typeof process !== 'undefined' && process.versions && process.versions.node) {
  console.log('Node.js');
}
```

###  核心机制：单线程与非阻塞 I/O
Node.js 的 JavaScript 代码运行在单个主线程上。当遇到耗时的 I/O 操作（如文件读取、网络请求）时，主线程不会傻等，而是：
1. 将该操作委托给底层的 libuv 库。
2. libuv 会利用操作系统的内核能力或其内部的线程池来在后台执行这个任务。
3. 主线程立即返回，继续执行后续的代码。
4. 当后台任务完成后，libuv 会将该任务对应的回调函数放入一个任务队列中。
5. 事件循环会持续监控，一旦主线程的调用栈清空，它就会从任务队列中取出回调函数并执行。


#### 事件循环的六个阶段
事件循环不是一个简单的队列，而是一个包含多个阶段的无限循环。每一轮循环被称为一个 Tick。它按固定顺序遍历以下六个阶段：  
1. timers (定时器阶段)
  - 执行到期的 setTimeout() 和 setInterval() 回调。
2. pending callbacks (待处理回调阶段)
  - 执行某些被推迟到下一轮循环的 I/O 回调，例如一些 TCP 连接错误。
3. idle, prepare (空闲，准备阶段)
  - Node.js 内部使用，开发者通常无需关心。
4. poll (轮询阶段)
  - 这是最重要的阶段。事件循环在这里获取新的 I/O 事件，并执行几乎所有与 I/O 相关的回调（如 fs.readFile、网络请求回调）。
  - 如果队列为空且没有 `setImmediate` 任务，事件循环会在此阶段阻塞等待新的 I/O 事件，这避免了 CPU 资源的浪费。
5. check (检查阶段)
  - 专门用于执行 `setImmediate()` 的回调。
6. close callbacks (关闭回调阶段)
  - 执行一些关闭事件的回调，例如 `socket.on('close', ...)`。

### 宏任务与微任务：理解执行优先级
除了上述的六个阶段，理解任务优先级是掌握事件循环的关键。任务分为宏任务（Macrotask）和微任务（Microtask）。  
#### 宏任务 (Macrotasks)
宏任务是相对较大的、耗时的操作。事件循环的每个阶段都对应一个宏任务队列。  
- `setTimeout` / `setInterval`
- `setImmediate`
- I/O 操作（文件、网络等）

#### 微任务 (Microtasks)
微任务是更紧急、优先级更高的任务。它们**不属于**事件循环的任何一个阶段，而是在每个**宏任务执行完毕后、下一个宏任务开始前**被立即执行。
- `Promise.then()`, `.catch()`, `.finally()`
- `async/await` (在 `await` 之后的代码)
- `queueMicrotask()`

#### 特权任务：process.nextTick()
`process.nextTick()` 拥有一个特殊的、优先级最高的队列。它甚至会在微任务之前执行。事件循环在每个阶段切换的间隙，都会优先清空 process.nextTick() 队列。

### 执行顺序总结
一个简化的执行流程如下：  
1. 执行所有同步代码。
2. 清空 process.nextTick() 队列。
3. 清空微任务队列。
4. 从事件循环的某个阶段取出一个宏任务执行。
5. 重复步骤 2 和 3。
6. 进入下一个事件循环阶段，重复步骤 4。

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 

## 二、EventEmitter做了什么？
Node.js 的 ‌EventEmitter‌ 是内置模块 events 提供的核心类，用于注册监听器和触发事件。它是一个构造函数，我们需要先创建一个EventEmitter实例，然后才能使用它的方法。

### 核心概念
- ‌事件（Event）‌：程序中发生的特定动作或状态变化（如“请求到达”、“文件打开”）。
- ‌监听器（Listener）‌：绑定到事件上的回调函数，事件触发时自动执行。
- ‌发射器（Emitter）‌：通过 EventEmitter 实例发出事件。

### 基本使用方法
1. 引入模块‌

```javascript
const EventEmitter = require('events');
```

2. ‌创建实例‌. 
```javascript
const myEmitter = new EventEmitter();
```

3. ‌注册监听器‌  
`on(event, listener)`：持续监听事件。  
`once(event, listener)`：仅触发一次后自动移除。
```javascript
// 持续监听
myEmitter.on('greet', (name) => {
  console.log(`Hello, ${name}!`);
});
// 仅触发一次后自动移除
myEmitter.once('init', () => {
  console.log('Initialized once.');
});
```

4. ‌触发事件‌
```javascript
myEmitter.emit('greet', 'Alice'); // 输出: Hello, Alice!
myEmitter.emit('init');           // 输出: Initialized once.
myEmitter.emit('init');           // 无输出（已自动移除）
```

### 实际应用场景
1. 自定义事件类

```js
class MyEmitter extends EventEmitter {}
const emitter = new MyEmitter();
emitter.on('custom', () => console.log('Custom event fired!'));
emitter.emit('custom');

```

2. 模拟异步流程解耦
```js
const fs = require('fs');
const fileEmitter = new EventEmitter();

fs.readFile('data.txt', (err, data) => {
  if (err) {
    fileEmitter.emit('error', err);
  } else {
    fileEmitter.emit('dataReady', data);
  }
});

fileEmitter.on('dataReady', (data) => {
  console.log('Data loaded:', data.toString());
});

```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 

## 三、Node.js常见陷阱与错误处理

```js
process.on('uncaughtException', (err) => {
  console.error('Unhandled error:', err);
});
```
```js
process.on('unhandledRejection', (reason, promise) => {
  console.error('Unhandled rejection:', reason);
});
```

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 

## 四、如何在 Node.js 中创建一个返回 Hello World 的简单服务器？
```js
const http = require("http");
http
  .createServer(function (request, response) {
    response.writeHead(200, { "Content-Type": "text/plain" });
    response.end("Hello World\n");
  })
  .listen(3000);

const  http = require(‘http’);

const server = http.createServer((request, response) => {
	response.setHeader(‘Content-Type’, ‘text/plain’, ‘charset=uff-8’);
	response.end(‘hello world’);
});

server.listen(3000);
```

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 

## 五、Node.js 如何处理CORS 跨域？

### 1、原生 Node.js（不依赖框架） 

```js
const http = require('http');

const server = http.createServer((req, res) => {
  // 允许的来源（生产环境建议指定具体域名，不要用 *）
  res.setHeader('Access-Control-Allow-Origin', 'http://localhost:3000');
  
  // 允许的 HTTP 方法
  res.setHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS');
  
  // 允许的请求头
  res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization');
  
  // 是否允许携带 Cookie
  res.setHeader('Access-Control-Allow-Credentials', 'true');

  // 处理预检请求（OPTIONS）
  if (req.method === 'OPTIONS') {
    res.writeHead(204);
    res.end();
    return;
  }

  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({ message: 'Hello' }));
});

server.listen(8080);
```

### 2、使用 cors 包（推荐）
```js
const http = require('http');
const cors = require('cors')();

const server = http.createServer((req, res) => {
  cors(req, res, () => {
    res.writeHead(200);
    res.end('ok');
  });
});
```


### 3、Egg.js 框架

Egg.js 没有内置 CORS 插件，需要安装官方插件：  
1. 安装插件 
`npm install egg-cors --save`

2. 启用插件
```js
// config/plugin.js
exports.cors = {
  enable: true,
  package: 'egg-cors',
};
```

3. 配置跨域规则
```js
// config/config.default.js
exports.cors = {
  // 允许的来源（数组或字符串，* 表示允许所有）
  origin: 'http://localhost:3000',
  
  // 是否允许携带 Cookie（设为 true 时 origin 不能为 *）
  credentials: true,
  
  // 允许的 HTTP 方法
  allowMethods: 'GET,HEAD,PUT,POST,DELETE,PATCH,OPTIONS',
  
  // 允许的请求头
  allowHeaders: 'Content-Type,Authorization,X-Requested-With',
  
  // 预检请求缓存时间
  maxAge: 86400,
};
```

4. 环境差异化配、
```js
// config/config.local.js（本地开发）
exports.cors = {
  origin: '*',  // 开发环境放宽限制
  credentials: false,
};

// config/config.prod.js（生产环境）
exports.cors = {
  origin: 'https://your-frontend.com',
  credentials: true,
};
```

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 

## 六、Node.js 中间件是什么？有哪些用途？

Node.js 中间件（Middleware）是一种函数机制，用于在 HTTP 请求到达路由处理程序之前（或响应发送给客户端之前），对请求和响应对象进行加工、拦截或执行某些通用逻辑。基本上是任何不属于业务逻辑的部分。  
一句话：**中间件 = 可复用的请求/响应拦截器**

### 用途：
1. 日志记录：记录每个请求的 URL、耗时
2. 身份验证：检查用户是否登录、Token 是否有效
3. 数据解析：解析 JSON 请求体、表单数据
4. 错误处理：统一捕获和处理异常
5. CORS 跨域：设置跨域响应头
6. 静态文件服务


- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 

## 七、Node.js的异步问题

1. 回调地狱（Callback Hell）
嵌套层级深、代码横向膨胀，逻辑难以跟踪：

2. 错误处理繁琐
每个回调都要手动检查 err，容易遗漏，且没有统一的异常捕获机制（try/catch 对异步回调无效）

3. 控制流复杂
并行执行、串行执行、竞态条件等需要借助 async 等库，原生实现很笨拙。

4. 代码复用困难
回调的签名不统一，中间结果传递靠闭包，模块化程度低。

### 解决方案
- Promise	ES6 标准: 链式调用，统一错误处理（.catch()）  
- async/await	ES2017: 基于 Promise 的语法糖，写同步风格的异步代码  
- util.promisify：	Node.js 内置，将 callback API 包装为 Promise  
- fs/promises 等：	Node.js 官方已提供 Promise 版本的 API，  
```js
const fs = require('fs').promises;`

async function main() {
  try {
    const data = await fs.readFile('a.json', 'utf8');
    const result = await parseJson(data);
    const rows = await queryDb(result.id);
    // 清晰、可 try/catch、可复用
  } catch (err) {
    // 统一捕获所有异步错误
  }
}
```

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 

### 八、webSocket
webSocket与传统的http有什么优势 ？  
- 客户端与服务器只需要一个TCP连接，比http长轮询使用更少的连接
- webSocket服务端可以推送数据到客户端
- 更轻量的协议头，减少数据传输量

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 

## Node.js快问快答
1. **什么是Node.js？为什么需要它？**

Node.js是一个基于Chrome V8引擎的JavaScript运行环境，它允许开发者使用JavaScript编写服务器端应用。Node.js的出现解决了JavaScript只能在浏览器端运行的问题，使得JavaScript成为了全栈语言。它的异步I/O和非阻塞I/O模型使得它非常适合处理高并发场景。

2. **解释Node.js的事件循环(Event Loop)和工作原理。**

Node.js的事件循环是其核心特性之一，它允许Node.js处理非阻塞I/O操作。事件循环会不断检查事件队列，当有新的异步事件返回结果时，就会将其放入回调队列中，然后执行对应的回调函数。这种机制使得Node.js能够高效地处理大量并发请求。

3. **什么是回调地狱(Callback Hell)？如何避免？**

回调地狱是指在Node.js中过多嵌套回调函数导致代码难以阅读和维护的现象。为了避免回调地狱，我们可以使用Promises、Async/Await等异步编程解决方案，使代码更加清晰易读。

4. **描述一下Node.js中的Stream和它的用途。**

Stream是Node.js中处理流式数据的核心模块。它允许开发者以流的方式处理数据，从而提高数据的处理效率。Stream适用于处理大文件、网络请求等场景，可以分块读取和写入数据，减轻内存压力。

5. **Node.js中如何处理错误和异常？**

在Node.js中，我们可以通过try/catch语句来捕获和处理运行时异常。对于异步代码，我们可以使用Promise的.catch方法或async/await结构中的try/catch来捕获异常。此外，还可以使用Node.js提供的process.on(‘uncaughtException’)事件来监听未捕获的异常。

6. **Node.js中的模块系统是如何工作的？**

Node.js的模块系统基于CommonJS规范，允许开发者通过require()函数导入其他模块，并通过module.exports或exports对象导出模块功能。这种机制使得代码可以更加模块化，便于维护和复用。

7. **解释一下Node.js中的Buffer对象及其应用场景。**

Buffer是Node.js中用于处理二进制数据的核心模块。由于JavaScript本身不直接支持二进制数据操作，因此需要使用Buffer对象来处理网络数据、文件操作等场景中的二进制数据。

8. **如何优化Node.js应用的性能？**

优化Node.js应用的性能可以从多个方面入手，例如减少I/O操作、优化数据库查询、使用缓存、代码优化等。此外，还可以利用Node.js的集群(cluster)模块创建多个子进程，充分利用多核CPU资源，提高应用的并发处理能力。

9. **Node.js中的进程和线程有何区别？如何合理使用它们？**

在Node.js中，进程是资源分配的最小单位，而线程是CPU调度的最小单位。Node.js本身是单线程的，但通过libuv库实现了事件循环和异步I/O操作。对于计算密集型任务，可以使用Node.js的worker_threads模块创建多个线程来并行处理。合理使用进程和线程可以提高应用的性能和响应速度。

10. **请描述一下你在实际项目中如何使用Node.js解决过哪些问题。**

这个问题是考察你的实践经验。你可以结合实际项目经验，讲述你是如何使用Node.js解决并发处理、数据处理、网络通信等问题的。通过分享具体案例，可以让面试官更好地了解你的技术实力和解决问题的能力。