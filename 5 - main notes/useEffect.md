2024/08/16  -  02:38

Tags: [[react]] [[hooks]] 

# `useEffect`
```js
useEffect(setup, dependecies?);
//////////////////////
function ChatRoom({ roomId }) {  
	const [serverUrl, setServerUrl] = useState('https://localhost:1234');  
	useEffect(() => {  
		const connection = createConnection(serverUrl, roomId);  
		connection.connect();  

		return () => {  
			connection.disconnect();  
		};  
}, [serverUrl, roomId]);   

}  
```

- `setup`: The function with your Effect’s logic. Your setup function may also optionally return a _cleanup_ function. When your component is added to the DOM, React will run your setup function. After every re-render with changed dependencies, React will first run the cleanup function (if you provided it) with the old values, and then run your setup function with the new values. After your component is removed from the DOM, React will run your cleanup function.

- **optional** `dependencies`: The list of all reactive values referenced inside of the `setup` code. Reactive values include props, state, and all the variables and functions declared directly inside your component body. If your linter is [configured for React](https://react.dev/learn/editor-setup#linting), it will verify that every reactive value is correctly specified as a dependency. The list of dependencies must have a constant number of items and be written inline like `[dep1, dep2, dep3]`. React will compare each dependency with its previous value using the [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is) comparison. If you omit this argument, your Effect will re-run after every re-render of the component

- Effects **only run on the client.** They don’t run during server rendering.

downsides of writing fetches in `useEffect`:
- **Effects don’t run on the server.** This means that the initial server-rendered HTML will only include a loading state with no data.
- **Fetching directly in Effects makes it easy to create “network waterfalls”.** 
- **Fetching directly in Effects usually means you don’t preload or cache data.** For example, if the component unmounts and then mounts again, it would have to fetch the data again.
- **It’s not very ergonomic.** There’s quite a bit of boilerplate code involved when writing `fetch` calls in a way that doesn’t suffer from bugs like [race conditions.](https://maxrozen.com/race-conditions-fetching-data-react-with-useeffect)
If your Effect truly doesn’t use any reactive values, it will only run **after the initial render.**
If you pass no dependency array at all, your Effect runs **after every single render (and re-render)** of your component.
# References
https://react.dev/reference/react/useEffect