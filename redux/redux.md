## Redux
Redux 是 React 生态中用于集中管理应用状态的工具库。它通过将状态从组件中提取出来，存放在一个全局的“仓库”（Store）中，解决了复杂应用中组件间状态共享和通信的难题。
为了简化开发，官方推荐使用 Redux Toolkit (RTK)，它封装了 Redux 的核心逻辑，大大减少了样板代码。
### 🚀 快速开始
1. 安装依赖  
在你的 React 项目中，安装 Redux Toolkit 和用于连接 React 的 react-redux 库。
```bash
npm install @reduxjs/toolkit react-redux
```
2. 创建 Store 和 Slice.  
Redux Toolkit 的核心是 createSlice，它能根据你定义的状态和修改方法，自动生成 action 创建器和 reducer。 
```jsx
// src/features/counter/counterSlice.js
import { createSlice } from '@reduxjs/toolkit';

// 1. 定义初始状态
const initialState = {
  value: 0,
};

// 2. 创建一个 slice
export const counterSlice = createSlice({
  name: 'counter', // slice 的名称
  initialState,
  reducers: {
    // 定义修改状态的方法，这里的 state 可以被直接“修改”，RTK 内部会使用 immer 库保证不可变性
    increment: (state) => {
      state.value += 1;
    },
    decrement: (state) => {
      state.value -= 1;
    },
    incrementByAmount: (state, action) => {
      state.value += action.payload;
    },
  },
});

// 3. 导出 actions，供组件 dispatch 使用
export const { increment, decrement, incrementByAmount } = counterSlice.actions;

// 4. 导出 reducer，供 store 配置使用
export default counterSlice.reducer;
```

3. 配置 Store
使用 configureStore 来创建应用的 store，并将你的 slice reducer 传入。
```jsx
// src/app/store.js
import { configureStore } from '@reduxjs/toolkit';
import counterReducer from '../features/counter/counterSlice';

export const store = configureStore({
  reducer: {
    // 将 counter reducer 挂载到 state.counter 上
    counter: counterReducer,
  },
});
```

4. 在 React 中注入 Store  
使用 `react-redux` 提供的 `<Provider>` 组件将 store 注入到你的 React 应用中，使其对所有子组件可用。
```jsx
// src/index.js
import React from 'react';
import ReactDOM from 'react-dom/client';
import { Provider } from 'react-redux'; // 导入 Provider
import { store } from './app/store'; // 导入配置好的 store
import App from './App';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <Provider store={store}>
    <App />
  </Provider>
);
```

5. 在组件中使用  
在组件中，使用 `useSelector` Hook 来读取状态，使用 `useDispatch` Hook 来派发 action 以更新状态。
```jsx
// src/App.js
import React from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { increment, decrement, incrementByAmount } from './features/counter/counterSlice';

function App() {
  // 1. 使用 useSelector 从 store 中读取状态
  const count = useSelector((state) => state.counter.value);
  // 2. 使用 useDispatch 获取 dispatch 函数
  const dispatch = useDispatch();

  return (
    <div>
      <h1>Count: {count}</h1>
      {/* 3. 通过 dispatch action 来更新状态 */}
      <button onClick={() => dispatch(increment())}>
        +
      </button>
      <button onClick={() => dispatch(decrement())}>
        -
      </button>
      <button onClick={() => dispatch(incrementByAmount(5))}>
        +5
      </button>
    </div>
  );
}

export default App;
```
## 📚 核心概念
理解 Redux 的关键在于掌握其单向数据流，它由三个核心部分组成： 

1. **Store (仓库)**
应用的全局状态容器，整个应用只有一个 store。
通过 configureStore 创建，它保存了应用的状态树（State Tree）。

2. **Action (动作)**
一个描述“发生了什么”的普通 JavaScript 对象。
必须包含一个 type 字段来标识 action 的类型，例如 { type: 'counter/increment' }。
可以携带额外的数据 payload，例如 { type: 'counter/incrementByAmount', payload: 5 }。

3. **Reducer (化简器)**
一个纯函数，它根据当前的 state 和接收到的 action，来决定如何更新状态，并返回一个新的 state。
它不能直接修改旧的 state，必须返回一个新对象。Redux Toolkit 通过 immer 库让你可以写出看似直接修改的代码，但底层仍保证了不可变性。

### 数据流向：
View (组件) → dispatch(action) → Reducer → Store → View (组件更新)

## 💡 进阶实践
**处理异步逻辑**
Redux 本身是同步的。处理异步操作（如 API 请求）通常使用 redux-thunk 中间件，而 Redux Toolkit 的 createAsyncThunk API 让异步处理变得非常简单。
```jsx
// src/features/users/userSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import axios from 'axios';

// 1. 创建一个异步 thunk
export const fetchUsers = createAsyncThunk(
  'users/fetchUsers',
  async () => {
    const response = await axios.get('https://api.example.com/users');
    return response.data; // 成功时返回的数据会成为 action.payload
  }
);

const initialState = {
  users: [],
  loading: false,
  error: null,
};

export const userSlice = createSlice({
  name: 'users',
  initialState,
  reducers: {},
  // 2. 处理异步 action 的不同状态
  extraReducers: (builder) => {
    builder
      .addCase(fetchUsers.pending, (state) => {
        state.loading = true;
      })
      .addCase(fetchUsers.fulfilled, (state, action) => {
        state.loading = false;
        state.users = action.payload;
      })
      .addCase(fetchUsers.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message;
      });
  },
});

export default userSlice.reducer;
```

### 性能优化
使用 React.memo：避免组件因父组件重渲染而进行不必要的渲染。
使用 reselect：创建“记忆化选择器”（memoized selectors）。当组件需要从 store 中派生或计算复杂数据时，reselect 可以避免重复计算，提升性能。

```jsx
// 使用 createSelector 创建记忆化选择器
import { createSelector } from '@reduxjs/toolkit';

const selectTodos = state => state.todos;
const selectFilter = state => state.filter;

export const selectVisibleTodos = createSelector(
  [selectTodos, selectFilter],
  (todos, filter) => {
    // 这个函数只有在 todos 或 filter 变化时才会重新执行
    return todos.filter(todo =>
      filter === 'ALL' ||
      (filter === 'COMPLETED' && todo.completed) ||
      (filter === 'ACTIVE' && !todo.completed)
    );
  }
);
```

### 调试
安装 Redux DevTools 浏览器扩展，它可以让你清晰地查看每一次 state 的变化、dispatch 的 action 以及 action 触发前后的 state 快照，是开发过程中不可或缺的调试工具。