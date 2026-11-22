---
created: 2026-10-17 20:39
updated: 2026-10-17 20:39
type: note
status: seed
tags:
  - rust
aliases: []
topic: "[[Rust]]"
---

## Summary


## Notes
In **Chapter 19: Advanced Features**, *The Rust Programming Language* provides a reference guide for specialized language features designed for low-level systems programming, advanced type design, and complex API abstractions.

---

### **1. Unsafe Rust: Opting Out of Compile-Time Guarantees**

All standard Rust code enforces memory safety at compile time. However, because static analysis is inherently conservative, the compiler may reject valid programs it cannot mathematically verify. To perform low-level tasks—such as interacting directly with hardware or building performance-critical abstractions—Rust provides **Unsafe Rust**.

Annotating a block with **`unsafe`** does not disable the borrow checker or turn off standard safety checks on references; it simply grants four explicit **unsafe superpowers**:

1. **Dereferencing a raw pointer**.
2. **Calling an unsafe function or method**.
3. **Accessing or modifying a mutable static variable**.
4. **Implementing an unsafe trait**.

#### **A. Dereferencing Raw Pointers (`*const T` and `*mut T`)**
Raw pointers are denoted by `*const T` (immutable raw pointer) and `*mut T` (mutable raw pointer). Unlike references or smart pointers, raw pointers:
* Are allowed to ignore borrowing rules by having simultaneous mutable and immutable pointers to the same memory location.
* Are not guaranteed to point to valid memory and can be `null`.
* Do not implement automatic cleanup (`drop`).

Creating raw pointers is allowed in safe Rust; however, **dereferencing** raw pointers to read or write data requires an `unsafe` block.

```rust
let mut num = 5;

// Creating raw pointers in safe code
let r1 = &num as *const i32;
let r2 = &mut num as *mut i32;

// Dereferencing raw pointers requires an unsafe block
unsafe {
    println!("r1 is: {}", *r1);
    println!("r2 is: {}", *r2);
}
```
*(Source: Listings 19-1 & 19-3)*

#### **B. Creating Safe Abstractions over Unsafe Code (`split_at_mut`)**
A common Rust pattern is wrapping unsafe code inside a safe API. For example, the standard library function **`split_at_mut`** splits a mutable slice into two non-overlapping mutable slices at a given index.

The borrow checker rejects a safe implementation because it sees two mutable borrows targeting the same underlying slice. Using raw pointers (`as_mut_ptr()`, `offset()`, and `slice::from_raw_parts_mut()`) inside an `unsafe` block allows constructing non-overlapping slices safely:

```rust
use std::slice;

fn split_at_mut(slice: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = slice.len();
    let ptr = slice.as_mut_ptr();

    assert!(mid <= len);

    unsafe {
        (
            slice::from_raw_parts_mut(ptr, mid),
            slice::from_raw_parts_mut(ptr.offset(mid as isize), len - mid),
        )
    }
}
```
*(Source: Listing 19-6)*

#### **C. Foreign Function Interface (`extern` Blocks)**
To interoperate with other languages (such as C), Rust uses the **`extern`** keyword to set up a Foreign Function Interface (FFI). Functions declared inside `extern` blocks are treated as unsafe to call because foreign languages do not enforce Rust's safety guarantees.

```rust
extern "C" {
    fn abs(input: i32) -> i32;
}

fn main() {
    unsafe {
        println!("Absolute value of -3 according to C: {}", abs(-3));
    }
}
```
*(Source: Listing 19-8)*

Conversely, exposing a Rust function to C requires using `#[no_mangle]` (to preserve the function symbol name) and `pub extern "C"`:

```rust
#[no_mangle]
pub extern "C" fn call_from_c() {
    println!("Just called a Rust function from C!");
}
```
*(Source: Foreign Function Interface)*

#### **D. Mutable Static Variables (`static mut`)**
Global variables in Rust are called **static variables**. While immutable static variables (`static`) are safe to read, **`static mut`** variables can cause data races if accessed concurrently across threads. Consequently, reading or writing to a `static mut` variable requires an `unsafe` block.

```rust
static mut COUNTER: u32 = 0;

fn add_to_count(inc: u32) {
    unsafe {
        COUNTER += inc;
    }
}
```
*(Source: Listing 19-10)*

#### **E. Implementing Unsafe Traits**
A trait is marked `unsafe` when at least one of its methods relies on invariants that the compiler cannot automatically verify. Implementing an unsafe trait (such as `Send` or `Sync` for types wrapping raw pointers) requires `unsafe impl`:

