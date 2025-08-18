2024/07/27  -  10:46

Tags: [[hoisting]] [[js]]

# The compiler strikes again
when you see `var a = 2` you probably think of that as one statement but the engine will first compile the code before interpreting, and sees it as two statements: `var a` and `a = 2` and the first statement is processed during the compilation phase. the second one is left in place for the execution phase. 
it's also important to note hoisting is **per-scope**. 

```js
function foo() {
	var a;
	console.log( a ); // undefined
	a = 2;
}
```

function declarations are hoisted, but function expressions are not.
and functions are hoisted first and then variables.

```js
foo(); // not RefrenceError, but TypeError!!
bar(); // RefrenceError

var foo = function bar() {
	//...
}
```


# References
