2024/07/26  -  19:35

Tags: [[js]] [[scope]] 

# Function vs. Block scope
Scope defines the accessibility of variables and functions at different parts of your code.
Functions are the most common unit of scope in JavaScript. Variables and functions that are declared inside another function are essentially "hidden" from any of the enclosing "scopes", which is an intentional design principle of good software.

## try / catch 
It's a very little known fact that JavaScript in ES3 specified the variable declaration in the catch clause of a try/catch to be block-scoped to the catch block.

```js
try { 
	undefined(); // illegal operation to force an exception! 
} catch (err) { 
	console.log( err ); // works! 
} 
console.log( err ); // ReferenceError: `err` not found
```

## `let` / `const`
let key word attaches the variable declaration to the scope of whatever block it's contained in.
However, declarations made with let will not hoist to the entire scope of the block they appear in.
In addition to let , ES6 introduces `const` , which also creates a block-scoped variable, but whose value is fixed (constant)

```js
{
	console.log( bar ); // Refrence error!
	let bar = 2;
}
```


# References
