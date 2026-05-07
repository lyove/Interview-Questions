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

## 四、Node.js Stream
Stream（流）是一种用于高效处理流式数据的抽象接口。它的核心思想是将数据分成小块（chunk）进行连续处理，而不是一次性将整个数据加载到内存中。
**内存效率高**：Stream 通过分块处理，能始终保持很低的内存占用；  
**时间效率高**：数据可以边接收边处理，无需等待所有数据准备就绪。这对于网络请求、实时数据处理等高并发场景至关重要，能够显著降低延迟。

### Stream 的四种类型
| 类型 | 描述 | 典型示例 |
| :--- | :--- | :--- |
| Readable | 用于读取数据 | `fs.createReadStream()` (读取文件) |
| Writable | 用于写入数据 | `fs.createWriteStream()` (写入文件) |
| Duplex | 双向流，既可读又可写 | `net.Socket` (TCP 连接) |
| Transform | 转换流，在读写过程中可以修改或转换数据 | `zlib.createGzip()` (数据压缩) |

### 如何使用 Stream
Stream 的使用主要依赖于事件和管道（pipe）机制。  
1. 事件驱动
  Stream 是事件发射器（EventEmitter），通过监听事件来响应数据的变化。  
  Readable 流：主要监听 data 事件（数据块到达时触发）和 end 事件（数据读取完毕时触发）。  
  Writable 流：主要使用 write() 方法写入数据，并监听 finish 事件（所有数据写入完成）。  
2. 管道操作 (.pipe())
.pipe() 方法是 Stream 的核心，它能将一个可读流的输出自动连接到另一个可写流的输入，形成一个高效的数据处理链。更重要的是，pipe 内置了背压（Backpressure）机制，当下游处理不过来时，会自动通知上游暂停发送数据，从而防止内存溢出。

### 实际应用场景
Stream 在处理大量数据的场景中表现尤为出色：
1. 处理大文件：无论是读取一个巨大的 CSV 文件进行数据分析，还是将用户上传的视频文件保存到磁盘，使用 Stream 都可以避免内存爆炸。  
```javascript
const fs = require('fs');
// 将大文件 input.txt 的内容复制到 output.txt
fs.createReadStream('input.txt')
  .pipe(fs.createWriteStream('output.txt'));
```
2. 网络请求与响应：在 Web 开发中，可以直接将 HTTP 请求的数据流式地写入文件，或者将一个文件流式地发送给客户端。  
```javascript
const https = require('https');
const fs = require('fs');
// 从网络下载文件并保存
const file = fs.createWriteStream('data.txt');
https.get('https://example.com/large-file.zip', (response) => {
  response.pipe(file);
});
```

3. 数据压缩与解压：利用 Transform 流，可以在数据传输过程中实时进行压缩或解压。 
```javascript
const fs = require('fs');
const zlib = require('zlib');
// 将文件压缩成 .gz 格式
fs.createReadStream('input.txt')
  .pipe(zlib.createGzip())
  .pipe(fs.createWriteStream('input.txt.gz'));
```

4. 实时数据处理：处理来自传感器、金融市场或物联网设备的连续数据流时，Stream 可以实现高效的实时过滤、聚合和分析。


- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 

## 五、如何在 Node.js 中创建一个返回 Hello World 的简单服务器？
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

## 六、Node.js 如何处理CORS 跨域？

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

## 七、Node.js 中间件是什么？有哪些用途？

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

## 八、Node.js的异步问题

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

### 九、webSocket
**核心特点：**. 
- 持久连接，避免 HTTP 轮询开销
- 双向实时通信
- 基于帧传输，头部开销小
- 使用 ws://（非加密）或 wss://（加密，推荐生产环境使用）

**webSocket与传统的http有什么优势 ？**   
- 客户端与服务器只需要一个TCP连接，比http长轮询使用更少的连接
- webSocket服务端可以推送数据到客户端
- 更轻量的协议头，减少数据传输量

前端 WebSocket 需要服务端支持，以下是常见服务端配置参考：

Nginx 反向代理配置
```
location /ws {
    proxy_pass http://backend_server;
    proxy_http_version 1.1;
    
    # 关键：升级协议
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    
    # 超时设置
    proxy_read_timeout 86400;  # 24小时，保持长连接
}
```

Node.js (ws 库) 简单示例 
```js
const WebSocket = require('ws');
const wss = new WebSocket.Server({ port: 8080 });

wss.on('connection', (ws) => {
  ws.on('message', (message) => {
    // 广播给所有客户端
    wss.clients.forEach(client => {
      if (client.readyState === WebSocket.OPEN) {
        client.send(message);
      }
    });
  });
});
```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 

## 十、优化Node.js应用
### 代码层面优化
1. **坚持使用异步编程**
  Node.js 的核心是单线程事件循环，任何耗时的同步操作（如 fs.readFileSync）都会阻塞整个事件循环，严重影响性能。务必使用异步 API，并优先采用 Promise 和 async/await 来管理异步流程，避免“回调地狱”，提升代码可读性。
