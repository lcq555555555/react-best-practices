---
name: "react-effects"
description: "React useEffect and Side Effects patterns. Invoke when dealing with useEffect, DOM manipulation, subscriptions, or synchronizing with external systems."
---

# React Effects 最佳实践

基于 React 官方文档的 useEffect 指南。

## 什么是 Effect

Effect 用于将组件与外部系统同步（如浏览器 API、网络、第三方库）。

```javascript
useEffect(() => {
  // 这里是副作用代码
  // 在渲染完成后执行
}, [dependencies]);
```

## 何时使用 Effect

### ✅ 适合使用 Effect

- 与外部系统同步（浏览器 API、WebSocket、第三方库）
- 订阅/取消订阅事件
- 数据获取
- 操作 DOM

### ❌ 不需要使用 Effect

- 根据其他 state 调整某些 state
- 简单的计算派生值

```javascript
// ❌ 不需要 Effect
function Counter() {
  const [count, setCount] = useState(0);
  const [double, setDouble] = useState(0);

  useEffect(() => {
    setDouble(count * 2);  // 不需要！
  }, [count]);

  return <div>{double}</div>;
}

// ✅ 直接计算
function Counter() {
  const [count, setCount] = useState(0);
  const double = count * 2;  // 直接计算

  return <div>{double}</div>;
}
```

## Effect 三步曲

### 1. 声明 Effect

```javascript
function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  });  // 每次渲染后都运行

  return <video ref={ref} src={src} />;
}
```

### 2. 指定依赖项

```javascript
useEffect(() => {
  if (isPlaying) {
    ref.current.play();
  } else {
    ref.current.pause();
  }
}, [isPlaying]);  // 只有 isPlaying 变化时才运行
```

### 3. 添加清理函数

```javascript
useEffect(() => {
  const connection = createConnection();
  connection.connect();

  return () => {
    connection.disconnect();  // 清理函数
  };
}, [roomId]);
```

## 常见模式

### 1. 订阅事件

```javascript
useEffect(() => {
  function handleScroll(e) {
    console.log(window.scrollY);
  }

  window.addEventListener('scroll', handleScroll);

  return () => {
    window.removeEventListener('scroll', handleScroll);
  };
}, []);
```

### 2. 数据获取

```javascript
useEffect(() => {
  let ignore = false;

  async function fetchData() {
    const response = await fetch(`/api/user/${userId}`);
    const json = await response.json();
    if (!ignore) {
      setData(json);
    }
  }

  fetchData();

  return () => {
    ignore = true;  // 忽略过期的响应
  };
}, [userId]);
```

### 3. 操作 DOM

```javascript
useEffect(() => {
  const node = ref.current;
  node.style.opacity = 1;

  return () => {
    node.style.opacity = 0;  // 清理动画
  };
}, []);
```

### 4. 订阅第三方库

```javascript
useEffect(() => {
  const map = new MapWidget(ref.current);
  map.setZoom(zoomLevel);

  return () => {
    map.remove();  // 清理
  };
}, [zoomLevel]);
```

## 依赖项规则

### ⚠️ 常见错误

```javascript
// ❌ 错误：依赖项与代码不匹配
useEffect(() => {
  console.log('Count: ' + count);
}, []);  // 错误！count 在代码中使用了

// ✅ 正确
useEffect(() => {
  console.log('Count: ' + count);
}, [count]);

// ✅ 正确：如果不需要依赖
useEffect(() => {
  console.log('Mounted');
}, []);  // 空数组 = 只运行一次
```

### 稳定引用

ref 和 setState 是稳定的，可以省略：

```javascript
// ref 是稳定的
const ref = useRef(null);
// ref 可以在依赖中省略

// setState 是稳定的
const [_, setCount] = useState(0);
// setCount 可以在依赖中省略
```

## 开发环境行为

### Strict Mode 会运行两次

在开发环境下，React 会**故意**挂载-卸载-再挂载组件，来检测是否正确实现了清理函数。

```javascript
// 开发环境输出：
// ✅ 连接中...     (第一次挂载)
// ❌ 连接断开。    (卸载)
// ✅ 连接中...     (再次挂载)

// 生产环境输出：
// ✅ 连接中...     (只运行一次)
```

**这是正常的**，不要试图"修复"它，而是要正确实现清理函数。

### ❌ 错误的"修复"

```javascript
// ❌ 不要这样"修复"
const ref = useRef(null);
useEffect(() => {
  if (!ref.current) {
    ref.current = createConnection();
    ref.current.connect();
  }
}, []);

// 这只是隐藏了 bug，并没有真正解决问题！
```

## 不适合使用 Effect

### 1. 用户交互

```javascript
// ❌ 错误：购买是用户交互，不是渲染结果
useEffect(() => {
  fetch('/api/buy', { method: 'POST' });
}, []);

// ✅ 正确：购买应该在事件处理中
function BuyButton() {
  function handleClick() {
    fetch('/api/buy', { method: 'POST' });
  }
  return <button onClick={handleClick}>购买</button>;
}
```

### 2. 初始化逻辑

```javascript
// ❌ 错误：不需要 Effect
useEffect(() => {
  loadUserData();
}, []);

// ✅ 正确：放在组件外部
if (typeof window !== 'undefined') {
  loadUserData();
}
```

## 常见问题

### 1. 死循环

```javascript
// ❌ 错误：Effect 中更新 state
useEffect(() => {
  setCount(count + 1);  // 触发重新渲染 → 再次运行 Effect → 死循环！
}, [count]);

// ✅ 正确：使用函数式更新
useEffect(() => {
  setCount(c => c + 1);  // 不依赖外部 count
}, []);
```

### 2. 竞态条件

```javascript
// ❌ 错误：没有清理，可能出现旧请求覆盖新请求
useEffect(() => {
  fetch(`/api/user/${id}`).then(setUser);
}, [id]);

// ✅ 正确：使用清理标志
useEffect(() => {
  let ignore = false;
  fetch(`/api/user/${id}`)
    .then(res => res.json())
    .then(data => {
      if (!ignore) setUser(data);
    });
  return () => { ignore = true; };
}, [id]);
```

### 3. 闭包陷阱

```javascript
// ⚠️ 注意：Effect 会"捕获"渲染时的值
useEffect(() => {
  setTimeout(() => {
    console.log('Count: ' + count);  // 这里 count 是捕获时的值
  }, 3000);
}, [count]);
// 如果 count 变为 5，3秒后仍然打印 "Count: 0"
```

## 总结

| 场景 | 做法 |
|------|------|
| 连接外部系统 | 清理函数断开连接 |
| 订阅事件 | 清理函数退订 |
| 获取数据 | 清理函数忽略过期响应 |
| 动画 | 清理函数重置状态 |
| 用户交互 | 用事件处理函数，不用 Effect |

## 替代方案

对于数据获取，推荐使用：

- **框架内置**（Next.js, Remix 等）
- **TanStack Query / SWR** - 自动缓存、去重、背景刷新

```javascript
// 推荐：使用 SWR
import useSWR from 'swr';

function Profile() {
  const { data, error } = useSWR('/api/user', fetcher);
  // 自动缓存、去重、处理 loading/error 状态
}
```

更多详情请参考：
- [React 官方文档 - Synchronizing with Effects](https://zh-hans.react.dev/learn/synchronizing-with-effects)
- [React 官方文档 - useEffect API](https://zh-hans.react.dev/reference/react/useEffect)
