---
created: 2026-10-15 22:07
updated: 2026-10-15 22:07
type: note
status: seed
tags:
  - rust
aliases: []
topic: "[[Rust]]"
---

## Summary


## Notes

In **Chapter 13: Functional Language Features: Iterators and Closures**, *The Rust Programming Language* explores features inspired by functional programming styles. It focuses on two core primitives—**closures** and **iterators**—showing how they enable high-level abstractions without incurring runtime performance costs.

---

### **1. Closures: Anonymous Functions That Capture Their Environment**

A **closure** is an anonymous function that can be stored in a variable, passed as an argument to other functions, or evaluated in a different context. Unlike standard functions (`fn`), closures can capture values from the scope in which they are defined.

#### **A. Closure Syntax vs. Function Syntax**
Closure definitions use vertical pipes (`|`) to surround parameter lists instead of parentheses. Parameter types and return type annotations are optional because the compiler infers them from usage context.

##### **Syntax Comparison Example**
```rust
fn  add_one_v1 = (x: u32) -> u32 { x + 1 } // Standard function
let add_one_v2 = |x: u32| -> u32 { x + 1 }; // Fully annotated closure
let add_one_v3 = |x|             { x + 1 }; // Inferred types closure
let add_one_v4 = |x|               x + 1  ; // Single-expression body
```
*(Source: Listing 13-7)*

#### **B. Type Inference and Concrete Type Locking**
While closures infer their types automatically, the compiler locks in a single concrete type for each parameter and return value on the **first invocation**.

```rust
let example_closure = |x| x;

let s = example_closure(String::from("hello")); // Inferred as String
// let n = example_closure(5); // Compile error! Expected String, found integer
```
*(Source: Listing 13-8)*

---

#### **C. `Memoization` / Lazy Evaluation using `Cacher` and `Fn` Traits**

To prevent executing expensive computations repeatedly or unnecessarily, you can store a closure alongside its cached result inside a struct—a design pattern known as **`memoization`** or **lazy evaluation**.

Every closure implements at least one of the standard library's **`Fn` traits** (`Fn`, `FnMut`, or `FnOnce`).

##### **Code Example: The `Cacher` Struct**
```rust
use std::thread;
use std::time::Duration;

struct Cacher<T>
where
    T: Fn(u32) -> u32, // Trait bound specifying closure parameter and return types
{
    calculation: T,
    value: Option<u32>,
}

impl<T> Cacher<T>
where
    T: Fn(u32) -> u32,
{
    fn new(calculation: T) -> Cacher<T> {
        Cacher {
            calculation,
            value: None,
        }
    }

    fn value(&mut self, arg: u32) -> u32 {
        match self.value {
            Some(v) => v,
            None => {
                let v = (self.calculation)(arg);
                self.value = Some(v);
                v
            }
        }
    }
}
```
*(Source: Listings 13-9, 13-10)*

##### **Limitations of the Basic Cacher:**
1. **Single Argument Value**: Assumes `value(&mut self, arg)` is always called with the same argument (can be fixed using a `HashMap` mapping keys to results).
2. **Fixed Type Bounds**: Restricts parameters and outputs to `u32` (can be fixed using generic parameters).

---

#### **D. How Closures Capture Environment & The `Fn` Traits**

When a closure references a variable from its surrounding scope, Rust manages memory access through one of three traits based on ownership semantics:

1. **`FnOnce`**: Consumes captured variables, moving them out of the environment. It can only be called once because ownership has been transferred.
2. **`FnMut`**: Mutably borrows captured variables to modify the environment in place.
3. **`Fn`**: Immutably borrows captured values from the environment.

#### **E. The `move` Keyword**
To force a closure to take explicit ownership of environment variables (such as when transferring data across thread boundaries), place the **`move`** keyword before the parameter list.

```rust
fn main() {
    let x = vec!;

    // Force closure to take ownership of x
    let equal_to_x = move |z| z == x;

    // println!("can't use x here: {:?}", x); // Compile Error: use of moved value `x`
}
```
*(Source: Capturing Environment with `move`)*

---

### **2. Processing Sequences with Iterators**

Iterators encapsulate the logic of traversing a sequence of items and signaling when the sequence has ended.

#### **A. Lazy Evaluation**
In Rust, **iterators are lazy**: creating an iterator performs no computation until you call methods that consume it.

```rust
let v1 = vec!;
let v1_iter = v1.iter(); // No iteration happens on this line!
```
*(Source: Listing 13-13)*

---

#### **B. The `Iterator` Trait & `next()` Method**
All iterators implement the standard library **`Iterator`** trait:

