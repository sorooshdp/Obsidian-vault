2025/09/01  -  02:18
status: 

Tags: [[react]] 

**React batching** is a mechanism where React groups multiple state updates together and processes them in a single render cycle, rather than rendering after every individual state change. This approach increases performance by minimizing unnecessary renders and ensures your UI stays in sync with the latest state.[Geeks for Geeks](https://www.geeksforgeeks.org/reactjs/what-is-automatic-batching-in-react-18/)[Geeks for Geeks](https://www.geeksforgeeks.org/reactjs/what-is-automatic-batching-in-react-18/)

## How Batching Works
- **Before React 18:**  
    Batching mainly worked for state updates inside React event handlers (like `onClick`). If you updated state multiple times in a handler, React would combine them into one render.
- **React 18 and onward:**  
    React introduced _automatic batching_, which means state updates inside timeouts, promises, `async` code, and native handlers are also batched by default. React waits for these operations to finish, then performs a single render with all state changes applied.[react](https://react.dev/learn/queueing-a-series-of-state-updates)[react](https://react.dev/learn/queueing-a-series-of-state-updates)
```js
// Counter example
const [count, setCount] = useState(0);
function handleClick() {
  setCount(count + 1);
  setCount(count + 1);
  // React will batch both updates and only re-render once, resulting in count + 1 (not +2)
}

```


# References
- [What is Automatic Batching in React 18?](https://www.geeksforgeeks.org/reactjs/what-is-automatic-batching-in-react-18/)
- [What is batching in react](https://www.geeksforgeeks.org/reactjs/what-is-automatic-batching-in-react-18/)
- [React docs: Queueing State Updates](https://react.dev/learn/queueing-a-series-of-state-updates)
- [queuing a series of updates](https://react.dev/learn/queueing-a-series-of-state-updates)