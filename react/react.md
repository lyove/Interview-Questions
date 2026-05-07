前端-react

1. React 的核心特性有哪些？
* 声明式编程：关注“做什么”而非“怎么做”，只需描述UI状态，React 自动处理DOM更新；
* 组件化(基于组件)：UI拆分为独立可复用的组件，降低耦合、提升维护性；
* 单向数据流：数据从父组件流向子组件，子组件不直接修改父组件数据，保证数据可追溯；
* 虚拟DOM（VDOM）：用JS对象描述DOM结构，通过diff算法最小化DOM操作；
* JSX：语法糖（React.createElement的简写），允许在JS中写HTML结构，提升开发效率。
* Hooks：它让你无需编写类组件，就能在函数组件中使用状态（useState）和其他 React 特性。这使得代码逻辑更易于组织和复用。


----------------------

2. 简述 React 的单向数据流及优势
单向数据流： 数据只能从父组件通过props传递给子组件，子组件若要修改数据，需调用父组件传递的回调函数，由父组件更新数据后再回传；

优势：
* 数据流向清晰，便于调试（可快速定位数据修改位置）；
* 避免子组件随意修改数据导致的状态混乱；
* 组件解耦，父组件控制数据，子组件仅负责展示/交互。

----------------------

3、props 和 state 的区别？
props	                                     state
**来源**	
父组件传递	                                 组件内部定义

**可变性**	                                                              
只读（子组件不能修改）	                       可变（通过setState/useState更新）

**触发更新**	
父组件修改props，子组件重新渲染	                组件内部更新state，自身重新渲染
**用途**	
组件间通信	                                 组件内部状态管理

----------------------

4. 组件key属性的作用与原理,为什么不能用index做key
 key作用：
       1、提高渲染性能: 比较新旧dom精确到每个元素,复用已存在dom而不是销毁重建,所以提高性能
       2、维持状态: 组件内部状态和key是关联的，

 如何正确使用 key
* 优先使用数据自带的唯一 ID
* 避免使用数组索引 index 作为 key
* 确保 key 在同级中唯一且稳定: key 只需要在同一个父元素下的兄弟节点中保持唯一即可，无需全局唯一


----------------------

5. setSate后发生了哪些事情：
   1.触发更新请求（不会立即更新状态，先加入更新队列）
   2.批量更新（多个 setState 调用合并为一次更新）
   3.渲染（render）阶段
   4.提交更新 触发生命周期

详细流程如下：
setState(newState)
    ↓
更新入队 (Update Queue)
    ↓
调度器安排渲染 (Scheduler)
    ↓
批处理多个更新 (Batching)
    ↓
组件函数重新执行 (Re-render)
    ↓
生成新虚拟 DOM
    ↓
与旧虚拟 DOM 比较 (Diffing)
    ↓
更新真实 DOM (Commit)
    ↓
执行 useEffect 等副作用

----------------------

6、React 生命周期
现代函数组件的生命周期 (Hooks 方式)
* 挂载阶段 (Mounting) - 组件诞生
* 更新阶段 (Updating) - 组件成长
* 卸载阶段 (Unmounting) - 组件销毁
* 特殊生命周期：useLayoutEffect：它在所有 DOM 变更之后、浏览器绘制之前同步执行

传统类Class组件的生命周期
挂载阶段
  constructor()
  getDerivedStateFromProps()
  render()
  componentDidMount()

更新阶段
  getDerivedStateFromProps(): 在调用 render 方法之前调用，并且在初始挂载及后续更新时都会被调用。根据shouldComponentUpdate() 的返回值，判断是否更新。
  shouldComponentUpdate():当 props 或 state 发生变化时，会在渲染执行之前被调用。
  render(): render() 方法是 class 组件中唯一必须实现的方法。
  getSnapshotBeforeUpdate(): 在最近一次渲染输出（提交到 DOM 节点）之前调用。
  componentDidUpdate(): 在更新后会被立即调用。

卸载阶段
componentWillUnmount()

----------------------

7、react hooks 父子组件通信:
* props
* 回调函数
* Context
* forwardRef(组件,ref)&useImperativeHandle(ref,()=>({})


---------------------

8. HOC 
是使用接受组件作为参数并返回新组件的函数创建的。

----------------------

9. react闭包：
闭包陷阱是指：在 Hooks（如 useEffect、useCallback）的回调函数中，访问到的状态（state）或属性（props）是“过时”的旧值，而不是最新的值。
常见场景：
1、useEffect + setInterval/setTimeout：定时器回调捕获旧值/过时状态
2、useCallback 依赖遗漏：缓存的函数使用了旧状态
3、异步回调（fetch、Promise）：请求响应时状态已更新

解决方案：
1、正确使用依赖数组（最基础）
2、使用函数式更新（推荐 ✅）：setCount(prev => prev + 1);因为setState(prev => prev + 1) 不依赖外部闭包中的 count，React 会自动传入最新的状态值。
3、使用 useRef 保存最新值
4、使用 useReducer（复杂状态逻辑）
5、自定义 Hook 封装



----------------------

10. react优化：
（1）渲染优化
    避免不必要的渲染：
    用React.memo缓存纯组件；
    用useCallback缓存传递给子组件的函数；
    用useMemo缓存复杂计算结果；
    拆分组件：将大组件拆分为小型纯组件，减少渲染范围；
    懒加载组件：React.lazy + Suspense 按需加载组件。
（2）数据优化
    列表虚拟滚动：用react-window/react-virtualized渲染长列表，只渲染可视区域；
    防抖节流：搜索框输入防抖、滚动/resize事件节流；
    缓存请求数据：用SWR/React Query缓存接口数据，避免重复请求。
（3）打包优化
    代码分割：React.lazy动态导入组件，实现代码分割（将组件代码拆分为单独的 chunk 文件），减少首屏加载体积；
    Tree Shaking：移除未使用的代码（需ES模块）；
    CDN引入第三方库：如React、ReactDOM通过CDN引入，减少打包体积。
（4）运行时优化
    减少DOM操作：用React状态驱动视图，避免手动操作DOM；
    清理副作用：useEffect中及时清理定时器/事件监听，避免内存泄漏；
    避免在渲染中创建函数/对象：如<Child onClick={() => {}} /> 会导致子组件每次重新渲染。


----------------------

11、自定义Hooks防抖（debounce）/节流（throttle）
防抖：
import { useState, useEffect } from 'react';
const useDebounce = (value, delay) => {
  const [debouncedValue, setDebouncedValue] = useState(value);
  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);
    return () => {
      clearTimeout(timer);
    };
  }, [value, delay]);
  return debouncedValue;
}
export default useDebounce;

