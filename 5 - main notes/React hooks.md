2025/08/14  -  12:18
status: 

Tags: [[react]]

Here are some important notes about how React Hooks actually work:

• **Hooks fundamentally rely on arrays and indexes**.
    ◦ When you call a hook like `useState`, React maintains an internal array.
    ◦ Each call to `useState` or `useEffect` accesses this array using a specific index.
    ◦ An internal `index` variable is incremented after each hook call to prepare for the next hook.
    ◦ A `localIndex` variable is used within `useState` to ensure the setter function uses the correct index, as it runs after the hooks are called.
    ◦ This global `index` needs to be **reset to 0 between component renders** via a `resetIndex` function (or `React's` equivalent internal mechanism) to ensure hooks retrieve the correct state or dependencies on subsequent renders.

• **The dependency on call order explains why hooks cannot be called inside loops or conditions**.
    ◦ If hooks were called conditionally or within loops, their indexes might change on every render. This would lead to hooks using the wrong state or dependencies, causing application-breaking bugs.

• **useState****'s internal workings for state management:**
    ◦ The state value is initially set only when the hook runs for the very first time (i.e., if it's currently `undefined`), preventing it from being reset on every re-render.
    ◦ When you update state, it triggers a re-render of the component.
    ◦ React forwards the `useState` call to its renderer using a special `currentDispatcher` field, which is set internally by the renderer (like React DOM) before the component is called.

• **useEffect****'s synchronization lifecycle and dependencies:**
    ◦ `useEffect` is called with a callback function and an optional dependency array.
    ◦ The callback runs either always (if no dependencies) or only if the dependencies have changed.
    ◦ React stores the "old" dependencies (similar to how state is stored, using the same `hooks` array and index mechanism) to compare them with the "current" dependencies.
    
    ◦ **Effects have a different lifecycle than components:** Components mount, update, or unmount, but an Effect's lifecycle is about **starting and stopping synchronization**. This cycle can happen multiple times while a component is mounted if its dependencies change.

    ◦ The effect's body defines how to **start synchronizing**, and the optional cleanup function it returns defines how to **stop synchronizing**.
 
    ◦ Mutable values (like global variables or `ref.current`) are generally **not reactive** and should not be included as dependencies because their changes don't trigger re-renders, and reading them can violate rendering purity.

• **Official React Hooks follow key rules:**
    ◦ **Composition:** Hooks should not conflict with each other. If one hook causes another to stop working, it breaks composition.
    ◦ **Debugging:** Bugs related to hooks should be easy to find.



# References
