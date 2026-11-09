---
created: 2026/09/28  -  11:26
updated: 2026-11-09 22:45
type: note
status: seed
topic:
tags:
  - web/javascript
aliases: []
---


**Chapter 5: Prototypes** of _You Don't Know JS: this & Object Prototypes_ examines JavaScript's internal `[[Prototype]]` mechanism. It unpacks how property lookups work across object links, dispels misconceptions around "prototypal inheritance" and "constructors," and demonstrates how to properly wire objects together.

---

### 1. The `[[Prototype]]` Mechanism & Chain

- **Internal Reference:** Objects in JavaScript possess an internal specification property denoted as `[[Prototype]]`, which is a reference to another object.
- **Property Look-Up (`[[Get]]`):** When accessing a property on an object (such as `myObject.a`), the engine's default `[[Get]]` operation first inspects the object itself. If the property is missing, it follows the `[[Prototype]]` link.
- **Top of the Chain:** The traversal continues link-by-link until either the property is found or the chain ends at **`Object.prototype`**. If the property is not found anywhere on the chain, `[[Get]]` returns `undefined`.
- **Enumeration & Existence:** The `for..in` loop iterates over all accessible enumerable properties on the object and its prototype chain, while the `in` operator checks existence across the entire chain regardless of enumerability.

```js
var anotherObject = {
    a: 2
};

// Create an object linked to `anotherObject`
var myObject = Object.create( anotherObject );

myObject.a; // 2

for (var k in myObject) {
    console.log("found: " + k);
}
// found: a

("a" in myObject); // true
```

---

### 2. Setting & Shadowing Properties

When assigning a property to an object (`myObject.foo = "bar"`), if `foo` is **not** present directly on `myObject` but exists higher up on its `[[Prototype]]` chain, one of three scenarios occurs:

1. **Normal Data Property (`writable: true`):** A new data property `foo` is added directly to `myObject`, resulting in a **shadowed property**.
2. **Read-Only Data Property (`writable: false`):** **No property is created on `myObject` and no shadowing occurs**. In `strict mode`, a `TypeError` is thrown; otherwise, the assignment fails silently.
3. **Setter Property:** The setter function higher on the chain is invoked, and no `foo` property is added to `myObject`.

> **Note:** To force shadowing in scenarios 2 and 3, you must use `Object.defineProperty(myObject, "foo", { ... })` instead of `=` assignment.

#### **Implicit Shadowing Trap**

```js
var anotherObject = {
    a: 2
};

var myObject = Object.create( anotherObject );

anotherObject.a; // 2
myObject.a;      // 2

myObject.a++; // Oops, implicit shadowing!
// Evaluates to: myObject.a = myObject.a + 1

anotherObject.a; // 2
myObject.a;      // 3
myObject.hasOwnProperty( "a" ); // true
```

---

### 3. "Class" Mechanics and Constructor Deceptions

JavaScript does not perform copy actions from classes to instances; it only links objects together.

- **Function Prototypes:** All functions automatically receive a public, non-enumerable property called `.prototype`, pointing to an arbitrary object. Calling `new Foo()` creates a brand-new object that is `[[Prototype]]`-linked to `Foo.prototype`.
- **Constructor Calls:** Functions themselves are not constructors; `new` simply triggers a **constructor call** of a normal function.
- **The `.constructor` Illusion:** By default, `Foo.prototype` gets a `.constructor` property pointing back to `Foo`. Objects created via `new Foo()` appear to have a `.constructor` property, but this is merely a delegated lookup to `Foo.prototype.constructor`.

#### **Why `.constructor` Cannot Be Trusted**

If you replace a function's prototype object, `.constructor` disappears unless manually re-configured:

```js
function Foo() { /* .. */ }

Foo.prototype = { /* .. */ }; // Replaced default prototype object

var a1 = new Foo();
a1.constructor === Foo; // false!
a1.constructor === Object; // true! (delegated up to Object.prototype)

// Fixing .constructor requires manual configuration:
Object.defineProperty( Foo.prototype, "constructor", {
    enumerable: false,
    writable: true,
    configurable: true,
    value: Foo // point .constructor back to Foo
} );
```

