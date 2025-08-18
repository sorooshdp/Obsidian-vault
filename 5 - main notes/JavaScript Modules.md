2025/08/18  -  16:31
status: 

Tags: [[js]] 
## What Is a Module in JavaScript?

A **module** is a self-contained block of code (usually a file) that encapsulates functionality and exposes only select parts for outside use. Modules help improve code organization, avoid polluting the global scope, and make testing or maintaining large codebases more manageable.[developer.mozilla+1](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)

---

## `require` vs `import`

|Feature|CommonJS (`require`)|ES Modules (`import`)|
|---|---|---|
|**Syntax**|`require()`, `module.exports`|`import`, `export`|
|**Loading**|Synchronous|Asynchronous (default)|
|**Dynamic Import**|Supported (anywhere)|`import()` (returns a Promise)|
|**Environment**|Node.js (mostly)|Browser + Node.js|
|**Tree shaking**|✘ No|✔ Yes|
|**Default Exports**|`module.exports = x`|`export default x`|
|**Top-level await**|✘ No|✔ Yes (Node.js 14.8.0+)|
|**File extensions**|Optional (`.js`)|Required (`.js`, `.mjs`)|
|**Caching**|`require.cache`|Separate, internal cache|
|**JSON imports**|Native|Supported (Node.js 17.1.0+)|

[betterstack+3](https://betterstack.com/community/guides/scaling-nodejs/commonjs-vs-esm/)

## Key Differences

- **Syntax:** `CommonJS` uses `require`, ESM uses `import`. ESM syntax is more concise and future-proof.
- **Loading:** `CommonJS` is **synchronous** (blocks code), ESM is **asynchronous** (does not block).
- **Dynamic Import:** `CommonJS` supports dynamic `require` anywhere; ESM supports `import()` as a function returning a promise (also dynamic).
- **Environment:** `CommonJS` is Node.js-native; ESM works in browsers and Node.js (with `.mjs`, or `"type": "module"` in `package.json`).
- **Tree Shaking:** ESM supports tree shaking (removes unused code during bundling); `CommonJS` does not.

---

## Usage Guidelines

- **Legacy Node.js projects:** Continue with **`CommonJS`/require** for compatibility.
- **Modern apps (web or Node.js):** Prefer **ES Modules/import** for future compatibility and better toolchain support.
- **Dynamic module loading:** Use `require` in `CommonJS`, `import()` in ESM.
- **Browser modules:** Only ES Modules are natively supported by browsers—no browser supports `require` without a bundler like `Webpack` or `Browserify`.

---

## Module Patterns Before Native Support

Before ES6 modules were standardized, modularity was achieved with:

- **Object Encapsulation**: Wrapping code in objects.
- **IIFE (Immediately Invoked Function Expression):** Creating isolated scopes to avoid leaking variables globally.[jsdevspace.substack](https://jsdevspace.substack.com/p/understanding-javascript-modules)

---

## Dynamic Imports in ESM

You can dynamically load ES modules like so:
```js
import('./myModule.js').then(module => {     module.function(); });
```
This is useful for code-splitting, lazy-loading parts of your app only when needed.[developer.mozilla](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)

---
## Best Practices

- Use **named exports** for clarity (`export function`).
- Use **default exports** when a module only exposes a single main function/class.
- Avoid mixing `CommonJS` and ESM in the same file.
- Take advantage of tree-shaking by using ES Modules when possible.

---
	
## Summary of When to Use What

|Scenario|Module System|
|---|---|
|Node.js only, legacy|CommonJS (`require`)|
|Browser + Node.js, modern|ES Modules (`import`)|
|Need for dynamic runtime import|CommonJS: `require`; ESM: `import()`|
|Frontend frameworks (React, etc.)|ES Modules (`import`)|

# References
1. [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)
2. [https://www.turing.com/kb/javascript-modules](https://www.turing.com/kb/javascript-modules)
3. [https://dev.to/nishanthan-k/understanding-require-vs-import-in-javascript-a-practical-guide-4p8l](https://dev.to/nishanthan-k/understanding-require-vs-import-in-javascript-a-practical-guide-4p8l)
4. [https://dev.to/rahulvijayvergiya/commonjs-cjs-vs-ecmascript-modules-esm-in-javascript-369c](https://dev.to/rahulvijayvergiya/commonjs-cjs-vs-ecmascript-modules-esm-in-javascript-369c)
5. [https://betterstack.com/community/guides/scaling-nodejs/commonjs-vs-esm/](https://betterstack.com/community/guides/scaling-nodejs/commonjs-vs-esm/)
6. [https://dev.to/vinay_madan/difference-between-commonjs-and-esm-modules-4pff](https://dev.to/vinay_madan/difference-between-commonjs-and-esm-modules-4pff)
7. [https://jsdevspace.substack.com/p/understanding-javascript-modules](https://jsdevspace.substack.com/p/understanding-javascript-modules)
8. [https://www.freecodecamp.org/news/javascript-es-modules-and-module-bundlers/](https://www.freecodecamp.org/news/javascript-es-modules-and-module-bundlers/)
9. [https://dev.to/sojinsamuel/a-complete-beginners-guide-to-javascript-modules-2022-noobs-are-welcome-4lhm](https://dev.to/sojinsamuel/a-complete-beginners-guide-to-javascript-modules-2022-noobs-are-welcome-4lhm)
10. [https://savvy.co.il/en/blog/complete-javascript-guide/javascript-modules/](https://savvy.co.il/en/blog/complete-javascript-guide/javascript-modules/)
11. [https://dev.to/doccaio/why-are-they-avoiding-using-require-and-using-import-in-javascript-k70](https://dev.to/doccaio/why-are-they-avoiding-using-require-and-using-import-in-javascript-k70)
12. [https://codeparrot.ai/blogs/require-vs-import](https://codeparrot.ai/blogs/require-vs-import)