2. **高效处理 I/O 操作**
  - **并行执行**：对于相互独立的异步 I/O 操作（如读取多个文件、查询多个数据库），应使用 Promise.all() 来并行执行，而不是串行等待，可以显著减少总等待时间。 
  - **使用流（Stream）**：处理大文件或大数据时，使用流（Stream）和 pipe() 方法可以逐块处理数据，避免将整个文件加载到内存中，从而大幅降低内存占用。
3. **实施缓存策略**
  缓存是提升响应速度最有效的手段之一。对于频繁访问但变动不频繁的数据（如用户信息、配置、热点文章），应使用缓存。
  - **内存缓存**：适用于单实例应用，可使用 node-cache 等库进行短期缓存。  
  - **外部缓存**：对于多实例部署或需要持久化的缓存，推荐使用 Redis 或 Memcached。
4. **优化数据库交互**
  - **使用连接池**：避免为每个请求都创建和销毁数据库连接。使用连接池（如 mysql2/promise 的 createPool）可以复用连接，减少开销。  
  - **查询优化**：为常用查询字段建立索引，并使用 SELECT 语句只获取需要的字段，而非 SELECT *。

### 运行时层面优化
这一层面主要关注如何充分利用服务器资源和 Node.js 运行时的特性。
1. **利用多核 CPU (集群模式)** 
  Node.js 默认是单线程的，无法利用多核 CPU 的优势。使用内置的 cluster 模块或更强大的进程管理工具 PM2，可以轻松启动多个工作进程（Worker Processes）来分担负载，使应用能够充分利用服务器的所有 CPU 核心。
```bash
# 使用 PM2 启动集群，自动利用所有 CPU 核心
pm2 start app.js -i max
```

2. **处理 CPU 密集型任务**
事件循环不适合处理复杂的计算等 CPU 密集型任务，它们会阻塞其他请求。  
  - 使用 worker_threads：对于需要共享内存的高频 CPU 任务，可以使用 worker_threads 模块将任务放到独立的线程中执行，避免阻塞主线程。  
  - 使用 child_process：对于独立的、开销较大的任务，可以启动子进程来处理。

3. **优化内存管理**
  - 避免内存泄漏：谨慎使用全局变量和闭包，它们可能导致对象无法被垃圾回收（GC）。可以使用 WeakMap 来存储大对象的缓存。
  - 调整内存上限：默认情况下，V8 引擎对老生代内存有限制。在内存需求大的应用中，可以通过 --max-old-space-size 参数来增加内存上限，例如 node --max-old-space-size=4096 app.js。

### 架构层面优化
从宏观架构上进行调整，可以带来数量级的性能提升。  
1. **启用 Gzip 压缩**
  在 Express 或 Koa 等框架中，使用 compression 中间件可以轻松启用 Gzip 压缩。这通常能将响应体的体积减少 60-80%，显著提升页面加载速度并降低带宽成本。
2. **水平扩展**
  当单台服务器的性能达到瓶颈时，与其不断升级硬件（纵向扩展），不如增加更多的服务器实例（横向扩展）。结合 Docker 容器化和 Kubernetes 等编排工具，可以实现应用的弹性伸缩，轻松应对高并发流量。
3. **使用负载均衡**
  在多个应用实例前部署一个反向代理（如 Nginx），可以将传入的 API 请求均匀地分发到不同的实例上，确保系统整体的稳定性和高可用性。
4. **分离静态资源服务**
  不要让 Node.js 服务器处理静态文件（如图片、CSS、JS 文件）。将这些资源托管到 Nginx 或 CDN（内容分发网络）上，可以极大地减轻 Node.js 应用的压力，让它专注于处理动态业务逻辑。

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 

## Node.js快问快答
1. **什么是Node.js？为什么需要它？**  
Node.js是一个基于Chrome V8引擎的JavaScript运行环境，它允许开发者使用JavaScript编写服务器端应用。Node.js的出现解决了JavaScript只能在浏览器端运行的问题，使得JavaScript成为了全栈语言。它的异步I/O和非阻塞I/O模型使得它非常适合处理高并发场景。

2. **解释Node.js的事件循环(Event Loop)和工作原理。**  
Node.js的事件循环是其核心特性之一，它允许Node.js处理非阻塞I/O操作。事件循环会不断检查事件队列，当有新的异步事件返回结果时，就会将其放入回调队列中，然后执行对应的回调函数。这种机制使得Node.js能够高效地处理大量并发请求。

3. **什么是回调地狱(Callback Hell)？如何避免？**  
回调地狱是指在Node.js中过多嵌套回调函数导致代码难以阅读和维护的现象。为了避免回调地狱，我们可以使用Promises、Async/Await等异步编程解决方案，使代码更加清晰易读。

4. **描述一下Node.js中的Stream和它的用途。**. 
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