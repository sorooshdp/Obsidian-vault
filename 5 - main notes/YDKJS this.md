2025/09/19  -  22:53
status: 

Tags: [[js]] [[this-keyword]]

One of the most confused mechanisms in JavaScript is the `this` keyword. It's a special identifier keyword that's automatically defined in the scope of every function, but what exactly it refers to bedevils even seasoned JavaScript developers.

`this` does not, in any way, refer to a function's lexical scope. It is true that internally, scope is kind of like an object with properties for each of the available identifiers. But the scope "object" is not accessible to JavaScript code. It's an inner part of the Engine's implementation.

When a function is invoked, an activation record, otherwise known as an execution context, is created. This record contains information about where the function was called from (the call-stack), how the function was invoked, what parameters were passed, etc. One of the properties of this record is the `this` reference which will be used for the duration of that function's execution.
 
To understand `this` binding, we have to understand the call-site: the location in code where a function is called (not where it's declared). We must inspect the call-site to answer the question: what's this `this` a reference to?

What's important is to think about the call-stack (the stack of functions that have been called to get us to the current moment in execution). The call-site we care about is in the invocation before the currently executing function.

```javascript
function baz() { 
// call-stack is: `baz` 
// so, our call-site is in the global scope
console.log( "baz" ); 
bar(); // <-- call-site for `bar` 
} 

function bar() { 
// call-stack is: `baz` -> `bar` 
// so, our call-site is in `baz` 
console.log( "bar" ); 
foo(); // <-- call-site for `foo` 
} 

function foo() { 
// call-stack is: `baz` -> `bar` -> `foo` 
// so, our call-site is in `bar` 
console.log( "foo" ); 
} 

baz(); // <-- call-site for `baz`
```

To determine where this points to, we have 4 rules. after finding the call-site we can you this rules to find out where "this" points to.

## Default Binding

The first rule we will examine comes from the most common case of function calls: standalone function invocation. Think of this `this` rule as the default catch-all rule when none of the other rules apply.

```javascript
function foo() { 
console.log( this.a ); 
} 
var a = 2; 
foo(); // 2
```

How do we know that the default binding rule applies here? We examine the call-site to see how foo() is called. In our snippet, foo() is called with a plain, un-decorated function reference. None of the other rules we will demonstrate will apply here, so the default binding applies instead.

```javascript

function foo() { 
"use strict";
 console.log( this.a );
}

var a = 2; 
foo(); // TypeError: `this` is `undefined`
```

## Implicit Binding 

Another rule to consider is: does the call-site have a context object, also referred to as an owning or containing object, though these alternate terms could be slightly misleading.

```javascript
function foo() { 
console.log( this.a ); 
} 

var obj = { a: 2, foo: foo }; 

obj.foo(); // 2
```

Firstly, notice the manner in which foo() is declared and then later added as a reference property onto `obj` . Regardless of whether foo() is initially declared on `obj` , or is added as a reference later (as this snippet shows), in neither case is the function really "owned" or "contained" by the `obj` object. However, the call-site uses the `obj` context to reference the function, so you could say that the `obj` object "owns" or "contains" the function reference at the time the function is called.

## Implicitly Lost 
One of the most common frustrations that this binding creates is when an implicitly bound function loses that binding, which usually means it falls back to the default binding, of either the global object or undefined , depending on strict mode .

```javascript
function foo() { 
console.log( this.a ); 
} 

var obj = { a: 2, foo: foo }; 

var bar = obj.foo; // function reference/alias! 

var a = "oops, global"; // `a` also property on global object
 
bar(); // "oops, global"
```

Even though bar appears to be a reference to reference to obj.foo , in fact, it's really just another foo itself. Moreover, the call-site is what matters, and the call-site is which is a plain, un-decorated call and thus the default binding applies.

# References