import { useCallback, useRef } from 'react';
function useDebounceFn(fn, delay) {
  const timerRef = useRef(null);
  const debouncedFn = useCallback((...args) => {
    if (timerRef.current) {
      clearTimeout(timerRef.current);
    }
    timerRef.current = setTimeout(() => {
      fn(...args);
    }, delay);
  }, [fn, delay]);
  return debouncedFn;
}
export default useDebounceFn;

节流
import { useState, useEffect, useRef } from 'react';
function useThrottle(value, limit) {
  const [throttledValue, setThrottledValue] = useState(value);
  const lastRun = useRef(Date.now());
  useEffect(() => {
    const handler = setTimeout(() => {
      const now = Date.now();
      if (now - lastRun.current >= limit) {
        setThrottledValue(value);
        lastRun.current = now;
      }
    }, limit - (Date.now() - lastRun.current)); // 计算剩余等待时间
    return () => {
      clearTimeout(handler);
    };
  }, [value, limit]);
  return throttledValue;
}
export default useThrottle;

import { useRef, useCallback } from 'react';
function useThrottleFn(fn, limit) {
  const lastRun = useRef(0);
  const timeoutRef = useRef(null);

  const throttledFn = useCallback((...args) => {
    const now = Date.now();
    if (now - lastRun.current >= limit) {
      fn(...args);
      lastRun.current = now;
    } 
    else {
      if (timeoutRef.current) clearTimeout(timeoutRef.current);
      timeoutRef.current = setTimeout(() => {
        fn(...args);
        lastRun.current = Date.now();
      }, limit - (now - lastRun.current));
    }
  }, [fn, limit]);
  return throttledFn;
}
export default useThrottleFn;

----------------------

12、为什么 useState 返回的是数组而不是对象?
核心原因：解构赋值的灵活性，数组解构允许你自由地为状态变量和更新函数命名，而对象解构则必须使用固定的属性名，或者在每次使用时进行重命名

----------------------


13、如何让 useEffect 支持async/await?
在 useEffect 内部定义并调用异步函数
useEffect(() => {
  // 定义异步函数
  const fetchData = async () => {
    try {
      const response = await fetch('/api/data');
    } catch (error) {}
  };
  // 执行
  fetchData();
}, []); 

----------------------

14、说说对受控组件和非受控组件的理解，以及应用场景？

----------------------


15、useRef/ref/forwardsRef 的区别是什么?

useRef 和 ref 都是 React 中用于操作 DOM 元素或自定义组件实例的工具，而 forwardRef 则是用于访问嵌套子组件中的 DOM 元素或自定义组件实例。 它们之间的区别如下：
* useRef 是一个 hook 函数，可以在函数组件中使用；ref 是一个对象属性，只能在类组件中使用。
* useRef 返回一个可变的 ref 对象，可以在组件的整个生命周期内保持不变，也就是说不会因为重新渲染而改变。而 ref 每次渲染都会被重新创建。
* useRef 主要用于存储和更新组件内部状态，以及操作 DOM 元素。而 ref 主要用于获取 DOM 元素或自定义组件实例。
* forwardRef 是用于将 ref 属性“向下传递”给一个函数式子组件或自定义组件的工具函数。它允许父组件调用子组件中的 DOM 元素或自定义组件实例。

综上所述，useRef 和 ref 都是用于操作 DOM 元素或自定义组件实例的工具，与之相比，forwardRef 则是一个更高级的工具，用于处理专门的情况，即访问嵌套子组件中的 DOM 元素或自定义组件实例。

----------------------

16、 React Hooks 出现的背景？解决了什么问题？
背景：类组件存在的痛点——
* 逻辑复用困难（HOC/Render Props 导致嵌套地狱）；
* 复杂组件逻辑分散（生命周期方法中混杂不同逻辑，如componentDidMount同时请求数据+初始化定时器）；
* this指向问题（类组件方法需绑定this，易出错）；

解决的问题：
* 用Hooks将分散的逻辑聚合，按功能组织代码；
* 函数组件支持状态管理和生命周期逻辑；
* 逻辑复用更简洁（自定义Hooks），无嵌套地狱；
* 无需处理this指向问题。

----------------------

17、自定义 Hooks 的定义及使用场景？
* 定义：以use开头的自定义函数，封装可复用的逻辑，内部可调用内置Hooks；
* 核心规则：只能在函数组件/自定义Hooks中调用，不能在普通函数/条件语句中调用；
* 典型场景：请求数据、表单校验、防抖节流、监听窗口大小等。

----------------------

18、React.memo、useMemo、useCallback 的区别及使用场景？
React.memo	缓存组件，仅props变化时重新渲染	纯展示型子组件，props不频繁变化
useMemo	缓存计算结果，避免重复计算	复杂计算（如列表过滤排序）
useCallback	缓存函数引用，避免函数重新创建	传递给子组件的回调函数（配合React.memo）