```rust
unsafe trait Foo {
    // Methods go here
}

unsafe impl Foo for i32 {
    // Method implementations go here
}
```
*(Source: Listing 19-11)*

---

### **2. Advanced Lifetimes**

#### **A. Lifetime Subtyping (`'b: 'a`)**
**Lifetime subtyping** specifies that one lifetime parameter lives at least as long as another. The syntax `'s: 'c` is read as *"lifetime `'s` outlives lifetime `'c`"*.

This syntax is required when a struct holds a reference to another struct containing references (such as a `Parser` holding a reference to a `Context`), ensuring the inner referenced data remains valid for the duration of the outer reference:

```rust
struct Context<'s>(&'s str);

// Specifies that 's must outlive 'c ('s: 'c)
struct Parser<'c, 's: 'c> {
    context: &'c Context<'s>,
}

impl<'c, 's> Parser<'c, 's> {
    fn parse(&self) -> Result<(), &'s str> {
        Err(&self.context.0[1..])
    }
}
```
*(Source: Lifetime Subtyping)*

#### **B. Lifetime Bounds on Generic Types (`T: 'a`)**
You can apply lifetime bounds to generic type parameters (`T: 'a`) to ensure that any references contained inside the generic type `T` live at least as long as `'a`:

```rust
// Specifies that any references inside T must outlive 'a
struct Ref<'a, T: 'a>(&'a T);

// Specifies that T contains no borrowed references or only 'static ones
struct StaticRef<T: 'static>(&'static T);
```
*(Source: Listings 19-17 & 19-18)*

#### **C. Trait Object Lifetime Inference Rules**
When trait objects hold references with lifetimes, Rust applies default inference rules:
* The default lifetime of an unbounded trait object is **`'static`**.
* For `&'a Trait` or `&'a mut Trait`, the default trait object lifetime is **`'a`**.
* For `T: 'a`, the default lifetime of the trait object is **`'a`**.
* If multiple conflicting bounds exist, the lifetime must be specified explicitly (e.g., `Box<Red + 'a>` or `Box<Red + 'static>`).

---

### **3. Advanced Traits**

#### **A. Associated Types vs. Generics**
**Associated types** connect placeholder type identifiers with a trait definition (e.g., `type Item;` inside `Iterator`).

```rust
pub trait Iterator {
    type Item; // Associated placeholder type

    fn next(&mut self) -> Option<Self::Item>;
}
```
*(Source: Listing 19-20)*

##### **Why Use Associated Types Instead of Generics?**
* **Generic Trait (`trait Iterator<Item>`)**: Allows implementing `Iterator` for a concrete type multiple times with different type arguments. However, this forces every call to `.next()` to include explicit type annotations to resolve ambiguity.
* **Associated Type (`type Item;`)**: Allows only **one implementation** of `Iterator` per type, eliminating the need for type annotations at call sites.

#### **B. Default Generic Parameters & Operator Overloading**
You can specify default concrete types for generic parameters using `<Placeholder=ConcreteType>`. This is used extensively for **operator overloading** via traits in `std::ops`.

For example, `std::ops::Add` defines `trait Add<RHS=Self>`, defaulting the right-hand side (`RHS`) to `Self`:

```rust
use std::ops::Add;

struct Millimeters(u32);
struct Meters(u32);

// Customizing RHS to add Meters to Millimeters
impl Add<Meters> for Millimeters {
    type Output = Millimeters;

    fn add(self, other: Meters) -> Millimeters {
        Millimeters(self.0 + (other.0 * 1000))
    }
}
```
*(Source: Listing 19-23)*

#### **C. Fully Qualified Syntax for Disambiguation**
When two implemented traits (or an inherent method) share identical function names, Rust requires explicit disambiguation.

* **Methods with `self`**: Call `Pilot::fly(&person);` or `Wizard::fly(&person);`.
* **Associated Functions without `self`**: Use **fully qualified syntax** (`<Type as Trait>::function(...)`) because no receiver object exists to infer the type:

```rust
trait Animal {
    fn baby_name() -> String;
}

struct Dog;

impl Dog {
    fn baby_name() -> String { String::from("Spot") }
}

impl Animal for Dog {
    fn baby_name() -> String { String::from("puppy") }
}

fn main() {
    // Calls Dog's inherent function: "Spot"
    println!("A baby dog is called a {}", Dog::baby_name());

    // Fully qualified syntax calls Animal's trait function: "puppy"
    println!("A baby dog is called a {}", <Dog as Animal>::baby_name());
}
```
*(Source: Listings 19-27 & 19-29)*

