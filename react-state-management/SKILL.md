---
name: "react-state-management"
description: "React State Management patterns. Invoke when managing state in React components - useState, useReducer, Context, and state best practices."
---

# React State Management 最佳实践

基于 React 官方文档的状态管理指南。

## State 原则

### 1. State 是只读的

不要直接修改 state，而是使用 setState：

```javascript
// ❌ 错误
function Counter() {
  const [count, setCount] = useState(0);
  function handleClick() {
    count++; // 直接修改！
    setCount(count);
  }
}

// ✅ 正确
function Counter() {
  const [count, setCount] = useState(0);
  function handleClick() {
    setCount(count + 1);
  }
}
```

### 2. 使用函数式更新

当新值依赖旧值时，使用函数：

```javascript
function Counter() {
  const [count, setCount] = useState(0);
  
  function handleClick() {
    setCount(prevCount => prevCount + 1);
  }
  
  function decrement() {
    setCount(prevCount => prevCount - 1);
  }
}
```

## State 位置选择

### 1. 提升 State

多个组件需要共享 state 时，提取到最近公共祖先：

```javascript
// ❌ 各自管理
function Parent() {
  return (
    <>
      <Child1 count={5} />
      <Child2 count={5} />
    </>
  );
}

// ✅ 提升到 Parent
function Parent() {
  const [count, setCount] = useState(5);
  return (
    <>
      <Child1 count={count} onSetCount={setCount} />
      <Child2 count={count} onSetCount={setCount} />
    </>
  );
}
```

### 2. 避免冗余 State

不要复制 props 到 state：

```javascript
// ❌ 冗余
function Child({ name }) {
  const [childName, setChildName] = useState(name);
}

// ✅ 直接使用
function Child({ name }) {
  // 直接使用 props
  return <h1>{name}</h1>;
}
```

### 3. 避免派生 State

计算得出的值不需要 state：

```javascript
// ❌ 派生 state
function OrderSummary() {
  const [items, setItems] = useState([...]);
  const [total, setTotal] = useState(0); // 不需要！
  
  useEffect(() => {
    setTotal(items.reduce((sum, item) => sum + item.price, 0));
  }, [items]);
}

// ✅ 直接计算
function OrderSummary() {
  const [items, setItems] = useState([...]);
  const total = items.reduce((sum, item) => sum + item.price, 0);
  
  return <div>Total: {total}</div>;
}
```

## useReducer 模式

当 state 逻辑复杂时使用：

```javascript
function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    case 'reset':
      return { count: 0 };
    default:
      return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0 });
  
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
    </>
  );
}
```

## Context 模式

### 1. 何时使用 Context

当数据需要被"深层"组件访问时：

```javascript
// 创建 Context
const ThemeContext = useContext(createContext);

// Provider
function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Page />
    </ThemeContext.Provider>
  );
}

// 使用
function Page() {
  const theme = useContext(ThemeContext);
  return <div className={theme}>...</div>;
}
```

### 2. Context 分离原则

拆分频繁变化的 Context：

```javascript
// ❌ 一个 Context 包含所有
const AppContext = createContext({
  theme: 'dark',
  user: null,
  notifications: []
});

// ✅ 分离
const ThemeContext = createContext('dark');
const UserContext = createContext(null);
const NotificationsContext = createContext([]);
```

### 3. 派生 Context

```javascript
function App() {
  const [user, setUser] = useState(null);
  
  const authValue = useMemo(() => ({
    user,
    login: (u) => setUser(u),
    logout: () => setUser(null)
  }), [user]);
  
  return (
    <AuthContext.Provider value={authValue}>
      <Page />
    </AuthContext.Provider>
  );
}
```

## State 初始化

### useState 惰性初始化

复杂初始值使用函数：

```javascript
// ❌ 每次渲染都执行
const [items, setItems] = useState(expensiveCalculation());

// ✅ 只执行一次
const [items, setItems] = useState(() => expensiveCalculation());
```

## 最佳实践总结

| 场景 | 推荐方案 |
|------|----------|
| 简单值 | useState |
| 复杂逻辑 | useReducer |
| 深层传递 | Context |
| 派生值 | 直接计算，不用 state |
| 初始值复杂 | 惰性初始化 |
| 依赖上次值 | 函数式更新 |

更多详情请参考：[React 官方文档 - State 状态管理](https://zh-hans.react.dev/learn/managing-state)
