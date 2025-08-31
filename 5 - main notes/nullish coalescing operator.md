2025/09/01  -  02:36
status: 

Tags: [[js]] 

The `??` operator in JavaScript is called the **nullish coalescing operator**.

It returns the **right-hand side value only if the left-hand side is `null` or `undefined`**, otherwise it returns the left-hand side value.

---

```js
const otpCode = (passOtp ?? otp).join("");
```

- This means:
    - Use `passOtp` if it is _not_ `null` or `undefined`.
    - Otherwise, use `otp`.
- It’s useful to provide fallback/default values only when a variable is truly missing (`null` or `undefined`), as opposed to other `falsy` values like `0`, `false`, or `""`.

---
## Example:

```js
let a = null;
console.log(a ?? "default"); // "default"

let b = 0;
console.log(b ?? 42); // 0 (not 42), because 0 is NOT null or undefined

let c;
console.log(c ?? "missing"); // "missing"

```

---

**References:**

- [MDN: Nullish coalescing operator (??)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing)[](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing)[developer.mozilla](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing)
    

1. [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing)
2. [https://javascript.info/nullish-coalescing-operator](https://javascript.info/nullish-coalescing-operator)
3. [https://www.freecodecamp.org/news/what-is-the-nullish-coalescing-operator-in-javascript-and-how-is-it-useful/](https://www.freecodecamp.org/news/what-is-the-nullish-coalescing-operator-in-javascript-and-how-is-it-useful/)
4. [https://fa.javascript.info/nullish-coalescing-operator](https://fa.javascript.info/nullish-coalescing-operator)
5. [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing_assignment](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing_assignment)
6. [https://www.youtube.com/watch?v=TOBlbEe224Q](https://www.youtube.com/watch?v=TOBlbEe224Q)
7. [https://www.geeksforgeeks.org/javascript/javascript-nullish-coalescing-operator/](https://www.geeksforgeeks.org/javascript/javascript-nullish-coalescing-operator/)
8. [https://www.freecodecamp.org/news/javascript-advanced-operators/](https://www.freecodecamp.org/news/javascript-advanced-operators/)
9. [https://en.wikipedia.org/wiki/Null_coalescing_operator](https://en.wikipedia.org/wiki/Null_coalescing_operator)


# References