---

### 4. Linking Prototype Chains ("Inheritance")

To link a "child class" prototype (`Bar.prototype`) to a "parent class" prototype (`Foo.prototype`), you establish a prototype linkage:

```js
function Foo(name) {
    this.name = name;
}

Foo.prototype.myName = function() {
    return this.name;
};

function Bar(name, label) {
    Foo.call( this, name );
    this.label = label;
}

// Pre-ES6: Creates a new object linked to Foo.prototype
Bar.prototype = Object.create( Foo.prototype );

// ES6+: Modifies existing Bar.prototype in place
// Object.setPrototypeOf( Bar.prototype, Foo.prototype );

Bar.prototype.myLabel = function() {
    return this.label;
};

var a = new Bar( "a", "obj a" );
a.myName();  // "a"
a.myLabel(); // "obj a"
```

#### **Common Anti-Patterns to Avoid:**

- **`Bar.prototype = Foo.prototype;`** — Does not create a link; makes `Bar.prototype` a direct reference to `Foo.prototype`. Modifying `Bar.prototype` directly mutates `Foo.prototype`.
- **`Bar.prototype = new Foo();`** — Creates a link, but executes `Foo(..)` as a constructor call, triggering unintended side-effects at linkage time.

---

### 5. Inspecting Prototype Relationships (Introspection)

To check if an object delegates to another object across the prototype chain:

- **`instanceof`:** Tests if `Foo.prototype` appears anywhere in `a`'s `[[Prototype]]` chain. Requires a constructor function `Foo`.
- **`isPrototypeOf(..)` (Cleaner):** Checks directly whether an object appears in another object's `[[Prototype]]` chain.
- **`Object.getPrototypeOf(a)`:** Standard ES5 utility to retrieve an object's `[[Prototype]]`.
- **`__proto__` ("dunder proto"):** Non-standard (standardized in ES6) property exposing the internal `[[Prototype]]`. It is actually implemented as a getter/setter on `Object.prototype`.

```js
function Foo() { /* .. */ }
var a = new Foo();

// instanceof check
a instanceof Foo; // true

// Direct object-to-object check (no constructor function required)
Foo.prototype.isPrototypeOf( a ); // true
Object.getPrototypeOf( a ) === Foo.prototype; // true
a.__proto__ === Foo.prototype; // true
```

#### **How `__proto__` is implemented internally:**

```js
Object.defineProperty( Object.prototype, "__proto__", {
    get: function() {
        return Object.getPrototypeOf( this );
    },
    set: function(o) {
        Object.setPrototypeOf( this, o );
        return o;
    }
} );
```

---

### 6. `Object.create(..)` and Behavior Delegation

`Object.create(..)` simplifies prototype linkage by creating objects directly linked to other objects without class syntax or constructor functions.

```js
var foo = {
    something: function() {
        console.log( "Tell me something good..." );
    }
};

var bar = Object.create( foo );
bar.something(); // Tell me something good...
```

#### **`Object.create(null)` ("Dictionary" Objects)**

Calling `Object.create(null)` produces an object with an empty (`null`) `[[Prototype]]` link. Because it has no prototype chain, it does not inherit utilities like `.toString()` or `.hasOwnProperty()` and is ideal for flat data storage.

#### **Polyfill for `Object.create(..)` (Pre-ES5)**

```js
if (!Object.create) {
    Object.create = function(o) {
        function F(){}
        F.prototype = o;
        return new F();
    };
}
```

#### **Internal Delegation vs. Magic Fallbacks**

Rather than relying on `[[Prototype]]` links as fallback "magic" where missing methods are implicitly resolved, it is clearer API design to use explicit internal delegation:

```js
var anotherObject = {
    cool: function() {
        console.log( "cool!" );
    }
};

var myObject = Object.create( anotherObject );

// Explicit internal delegation method
myObject.doCool = function() {
    this.cool(); // internal delegation!
};

myObject.doCool(); // "cool!"
```


# References
