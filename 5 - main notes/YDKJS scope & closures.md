
tags:  [[js]] [[scope]] [[closures]]

## compiler theory
Let's just say, for simplicity's sake, that any snippet of JavaScript has to be compiled before (usually right before!) it's executed. So, the JS compiler will take the program `var a = 2;` and compile it first, and then be ready to execute it, usually right away.

## understanding scope

- engine: responsible for start-to-finish compilation and execution of our JavaScript program.
- compiler: one of engine's friends, handles parsing and code generation
- scope: another friend of engine, collects and maintains a look-up list of all the declared identifiers (variables), and enforces a strict set of rules as to how these are accessible to currently executing code.
two distinct actions are taken for a variable assignment: First, Compiler declares a variable (if not previously declared in the current scope), and second, when executing, Engine looks up the variable in Scope and assigns to it, if found.

### RHS

```javascript
console.log(a);
```
The reference to `a` is an RHS reference, because nothing is being assigned to a here. Instead, we're looking-up to retrieve the value of a , so that the value can be passed to `console.log(..)`

### LHS 

```js 
a = 2;
```
The reference to a here is an LHS reference, because we don't actually care what the current value is, we simply want to find the variable as a target for the `= 2` assignment operation.

Unfulfilled RHS references result in `ReferenceError` s being thrown. Unfulfilled LHS references result in an automatic, implicitly-created global of that name (if not in "Strict Mode" ), or a `ReferenceError` (if in "Strict Mode" ).
### nested scope 
The simple rules for traversing nested Scope: Engine starts at the currently executing Scope, looks for the variable there, then if not found, keeps going up one level, and so on. If the outermost global scope is reached, the search stops, whether it finds the variable or not.

