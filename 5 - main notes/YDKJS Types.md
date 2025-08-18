2024/07/29  -  11:34

Tags: [[types]] [[js]] 

## Built-in Types
JavaScript has seven built-in types, six of which are primitives:
- Null
- Undefined
- Boolean
- Number
- String
- Symbol (added in ES6)

```js
typeof undefined === "undefined"; // true
typeof true === "boolean"; // true
typeof 42 === "number"; // true
typeof "42" === "string"; // true
typeof { life: 42 } === "object"; // true
typeof Symbol() === "symbol"; // true
typeof null === "object"; // true
```
null is the only primitive value that is `"falsy"`, but that also returns `"object"` from the `typeof` check.
```js
typeof function a(){ /* .. */ } === "function"; // true
```
function is a "subtype" of object, is referred to as a "callable object" -- an object that has an internal `[[call]]` property that allows it to be invoked. Arrays, another common structure in JavaScript, are also objects. They are a subtype of objects with numerical indexing and an automatically updated `length` property.

```js
typeof [1,2,3] === "object"; // true, subtype of object
```

In JavaScript, variables don't have types -- **values have types**. Variables can hold any value, at any time. JS doesn't have "type enforcement," in that the engine doesn't insist that a variable always holds values of the same initial type that it starts out with.

### `undefined` vs. "undeclared"

Variables that have not been assigned a value have the `undefined` type. In contrast, "undeclared" variables are those that have not been declared in the accessible scope. This distinction is important for error handling and debugging in JavaScript.

```js
var a; console.log(a); // undefined 
console.log(b); // ReferenceError: b is not defined
```

# References