```rust
pub trait Iterator {
    type Item; // Associated type
    fn next(&mut self) -> Option<Self::Item>;
}
```
*(Source: `Iterator` Trait Definition)*

Calling **`next()`** directly returns an `Option<Self::Item>` (`Some(item)` or `None`) and mutates the iterator's internal state. Therefore, calling `.next()` manually requires the iterator variable to be declared as **`mut`**.

##### **Three Iteration Reference Types:**
* **`.iter()`**: Iterates over **immutable references** (`&T`).
* **`.into_iter()`**: Consumes the collection and yields **owned values** (`T`).
* **`.iter_mut()`**: Iterates over **mutable references** (`&mut T`).

---

#### **C. Consuming Adaptors vs. Iterator Adaptors**

1. **Consuming Adaptors**: Methods that call `next()` internally, consuming the iterator and producing a final result (e.g., **`.sum()`**, **`.collect()`**).
2. **Iterator Adaptors**: Methods that take an iterator and produce a *new* lazy iterator (e.g., **`.map()`**, **`.filter()`**). Adaptors do nothing unless followed by a consuming adaptor.

##### **Code Example: Adaptors in Action**
```rust
let v1: Vec<i32> = vec!;

// Chaining iterator adaptor `.map()` with consuming adaptor `.collect()`
let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();

assert_eq!(v2, vec!);
```
*(Source: Listing 13-18)*

##### **Filtering Environment Capture Example**
```rust
struct Shoe {
    size: u32,
    style: String,
}

fn shoes_in_my_size(shoes: Vec<Shoe>, shoe_size: u32) -> Vec<Shoe> {
    shoes.into_iter()
        .filter(|s| s.size == shoe_size) // Captures `shoe_size` from environment
        .collect()
}
```
*(Source: Listing 13-19)*

---

#### **D. Creating Custom Iterators**
To create a custom iterator, implement the `Iterator` trait on your struct by defining the **`next()`** method. Once `next()` is implemented, your type automatically gains access to all default methods provided by `Iterator` (e.g., `zip`, `map`, `filter`, `sum`).

##### **Code Example: The `Counter` Iterator**
```rust
struct Counter {
    count: u32,
}

impl Counter {
    fn new() -> Counter {
        Counter { count: 0 }
    }
}

impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        self.count += 1;

        if self.count < 6 {
            Some(self.count)
        } else {
            None
        }
    }
}
```
*(Source: Listings 13-20, 13-21)*

---

### **3. Refactoring the `minigrep` Project (`Chapter 12`)**

Iterators allow refactoring the `minigrep` command-line application to eliminate `.clone()` calls and simplify loops.

#### **A. Eliminating `.clone()` in `Config::new`**
Instead of receiving a slice `&[String]` and cloning strings, `Config::new` can take **ownership of the `std::env::Args` iterator** directly.

```rust
impl Config {
    pub fn new(mut args: std::env::Args) -> Result<Config, &'static str> {
        args.next(); // Skip executable path (index 0)

        let query = match args.next() {
            Some(arg) => arg,
            None => return Err("Didn't get a query string"),
        };

        let filename = match args.next() {
            Some(arg) => arg,
            None => return Err("Didn't get a file name"),
        };

        let case_sensitive = env::var("CASE_INSENSITIVE").is_err();

        Ok(Config { query, filename, case_sensitive })
    }
}
```
*(Source: Listing 13-27)*

#### **B. Simplifying `search` with Iterator Adaptors**
Replacing manual `for` loops and mutable result vectors with `.filter()` and `.collect()` produces cleaner, functional code:

```rust
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    contents.lines()
        .filter(|line| line.contains(query))
        .collect()
}
```
*(Source: Listing 13-29)*

---

### **4. Performance: Loops vs. Iterators & Zero-Cost Abstractions**

#### **Zero-Cost Abstraction Principle**
Iterators in Rust are **zero-cost abstractions**. Using higher-level functional abstractions introduces **no additional runtime overhead** compared to hand-written low-level loops.

> *"What you don't use, you don't pay for. And further: What you do use, you couldn't hand code any better."* — Bjarne Stroustrup

#### **Compiler Optimizations Under the Hood:**
* **Loop Unrolling**: When iteration bounds are fixed or known, the compiler unrolls the loop into repeated instructions, removing loop control overhead.
* **Register Storage**: Variables used in iterator chains are stored directly in CPU registers rather than stack memory addresses.
* **Elimination of Bounds Checking**: The compiler proves that iterator bounds remain valid, stripping runtime bounds checks on array/slice accesses.

---
🦀 
## References
