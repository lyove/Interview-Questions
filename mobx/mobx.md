Mobx在React中的应用

## 一、MobX 核心概念（一句话版）
MobX 的哲学是"一切皆可观察"，核心概念：  
- State（状态）：可被观察的数据（observable）
- Actions（动作）：修改状态的函数（action）
- Computed（计算值）：基于 state 派生的自动缓存值（computed）
- Reactions（响应）：state 变化时自动触发的副作用（如更新 UI）
- observer：把 React 组件变成响应式，用到的 state 变了就自动重渲染

## 二、基础用法
1. **创建 Store（状态仓库）** 

使用 makeAutoObservable 自动识别所有属性、方法和计算属性：
```jsx
// src/stores/TodoStore.ts
import { makeAutoObservable } from 'mobx';

interface Todo {
  id: number;
  title: string;
  completed: boolean;
}

class TodoStore {
  todos: Todo[] = [];
  nextId = 1;

  constructor() {
    makeAutoObservable(this);
  }

  // Action：添加待办
  addTodo(title: string) {
    this.todos.push({
      id: this.nextId++,
      title,
      completed: false
    });
  }

  // Action：切换完成状态
  toggleTodo(id: number) {
    const todo = this.todos.find(t => t.id === id);
    if (todo) {
      todo.completed = !todo.completed;
    }
  }

  // Action：删除待办
  removeTodo(id: number) {
    this.todos = this.todos.filter(t => t.id !== id);
  }

  // Computed：计算已完成数量
  get completedCount() {
    return this.todos.filter(t => t.completed).length;
  }

  // Computed：计算未完成数量
  get remainingCount() {
    return this.todos.length - this.completedCount;
  }
}

// 导出单例
export const todoStore = new TodoStore();
```

2. **在组件中使用**  
使用 observer 包裹组件，使其成为响应式观察者

```jsx
// src/components/TodoList.tsx
import React from 'react';
import { observer } from 'mobx-react-lite';
import { todoStore } from '../stores/TodoStore';

const TodoList = observer(() => {
  return (
    <div>
      <h2>待办事项</h2>
      <p>已完成: {todoStore.completedCount} / 剩余: {todoStore.remainingCount}</p>
      
      <ul>
        {todoStore.todos.map(todo => (
          <li key={todo.id}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => todoStore.toggleTodo(todo.id)}
            />
            <span style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}>
              {todo.title}
            </span>
            <button onClick={() => todoStore.removeTodo(todo.id)}>删除</button>
          </li>
        ))}
      </ul>
    </div>
  );
});

export default TodoList;
```

```js
// src/components/AddTodo.tsx
import React, { useState } from 'react';
import { todoStore } from '../stores/TodoStore';

const AddTodo = () => {
  const [text, setText] = useState('');

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    if (text.trim()) {
      todoStore.addTodo(text.trim());
      setText('');
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={text}
        onChange={e => setText(e.target.value)}
        placeholder="输入待办事项..."
      />
      <button type="submit">添加</button>
    </form>
  );
};

export default AddTodo;
```

3. **根组件挂载**. 

```jsx
// src/App.tsx
import React from 'react';
import TodoList from './components/TodoList';
import AddTodo from './components/AddTodo';

function App() {
  return (
    <div style={{ maxWidth: 500, margin: '0 auto', padding: 20 }}>
      <h1>MobX Todo 示例</h1>
      <AddTodo />
      <TodoList />
    </div>
  );
}

export default App;
```

## 三、进阶技巧

1. 模块化 Store 设计
大型应用建议拆分多个 Store，通过 RootStore 整合：  
```jsx
// src/stores/RootStore.ts
import { TodoStore } from './TodoStore';
import { UserStore } from './UserStore';

class RootStore {
  todoStore: TodoStore;
  userStore: UserStore;

  constructor() {
    this.todoStore = new TodoStore();
    this.userStore = new UserStore();
  }
}

export const rootStore = new RootStore();
```
使用 Context 传递： 
```jsx
// src/context/StoreContext.tsx
import React, { createContext, useContext } from 'react';
import { rootStore } from '../stores/RootStore';

const StoreContext = createContext(rootStore);

export const StoreProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  return (
    <StoreContext.Provider value={rootStore}>
      {children}
    </StoreContext.Provider>
  );
};

export const useStore = () => useContext(StoreContext);
```

2. **局部状态管理**
对于不需要全局共享的状态，使用 `useLocalObservable`  
```js
import { useLocalObservable, observer } from 'mobx-react-lite';

const Counter = observer(() => {
  const store = useLocalObservable(() => ({
    count: 0,
    increment() {
      this.count++;
    },
    decrement() {
      this.count--;
    }
  }));

  return (
    <div>
      <p>Count: {store.count}</p>
      <button onClick={() => store.decrement()}>-</button>
      <button onClick={() => store.increment()}>+</button>
    </div>
  );
});
```

3. **异步 Action 处理**

```jsx
class UserStore {
  users: User[] = [];
  loading = false;
  error: string | null = null;

  constructor() {
    makeAutoObservable(this);
  }

  async fetchUsers() {
    this.loading = true;
    this.error = null;
    try {
      const response = await fetch('/api/users');
      const data = await response.json();
      this.users = data;
    } catch (err) {
      this.error = '加载失败';
    } finally {
      this.loading = false;
    }
  }
}
```

## 四、关键要点总结
- 自动追踪：observer 组件会自动追踪其渲染中使用的可观察状态，实现精准更新
- 直接修改：在 action 中可以直接修改状态（如 this.count++），无需不可变更新
- 计算属性：computed 值会自动缓存，只有依赖变化时才重新计算
- 严格模式：生产环境建议开启 configure({ enforceActions: 'always' }) 强制使用 action 修改状态