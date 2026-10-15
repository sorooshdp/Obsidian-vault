2026/10/09  -  17:31
status: 

Tags: [[js]] [[closures]]

In JavaScript, **closure** is widely considered one of the most important yet frequently misunderstood concepts. As author Kyle Simpson emphasizes in *You Don't Know JS: Scope & Closures*, closure is not a special opt-in feature or a complex syntax you have to opt into—it is a direct, natural consequence of writing code in a **lexically scoped** environment where functions are first-class values.

---

### 1. What is Closure?

#### **The Formal Definition**
> **Closure** is when a function is able to remember and access its lexical scope even when that function is executing outside its author-time lexical scope.

#### **Understanding the Mechanics**
Under standard lexical scope rules, a function can always look up identifiers in its parent or enclosing scope bubbles. Normally, when an outer function finishes executing, the engine's **Garbage Collector** steps in and reclaims its memory. 

However, when an inner function is passed around as a value and invoked elsewhere, **the inner function maintains a hidden reference to its original scope**. That reference is the closure, and it prevents the Engine from garbage collecting the outer function's scope.

```javascript
function foo() {
    var a = 2;
    function bar() {
        console.log( a );
    }
    return bar; // returning the inner function object itself
}

var baz = foo();
baz(); // 2 — Closure observed!
```
*(Source:)*

* **Why is this closure?** `bar()` is executed via `baz()` well outside of its author-time lexical scope. Even though `foo()` has completed running, `bar()` retains an intact closure over `foo()`'s inner scope, keeping variable `a` alive.

---

### 2. Everyday Examples of Closure

Closure occurs naturally across standard JavaScript code whenever callbacks are passed around as first-class values.

#### **Timers & Asynchronous Callbacks**
```javascript
function wait(message) {
    setTimeout( function timer(){
        console.log( message );
    }, 1000 );
}

wait( "Hello, closure!" );
```
*(Source:)*
* A thousand milliseconds after `wait(..)` finishes executing, the inner `timer()` function still holds a closure over `wait(..)`'s scope, allowing it to access `message`.

#### **Event Handlers & Frameworks**
```javascript
function setupBot(name, selector) {
    $( selector ).click( function activator(){
        console.log( "Activating: " + name );
    } );
}

setupBot( "Closure Bot 1", "#bot_1" );
setupBot( "Closure Bot 2", "#bot_2" );
```
*(Source:)*

---

### 3. The Classic Pitfall: Loops + Closures

The classic illustration of closure confusion involves putting callbacks inside a `for` loop.

#### **The Bug**
```javascript
for (var i=1; i<=5; i++) {
    setTimeout( function timer(){
        console.log( i );
    }, i*1000 );
}
// Outputs: 6, 6, 6, 6, 6 (instead of 1, 2, 3, 4, 5)
```
*(Source:)*

* **Why it fails:** All 5 `timer()` callbacks are defined separately, but they all close over the **same shared global scope** containing a single variable `i`. By the time any callback actually runs, the loop has long completed and `i` holds the value `6`.

#### **Solution 1: IIFE (Creating per-iteration scope pre-ES6)**
To fix this prior to ES6, you must create a new closured block scope for every single iteration using an Immediately Invoked Function Expression (IIFE) and pass in the current value of `i`:

```javascript
for (var i=1; i<=5; i++) {
    (function(j){
        setTimeout( function timer(){
            console.log( j );
        }, j*1000 );
    })( i );
}
```
*(Source:)*

#### **Solution 2: Block Scoping with `let` (Modern ES6)**
ES6 introduced `let`, which hijacks a block and binds variables directly to it. Specifically in a `for` loop header, `let` declares a **brand-new variable for every iteration** of the loop:

```javascript
for (let i=1; i<=5; i++) {
    setTimeout( function timer(){
        console.log( i );
    }, i*1000 );
}
// Outputs: 1, 2, 3, 4, 5
```
*(Source:)*

---

### 4. Closures and Design Patterns: The Module Pattern

The most practical architectural application of closure in JavaScript is the **Module Pattern**.

#### **Requirements for a Module:**
1. **An outer enclosing function** that is executed at least once to create the private inner scope.
2. **At least one inner function returned** from the wrapper function that retains closure over that private scope to serve as the public API.

#### **Revealing Module Pattern**
```javascript
function CoolModule() {
    var something = "cool";      // Private state
    var another =;     // Private state

    function doSomething() {
        console.log( something );
    }

    function doAnother() {
        console.log( another.join( " ! " ) );
    }

    // Public API object
    return {
        doSomething: doSomething,
        doAnother: doAnother
    };
}

var foo = CoolModule();
foo.doSomething(); // "cool"
foo.doAnother();   // "1 ! 2 ! 3"
```
*(Source:)*

#### **Singleton Module (IIFE Variant)**
If you only need a single instance of a module, you can wrap the module definition in an IIFE:

```javascript
var foo = (function CoolModule() {
    var something = "cool";

    function doSomething() {
        console.log( something );
    }

    return {
        doSomething: doSomething
    };
})();

foo.doSomething(); // "cool"
```
*(Source:)*

#### **ES6 Modules vs. Closure Modules**
While traditional module patterns rely on outer functions and closures at runtime, **ES6 Modules** (`import` / `export`) provide static, file-based module support. Each ES6 module file is treated as its own implicit scope closure where exports act as live read-only bindings to internal identifiers.

```javascript
// bar.js
function hello(who) {
    return "Let me introduce: " + who;
}
export hello;

// foo.js
import hello from "bar";
var hungry = "hippo";
function awesome() {
    console.log( hello( hungry ).toUpperCase() );
}
export awesome;
```
*(Source:)*

---

### 5. Lexical Scope vs. `this`: `var self = this` and Arrow Functions

A common point of confusion is using closure to bypass JavaScript's dynamic `this` binding rules.

```javascript
// Pre-ES6 "hack" using closure to save `this`
var obj = {
    count: 0,
    cool: function coolFn() {
        var self = this; // Capture `this` lexically
        setTimeout( function timer(){
            self.count++; // Closured access to `self`
            console.log( "awesome?" );
        }, 100 );
    }
};
obj.cool();
```
*(Source:)*

#### **Arrow Functions (`=>`) & Lexical `this`**
ES6 introduced fat-arrow functions, which **discard dynamic `this` binding entirely** and instead adopt the `this` binding from their enclosing lexical scope. Under the hood, this syntactically codifies the `var self = this` closure pattern into the language.

```javascript
var obj = {
    count: 0,
    cool: function coolFn() {
        if (this.count < 1) {
            setTimeout( () => { // Lexical `this` inherited from coolFn()
                this.count++;
                console.log( "awesome?" );
            }, 100 );
        }
    }
};
obj.cool();
```
*(Source:)*

---

💡 Would you like to explore how closures interact with **asynchronous flow control** (such as Generators + Promises) or examine **ES6 Block Scoping details** like the Temporal Dead Zone (TDZ)?
# References
