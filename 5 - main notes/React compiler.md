2025/08/10  -  11:43
status: 

Tags: [[react]]  [[compiler]] [[Performance]]

The React Compiler is a highly anticipated feature in the React ecosystem, designed to **automatically `memoize` everything it can**, addressing long-standing debates about when and how often to use `React's` `memoization` features.

**Key Problems Solved by React Compiler:**

• **Unnecessary Re-renders:** Traditionally, when a parent component re-renders, all its child components also re-render, even if their props haven't changed. To avoid this, developers had to manually wrap components in `memo()`. The React Compiler eliminates this need, as it automatically handles `memoization`.

• **Manual `Memoization` Overhead:** Developers often found themselves manually writing `useMemo` or `useCallback` hooks, wondering why this couldn't be automated. The compiler provides **fine-grained reactivity**, ensuring that only the relevant parts of your application re-render when state changes, without manual intervention.

**How the React Compiler Works:**

• **Automatic `Memoization`:** It automatically `memoizes` as much as possible.

• **Efficiency Gains:** In some cases, the React Compiler can `memoize` things **more efficiently** than `useMemo` or `useCallback`. For example, for simple string concatenation, it can directly put the result into JSX, whereas `useMemo` would still call a function.

• **Underlying Mechanism:** Instead of using `useMemo` or `useCallback`, the compiler **manually caches values** and reuses them if dependencies remain the same. For a function like `findFavoriteMovies`, it calculates a cache size, stores dependencies (e.g., `movies`, `person`), and the computed value (e.g., `favorite movies`). It then checks if dependencies have changed to decide whether to recalculate or reuse the cached result.

• **Replacing** **memo****:** It `memoizes` the output of the component itself. If the output (e.g., `favorite movies`) differs from the cached value, it's recalculated and cached; otherwise, the cached value is reused.

• **Symbol-based `Memoization`:** For components with a single child that lack props, the compiler uses a symbol to **`memoize` the output once and reuse it indefinitely**.

**Assumptions and Compatibility:** The optimizations performed by the React Compiler rely on three main assumptions about your code:

1. **Valid JavaScript:** Your code must be valid JavaScript.
2. **Defined Values:** You test if values and properties are defined before accessing them.
3. **Rules of React:** Your code must follow the "Rules of React," such as calling hooks only at the top level.

Most applications, especially those using linters and TypeScript, are likely to satisfy these assumptions, meaning you might not need to rewrite existing code to benefit from the compiler. If these rules are broken, the compiler is designed to **safely skip compilation** for the affected component, reducing the risk of unexpected behavior. Additionally, a new **`ESLint` plugin** will display any detected violations of the React Rules.

**Integration with Existing `Memoization`:**

• While the compiler automates much of `memoization`, there are still cases where **manual `memoization` outside of React** might make sense, especially for very expensive functions reused across the entire application.

• The current recommendation is to **start omitting** **`useMemo`** **or** **`useCallback`** **hooks** once you begin using the compiler.

• The compiler will also compile existing manually `memoized` code and verify that the `memoization` logic remains the same, ensuring compatibility with "hacky code" centered around `useMemo`.

**How to Use the React Compiler:**

1. **Check Compatibility:** Run a script to determine how much of your codebase is compatible and identify any incompatible libraries (e.g., `MobX`).

2. **Upgrade React:** You need to upgrade to **React 19 RC** (Release Candidate), which supports the compiler.

3. **Install Babel Plugin:** Add a Babel plugin to your project to integrate the React Compiler into the compilation process.

4. **Verify Compilation:** After setup, you can check if your components are being compiled by hovering over them in the **React `DevTools`**; a small star icon with text indicates auto-`memoization` by the compiler.

5. **Disable Compilation:** If needed, you can disable compilation for specific components using the `"use no memo"` directive. It's also possible to enable compilation only for files with specific prefixes or directories.



# References