#### **D. Supertraits**
When a trait definition requires another trait's functionality, the dependent trait is called a **supertrait**.

```rust
use std::fmt;

// OutlinePrint requires fmt::Display as a supertrait
trait OutlinePrint: fmt::Display {
    fn outline_print(&self) {
        let output = self.to_string(); // Available because Display is required
        let len = output.len();
        println!("{}", "*".repeat(len + 4));
        println!("* {} *", output);
        println!("{}", "*".repeat(len + 4));
    }
}
```
*(Source: Listing 19-30)*

#### **E. The Newtype Pattern to Bypass the Orphan Rule**
The **orphan rule** dictates that you can implement a trait on a type only if either the trait or the type is local to your crate. To implement an external trait (e.g., `Display`) on an external type (e.g., `Vec<T>`), wrap the external type inside a local single-element tuple struct (**newtype pattern**):

```rust
use std::fmt;

// Wrapper is a local newtype around external Vec<String>
struct Wrapper(Vec<String>);

impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}
```
*(Source: Listing 19-31)*

---

### **4. Advanced Types**

#### **A. Newtypes for Type Safety and Encapsulation**
Beyond bypassing the orphan rule, newtypes statically prevent unit mix-ups (e.g., distinguishing `Millimeters(u32)` from `Meters(u32)`) and encapsulate private inner structures (e.g., wrapping a `HashMap` inside a `People` struct).

#### **B. Type Aliases / Synonyms (`type`)**
The **`type`** keyword creates a **type alias** (a synonym) for an existing type without creating an entirely new type:

```rust
// Type synonym (Kilometers and i32 are treated identically)
type Kilometers = i32;

// Reducing repetition for verbose closure types
type Thunk = Box<Fn() + Send + 'static>;

// Standard library alias in std::io
type Result<T> = std::result::Result<T, std::io::Error>;
```
*(Source: Listings 19-33 & Type Aliases)*

#### **C. The Never Type (`!`) and Diverging Functions**
Rust features a special type named **`!`** (known as the **never type** or empty type) for **diverging functions** that never return. 

Expressions of type `!` can be transparently coerced into any other type, which is why `match` arms containing `continue`, `break`, `loop`, or `panic!` compile cleanly regardless of the other arms' return types:

```rust
let guess: u32 = match guess.trim().parse() {
    Ok(num) => num,
    Err(_) => continue, // continue has type !, coercing to u32
};
```
*(Source: Listing 19-34)*

#### **D. Dynamically Sized Types (DSTs) and the `Sized` Trait**
**Dynamically Sized Types (DSTs)** or unsized types represent values whose size can be determined only at runtime (such as `str` or trait objects).

* **The Golden Rule of DSTs**: Values of unsized types must **always be placed behind a pointer** (e.g., `&str`, `Box<str>`, `Box<dyn Trait>`) that stores both the address and metadata (like slice length).
* **The `Sized` Trait**: Automatically implemented for types whose size is known at compile time. All generic functions implicitly include a `T: Sized` bound.
* **Relaxing `Sized` (`?Sized`)**: To allow a generic function to accept DSTs, use the **`?Sized`** bound (*"T may or may not be Sized"*) and pass parameters by reference (`&T`):

```rust
// Generic function accepting both Sized types and Dynamically Sized Types
fn generic<T: ?Sized>(t: &T) {
    // ...
}
```
*(Source: Relaxing `Sized`)*

---

### **5. Advanced Functions and Closures**

#### **A. Function Pointers (`fn` Type)**
Regular functions coerce to the **`fn`** (lowercase f) **function pointer type**. Unlike closure traits (`Fn`), `fn` is a concrete type. Function pointers implement all three closure traits (`Fn`, `FnMut`, `FnOnce`), allowing function pointers to be passed to functions that expect closures.

```rust
fn add_one(x: i32) -> i32 {
    x + 1
}

// Accepts a function pointer `f`
fn do_twice(f: fn(i32) -> i32, arg: i32) -> i32 {
    f(arg) + f(arg)
}

fn main() {
    let answer = do_twice(add_one, 5);
    println!("The answer is: {}", answer); // Prints: 12
}
```
*(Source: Listing 19-35)*

#### **B. Returning Closures**
Because closures have unique, unnameable compiler-generated types that do not have a statically known size, returning a closure directly fails the `Sized` trait bound. To return a closure from a function, wrap it inside a **trait object** (`Box<Fn(...) -> ...>`):

```rust
fn returns_closure() -> Box<Fn(i32) -> i32> {
    Box::new(|x| x + 1)
}
```
*(Source: Returning Closures)*

---

🦀 

## References
