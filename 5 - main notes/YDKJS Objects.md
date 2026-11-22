---
created: 2026/09/15  -  12:47
updated: 2026-11-09 22:47
type: note
status: seed
topic:
tags:
  - web/javascript
aliases: []
---

### 1. Object Syntax and Types

Objects in JavaScript can be created in two ways:

- **Declarative (Literal) Form:** `var myObj = { key: value };`
- **Constructed Form:** `var myObj = new Object(); myObj.key = value;`

Both forms produce the exact same type of object. However, the **literal form is almost universally preferred** because it allows you to declare multiple properties at once, whereas the constructed form forces you to assign properties one-by-one.

#### The Object Type vs. Primitives

Objects are one of the primary language types in JavaScript. Simple primitives—such as `string`, `number`, `boolean`, `null`, and `undefined`—are **not** objects. (The output of `typeof null === "object"` is a famous, long-standing language bug; `null` is actually its own primitive type).

- **Sub-types of Objects:** Specialized object sub-types (often called complex primitives) include **Functions** (which are "callable objects" with an internal `[[Call]]` property) and **Arrays** (which feature numeric indexing and auto-updated `.length` properties).
- **Natives (Object Wrappers):** Primitives do not have properties or methods. However, when you perform an operation like `"abc".length`, the JavaScript engine automatically **"boxes"** (coerces) the primitive string into a `String` object wrapper under the covers so that the property access succeeds. It is highly recommended to stick to primitive literals and let the engine handle boxing implicitly.

---

### 2. Contents and Property Access

An object's contents do not actually live "inside" the object container. Instead, the object stores **property names** which act as pointers (references) to where the actual values are stored.

- **Dot vs. Bracket Notation:** Properties can be accessed via dot notation (`myObj.a`) or bracket notation (`myObj["a"]`). Bracket notation is required if your property name contains special characters (like `"Super-Fun!"`) or if you are dynamically building the key name using a variable.
- **Computed Property Names (ES6):** You can use an expression surrounded by `[ ]` inside an object-literal declaration to dynamically compute the key name at author-time. This is highly useful for assigning **Symbol** properties.
- **Properties vs. Methods:** In JavaScript, functions never truly "belong" to an object, even if they are declared inside the literal. Because `this` is dynamically bound at run-time at the call-site, "function" and "method" are essentially interchangeable terms in JavaScript.

---

### 3. Property Descriptors (ES5)

Since ES5, all properties are defined in terms of a **property descriptor** (or "data descriptor"). You can inspect these characteristics using `Object.getOwnPropertyDescriptor(..)` or modify them using `Object.defineProperty(..)`. Descriptors contain four key attributes:

1. **`value`:** The actual data value stored.
2. **`writable`:** Controls whether you can mutate the property's value. If `writable` is `false`, assignments will silently fail in non-strict mode and throw a `TypeError` in strict mode.
3. **`configurable`:** Controls whether you can modify the descriptor attributes or `delete` the property. Setting `configurable` to `false` is a permanent, **one-way action** that cannot be undone.
4. **`enumerable`:** Controls whether the property will show up when iterating through the object's properties (such as in a `for..in` loop).

---

### 4. Object Immutability

JavaScript allows you to configure objects to prevent accidental or intentional changes. However, all built-in immutability is **shallow**—if an immutable object contains a reference to another object (like an array), the contents of that nested object remain fully mutable.

- **Object Constant:** Created by defining a property with both `writable: false` and `configurable: false`.
- **Prevent Extensions:** Calling `Object.preventExtensions(myObj)` stops any new properties from being added to the object.
- **Seal:** `Object.seal(myObj)` prevents extensions and marks all existing properties as `configurable: false` (values can still be modified, but no properties can be added, deleted, or reconfigured).
- **Freeze:** `Object.freeze(myObj)` is the highest level of shallow immutability. It seals the object and marks all data descriptors as `writable: false` so their values cannot be changed.

---

### 5. `[[Get]]` and `[[Put]]` Internal Operations

Interactions with object properties trigger default internal specification algorithms:

- **`[[Get]]`:** When accessing a property, the default `[[Get]]` operation inspects the object for the property. If it cannot find it, it traverses the `[[Prototype]]` chain. If the property cannot be found anywhere, **it returns `undefined`**. This is highly distinct from accessing an undeclared variable in lexical scope, which throws a `ReferenceError`.
- **`[[Put]]`:** When setting a property, `[[Put]]` behaves differently based on whether the property is already present. If present, it checks if it is an accessor descriptor (invoking the setter), a read-only data descriptor (failing or throwing a `TypeError`), or a normal property (setting the value).

---

### 6. Getters, Setters, and existence

- **Getters and Setters:** You can override default `[[Get]]` and `[[Put]]` operations on a per-property level by defining **accessor descriptors**. These are properties that do not hold a value but instead execute hidden functions to retrieve (`get`) or assign (`set`) values.
- **Existence checks:** Since missing properties and properties explicitly set to `undefined` both return `undefined` when accessed, you must use specific utilities to check for existence:
    - **The `in` operator:** Checks if a property name exists on the object _or_ any higher level of its `[[Prototype]]` chain.
    - **`hasOwnProperty(..)`:** Checks _only_ if the direct object contains the property.
- **Enumeration vs. Iteration:**
    - `Object.keys(..)` returns an array of all **enumerable, owned** properties, while `Object.getOwnPropertyNames(..)` returns all owned properties regardless of enumerability. Both inspect only the direct object.
    - `for..in` loops iterate over all enumerable properties, including those on the prototype chain.
    - `for..of` (ES6) loops directly over **values** rather than properties by calling the object's internal `@@iterator` (or `Symbol.iterator`). Arrays have a built-in iterator, but plain objects do not have one by default (though you can manually define a custom iterator).

---

### 7. Duplicating Objects

Copying an object is highly complex because of the choice between a **[[Deep Copy vs. Shallow Copy#Shallow Copy Defined|shallow copy]]** and a **[[Deep Copy vs. Shallow Copy#Deep Copy Defined|deep copy]]** (which can run into infinite circular duplication loops). ES5 introduced **`Object.assign(..)`**, which performs a shallow, `=` style copy of all enumerable, owned keys from source objects to a target object.

---

# References
