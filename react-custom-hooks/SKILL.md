---
name: "react-custom-hooks"
description: "React Custom Hooks best practices and patterns. Invoke when writing or refactoring React components to extract reusable logic."
---

# React Custom Hooks 最佳实践

基于 React 官方文档的自定义 Hook 指南。

## 什么是自定义 Hook

自定义 Hook 是一个函数，以 `use` 开头，用于在组件间共享可复用的逻辑。

```javascript
function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function handleOnline() { setIsOnline(true); }
    function handleOffline() { setIsOnline(false); }
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);
  return isOnline;
}
```

## 命名规则

- **必须**以 `use` 开头（如 `useOnlineStatus`）
- 紧跟大写字母
- 这是 React 强制执行的命名公约

```javascript
// ✅ 正确
function useAuth() { return useContext(Auth); }

// ✅ 正确 - 没有调用任何 Hook
function getSorted(items) { return items.slice().sort(); }

// ❌ 错误 - 以 use 开头但没有调用 Hook
function useData() { return fetch('/api/data'); }
```

## 核心原则

### 1. 共享逻辑，不共享状态

每次调用 Hook 都是独立的：

```javascript
function Form() {
  const firstNameProps = useFormInput('Mary');
  const lastNameProps = useFormInput('Poppins');
  // ...
}
// firstNameProps 和 lastNameProps 是完全独立的
```

### 2. Hook 响应最新值

组件重新渲染时，Hook 会接收到最新的 props 和 state：

```javascript
function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');
  useChatRoom({ roomId, serverUrl });
  // 每次 roomId 或 serverUrl 变化，Hook 会自动重新运行
}
```

### 3. 保持纯函数

Hook 代码应该和组件一样保持纯粹。

## 提取时机

**适合提取**：
- 多个组件使用相同的 state + Effect 逻辑
- 需要与外部系统同步（API、WebSocket、浏览器 API）
- 复杂的数据获取逻辑

**不需要提取**：
- 简单的单个 useState 调用

```javascript
// ❌ 过度提取
function useName(initial) {
  const [name, setName] = useState(initial);
  return [name, setName];
}

// ✅ 简单直接
const [name, setName] = useState('');
```

## 高级模式

### 1. 传递回调函数给 Hook

```javascript
// Hook
export function useChatRoom({ serverUrl, roomId, onReceiveMessage }) {
  const onMessage = useEffectEvent(onReceiveMessage);
  useEffect(() => {
    const connection = createConnection({ serverUrl, roomId });
    connection.on('message', (msg) => { onMessage(msg); });
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
}

// 使用
function ChatRoom() {
  useChatRoom({
    roomId,
    serverUrl,
    onReceiveMessage(msg) {
      showNotification(msg);
    }
  });
}
```

### 2. 提取数据获取逻辑

```javascript
function useData(url) {
  const [data, setData] = useState(null);
  useEffect(() => {
    if (!url) return;
    let ignore = false;
    fetch(url)
      .then(r => r.json())
      .then(json => { if (!ignore) setData(json); });
    return () => { ignore = true; };
  }, [url]);
  return data;
}

// 使用
function ShippingForm({ country }) {
  const cities = useData(`/api/cities?country=${country}`);
  const [city, setCity] = useState(null);
  const areas = useData(city ? `/api/areas?city=${city}` : null);
}
```

## 最佳实践

### ✅ 推荐

```javascript
// 1. 名称清晰表达用途
function useData(url)
function useChatRoom(options)
function useMediaQuery(query)

// 2. 专注于具体用例
function useOnlineStatus()  // 专注
function useEffectOnce()   // ❌ 避免 - 抽象过度
```

### ❌ 避免

```javascript
// 1. 避免"生命周期"Hook
useMount(fn)      // ❌
useEffectOnce(fn) // ❌
useUpdateEffect(fn) // ❌

// 2. 避免过度抽象
function useStateWrapper(initial) {  // ❌
  return useState(initial);
}
```

## 迁移到更好模式

自定义 Hook 让升级到更好的 React 模式更容易：

```javascript
// 当前 - 使用 useEffect
function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    // ...
  }, []);
  return isOnline;
}

// 未来 - 可以直接替换为 useSyncExternalStore
function useOnlineStatus() {
  return useSyncExternalStore(
    subscribe,
    () => navigator.onLine,
    () => true
  );
}
```

## 总结

| 原则 | 说明 |
|------|------|
| 命名 | 必须以 `use` 开头 |
| 独立性 | 每次调用完全独立 |
| 响应性 | 自动响应最新 props/state |
| 纯粹性 | 和组件一样保持纯函数 |
| 聚焦 | 名称表达具体用途 |

更多详情请参考：[React 官方文档 - 自定义 Hook](https://zh-hans.react.dev/learn/reusing-logic-with-custom-hooks)
