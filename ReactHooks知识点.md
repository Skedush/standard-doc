# React Hooks知识点

[TOC]

项目中必须安装eslint插件: eslint-plugin-react-hooks 

```
// 你的 ESLint 配置
{
  "plugins": [
    // ...
    "react-hooks"
  ],
  "rules": {
    // ...
    "react-hooks/rules-of-hooks": "error", // 检查 Hook 的规则
    "react-hooks/exhaustive-deps": "warn" // 检查 effect 的依赖
  }
}
```

## Hooks 顺序调用

React 官网介绍了 Hook 的这样一个限制：

> 不要在循环，条件或嵌套函数中调用 Hook， 确保总是在你的 React 函数的最顶层以及任何 return 之前调用他们。遵守这条规则，你就能确保 Hook 在每一次渲染中都按照同样的顺序被调用。这让 React 能够在多次的 useState 和 useEffect 调用之间保持 hook 状态的正确。(如果你对此感到好奇，我们在下面会有更深入的解释。)

这里涉及到 Hooks实现原理中的 `Hooks链表` 概念，关于这方面，我推荐下面两篇文章

* [关于React hook，我做了个违背祖训的决定](https://mp.weixin.qq.com/s/BxyzWDNGxPgRtz9WujT3_Q)
* [为什么顺序调用对 React Hooks 很重要？](https://overreacted.io/zh-hans/why-do-hooks-rely-on-call-order/)

## Hooks 闭包

> 关于 Hook 中的闭包：
useEffect、useMemo、useCallback都是自带闭包的。也就是说，每一次组件的渲染，其都会捕获当前组件函数上下文中的状态(state、props)。所以每一次这三种 Hook 的执行，反映的也都是当时的状态，无法获取最新的状态。对于这种情况，应该使用 ref 来访问。

可以在 `codesandbox` 上使用下面的例子进行验证：

```
function DelayCount() {
  const [count, setCount] = useState(0);

  function handleClickAsync() {
    setTimeout(function delay() {
      setCount(count + 1); // 问题所在：此时的 count 为5s前的count！！！
    }, 5000);
  }

  function handleClickSync() {
    setCount(count + 1);
  }

  return (
    <div>
      {count}
      <button onClick={handleClickAsync}>异步加1</button>
      <button onClick={handleClickSync}>同步加1</button>
    </div>
  );
}
```

## useCallback & useMemo

useCallback 和 useMemo 都可缓存函数的引用或值，但是从更细的使用角度来说 useCallback 缓存函数的引用，useMemo 缓存计算数据的值。

### useCallback

先看如下代码：

```
function App() {
  const [count, setCount] = React.useState(0);
  const handleCount = React.useCallback(
  	() => setCount(count => count + 1), 
  	[]
  );
  
  return (
    <div>
      <div>Count is {count}</div>
      <div>
        <button onClick={handleCount}>Increment Count</button>
        <button onClick={handleTotal}>Increment Total</button>
      </div>
    </div>
  )
}

```

设想下在没有使用 useCallback 情况下，每次渲染时 handleCount 都是重新创建的一个新函数，当我们将 handleCount 作为props传递给其他组件时会导致像 PureComponent、shouldComponentUpdate、React.memo 等相关优化失效(因为每次都是不同的函数)，为了解决上述的问题，我们需要引入 useCallback，通过使用它的依赖缓存功能，在合适的时候将 handleCount 缓存起来

### useMemo

useMemo 和 useCallback 几乎是99%像是，当我们理解了 useCallback 后理解 useMemo 就非常简单。  
他们的唯一区别就是：useCallback 是根据依赖 (deps) 缓存第一个入参的 (callback)。useMemo是根据依赖 (deps) 缓存第一个入参(callback) 执行后的值。  

**useMemo一般用于密集型计算大的一些缓存**。

## useEffect

> 非常重要，细节很多

### useEffect的执行顺序

* 触发更新时，`FunctionComponent` 被执行，执行到 `useEffect` 时会判断他的第二个参数 `deps` 是否有变化。
* 如果 `deps` 变化，则 `useEffect` 对应 `FunctionComponent` 的 `fiber` 会被打上 `Passive`（即：需要执行useEffect）的标记。
* **在渲染器中，遍历 `effectList` 过程中遍历到该 `fiber` 时，发现 `Passive` 标记，则依次执行该 `useEffect` 的 `destroy`（即useEffect回调函数的返回值函数）与 `create`（即useEffect回调函数）**。

加粗部分请仔细理解，这对你写 `custom hooks` 非常重要，我将用下面的例子进行佐证，以下是一个选择信息收件人的模拟示例，它显示了当前选择的好友是否在线：

```
const friendList = [
  { id: 1, name: 'Phoebe' },
  { id: 2, name: 'Rachel' },
  { id: 3, name: 'Ross' },
];

function ChatRecipientPicker() {
  const [recipientID, setRecipientID] = useState(1);
  const isRecipientOnline = useFriendStatus(recipientID);

  return (
    <>
      <Circle color={isRecipientOnline ? 'green' : 'red'} />
      <select
        value={recipientID}
        onChange={e => setRecipientID(Number(e.target.value))}
      >
        {friendList.map(friend => (
          <option key={friend.id} value={friend.id}>
            {friend.name}
          </option>
        ))}
      </select>
    </>
  );
}

function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);
  const handleStatusChange = (status) => setIsOnline(status.isOnline);
  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
    };
  });
  return isOnline;
}
```

当改变收件人时，useFriendStatus Hook 就会退订上一个好友的状态，订阅接下来的这个。


### 多个 useEffect 的执行顺序

useEffect 和 useLayoutEffect 是按照它们在代码中出现的顺序调用的，就像 useState 调用

```
useEffect(() => {
  console.log('useEffect-1')
})
useEffect(() => {
  console.log('useEffect-2')
})
useLayoutEffect(() => {
  console.log('useLayoutEffect-1')
})
useLayoutEffect(() => {
  console.log('useLayoutEffect-2')
})
```

打印的结果是 `useLayoutEffect-1` -> `useLayoutEffect-2` -> `useEffect-1` -> `useEffect-2`


### 如何在 useEffect 中使用 async 

> useEffect的 callback 函数要么返回一个能清除副作用的函数，要么就不返回任何内容。

而 async 函数返回的是 Promise 对象，那我们要怎么在 useEffect 的callback 中使用 async 呢？
最简单的方法是IIFE（自执行函数）：

```
useEffect(() => {
  (async () => {
    await fetchSomething();
  })();
}, []);
```

## 项目实战

### 如何获取组件实例

```
const { forwardRef, useRef, useImperativeHandle } = React;

// We need to wrap component in `forwardRef` in order to gain
// access to the ref object that is assigned using the `ref` prop.
// This ref is passed as the second parameter to the function component.
const Child = forwardRef((props, ref) => {

  // The component instance will be extended
  // with whatever you return from the callback passed
  // as the second argument
  useImperativeHandle(ref, () => ({

    getAlert() {
      alert("getAlert from Child");
    }

  }));

  return <h1>Hi</h1>;
});

const Parent = () => {
  // In order to gain access to the child component instance,
  // you need to assign it to a `ref`, so we call `useRef()` to get one
  const childRef = useRef();

  return (
    <div>
      <Child ref={childRef} />
      <button onClick={() => childRef.current.getAlert()}>Click</button>
    </div>
  );
};
```






