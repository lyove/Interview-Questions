# JavaScript

0、ES6新特性
* Let  const 
* 箭头函数
* 解构赋值
* 模板字符串
* 函数默认参数
* 展开运算符 / 剩余参数 ...
* 对象简写:{name, age}
* Promise 异步解决方案
* 类 class
* 模块化 import / export
* Set / Map：数据结构（去重、键值对）
* for...of：遍历数组 / 字符串
* Symbol：唯一值，避免属性冲突
* async/await

扩展解释
Set
* 有序、无重复值的列表
* 用于：数组去重、判断存在
* add、has、delete、size

Map
* 键值对，键可以是任意类型
* 比对象更强大
* set、get、has、delete、size

for...of
* 统一遍历语法
* 遍历数组、字符串、Set、Map，（不能遍历Object，因为普通对象不是可迭代的）
* 最简洁、最安全

Symbol
* 独一无二的值（永远不会重复）
* 用作对象唯一键
* 避免属性冲突、保护私有属性


1、什么是闭包，以及如何/为什么使用闭包？
函数 + 函数外部可访问的变量 = 闭包

为什么使用闭包？
1）数据私有化（模拟私有变量）
2） 函数柯里化 / 偏函数
3. 事件处理 / 回调中保留上下文
4. 防抖 / 节流

闭包的"坑"

1、经典陷阱：循环中的闭包
```js
// ❌ 错误：全部输出 3
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100)
}
// ✅ 修复 1：用 let（块级作用域，每次循环新建绑定）
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100)  // 0, 1, 2
}
// ✅ 修复 2：用 IIFE 制造独立作用域
for (var i = 0; i < 3; i++) {
  ;(function(j) {
    setTimeout(() => console.log(j), 100)
  })(i)
}
```
2、内存泄漏风险
解决：闭包中只引用必要变量，或及时释放引用

