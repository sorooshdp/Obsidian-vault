---
created: 2025/09/19  -  22:53
updated: 2026-11-09 22:45
type: note
status: seed
topic:
tags:
  - web/javascript
aliases: []
---

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
The more subtle, more common, and more unexpected way this occurs is when we consider passing a callback function:

```javascript
function foo() { 
	console.log( this.a );
} 
function doFoo(fn) { // `fn` is just another reference to `foo` 
	fn(); // <-- call-site! 
} 

var obj = { a: 2, foo: foo }; 

var a = "oops, global"; // `a` also property on global object 

doFoo( obj.foo ); // "oops, global"
```

Parameter passing is just an implicit assignment, and since we're passing a function, it's an implicit reference assignment, so the end result is the same as the previous snippet. What if the function you're passing your callback to is not your own, but built-in to the language? No difference, same outcome.

```javascript
function foo() { 
	console.log( this.a );
} 

var obj = { a: 2, foo: foo }; 

var a = "oops, global"; // `a` also property on global object 

setTimeout( obj.foo, 100 ); // "oops, global"

```

All" functions in the language have some utilities available to them (via their -- more on that later) which can be useful for this task. Specifically, functions have and [[Prototype]] `call(..) apply(..)` methods.

```javascript
function foo() { console.log( this.a ); } 
var obj = { a: 2 }; 
foo.call( obj ); // 2
```

Note: As of ES6, the hard-bound function produced by bind(..) has a .name property that derives from the original target function. For example: `bar = foo.bind(..)` should have a `bar.name` value of "bound foo" , which is the function call name that should show up in a stack trace.

## `new` Binding

there really is no connection to class oriented functionality implied by new usage in JS. In JS, constructors are just functions that happen to be called with the new operator in front of them.

When a function is invoked with new in front of it, otherwise known as a constructor call, the following things are done automatically: 
1. a brand new object is created (aka, constructed) out of thin air 
2. the newly constructed object is [[Prototype]] -linked 
3. the newly constructed object is set as the `this` binding for that function call 
4. unless the function returns its own alternate object, the new -invoked function call will automatically return the newly constructed object.

```js
function foo(a) { this.a = a; } 
var bar = new foo( 2 ); 
console.log( bar.a ); // 2
```
## Binding Precedence

Now, we can summarize the rules for determining this from a function call's call-site, in their order of precedence. Ask these questions in this order, and stop when the first rule applies. 
1. Is the function called with new (new binding)? If so, object.` var bar = new foo() `
2. Is the function called with call or bind hard binding? If so, this is the newly constructed apply (explicit binding), even hidden inside a this is the explicitly specified object. `var bar = foo.call( obj2 )` 
3. Is the function called with a context (implicit binding), otherwise known as an owning or containing object? If so, this is that context object. `var bar = obj1.foo()`
4. Otherwise, default the this (default binding). If in otherwise pick the global object. `var bar = foo()`

Note: new and call / apply cannot be used together, so `new foo.call(obj1)` is not allowed, to test new binding directly against explicit binding. But we can still use a hard binding to test the precedence of the two rules.

Why is new being able to override hard binding useful? The primary reason for this behavior is to create a function (that can be used with constructing objects) that essentially ignores the new for this hard binding but which presets some or all of the function's arguments. One of the capabilities of bind(..) is that any arguments passed after the first this binding argument are defaulted as standard arguments to the underlying function (technically called "partial application", which is a subset of "currying"). 

## Binding Exceptions: The DMZ Object

Sometimes you need to pass a placeholder for `this` when you don't actually care about the binding (such as when spreading array values with `.apply` or currying arguments with `.bind`).

Using `null` or `undefined` as a placeholder falls back to the default binding rule, which risks accidentally referencing or mutating the global `window` object if the target function contains a `this` reference. To protect against this, you can create a **"DMZ" (demilitarized zone) object** using `Object.create(null)`. This creates a completely empty object with no link to `Object.prototype`, making it much safer.

---

## ES6 Arrow Functions and Lexical `this`

ES6 introduced **arrow functions (****=>****)**, which completely discard the four standard rules of `this` binding.

Instead of binding `this` dynamically at call-time, arrow functions adopt the `this` binding from their **immediate enclosing lexical scope**. This behaves exactly like the popular pre-ES6 pattern of saving a lexical copy of `this` (e.g., `var self = this`).

This is incredibly useful for inline callbacks (such as event handlers or timers) where you want `this` to predictably inherit the outer method's context:

```js
function foo() {
    // The arrow function lexically inherits `this` from `foo`
    setTimeout(() => {
        console.log(this.a); 
    }, 100);
}
var obj = { a: 2 };
foo.call(obj); // 2
```

If you find yourself writing this instead of understanding and this -style code, but most or all the time, you defeat the mechanism with lexical this self = this or arrow-function "tricks", perhaps you should either: 
1. Use only lexical scope and forget the false pretense of this -style code. 
2. Embrace this -style mechanisms completely, including using bind(..) where necessary, and try to avoid self = this and arrow-function "lexical this" tricks. 

A program can effectively use both styles of code (lexical and this ), but inside of the same function, and indeed for the same sorts of look-ups, mixing the two mechanisms is usually asking for harder-to-maintain code, and probably working too hard to be clever.

## Review (TL;DR) 

Determining the this binding for an executing function requires finding the direct call-site of that function. Once examined, four rules can be applied to the call-site, in this order of precedence: 
1. Called with new ? Use the newly constructed object. 
2. Called with call or apply (or bind )? Use the specified object. 
3. Called with a context object owning the call? Use that context object. 
4. Default: undefined in strict mode , global object otherwise.
Be careful of accidental/unintentional invoking of the default binding rule. In cases where you want to "safely" ignore a this binding, a "DMZ" object like good placeholder value that protects the `ø = Object.create(null)` is a global object from unintended side-effects.

# References
