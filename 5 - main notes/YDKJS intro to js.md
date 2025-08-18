
status: 

Tags:  [[js]]
# intro to js

js has typed values not typed variables. 
- string 
- number 
- Boolean 
- null and undefined 
- object symbol (new to ES6)
Only values have types in JavaScript; variables are just simple containers for those values.
array and function. But rather than being proper built-in types, these should be thought of more like subtypes -- specialized versions of the object type.

```javascript
var a; 
typeof a; // "undefined" 
a = "hello world"; 
typeof a; // "string" 
a = 42; typeof a; // "number" 
a = true; typeof a; // "boolean" 
a = null; typeof a; // "object" -- weird, bug 
a = undefined; typeof a; // "undefined" 
a = { b: "c" }; typeof a; // "object"
```

functions are a subtype of objects -- typeof returns "function" , which implies that a function is a main type -- and can thus have properties

```javascript
function foo() { 
	return 42; 
} 
foo.bar = "hello world"; 
typeof foo; // "function" 
typeof foo(); // "number" 
typeof foo.bar; // "string"
```

falsy values in js:
- ""
- 0, -0, NaN
- null, undefined
- false
truthy values in js:
- "hello"
- 43
- true
- (arrays)
- (objects)
- functions

## equality
== checks for value equality with coercion allowed, and === checks for value equality without allowing coercion; === is often called "strict equality" for this reason.
## hoisting
Wherever a var appears inside a scope, that declaration is taken to belong to the entire scope and accessible everywhere throughout
## polyfill
used to refer to taking the definition of a newer feature and producing a piece of code that's equivalent to the behavior, but is able to run in older JS environments.

## transpiling 
There's no way to polyfill new syntax that has been added to the language. The new syntax would throw an error in the old JS engine as unrecognized/invalid. So the better option is to use a tool that converts your newer code into older code equivalents. This process is commonly called "transpiling," a term for transforming + compiling.

# References
