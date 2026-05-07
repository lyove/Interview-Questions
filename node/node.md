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
1）. 安装插件 
`npm install egg-cors --save`

2）. 启用插件
```js
// config/plugin.js
exports.cors = {
  enable: true,
  package: 'egg-cors',
};
```

3）. 配置跨域规则
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

4）. 环境差异化配、
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
一句话：中间件 = 可复用的请求/响应拦截器

用途：
日志记录：记录每个请求的 URL、耗时
身份验证：检查用户是否登录、Token 是否有效
数据解析：解析 JSON 请求体、表单数据
错误处理：统一捕获和处理异常
CORS 跨域：设置跨域响应头
静态文件服务


- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 

## 七、Node.js的异步问题

1、 回调地狱（Callback Hell）
嵌套层级深、代码横向膨胀，逻辑难以跟踪：

2、错误处理繁琐
每个回调都要手动检查 err，容易遗漏，且没有统一的异常捕获机制（try/catch 对异步回调无效）

3. 控制流复杂
并行执行、串行执行、竞态条件等需要借助 async 等库，原生实现很笨拙。

4. 代码复用困难
回调的签名不统一，中间结果传递靠闭包，模块化程度低。

解决方案
方案	说明
Promise	ES6 标准，链式调用，统一错误处理（.catch()）
async/await	ES2017，基于 Promise 的语法糖，写同步风格的异步代码
util.promisify	Node.js 内置，将 callback API 包装为 Promise
fs/promises 等	Node.js 官方已提供 Promise 版本的 API
const fs = require('fs').promises;

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

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