2.  for of 循环 
javaScript for...of 是 ES6 引入的循环语法，用于直接遍历可迭代对象（如数组、字符串、Map、Set 等）的元素值。
‌语法结构‌：for (const item of list) { // 代码块 }
不能遍历Object，因为普通对象不是可迭代的

3. 使异步函数按照顺序执行的方法有哪些？
 1 :  async/await 
 2：Promise.then()链
 3：用for...of循环
```js
    // 模拟一个异步任务（比如接口请求、文件读取）
    function delay(ms) {
        return new Promise(resolve => {
        setTimeout(() => {
            console.log(`等待 ${ms}ms 完成`);
            resolve();
        }, ms);
        });
    }
    
    // 异步函数（必须加 async）
    async function asyncLoop() {
        // 要遍历的数据
        const list = [1000, 2000, 3000];
        // ✅ for...of + await：按顺序执行异步
        for (const item of list) {
            console.log("开始执行：", item);
            await delay(item); // 等待当前异步完成，再走下一次循环
            console.log("执行完成：", item, "\n");
        }
        console.log("所有异步任务 按顺序 全部执行完毕！");
    }
    
    // 调用
    asyncLoop();
```
 4：reduce构建Promise链


----------------------

4. 伪数组:
一、什么是伪数组？
长得像数组，有length 属性，也可以通过下标 [index] 访问元素，但不能使用数组的原生方法（如 push、pop、forEach、map 等）。

伪数组的 3 个核心特征
1. 有 length 属性
2. 可以通过数字下标访问元素
3. 不是 Array 实例，但不能使用数组的原生方法（如 push、pop、forEach、map 等）
4. （补充）length 不会随元素增减自动变化

二、伪数组常见的例子：
1、函数的arguments对象，2、document.querySelectorAll()返回的NodeList对象，3、字符串 String（也是伪数组）
     1. arguments
```js
       function test() { 
          console.log(arguments); // 伪数组
       }
```
     2. DOM 集合
```js
       const divs = document.getElementsByTagName('div'); 
       console.log(divs); // HTMLCollection 伪数组
```
     3. 字符串 
```js
       const str = 'hello'; console.log(str[0]); // 'h'
```

三、判断方法：
```js
console.log(Array.isArray(fakeArr)); // false 
console.log(fakeArr instanceof Array); // false
```
// 手写一个伪数组：这就是一个标准伪数组 
```js
const fakeArray = { 
    0: '张三', 
    1: '李四', 
    2: '王五', 
    length: 3 // 必须有 
};
```

四、如何处理伪数组？
‌转换为真正的数组‌：使用Array.from()、扩展运算符(...)或者Array.prototype.slice.call()方法。
```js
const trueArray = Array. from(fakeArr);
const trueArray2 = [...fakeArr];
const trueArray3 = Array.prototype.slice.call(fakeArr);
```

五、使用循环
可以使用如for或forEach循环
```js
for (let i = 0; i < items.length; i++) {
    console.log(items[i]);
}
```

----------------------

5. 数组去重复：
1、const newer = […new Set(arr)]
2、indexOf /includes 遍历去重
```js
    function unique(arr) {
        let res = []
        for (let i = 0; i < arr.length; i++) {
            if (res.indexOf(arr[i]) === -1) {
                res.push(arr[i])
            }
	   // 使用res.includes
	    //if (!res.includes(arr[i])) res.push(arr[i])
        }
        return res
    }
```
3、对象数组去重
```js
	const uniqueById = arr.filter((item, index, self) => 
 		 index === self.findIndex(t => t.id === item.id)
	);
```

----------------------

6. http协议：
基本概念：HTTP 协议（Hypertext Transfer Protocol，超文本传输协议），应用层协议，是一种规定怎么发请求、怎么回响应、格式长什么样、用什么方法（GET/POST）、状态码怎么约定的协议。
请求包含：
* 请求行：方法 路径 协议版本
* 请求头（Headers）
* 空行
* 请求体（Body，可选）
请求方法：get/post/put更新 patch部分更新 delete
常见头部字段：
*  Content-Type：请求体或响应体的数据类型，如 application/json、text/html、multipart/form-data
*  Authorization：认证信息，如 Bearer <token>
*  Cookie / Set-Cookie：管理会话状态
*  User-Agent：客户端信息
*  Accept：客户端可接受的响应类型
*  Cache-Control：控制缓存行为
*  Origin / Referer：用于跨域和来源识别
响应包含：
* 响应行：HTTP/版本 状态码 描述 
* 响应头：Key: Value（如 Content-Type、Set-Cookie） 
* 空行 
* 响应体（返回的数据，如 HTML、JSON）
状态码：200成功、301重定向 302重定向 400错误 403无权限 404未找到  500服务器错误

HTTP：明文传输、端口 80、不安全（易被窃听 / 篡改）。
HTTPS：HTTP+TLS/SSL 加密、端口 443、安全（加密 + 认证 + 防篡改）。

HTTP/1.1：现在默认标配，不用你手动管；
HTTP/2：主流网站、CDN 基本都开了，前端不用改代码，后端 / 服务器配一下就行；
HTTP/3(QUIC)：基于 UDP，前端不用改业务代码，只需要服务器、CDN 开启支持。

----------------------


7、性能优化

简易方案：
*  减少HTTP请求数（合并资源、雪碧图、HTTP/2）
*  使用CDN加速静态资源
*  启用Gzip压缩
*  使用HTTP/2或HTTP/3提升并发效率

详细方案：

一、加载性能优化
1. 资源体积优化
* 代码压缩与混淆：使用 Terser、esbuild 压缩 JS，CSSNano 压缩 CSS
* Tree Shaking：消除未使用的代码（ESM + Webpack/Rollup/Vite）
* 代码分割（Code Splitting）：按路由、组件懒加载，使用 import() 动态导入
* Gzip/Brotli 压缩：服务端开启压缩，Brotli 比 Gzip 压缩率高 15-25%
* 图片优化：
    * 使用 WebP/AVIF 格式替代 JPEG/PNG
    * 响应式图片：srcset + sizes 属性
    * 懒加载：loading="lazy" 或 Intersection Observer
    * 使用 CDN 图片处理服务（如 Cloudinary、阿里云 OSS 图片处理）
2. 网络传输优化
* HTTP/2 或 HTTP/3：多路复用、头部压缩、服务器推送
* CDN 加速：静态资源分发到边缘节点
* 缓存策略：
    * 强缓存：Cache-Control: max-age=31536000
    * 协商缓存：ETag / Last-Modified
    * Service Worker 离线缓存
* 资源预加载：
    * <link rel="preload">：预加载关键资源
    * <link rel="prefetch">：预获取下一页资源
    * <link rel="preconnect">：提前建立 DNS/TCP/TLS 连接
    * <link rel="dns-prefetch">：提前 DNS 解析
3. 关键渲染路径优化
* 提取关键 CSS（Critical CSS）：将首屏所需 CSS 内联到 <head>，其余异步加载
* 异步加载非关键 JS：defer（DOM 解析后执行）或 async（下载完立即执行）
* 减少重定向：避免不必要的 301/302 跳转

二、运行时性能优化
1. JavaScript 执行优化
* 减少主线程阻塞：
    * 长任务拆分为多个小任务（requestIdleCallback、setTimeout）
    * Web Workers 处理复杂计算
    * WebAssembly 用于计算密集型任务
* 避免内存泄漏：及时移除事件监听、清理定时器、避免闭包引用大对象
* 虚拟列表：长列表使用 react-window、vue-virtual-scroller 等只渲染可视区域
* 防抖（Debounce）与节流（Throttle）：高频事件（滚动、输入、resize）
2. 渲染性能优化
* 减少重排（Reflow）与重绘（Repaint）：
    * 批量修改样式：使用 class 一次性修改，避免频繁操作单个样式属性
    * 离线操作 DOM：DocumentFragment、克隆节点修改后替换
    * 使用 transform 和 opacity 做动画（触发 GPU 合成层）
* CSS 优化：
    * 避免深层嵌套选择器
    * 使用 will-change 提示浏览器优化（动画结束后移除）
    * 避免使用 @import（阻塞渲染）
* 合成层优化：
    * 合理使用 transform: translateZ(0) 或 will-change 创建独立层
    * 避免过多合成层导致内存占用过高

----------------------

8. 深拷贝与浅拷贝

JS 数据分为两类，拷贝的区别就出在这里：

1. 基本类型：String/Number/Boolean/Null/Undefined/Symbol
    * 存在栈内存，赋值时直接复制值，不存在深浅拷贝
2. 引用类型：Object/Array/Function/Date 等
    * 栈存地址，堆存真实数据，拷贝时复制的是地址，这就有了深浅拷贝之分

  常用浅拷贝方法
1. 对象：Object.assign()、扩展运算符 {...obj}
2. 数组：Array.prototype.slice()、concat()、扩展运算符 [...arr]
   数组array.slice():array.slice(start, end)提取数组的一部分，返回一个新数组

  常用深拷贝方法
1. 简易版：JSON.parse(JSON.stringify(obj))，不支持：函数、Symbol、undefined、循环引用、Date（会转成字符串）
2. 完整版：手写递归深拷贝（支持函数、Symbol、循环引用）
3. 工具库：Lodash 的 _.cloneDeep(obj)
4.  ES2022中的structuredClone：const deepCopy = structuredClone(originalObj);不支持函数、Generator

----------------------

9. 跨域：
1. 什么是跨域
      浏览器前端AJAX/fetch 请求时候，协议、域名、端口只要有一个不同，就是跨域。
      同源策略目的：防止恶意网站窃取数据/防止CSRF(跨站请求伪造)攻击/保护用户隐私和安全。

2. 解决跨域：
      1）.CORS跨域解决方案（后端常用）。在服务器端设置响应头，告诉浏览器允许哪些来源的请求,
          Access-Control-Allow-Origin: 允许的域名 / * 
          Access-Control-Allow-Methods: GET,POST,PUT,DELETE 
          Access-Control-Allow-Headers: 自定义请求头
      2）. 代理服务器（Proxy）
      3）. JSONP（JSON with Padding）—— 仅限 GET:一种老旧的跨域解决方案，利用 <script> 标签不受跨域限制的特性
      4）. WebSocket：const socket = new WebSocket('ws://localhost:8080');
      5）. postMessage（页面 /iframe 通信）

----------------------

10. Cookies
1. 设置 Cookie
// 基础格式：document.cookie = "键=值; 过期时间; 路径;" 
document.cookie = "username=张三; max-age=86400; path=/"; 
// max-age=86400 = 1天（单位：秒）

2. 获取 Cookie (所有cookie)
// 获取到的是字符串，格式："username=张三; age=20"
```js
const allCookies = document.cookie; 
console.log(allCookies);
```

4. 修改 Cookie
直接覆盖同名cookie即可

4、删除 Cookies：
只能通过设置过期时间为过去来“删除”：把 max-age 设为 0 或负数

----------------------

11. localStorage、sessionStorage
* localStorage：持久化存储｜同源共享｜手动删才没；
* sessionStorage：会话级存储｜仅在同一个浏览器标签页内有效｜窗口或标签页关闭就清空；
    两个存储的用法完全相同，只有生命周期不同！
* storage.setItem('key', 'value');
* const value = storage.getItem('key');
* storage.removeItem('key');
* storage.clear();

----------------------

10. 防抖、节流 
防抖: 连续触发只执行最后一次；
场景：搜索框输入/窗口调整/按钮多次提交/滚动事件。
```js
function debounce(fn, delay) {
  // 用来保存定时器 ID
  let timer = null;
  // 返回一个新函数（闭包）
  return (...args) => {
    // 清除上一次定时器
    clearTimeout(timer);
    // 设置新定时器
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}
```

节流throttle(/ˈθrɑːtl/): 每隔一段时间执行一次,固定的时间间隔内,无论件触发多少次,函数最多只执行一次；
场景：滚动事件监听(如懒加载、下拉加载)/鼠标移动(mousemove)；
```js
function throttle(fn, delay) { 
    // 记录上一次执行的时间戳 
    let lastTime = 0; 
    // 返回一个新函数（闭包） 
    return function (...args) { 
        // 获取当前时间戳 
        const now = Date. now(); 
        // 判断：当前时间 - 上一次执行时间 >= 延迟时间 → 执行函数 
        if (now - lastTime >= delay) {
             lastTime = now; 
            // 更新上一次执行时间 
            fn.apply(this, args); 
            // 绑定this并传参 
        } 
    }; 
}
```
