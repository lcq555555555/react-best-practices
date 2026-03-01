---
name: "react-component-patterns"
description: "React Component Patterns - composition, props, children, and component design. Invoke when designing or refactoring React components."
---

# React 组件模式

基于 React 官方文档的组件设计模式。

## 组件组合

### 1. Children Props

使用 props.children 组合组件：

```javascript
function Card({ children }) {
  return (
    <div className="card">
      {children}
    </div>
  );
}

function App() {
  return (
    <Card>
      <h2>标题</h2>
      <p>内容</p>
    </Card>
  );
}
```

### 2. Slot Props ( Props 插槽)

更灵活的组合方式：

```javascript
function SplitPane({ left, right }) {
  return (
    <div className="split-pane">
      <div className="left">{left}</div>
      <div className="right">{right}</div>
    </div>
  );
}

function App() {
  return (
    <SplitPane
      left={<Contacts />}
      right={<Chat />}
    />
  );
}
```

## Props 设计

### 1. 向下流动原则

Props 只能向下流动：

```javascript
// 父组件
function Parent() {
  const data = getData();
  return <Child data={data} />;
}

// 子组件
function Child({ data }) {
  return <div>{data.name}</div>;
}
```

### 2. 避免 Props 穿透

逐层传递太深时使用 Context：

```javascript
// ❌ Props 穿透
function A({ user }) {
  return <B user={user} />;
}
function B({ user }) {
  return <C user={user} />;
}
function C({ user }) {
  return <div>{user.name}</div>;
}

// ✅ 使用 Context
const UserContext = createContext();
function C() {
  const user = useContext(UserContext);
  return <div>{user.name}</div>;
}
```

## 组件分类

### 1. 受控组件 vs 非受控组件

```javascript
// 受控组件 - state 由父组件控制
function ControlledInput({ value, onChange }) {
  return <input value={value} onChange={onChange} />;
}

// 非受控组件 - 内部自己管理 state
function UncontrolledInput() {
  const [value, setValue] = useState('');
  return <input value={value} onChange={e => setValue(e.target.value)} />;
}
```

### 2. 展示组件 vs 容器组件

```javascript
// 展示组件 - 只负责 UI
function UserCard({ name, email }) {
  return (
    <div>
      <h3>{name}</h3>
      <p>{email}</p>
    </div>
  );
}

// 容器组件 - 负责逻辑和数据
function UserCardContainer({ userId }) {
  const { user, isLoading } = useUser(userId);
  
  if (isLoading) return <Loading />;
  
  return <UserCard name={user.name} email={user.email} />;
}
```

## 组件设计原则

### 1. 单一职责

一个组件只做一件事：

```javascript
// ❌ 职责过多
function UserProfile({ user, onEdit, onDelete, onFollow }) {
  return (
    <div>
      <Avatar src={user.avatar} />
      <Info name={user.name} bio={user.bio} />
      <Actions onEdit={onEdit} onDelete={onDelete} onFollow={onFollow} />
    </div>
  );
}

// ✅ 分离
function UserProfile({ user, onEdit, onDelete, onFollow }) {
  return (
    <div>
      <Avatar src={user.avatar} />
      <Info name={user.name} bio={user.bio} />
      <Actions onEdit={onEdit} onDelete={onDelete} onFollow={onFollow} />
    </div>
  );
}
```

### 2. 提取可变部分

```javascript
// ❌ 硬编码
function PostList() {
  return (
    <ul>
      <li>第一篇博客</li>
      <li>第二篇博客</li>
      <li>第三篇博客</li>
    </ul>
  );
}

// ✅ 提取数据
function PostList({ posts }) {
  return (
    <ul>
      {posts.map(post => <li key={post.id}>{post.title}</li>)}
    </ul>
  );
}
```

## 组件通信

### 1. 父 -> 子: Props

```javascript
function Parent() {
  return <Child message="hello" />;
}

function Child({ message }) {
  return <div>{message}</div>;
}
```

### 2. 子 -> 父: 回调函数

```javascript
function Parent() {
  const handleSubmit = (data) => {
    console.log(data);
  };
  
  return <ChildForm onSubmit={handleSubmit} />;
}

function ChildForm({ onSubmit }) {
  const handleClick = () => {
    onSubmit({ name: 'John' });
  };
  
  return <button onClick={handleClick}>提交</button>;
}
```

### 3. 兄弟组件: 父组件协调

```javascript
function Parent() {
  const [count, setCount] = useState(0);
  
  return (
    <>
      <Display count={count} />
      <Counter value={count} onChange={setCount} />
    </>
  );
}
```

## 组件复用

### 1. 提取为组件

```javascript
// 重复的代码
function LoginButton() {
  return <button className="btn-primary">登录</button>;
}

function RegisterButton() {
  return <button className="btn-primary">注册</button>;
}

// ✅ 提取
function Button({ children, variant = 'primary' }) {
  return <button className={`btn-${variant}`}>{children}</button>;
}

function LoginButton() {
  return <Button>登录</Button>;
}

function RegisterButton() {
  return <Button>注册</Button>;
}
```

### 2. Props 化

```javascript
// 类似的 UI，不同的数据
function ProfileCard1() {
  return (
    <div className="card">
      <img src="/avatar1.jpg" />
      <h3>用户A</h3>
    </div>
  );
}

function ProfileCard2() {
  return (
    <div className="card">
      <img src="/avatar2.jpg" />
      <h3>用户B</h3>
    </div>
  );
}

// ✅ 提取 Props
function ProfileCard({ src, name }) {
  return (
    <div className="card">
      <img src={src} />
      <h3>{name}</h3>
    </div>
  );
}
```

## 总结

| 模式 | 用途 |
|------|------|
| Children Props | 灵活的 UI 组合 |
| 受控/非受控 | State 管理方式选择 |
| 展示/容器 | 分离 UI 和逻辑 |
| Props 回调 | 子向父通信 |
| 单一职责 | 组件职责清晰 |

更多详情参考：[React 官方文档 - 组件和 Props](https://zh-hans.react.dev/learn/passing-props-to-a-component)
