一、Node.js 工作机制
核心特点
* 单线程：Node.js 使用单线程处理请求
* 事件循环：通过事件驱动机制处理并发
* 非阻塞 I/O：I/O 操作不会阻塞主线程
* 跨平台：可以在 Windows、Linux、macOS 等系统上运行

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 

二、如何在 Node.js 中创建一个返回 Hello World 的简单服务器？

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

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 

三、Node.js 如何处理CORS 跨域？
1、原生 Node.js（不依赖框架）

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

2、使用 cors 包（推荐）

const http = require('http');
const cors = require('cors')();

const server = http.createServer((req, res) => {
  cors(req, res, () => {
    res.writeHead(200);
    res.end('ok');
  });
});


3、Egg.js 框架

Egg.js 没有内置 CORS 插件，需要安装官方插件：
1）. 安装插件
npm install egg-cors --save

2）. 启用插件
// config/plugin.js
exports.cors = {
  enable: true,
  package: 'egg-cors',
};

3）. 配置跨域规则
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

4）. 环境差异化配、
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


- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 

四、Node.js 中间件是什么？有哪些用途？

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

五、Node.js的异步问题

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
